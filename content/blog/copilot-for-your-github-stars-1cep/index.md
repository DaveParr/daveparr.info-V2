+++
title = "Copilot for your GitHub stars"
description = "An experiment in large language models, vector stores and cli tools"
date = 2023-11-19
[taxonomies]
tags = ["python", "langchain", "gpt4all", "chroma", "typer", "github", "llm", "vectorstore", "rag"]
[extra]
toc = true
+++

How do you use your GitHub stars? 

I'd guess if you've been programming for a few years you've probably hit the star button at the top of a few of your favourite repos. I know some people I follow have done it _thousands_ of times. Do you go back to them though? Do you review them for inspiration for your next project or go to them when you're stuck on a partictular problem?

## Inspiration

I've always assumed I would use them but I never have. I found myself doing some research recently into how to build software that uses LLMs, with the deliberate goal of building an as yet undefined side-project. I wanted to build something I hadn't built before, something that was hopefully a little original, and maybe even **useful**! So yet again I was starring repos like LangChain and Chroma, swearing this time would be different. 

As I was running through blog posts and diligently smashing the star buttons I realised that I had just hit on exactly what I wanted to try. I wanted to bring my GitHub stars right into my editor. I wanted to be able to have them next to me as I was working and get a sensible set of suggestions on what might be useful for my needs at that moment, and I had just been starring the exact repos that could make this happen!

## The original idea

> Use a dataset of your personal stars to inform retrival augmented generation for a question and answer large language model deployed in a command line interface

I thought this would be useful for a few reasons:
* By having it in the CLI its available right in my editor, and to every project.
* By having a set of your personal stars the suggestions are already curated by your interests and preferences. Mine are all Python and R librarys, wierd data bases and charting libraries. Yours might mostly be Ruby gems, or web frameworks, or tools for embedded systems.
* By using a large language model the tool *might* be more capable of understanding the intentention of your goals, for instance the query "Suggest how to build a web app" might be able to infer that you'd likely want a front end component, a backend component and a data storage component, and might even deal with servers and deployment.
* By using large language models the tool *might* be more capable of semantic search rather than keyword matching which suits this problem as there is no strong standard on how a library describes it self through it's topics, description and documentation.

### Semantic vs keyword

#### Keyword
A keyword search looks for the exact letters in a string, or potentially a partial match. As an example the query `"Data Science"` would find things that exactly matched the charcters in the string `"Data Science"` and maybe also `["Data", "Science", "DS"]`.

#### Semantic 
A semantic search looks for the _conceptual_ similarity between things, so in this context `"Data Science"` would find things that matched the vector embedding of `"Data Science"` as well as maybe also the vector embeddings that are associated with `["Machine Learning", "Artificial Intelligence"]`

And so I ran `poetry new starpilot`:

