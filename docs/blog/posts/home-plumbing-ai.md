---
date:
  created: 2025-08-14
tags:
  - Airflow
  - Python
  - PydanticAI
  - AI
categories:
  - Personal Project
links:
  - External links:
    - Pydantic: https://github.com/pydantic/pydantic
    - Pydantic AI: https://ai.pydantic.dev/
    - Logfire: https://github.com/pydantic/logfire
---

# Plumbing AI into my personal data platform

This week I brought AI capabilities to my personal data platform - _home plumbing_. I was skeptical about using AI for financial categorization. Then I built it anyway.

I had a concrete use case in mind that I wanted to solve: Categorizing my transactions.

<!-- more -->

Only with categorized transactions can I perform analytics on the spending categories relevant to me and answer questions like "_how did my savings rate develop over the last months_" or "_in which months do I spend more eating out_?".

I chose to build an agentic workflow to tackle this challenge. By leveraging [Pydantic AI](https://ai.pydantic.dev/), a leading python framework for building production-grade GenAI applications, I was able to dive deep into important components of building AI-applications: Type safety and observability.

The following paragraphs offer a glimpse into my considerations and learnings dipping my toes into GenAI-powered data pipelines.

On a side note:
My updated data platform can be found in the `plumbing-airflow: v0.4.0` release and its accompanying [Docker image](https://github.com/jonathanschwarzhaupt/home-plumbing/pkgs/container/plumbing-airflow). For a primer on my personal data platform, check out my [previous post](./home-plumbing-my-own-data-platform.md)

## Why Not Traditional ML?

Using a GenAI approach to solving the challenge of categorizing my transactions was not my first thought. I was initially against it given the non-deterministic nature. Instead I wanted to test a model that I could explain. One where I know the features that are important, where I have control over how the model behaves. Very quickly though I was confronted with a myriad of complexity, that although far from being too difficult, would have required a lot of time. More time than I wanted to plan in for a first version of categorization. To put it succint: The ROI wasn't there for a v1.

See, to train a model, I'd require labeled data, or training data. I sat down for about an hour last Sunday to manually label around 150 transactions. I got through until last summer. Then I realized that since I moved in April of this year, I would need to label my transactions until one or two months after I had moved - to account for the new spending patterns and new shops that I now spend my money at in the new neighborhood. And immediately my motivation to build MLOps on top of my data platform vanished.

But my enthusiasm was not gone for too long. I started looking for alternatives. Pydantic AI stood out. I am already familiar with the `pydantic` library to validate data structures in python. It is an awesome library and thus I was curious to check out their new project `pydantic-ai`. It promises the same type safety developers know and love from their core package, but in the context of GenAI applications.

After following some quick getting started examples in my [lab](https://github.com/jonathanschwarzhaupt/lab) repository, I had my "WOW" moment. Wow - can it really be this simple? Can it really work that flawlessly? Looking back I would say yes and no. I was able to quickly set up an agentic workflow which works for all intents and purposes. But how can I observe what the model does? What API calls to the provider does the libary make ([see this really great blog post on this topic](https://hamel.dev/blog/posts/prompt/))? And how can I ensure that my agent categorizes the same transaction the same way? Needless to say, there is more depth, more strategies and tooling required to make this a "enterprise" agent. But for my simple purpose, it is a great proof-of-concept and most importantly learning experience.

## PydanticAI

### Type Safety

I think this is the key feature that I like the most about Pydantic AI: Type safety. From a developer point of view, it really is as simple as passing a pydantic model to the agent as a output_type. You can have high confidence that the model's output will be of that type. You do not need to tediously validate every expectation you have on your model's output by hand, pydantic handles it for you.

This type safety not only gives you confidence in your agent's output, but by extension your whole system. In data intensive applications, the flow of data is important. When data that crosses boundaries or domains is well defined, i.e. well-typed, handling these handovers can become seamless. IDEs pick up the fields, developers become confident.

Here a small example:

```py
from pydantic import Field, BaseModel
from typing import Literal
from pydanti_ai import Agent
from pydantic_ai.models.anthropic import AnthropicModel
from pydantic_ai.providers.anthropic import AnthropicProvider # (1)

# Define your model's output
class TransactionCategorization(BaseModel):
    improved_description: str = Field(
        description="The improved description of the transaction"
    )
    category: Literal["Groceries", "Travel", "Restaurant"]

# Define your model and pass it to an agent given instructions or a system prompt
model = AnthropicModel(
    "claude-3-5-sonnet-latest",
    provider=AnthropicProvider(api_key="your-api-key")
)
agent = Agent(
    model=model,
    system_prompt="""
    You are a very knowledgeable personal finance assistant.
    Your speciality lies in:
      * determining the category of a financial transactions
      * providing a concise description of a financial transaction better than the original
    given the details of the transactions.
    """,
    output_type=TransactionCategorization  # specify the type of desired output
)

# Use agent
...
```

1.  :octicons-package-dependents-16: Many different [providers](https://ai.pydantic.dev/models/) can be used.

### Observability

Pydantic AI also comes with great observability tool - pydantic logfire. Logfire is Pydantic's new commercial offering (with a generous perpetual free tier) around a complete observability platform build on open standards. This means that you can observe any of your applications with spans, metrics and traces. But Logfire is special in that it makes LLMs a first-class citizen in that observability.

Setting up a Logfire project can be easily done in their UI and setting up your code to use Logfire is equally simple, see the example below:

![Pydantic Logfire UI](../images/home-plumbing-ai-logfire-screenshot.png){ align=right width=800 }

```py
import logfire
import nest_asyncio

logfire.configure(
    send_to_logfire="if-token-present",  # only send to Logfire if the token exists in the env
    service_name="testing-env",  # name of the service name that shows in the UI
)
logfire.instrument_pydantic_ai()
logfire.instrument_httpx(capture_all=True) # extra package that observes all http requests to model providers
```

Using only these few lines, I have a complete observability platform for my GenAI data pipeline

I can see each operation the model took - most of the times it is 3. One for the agent, one for the model and finally the API call to the provider with the 'final_result' as the tool indicating the enforced result type. On one instance, a fourth step was made: The API request had an exception, so it was retried.

In addition, I can clearly see the model's instruction or system prompt as well as the user's input and the model's output. While I still do not know the weights of the model, i.e. what it deems important and what not (and I don't think anyone knows this), I have a clear view of what data goes into my system, the steps that are taken and lastly the output that was returned.

With the abilities to write custom SQL queries, filter by time range, etc. I am really content. I managed to integrate observability from the get-go, even though it is still a small proof-of-concept. But maybe, because it is a proof-of-concept, that observability is a non-negotiable.

### Integration into my data platform

Extending my core package to include such data processing capabilities was quite straight-forward, although I did spend some time on how I should structure this. I settled with creating a `processor` module that acts like a logical container just like `sources` and `destinations` already do. I also refactored parts of my `sources` module. I moved the creation of DDLs (or schema definitions) into a share module, so that processors would also have it available.

I then needed to create one new reader and one new writer function in my `destination` module. The reader was quite fun, actually! As I wanted to only categorize transactions that were not yet categorized, I needed to check non-existing IDs in the destination table compared to the source table. This led to me discovering some different scenarios for when a source or destination table does not exist or both do not exist.

Lastly, I wrote another DAG in my `plumbing_airflow` project to bring it all together. Originally I wanted 3 tasks: One for extracting the data, one for processing it and the last for saving the processed data. I encountered some issues with serialization of the Pydantic objects and since I wanted to I wanted to use Airflow's .expand() to parallelize transaction processing, but I found it simpler to put everything into a single task.

While not my favored design, being pragmatic counts and I leave my future self something to look forward to to tackle and improve.

## Outlook

It is clear to me that I am just starting to scratch the surface of integrating agentic workflows into data engineering pipelines.

After completing this small pipeline I am actually convinced that there is an added benefit to using agents, I generally favor explainability over black-boxes. But with observability integrated from the start combined with the type safety of the results, I am confident in the durability of my pipeline.

Now, I have lots of ideas of how to improve the categorization, i.e. the output generated by my categorization agent.

For one, there is room to improve the instructions. I want to experiment with different prompts, provide examples, etc.

Second, I have yet to explore the tools that agents can access. This includes generic ones like web-search, to customized ones, like running a SQL query. An obvious choice here is that the agent should first query the existing transactions if a similar transaction was already categorized. Or if the provided information e.g. on the seller is ambiguous, the agent could do a quick google search to obtain further information.

Third, I should periodically review and edit the categories assigned by the agent. I believe it is the interaction of human and agent that drives GenAI data platforms and thus their effectiveness.

Needless to say, I am excited for what's ahead!

## Links

Check out the GitHub repositories:

- [Home-Plumbing](https://github.com/jonathanschwarzhaupt/home-plumbing): My python package and Airflow code
- [Homelab](https://github.com/jonathanschwarzhaupt/homelab): My kubernetes cluster running my Airflow deployment

And reach out to me for any comments or questions, I am always happy to chat:

- [Linkedin](https://www.linkedin.com/in/jonathanschwarzhaupt/)
