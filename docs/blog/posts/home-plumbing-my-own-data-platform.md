---
date:
  created: 2025-08-08
tags:
  - Airflow
  - K8s
  - Python
categories:
  - Personal Project
links:
  - External links:
    - Home-Plumbing: https://github.com/jonathanschwarzhaupt/home-plumbing
    - Homelab: https://github.com/jonathanschwarzhaupt/homelab
    - Linkedin: https://www.linkedin.com/in/jonathanschwarzhaupt/
---

# Home-Plumbing my personal data platform

This last weekend I achieved the milestone I had my eyes on since May 2025! I finally deployed Airflow 3 (a workflow orchestration tool) in my own Kubernetes cluster running on K3s (lightweight Kubernetes) and Flux (GitOps automation). <!-- more --> The Airflow deployment runs my custom [Airflow image](https://github.com/jonathanschwarzhaupt/home-plumbing/pkgs/container/plumbing-airflow) and my DAGs orchestrate the code from my own [`home-plumbing` python package](https://github.com/jonathanschwarzhaupt/home-plumbing).

This deployment marks the "go live" of my personal data platform which I jokingly named *home plumbing* - a nod toward the messy plumbing involved in the day-to-day work I face as a data engineer.

I also learned a lot plumbing away mornings and evenings. From new python libraries like `httpx`, `pydantic`and `pydantic-settings` that I plan to integrate into my daily work, to DuckDB's integrations with SQLite, [Turso's](https://turso.tech/) embedded SQLite databases to the vast world of inner workings and configurations of Airflow.

My `home-plumbing` project finally exists. I have achieved a milestone I am really proud of, yet there is much to improve and extend which I am really excited about! It gives me a purpose to direct my learning towards and apply the skills I pick up in my day-to-day job but also on the side.

In the following sections I will lay out my motivation for creating my `home-plumbing` project, the journey I have had so far and a brief glimpse into the outlook I plan.

## Motivation

The original idea came from my own frustration. My bank, comdirect, does not offer spending analytics in their mobile app like N26 or Revolut do. Since I use comdirect for my day-to-day I have pretty much been "blind" on my spending patterns for years. I tried solutions like Finanzguru but felt unease with the thought of someone else storing such personal information. And I also did not want to pay. So, I will simply build it myself, I thought.

This personal need of mine also gave me purpose in my learning: I had a real-world use case to apply my learnings to. In the years prior, even though I studied a lot in my own time, the learnings were isolated, seemingly had an end and did not evolve, improve, or grow. Take for example the [backend API project](https://github.com/jonathanschwarzhaupt/go-demo-pokemon-ingestion) I created for a hackathon I organized for my team at work: I spent half a year learning Go, from the ground up. I created this API complete with logging middleware and data validation. For myself I wrote a small recipe manager that added user authentication and repsonded with HTML components instead of JSON data. But in both projects I did not feel the purpose, they did not evolve.

With `home-plumbing` I build myself a foundation that I can extend, break, evolve. It enables me to add further data sources and destinations, deep-dive into Airflow's configuration and break services so that Flux automatically reconciles them, testing the recovery and backup configurations of my kubernetes cluster. I wrote my core code as a python package so that it stays independent of any orchestrator code giving me the flexibility to swap orchestrators without rewriting business logic. A pattern I've seen save months of work in production systems. In the future I could test out [Dagster](https://dagster.io/) or [Prefect](https://www.prefect.io/) Needless to say, I am really excited about the journey ahead.

## Journey

### The API Challenge

I do not remember when exactly, but it was some months ago when I stumbled upon comdirect's API documentation. It was quite the read. Even though integrating APIs into a data platform is my "bread and butter" at work, reading comdirect's documentation was... interesting. Looking back, it provided most of the relevant information. But how it was written made the process of using the API painful. It took me the better part of a month to figure out their somewhat custom OAuth2 authentication process given the few clear examples, or even clear explanations. But hey, it was a nice challenge and I am not blaming anyone here!

### From Scripts to Package

What started as cells in a Jupyter Notebook turned into functions in a script. I decided to give the `httpx` library a go - thinking it would be a drop-in replacement for `requests` (it wasn't). So back to testing each API call separately. But once the auth flow was set, I had great motivation to write the API calls to get actual data. The moment I first saw my data was such a rewarding moment!

Then it was onto marshaling that data into some structure. Pandas dataframes are an obvious choice. The library is super popular and well understood by pretty much all python developers. What I tend dislike when using it though is that I have no idea of the schema of the data, especially when I review or debug code from a colleague. I wanted to do it differently this time and opted for using `pydantic`.

### Choosing the Right Storage

With typed and validated data models in place, it was time to think about the destination to save the data to. I am a big fan of [duckdb](https://duckdb.org/) and wanted to test out it's bindings to SQLite to not only run analytical queries, but also simpilify writing to SQLite tables. This resulted in the sqlite destination module. I could insert directly into a SQLite table from a pandas dataframe replacing the use of a staging table. But it had drawbacks. I knew I needed to mount a volume to my worker pod(s) - but with the `local path` storage class of my K3s cluster, I could not share that volume with services outside Airflow's namespace. I did not want to figure that out before getting a first version of my platform live, so I added a second destination - [Turso](https://turso.tech/). I have been following their developments for quite a while and opted to use the more mature `libsql` package. Now I can use SQLite like I normally would, but when the connection is instantiated with remote credentials, I can `.sync()` the changes to a remote database.

### Deployment Victory

Finally I could write my Airflow DAGs. Since I use Airflow daily at work, that part was quiet straight-forward. I chose Airflow because I wanted to try out it's version 3 which has features I am really excited about, such as its revamped architecture, but also features like 'assets' which bring a different approach to cross-DAG dependencies. Something new I learned though was the package structure of the `dags/` folder which allows for sharing configurations between DAGs. Lastly, I created my custom Airflow image by installing my `home-plumbing` package and baking the DAGs into the image. Given the DAGs don't change frequently, it was the recommended, but also pragmatic approach.

Lastly, I could sit down and figure out how to use and configure Airflow's official helm chart. I used a local K3s cluster running on my laptop using Rancher Desktop to test out different `values.yaml`. This was a fun, but also frustrating process but it allowed me to dig deeper and deeper. When I finally had a configuration I was happy with, I could add the repository and release to my [homelab](https://github.com/jonathanschwarzhaupt/homelab) repository running Flux. With ingress configured, I could finally access the Airflow UI from my own domain without port-forwarding to the service.

The journey was finally complete. The first milestone reached. Months of work complete.

## Outlook

I have plenty of ideas, features to implement and configurations to explore.

Now that I have my bank account balance and transactions, I need to categorize them. Of course, this can be achieved through a simple rule-based system, and that might be the appropriate and pragmatic approach to take. But why not use that as an excuse to dive deep into Langchain and Langraph to build an agentic workflow into my data pipelines? There is also a `airflow-ai` sdk made my astronomer that provides convenient abstractions for orchestrating LLM calls providing structured output.

Then there is the whole frontend-part of my data platform. I need a visual interface to understand my spending over time. An interesting candidate to try out is [evidence](https://evidence.dev/). But also here, why should I not use this as an excuse to deepen my experience in JavaScript/ TypeScript and build a custom frontend with e.g. React?

And of course, my deployment and monitoring setup needs improvements. My custom Airflow image is consuming too much memory for my liking, which I want to dig into. I also want to leverage the statsd endpoint exposed by my Airflow deployment to build a proper monitoring and alerting system on prometheus and grafana.

I don't think I will get bored any time soon. But I am more excited than ever for what is to come.

## Links

Check out the GitHub repositories:

* [Home-Plumbing](https://github.com/jonathanschwarzhaupt/home-plumbing): My python package and Airflow code
* [Homelab](https://github.com/jonathanschwarzhaupt/homelab): My kubernetes cluster running my Airflow deployment

And reach out to me for any comments or questions, I am always happy to chat:

* [Linkedin](https://www.linkedin.com/in/jonathanschwarzhaupt/)
