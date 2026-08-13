# Vole Release Artifacts

This directory contains the Vole search-agent artifacts for the TREC 2024 RAG
Track `man86` evaluation slice.

Vole is an LLM-based search agent that issues Boolean queries to a Shortest
Substring Ranking (SSR) retrieval server, judges the returned results, and
continues searching under a bounded model-call budget.

## Contents

- `vole.py`: the Vole search-agent driver.
- `instructions.txt`: the model-facing search instructions used by `vole.py`.
- `rag2024.man86.topics`: the 86-topic evaluation slice.
- `Results/`: saved Vole JSONL logs for the 86 topics.
- `Qrels/`: base qrels, local UMBRELA hole-filling qrels, and the combined
  qrels used for evaluation.
- `UMBRELA.md`: the recipe for generating relevance judgments with UMBRELA.
- `vole_run.py`: converts Vole logs to a TREC-format run.
- `vole.man86.run`: the Vole run derived from the saved logs.
- `ndcg.man86.tsv`: the full version of Figure 3 from the paper, including
  runs from all groups.
- `requirements.txt`: minimal Python dependency list for running `vole.py`.

## SSR Server

The Shortest Substring Ranking algorithm is implemented by Cottontail:

https://github.com/claclark/Cottontail

A simple (but relatively slow) way to build the MS MARCO V2.1
segmented-document index is:

```bash
apps/jsonl --simple msmarco_v2.1_doc_segmented/*.json.gz
```

This runs the single-thread builder and creates a static index, called a
burrow, with the default name `json.burrow`.

Run the SSR server with:

```bash
apps/ssr-server --fields title:,headings:,segment: : : docid: json.burrow
```

The server prints a port number. Pass that port as the first command-line
argument to `vole.py`.

## Running Vole

Vole requires Python 3.9+ and the OpenAI Python package:

```bash
python -m pip install -r requirements.txt
export OPENAI_API_KEY=...
```

Optionally choose a model:

```bash
export OPENAI_MODEL=gpt-5.5
```

Run Vole against an SSR server port and the topic TSV:

```bash
python vole.py PORT rag2024.man86.topics
```

During a run, `vole.py` writes JSONL logs under `results/`.

## Evaluation

The saved Vole run is `vole.man86.run`. To evaluate it with `trec_eval` against
the combined qrels:

```bash
trec_eval -c -m ndcg_cut.10 Qrels/combined.man86.qrels vole.man86.run
```

To regenerate the run file from the saved logs:

```bash
python vole_run.py Results/*.log > vole.man86.run
```

`ndcg.man86.tsv` reports NDCG@10 on the same qrels for the Vole run and the
submitted TREC 2024 RAG Track runs from all groups. It is the full version of
Figure 3 from the paper.
