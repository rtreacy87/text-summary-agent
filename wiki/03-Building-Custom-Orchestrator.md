# Building the Custom Orchestrator

In this section, we'll create a custom orchestrator agent that coordinates the specialized LLM agents we defined earlier. This orchestrator will manage the flow of data between agents and handle the iterative refinement process.

## Understanding Custom Agents in ADK

In ADK, a custom agent is created by inheriting from `BaseAgent` and implementing the `_run_async_impl` method. This method defines the orchestration logic and yields events from sub-agents.

The custom agent allows us to:
- Define a specific sequence of agent executions
- Implement conditional logic based on agent outputs
- Manage state between agent calls
- Control iterative processes

## Creating the TextSummarizerAgent Class

Add the following code to your `src/text_summarizer_agent.py` file, after the LLM agent definitions:

```python
class TextSummarizerAgent(BaseAgent):
    """
    A custom agent that orchestrates the text summarization process.
    
    This agent coordinates several specialized sub-agents to analyze, extract,
    summarize, refine, and format text content.
    """
    
    def __init__(
        self,
        content_analyzer: LlmAgent,
        extraction_agent: LlmAgent,
        summarization_agent: LlmAgent,
        refinement_agent: LlmAgent,
        format_agent: LlmAgent,
        name: str = "text_summarizer",
    ):
        """
        Initialize the TextSummarizerAgent.
        
        Args:
            content_analyzer: Agent for analyzing the content
            extraction_agent: Agent for extracting important information
            summarization_agent: Agent for generating the initial summary
            refinement_agent: Agent for refining the summary
            format_agent: Agent for formatting the final summary
            name: Name of the agent
        """
        self.content_analyzer = content_analyzer
        self.extraction_agent = extraction_agent
        self.summarization_agent = summarization_agent
        self.refinement_agent = refinement_agent
        self.format_agent = format_agent
        
        # Define the refinement loop for iterative improvement
        self.refinement_loop = LoopAgent(
            sub_agents=[refinement_agent],
            name="refinement_loop",
            max_iterations=3,
            condition_key="needs_refinement",
            condition_value="true"
        )
        
        super().__init__(
            name=name,
            sub_agents=[
                content_analyzer,
                extraction_agent,
                summarization_agent,
                self.refinement_loop,
                format_agent
            ]
        )
```

## Implementing the Orchestration Logic

Now, let's implement the `_run_async_impl` method that defines the orchestration logic:

```python
    async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        """
        Implement the custom orchestration logic for the text summarization process.
        
        Args:
            ctx: The invocation context
            
        Yields:
            Events from the sub-agents
        """
        # Get input text and preferences from state
        input_text = ctx.session.state.get("input_text", "")
        preferences = ctx.session.state.get("preferences", {})
        
        # Store input text in state for sub-agents
        ctx.session.state["input_text"] = input_text
        
        # 1. Run content analyzer
        async for event in self.content_analyzer.run_async(ctx):
            yield event
        
        # Get content analysis from state
        content_analysis = ctx.session.state.get("content_analysis", {})
        
        # 2. Run extraction agent
        async for event in self.extraction_agent.run_async(ctx):
            yield event
        
        # 3. Run summarization agent
        async for event in self.summarization_agent.run_async(ctx):
            yield event
        
        # 4. Run refinement loop
        # Initialize refinement condition
        ctx.session.state["needs_refinement"] = "true"
        
        # Store the initial summary as the current refined summary
        ctx.session.state["refined_summary"] = ctx.session.state.get("initial_summary", "")
        
        async for event in self.refinement_loop.run_async(ctx):
            # Update the refined summary after each iteration
            refinement_result = ctx.session.state.get("refinement_result", {})
            if isinstance(refinement_result, dict) and "refined_summary" in refinement_result:
                ctx.session.state["refined_summary"] = refinement_result["refined_summary"]
                ctx.session.state["needs_refinement"] = refinement_result.get("needs_refinement", "false")
            yield event
        
        # 5. Apply formatting based on user preferences
        ctx.session.state["format_preferences"] = preferences
        
        async for event in self.format_agent.run_async(ctx):
            yield event
        
        # Final summary is now in ctx.session.state["final_summary"]
```

## Understanding the Orchestration Flow

Let's break down the orchestration logic:

1. **Input Preparation**:
   - Get the input text and preferences from the session state
   - Store the input text in the state for sub-agents to access

2. **Content Analysis**:
   - Run the content analyzer agent to determine the text's characteristics
   - The result is stored in `content_analysis` in the session state

3. **Information Extraction**:
   - Run the extraction agent to identify the most important information
   - The result is stored in `extracted_content` in the session state

4. **Initial Summarization**:
   - Run the summarization agent to generate the initial summary
   - The result is stored in `initial_summary` in the session state

5. **Iterative Refinement**:
   - Initialize the refinement condition to `true`
   - Store the initial summary as the current refined summary
   - Run the refinement loop, which will execute the refinement agent up to 3 times
   - After each iteration, update the refined summary and check if further refinement is needed
   - The loop continues until either the refinement agent returns `needs_refinement: false` or the maximum iterations are reached

6. **Final Formatting**:
   - Store the user preferences for the format agent
   - Run the format agent to format the final summary according to the preferences
   - The result is stored in `final_summary` in the session state

## Creating a Helper Function

To make it easier to use the Text Summarizer Agent, let's add a helper function:

```python
async def run_text_summarizer(article_text: str, preferences: Dict[str, str]) -> str:
    """
    Run the text summarizer on the given article text with the specified preferences.
    
    Args:
        article_text: The text to summarize
        preferences: Summarization preferences (length, style, focus, audience)
        
    Returns:
        The final summary
    """
    # Initialize the Text Summarizer Agent
    text_summarizer = TextSummarizerAgent(
        content_analyzer=content_analyzer_agent,
        extraction_agent=extraction_agent,
        summarization_agent=summarization_agent,
        refinement_agent=refinement_agent,
        format_agent=format_agent
    )
    
    # Set up session service
    session_service = InMemorySessionService()
    
    # Create runner
    runner = Runner(root_agent=text_summarizer, session_service=session_service)
    
    # Run the summarizer with preferences
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
        state={
            "input_text": article_text,
            "preferences": preferences
        }
    )
    
    # Get the final summary
    final_summary = result.session.state.get("final_summary", "")
    return final_summary
```

## Understanding the LoopAgent

The `LoopAgent` is a special type of workflow agent in ADK that repeatedly executes its sub-agents until a condition is met or a maximum number of iterations is reached.

In our case:
- The `refinement_agent` is the sub-agent that will be executed repeatedly
- `max_iterations=3` limits the loop to at most 3 iterations
- `condition_key="needs_refinement"` specifies the key in the session state to check
- `condition_value="true"` specifies that the loop continues as long as the value is "true"

The loop will stop when either:
- The refinement agent sets `needs_refinement` to "false", indicating that no further refinement is needed
- The maximum number of iterations (3) is reached

## Next Steps

Now that we've built the custom orchestrator, we're ready to create an example script to run and test our Text Summarizer Agent.

Continue to [Running and Testing](04-Running-and-Testing.md).
