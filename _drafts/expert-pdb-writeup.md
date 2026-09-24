---
layout: post
title: My first post
---

# Acquiring protein production data with agentic AI: what I've learned.

    For now, italics means that I am not happy with the wording and style. ***TODO*** edit further, and address/rephrase all italics, possibly borrowing phrases from other publications of mine or using LLMs.

    Carrots(^) means that a citations would be nice here. ***TODO*** insert citations as hyperlinks

## TL;DR
* It works
* Tool calling for final output
* Long context

## Background
* **TODO** talk about the Expert paper here
* This is an unfinished project. Mostly this is a long "note to self" in case I have an opportunity to get back to this. But maybe others will find some of this interesting too.

## Rationale.

Production of protein reagents is a critical early step in drug discovery^, structure determination^ and _many other life sciences applications_^. 
Modern recombinant protein production process typically works as follows.
The DNA sequence of the protein of interest (POI) is cloned into a vector - some biological entity that is capable of infecting cells, like a virus or a plasmid. 
The vectors are then transfected into the host cells that express the protein. 
There are a few commonly used host strains in biosciences: bacterial (_Escherichia coli (E. coli)_), yeast, insect or mammalian. Cell free expression systems have also started to gain traction recently. 
Once the protein is expressed, it needs to be purified, before it can be used for downstream experiments like measuring the potency of a drug candidate or the immune response to the antibody.

Protein production is an error prone process that often goes wrong: proteins either fail to express or cannot be purified. 
In nature, proteins are produced in living cells under tightly regulated conditions, often in tiny concentrations and bound to other molecules. 
Replicating these conditions in the lab, or finding other suitable conditions to recombinantly express and purify the POI, may be challenging or impossible. 
First and foremost, success of protein production depends on protein - intrinsic conditions, such as protein and DNA sequence, physico-chemichal properties of the construct, usage of tags, co-expressing partner proteins and chaperones. 
Other factors that affect the outcome include the choice of vector and host, cell growth time and temperature, induction and lysation methods, purification protocol details, composition of buffers, etc. 
 
Results and protocols of protein production experiments are regularly published in peer reviewed academic journals (typically within the context of downstream results, such as protein structures or drug discovery studies), however I am not aware of any bioinformatics resource or a database that _systematises_ this information, like the PDB^ does for protein structures or Uniprot^ for proteins in general.
Such a resource would be extremely valuable for academia and industry as a historic reference of protein production protocols (both successful and failed), for experiment reproducibility and as a source of training and evaluation data for AI models. 
This is a writeup of the first stage of curating such a dataset from academic publications with Large Language Models (LLMs).

## Deciphering the modern AI jargon

LLMs and AI agents are complex software systems. All the details of LLM training and inner workings of AI agents are too numerous to be covered by a blog post. Some are in fact trade secrets of the multi-billion dollar companies that develop these systems. The next few paragraphs explain the AI terminology that is used throughout the rest of this blog post.

Fundamentally, LLMs are tools that predict the next token (word or part of a word) based on the context: the input tokens plus the previously generated tokens. AI agents are applications that allow users to interact with LLMs and allow the LLMs to use tools. General purpose AI agents like ChatGPT, Codex or Gemini have become ubiquitous in recent years. This project is essentially about developing a custom AI assistant on an open source software stack, which is purpose-built for extracting data from academic publications and can be run in an automated setting.

The user interacts with AI agent by issuing instructions called "prompts". There are very loose restrictions on what can constitute a prompt: it can include text ("please check the following paragraph for grammar and style"), images ("update the attached photo to use natural light"), sound (the user talking to the AI assistant, rather than typing in the prompt), computer code ("write a tic-tac-toe game in JavaScript"), or, in out case, something like "extract the protein production protocols from the following academic publication". LLM ability to generate the next token can on its own be sufficient for handling certain simpler prompts. More complicated prompts, however, require the LLM to use other software systems, or "tools". For example, to update the photo, the agent might need to run image editing software with certain arguments. To verify that the part of the computer program that the agent has just written is syntactically correct and works, the agent might need to run the compiler and the unit tests. The agents often need to search the internet to respond to user queries with up-to-date information. Corporate chatbots extract data from the company databases.

