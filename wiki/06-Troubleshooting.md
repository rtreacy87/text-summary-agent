# Troubleshooting

In this section, we'll cover common issues you might encounter when working with the Text Summarizer Agent and how to resolve them.

## Common Issues and Solutions

### API Key and Authentication Issues

**Issue**: You receive authentication errors when trying to use LLM models.

**Solutions**:
1. **Check Environment Variables**:
   ```bash
   echo $GOOGLE_API_KEY
   echo $OPENAI_API_KEY
   echo $ANTHROPIC_API_KEY
   ```
   If they're not set or incorrect, set them again:
   ```bash
   export GOOGLE_API_KEY="your-api-key"
   ```

2. **Verify API Key Validity**:
   Create a simple test script to check if your API key works:
   ```python
   import google.generativeai as genai
   
   genai.configure(api_key="your-api-key")
   
   try:
       model = genai.GenerativeModel("gemini-2.0-flash")
       response = model.generate_content("Hello, world!")
       print("API key is valid. Response:", response.text)
   except Exception as e:
       print("API key validation failed:", e)
   ```

3. **Check Billing Status**:
   Ensure your account has active billing if required by the API provider.

### Model Availability Issues

**Issue**: The specified model is not available or returns an error.

**Solutions**:
1. **Check Model Name**:
   Ensure you're using the correct model name. Model names can change over time.

2. **Try Alternative Models**:
   If a specific model is unavailable, try an alternative:
   ```python
   # If gemini-2.0-flash is unavailable
   GEMINI_MODEL = "gemini-1.5-pro"  # Use an alternative model
   ```

3. **Check Model Quotas**:
   Some API providers have quotas or rate limits for specific models.

### State Management Issues

**Issue**: Agents aren't receiving the correct input or producing the expected output.

**Solutions**:
1. **Debug State Flow**:
   Add print statements to track the state at different points:
   ```python
   async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
       # Get input text and preferences from state
       input_text = ctx.session.state.get("input_text", "")
       print(f"Input text: {input_text[:100]}...")  # Print first 100 chars
       
       # After content analyzer
       print(f"Content analysis: {ctx.session.state.get('content_analysis', {})}")
       
       # ... rest of the method ...
   ```

2. **Check Input/Output Keys**:
   Ensure the input and output keys match between agents:
   ```python
   # If summarization_agent outputs to "summary" but refinement_agent expects "initial_summary"
   summarization_agent = LlmAgent(
       # ...
       output_key="initial_summary"  # Make sure this matches the input_key of the next agent
   )
   
   refinement_agent = LlmAgent(
       # ...
       input_key="initial_summary"  # Make sure this matches the output_key of the previous agent
   )
   ```

3. **Initialize Missing State**:
   Ensure all required state keys are initialized:
   ```python
   # Initialize state keys with default values if missing
   ctx.session.state["input_text"] = ctx.session.state.get("input_text", "")
   ctx.session.state["preferences"] = ctx.session.state.get("preferences", {})
   ```

### LLM Response Format Issues

**Issue**: LLM responses don't match the expected format (e.g., not valid JSON).

**Solutions**:
1. **Improve Prompts**:
   Make the instructions more explicit:
   ```python
   instruction="""
   Return a JSON object with the following fields:
   - refined_summary: The improved summary
   - needs_refinement: "true" if further refinement is needed, "false" otherwise
   
   IMPORTANT: Your response must be a valid JSON object with exactly these fields.
   Do not include any explanations or additional text outside the JSON object.
   Example response:
   {"refined_summary": "This is the refined summary text.", "needs_refinement": "false"}
   """
   ```

2. **Add Response Parsing**:
   Add robust parsing logic to handle unexpected formats:
   ```python
   refinement_result = ctx.session.state.get("refinement_result", {})
   
   # Handle string responses (non-JSON)
   if isinstance(refinement_result, str):
       try:
           import json
           refinement_result = json.loads(refinement_result)
       except:
           # If parsing fails, create a default structure
           refinement_result = {
               "refined_summary": refinement_result,
               "needs_refinement": "false"
           }
   
   # Now use refinement_result safely
   if isinstance(refinement_result, dict):
       ctx.session.state["refined_summary"] = refinement_result.get("refined_summary", ctx.session.state.get("refined_summary", ""))
       ctx.session.state["needs_refinement"] = refinement_result.get("needs_refinement", "false")
   ```

