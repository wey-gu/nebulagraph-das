# NebulaGraph Data Intelligence(ngdi) Suite

![image](https://user-images.githubusercontent.com/1651790/221876073-61ef4edb-adcd-4f10-b3fc-8ddc24918ea1.png)

[![pdm-managed](https://img.shields.io/badge/pdm-managed-blueviolet)](https://pdm.fming.dev) [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE) [![PyPI version](https://badge.fury.io/py/ngdi.svg)](https://badge.fury.io/py/ngdi) [![Python](https://img.shields.io/badge/python-3.6%2B-blue.svg)](https://www.python.org/downloads/release/python-360/)

NebulaGraph Data Intelligence Suite for Python (ngdi) is a powerful Python library that offers a range of APIs for data scientists to effectively read, write, analyze, and compute data in NebulaGraph. This library allows data scientists to perform these operations on a single machine using NetworkX, or in a distributed computing environment using Spark, in unified and intuitive API. With ngdi, data scientists can easily access and process data in NebulaGraph, enabling them to perform advanced analytics and gain valuable insights.

```
        ┌───────────────────────────────────────────────────┐            
        │   Spark Cluster                                   │            
        │    .─────.    .─────.    .─────.    .─────.       │            
     ┌─▶│   :       ;  :       ;  :       ;  :       ;      │            
     │  │     `───'      `───'      `───'      `───'        │            
Algorithm                                                   │            
  Spark └───────────────────────────────────────────────────┘            
 Engine ┌────────────────────────────────────────────────────────────────┐
     └──┤                                                                │
        │   NebulaGraph Data Intelligence Suite(ngdi)                    │
        │     ┌────────┐    ┌──────┐    ┌────────┐   ┌─────┐             │
        │     │ Reader │    │ Algo │    │ Writer │   │ GNN │             │
        │     └────────┘    └──────┘    └────────┘   └─────┘             │
        │          ├────────────┴───┬────────┴─────┐    └──────┐         │
        │          ▼                ▼              ▼           ▼         │
        │   ┌─────────────┐ ┌──────────────┐ ┌──────────┐┌───────────┐   │
     ┌──┤   │ SparkEngine │ │ NebulaEngine │ │ NetworkX ││ DGLEngine │   │
     │  │   └─────────────┘ └──────────────┘ └──────────┘└───────────┘   │
     │  └──────────┬─────────────────────────────────────────────────────┘
     │             │        Spark                                        
     │             └────────Reader ────────────┐                         
Spark Reader              Query Mode           │                         
Scan Mode                                      ▼                         
     │  ┌───────────────────────────────────────────────────┐            
     │  │  NebulaGraph Graph Engine         Nebula-GraphD   │            
     │  ├──────────────────────────────┬────────────────────┤            
     │  │  NebulaGraph Storage Engine  │                    │            
     └─▶│  Nebula-StorageD             │    Nebula-Metad    │            
        └──────────────────────────────┴────────────────────┘            
```

## Quick Start in 5 Minutes

- Setup env with Nebula-Up following [this guide](https://github.com/wey-gu/nebulagraph-di/blob/main/docs/Environment_Setup.md).
- Install ngdi with pip from the Jupyter Notebook with http://localhost:8888 (password: `nebula`).
- Open the demo notebook and run cells with `Shift+Enter` or `Ctrl+Enter`.

## Installation

```bash
pip install ngdi
```

### Spark Engine Prerequisites
- Spark 2.4, 3.0(not yet tested)
- [NebulaGraph 3.4+](https://github.com/vesoft-inc/nebula)
- [NebulaGraph Spark Connector 3.4+](https://repo1.maven.org/maven2/com/vesoft/nebula-spark-connector/)
- [NebulaGraph Algorithm 3.1+](https://repo1.maven.org/maven2/com/vesoft/nebula-algorithm/)

### NebulaGraph Engine Prerequisites
- [NebulaGraph 3.4+](https://github.com/vesoft-inc/nebula)
- [NebulaGraph Python Client 3.4+](https://github.com/vesoft-inc/nebula-python)
- [NetworkX](https://networkx.org/)


## Documentation

[API Reference](https://github.com/wey-gu/nebulagraph-di/docs/API.md)

## Usage

### Spark Engine Examples

See also: [examples/spark_engine.ipynb](https://github.com/wey-gu/nebulagraph-di/examples/spark_engine.ipynb)

Run Algorithm on top of NebulaGraph:

```python
from ngdi import NebulaReader

# read data with spark engine, query mode
reader = NebulaReader(engine="spark")
query = """
    MATCH ()-[e:follow]->()
    RETURN e LIMIT 100000
"""
reader.query(query=query, edge="follow", props="degree")
df = reader.read() # this will take some time
df.show(10)

# read data with spark engine, scan mode
reader = NebulaReader(engine="spark")
reader.scan(edge="follow", props="degree")
df = reader.read() # this will take some time
df.show(10)

# read data with spark engine, load mode (not yet implemented)
reader = NebulaReader(engine="spark")
reader.load(source="hdfs://path/to/edge.csv", format="csv", header=True, schema="src: string, dst: string, rank: int")
df = reader.read() # this will take some time
df.show(10)

# run pagerank algorithm
pr_result = df.algo.pagerank(reset_prob=0.15, max_iter=10) # this will take some time

# convert dataframe to NebulaGraphObject
graph = reader.to_graphx() # not yet implemented
```

Write back to NebulaGraph:

```python
from ngdi import NebulaWriter
from ngdi.config import NebulaGraphConfig

config = NebulaGraphConfig()

properties = {
    "louvain": "cluster_id"
}

writer = NebulaWriter(data=df_result, sink="nebulagraph_vertex", config=config, engine="spark")
writer.set_options(
    tag="louvain",
    vid_field="_id",
    properties=properties,
    batch_size=256,
    write_mode="insert",
)
writer.write()
```

Then we could query the result in NebulaGraph:

```cypher
MATCH (v:louvain)
RETURN id(v), v.louvain.cluster_id LIMIT 10;
```

### NebulaGraph Engine Examples(not yet implemented)

Basically the same as Spark Engine, but with `engine="nebula"`.

```diff
- reader = NebulaReader(engine="spark")
+ reader = NebulaReader(engine="nebula")
```
