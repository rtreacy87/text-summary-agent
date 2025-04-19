# Hosting Models Locally

While cloud-based LLM APIs offer powerful capabilities, there are several reasons you might want to host models locally:

- **Privacy**: Keep sensitive data within your infrastructure
- **Cost**: Avoid per-token charges for high-volume applications
- **Latency**: Reduce network-related delays
- **Offline Access**: Run your summarizer without internet connectivity
- **Customization**: Fine-tune models on your specific data

This section explores how to use locally hosted models with your Text Summarizer Agent.

## Using Ollama for Local Model Hosting

[Ollama](https://ollama.ai/) is an open-source tool that makes it easy to run large language models locally. It provides a simple API that's compatible with many popular models.

### Setting Up Ollama

1. **Install Ollama**:
   
   Visit [ollama.ai](https://ollama.ai/) and download the appropriate version for your operating system.

   For macOS:
   ```bash
   curl -fsSL https://ollama.ai/install.sh | sh
   ```

   For Linux:
   ```bash
   curl -fsSL https://ollama.ai/install.sh | sh
   ```

   For Windows, download the installer from the website.

2. **Pull a Model**:

   Pull a model that's suitable for summarization tasks:
   ```bash
   # Pull Llama 3 8B
   ollama pull llama3
   
   # Pull Mistral 7B
   ollama pull mistral
   
   # Pull Phi-3 Mini
   ollama pull phi3
   ```

3. **Start the Ollama Server**:

   Ollama automatically starts a server on port 11434. You can verify it's running:
   ```bash
   curl http://localhost:11434/api/tags
   ```

### Integrating Ollama with the Text Summarizer Agent

To use Ollama models with your Text Summarizer Agent, you'll need to create a custom client that interfaces with the Ollama API.

1. **Create an Ollama Client**:

   Create a new file `src/ollama_client.py`:

   ```python
   """
   Ollama API client for the Text Summarizer Agent.
   """
   
   import json
   import requests
   from typing import Dict, Any, List, Optional
   
   class OllamaClient:
       """Client for interacting with locally hosted Ollama models."""
       
       def __init__(self, base_url: str = "http://localhost:11434"):
           """
           Initialize the Ollama client.
           
           Args:
               base_url: The base URL of the Ollama API server
           """
           self.base_url = base_url
           self.api_endpoint = f"{base_url}/api/generate"
           
       def generate(
           self,
           model: str,
           prompt: str,
           system: Optional[str] = None,
           temperature: float = 0.7,
           max_tokens: int = 2000
       ) -> str:
           """
           Generate text using an Ollama model.
           
           Args:
               model: The name of the model to use
               prompt: The prompt to send to the model
               system: Optional system message
               temperature: Sampling temperature (0.0 to 1.0)
               max_tokens: Maximum number of tokens to generate
               
           Returns:
               The generated text
           """
           payload = {
               "model": model,
               "prompt": prompt,
               "temperature": temperature,
               "max_tokens": max_tokens
           }
           
           if system:
               payload["system"] = system
           
           try:
               response = requests.post(self.api_endpoint, json=payload)
               response.raise_for_status()
               
               # Ollama returns a JSON string with a 'response' field
               result = response.json()
               return result.get("response", "")
               
           except requests.exceptions.RequestException as e:
               print(f"Error calling Ollama API: {e}")
               return f"Error: {str(e)}"
       
       def list_models(self) -> List[str]:
           """
           List available models in Ollama.
           
           Returns:
               List of model names
           """
           try:
               response = requests.get(f"{self.base_url}/api/tags")
               response.raise_for_status()
               
               result = response.json()
               models = [model["name"] for model in result.get("models", [])]
               return models
               
           except requests.exceptions.RequestException as e:
               print(f"Error listing Ollama models: {e}")
               return []
   ```

2. **Create a Custom LLM Agent Factory**:

   Create a new file `src/local_llm_agent.py`:

   ```python
   """
   Custom LLM Agent implementation for locally hosted models.
   """
   
   from google.adk.agents.llm_agent import LlmAgent
   from google.adk.contexts.invocation_context import InvocationContext
   from google.adk.events.event import Event
   from google.genai.types import Content, Part
   from typing import AsyncGenerator, Dict, Any, Optional
   
   from ollama_client import OllamaClient
   
   class LocalLlmAgentFactory:
       """Factory for creating LLM agents that use locally hosted models."""
       
       def __init__(self, ollama_client: OllamaClient):
           """
           Initialize the factory.
           
           Args:
               ollama_client: The Ollama client to use
           """
           self.ollama_client = ollama_client
           
       def create_agent(
           self,
           model: str,
           name: str,
           instruction: str,
           input_key: str,
           output_key: str,
           temperature: float = 0.7,
           max_tokens: int = 2000,
           system_message: Optional[str] = None
       ) -> LlmAgent:
           """
           Create an LLM agent that uses a locally hosted model.
           
           Args:
               model: The name of the model to use
               name: The name of the agent
               instruction: The instruction for the agent
               input_key: The key in the session state to get input from
               output_key: The key in the session state to store output to
               temperature: Sampling temperature
               max_tokens: Maximum number of tokens to generate
               system_message: Optional system message
               
           Returns:
               An LLM agent configured to use the local model
           """
           # Create a custom handler for the LLM agent
           async def custom_handler(ctx: InvocationContext, content: Content) -> str:
               # Get the input from the session state
               input_text = ctx.session.state.get(input_key, "")
               
               # Combine the instruction and input
               prompt = f"{instruction}\n\nInput:\n{input_text}"
               
               # Generate text using the Ollama client
               response = self.ollama_client.generate(
                   model=model,
                   prompt=prompt,
                   system=system_message,
                   temperature=temperature,
                   max_tokens=max_tokens
               )
               
               return response
           
           # Create the LLM agent with the custom handler
           agent = LlmAgent(
               model="custom",  # This is just a placeholder
               name=name,
               instruction=instruction,
               input_key=input_key,
               output_key=output_key,
               custom_llm_handler=custom_handler
           )
           
           return agent
   ```

3. **Update the Text Summarizer Agent to Use Local Models**:

   Modify your `src/text_summarizer_agent.py` to use the local models:

   ```python
   # Add imports for local model support
   from ollama_client import OllamaClient
   from local_llm_agent import LocalLlmAgentFactory
   
   # Create Ollama client
   ollama_client = OllamaClient()
   
   # Create local LLM agent factory
   local_llm_factory = LocalLlmAgentFactory(ollama_client)
   
   # Define local model constants
   LOCAL_MODEL = "llama3"  # or "mistral", "phi3", etc.
   
   # Create agents using local models
   content_analyzer_agent = local_llm_factory.create_agent(
       model=LOCAL_MODEL,
       name="content_analyzer",
       instruction="""
       Analyze the provided text to determine its type, structure, and key characteristics.
       Identify the domain/subject matter and important entities or concepts.
       
       Return a JSON object with the following fields:
       - content_type: The type of content (article, code, conversation, etc.)
       - structure: The structure of the content (sections, paragraphs, dialogue)
       - length: Approximate length (short, medium, long)
       - complexity: Complexity level (simple, moderate, complex)
       - domain: The subject matter or domain of the content
       - key_entities: List of important entities or concepts
       """,
       input_key="input_text",
       output_key="content_analysis",
       system_message="You are an expert content analyzer. Your task is to analyze text and identify its key characteristics."
   )
   
   # ... Create other agents similarly ...
   ```

## Using LM Studio

[LM Studio](https://lmstudio.ai/) is another excellent option for running models locally with a user-friendly interface.

### Setting Up LM Studio

1. **Download and Install LM Studio**:
   
   Visit [lmstudio.ai](https://lmstudio.ai/) and download the appropriate version for your operating system.

2. **Download Models**:
   
   Use the built-in model browser to download models suitable for summarization.

3. **Start the Local Server**:
   
   In LM Studio, click on "Local Server" and start the server. This will provide an OpenAI-compatible API endpoint.

### Integrating LM Studio with the Text Summarizer Agent

LM Studio provides an OpenAI-compatible API, so you can use the standard OpenAI client:

```python
from openai import OpenAI

# Configure the client to use the local server
client = OpenAI(
    base_url="http://localhost:1234/v1",  # LM Studio default
    api_key="lm-studio"  # Can be any string
)

# Example function to generate text
def generate_with_lm_studio(prompt, model="local-model"):
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
        max_tokens=2000
    )
    return response.choices[0].message.content
```

## Using LocalAI

[LocalAI](https://github.com/go-skynet/LocalAI) is an open-source project that provides a drop-in replacement for OpenAI API using local models.

### Setting Up LocalAI

1. **Install LocalAI**:
   
   Using Docker:
   ```bash
   docker run -p 8080:8080 localai/localai:latest
   ```

2. **Download Models**:
   
   LocalAI can automatically download models from Hugging Face:
   ```bash
   curl -X POST http://localhost:8080/models/apply -d '{
     "url": "github:go-skynet/model-gallery/mistral-7b.yaml"
   }'
   ```

### Integrating LocalAI with the Text Summarizer Agent

LocalAI provides an OpenAI-compatible API, so you can use the standard OpenAI client:

```python
from openai import OpenAI

# Configure the client to use LocalAI
client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="localai"  # Can be any string
)

# Example function to generate text
def generate_with_localai(prompt, model="mistral-7b"):
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
        max_tokens=2000
    )
    return response.choices[0].message.content
```

## Hybrid Approach: Combining Local and Cloud Models

For the best of both worlds, you can implement a hybrid approach that uses local models for some tasks and cloud models for others:

```python
class HybridTextSummarizerAgent(BaseAgent):
    """
    A text summarizer agent that uses both local and cloud models.
    
    Uses local models for initial processing and cloud models for refinement.
    """
    
    def __init__(
        self,
        local_content_analyzer: LlmAgent,
        local_extraction_agent: LlmAgent,
        cloud_summarization_agent: LlmAgent,
        cloud_refinement_agent: LlmAgent,
        cloud_format_agent: LlmAgent,
        name: str = "hybrid_text_summarizer",
    ):
        # ... initialization code ...
        
    async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        # Use local models for initial processing
        async for event in self.local_content_analyzer.run_async(ctx):
            yield event
            
        async for event in self.local_extraction_agent.run_async(ctx):
            yield event
        
        # Use cloud models for summarization and refinement
        async for event in self.cloud_summarization_agent.run_async(ctx):
            yield event
        
        # ... rest of the implementation ...
```

## Performance Considerations for Local Models

When using locally hosted models, consider the following:

1. **Hardware Requirements**:
   - Most modern LLMs require a GPU for reasonable performance
   - For CPU-only systems, use smaller models (7B parameters or less)
   - Recommended: 16GB+ RAM, dedicated GPU with 8GB+ VRAM

2. **Model Selection**:
   - Smaller models (7B) are faster but may produce lower quality summaries
   - Quantized models (4-bit, 8-bit) use less memory but may have slightly reduced quality
   - Consider models specifically fine-tuned for summarization tasks

3. **Batch Processing**:
   - For summarizing multiple documents, implement a queue system
   - Process documents sequentially to avoid memory issues

4. **Caching**:
   - Implement caching for repeated summarization requests
   - Cache intermediate results (content analysis, extraction) to speed up processing

## Example: Configurable Model Source

Create a configuration system that allows switching between local and cloud models:

```python
# config.py
MODEL_CONFIG = {
    "use_local_models": True,
    "local_model_name": "llama3",
    "cloud_model_name": "gemini-2.0-flash",
    "local_server_url": "http://localhost:11434",
    "hybrid_mode": False  # If True, uses both local and cloud models
}

# Factory function to create agents based on configuration
def create_agent(name, instruction, input_key, output_key, system_message=None):
    if MODEL_CONFIG["use_local_models"]:
        return local_llm_factory.create_agent(
            model=MODEL_CONFIG["local_model_name"],
            name=name,
            instruction=instruction,
            input_key=input_key,
            output_key=output_key,
            system_message=system_message
        )
    else:
        return LlmAgent(
            model=MODEL_CONFIG["cloud_model_name"],
            name=name,
            instruction=instruction,
            input_key=input_key,
            output_key=output_key
        )
```

## Next Steps

Now that you've learned how to host models locally, you might want to explore:

- Fine-tuning local models on your specific summarization tasks
- Implementing a model evaluation framework to compare local and cloud models
- Creating a web interface that allows users to choose between local and cloud models

Continue to [Secure API Key Management](08-Secure-API-Key-Management.md) to learn how to securely manage API keys for cloud models.
