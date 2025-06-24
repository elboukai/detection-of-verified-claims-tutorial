# Tutorial: Fact-Checking a Claim Using SimBa

##  Problem: What is the veracity of a claim? Has it been fact-checked before?

Suppose you find this claim in a political debate or online:  
**"Dog-owners face 78% higher risk of catching Covid-19."**  
How can you check whether this claim has already been fact-checked? What evidence did the fact-checkers find for or against it and what was their verdict? Or are there maybe related fact-checks that may help you find relevant evidence concerning this claim?

This tutorial walks you through how to use **SimBa**, a high-performance method that retrieves **fact-checks for similar claims** from **ClaimsKG**, a structured database which serves as a registry of fact-checked claims.

## Preliminaries: What is ClaimsKG?

## Preliminaries: How does SimBa work?

---

##  Step 1: Install SimBa

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

##  Step 2: Prepare Your Input File

First, create a directory in the data subfolder. You can assign any name to it. Let's call it "mydata" for this tutorial. 

```bash
cd detection-of-verified-claims/data
mkdir mydata
cd ..
```

Create a file in your new subfolder:

```
data/mydata/queries.tsv
```

It should contain:

```
1	Dog-owners face 78% higher risk of catching Covid-19.
```

- `1`: the ID for your input claim (from now on called "query"). You can use any number as the ID.
- (`	`): a tab separating the ID and the query text
- `Dog-owners face 78% higher risk of catching Covid-19.`: the query text. Please make sure that it does not contain any tabs or newlines. 
- You can enter an arbitrary number of queries to the file. For this, enter one query per line, each starting with an ID, followed by a tab and a query text. Please make sure that your IDs are unique, i.e. every query has a different ID. 

---



##  Step 3: Run SimBa

```bash
python main.py mydata
```

This compares your query to all claims in the `data/claimsKG/corpus.tsv` file and retrieves the most similar ones. This file contains ....... verified claims in English language from **ClaimsKG**. 

---

##  Step 4: Understanding the Output

You’ll get two output files:

- `data/mydata/pred_client.tsv`: Primary output file
- `data/mydata/pred_qrels.tsv`: CLEF format for automatic evaluation


### Output: `pred_client.tsv`

| Query | Verified Claim | URL | Rating | Similarity |
|-------|----------------|-----|--------|------------|
| ....

### ❗ Note:

- The **similarity score** ranges from **0 to ~100**, indicating relative similarity based on all employed features.
- You will **always get 5 results** (default), even if the most similar claims do not have a high similarity score.
- The **top-ranked claim is the most similar claim in the database** — it may still be unrelated.

---

##  Optional: Customize the Similarity Method

Edit `main.py` to modify the `retrieval_command` list.

###  Default Behavior

SimBa **first retrieves top-50 candidates** using *one* embedding model (default: `all-mpnet-base-v2`).

Then it **re-ranks** those using:

- **4 different pre-trained sentence embedding models**:
  - `all-mpnet-base-v2`
  - `sentence-t5-base`
  - `unsup-simcse-roberta-base`
  - `paraphrase-MiniLM-L6-v2`
- Plus **lexical, referential, and string similarity features**

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

##  How to Evaluate and Tune Parameters

The feature combination that performs best on CheckThat! benchmarks is (Boland et al. 2023)
- Retrieval with `...`
- ...
- braycurtis similarity measure

If you would like to evaluate if for your data, other feature combinations may yield better results, do the following:

1. Prepare a goldstandard file (`gold.tsv`) for your query listing correct matches. ........ format?
2. Run SimBa with various combinations.
3. Use CLEF's evaluation script to compute MAP@k scores.    ......... how?



---

##  Fast Reruns

Use `-c` to reuse cached embeddings for your corpus:

```bash
python main.py sample -c
```

You may do so whenever you are repeatedly working with the same corpus of previously fact-checked claims. SimBa will compute the embeddings once, store and reuse them. When you want to use a different corpus than for your previous SimBa call, do not use the cached embeddings of the previously used corpus. 

---

##  Further Reading

- Hövelmeyer, Alica, Katarina Boland, and Stefan Dietze. 2022. *SimBa at CheckThat! 2022: Lexical and Semantic Similarity-Based Detection of Verified Claims in an Unsupervised and Supervised Way.* In: CEUR Workshop Proceedings, Vol. 3180, pp. 511–531. [PDF](https://ceur-ws.org/Vol-3180/paper-40.pdf)

- Boland, Katarina, Hövelmeyer, Alica, Fafalios, Pavlos, Todorov, Konstantin, Mazhar, Usama, & Dietze, Stefan. 2023. *Robust and Efficient Claim Retrieval for Online Fact-Checking Applications.* Preprint. [DOI](https://doi.org/10.21203/rs.3.rs-3007151/v1)

---

##  Need Help?

Contact: [katarina.boland@hhu.de](mailto:katarina.boland@hhu.de)

---


