---
title: ElasticSearch Index Lifecycle Policy (ILM) – Deep Dive
date: 2024-10-24 12:39:24
categories:
- Elasticsearch
tags:
- Elasticsearch
---


- [Introduction to Index Lifecycle Management (ILM)](#introduction-to-index-lifecycle-management-ilm)
- [ElasticSearch ILM Phases](#elasticsearch-ilm-phases)
  - [1. Hot Phase](#1-hot-phase)
  - [2. Warm Phase](#2-warm-phase)
  - [3. Cold Phase](#3-cold-phase)
  - [4. Delete Phase](#4-delete-phase)
- [Creating an Index Lifecycle Policy](#creating-an-index-lifecycle-policy)
- [Real-Life Scenarios of ILM](#real-life-scenarios-of-ilm)
  - [Scenario 1: Logs Retention Policy](#scenario-1-logs-retention-policy)
  - [Scenario 2: Time-Based Data Management](#scenario-2-time-based-data-management)
- [ILM Best Practices](#ilm-best-practices)
- [Useful Commands and Operations](#useful-commands-and-operations)
- [Conclusion](#conclusion)

---

## Introduction to Index Lifecycle Management (ILM)

---

ElasticSearch’s **Index Lifecycle Management (ILM)** allows you to define and automate how indices are managed throughout their lifecycle. This is critical for controlling the size, performance, and cost of your Elasticsearch cluster by ensuring data is stored in appropriate storage tiers as it ages.

An ILM policy divides an index’s lifecycle into phases, during which specific actions are applied to the data. These actions include closing, freezing, moving to different nodes, or deleting indices.

---

<a name="elasticsearch-ilm-phases"></a> 
## ElasticSearch ILM Phases

<a name="1-hot-phase"></a> 
### 1. Hot Phase
- The hot phase is for indexing and querying fresh data.
- Data is stored on high-performance nodes for rapid access.
- Indices in this phase are actively written and read.

**Operations:**
- Force merge segments
- Shrink the index
- Roll over to a new index when the current one is too large

**Command:**
```json
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "30d"
          }
        }
      }
    }
  }
}
```

<a name="2-warm-phase"></a> 
### 2. Warm Phase
- The warm phase is for data that is no longer being actively written but still queried.
- Data is stored on less powerful nodes.
- The index can be optimized by reducing the number of replicas and relocating shards to nodes with less CPU and memory resources.

**Operations:**
- Shrink index to fewer shards
- Reduce the number of replicas
- Move the index to nodes with less powerful resources

**Command:**
```json
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "allocate": {
            "require": {
              "box_type": "warm"
            }
          }
        }
      }
    }
  }
}
```

<a name="3-cold-phase"></a> 
### 3. Cold Phase
- The cold phase is for data that is rarely accessed but needs to be retained.
- Data is stored on the least powerful nodes, with a focus on cost-effectiveness over performance.
- Indices can be marked as read-only.

**Operations:**
- Set the index as read-only
- Move the index to cold storage nodes
- Freeze the index (lowers resource consumption even further)

**Command:**
```json
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "allocate": {
            "require": {
              "box_type": "cold"
            }
          },
          "freeze": {}
        }
      }
    }
  }
}
```

<a name="4-delete-phase"></a> 
### 4. Delete Phase
- The delete phase removes indices once they are no longer needed, preventing unnecessary resource usage.

**Operations:**
- Delete the index after a specific time period or based on certain criteria.

**Command:**
```json
{
  "policy": {
    "phases": {
      "delete": {
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

<a name="creating-an-index-lifecycle-policy"></a> 
## Creating an Index Lifecycle Policy

To create a custom ILM policy, use the following steps:

1. Define the policy by specifying the different phases (hot, warm, cold, delete).
2. Apply the policy to your indices using an alias or direct index management.

**Command Example:**
```bash
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "7d"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

<a name="real-life-scenarios-of-ilm"></a> 
## Real-Life Scenarios of ILM

<a name="scenario-1-logs-retention-policy"></a> 
### Scenario 1: Logs Retention Policy

In a logging system (e.g., centralized logging with ElasticSearch), the hot phase may consist of daily log ingestion, followed by moving the logs to the warm phase after 30 days. Finally, after 90 days, the logs are deleted.

**Policy Example:**
```bash
PUT _ilm/policy/log_retention_policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "warm"
            }
          },
          "shrink": {
            "number_of_shards": 1
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

<a name="scenario-2-time-based-data-management"></a> 
### Scenario 2: Time-Based Data Management

For applications dealing with time-based data (e.g., financial transactions or event logs), ILM helps archive data efficiently. You might retain recent data in the hot phase for instant querying, move older data to warm nodes, and eventually store data in cold nodes before deleting it.

---

<a name="ilm-best-practices"></a> 
## ILM Best Practices

1. **Define Clear Retention Policies:** Tailor lifecycle policies to the nature of your data.
2. **Monitor Resource Usage:** Ensure you have enough storage in warm and cold nodes.
3. **Test Policy Changes:** Roll out new policies incrementally to avoid production outages.
4. **Leverage Aliases:** Use index aliases to manage indices during rollovers without affecting search queries.

---

<a name="useful-commands-and-operations"></a> 
## Useful Commands and Operations

- **Create a policy:**
  ```bash
  PUT _ilm/policy/my_policy
  ```
- **Get details of an existing policy:**
  ```bash
  GET _ilm/policy/my_policy
  ```
- **Delete a policy:**
  ```bash
  DELETE _ilm/policy/my_policy
  ```
- **Apply a policy to an index:**
  ```bash
  PUT /my_index/_settings
  {
    "index.lifecycle.name": "my_policy"
  }
  ```

---

<a name="conclusion"></a> 
## Conclusion

ElasticSearch's Index Lifecycle Management (ILM) is a powerful tool that helps organizations manage their indices' lifecycle by automating transitions between different storage phases (hot, warm, cold, and delete). By optimizing storage and performance, ILM policies improve the overall cost-effectiveness and scalability of your ElasticSearch cluster. Proper use of ILM ensures your data remains accessible and performant while controlling operational costs effectively.