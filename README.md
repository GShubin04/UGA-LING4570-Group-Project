# LING 4570 — Group -1's Project

Computational analysis of EU political speech using the EUSpeech dataset.

## Dataset

**EUSpeech** — [Harvard Dataverse (DOI: 10.7910/DVN/XPCVEI)](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/XPCVEI)

18,403 speeches from EU leaders collected from 2007 to 2015, covering heads of government from 10 member states (CZ, DE, FR, GR, IT, NL, PL, PO, SP, UK), EU institutional actors (European Commission, European Parliament, EU Council, ALDE, ECR), and international bodies (ECB, IMF). All non-English speeches were pre-translated.

> Schumacher et al. (2016). _EUSpeech: a New Dataset of EU Elite Speeches._ Proceedings of the International Conference on the Advances in Computational Analysis of Political Text. [dx.doi.org/10.7910/DVN/XPCVEI](https://dx.doi.org/10.7910/DVN/XPCVEI)

## Notebooks

### `nb/preprocess.ipynb`

Prepares the corpus and measures Jensen-Shannon Divergence (JSD) between nations using n-grams as probability distributions.

**What it does:**

1. Loads the 17 translated CSV files from `data/Translated/` and separates speeches into a national corpus (10 countries) and an institutional corpus (EU bodies, ECB, IMF)
2. Cleans text (lowercases, strips punctuation, and splits each speech into sentences)
3. Exports one sentence per line to `output/countries/<NATION>.txt` for use by the SRILM notebook
4. Tokenizes all speeches and builds per-nation unigram, bigram, and trigram count distributions
5. Constructs a universal n-gram vocabulary across all nations
6. Computes pairwise JSD across all nation pairs at each n-gram order, displayed as a heatmap (a score of 0 means identical distributions, 1 means completely disjoint)
7. Samples a random institutional speech and finds which national corpus it is most similar to by JSD score

**Why:** JSD on n-gram distributions is a lightweight, interpretable way to measure linguistic similarity between political actors without requiring a trained model

---

### `nb/srilm.ipynb`

Trains and evaluates n-gram language models on each nation's speeches using SRILM.

**What it does:**

1. Loads the per-nation `.txt` files produced by `preprocess.ipynb`
2. Applies a document count minimum of 2,000 sentences per nation, skipping nations with insufficient data to ensure models are trained on enough text
3. Randomly shuffles and splits each qualifying nation's sentences into 80% train / 20% test sets
4. Trains bigram, trigram, 4-gram, and 5-gram language models with Kneser-Ney discounting
5. Evaluates four smoothing configurations on 4-gram models: standard KND, KND with interpolation, modified KND, and modified KND with interpolation
6. Generates sample text from the best-performing model

**Why:** Perplexity measures how well a language model predicts unseen text, and comparing smoothing strategies across languages reveals how much the choice of discounting method matters

## Project Structure

```
project-group-1/
├── data/
│   ├── Cleaned_Speeches/      # Cleaned CSVs (pre-translation)
│   └── Translated/            # Translated CSVs (used as input)
├── nb/
│   ├── preprocess.ipynb       # JSD analysis (Parker)
│   └── srilm.ipynb            # Perplexity analysis (Grant)
└── output/
    ├── countries/             # Per-nation sentence files (preprocess.ipynb)
    ├── train/                 # Train splits (srilm.ipynb)
    ├── test/                  # Test splits (srilm.ipynb)
    └── models/                # Trained SRILM .lm files (srilm.ipynb)
```
