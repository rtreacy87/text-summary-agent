# Customization and Extensions

Now that you have a working Text Summarizer Agent, let's explore ways to customize and extend it to better suit your specific needs.

## Customizing the Existing Agents

### Changing the Models

You can easily switch to different LLM models based on your preferences or requirements:

```python
# Use GPT-4o for the summarization agent
summarization_agent = LlmAgent(
    model=GPT_MODEL,  # "openai/gpt-4o"
    name="summarization_agent",
    instruction="...",
    input_key="extracted_content",
    output_key="initial_summary"
)

# Use Claude for the refinement agent
refinement_agent = LlmAgent(
    model=CLAUDE_MODEL,  # "anthropic/claude-3-sonnet-20240229"
    name="refinement_agent",
    instruction="...",
    input_key="initial_summary",
    output_key="refinement_result"
)
```

### Modifying the Instructions

You can customize the instructions to focus on specific aspects or change the behavior of the agents:

```python
# Customize the content analyzer to focus on technical content
content_analyzer_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="content_analyzer",
    instruction="""
    Analyze the provided text with a focus on technical content.
    Identify technical terms, concepts, algorithms, and methodologies.
    Pay special attention to code snippets, mathematical formulas, and technical diagrams.
    
    Return a JSON object with the following fields:
    - content_type: The type of technical content (code, documentation, research paper, etc.)
    - technical_domain: The specific technical domain (e.g., machine learning, web development)
    - key_technical_concepts: List of important technical concepts or terms
    - complexity_level: Technical complexity (beginner, intermediate, advanced)
    """,
    input_key="input_text",
    output_key="content_analysis"
)
```

### Adjusting the Refinement Loop

You can change the number of refinement iterations or the condition for continuing refinement:

```python
# Increase the maximum number of refinement iterations
self.refinement_loop = LoopAgent(
    sub_agents=[refinement_agent],
    name="refinement_loop",
    max_iterations=5,  # Increased from 3 to 5
    condition_key="needs_refinement",
    condition_value="true"
)
```

## Adding New Capabilities

### Adding a Fact-Checking Agent

You can add a fact-checking agent to verify the accuracy of the summary:

```python
fact_checking_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="fact_checking_agent",
    instruction="""
    Compare the summary with the original text to verify factual accuracy.
    Identify any factual errors, misrepresentations, or omissions.
    
    Return a JSON object with the following fields:
    - accuracy_score: A score from 1-10 indicating factual accuracy
    - identified_errors: List of any factual errors found
    - suggested_corrections: List of suggested corrections
    """,
    input_key="fact_check_data",
    output_key="fact_check_result"
)
```

Then update the `TextSummarizerAgent` to include this new agent:

```python
def __init__(
    self,
    content_analyzer: LlmAgent,
    extraction_agent: LlmAgent,
    summarization_agent: LlmAgent,
    refinement_agent: LlmAgent,
    format_agent: LlmAgent,
    fact_checking_agent: LlmAgent,  # New agent
    name: str = "text_summarizer",
):
    # ... existing code ...
    self.fact_checking_agent = fact_checking_agent
    
    # ... existing code ...
    
    super().__init__(
        name=name,
        sub_agents=[
            content_analyzer,
            extraction_agent,
            summarization_agent,
            self.refinement_loop,
            fact_checking_agent,  # Add to sub_agents list
            format_agent
        ]
    )
```

And update the `_run_async_impl` method to include the fact-checking step:

```python
# After refinement loop and before formatting
# Prepare data for fact checking
ctx.session.state["fact_check_data"] = {
    "original_text": input_text,
    "summary": ctx.session.state["refined_summary"]
}

# Run fact checking agent
async for event in self.fact_checking_agent.run_async(ctx):
    yield event

# Get fact check results
fact_check_result = ctx.session.state.get("fact_check_result", {})

# If accuracy is low, add a note to the summary
if isinstance(fact_check_result, dict) and fact_check_result.get("accuracy_score", 10) < 7:
    refined_summary = ctx.session.state["refined_summary"]
    ctx.session.state["refined_summary"] = refined_summary + "\n\nNote: This summary may contain factual inaccuracies. Please verify important information against the original source."
```

