# Week 2 Hands-On — Applied RAG Product Results (CS 5588)

## Product Overview
- Product name: MedScan Advisor
- Target users: Radiologists and oncologists reviewing brain MRI scans
- Core problem: 15-20 minutes wasted per case searching scattered protocol documents
- Why RAG: Generic chatbots could hallucinate clinical guidelines; RAG ensures verifiable sources

## Dataset Reality
- Source / owner: Hospital radiology dept + ACR/ASCO published guidelines
- Sensitivity: Regulated (HIPAA-adjacent, de-identified documents)
- Document types: Clinical practice guidelines, staging manuals, imaging protocols
- Expected scale in production: 200-500 documents initially, 2000+ at scale

## User Stories + Rubric
- U1: Radiologist looking up RANO criteria for glioblastoma response assessment
- U2: Oncologist verifying contrast contraindications for renal patients (HIGH STAKES)
- U3: Resident handling incidental findings with incomplete protocols (AMBIGUOUS)

## System Architecture
- Chunking: Semantic (paragraph-based, 1000 char max)
- Keyword retrieval: BM25Okapi
- Vector retrieval: all-MiniLM-L6-v2 + FAISS
- Hybrid α: 0.6 (balanced for medical terminology precision)
- Reranking: ms-marco-MiniLM-L-6-v2 cross-encoder
- LLM: flan-t5-base (optional, fallback to evidence summary)

## Results
| User Story | Method | Precision@5 | Recall@10 | Trust (1–5) | Confidence (1–5) |
|---|---|---:|---:|---:|---:|
| U1_normal | hybrid+rerank | 0.60 | 0.75 | 4 | 4 |
| U2_high_stakes | hybrid+rerank | 0.60 | 1.00 | 3 | 3 |
| U3_ambiguous | hybrid+rerank | 0.20 | 0.50 | 2 | 2 |

## Failure + Fix
- Failure: U3 retrieved general protocols instead of incidental findings doc
- Layer: Retrieval (both BM25 and vector)
- Consequence: Resident might miss escalation requirement
- Fix: Boosted terms for edge-case documents + confidence thresholding

## Evidence of Grounding
**Query**: "What are the RANO criteria for glioblastoma response?"

**Answer**: According to [Chunk 1], RANO criteria require bidimensional measurement of enhancing tumor on T1-weighted post-contrast MRI. [Chunk 2] specifies that measurable disease requires at least 10mm in two perpendicular diameters. Response categories include complete response (CR), partial response (PR), stable disease (SD), and progressive disease (PD) based on percentage change in tumor measurements [Chunk 1].
