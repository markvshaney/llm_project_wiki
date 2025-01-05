# Ollama Guide

## Overview
Ollama is an open-source tool for running large language models (LLMs) locally. It simplifies the process of downloading, running, and managing various open-source LLMs on your local machine.

## Installation

### macOS and Linux
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows
Download the installer from the official website: [Ollama.com](https://ollama.com)

## Basic Usage

### Starting Ollama
```bash
# Start the Ollama service
ollama serve

# In a new terminal, run a model
ollama run llama2
```

### Managing Models

```bash
# List available models
ollama list

# Pull a specific model
ollama pull mistral

# Remove a model
ollama rm mistral

# Show model information
ollama show llama2
```

## Working with Different Models

### Available Models
- Llama 2
- Mistral
- CodeLlama
- Phi
- Neural Chat
- And many more from [Ollama Library](https://ollama.com/library)

### Model Tags
```bash
# Pull specific model variants
ollama pull llama2:7b
ollama pull llama2:13b
ollama pull llama2:70b
```

## Using the API

### Python Integration
```python
import requests

def generate_text(prompt):
    response = requests.post('http://localhost:11434/api/generate',
        json={
            'model': 'llama2',
            'prompt': prompt
        },
        stream=True
    )
    
    for line in response.iter_lines():
        if line:
            json_response = json.loads(line)
            if json_response.get('response'):
                print(json_response['response'], end='')

# Usage
generate_text("Write a poem about coding")
```

### REST API Endpoints

#### Generate Response
```bash
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Why is the sky blue?"
}'
```

#### Create Model
```bash
curl -X POST http://localhost:11434/api/create -d '{
  "name": "custom-model",
  "modelfile": "FROM llama2\nSYSTEM You are a helpful assistant."
}'
```

## Creating Custom Models

### Modelfile Syntax
```dockerfile
FROM llama2
PARAMETER temperature 0.7
PARAMETER top_p 0.9
SYSTEM You are a helpful assistant specialized in programming.
```

### Building Custom Models
```bash
# Create and build a custom model
ollama create custom-model -f Modelfile

# Run the custom model
ollama run custom-model
```

## Performance Optimization

### Hardware Requirements
- Minimum 8GB RAM
- Recommended 16GB+ RAM for larger models
- GPU acceleration supported for NVIDIA GPUs

### GPU Acceleration
```bash
# Check GPU availability
nvidia-smi

# Run model with GPU
CUDA_VISIBLE_DEVICES=0 ollama run llama2
```

### Memory Management
```bash
# Run with specific GPU memory constraints
CUDA_VISIBLE_DEVICES=0 CUDA_MEM_FRACTION=0.7 ollama run llama2
```

## Best Practices

1. Model Selection
   - Choose smaller models for faster responses
   - Use larger models for more complex tasks
   - Consider quantized versions for resource constraints

2. Prompt Engineering
   - Be specific and clear in prompts
   - Use system prompts to set context
   - Include examples for better responses

3. Resource Management
   - Monitor system resources
   - Clean unused models
   - Use appropriate model sizes for your hardware

## Common Use Cases

### Chat Applications
```python
from typing import Iterator
import requests

def chat(messages: list, model: str = "llama2") -> Iterator[str]:
    response = requests.post(
        'http://localhost:11434/api/chat',
        json={'model': model, 'messages': messages},
        stream=True
    )
    for line in response.iter_lines():
        if line:
            yield json.loads(line)['message']['content']

# Usage
messages = [
    {"role": "user", "content": "What is Python?"}
]
for response in chat(messages):
    print(response)
```

### Code Generation
```python
def generate_code(prompt: str) -> str:
    response = requests.post(
        'http://localhost:11434/api/generate',
        json={
            'model': 'codellama',
            'prompt': prompt,
            'system': 'You are a helpful coding assistant.'
        }
    )
    return response.json()['response']
```

## Troubleshooting

1. Connection Issues
   - Verify Ollama service is running
   - Check port 11434 is available
   - Confirm firewall settings

2. Memory Problems
   - Monitor RAM usage
   - Use smaller models
   - Close unnecessary applications

3. GPU Issues
   - Update NVIDIA drivers
   - Check CUDA installation
   - Verify GPU compatibility

## Resources

- [Official Documentation](https://ollama.com/docs)
- [GitHub Repository](https://github.com/ollama/ollama)
- [Model Library](https://ollama.com/library)
- [Community Discord](https://discord.gg/ollama)