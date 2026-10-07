# Name matching across identity documents

This repository compares three small, explainable methods for matching name strings from identity documents. The only implementation is the Jupyter notebook. The data contains 200 synthetic, labeled document pairs, and the saved predictions let a reviewer inspect every score.

## Files

| File | Purpose |
| --- | --- |
| `name_matching_three_approaches.ipynb` | Run the three methods one cell at a time, tune cutoffs, and compare results. Saved cell output is included. |
| `name_matching_test_cases.csv` | 200 labeled pairs: 105 matches and 95 non-matches. Each row has two synthetic document IDs, a category, and an ambiguity flag. |
| `name_matching_predictions.csv` | Scores and decisions for all 200 pairs from the recorded run. Recreated when the notebook runs. |

## Run from a clean checkout

Install Python 3 and Git, or download and extract the repository ZIP from GitHub instead of cloning it. The matching code needs only Python's standard library. Jupyter Notebook is needed to open and execute the `.ipynb` file interactively. The installation and launch commands follow [Project Jupyter's instructions](https://jupyter.org/install).

**Windows PowerShell**

```powershell
git clone https://github.com/Maniteja-ai/name-matching-identity-documents.git
cd name-matching-identity-documents
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install notebook
.\.venv\Scripts\python.exe -m notebook
```

**macOS or Linux**

```sh
git clone https://github.com/Maniteja-ai/name-matching-identity-documents.git
cd name-matching-identity-documents
python3 -m venv .venv
.venv/bin/python -m pip install notebook
.venv/bin/python -m notebook
```

In the browser, open `name_matching_three_approaches.ipynb`. Run cells from top to bottom, or use **Run All Cells**. The last cell writes `name_matching_predictions.csv` in the same directory. If you downloaded a ZIP, start with the `cd` command in the extracted repository directory.

## What each approach does

All methods lowercase and split names into words. They do not remove titles, relationship text, or known spelling variants. Each method produces a similarity score between 0 and 1.

### 1. Trie with edit distance

The notebook puts the second document's words in a trie. It finds exact words, compatible initials, and words within a small edit distance. It then chooses the best **one-to-one alignment** between the two names, so word order can change and a word cannot be reused. Unmatched words contribute zero.

Example: `Kumar Ramesh` and `Ramesh Kumar` align both words exactly and score **1.00**. `S. Kumar` and `Suresh Kumar` receive a weaker score (**0.775**) because `S.` could expand to many names.

### 2. Inverted index with weighted word overlap

Each exact word maps to the IDs of documents containing it. Shared words retrieve candidates. The final score is **weighted Jaccard similarity**: the weight of shared words divided by the weight of all distinct words across both names. A frequent word contributes less than a rare one, and an extra word enlarges the denominator.

Example: `Rahul Dev Sharma` versus `Rahul Sharma` scores **0.5653**. The extra `Dev` reduces the score instead of being ignored. This approach does not find spelling variants unless another word overlaps.

### 3. Character vectors with cosine similarity

Each word becomes a count vector of its two- and three-character fragments. Cosine similarity compares word vectors, and the best one-to-one word alignment becomes the name score. This is a character-vector method run in memory; a vector database is unnecessary for 200 pairs.

Example: `Mohammed Khan` versus `Muhammad Khan` scores **0.7059**. The exact `Khan` helps, while the two spellings of the first word have only partial fragment overlap.

## Recorded result

The notebook scored all 200 pairs. It used every fifth pair (40 pairs) to choose each method's cutoff by minimizing `5 × false positives + false negatives`; ties favor the higher cutoff. The table reports the other **160 pairs**: 85 matches and 75 non-matches.

| Method | Cutoff | True matches | False matches | Missed matches | True non-matches | Precision | Recall | Cost |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Trie | 0.94 | 14 | 0 | 71 | 75 | 1.00 | 0.16 | 71 |
| Inverted index | 1.00 | 11 | 0 | 74 | 75 | 1.00 | 0.13 | 74 |
| Character vectors | 1.00 | 10 | 0 | 75 | 75 | 1.00 | 0.12 | 75 |

The trie has the lowest cost under this illustrative weighting, but **none is suitable for automatic acceptance as written**: each misses most constructed matches. For example, `Shri Ramesh Kumar` versus `Ramesh Kumar` is labeled a match but scores below every cutoff because `Shri` remains an extra word. The full predictions file shows the score and decision for each case.

## Dataset and interpretation

The examples cover reordered words, initials, transliterations, missing middle names, titles, typos, shared surnames, close given names, and spouse/father-name confusion. A `structural_ambiguity=yes` flag marks cases where an initial, omission, subset, or relationship form leaves important name information unresolved.

The labels describe constructed same-person or different-person scenarios. Names alone cannot prove identity, and the dataset does not label relationships between documents in different rows. The comparison measures behavior on these pairs, not production KYC accuracy. A real system would validate on identity-verified records and send uncertain pairs to review.

## Next improvement

Add explicit handling for document titles and relationship fields, then measure how much recall improves without creating unacceptable false matches. Keep those rules visible and test their effect separately from the three similarity calculations.