Reasoning, or thinking LLMs were introduced by OpenAI and DeepSeek in late 2024 / early 2025. Reasoning LLMs split their output into two parts: the "chain of thought", or thinking part, when the LLM tries to explain to itself (and, to a lesser extent, the user), what needs to be done to produce the output, and the final output. The tools are typically called during the thinking phase.

LLMs are fundamentally stochastic systems, so running the same prompt through the same LLM multiple times is not guaranteed to yield identical chain of thought and results. For data extraction this is undesirable: we hope that the protein production protocols contained within a publication leave very little room for interpretation. 


## The software stack.

When I first embarked on this project, I was fortunate enough to be completing my final weeks at EBI^, so I could still access the state of the art GPU cluster there.
I did not have access to paid frontline LLMs, like ChatGPT (OpenAI) or Claude (Anthropic), so I decided to use the Qwen^ family of open-weights LLMs (Qwen3.8:27b, to be precise) via Ollama^.
I picked this combination mostly due to its simplicity, availability and ease of deployment.
Rapid development of LLM technology makes it imperative to have as much control as possible over model and runner deployment.
Having to rely on a centralised admin team for this would have halted the project. 
For example, within the short (few weeks) course of this project, a new version of Qwen (3.8) came out, that also triggered the runner upgrade: previous version of Ollama was not working properly with the lates Qwen. 
I have tried migrating the pipeline to vLLM, but it turned out that the (quite recent) version of CUDA drivers that was installed at EBI was not supported, despite the documentation saying that it was. 
I have not exhaustively evaluated all combinations of LLMs and runners. 
This might be a good subject for extending this research.

A lot of technical details discussed in the rest of this post are LLM specific, and would look slightly different if a different LLM or runner was used. However, the features that this project mostly relies on, like tool calling, thinking and output formatting, are supported by most modern LLMs.

## The dataset.

I have chosen to use the Protein Data Bank (PDB)^ as a starting point. Each PDB structure is typically accompanied by a publication, and that publication is referenced from the PDB entry. 
A protein must be expressed and purified before its structure can be determined, so these papers are quite likely to contain successful protein production protocols, either in the main text of the publication, or in the supplements, or among the cited literature. 
The PDB entry itself contains a lot of relevant information, like the protein sequence and information about the protein complex.
This is a "low hanging fruit" approach. 
Of course, more protein production protocols could be extracted from the publications that are not referenced from the PDB. 
**TODO** how many papers?
But _N_ papers is a good start.

