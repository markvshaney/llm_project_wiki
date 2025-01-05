# LangChain Guide

## Overview
LangChain is a framework for developing applications powered by language models. It provides a standard interface for integrating:
- Large Language Models (LLMs)
- Prompt management
- Memory systems
- Document processing
- Chain and agent orchestration

## Core Components

### 1. Models
LangChain supports various language models and chat models:

```python
from langchain_openai import OpenAI, ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_community.llms import HuggingFaceHub

# OpenAI
llm = OpenAI(model_name="gpt-3.5-turbo-instruct")
chat_model = ChatOpenAI(model_name="gpt-4")

# Anthropic
claude = ChatAnthropic(model="claude-3-opus-20240229")

# HuggingFace
hugging_llm = HuggingFaceHub(repo_id="google/flan-t5-xxl")
```

### 2. Prompts
LangChain offers structured prompt management:

```python
from langchain.prompts import PromptTemplate, ChatPromptTemplate
from langchain.prompts.chat import SystemMessagePromptTemplate, HumanMessagePromptTemplate

# Basic prompt template
prompt = PromptTemplate(
    input_variables=["topic"],
    template="Write a summary about {topic}."
)

# Chat prompt template
chat_prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template(
        "You are a helpful assistant specialized in {domain}."
    ),
    HumanMessagePromptTemplate.from_template(
        "{question}"
    )
])
```

### 3. Memory
LangChain provides various memory systems for maintaining conversation context:

```python
from langchain.memory import ConversationBufferMemory, ConversationBufferWindowMemory

# Simple buffer memory
memory = ConversationBufferMemory()

# Window memory (keeps last n interactions)
window_memory = ConversationBufferWindowMemory(k=3)

# Chat memory with summaries
summary_memory = ConversationSummaryMemory(llm=chat_model)
```

### 4. Chains
Chains combine multiple components into a single workflow:

```python
from langchain.chains import LLMChain, SimpleSequentialChain

# Basic LLM chain
chain = LLMChain(
    llm=chat_model,
    prompt=prompt,
    verbose=True
)

# Sequential chain
first_chain = LLMChain(llm=llm, prompt=first_prompt)
second_chain = LLMChain(llm=llm, prompt=second_prompt)
sequence = SimpleSequentialChain(chains=[first_chain, second_chain])
```

### 5. Agents
Agents can use tools to accomplish tasks:

```python
from langchain.agents import initialize_agent, Tool
from langchain.tools import DuckDuckGoSearchRun

# Define tools
search = DuckDuckGoSearchRun()
tools = [
    Tool(
        name="Search",
        func=search.run,
        description="useful for searching the internet"
    )
]

# Create agent
agent = initialize_agent(
    tools=tools,
    llm=chat_model,
    agent="zero-shot-react-description",
    verbose=True
)
```

## Advanced Features

### Document Processing
```python
from langchain.document_loaders import PyPDFLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

# Load and split documents
loader = PyPDFLoader("document.pdf")
documents = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(documents)

# Create vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embeddings
)
```

### Retrieval Augmented Generation (RAG)
```python
from langchain.chains import RetrievalQA

# Create RAG chain
qa_chain = RetrievalQA.from_chain_type(
    llm=chat_model,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(),
    verbose=True
)

# Query documents
response = qa_chain.run("What are the key points in the document?")
```

### Custom Tools
```python
from langchain.tools import StructuredTool
from typing import Optional

def calculate_metrics(numbers: str) -> str:
    """Calculate basic statistics for a list of numbers."""
    nums = [float(n) for n in numbers.split(',')]
    return f"Mean: {sum(nums)/len(nums)}, Min: {min(nums)}, Max: {max(nums)}"

calculator_tool = StructuredTool.from_function(
    func=calculate_metrics,
    name="Calculator",
    description="Calculate statistics for a comma-separated list of numbers"
)
```