### Adding Support for Multi-modal Content

To handle images along with text, you can extend the content analyzer:

```python
# In example_usage.py
from google.genai.types import Part

# Example with an image
image_path = "path/to/image.jpg"
with open(image_path, "rb") as f:
    image_data = f.read()

# Create a multi-modal content
multimodal_content = {
    "text": "This chart shows the quarterly sales figures for our company.",
    "image": image_data
}

# Run the summarizer with multi-modal content
async def summarize_with_image():
    result = await runner.run_async(
        app_name="text_summarizer_app",
        user_id="user_123",
        session_id="session_456",
        input_content=Content(
            parts=[
                Part.from_text("Please summarize this content."),
                Part.from_text(multimodal_content["text"]),
                Part.from_image(multimodal_content["image"])
            ]
        ),
        state={
            "input_text": multimodal_content["text"],
            "input_image": True,  # Flag indicating image is present
            "preferences": PREFERENCES["article"]
        }
    )
    return result.session.state.get("final_summary", "")
```

Then update the content analyzer instruction to handle images:

```python
content_analyzer_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="content_analyzer",
    instruction="""
    Analyze the provided content, which may include both text and images.
    If an image is present, describe its content and how it relates to the text.
    
    Return a JSON object with the following fields:
    - content_type: The type of content (article, code, conversation, image, etc.)
    - has_image: Boolean indicating if an image is present
    - image_description: Description of the image (if present)
    - image_text_relationship: How the image relates to the text (if both are present)
    - ... other fields as before ...
    """,
    input_key="input_text",
    output_key="content_analysis"
)
```

## Creating Domain-Specific Variants

### Legal Document Summarizer

```python
legal_analyzer_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="legal_analyzer",
    instruction="""
    Analyze the provided legal document to identify its type, structure, and key legal elements.
    Identify parties, obligations, rights, conditions, dates, and jurisdictional information.
    
    Return a JSON object with the following fields:
    - document_type: The type of legal document (contract, statute, case, etc.)
    - parties_involved: List of parties mentioned in the document
    - key_provisions: List of important legal provisions
    - obligations: List of obligations specified in the document
    - effective_dates: Any relevant dates mentioned
    - jurisdiction: The applicable jurisdiction
    """,
    input_key="input_text",
    output_key="legal_analysis"
)

legal_summarization_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="legal_summarization",
    instruction="""
    Generate a concise summary of the legal document based on the legal analysis.
    Focus on the key provisions, obligations, and rights.
    Maintain legal precision while making the content accessible.
    
    The summary should be legally accurate and highlight the most important aspects of the document.
    """,
    input_key="legal_analysis",
    output_key="legal_summary"
)
```

### Technical Documentation Summarizer

```python
technical_analyzer_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="technical_analyzer",
    instruction="""
    Analyze the provided technical documentation to identify its structure, purpose, and key technical components.
    Identify APIs, functions, classes, parameters, return values, and examples.
    
    Return a JSON object with the following fields:
    - doc_type: The type of technical documentation (API reference, tutorial, guide, etc.)
    - programming_language: The programming language used (if applicable)
    - key_components: List of important technical components (APIs, functions, classes)
    - parameters: List of parameters and their descriptions
    - return_values: List of return values and their descriptions
    - examples: List of code examples provided
    """,
    input_key="input_text",
    output_key="technical_analysis"
)

technical_summarization_agent = LlmAgent(
    model=GEMINI_MODEL,
    name="technical_summarization",
    instruction="""
    Generate a concise summary of the technical documentation based on the technical analysis.
    Focus on the purpose, key components, and usage examples.
    Maintain technical accuracy while making the content accessible.
    
    The summary should be technically accurate and provide a clear overview of how to use the described components.
    """,
    input_key="technical_analysis",
    output_key="technical_summary"
)
```

