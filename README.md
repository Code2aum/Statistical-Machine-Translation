# Assignment 4: Phrase-Based Decoder

## Team 8
- Sabiha Tahsin Soha
- Halleluyah Brhanemesqel
- Harshith Ravi Kopparam

We pair programmed throughout this assignment, with all team members contributing equally to the code, debugging, and writeup.

## Overview

This project ties together our phrase-based machine translation system; word alignments (Assignment 1), language model (Assignment 2), and phrase extraction (Assignment 3); into a **stack decoder** for Spanish → English translation.

The decoder includes distortion penalties, hypothesis recombination, proper LM backoff, and bidirectional phrase scores.

### Quick Start

```bash
./run_pipeline.sh
```

This runs the complete pipeline:
1. **phrasetable.py** — Loads the 5,575 scored phrase pairs extracted in Assignment 3
2. **ngram_lm.py** — Tests the trained 4-gram English language model from Assignment 2
3. **translate.py** — Tokenizes Spanish input sentences and runs the stack decoder
4. **postprocessing.py** - Fixes an edge case whereby the article "the" appears before an abstract noun and the abstract noun is followed by a verb. 
Eg: Fixes "the love is blind" to "love is blind" 

### Results

| # | Spanish | Best Translation | Reference | Match? |
|---|---------|-----------------|-----------|--------|
| 1 | la comisión europea | the european commission | the european commission | YES |
| 2 | señor presidente , este informe es importante | mr president , this report is important | mr president , this report is important | YES |
| 3 | la unión europea | the european union | the european union | YES |
| 4 | política regional | regional policy | regional policy | YES |
| 5 | los estados miembros | member states | member states | YES |
| 6 | este debate es importante | this debate is important | this debate is important | YES |
| 7 | el parlamento europeo | the european parliament | the european parliament | YES |
| 8 | sin embargo , la comisión | however , the commission | however , the commission | YES |
| 9 | mi grupo no ha | my group did not | my group has not | no |
| 10 | la seguridad es importante | safety is important | safety is important | YES |

**9 out of 10 exact matches.** The 1 mismatch is still reasonable:
- Sentence 9: "did not" vs "has not" —> both valid translations of "no ha"; the LM prefers "did not"

Sentence 10 is corrected by a post-processing step that drops a spurious sentence-initial "the" before abstract/uncountable nouns that are followed by a verb (confirming subject position). In Spanish, abstract concepts used as subjects require a definite article ("la seguridad es importante"), but in English we do the opposite: when we talk about general abstract concepts, we drop the word "the." ("safety is important"). The decoder faithfully translates "la" → "the," and the post-processor removes it when the noun is abstract and verifiably a subject.

### Requirements

- Python 3.13.2 or Python 3.12.4
- No external dependencies (uses only standard library)
- Requires outputs from Assignment 2 (language model) and Assignment 3 (phrase table)

### Running Individual Steps

```bash
python3 phrasetable.py    # Step 1: Load phrase table
python3 ngram_lm.py       # Step 2: Test language model
python3 translate.py      # Step 3: Run stack decoder on test sentences and calls postprocessing.py at the end
```

### Components

- **Phrase Table** (`phrasetable.py`) — Loads the scored phrase pairs from Assignment 3. Each entry has a Spanish source phrase, English target phrase, and a combined cost in bits (cost(f|e) + cost(e|f), following the bidirectional approach).

- **Language Model** (`ngram_lm.py`) — Wraps the trained 4-gram English language model from Assignment 2. Scores new English words given the translation so far using proper backoff.

- **Stack Decoder** (`decoder.py`) — The core search algorithm:
  - Maintains one stack per number of source words covered
  - Expands hypotheses by trying all applicable phrase pairs
  - Scores each hypothesis: `phrase_cost + LM_cost + distortion_cost`
  - Recombines equivalent hypotheses (same coverage + LM state)
  - Prunes each stack to the top 200 hypotheses via histogram pruning

- **Post-Processing** (`postprocessing.py`) — Rule-based corrections applied after decoding. Drops spurious sentence-initial "the" when it precedes an abstract/uncountable noun that is followed by a verb (confirming the noun is a subject). This handles a Spanish-English divergence: Spanish requires a definite article before abstract subjects ("la seguridad es importante"), but English uses no article ("safety is important").

- **Translation Script** (`translate.py`) — Tokenizes Spanish input sentences, runs the decoder, applies post-processing, and displays the best translations alongside reference English.

### Output Files

```
output/
└── translations.txt       # Saved translation results
```

### Project Structure

```
├── phrasetable.py         # Loads phrase table from Assignment 3
├── ngram_lm.py            # Wraps Assignment 2 language model
├── decoder.py             # Stack decoder with distortion + recombination
├── postprocessing.py      # Rule-based post-processing (article correction)
├── translate.py           # Main script: tokenizes, translates, and post-processes
├── run_pipeline.sh        # Bash script to run the complete pipeline
└── README.md
```



### Reproducibility

This assignment loads two artifacts produced by earlier assignments(the first one should be downloaded by the tester from the google drive link provided below):

| File | Loaded by | Path used in code | Source |
|------|-----------|-------------------|--------|
| `ngram_model.pkl` | `ngram_lm.py` | `Assignment 4/models/ngram_model.pkl` | Assignment 2 (trained 4-gram English language model) |
| `phrase_table.txt` | `phrasetable.py` | `../Assignment 3/tables/phrase_table.txt` | Assignment 3 (scored phrase pairs extracted from word alignments) |

- **`ngram_model.pkl`** should be stored as a local copy inside `Assignment 4/models/`. Download it from [Google Drive](https://drive.google.com/drive/folders/1WSazkU_zG83_SkGuSBkk-3GZxG7JOUzi?usp=sharing) and create a `models` directory inside `Assignment 4` and place it in that directory. See the Assignment 2 README for instructions on regenerating it from scratch.

- **`phrase_table.txt`** is read directly from the `Assignment 3/tables/` directory (the code uses a relative path `../Assignment 3/tables/phrase_table.txt`). This means the Assignment 3 directory must be present alongside Assignment 4. See the Assignment 3 README for instructions on regenerating it from scratch. By default `phrase_table.txt` should be generated when you clone the repo. 
