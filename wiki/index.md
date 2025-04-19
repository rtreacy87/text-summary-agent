# Text Summarizer Agent Wiki

Welcome to the Text Summarizer Agent Wiki! This wiki provides comprehensive guidance on building, customizing, and troubleshooting a text summarizer agent using the Agent Development Kit (ADK).

## Table of Contents

1. [Introduction](00-Introduction.md)
2. [Setting Up Your Environment](01-Environment-Setup.md)
3. [Creating the LLM Agents](02-Creating-LLM-Agents.md)
4. [Building the Custom Orchestrator](03-Building-Custom-Orchestrator.md)
5. [Running and Testing](04-Running-and-Testing.md)
6. [Customization and Extensions](05-Customization-and-Extensions.md)
7. [Troubleshooting](06-Troubleshooting.md)
8. [Local Model Hosting](07-Local-Model-Hosting.md)
9. [Secure API Key Management](08-Secure-API-Key-Management.md)

## Overview

The Text Summarizer Agent is a powerful tool for generating concise, accurate summaries of various types of text content. It uses a multi-agent approach, orchestrating several specialized agents to handle different aspects of the summarization process.

### Key Features

- **Multi-agent Architecture**: Coordinates specialized agents for analysis, extraction, summarization, refinement, and formatting
- **Iterative Refinement**: Improves summaries through multiple refinement iterations
- **Customizable Preferences**: Adjusts summary length, style, focus, and target audience
- **Versatile Content Support**: Handles various content types including articles, code, and conversations
- **Extensible Design**: Easily customizable for specific domains or requirements

### Getting Started

Start with the [Introduction](00-Introduction.md) to understand the overall architecture and flow of the Text Summarizer Agent. Then follow the step-by-step guides to build your own implementation.

### Prerequisites

- Python 3.9 or higher
- Agent Development Kit (ADK)
- Access to LLM APIs (Gemini, GPT, Claude, etc.) or locally hosted models

## Quick Start

For those who want to get up and running quickly:

1. Set up your environment ([Setting Up Your Environment](01-Environment-Setup.md))
2. Create the LLM agents ([Creating the LLM Agents](02-Creating-LLM-Agents.md))
3. Build the custom orchestrator ([Building the Custom Orchestrator](03-Building-Custom-Orchestrator.md))
4. Run the example script ([Running and Testing](04-Running-and-Testing.md))
5. For local deployment, explore [Local Model Hosting](07-Local-Model-Hosting.md)
6. Secure your API keys using [Secure API Key Management](08-Secure-API-Key-Management.md)

## Additional Resources

- [Agent Development Kit Documentation](https://google.github.io/adk-docs/)
- [Text Summarizer Agent Design Document](../docs/agents/text-summarizer-agent.md)
- [Example Implementation](../examples/python/snippets/agents/text-summarizer/)
