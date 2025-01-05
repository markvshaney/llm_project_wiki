# Chroma Guide

## Overview
Chroma is an open-source embedding database designed for building AI applications. It allows you to store, index, and query embedding vectors efficiently, making it ideal for semantic search, recommendation systems, and other AI applications.

## Installation

```bash
# Basic installation
pip install chromadb

# With all optional dependencies
pip install chromadb[all]
```

## Basic Usage

### Initializing Chroma
```python
import chromadb
from chromadb.config import Settings

# In-memory client (for testing)
client = chromadb.Client()

# Persistent client
client = chromadb.PersistentClient(path="./chroma_db")

# Client with custom settings
client = chromadb.Client(Settings(
    chroma_db_impl="duckdb+parquet",
    persist_directory="./chroma_db"
))
```

### Collections
Collections are groups of related embeddings and their metadata.

```python
# Create a collection
collection = client.create_collection(
    name="my_collection",
    metadata={"description": "My first collection"}
)

# Get existing collection
collection = client.get_collection(name="my_collection")

# List all collections
collections = client.list_collections()

# Delete collection
client.delete_collection(name="my_collection")
```

### Adding Documents
```python
# Add documents with metadata
collection.add(
    documents=["This is a document", "This is another document"],
    metadatas=[{"source": "doc1"}, {"source": "doc2"}],
    ids=["id1", "id2"]
)

# Add with embeddings
collection.add(
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.7, 8.9]],
    documents=["Doc 1", "Doc 2"],
    metadatas=[{"type": "article"}, {"type": "article"}],
    ids=["doc1", "doc2"]
)
```

## Advanced Usage

### Custom Embedding Functions
```python
from sentence_transformers import SentenceTransformer

class CustomEmbeddingFunction:
    def __init__(self):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
    
    def __call__(self, texts):
        embeddings = self.model.encode(texts)
        return embeddings.tolist()

# Create collection with custom embedding function
collection = client.create_collection(
    name="custom_embeddings",
    embedding_function=CustomEmbeddingFunction()
)
```

### Querying

#### Basic Query
```python
results = collection.query(
    query_texts=["What is machine learning?"],
    n_results=3
)
```

#### Advanced Query Parameters
```python
results = collection.query(
    query_texts=["What is machine learning?"],
    n_results=3,
    where={"type": "article"},  # Filter by metadata
    where_document={"$contains": "machine learning"},  # Filter by content
    include=["metadatas", "distances", "documents"]
)
```

### Updating and Deleting

#### Update Documents
```python
collection.update(
    ids=["id1"],
    documents=["Updated document content"],
    metadatas=[{"updated": True}]
)
```

#### Delete Documents
```python
# Delete by IDs
collection.delete(ids=["id1", "id2"])

# Delete by metadata filter
collection.delete(where={"type": "outdated"})
```

## Integration Patterns

### Integration with LangChain
```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

# Initialize embedding function
embedding_function = OpenAIEmbeddings()

# Create vector store
vectorstore = Chroma(
    collection_name="langchain_store",
    embedding_function=embedding_function,
    persist_directory="./chroma_db"
)

# Add documents
vectorstore.add_texts(
    texts=["Text 1", "Text 2"],
    metadatas=[{"source": "doc1"}, {"source": "doc2"}]
)

# Query
docs = vectorstore.similarity_search(
    query="Sample query",
    k=3
)
```

### Integration with OpenAI
```python
import openai
from chromadb.utils import embedding_functions

# Initialize OpenAI embedding function
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="your-api-key",
    model_name="text-embedding-ada-002"
)

# Create collection with OpenAI embeddings
collection = client.create_collection(
    name="openai_embeddings",
    embedding_function=openai_ef
)
```

## Advanced Features

### Tenant Management
```python
# Create tenants
client.create_tenant("tenant1")
client.create_tenant("tenant2")

# Create collection in tenant
collection = client.create_collection(
    name="tenant_collection",
    tenant="tenant1"
)

# List tenant collections
collections = client.list_collections(tenant="tenant1")
```

