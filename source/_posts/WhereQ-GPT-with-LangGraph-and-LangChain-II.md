---
title: WhereQ-GPT with LangGraph and LangChain - II
date: 2025-05-06 13:10:05
categories:
- LangGraph
- LangChain
tags:
- LangGraph
- LangChain
---

## 🧩 Workflow Overview

1. **User Input**: Receive a natural language question.
2. **Embedding**: Convert the question into a vector representation.
3. **Schema Retrieval**: Use FAISS to find relevant database schemas based on the vector.
4. **Term Mapping**: Map domain-specific terms to database fields using a local knowledge base.
5. **Time Parsing**: Identify and standardize time expressions in the question.
6. **SQL Generation**: Use OpenAI's GPT model to generate the SQL query.
7. **Execution**: Run the SQL query against the database and return results.([Kaggle][2])

---

## 🛠️ Implementation Details

### 1. Setup

Ensure you have the necessary packages installed:([Introduction | 🦜️🔗 LangChain][3])

```bash
pip install langgraph langchain langchain-community openai faiss-cpu
```



### 2. Define the LangGraph Nodes

Each step in the workflow is represented as a node in LangGraph.

#### a. Embedding Node

Converts the user's question into a vector.([YouTube][4])

```python
from langchain.embeddings import OpenAIEmbeddings

def embed_question(state):
    question = state['question']
    embedding_model = OpenAIEmbeddings()
    vector = embedding_model.embed_query(question)
    state['vector'] = vector
    return state
```



#### b. Schema Retrieval Node

Uses FAISS to find relevant schemas.

```python
import faiss
import numpy as np

def retrieve_schema(state):
    vector = np.array(state['vector']).astype('float32')
    D, I = state['faiss_index'].search(np.array([vector]), k=3)
    matched_schemas = [state['schema_texts'][i] for i in I[0]]
    state['matched_schemas'] = matched_schemas
    return state
```



#### c. Term Mapping Node

Maps domain-specific terms to database fields.

```python
def map_terms(state):
    question = state['question']
    term_mappings = state['term_mappings']  # e.g., {'soft drinks': 'beverages'}
    for term, field in term_mappings.items():
        question = question.replace(term, field)
    state['mapped_question'] = question
    return state
```



#### d. Time Parsing Node

Identifies and standardizes time expressions.

```python
import re
from datetime import datetime

def parse_time(state):
    question = state['mapped_question']
    # Simple example for Q3 2024
    match = re.search(r'Q([1-4]) (\d{4})', question)
    if match:
        quarter = int(match.group(1))
        year = int(match.group(2))
        month_start = (quarter - 1) * 3 + 1
        start_date = datetime(year, month_start, 1).strftime('%Y-%m-%d')
        end_month = month_start + 2
        end_date = datetime(year, end_month, 28).strftime('%Y-%m-%d')  # Simplified
        state['time_range'] = (start_date, end_date)
    else:
        state['time_range'] = None
    return state
```



#### e. SQL Generation Node

Uses OpenAI's GPT model to generate the SQL query.

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import PromptTemplate

def generate_sql(state):
    llm = ChatOpenAI(model_name="gpt-3.5-turbo")
    prompt = PromptTemplate.from_template("""
    Given the following database schemas:
    {schemas}

    And the question:
    {question}

    Generate an appropriate SQL query.
    """)
    input_prompt = prompt.format(
        schemas='\n'.join(state['matched_schemas']),
        question=state['mapped_question']
    )
    sql_query = llm.predict(input_prompt)
    state['sql_query'] = sql_query
    return state
```



#### f. Execution Node

Executes the SQL query against the database.

```python
import sqlite3

def execute_query(state):
    conn = sqlite3.connect('your_database.db')
    cursor = conn.cursor()
    cursor.execute(state['sql_query'])
    results = cursor.fetchall()
    state['results'] = results
    conn.close()
    return state
```



### 3. Build the LangGraph

Assemble the nodes into a workflow.

```python
from langgraph.graph import StateGraph, END

graph_builder = StateGraph()

graph_builder.add_node("embed_question", embed_question)
graph_builder.add_node("retrieve_schema", retrieve_schema)
graph_builder.add_node("map_terms", map_terms)
graph_builder.add_node("parse_time", parse_time)
graph_builder.add_node("generate_sql", generate_sql)
graph_builder.add_node("execute_query", execute_query)

graph_builder.set_entry_point("embed_question")
graph_builder.add_edge("embed_question", "retrieve_schema")
graph_builder.add_edge("retrieve_schema", "map_terms")
graph_builder.add_edge("map_terms", "parse_time")
graph_builder.add_edge("parse_time", "generate_sql")
graph_builder.add_edge("generate_sql", "execute_query")
graph_builder.add_edge("execute_query", END)

graph = graph_builder.compile()
```



### 4. Run the Workflow

Initialize the state and run the graph.

```python
initial_state = {
    'question': "Sales statistics for soft drinks in Q3 2024",
    'faiss_index': your_faiss_index,
    'schema_texts': your_schema_texts,
    'term_mappings': {'soft drinks': 'beverages'}
}

final_state = graph.invoke(initial_state)
print(final_state['results'])
```


## References:
[1]: https://medium.com/%40hayagriva99999/building-a-powerful-sql-agent-with-langgraph-a-step-by-step-guide-part-2-24e818d47672?utm_source=chatgpt.com "Building a Powerful SQL Agent with LangGraph: A Step-by ... - Medium"
[2]: https://www.kaggle.com/code/ksmooi/langgraph-sql-query-generation?utm_source=chatgpt.com "LangGraph: SQL Query Generation - Kaggle"
[3]: https://python.langchain.com/v0.2/docs/tutorials/sql_qa/?utm_source=chatgpt.com "Build a Question/Answering system over SQL data | 🦜️ LangChain"
[4]: https://m.youtube.com/watch?v=N9mPo6jFOuc&utm_source=chatgpt.com "Langchain v0.3 Agents P3 - LangGraph SQL Agent w/Tools, Custom ..."