### Custom Chains
```python
from langchain.chains import TransformChain, SequentialChain

def transform_func(inputs: dict) -> dict:
    text = inputs["text"]
    word_count = len(text.split())
    return {"word_count": word_count}

transform_chain = TransformChain(
    input_variables=["text"],
    output_variables=["word_count"],
    transform=transform_func
)
```

## Integration Examples

### OpenAI Functions
```python
from langchain.agents import OpenAIFunctions

# Define function schema
function_schema = {
    "name": "calculate_metrics",
    "description": "Calculate statistics for numbers",
    "parameters": {
        "type": "object",
        "properties": {
            "numbers": {
                "type": "string",
                "description": "Comma-separated numbers"
            }
        },
        "required": ["numbers"]
    }
}

# Create function agent
function_agent = OpenAIFunctions.from_llm_and_tools(
    llm=chat_model,
    tools=[calculator_tool],
    verbose=True
)
```

### Database Integration
```python
from langchain.utilities import SQLDatabase
from langchain_community.tools import SQLDatabaseTool

# Connect to database
db = SQLDatabase.from_uri("sqlite:///data.db")

# Create SQL tool
sql_tool = SQLDatabaseTool(db=db)

# Create SQL agent
sql_agent = create_sql_agent(
    llm=chat_model,
    toolkit=SQLDatabaseToolkit(db=db),
    verbose=True
)
```

## Best Practices

### 1. Prompt Engineering
```python
# Use structured prompts with clear instructions
prompt = PromptTemplate(
    template="""
    Task: {task}
    
    Context: {context}
    
    Requirements:
    1. {requirement1}
    2. {requirement2}
    
    Response format:
    {format}
    
    Response:
    """,
    input_variables=["task", "context", "requirement1", "requirement2", "format"]
)
```

### 2. Error Handling
```python
from langchain.callbacks import ErrorHandler

class CustomErrorHandler(ErrorHandler):
    def on_llm_error(self, error, **kwargs):
        print(f"LLM Error: {error}")
        # Implement retry or fallback logic
    
    def on_chain_error(self, error, **kwargs):
        print(f"Chain Error: {error}")
        # Handle chain-specific errors

# Use error handler
chain = LLMChain(
    llm=chat_model,
    prompt=prompt,
    callbacks=[CustomErrorHandler()]
)
```

### 3. Memory Management
```python
# Implement custom memory
class CustomMemory(BaseMemory):
    def __init__(self):
        self.chat_history = []
    
    def load_memory_variables(self, inputs):
        return {"chat_history": self.chat_history}
    
    def save_context(self, inputs, outputs):
        self.chat_history.append({
            "input": inputs,
            "output": outputs
        })
```

## Performance Optimization

### 1. Caching
```python
from langchain.cache import InMemoryCache
import langchain

# Set up caching
langchain.llm_cache = InMemoryCache()

# Or use Redis cache
from langchain.cache import RedisCache
import redis

redis_client = redis.Redis()
langchain.llm_cache = RedisCache(redis_client)
```

### 2. Batch Processing
```python
# Process multiple inputs efficiently
async def process_batch(inputs, chain):
    tasks = [chain.arun(input_text) for input_text in inputs]
    results = await asyncio.gather(*tasks)
    return results
```

## Debugging and Testing

### 1. Verbose Output
```python
# Enable verbose mode for debugging
chain = LLMChain(
    llm=chat_model,
    prompt=prompt,
    verbose=True
)

# Use callbacks for detailed logging
from langchain.callbacks import StdOutCallbackHandler

handler = StdOutCallbackHandler()
chain.run(input="test", callbacks=[handler])
```

### 2. Unit Testing
```python
from unittest.mock import Mock
import pytest

def test_chain():
    mock_llm = Mock()
    mock_llm.predict.return_value = "test response"
    
    chain = LLMChain(
        llm=mock_llm,
        prompt=prompt
    )
    
    result = chain.run(input="test")
    assert result == "test response"
```

## Resources

- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction.html)
- [LangChain GitHub](https://github.com/langchain-ai/langchain)
- [LangChain Examples](https://python.langchain.com/docs/gallery)