[starpilot](https://github.com/DaveParr/starpilot)

## Why retrival augmented generation matters

Retrieval Augmented Generation (RAG) is a technique used by large language models to cope with some of the limitations inherent in what are also sometimes referred to as 'Foundational' models. 

When a model like GPT3 is trained, it is fed large amounts of textual data written by humans. These get translated into 'weights' in a nueral net. To overly simplify, these weights tell the model what the next most likely text is that follows the text it has already been shown. 
 
However, these models don't know much about what has happened recently, what other programming resources really exist rather than what just sounds like it should exist, or where to exactly get a specific repo or webpage.

Retrieval augmented generation solves this by allowing you to feed the large language model with known real, up to date and relevant information. 

## Vectorstores

A type of data base called a vectorstore is commonly used for this because they are deliberately optimised towards a similarity search use case. They achieve this in a few ways:

* Vectorstores store what you pass them as a 'vector embedding'. A vector embedding takes data (like text or images) and converts them to a list like representation of numbers.
* Vectorstores keep similar vector embeddings close together in memory. This means that they are as fast as possible at returning lots of documents that have similar semantic meaning, because they are all clustered together.
* Vectorstores have APIs that are specifically designed for these use cases, with querying methods that lean towards semantic searches more than sql queries, and loading techniques that integrate tightly into other systems that generate these vector embeddings from large language models.

## Designing a system

With this set of goals and new knowledge I got to work working out which puzzle pieces I needed and how to fit them together. This time I did go through my stars (and a few other things), though maybe this is for the last time!

I figured I could get started using 4 main open source repos. [My first commit to my pyproject.toml](https://github.com/DaveParr/starpilot/commit/4846bcd43b598d0c5d6fb408fbe2d622076b999f#diff-50c86b7ed8ac2cf95bd48334961bf0530cdc77b5a56f852c5c61b89d735fd711) used these projects:

### Typer

[tiangolo/typer](https://github.com/tiangolo/typer)

`typer` is a pretty trendy framework for building CLI tools in python right now. It embraces typing, uses function decorators to magically turn your functions into CLI commands, and has relatively clear documention.

I chose `typer` specifically because:
* I wanted to see what the hype was about
* I think typing helps write better code
* I found the documentation really helpful to get started easily

[langchain-ai/langchain](https://github.com/langchain-ai/langchain)

`langchain` is the most mature and well embraced large language model orchestration framework. Langchain itself doesn't supply you with any specific llm or vector store or embedding approach. Instead it is deliberately 'vendor agnostic'. It provides a common set of APIs and abstractions across a staggering number of vector data bases, large language models and embedding engines. 

I chose `langchain` because:
* It is the most established tool in a brand new space
* I wasn't really sure which suppliers of vectorstores and large languge models made the most sense for my use case
* I found the documentation really helpful to get started

### Chroma

[chroma-core/chroma](https://github.com/chroma-core/chroma)

`chroma` is a vectorstore that has great support from Langchain. There are many others as well but Chroma won out at this stage because:

* I can run `chroma` as an 'embedded' data store, e.g. it runs locally on the users machine
* `chroma` was the most often used vectorstore in the Langchain docs for RAG tasks
* It was trivially easy to set up to the point at which I was convinced reading the tutorials that they had to have made a mistake

### GPT4All

[nomic-ai/gpt4all](https://github.com/nomic-ai/gpt4all)

`gpt4all` provides a set of LLM models and embedding engines that are also well supported by Langchain. `gpt4all` was appealing because:

* It runs locally on 'normal' machines
* It seems well supported and maintained
* It is open about what data it was trained on and what data it will use to train on

### PyGithub

[Pygithub/PyGithub](https://github.com/PyGithub/PyGithub)

Soon after this I realised that `pygithub` would be an easy way to go to GitHub to get the information I needed and bring it back into `starpilot` to load into the vectorstore. I had initially thought I might be able to use the [GitHub Document Loader](https://python.langchain.com/docs/integrations/document_loaders/github) built into `langchain`, though once I sat down to really work it out I realised that this doesn't give access to a users stars, so I needed an alternative.

## The other way to build

There were alternatives in all these choices. I think these are all totally viable parts to build effectively the same system:

### Click

[pallets/click](https://github.com/pallets/click)

I actually am using `click`, sort of. `typer` is built ontop of `click`, but to be honest I didn't really know that before I'd mostly decided. `click` looks like a really great project, but it wasn't _as_ clear how to get started.

### Llama Index

[run-llama/llama_index](https://github.com/run-llama/llama_index)

`llama_index` is probably a great project, but I only found it late in my thinking on this project. If I start a different project it's suitable for any time soon I'm definately going to try it out as a comparison.

### Faiss

[facebookresearch/faiss](https://github.com/facebookresearch/faiss)

I'd used `faiss` in a tutorial on vectorstores before. It didn't strike me as hugely intuitive to use or as simple to set up (it's recommended installation path is via conda). I also don't particularly like Facebook so I'm happy to use an alternative. 

### OpenAI

[openai/openai-python](https://github.com/openai/openai-python)

I'd used `openai` for a handful of tutorials and notebook experiments already and been very happy with it. However for a project like this I wasn't really sure what the operational costs would be, and if they would be worth it for the benefit the tool provides. That combined with the requirement to have network connectivity while using the tool pushed me towards experimenting with alternatives. Luckily with `langchain` I should be able to provide it as an optional backend in the future?

## What state is `starpilot` now?

"actively developed", "v0.1.0", "untested" and "it runs on my machine" are good descriptions of the project right now.

I've spent a few evenings this month on it, and see myself at least spending a few more on it next month. The API is getting breaking changes almost everytime I open the project. It's got 0 real tests. It should get some soon though. It requires a few manual installation steps that are documented in `README.md` but haven't yet even been attempted on another machine other than the one I'm on right now.

It also doesn't yet achieve exactly what I want it to, but I see no reason yet that it can't with some more development time.

### Current features

#### `starpilot read MyCoolUserName` 

This will connect to Github and read the starred repos of the user `MyCoolUserName`. Then it will go to each of those repos and get the topics and descriptions (and optionally the readmes) and load these into `chroma` which is persisted on the local hard drive.

#### `starpilot shoot "insert topic here"`

This will spin up the `chroma` database and perform a semantic similarity search on the string given in the command, then return the documents that seem to be the most relevant.

#### `starpilot fortuneteller "Insert a question here"`

This will perform the exact same search as the `shoot` command, but then spin up a large language model and pass the results into the large language model for processing. It then returns the documents it found as well as the response from the LLM

## So....

That's where this project is at. I've learnt a tonne about the available tools and relevant techniques in this space already, which was really the main goal of starting to begin with!

That said the progress I've made so far only makes me more curious about what else can be done with this and what else can be solved towards the vision of "Making your GitHub stars more valuable in your daily coding". Here's some ideas that I've found exciting while getting my hands dirty that might show up in the future. These are along with the obvious things like any testing at all, a simpler way to set up the project on your machine, better error handling, a more sensible way to update the vectorstore than drop everything and rebuild each time, etc. 

* Inspecting the current projects description (both it's loose goals as well as more specific things like what packages it already uses) so that things that are already used aren't suggested and are instead used to inform the response.
* Dynamically creating a GitHub list of similar starred repos for your user (though that would probably rely on [this suggestion to extend the GitHub API](https://github.com/orgs/community/discussions/8293?sort=new)) so that you naturally have some ways of saving and sharing your starred repos that solve a specific problem between sessions in your terminal
* Building starpilot into a research agent that can perform actions such as installing the selected suggestion into the current project or be sent to GitHub to find new projects that solve the current goals that you haven't starred yet

## What do you think?

Does this sound like something intersting to you, maybe even something useful? Did this just spark inspiration in you for a new project? Does this actually already exist somewhere and I'm just being an idiot? Let me know :)