### Data Persistence
```python
# Configure persistence
client = chromadb.PersistentClient(
    path="./chroma_db",
    settings=Settings(
        allow_reset=True,
        anonymized_telemetry=False
    )
)

# Backup and restore
import shutil

# Backup
shutil.copytree("./chroma_db", "./backup_db")

# Restore
shutil.rmtree("./chroma_db")
shutil.copytree("./backup_db", "./chroma_db")
```

### Batch Operations
```python
# Batch add
documents = ["Doc " + str(i) for i in range(1000)]
ids = [f"id{i}" for i in range(1000)]
metadatas = [{"index": i} for i in range(1000)]

# Add in batches
batch_size = 100
for i in range(0, len(documents), batch_size):
    collection.add(
        documents=documents[i:i+batch_size],
        ids=ids[i:i+batch_size],
        metadatas=metadatas[i:i+batch_size]
    )
```

## Performance Optimization

### Indexing Strategies
```python
# Create collection with optimized settings
collection = client.create_collection(
    name="optimized_collection",
    metadata={"hnsw:space": "cosine"},  # Distance metric
    embedding_function=embedding_function
)

# Bulk operation for better performance
with collection.batch_writer() as writer:
    for i in range(1000):
        writer.add(
            documents=[f"Document {i}"],
            ids=[f"id{i}"],
            metadatas=[{"index": i}]
        )
```

### Query Optimization
```python
# Optimize query performance
results = collection.query(
    query_texts=["Sample query"],
    n_results=10,
    include=["documents", "distances"],  # Only include needed data
    where={"metadata_field": "value"}    # Use metadata filtering
)
```

## Error Handling and Validation

### Input Validation
```python
def validate_documents(documents, ids, metadatas):
    if len(documents) != len(ids):
        raise ValueError("Number of documents must match number of ids")
    if metadatas and len(metadatas) != len(documents):
        raise ValueError("Number of metadata items must match number of documents")
    return True

# Use validation
try:
    if validate_documents(documents, ids, metadatas):
        collection.add(
            documents=documents,
            ids=ids,
            metadatas=metadatas
        )
except ValueError as e:
    print(f"Validation error: {e}")
except Exception as e:
    print(f"Error adding documents: {e}")
```

### Error Recovery
```python
def safe_query(collection, query_text, retries=3):
    for attempt in range(retries):
        try:
            return collection.query(
                query_texts=[query_text],
                n_results=3
            )
        except Exception as e:
            if attempt == retries - 1:
                raise
            print(f"Query attempt {attempt + 1} failed: {e}")
            time.sleep(1)  # Wait before retry
```

## Monitoring and Logging

### Basic Monitoring
```python
import logging
import time

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class MonitoredCollection:
    def __init__(self, collection):
        self.collection = collection
        self.query_times = []
    
    def query(self, *args, **kwargs):
        start_time = time.time()
        try:
            result = self.collection.query(*args, **kwargs)
            query_time = time.time() - start_time
            self.query_times.append(query_time)
            logger.info(f"Query completed in {query_time:.2f} seconds")
            return result
        except Exception as e:
            logger.error(f"Query failed: {e}")
            raise
```

## Best Practices

1. Collection Management
   - Use meaningful collection names
   - Document collection purposes in metadata
   - Regularly backup important collections

2. Data Organization
   - Structure metadata consistently
   - Use meaningful document IDs
   - Keep related data in the same collection

3. Performance
   - Use batch operations for bulk updates
   - Implement proper indexing
   - Monitor query performance

4. Error Handling
   - Implement proper validation
   - Use retry mechanisms
   - Log errors and exceptions

5. Security
   - Secure API keys
   - Implement access control
   - Regular backups

## Troubleshooting

Common Issues and Solutions:

1. Slow Queries
   - Check index configuration
   - Optimize metadata filters
   - Use batch operations

2. Memory Issues
   - Implement pagination
   - Use persistent storage
   - Monitor memory usage

3. Data Consistency
   - Validate inputs
   - Use transactions
   - Regular data checks

## Resources

- [Official Documentation](https://docs.trychroma.com/)
- [GitHub Repository](https://github.com/chroma-core/chroma)
- [API Reference](https://docs.trychroma.com/reference/)