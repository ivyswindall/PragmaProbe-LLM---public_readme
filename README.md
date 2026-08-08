# PragmaProbe-LLM: Mitigating Literal Traps in LLMs via End-to-End Context-Aware Retrieval and QLoRA Pipeline for Figurative Language Alignment

[![Dataset](https://shields.io)](https://huggingface.co)
[![Database](https://shields.io)](https://mysql.com)
[![Benchmark](https://shields.io)](https://huggingface.co)
## Table of Contents
- [Summary](#summary)
- [End-to-End Pipeline Architecture](#end-to-end-pipeline-architecture)
- [Pipeline Execution Steps](#pipeline-execution-steps)
- [Relational Database Schema (MySQL)](#relational-database-schema-mysql)
- [Future Scaling & Theoretical Extensibility](#future-scaling--theoretical-extensibility)

## Summary

**Core Objective & Linguistic Scope**

**PragmaProbe-LLM** is a lightweight prototype designed to demonstrate how linguistic (in patricular pragmatic) theories can systematically resolve contextual reasoning errors in pre-trained Large Language Models (LLMs). Specifically, the project targets the **ECONOMY IS WAR** conceptual metaphor framework within financial journalism and social media. 

Pre-trained models frequently suffer from "literal traps"—failing to distinguish between literal physical warfare and metaphorical economic competition, which flouts the Gricean Maxim of Quality (conversational implicature). They rely heavily on statistical patterns and frequency, their inability to interpret nascent or ambiguous metaphors reveals a critical limitation in capturing cross-domain semantic shifts. To overcome this, integrating a structured knowledge graph provides explicit mappings of conceptual domains, allowing the model to ground figurations in relational structures rather than relying solely on surface-level statistical correlations.

By filtering out obvious literal news and isolating the "Zone of Ambiguity" where the model struggles, the pipeline retrieves targeted contextual definitions from a relational MySQL knowledge graph. These structural mappings are converted into instruction-tuning pairs to realign the model's attention matrices via Low-Rank Adaptation (QLoRA). This proof-of-concept demonstrates that grounding NLP workflows in established linguistic theory creates efficient, small-scale alignment loops for downstream domain specialization.


**The PUB Benchmark and Framework assessment**

This framework has been developed to target the performance gaps highlighted in the Pragmatics Understanding Benchmark (PUB) (Sravanthi et a;. 2024). As documented in PUB Task 6 (Sarcasm and Metaphor Comprehension), models frequently suffer from Over-Correction Pattern Collapse and Generation Degeneration when processing non-literal text (a statistical finding that remains current as of 2026).

PragmaProbe-LLM offers a framework to mitigate these challenges by integrating a post-training phase to enhance the model's accuracy with non-literal language. This phase specifically enables relational reasoning based on structured domain knowledge. Through fine-tuning, the framework empowers the model to infer the figurative meaning of unseen concepts by applying learned cross-domain mappings to the surrounding textual context. While RAG serves to mitigate overgeneralization in large-scale deployments, this project instead relies on thorough data curation, utilizing the MIP (Metaphor Identification Procedure), to maintain domain specificity and manage resource constraints.

A manual analysis of selected sentenses was conducted to evaluate the initial capabilities of the framework. Comprehensive and generalized assessment of the model's metaphor interpretation will require future evaluation against established gold-standard datasets, such as the VU Amsterdam Metaphor Corpus.


---

## End-to-End Pipeline Architecture

```

[Hugging Face Data Ingestion] 
           │
           ▼
[Step 1: Regex Keyword Anchor] 
           │
           ▼
[Step 2: MIP Vector Distance Filter] ───(High Similarity/Literal)───> [Dropped / Logged]
           │
     (Zone of Ambiguity)
           ▼
[Step 3: MySQL Graph Augmentation] 
           │
           ▼
[Step 4: QLoRA Fine-Tuning Loop] ───> [Realigned Pragmatic Model Checkpoint]

```

## Pipeline Execution Steps

1. Ingestion & Rough Selection (Regex Phase)
   Programmatically stream financial headlines from the Hugging Face Reuters dataset. Run a rapid regular expression filter using a targeted warfare lexicon (e.g., "battle", "attack", and less lexicalized like: "bunkerization", "decapitation strike" or "drone swarming") to isolate potential candidates matching the 'ECONOMY IS WAR' conceptual framework.

2. Verification & Ambiguity Isolation (MIP-VU-inspired computational filter)
   Bypass blind graph mapping by validating candidates using a weak-supervision proxy for the contextual-versus-basic-meaning comparison central to MIP-VU. Compare each headline's sentence embedding against a literal baseline embedding. Filter out highly literal news entries, and capture only the high-entropy "Zone of Ambiguity" cases where the pre-trained model struggles to distinguish literal context from conversational implicature.

3. Pragmatic Knowledge Retrieval (MySQL Mapping)
   Route only the validated ambiguous headlines to MySQL database. Query the 'WarMetaphorGraph' relational table using the extracted keyword to pull the precise, structured economic interpretation (e.g., mapping "drone swarming" directly to "highly competitive marketing strategy"). 

4. Model Realignment (QLoRA Fine-Tuning)
   Format the ambiguous headlines alongside their retrieved MySQL graph definitions into structured instruction-tuning pairs. Execute a lightweight, parameter-efficient fine-tuning loop (QLoRA) targeting the attention matrices of a small base LLM, permanently teaching it to resolve these pragmatic boundary exceptions.

---

## Relational Database Schema (MySQL)
The MySQL schema maps source-domain warfare concepts to context-dependent economic interpretations through an indexed alignment table storing curated conceptual relationships.

```sql
CREATE TABLE WarMetaphorGraph (
    source_concept VARCHAR(50) NOT NULL,
    relationship VARCHAR(50) NOT NULL,
    target_business_meaning VARCHAR(255) NOT NULL,
    PRIMARY KEY (source_concept, target_business_meaning)
);
```

---

## Future Scaling & Theoretical Extensibility

### 1. Scaling Alignment: Transitioning from SFT to Reinforcement Learning
While this prototype relies on Supervised Fine-Tuning (SFT), the data isolated in the "Zone of Ambiguity" can naturally scale to Reinforcement Learning (RL) frameworks:
* **Preference Pair Generation**: The MIP-VU distance calculations can automatically compile contrastive pairs. A headline flagged as an ambiguous metaphor creates a "Chosen" slot (the correct pragmatic MySQL interpretation) and a "Rejected" slot (the literal trap interpretation).
* **Synthetic RL (DPO/RLHF)**: This structured pair format can be fed directly into Direct Preference Optimization (DPO) or used to train a Reward Model for RLHF. Instead of relying on manual human labeling, the pipeline acts as an automated, linguistically grounded data generator to teach the model how to make the correct contextual choice at scale.

### 2. Domain Scaling: Leveraging MetaNet and LCC Databases
The **ECONOMY IS WAR** schema serves as an introductory case study. To scale this approach across broader human discourse, the regular expression and embedding pipeline can be expanded to cover other foundational source-to-target mappings:
* **Multi-Domain Mapping**: By ingesting structural mappings from extensive repositories like the **Berkeley Conceptual Metaphor MetaNet** or the **Large Scale Conceptual Metaphor (LCC) Database**, the pipeline can target other vital frameworks (e.g., *POLITICS IS MEDICINE*, *TIME IS MONEY*, or *ARGUMENT IS WAR*).
* **Cross-Domain Specialization**: Replacing the specialized financial lexicon with these broader relational databases allows the core architecture to systematically clean and align models for political science, legal analysis, or medical communication fields.

