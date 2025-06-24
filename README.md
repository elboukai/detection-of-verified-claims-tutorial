# Tutorial: Fact-Checking a Claim Using SimBa

##  Problem: What is the veracity of a claim? Has it been fact-checked before?

Suppose you find this claim in a political debate or online:  
**"Dog-owners face 78% higher risk of catching Covid-19."**  
How can you check whether this claim has already been fact-checked? What evidence did the fact-checkers find for or against it and what was their verdict? Or are there maybe related fact-checks that may help you find relevant evidence concerning this claim?

This tutorial walks you through how to use **SimBa**, a high-performance method that retrieves **fact-checks for similar claims** from **ClaimsKG**, a structured database which serves as a registry of fact-checked claims.

## Preliminaries: What is ClaimsKG?

## Preliminaries: How does SimBa work?

---

##  Step 1: Prepare Your Input File

Create a file:

```
data/sample/queries.tsv
```

It should contain:

```
25603	Singer and actress Cher died in December 2022 or January 2023.
```

- `25603` is a query ID.
- One query per line, with a tab (`	`) separating ID and claim text.

---

##  Step 2: Install SimBa

```bash
git clone https://github.com/BDA-KTS/detection-of-verified-claims.git
cd detection-of-verified-claims
pip install -r requirements.txt
```

Then download required NLTK data:

```bash
python
>>> import nltk
>>> nltk.download('stopwords')
>>> nltk.download('punkt')
>>> nltk.download('wordnet')
>>> exit()
```

---

##  Step 3: Run SimBa

```bash
python main.py sample
```

This compares your query to 74,000 verified claims in **ClaimsKG** and ranks the most similar ones.

---

##  Step 4: Understanding the Output

You’ll get two output files:

- `data/sample/pred_qrels.tsv`: CLEF format for evaluation
- `data/sample/pred_client.tsv`: Human-readable

### Sample: `pred_client.tsv`

| Query | Verified Claim | URL | Rating | Similarity |
|-------|----------------|-----|--------|------------|
| Cher... | Cher died... | [Snopes](https://www.snopes.com/fact-check/cher-death-hoax) | False | 72.06 |
| Cher... | Chevy Chase... | snopes.com | False | 39.27 |
| Cher... | Cher + Plant... | snopes.com | False | 36.72 |
| Cher... | Karachi storm... | afp.com | False | 31.60 |
| Cher... | Hillary run... | truthorfiction.com | Decontextualized | 31.38 |

### ❗ Important Notes:

- You will **always get 5 results** (default), even if none are actually similar.
- The **top-ranked claim is just the most similar** — it may still be unrelated.
- The **similarity score** ranges from **0 to ~100**, indicating relative closeness based on all features. A higher score does *not* guarantee factual relevance.
- The CheckThat! paper (Hövelmeyer et al., 2022) shows that **semantic similarity alone is not always sufficient** — hence, SimBa uses re-ranking with multiple features.

---

##  Optional: Customize the Similarity Method

Edit `main.py` to modify the `retrieval_command` list.

###  Default Behavior

SimBa **first retrieves top-50 candidates** using *one* embedding model (default: `all-mpnet-base-v2`).

Then it **re-ranks** those using:

- **4 different sentence embedding models**
- Plus **lexical, referential, and string similarity features**

This combination performed best on CheckThat! benchmarks.

---

###  Available Similarity Features

| Type | Options | Description |
|------|---------|-------------|
| **Lexical** | `similar_words_ratio`, `similar_words_ratio_length` | Word overlap (excluding stopwords) |
| **Referential** | `spacy_ne_similarity`, `ne_similarity`, `synonym_similarity` | Based on named entities or synonyms |
| **String** | `levenshtein`, `jaccard_similarity`, `sequence_matching` | Raw text similarity |

You can combine them like this:

```python
"-lexical_similarity_measures", "similar_words_ratio",
"-referential_similarity_measures", "spacy_ne_similarity",
"-string_similarity_measures", "levenshtein"
```

---

###  Embedding Models Used in Re-Ranking

SimBa combines 4 pretrained models:

- `all-mpnet-base-v2`
- `sentence-t5-base`
- `unsup-simcse-roberta-base`
- `paraphrase-MiniLM-L6-v2`

These are fixed inside the code (re-ranking step). The candidate retrieval step uses one of these (default or user-defined).

---

##  How to Evaluate and Tune Parameters

To see which combination works best for your data:

1. Prepare your queries and a goldstandard file (`gold.tsv`) listing correct matches.
2. Run SimBa with various combinations.
3. Use CLEF's evaluation script to compute MAP@k scores.

According to [Hövelmeyer et al. 2022], the **best performing combination** was:

- Retrieval with `all-mpnet-base-v2`
- Re-ranking using the 4 embeddings + lexical + referential similarity

---

##  Fast Reruns

Use `-c` to reuse cached embeddings:

```bash
python main.py sample -c
```

---

##  Further Reading

- Hövelmeyer, Alica, Katarina Boland, and Stefan Dietze. 2022. *SimBa at CheckThat! 2022: Lexical and Semantic Similarity-Based Detection of Verified Claims in an Unsupervised and Supervised Way.* In: CEUR Workshop Proceedings, Vol. 3180, pp. 511–531. [PDF](https://ceur-ws.org/Vol-3180/paper-40.pdf)

- Boland, Katarina, Hövelmeyer, Alica, Fafalios, Pavlos, Todorov, Konstantin, Mazhar, Usama, & Dietze, Stefan. 2023. *Robust and Efficient Claim Retrieval for Online Fact-Checking Applications.* Preprint. [DOI](https://doi.org/10.21203/rs.3.rs-3007151/v1)

---

##  Need Help?

Contact: [katarina.boland@hhu.de](mailto:katarina.boland@hhu.de)

---


