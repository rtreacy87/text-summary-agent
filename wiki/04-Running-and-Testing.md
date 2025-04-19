# Running and Testing

In this section, we'll create an example script to run and test our Text Summarizer Agent with different types of content.

## Creating the Example Script

Create a new file `src/example_usage.py` with the following content:

```python
"""
Example usage of the Text Summarizer Agent

This script demonstrates how to use the Text Summarizer Agent to summarize
different types of text content with various preferences.
"""

import asyncio
from text_summarizer_agent import TextSummarizerAgent, run_text_summarizer
from text_summarizer_agent import (
    content_analyzer_agent,
    extraction_agent,
    summarization_agent,
    refinement_agent,
    format_agent
)

# Sample texts for different content types
SAMPLE_ARTICLE = """
Artificial Intelligence (AI) has made significant strides in recent years, transforming various industries and aspects of daily life. From healthcare to finance, transportation to entertainment, AI technologies are being integrated into systems and processes, enhancing efficiency and enabling new capabilities.

In healthcare, AI algorithms are being used to analyze medical images, detect diseases, and predict patient outcomes. For example, deep learning models have shown promising results in identifying cancerous cells in pathology slides and detecting abnormalities in radiological images. AI is also being employed to accelerate drug discovery and development, potentially reducing the time and cost involved in bringing new medications to market.

The financial sector has embraced AI for fraud detection, risk assessment, and algorithmic trading. Machine learning models can analyze vast amounts of transaction data to identify suspicious patterns indicative of fraudulent activity. AI-powered robo-advisors are providing personalized investment advice to clients, democratizing access to financial planning services.

In transportation, autonomous vehicles represent one of the most visible applications of AI. Companies like Tesla, Waymo, and Cruise are developing self-driving cars that use computer vision, sensor fusion, and reinforcement learning to navigate roads safely. The potential benefits include reduced accidents, decreased traffic congestion, and increased mobility for those unable to drive.

Despite these advancements, AI faces several challenges and limitations. Bias in AI systems remains a significant issue, as models trained on biased data can perpetuate and amplify existing societal inequities. The "black box" nature of some AI algorithms makes it difficult to understand how they arrive at specific decisions, raising concerns about transparency and accountability.
"""

SAMPLE_CODE = """
def fibonacci(n):
    """Calculate the Fibonacci sequence up to n."""
    fib_sequence = [0, 1]
    
    # Generate Fibonacci sequence
    while len(fib_sequence) < n:
        fib_sequence.append(fib_sequence[-1] + fib_sequence[-2])
    
    return fib_sequence[:n]

def is_prime(num):
    """Check if a number is prime."""
    if num <= 1:
        return False
    if num <= 3:
        return True
    
    # Check if divisible by 2 or 3
    if num % 2 == 0 or num % 3 == 0:
        return False
    
    # Check divisibility by numbers of form 6k±1
    i = 5
    while i * i <= num:
        if num % i == 0 or num % (i + 2) == 0:
            return False
        i += 6
    
    return True

def find_primes_in_fibonacci(n):
    """Find prime numbers in the Fibonacci sequence up to n terms."""
    fib_sequence = fibonacci(n)
    prime_fibs = [num for num in fib_sequence if is_prime(num)]
    
    return prime_fibs
"""

SAMPLE_CONVERSATION = """
Alice: Hi team, I wanted to discuss the quarterly results and our plans for the next quarter.
Bob: Sounds good. How did we perform against our targets?
Alice: We exceeded our revenue target by 12%, reaching $3.2 million for the quarter. However, we fell short on new customer acquisition by about 8%.
Charlie: That's interesting. What do you think contributed to the higher revenue despite fewer new customers?
Alice: Great question. Our analysis shows that existing customers increased their spending by an average of 18%. The customer success team's upselling strategy seems to be working well.
Bob: That's excellent news for retention. What about our expenses?
Alice: Overall expenses were 5% under budget, primarily due to lower marketing costs and the delay in office expansion.
Charlie: So what are the key priorities for next quarter?
Alice: I propose we focus on three areas: First, improving our customer acquisition strategy to address the shortfall. Second, continuing the successful upselling approach. And third, launching the new product line we've been developing.
Bob: For the customer acquisition, I think we should revisit our digital marketing strategy. The current channels aren't performing as well as they used to.
Charlie: I agree. We should also consider expanding into new markets. The market research team has identified some promising opportunities in the Asia-Pacific region.
Alice: Both good points. Let's allocate additional budget to explore those new markets, and Bob, can your team prepare a revised marketing strategy by next week?
Bob: Yes, we can have that ready by Wednesday.
Alice: Perfect. For the new product launch, we're still on track for a mid-quarter release. The development team has completed most of the core features.
Charlie: What about pricing for the new product?
Alice: We're thinking of a tiered approach, starting at $49 per month for the basic version and $129 for the premium version with all features.
Bob: That sounds reasonable based on the competitor analysis we did.
Alice: Great. Let's reconvene next week to finalize these plans. Charlie, can you prepare a detailed budget for the Asia-Pacific expansion?
Charlie: Will do. I'll have it ready by Monday.
Alice: Excellent. Thanks everyone for your input.
"""

# Different preference configurations
PREFERENCES = {
    "article": {
        "length": "medium",
        "style": "narrative",
        "focus": "general",
        "audience": "general"
    },
    "code": {
        "length": "short",
        "style": "structured",
        "focus": "functionality",
        "audience": "technical"
    },
    "conversation": {
        "length": "medium",
        "style": "bullet_points",
        "focus": "key_points",
        "audience": "executive"
    }
}

async def main():
    """Run the text summarizer on different content types with various preferences."""
    print("=" * 80)
    print("TEXT SUMMARIZER AGENT DEMONSTRATION")
    print("=" * 80)
    
    # 1. Summarize an article
    print("\n1. ARTICLE SUMMARIZATION")
    print("-" * 80)
    article_summary = await run_text_summarizer(SAMPLE_ARTICLE, PREFERENCES["article"])
    print(f"Original Length: {len(SAMPLE_ARTICLE)} characters")
    print(f"Summary Length: {len(article_summary)} characters")
    print("\nSummary:")
    print(article_summary)
    
    # 2. Summarize code
    print("\n\n2. CODE SUMMARIZATION")
    print("-" * 80)
    code_summary = await run_text_summarizer(SAMPLE_CODE, PREFERENCES["code"])
    print(f"Original Length: {len(SAMPLE_CODE)} characters")
    print(f"Summary Length: {len(code_summary)} characters")
    print("\nSummary:")
    print(code_summary)
    
    # 3. Summarize a conversation
    print("\n\n3. CONVERSATION SUMMARIZATION")
    print("-" * 80)
    conversation_summary = await run_text_summarizer(SAMPLE_CONVERSATION, PREFERENCES["conversation"])
    print(f"Original Length: {len(SAMPLE_CONVERSATION)} characters")
    print(f"Summary Length: {len(conversation_summary)} characters")
    print("\nSummary:")
    print(conversation_summary)
    
    print("\n" + "=" * 80)
    print("DEMONSTRATION COMPLETE")
    print("=" * 80)

if __name__ == "__main__":
    asyncio.run(main())
```