3. **Use Structured Output**:
   Some LLM APIs support structured output formats:
   ```python
   # For OpenAI with function calling
   summarization_agent = LlmAgent(
       model=GPT_MODEL,
       name="summarization_agent",
       instruction="Generate a concise summary based on the extracted content.",
       input_key="extracted_content",
       output_key="initial_summary",
       config={
           "response_format": {"type": "json_object"},
           "schema": {
               "type": "object",
               "properties": {
                   "summary": {"type": "string"}
               },
               "required": ["summary"]
           }
       }
   )
   ```

### Performance Issues

**Issue**: The summarization process is slow or times out.

**Solutions**:
1. **Use Faster Models**:
   Switch to faster, more efficient models:
   ```python
   # Use a faster model for less critical steps
   content_analyzer_agent = LlmAgent(
       model="gemini-2.0-flash",  # Faster model
       # ...
   )
   ```

2. **Reduce Refinement Iterations**:
   Limit the number of refinement iterations:
   ```python
   self.refinement_loop = LoopAgent(
       sub_agents=[refinement_agent],
       name="refinement_loop",
       max_iterations=2,  # Reduced from 3 to 2
       condition_key="needs_refinement",
       condition_value="true"
   )
   ```

3. **Implement Caching**:
   Cache results for frequently summarized content:
   ```python
   import hashlib
   
   # Simple in-memory cache
   summary_cache = {}
   
   async def run_text_summarizer_with_cache(text: str, preferences: Dict[str, str]) -> str:
       # Create a cache key based on text and preferences
       cache_key = hashlib.md5((text + str(preferences)).encode()).hexdigest()
       
       # Check if result is in cache
       if cache_key in summary_cache:
           return summary_cache[cache_key]
       
       # Run summarizer
       summary = await run_text_summarizer(text, preferences)
       
       # Cache the result
       summary_cache[cache_key] = summary
       
       return summary
   ```

### Content Length Issues

**Issue**: The LLM can't handle very long input texts due to token limits.

**Solutions**:
1. **Chunk the Input**:
   Split long texts into manageable chunks:
   ```python
   def chunk_text(text, max_chunk_size=8000):
       """Split text into chunks of approximately max_chunk_size characters."""
       # Split by paragraphs
       paragraphs = text.split('\n\n')
       
       chunks = []
       current_chunk = ""
       
       for paragraph in paragraphs:
           # If adding this paragraph would exceed the limit, start a new chunk
           if len(current_chunk) + len(paragraph) > max_chunk_size and current_chunk:
               chunks.append(current_chunk)
               current_chunk = paragraph
           else:
               if current_chunk:
                   current_chunk += "\n\n" + paragraph
               else:
                   current_chunk = paragraph
       
       # Add the last chunk if it's not empty
       if current_chunk:
           chunks.append(current_chunk)
           
       return chunks
   
   async def summarize_long_text(text, preferences):
       # Chunk the text
       chunks = chunk_text(text)
       
       # Summarize each chunk
       chunk_summaries = []
       for chunk in chunks:
           summary = await run_text_summarizer(chunk, preferences)
           chunk_summaries.append(summary)
       
       # If we have multiple chunks, summarize the combined summaries
       if len(chunk_summaries) > 1:
           combined_summaries = "\n\n".join(chunk_summaries)
           final_summary = await run_text_summarizer(combined_summaries, preferences)
           return final_summary
       else:
           return chunk_summaries[0]
   ```

2. **Use Models with Larger Context Windows**:
   Switch to models that can handle longer inputs:
   ```python
   # Models with larger context windows
   LARGE_CONTEXT_MODEL = "anthropic/claude-3-opus-20240229"  # 200K+ tokens
   ```

3. **Implement a Hierarchical Summarization**:
   Summarize sections individually, then summarize the summaries:
   ```python
   async def hierarchical_summarize(document, preferences):
       # Split document into sections (e.g., by headings)
       sections = split_into_sections(document)
       
       # Summarize each section
       section_summaries = []
       for section in sections:
           summary = await run_text_summarizer(section, preferences)
           section_summaries.append(summary)
       
       # Combine section summaries
       combined = "Document Section Summaries:\n\n" + "\n\n".join(section_summaries)
       
       # Generate final summary from section summaries
       final_summary = await run_text_summarizer(combined, preferences)
       
       return final_summary
   ```

## Debugging Techniques

### Logging Agent Events

Add logging to track the flow of events through the agent system:

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("text_summarizer.log"),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger("text_summarizer")

