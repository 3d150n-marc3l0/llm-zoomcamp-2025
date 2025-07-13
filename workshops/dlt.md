# From REST to reasoning: ingest, index, and query with dlt and Cognee

* Video: https://www.youtube.com/watch?v=MNt_KK32gys
* Homework solution: TBA

# Resources

* [Slides](https://docs.google.com/presentation/d/1oHQilxEVqGGW4S2ctNEE0wHY2LgcjYLaRUziAoinsis/edit?usp=sharing)
* [Colab Notebook](https://colab.research.google.com/drive/1vBA9OIGChcKjjg8r5hHduR0v3A5D6rmH?usp=sharing) 

--- 

# Homework

## Question 1. dlt Version

In this homework, we will load the data from our FAQ to Qdrant

Let's install dlt with Qdrant support and Qdrant client:

```bash
pip install -q "dlt[qdrant]" "qdrant-client[fastembed]"
```

What's the version of dlt that you installed?


```bash
import dlt
print(dlt.__version)

'1.13.0'
```

## dlt Resourse

For reading the FAQ data, we have this helper function:

```python
def zoomcamp_data():
    docs_url = 'https://github.com/alexeygrigorev/llm-rag-workshop/raw/main/notebooks/documents.json'
    docs_response = requests.get(docs_url)
    documents_raw = docs_response.json()

    for course in documents_raw:
        course_name = course['course']

        for doc in course['documents']:
            doc['course'] = course_name
            yield doc
```

Annotate it with `@dlt.resource`. We will use it when creating
a dlt pipeline.

## Question 2. dlt pipeline

Now let's create a pipeline. 

We need to define a destination for that. Let's use the `qdrant` one:

```python
from dlt.destinations import qdrant

qdrant_destination = qdrant(
  qd_path="db.qdrant", 
)
```

In this case, we tell dlt (and Qdrant) to create a folder with
our data, and the name for it will be `db.qdrant`

Let's run it:

```python
pipeline = dlt.pipeline(
    pipeline_name="zoomcamp_pipeline",
    destination=qdrant_destination,
    dataset_name="zoomcamp_tagged_data"

)
load_info = pipeline.run(zoomcamp_data())
print(pipeline.last_trace)
```

How many rows were inserted into the `zoomcamp_data` collection?

Look for `"Normalized data for the following tables:"` in the trace output.


```plaintext
Run started at 2025-07-13 18:21:43.986160+00:00 and COMPLETED in 11.58 seconds with 4 steps.
Step extract COMPLETED in 0.83 seconds.

Load package 1752430904.9536664 is EXTRACTED and NOT YET LOADED to the destination and contains no failed jobs

Step normalize COMPLETED in 0.10 seconds.
Normalized data for the following tables:
- _dlt_pipeline_state: 1 row(s)
- zoomcamp_data: 948 row(s)

Load package 1752430904.9536664 is NORMALIZED and NOT YET LOADED to the destination and contains no failed jobs

Step load COMPLETED in 9.68 seconds.
Pipeline zoomcamp_pipeline load step completed in 9.65 seconds
1 load package(s) were loaded to destination qdrant and into dataset zoomcamp_tagged_data
The qdrant destination used /home/emmuzoo/llm-zoomcamp-2025/workshops/dlt/db.qdrant location to store data
Load package 1752430904.9536664 is LOADED and contains no failed jobs

Step run COMPLETED in 11.58 seconds.
Pipeline zoomcamp_pipeline load step completed in 9.65 seconds
1 load package(s) were loaded to destination qdrant and into dataset zoomcamp_tagged_data
The qdrant destination used /home/emmuzoo/llm-zoomcamp-2025/workshops/dlt/db.qdrant location to store data
Load package 1752430904.9536664 is LOADED and contains no failed jobs
```


## Question 3. Embeddings

When inserting the data, an embedding model was used. Which one?

You can find this out by inspecting the `meta.json` file created
in the target folder. During the data insertion process, a folder named db.qdrant will be created, and the meta.json file will be located inside this folder.


```batch
!cat db.qdrant/meta.json | jq
```

```json
{
  "collections": {
    "zoomcamp_tagged_data": {
      "vectors": {
        "fast-bge-small-en": {
          "size": 384,
          "distance": "Cosine",
          "hnsw_config": null,
          "quantization_config": null,
          "on_disk": null,
          "datatype": null,
          "multivector_config": null
        }
      },
      "shard_number": null,
      "sharding_method": null,
      "replication_factor": null,
      "write_consistency_factor": null,
      "on_disk_payload": null,
      "hnsw_config": null,
      "wal_config": null,
      "optimizers_config": null,
      "init_from": null,
      "quantization_config": null,
      "sparse_vectors": null,
      "strict_mode_config": null
    },
    "zoomcamp_tagged_data__dlt_version": {
      "vectors": {
        "fast-bge-small-en": {
          "size": 384,
          "distance": "Cosine",
          "hnsw_config": null,
          "quantization_config": null,
          "on_disk": null,
          "datatype": null,
          "multivector_config": null
        }
      },
      "shard_number": null,
      "sharding_method": null,
      "replication_factor": null,
      "write_consistency_factor": null,
      "on_disk_payload": null,
      "hnsw_config": null,
      "wal_config": null,
      "optimizers_config": null,
      "init_from": null,
      "quantization_config": null,
      "sparse_vectors": null,
      "strict_mode_config": null
    },
    "zoomcamp_tagged_data__dlt_loads": {
      "vectors": {
        "fast-bge-small-en": {
          "size": 384,
          "distance": "Cosine",
          "hnsw_config": null,
          "quantization_config": null,
          "on_disk": null,
          "datatype": null,
          "multivector_config": null
        }
      },
      "shard_number": null,
      "sharding_method": null,
      "replication_factor": null,
      "write_consistency_factor": null,
      "on_disk_payload": null,
      "hnsw_config": null,
      "wal_config": null,
      "optimizers_config": null,
      "init_from": null,
      "quantization_config": null,
      "sparse_vectors": null,
      "strict_mode_config": null
    },
    "zoomcamp_tagged_data__dlt_pipeline_state": {
      "vectors": {
        "fast-bge-small-en": {
          "size": 384,
          "distance": "Cosine",
          "hnsw_config": null,
          "quantization_config": null,
          "on_disk": null,
          "datatype": null,
          "multivector_config": null
        }
      },
      "shard_number": null,
      "sharding_method": null,
      "replication_factor": null,
      "write_consistency_factor": null,
      "on_disk_payload": null,
      "hnsw_config": null,
      "wal_config": null,
      "optimizers_config": null,
      "init_from": null,
      "quantization_config": null,
      "sparse_vectors": null,
      "strict_mode_config": null
    },
    "zoomcamp_tagged_data_zoomcamp_data": {
      "vectors": {
        "fast-bge-small-en": {
          "size": 384,
          "distance": "Cosine",
          "hnsw_config": null,
          "quantization_config": null,
          "on_disk": null,
          "datatype": null,
          "multivector_config": null
        }
      },
      "shard_number": null,
      "sharding_method": null,
      "replication_factor": null,
      "write_consistency_factor": null,
      "on_disk_payload": null,
      "hnsw_config": null,
      "wal_config": null,
      "optimizers_config": null,
      "init_from": null,
      "quantization_config": null,
      "sparse_vectors": null,
      "strict_mode_config": null
    }
  },
  "aliases": {}
}
```

## Submit the results

* Submit your results here: https://courses.datatalks.club/llm-zoomcamp-2025/homework/dlt
