# Generating UMBRELA Judgments

[UMBRELA](https://github.com/castorini/umbrela) uses a language model to assign
graded relevance judgments to query-passage pairs. This recipe uses GPT-4o and
UMBRELA's zero-shot Bing relevance prompt.

## Prepare the input

Obtain the query text and complete passage text by whatever means are
appropriate for the collection. Write one query-passage pair per line of a
JSONL file:

```json
{"query":{"qid":"q1","text":"How long is the life cycle of a flea?"},"candidates":[{"docid":"d1","doc":{"segment":"The complete passage text."}}]}
```

Using one candidate per input line makes each output line correspond
unambiguously to one query-passage pair. Preserve the input file so that the
query and document identifiers can later be associated with each judgment.

## Install UMBRELA

Pinning a revision is important because prompt templates and model interfaces
can change. Revision `e2c4e4126bf7b568d9230008b465a58445cabc30` provides the
commands and prompt used in this recipe:

```bash
git clone https://github.com/castorini/umbrela.git
cd umbrela
git checkout e2c4e4126bf7b568d9230008b465a58445cabc30

uv python install 3.12
uv venv --python 3.12
source .venv/bin/activate
uv sync --group dev --extra cloud

export OPENAI_API_KEY=...
```

Keep credentials in the environment and out of input, output, and tracked
files.

## Inspect the prompt

The recipe uses UMBRELA's built-in zero-shot Bing prompt. Inspect the exact
template before running the judge:

```bash
umbrela prompt show --prompt-type bing --few-shot-count 0
```

It assigns an integer grade:

- `0`: the passage has nothing to do with the query;
- `1`: the passage is related but does not answer the query;
- `2`: the passage contains some answer, although it may be unclear or mixed
  with extraneous information;
- `3`: the passage is dedicated to the query and contains the exact answer.

The prompt asks the model to consider search intent, content match, and passage
trustworthiness. It requests only `##final score: N`, without an explanation.
The prompt has no system message and uses no few-shot examples.

## Run the judge

```bash
umbrela judge \
  --backend gpt \
  --model gpt-4o \
  --prompt-type bing \
  --few-shot-count 0 \
  --input-file /path/to/umbrela-input.jsonl \
  --output-file /path/to/umbrela-judgments.jsonl \
  --output json \
  --overwrite
```

Each output record contains an integer `judgment` from 0 through 3. Check that
the output has the expected number of records and that every judgment is in
that range before using it.

The name `gpt-4o` is a provider-managed alias. The provider may change the
model behind it, so judgments made at a later date are not guaranteed to be
bit-for-bit identical. Record the UMBRELA revision, model name, prompt type,
few-shot count, provider, and run date with every judgment set.

## Create qrels, if needed

Pair each output line with its input line. For TREC-style evaluation, emit the
input `qid`, input `docid`, and output `judgment` in four-column qrels format:

```text
qid 0 docid judgment
```

Retain the input JSONL, output JSONL, resulting qrels, exact command, software
revision, and model metadata. Together they identify the query and passage
that produced every relevance grade.