The PDB provides a [flat file](https://ftp.ebi.ac.uk/pub/databases/msd/sifts/flatfiles/csv/pdb_pubmed.csv.gz.) that maps each PDB id to a PubMed id of an academic paper. 
NCBI PubMed^ ID, or PMID, identifies a record that is essentially a library card for the publication. 
It provides information like the title, journal, date, authors, sometimes the abstract, but not the full text. 
The majority of publications on life sciences have PubMed entries. 

Some publications are also deposited to PubMed Central Repository^ (PMC). 
This repository does provide the publication text, along with the supplements. 
The publications within it are referenced by PMCID.
Note, however, that the fact that the publication can be accessed through PMC does not automatically mean that anyone can do with it whatever they want (e.g. extract information from it). 
Some of the publications in PMC come under restrictive licenses.
Those that are covered by permissive open source licenses, like Creative Commons^ (CC), are flagged as `open pmc`. 
Recently, the PMC has migrated to [Amazon Web Services (AWS) S3 storage](https://pmc.ncbi.nlm.nih.gov/tools/pmcaws/) . The openly accessible S3 "bucket" is called [`pmc-oa-opendata`](https://pmc-oa-opendata.s3.amazonaws.com/README.txt).
NCBI provides a [web API](https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/) that maps PMIDs to PMC IDs. It accepts batches of up to 200 PMIDs at a time.

Each publication (version) corresponds to a folder with files in PMC. Typically there would be the main text, plus some supplements. The main text is typically formatted as a PDF, according to the style of the journal where the paper was published. Supplements come in a variety of formats: there could be a zip archive with all the supplements, MS Word files, PDFs, MS Excel tables, images, movies, etc. Most importantly, though, almost always there is a [JATS XML](https://jats.nlm.nih.gov/) file with the main publication text. The text is marked up to show the structure (sections, paragraphs, ...), the typographics (subscripts, superscripts, equations, ...), citations, front and back matter (list of authors and affiliations, references, ...) and other important information. Since JATS XML is almost always available for the publication, and for some other reasons described below, I decided to use these files as inputs to the LLMs.




## Plan

* ✅ Define 'LLMs' and outline that this project is about using LLMs to curate the data from the papers somewhere in the beginning.
* ✅ Setup: ollama, qwen 3.8. 
* ✅ Amazon S3 for PMC. PDB provides mapping of protein structures to PMC IDs.
* ✅ Use the latest model! Must be able to upgrade stuff on short notice.
* Context length
* ✅ Usage of JATS
* ✅ Legal concerns - although the text might be downloadable from PMC, that does not necessarily mean that the publication is a part of "Open PMC". 

* No `json` format for LLM response. No schema in the format.
* Thinking and streaming - good

### Tool calling
* Tool calling - good, but does not mix well with instructions about LLM response
* Lots of time can be wasted on tool calls that "validate" 
* "If there is a tool, the LLM will use it"
* Three options
  1. Giving a model very general tools, like ability to execute any Python code
        * This can result in lots of unnecessary tool executions. For example to validate the details of the publication that have already been checked by the authours and reviewers (e.g. sequence length, tags). 
        * Security is a concern. Possible mitigation - run within a docker/singularity container, with external network and file system access sandboxed.
  2. Try more specific tools, like "look up the putative protein id in this database"
  3. Another option: don't let the model run tools at all, just ask it to return structured response and do the toll calling yourself. Like "give me all putative protein ids that are mentioned in the paper", and then run own code to resolve and verify those ids.
* Possibly the sensible compromise is option 2
* Better roll your own tool with the help of coding agents than rely on some shitty "unofficial" MCP server

## Plan (continued)
* A practical trade-off about whether to use single stage or multiple stages for retrieving references
* Some numbers (?)
  * How long does it take to AI-annotate one publication?
  * How many publications are there?
* Examples of correctly parsed papers.
* Positives - clearly the tool parses the papers quicker than a human, and understands them better than anyone except a very highly qualified scientist
  * For example, it made good distinction between constructs that have been expressed and purified, and other constructs of the same proteins that were also expressed but used in cell based assays.
* Async python
* ✅ Mention EBI
* Schema in the prompt
* Mention Expert, if published
* How to verify this data?
  * AI makes mistakes, but so do humans
  * Have another model that acts as a validator
  * Engage with original publication authors.
  * Distinguish entries reviewed by humans from purely AI-generated and reviewed entries.
* ✅ Next steps: vLLM, others?

## Technical tips

* PDB to PMID mapping: https://ftp.ebi.ac.uk/pub/databases/msd/sifts/flatfiles/csv/pdb_pubmed.csv.gz. There are other mirrors.
* NCBI REST API to resolve PMIDs to PMC IDs: https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/. It accepts batches of up to 200 comma-delimited ids at a time.
* PMC on AWS S3: https://pmc.ncbi.nlm.nih.gov/tools/pmcaws/; https://pmc-oa-opendata.s3.amazonaws.com/README.txt
  * Python library for working with S3: `pip install boto3`