## Running the Example

To run the example script:

```bash
cd text_summarizer_agent
python src/example_usage.py
```

This will demonstrate the agent's ability to summarize three different types of content:
1. An article about artificial intelligence
2. Python code for calculating Fibonacci numbers and prime numbers
3. A business conversation about quarterly results and planning

Each content type is summarized with different preferences:
- The article uses a medium-length narrative style for a general audience
- The code uses a short, structured style for a technical audience
- The conversation uses bullet points to highlight key points for an executive audience

## Understanding the Output

The output will show:
1. The original length of each content piece
2. The length of the generated summary
3. The summary itself

You should see that:
- The summaries are significantly shorter than the original content
- Each summary captures the key information from the original text
- The style and focus of each summary matches the specified preferences

## Testing with Your Own Content

You can easily modify the example script to test the summarizer with your own content:

```python
# Replace with your own text
MY_TEXT = """
[Your text here]
"""

# Define your preferences
MY_PREFERENCES = {
    "length": "medium",  # short, medium, long
    "style": "narrative",  # narrative, bullet_points, structured
    "focus": "general",  # general, specific aspects
    "audience": "general"  # technical, general, executive
}

async def test_my_content():
    summary = await run_text_summarizer(MY_TEXT, MY_PREFERENCES)
    print(f"Original Length: {len(MY_TEXT)} characters")
    print(f"Summary Length: {len(summary)} characters")
    print("\nSummary:")
    print(summary)

if __name__ == "__main__":
    asyncio.run(test_my_content())
```

## Troubleshooting Common Issues

If you encounter issues when running the example:

1. **API Key Errors**:
   - Make sure your API keys are correctly set as environment variables
   - Check that you have access to the specified models

2. **Model Availability**:
   - If a model is not available, try using a different model
   - Update the model constants in `text_summarizer_agent.py`

3. **Rate Limiting**:
   - If you hit rate limits, add delays between API calls or use a different model

4. **State Management Issues**:
   - If agents aren't receiving the correct input, check the input and output keys
   - Print the session state at different points to debug

## Next Steps

Now that you've successfully run and tested the Text Summarizer Agent, you can explore ways to customize and extend it to better suit your needs.

Continue to [Customization and Extensions](05-Customization-and-Extensions.md).
