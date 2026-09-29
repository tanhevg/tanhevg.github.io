---
layout: post
title: Expert PDB writeup draft
---

# Expert PDB writeup draft
LLMs and AI agents are complex software systems. All the details of LLM training and inner workings of AI agents are too numerous to be covered by a blog post. Some are in fact trade secrets of the multi-billion dollar companies that develop these systems. The next few paragraphs explain the AI terminology that is used throughout the rest of this blog post.

Fundamentally, LLMs are tools that predict the next token (word or part of a word) based on the context: the input tokens plus the previously generated tokens. AI agents are applications that allow users to interact with LLMs and allow the LLMs to use tools. General purpose AI agents like ChatGPT, Codex or Gemini have become ubiquitous in recent years. This project is essentially about developing a custom AI assistant on an open source software stack, which is purpose-built for extracting data from academic publications and can be run in an automated setting.

The user interacts with AI agent by issuing instructions called "prompts". There are very loose restrictions on what can constitute a prompt: it can include text ("please check the following paragraph for grammar and style"), images ("update the attached photo to use natural light"), sound (the user talking to the AI assistant, rather than typing in the prompt), computer code ("write a tic-tac-toe game in JavaScript"), or, in out case, something like "extract the protein production protocols from the following academic publication". LLM ability to generate the next token can on its own be sufficient for handling certain simpler prompts. More complicated prompts, however, require the LLM to use other software systems, or "tools". For example, to update the photo, the agent might need to run image editing software with certain arguments. To verify that the part of the computer program that the agent has just written is syntactically correct and works, the agent might need to run the compiler and the unit tests. The agents often need to search the internet to respond to user queries with up-to-date information. Corporate chatbots extract data from the company databases.

Reasoning, or thinking LLMs were introduced by OpenAI and DeepSeek in late 2024 / early 2025. Reasoning LLMs split their output into two parts: the "chain of thought", or thinking part, when the LLM tries to explain to itself (and, to a lesser extent, the user), what needs to be done to produce the output, and the final output. The tools are typically called during the thinking phase.

LLMs are fundamentally stochastic systems, so running the same prompt through the same LLM multiple times is not guaranteed to yield identical chain of thought and results. For data extraction this is undesirable: we hope that the protein production protocols contained within a publication leave very little room for interpretation. 

## Plan

* ✅ Define 'LLMs' and outline that this project is about using LLMs to curate the data from the papers somewhere in the beginning.
* ✅ Setup: ollama, qwen 3.8. 
* ✅ Amazon S3 for PMC. PDB provides mapping of protein structures to PMC IDs.
* ✅ Use the latest model! Must be able to upgrade stuff on short notice.
* ✅ Context length
* ✅ Usage of JATS
* ✅ Legal concerns - although the text might be downloadable from PMC, that does not necessarily mean that the publication is a part of "Open PMC". 

* ✅ No `json` format for LLM response. No schema in the format.
* ✅ Thinking and streaming - good

### Tool calling
* ✅ Tool calling - good, but does not mix well with instructions about LLM response
* ✅ Lots of time can be wasted on tool calls that "validate" 
* ✅ "If there is a tool, the LLM will use it"
* ✅ Three options
  1. Giving a model very general tools, like ability to execute any Python code
        * This can result in lots of unnecessary tool executions. For example to validate the details of the publication that have already been checked by the authours and reviewers (e.g. sequence length, tags). 
        * Security is a concern. Possible mitigation - run within a docker/singularity container, with external network and file system access sandboxed.
  2. Try more specific tools, like "look up the putative protein id in this database"
  3. Another option: don't let the model run tools at all, just ask it to return structured response and do the toll calling yourself. Like "give me all putative protein ids that are mentioned in the paper", and then run own code to resolve and verify those ids.
* ✅ Possibly the sensible compromise is option 2
* ✅ Better roll your own tool with the help of coding agents than rely on some shitty "unofficial" MCP server

## Plan (continued)
* ✅ A practical trade-off about whether to use single stage or multiple stages for retrieving references
* Some numbers (?)
  * How long does it take to AI-annotate one publication?
  * How many publications are there?
* Examples of correctly parsed papers.
* Positives - clearly the tool parses the papers quicker than a human, and understands them better than anyone except a very highly qualified scientist
  * For example, it made good distinction between constructs that have been expressed and purified, and other constructs of the same proteins that were also expressed but used in cell based assays.
* ✅ Async python
* ✅ Mention EBI
* ✅ Schema in the prompt
* ✅ Mention Expert, if published
* How to verify this data?
  * AI makes mistakes, but so do humans
  * Have another model that acts as a validator
  * Engage with original publication authors.
  * Distinguish entries reviewed by humans from purely AI-generated and reviewed entries.
* ✅ Next steps: vLLM, others?
* ✅ Example of protein production protocol

## Technical tips

* PDB to PMID mapping: https://ftp.ebi.ac.uk/pub/databases/msd/sifts/flatfiles/csv/pdb_pubmed.csv.gz. There are other mirrors.
* NCBI REST API to resolve PMIDs to PMC IDs: https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/. It accepts batches of up to 200 comma-delimited ids at a time.
* PMC on AWS S3: https://pmc.ncbi.nlm.nih.gov/tools/pmcaws/; https://pmc-oa-opendata.s3.amazonaws.com/README.txt
  * Python library for working with S3: `pip install boto3`
