---
title: Plain Language Explanation of Large Models
date: 2024-11-07 11:58:04
categories:
- LLM
tags:
- LLM
---

- [Introduction](#introduction)
- [Paper Introduction](#paper-introduction)
- [LLM in Brief](#llm-in-brief)
- [Token](#token)
  - [Example](#example)
  - [Attempt](#attempt)
  - [Predicting the Next Token](#predicting-the-next-token)
- [Model Training](#model-training)
- [Context Window](#context-window)
- [Improving the Context Window](#improving-the-context-window)
  - [From Markov Chains to Neural Networks](#from-markov-chains-to-neural-networks)
- [Transformer and Attention Mechanism](#transformer-and-attention-mechanism)
- [Self-Attention (Self-Attention Mechanism)](#self-attention-self-attention-mechanism)
  - [Self-Attention Introduction](#self-attention-introduction)
  - [Workflow](#workflow)
  - [Formula](#formula)
- [Multi-Head Attention](#multi-head-attention)
  - [Summary](#summary)
- [Add \& Norm and Feed Forward](#add--norm-and-feed-forward)
  - [Add \& Norm (Residual Connection and Layer Normalization)](#add--norm-residual-connection-and-layer-normalization)
    - [Residual Connection](#residual-connection)
    - [Layer Normalization](#layer-normalization)
    - [Combined Use](#combined-use)
  - [Feed Forward Neural Network](#feed-forward-neural-network)
    - [Purpose](#purpose)
    - [Structure](#structure)
  - [Summary](#summary-1)
- [Decoder Structure](#decoder-structure)
- [Appendix](#appendix)

---

<a name="introduction"></a>
## Introduction

This document aims to provide a detailed explanation of the current mainstream large model architectures, such as the Transformer architecture. We will cover various aspects from a technical overview, architecture introduction to specific model implementations. Through this document, we hope to provide readers with a comprehensive understanding, helping them grasp the working principles of large models and enhance their technical foundation for communicating with clients. This document is suitable for individuals interested in large models.

<a name="paper-introduction"></a>
## Paper Introduction

**Paper Title:** *Attention is all you need*

**Release Date:** 2017/06/12

**Published By:** Google, University of Toronto

**Brief Summary:** The progenitor of all LLMs, the foundational architecture for a new era in NLP.

**Chinese Summary:** Traditional sequence-to-sequence models use complex recurrent or convolutional neural networks, including encoders and decoders. The best-performing models connect the encoder and decoder through attention mechanisms.

The author's team proposes a new simple network structure, Transformer, which is entirely based on attention mechanisms and no longer uses recurrence and convolution.

Experiments on two machine translation tasks show that these models perform superiorly in quality and are easier to parallelize, significantly reducing the training time required.

The model achieved 28.4 BLEU on the WMT 2014 English-German translation task, over 2 BLEU higher than the best existing results (including ensemble models). On the WMT 2014 English-French translation task, the model achieved a new single-model best BLEU score of 41.8 after training on eight GPUs for 3.5 days, with training costs only a fraction of the best models in the literature.

The Transformer demonstrates generalization capabilities across other tasks, whether with extensive or limited training data, successfully applied to English constituency parsing.

**Paper Link:** [Attention Is All You Need](pdf/Plain-Language-Explaination-of-LLMs/Attention_Is_All_You_Need.pdf)

**Core Technology:** Model Architecture (here, leave a general impression of encode + decode)

![Model Architecture](images/Plain-Language-Explaination-of-LLMs/Image_01.jpg)

<a name="llm-in-brief"></a>
## LLM in Brief

Many believe that large models can directly answer questions or engage in conversations. However, their core function is to predict the next likely word, or "Token," based on the input text. This predictive ability makes LLMs excel in various applications, including but not limited to:

- **Text Generation:** LLMs can generate coherent and meaningful text passages for writing assistance, content creation, etc.
- **Question Answering Systems:** By understanding the context of a question, LLMs can generate accurate answers, widely used in intelligent customer service and information retrieval.
- **Translation:** LLMs can perform high-quality language translation based on context, supporting multilingual communication.
- **Text Summarization:** LLMs can extract key content from long documents to generate concise summaries, facilitating quick understanding.
- **Dialogue Systems:** LLMs can simulate human conversations, providing natural and smooth interaction experiences, applied in chatbots and virtual assistants.

By understanding the concept of Tokens, we can better grasp the working principles of LLMs and their powerful capabilities in practical applications.

<a name="token"></a>
## Token

<a name="token-example"></a>
### Example

When discussing Tokens, it's impossible not to mention a recent low-level error in a large model: "How many 'r's are in 'Strawberry'?"

![Strawberry](images/Plain-Language-Explaination-of-LLMs/Image_02.png)

After the ridicule, people calmed down and began to think: what is the essence behind this low-level error?

It is generally believed that it is the fault of Tokenization.

In China, Tokenization is often translated as "word segmentation." This translation is somewhat misleading because the Tokens in Tokenization do not necessarily refer to words; they can also be punctuation marks, numbers, or parts of a word. For example, in a tool provided by OpenAI, the word "Strawberry" is divided into three Tokens: Str-aw-berry. In this case, asking the AI large model to count how many 'r's are in the word is indeed a challenge for it.

To let everyone intuitively see the world of text in the eyes of large models, Karpathy specially wrote a small program to represent Tokens with emojis.

![Tokens With Emojis](images/Plain-Language-Explaination-of-LLMs/Image_03.png)

<a name="token-attempt"></a>
### Attempt

A Token is the basic unit of text that LLMs understand. Although it is convenient to consider a Token as a word, for LLMs, the goal is to encode text as efficiently as possible. Therefore, in many cases, a Token represents a character sequence shorter or longer than an entire word. Punctuation marks and spaces are also represented as Tokens, either individually or combined with other characters. Each Token in the LLM vocabulary has a unique identifier, usually a number. LLMs use tokenizers to convert regular text strings into equivalent lists of Token numbers.

```python
import tiktoken

# Get the encoder for the GPT-2 model
encoding = tiktoken.encoding_for_model("gpt-2")

# Encode example sentence
encoded_text = encoding.encode("A journey of a thousand miles begins with a single step.")
print(encoded_text)

# Decode back to original text
decoded_text = encoding.decode(encoded_text)
print(decoded_text)

# Encode and decode individual tokens
print(encoding.decode([32]))  # 'A'
print(encoding.decode([7002])) # ' journey'
print(encoding.decode([286]))   # 'of'

# Encode the word "thousand"
thousand_encoded = encoding.encode("thousand")
print(thousand_encoded)

# Decode the encoded result of "thousand"
print(encoding.decode([400])) # 'th'
print(encoding.decode([29910]))   # 'ousand'

[32, 7002, 286, 257, 7319, 4608, 6140, 351, 257, 2060, 2239, 13]
A journey of a thousand miles begins with a single step.
A
 journey
 of
[400, 29910]
th
ousand
```

<a name="predicting-the-next-token"></a>
### Predicting the Next Token

As mentioned above, given a piece of text, the task of a language model is to predict the next Token. If expressed in Python pseudocode, it looks like this:

```python
predictions = predict_next_token(['A', 'journey', 'of', 'a'])
```

Here, the `predict_next_token` function receives a series of input Tokens converted from a user-provided prompt. In this example, we assume each word constitutes a separate Token. In reality, each Token is encoded as a number rather than being directly passed into the model as text. The output of the function is a data structure containing the probability values for each possible Token in the vocabulary to appear after the current input sequence.

The language model needs to learn to make such predictions through a training process. During training, the model is exposed to a large amount of text data, learning language patterns and rules. After training, the model can use the learned knowledge to estimate the probability of the next Token for any given Token sequence.

To generate continuous text, the model needs to repeatedly call itself, generating a new Token each time and adding it to the existing sequence until the preset length is reached. Below is a more detailed Python pseudocode showing this process:

```python
def generate_text(prompt, num_tokens, hyperparameters):
    tokens = tokenize(prompt)  # Convert prompt to a list of Tokens
    for _ in range(num_tokens):  # Repeat for the specified number of Tokens
        predictions = predict_next_token(tokens)  # Get predictions for the next Token
        next_token = select_next_token(predictions, hyperparameters)  # Select the next Token based on probabilities
        tokens.append(next_token)  # Add the selected Token to the list
    return ''.join(tokens)  # Merge the Token list into a string

# Helper function to select the next Token
def select_next_token(predictions, hyperparameters):
    # Adjust Token selection using strategies like temperature, top-k, or top-p
    # Implementation details depend on the specific hyperparameter settings
    pass
```

In this code, the `generate_text` function accepts a prompt string, the number of Tokens to generate, and a set of hyperparameters as input. The `tokenize` function is responsible for converting the prompt into a list of Tokens, while the `select_next_token` function selects the next Token based on the predicted probability distribution. By adjusting the hyperparameters in `select_next_token`, such as temperature, top-k, and top-p, the diversity and creativity of the generated text can be controlled. As the loop iterates, new Tokens are continuously added to the sequence, eventually forming coherent text output.

<a name="model-training"></a>
## Model Training

Imagine we are training a model to predict the next word in a sentence. This is like playing a word-guessing game where the model needs to guess the next word based on the words already present. For a simplified vocabulary, let's assume there are only five words:

['I', 'you', 'love', 'oranges', 'grapes']

We do not consider spaces and punctuation, focusing only on these words.

We have three sentences as training data:

- I love oranges
- I love grapes
- you love oranges

We can imagine a 5x5 table where each cell represents the number of times one word follows another. This table might look like this:

![Cell Matrix](images/Plain-Language-Explaination-of-LLMs/Image_04.png)

In this example, "I" is followed by "love" twice, and "love" is followed by "oranges" once and "grapes" twice. To make this table useful, we need to convert these counts into probabilities so that the model can predict the likelihood of the next word. For example, the probability of "oranges" following "love" is 1/3 (about 33.3%), and the probability of "grapes" following "love" is 2/3 (about 66.7%).

However, we encounter a problem: "oranges" and "grapes" do not appear after other words. This means that without additional information, the model will not be able to predict what might follow these words. To solve this problem, we can assume that any word in the vocabulary might follow any other word, although this is not perfect, it ensures that the model can make predictions even with limited training data.

In the real world, large language models (LLMs) use vast amounts of training data, reducing the chances of such "gaps" occurring. However, due to the infrequent occurrence of certain word combinations in the training data, LLMs may perform poorly in some cases, resulting in generated text that is grammatically correct but logically flawed or inconsistent. This phenomenon is sometimes referred to as "model hallucination."

<a name="context-window"></a>
## Context Window

In previous discussions, we mentioned using Markov chains to train small models, which rely on only the last Token to predict the next Token. This means that any text before the last Token does not affect the prediction, so the context window is very small, only one Token. Due to the small context window, the model easily "forgets" previous information, leading to generated text that lacks consistency, jumping from one word to another.

<a name="improving-the-context-window"></a>
## Improving the Context Window

To improve the model's prediction quality, we can try to increase the size of the context window. For example, if we use a two-Token context window, we need to build a larger probability matrix where each row represents all possible two-Token sequences. For a five-Token vocabulary, this will add 25 rows (5^2). Each time the `predict_next_token()` function is called, the last two Tokens of the input will be used to find the corresponding row in the probability table.

However, even with a two-Token context window, the generated text may still lack coherence. To generate more consistent and meaningful text, the context window size needs to be further increased. For example, increasing the context window to three Tokens will increase the number of rows in the probability table to 125 (5^3), but this is still insufficient to generate high-quality text.

As the context window increases, the size of the probability table grows exponentially. Taking the GPT-2 model as an example, it uses a 1024-Token context window. If we were to implement such a large context window using Markov chains as in the previous example, each row of the probability table would need to represent a sequence of length between 1 and 1024 Tokens. For a vocabulary of 5 Tokens, the number of possible sequences is 5^1024, an astronomical number. This number is so large that it is impractical to store and process such a massive probability table. Therefore, Markov chains face serious scalability issues when dealing with large context windows.

### From Markov Chains to Neural Networks

Clearly, the probability table method is infeasible when dealing with large context windows. We need a more efficient method to predict the next Token. This is where neural networks come into play. Neural networks are special functions that accept input data, perform a series of calculations, and then output predictions. For language models, the input is a series of Tokens, and the output is the probability distribution of the next Token.

The key to neural networks lies in their parameters. These parameters are gradually adjusted during training to optimize the model's prediction performance. The training process involves extensive mathematical operations, including forward propagation and backpropagation. Forward propagation refers to the input data passing through the network's layers to generate predictions; backpropagation adjusts the network's parameters based on the difference between the predictions and the true labels to reduce errors.

Modern language models, such as GPT-2, GPT-3, and GPT-4, use very deep neural networks with hundreds of millions or even trillions of parameters. The training process for these models is very complex and usually takes weeks or even months. Despite this, well-trained LLMs maintain high coherence and consistency in text generation, thanks to their powerful context understanding and generation capabilities.

<a name="transformer-and-attention-mechanism"></a>
## Transformer and Attention Mechanism

The Transformer is one of the most popular neural network architectures, particularly suited for natural language processing tasks. The core feature of the Transformer model is its attention mechanism. The attention mechanism allows the model to dynamically focus on different parts of the input sequence when processing it, thereby better capturing context information.

The attention mechanism was initially applied to machine translation tasks to help the model identify key information in the input sequence. Through the attention mechanism, the model can "focus" on important Tokens in the input sequence, thereby generating more accurate translation results. In language generation tasks, the attention mechanism plays an equally important role, enabling the model to consider multiple Tokens in the context window when generating the next Token, thereby generating more coherent and meaningful text.

In summary, although Markov chains provide a simple method for text generation, they have obvious limitations when dealing with large context windows. Modern language models overcome these limitations by using neural networks and attention mechanisms, achieving efficient and high-quality text generation.

![Framework-05](images/Plain-Language-Explaination-of-LLMs/Image_05.jpg)

Returning to the framework diagram, the input representation **x** of a word in the Transformer is obtained by adding the word **Embedding** and the position **Embedding** (Positional Encoding).

There are many ways to obtain the word Embedding, such as using pre-trained algorithms like Word2Vec, Glove, or training it in the Transformer.

In addition to the word Embedding, the Transformer also uses position Embedding to represent the position of a word in the sentence. **Because the Transformer does not use an RNN structure but uses global information, it cannot utilize the order information of words, which is very important for NLP.** Therefore, the Transformer uses position Embedding to preserve the relative or absolute position of words in the sequence.

In short: taking "apple" as an example

In "There are apples and bananas in the fruit store," "apple" refers to the fruit, while in "The store has just launched the latest Apple 16," "apple" represents the brand.

<a name="self-attention"></a>
## Self-Attention (Self-Attention Mechanism)

![Self Attention](images/Plain-Language-Explaination-of-LLMs/Image_06.jpg)

The above image is the internal structure diagram of the Transformer in the paper, with the left side being the Encoder block and the right side being the Decoder block. The part circled in red is **Multi-Head Attention**, which consists of multiple **Self-Attention** layers. It can be seen that the Encoder block contains one Multi-Head Attention, while the Decoder block contains two Multi-Head Attention (one of which uses Masked). Above the Multi-Head Attention is an Add & Norm layer, where Add represents a residual connection (Residual Connection) to prevent network degradation, and Norm represents Layer Normalization to normalize the activation values of each layer.

Since **Self-Attention** is the key point of the Transformer, we focus on Multi-Head Attention and Self-Attention. First, let's take a detailed look at the internal logic of Self-Attention.

Of course, we can understand the Self-Attention mechanism in a more concise way.

<a name="self-attention-introduction"></a>
### Self-Attention Introduction

**Self-Attention** is a method that allows the model to focus on different parts of the sequence data when processing it, especially suitable for handling long texts. It achieves this by calculating the relevance between each element in the sequence and all other elements.

<a name="self-attention-workflow"></a>
### Workflow

1. **Transformation:**

   - Each input element (such as a word) is transformed into three vectors: **Query (query), Key (key), and Value (value)**. These vectors are obtained by multiplying the input vector by three different weight matrices WQ, WK, and WV.

2. **Calculate Attention Scores:**

   - For each element, use its **Query** vector to perform dot product operations with the **Key** vectors of all other elements, obtaining a list of scores. This score represents the relevance of the current element to all other elements.

3. **Normalization:**

   - Normalize these scores through the **softmax** function to obtain a probability distribution, representing the attention weights of the current element to all other elements.

4. **Weighted Sum:**

   - Use these attention weights to perform a weighted sum of the **Value** vectors of all elements, obtaining the final output vector.

<a name="self-attention-formula"></a>
### Formula

Suppose the input sequence is X = [x1, x2, ..., xn], where each xi is a vector.

1. Transformation:

![Transformation](images/Plain-Language-Explaination-of-LLMs/Image_07.png)

2. Calculate Attention Scores:

![Calculate Attention](images/Plain-Language-Explaination-of-LLMs/Image_08.png)

   where dk is the dimension of the Key vector.

3. Normalization: Attention Weights = softmax(Scores)

4. Weighted Sum: Output = Attention Weights · V

<a name="multi-head-attention"></a>
## Multi-Head Attention

- **Multi-Head Attention** is designed to allow the model to capture information from multiple different perspectives. Specifically, multiple Self-Attention layers (each called a "head") are run in parallel, and then all heads' outputs are concatenated and passed through a linear transformation.

<a name="multi-head-attention-summary"></a>
### Summary

- **Self-Attention** allows the model to focus on different parts of the sequence, thereby better capturing long-range dependencies.
- **Multi-Head Attention** enhances the model's expressive power by using multiple Self-Attention layers, allowing it to consider information from multiple perspectives.

<a name="add-norm-and-feed-forward"></a>
## Add & Norm and Feed Forward

<a name="add-norm-residual-connection"></a>
### Add & Norm (Residual Connection and Layer Normalization)

#### Residual Connection

- **Purpose:** The residual connection helps in training deep networks by allowing the gradient to flow directly through the network, mitigating the vanishing gradient problem.
- **Operation:** The residual connection adds the input of a layer directly to its output:
  \[
  \text{Output} = F(x) + x
  \]
  where \( F(x) \) is the output of the layer.

#### Layer Normalization

- **Purpose:** Layer normalization stabilizes the training process by normalizing the activations of each layer, ensuring that the mean and variance of the activations remain consistent.
- **Operation:** For each sample in the batch, layer normalization normalizes the activations across the feature dimension:
  \[
  \text{LayerNorm}(x) = \frac{x - \mu}{\sigma} \cdot \gamma + \beta
  \]
  where \( \mu \) and \( \sigma \) are the mean and standard deviation of the activations, and \( \gamma \) and \( \beta \) are learnable parameters.

#### Combined Use

- **Steps:**

   1. First, calculate the output \( F(x) \) of a layer.
   2. Add the input \( x \) to \( F(x) \), obtaining \( y = F(x) + x \).
   3. Perform layer normalization on \( y \) to obtain the final output.

<a name="feed-forward-neural-network"></a>
### Feed Forward Neural Network

#### Purpose

- **Add Nonlinearity:** Makes the model more flexible, capable of handling more complex data.

#### Structure

- **Two-Layer Fully Connected Network:**

   1. The first layer: Transform the input through a linear transformation (multiply by a matrix), then process it with the ReLU activation function.
   2. The second layer: Transform it again through a linear transformation (multiply by another matrix).

<a name="feed-forward-summary"></a>
### Summary

- **Add & Norm:** Through residual connections and layer normalization, the model becomes more stable and trains faster.
- **Feed Forward:** Through a two-layer fully connected network, it adds flexibility to the model, enabling it to handle more complex data.

<a name="decoder-structure"></a>
## Decoder Structure

![Decoder Structure](images/Plain-Language-Explaination-of-LLMs/Image_09.png)

The red part in the above image is the structure of the Transformer's Decoder block, similar to the Encoder block but with some differences:

- Contains two Multi-Head Attention layers.
- The first Multi-Head Attention layer uses a Masked operation.
- The second Multi-Head Attention layer's **K, V** matrices use the Encoder's **encoded information matrix C** for calculation, while **Q** uses the output of the previous Decoder block for calculation.
- Finally, a Softmax layer calculates the probability of the next translated word.

<a name="appendix"></a>
## Appendix

```python
# Import necessary libraries
import torch
import torch.nn as nn
import torch.optim as optim
import math

# Define the positional encoding layer, used to add positional information to the input sequence
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super(PositionalEncoding, self).__init__()
        # Initialize the positional encoding tensor
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        # Calculate the sine and cosine values
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0).transpose(0, 1)
        # Register the positional encoding as a buffer
        self.register_buffer('pe', pe)

    def forward(self, x):
        # Add the positional encoding to the input
        x = x + self.pe[:x.size(0), :]
        return x

# Define the Transformer-based model class
class TransformerModel(nn.Module):
    def __init__(self, input_dim, output_dim, d_model=512, nhead=8, num_encoder_layers=6, dim_feedforward=2048, dropout=0.1):
        super(TransformerModel, self).__init__()
        self.model_type = 'Transformer'
        # Define the embedding layer
        self.embedding = nn.Embedding(input_dim, d_model)
        # Define the positional encoding layer
        self.pos_encoder = PositionalEncoding(d_model)
        # Define the encoder layers
        encoder_layers = nn.TransformerEncoderLayer(d_model, nhead, dim_feedforward, dropout)
        self.transformer_encoder = nn.TransformerEncoder(encoder_layers, num_encoder_layers)
        self.d_model = d_model
        # Define the decoder layer
        self.decoder = nn.Linear(d_model, output_dim)
        # Initialize the weights
        self.init_weights()

    def init_weights(self):
        initrange = 0.1
        self.embedding.weight.data.uniform_(-initrange, initrange)
        self.decoder.bias.data.zero_()
        self.decoder.weight.data.uniform_(-initrange, initrange)

    def forward(self, src, src_mask):
        # Embed the input and multiply by the square root of d_model
        src = self.embedding(src) * math.sqrt(self.d_model)
        src = self.pos_encoder(src)
        # Encode the input
        output = self.transformer_encoder(src, src_mask)
        # Decode the output
        output = self.decoder(output)
        return output

# Generate a triangular matrix as a mask
def generate_square_subsequent_mask(sz):
    mask = (torch.triu(torch.ones(sz, sz)) == 1).transpose(0, 1)
    mask = mask.float().masked_fill(mask == 0, float('-inf')).masked_fill(mask == 1, float(0.0))
    return mask

# Example usage
input_dim = 1000  # Vocabulary size
output_dim = 1000  # Output dimension
seq_length = 10  # Sequence length

# Create a model instance
model = TransformerModel(input_dim=input_dim, output_dim=output_dim)

# Example data
src = torch.randint(0, input_dim, (seq_length, 32))  # (sequence length, batch size)
src_mask = generate_square_subsequent_mask(seq_length)

# Forward propagation
output = model(src, src_mask)
print(output.shape)  # Expected output: [sequence length, batch size, output dimension]

# Define a simple loss function and optimizer for training
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Example training loop
for epoch in range(10):  # Number of iterations
    optimizer.zero_grad()
    output = model(src, src_mask)
    loss = criterion(output.view(-1, output_dim), src.view(-1))
    loss.backward()
    optimizer.step()
print(f"Epoch {epoch+1}, Loss: {loss.item()}")
```
