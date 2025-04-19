# Setting Up Your Environment

Before building the Text Summarizer Agent, you need to set up your development environment. This guide will walk you through the necessary steps.

## Project Structure

First, create a directory structure for your project:

```bash
mkdir -p text_summarizer_agent/src
cd text_summarizer_agent
```

Your project structure will look like this:

```
text_summarizer_agent/
├── src/
│   ├── __init__.py
│   ├── text_summarizer_agent.py
│   └── example_usage.py
├── requirements.txt
└── README.md
```

## Dependencies

Create a `requirements.txt` file with the following dependencies:

```
google-adk>=0.1.0
google-generativeai>=0.3.0
openai>=1.0.0
anthropic>=0.5.0
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

## API Keys

To use LLM models, you'll need API keys from the respective providers. Here's how to set them up:

### Google Gemini API

1. Go to the [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Create an API key
3. Set it as an environment variable:
   ```bash
   export GOOGLE_API_KEY="your-api-key"
   ```

### OpenAI API (Optional)

If you want to use GPT models:

1. Go to [OpenAI API Keys](https://platform.openai.com/account/api-keys)
2. Create a new API key
3. Set it as an environment variable:
   ```bash
   export OPENAI_API_KEY="your-api-key"
   ```

### Anthropic API (Optional)

If you want to use Claude models:

1. Go to [Anthropic Console](https://console.anthropic.com/)
2. Create an API key
3. Set it as an environment variable:
   ```bash
   export ANTHROPIC_API_KEY="your-api-key"
   ```

## Create Basic Files

### Create `__init__.py`

Create an empty `__init__.py` file in the `src` directory:

```bash
touch src/__init__.py
```

### Create a Basic README.md

Create a `README.md` file in the root directory:

```markdown
# Text Summarizer Agent

A powerful text summarization system built with the Agent Development Kit (ADK).

## Features

- Summarize various types of text content (articles, code, conversations)
- Customize summaries based on preferences
- Iteratively refine summaries for improved quality

## Installation

```bash
pip install -r requirements.txt
```

## Usage

```bash
python src/example_usage.py
```
```

## Verify Setup

To verify that your environment is set up correctly, create a simple test script:

```python
# test_setup.py
import os
import google.generativeai as genai

# Check if API key is set
api_key = os.environ.get("GOOGLE_API_KEY")
if not api_key:
    print("Error: GOOGLE_API_KEY environment variable not set")
    exit(1)

# Configure the API
genai.configure(api_key=api_key)

# List available models
models = genai.list_models()
for model in models:
    if "generateContent" in model.supported_generation_methods:
        print(f"Available model: {model.name}")

print("Setup successful!")
```

Run the test script:

```bash
python test_setup.py
```

If everything is set up correctly, you should see a list of available models and "Setup successful!" printed to the console.

## Next Steps

Now that your environment is set up, you're ready to start building the Text Summarizer Agent. In the next section, we'll create the LLM agents that will handle different aspects of the summarization process.

Continue to [Creating the LLM Agents](02-Creating-LLM-Agents.md).
