# Creating the LLM Agents

In this section, we'll create the specialized LLM agents that form the core of our Text Summarizer Agent. Each agent is responsible for a specific aspect of the summarization process.

## Understanding LLM Agents in ADK

In the Agent Development Kit (ADK), an `LlmAgent` is a wrapper around a language model that:

1. Takes input from the session state
2. Sends a prompt to the LLM
3. Processes the response
4. Stores the result back in the session state

Each agent has:
- A model identifier
- A name
- An instruction (prompt)
- Input and output keys for state management

## Creating the Agent Definitions

Create a new file `src/text_summarizer_agent.py` and add the following imports:

```python
"""
Text Summarizer Agent Implementation

This module implements a text summarizer agent using the Agent Development Kit (ADK).
The agent orchestrates several specialized sub-agents to analyze, extract, summarize,
refine, and format text content.
"""

from typing import AsyncGenerator, Dict, Any
import asyncio

from google.adk.agents.base_agent import BaseAgent
from google.adk.agents.llm_agent import LlmAgent
from google.adk.agents.workflow_agents.loop_agent import LoopAgent
from google.adk.contexts.invocation_context import InvocationContext
from google.adk.events.event import Event
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types
from google.genai.types import Content, Part

# Define model constants
GEMINI_MODEL = "gemini-2.0-flash"
GPT_MODEL = "openai/gpt-4o"
CLAUDE_MODEL = "anthropic/claude-3-sonnet-20240229"
```

## 1. Content Analyzer Agent

The Content Analyzer Agent examines the input text to determine its type, structure, and key characteristics:

```python
content_analyzer_agent = LlmAgent(
    model=GEMINI_MODEL,
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
    output_key="content_analysis"
)
```

## 2. Extraction Agent

The Extraction Agent identifies and extracts the most important information from the text:

```python
extraction_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="extraction_agent",
    instruction="""
    Based on the content analysis, extract the most important information from the text.
    Focus on main ideas, key arguments, supporting evidence, and critical conclusions.
    
    Return a JSON object with the following fields:
    - main_ideas: List of the main ideas or arguments
    - supporting_evidence: List of supporting evidence or examples
    - key_facts: List of important facts or statistics
    - conclusions: List of critical conclusions or takeaways
    """,
    input_key="input_text",
    output_key="extracted_content"
)
```

## 3. Summarization Agent

The Summarization Agent generates the initial summary based on the extracted content:

```python
summarization_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="summarization_agent",
    instruction="""
    Generate a concise summary based on the extracted content.
    Preserve the key information and maintain logical flow.
    Prioritize information based on importance.
    
    The summary should be coherent, accurate, and capture the essence of the original text.
    """,
    input_key="extracted_content",
    output_key="initial_summary"
)
```

## 4. Refinement Agent

The Refinement Agent iteratively improves the summary:

```python
refinement_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="refinement_agent",
    instruction="""
    Review and improve the summary by checking for accuracy, completeness, clarity, and conciseness.
    Ensure the summary accurately represents the original text without distortion.
    
    Return a JSON object with the following fields:
    - refined_summary: The improved summary
    - needs_refinement: "true" if further refinement is needed, "false" otherwise
    """,
    input_key="initial_summary",
    output_key="refinement_result"
)
```

## 5. Format Agent

The Format Agent formats the final summary according to user preferences:

```python
format_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="format_agent",
    instruction="""
    Format the refined summary according to the specified preferences.
    Adjust length, style, and focus as requested.
    
    Preferences:
    - length: short, medium, long
    - style: narrative, bullet points, structured
    - focus: general, specific aspects
    - audience: technical, general, executive
    
    Return the formatted summary.
    """,
    input_key="refined_summary",
    output_key="final_summary"
)
```

## Understanding the Agent Configuration

Let's break down the key components of each agent:

- **model**: The LLM model to use (e.g., `gemini-2.0-flash`)
- **name**: A unique identifier for the agent
- **instruction**: The prompt that guides the LLM's behavior
- **input_key**: The key in the session state where the agent gets its input
- **output_key**: The key in the session state where the agent stores its output

## State Flow Between Agents

The agents communicate through the session state:

1. **Content Analyzer**: Reads `input_text` → Writes `content_analysis`
2. **Extraction Agent**: Reads `input_text` → Writes `extracted_content`
3. **Summarization Agent**: Reads `extracted_content` → Writes `initial_summary`
4. **Refinement Agent**: Reads `initial_summary` → Writes `refinement_result`
5. **Format Agent**: Reads `refined_summary` → Writes `final_summary`

## Customizing the Agents

You can customize these agents by:

- Changing the model (e.g., using GPT or Claude instead of Gemini)
- Modifying the instructions to focus on specific aspects
- Adjusting the input and output keys to match your workflow

## Next Steps

Now that we've defined our LLM agents, we need to create a custom orchestrator to coordinate them. The orchestrator will manage the flow of data between agents and handle the iterative refinement process.

Continue to [Building the Custom Orchestrator](03-Building-Custom-Orchestrator.md).