class TextSummarizerAgent(BaseAgent):
    # ... existing code ...
    
    async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        logger.info(f"Starting summarization process for text of length {len(ctx.session.state.get('input_text', ''))}")
        
        # ... existing code ...
        
        logger.info("Running content analyzer")
        async for event in self.content_analyzer.run_async(ctx):
            yield event
        
        logger.info(f"Content analysis complete: {ctx.session.state.get('content_analysis', {})}")
        
        # ... rest of the method with logging ...
```

### Using Callbacks

Implement callbacks to monitor the agent's execution:

```python
from google.adk.callbacks import CallbackContext, ToolContext

def before_agent_callback(callback_context: CallbackContext):
    agent_name = callback_context.agent.name
    print(f"Before agent: {agent_name}")
    return None

def after_agent_callback(callback_context: CallbackContext):
    agent_name = callback_context.agent.name
    print(f"After agent: {agent_name}")
    return None

def before_model_callback(callback_context: CallbackContext):
    model_name = callback_context.llm_request.model
    print(f"Before model call: {model_name}")
    print(f"Prompt: {callback_context.llm_request.contents}")
    return None

def after_model_callback(callback_context: CallbackContext):
    model_name = callback_context.llm_request.model
    print(f"After model call: {model_name}")
    print(f"Response: {callback_context.llm_response.text}")
    return None

# Add callbacks to the runner
runner = Runner(
    root_agent=text_summarizer,
    session_service=session_service,
    callbacks={
        "before_agent": before_agent_callback,
        "after_agent": after_agent_callback,
        "before_model": before_model_callback,
        "after_model": after_model_callback
    }
)
```

### Creating a Debug Mode

Add a debug mode to your summarizer function:

```python
async def run_text_summarizer(article_text: str, preferences: Dict[str, str], debug: bool = False) -> str:
    """
    Run the text summarizer with optional debug mode.
    
    Args:
        article_text: The text to summarize
        preferences: Summarization preferences
        debug: Whether to enable debug mode
        
    Returns:
        The final summary
    """
    # ... existing code ...
    
    # Add debug state if enabled
    if debug:
        state = {
            "input_text": article_text,
            "preferences": preferences,
            "debug_mode": True
        }
    else:
        state = {
            "input_text": article_text,
            "preferences": preferences
        }
    
    # Run the summarizer
    result = await runner.run_async(
        app_name="text_summarizer_app",
        user_id="user_123",
        session_id="session_456",
        input_content=Content(
            parts=[
                Part.from_text("Please summarize this article for me."),
                Part.from_text(article_text)
            ]
        ),
        state=state
    )
    
    # In debug mode, return the full state
    if debug:
        return {
            "final_summary": result.session.state.get("final_summary", ""),
            "content_analysis": result.session.state.get("content_analysis", {}),
            "extracted_content": result.session.state.get("extracted_content", {}),
            "initial_summary": result.session.state.get("initial_summary", ""),
            "refinement_result": result.session.state.get("refinement_result", {})
        }
    
    # Normal mode, just return the summary
    return result.session.state.get("final_summary", "")
```

## Best Practices

1. **Start Simple**:
   - Begin with a basic implementation and add complexity gradually
   - Test each component individually before integrating

2. **Test with Various Content Types**:
   - Use diverse test cases (short/long, simple/complex, different domains)
   - Create a test suite with expected outcomes

3. **Monitor Token Usage**:
   - Keep track of token usage to avoid unexpected costs
   - Implement token counting for large-scale applications

4. **Implement Error Handling**:
   - Add try/except blocks around API calls
   - Provide fallback mechanisms for when APIs fail

5. **Version Your Models**:
   - Specify exact model versions when possible
   - Document which model versions were tested

6. **Regular Maintenance**:
   - Update prompts as LLM capabilities evolve
   - Revisit performance periodically to identify improvements

## Conclusion

You now have a comprehensive understanding of how to build, customize, and troubleshoot the Text Summarizer Agent. By following the guidelines in this wiki, you should be able to create a robust summarization system that meets your specific needs.

Remember that LLM-based systems are continuously evolving, so stay updated with the latest developments in the field to keep improving your summarizer.

If you encounter issues not covered in this troubleshooting guide, consider:
- Checking the ADK documentation for updates
- Reviewing the LLM provider's documentation
- Joining community forums for the specific models you're using
- Experimenting with different prompts and configurations

Happy summarizing!