## Implementing a Web Interface

You can create a simple web interface using Flask:

```python
# app.py
from flask import Flask, request, render_template, jsonify
import asyncio
from text_summarizer_agent import run_text_summarizer

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/summarize', methods=['POST'])
def summarize():
    data = request.json
    text = data.get('text', '')
    preferences = data.get('preferences', {
        "length": "medium",
        "style": "narrative",
        "focus": "general",
        "audience": "general"
    })
    
    # Run the summarizer
    summary = asyncio.run(run_text_summarizer(text, preferences))
    
    return jsonify({
        'original_length': len(text),
        'summary_length': len(summary),
        'summary': summary
    })

if __name__ == '__main__':
    app.run(debug=True)
```

Create a simple HTML template:

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Text Summarizer</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        textarea { width: 100%; height: 200px; margin-bottom: 10px; }
        .preferences { margin-bottom: 20px; }
        .result { margin-top: 20px; border-top: 1px solid #ccc; padding-top: 20px; }
    </style>
</head>
<body>
    <h1>Text Summarizer</h1>
    
    <textarea id="input-text" placeholder="Enter text to summarize..."></textarea>
    
    <div class="preferences">
        <h3>Preferences</h3>
        
        <label>Length:</label>
        <select id="length">
            <option value="short">Short</option>
            <option value="medium" selected>Medium</option>
            <option value="long">Long</option>
        </select>
        
        <label>Style:</label>
        <select id="style">
            <option value="narrative" selected>Narrative</option>
            <option value="bullet_points">Bullet Points</option>
            <option value="structured">Structured</option>
        </select>
        
        <label>Focus:</label>
        <select id="focus">
            <option value="general" selected>General</option>
            <option value="specific">Specific Aspects</option>
        </select>
        
        <label>Audience:</label>
        <select id="audience">
            <option value="general" selected>General</option>
            <option value="technical">Technical</option>
            <option value="executive">Executive</option>
        </select>
    </div>
    
    <button id="summarize-btn">Summarize</button>
    
    <div class="result" id="result" style="display: none;">
        <h3>Summary</h3>
        <p id="summary-stats"></p>
        <div id="summary-text"></div>
    </div>
    
    <script>
        document.getElementById('summarize-btn').addEventListener('click', async () => {
            const text = document.getElementById('input-text').value;
            const preferences = {
                length: document.getElementById('length').value,
                style: document.getElementById('style').value,
                focus: document.getElementById('focus').value,
                audience: document.getElementById('audience').value
            };
            
            document.getElementById('summarize-btn').disabled = true;
            document.getElementById('summarize-btn').textContent = 'Summarizing...';
            
            try {
                const response = await fetch('/summarize', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ text, preferences })
                });
                
                const data = await response.json();
                
                document.getElementById('summary-stats').textContent = 
                    `Original: ${data.original_length} characters | Summary: ${data.summary_length} characters`;
                document.getElementById('summary-text').textContent = data.summary;
                document.getElementById('result').style.display = 'block';
            } catch (error) {
                console.error('Error:', error);
                alert('An error occurred while summarizing the text.');
            } finally {
                document.getElementById('summarize-btn').disabled = false;
                document.getElementById('summarize-btn').textContent = 'Summarize';
            }
        });
    </script>
</body>
</html>
```

To run the web interface:

```bash
pip install flask
python app.py
```

Then open your browser to `http://localhost:5000`.

## Next Steps

Now that you've learned how to customize and extend the Text Summarizer Agent, you might encounter some challenges along the way. In the next section, we'll cover common troubleshooting tips and best practices.

Continue to [Troubleshooting](06-Troubleshooting.md).
