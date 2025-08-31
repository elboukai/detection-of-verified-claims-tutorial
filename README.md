---
title: "Fact-Checking Claims Using SimBa: A Tutorial for Retrieving Similar Verified Claims"

---

## Learning Objectives

By the end of this tutorial, you will be able to:

1. Use SimBa to retrieve fact-checks for similar claims from the ClaimsKG database
2. Interpret similarity scores and understand the ranking methodology 
3. Evaluate and customize SimBa's similarity features for your specific use case
4. Set up proper evaluation frameworks using goldstandard files and CLEF metrics

## Target audience

This tutorial is aimed at researchers and practitioners working with fact-checking and claim verification. Prior knowledge of Python programming and basic command line usage is required for this tutorial. Familiarity with machine learning concepts (embeddings, similarity measures) is helpful but not essential.

## Setting up the computational environment

Install the required packages and clone the repository:

```bash
# Clone the SimBa repository
git clone https://github.com/BDA-KTS/detection-of-verified-claims.git
cd detection-of-verified-claims

# Install dependencies
pip install -r requirements.txt
```

## Duration

Around 30-45 minutes

## Social Science Usecase(s)

This method addresses the critical social science problem of misinformation and fact-checking in digital media. SimBa has been used to support fact-checkers in identifying previously verified claims, helping to combat the spread of false information during political campaigns, health crises, and other socially relevant events. The method is particularly valuable for analyzing political discourse, health misinformation, and social media content verification.

## Understanding ClaimsKG and the Problem Context

Fact-checking information has become crucial for maintaining information integrity across different platforms and initiatives globally. Fact-checks are scattered across different portals and published in various formats. ClaimsKG ([https://data.gesis.org/claimskg/](https://data.gesis.org/claimskg/)) addresses this by harvesting claims, claim reviews, and respective metadata from popular fact-checking sites, making them available in a homogeneous, machine-readable format.

**Example scenario**: Suppose you encounter this claim in a political debate or online: *"Covid-19 vaccines increase the risk of dying from the new Covid-19 variants."* How can you check whether this claim has already been fact-checked? What evidence did fact-checkers find, and what was their verdict?

## How SimBa Works

SimBa operates through a two-step process:

1. **Candidate retrieval**: Given an input claim, the 50 most similar claims in the corpus are selected as candidates using embedding-based similarities
2. **Re-ranking**: Using a combination of different features, candidates are re-ranked and the most similar ones are returned

![Pipeline](screenshots/SimBA_2023_architecture.png)

## Preparing Your Input Data

Create a directory structure for your data:

```bash
cd detection-of-verified-claims/data
mkdir mydata
cd ..
```

Create your input file at `data/mydata/queries.tsv` with the following format:

```
1	Covid-19 vaccines increase the risk of dying from the new Covid-19 variants
```

Format specifications:
- `1`: Unique ID for your input claim (query)
- Tab character separating ID and query text  
- Query text without tabs or newlines
- One query per line for multiple queries

## Running SimBa

Execute the main script with your data folder:

```bash
python main.py mydata
```

This compares your query to all claims in the `data/claimsKG/corpus.tsv` file, which contains approximately 40,000 verified claims from ClaimsKG.

## Interpreting the Results

SimBa generates two output files:

- `data/mydata/pred_client.tsv`: Primary human-readable output
- `data/mydata/pred_qrels.tsv`: CLEF format for automatic evaluation

### Understanding pred_client.tsv

| Query | VClaim | ClaimReviewURL | Rating | Similarity |
|-------|--------|----------------|--------|------------|
| Covid-19 vaccines increase the risk of dying from the new Covid-19 variants | Vaccinated people are more susceptible to Covid-19 variants | https://factcheck.afp.com/http%253A%252F%252Fdoc.afp.com%252F9PB64D-1 | b'False' | 51.24902489669692 |
| Covid-19 vaccines increase the risk of dying from the new Covid-19 variants | Covid-19 vaccines will leave people exposed to deadly illness during the next cold and flu season and germ theory is a hoax | https://factcheck.afp.com/covid-19-shots-not-designed-increase-cold-flu-lethality | b'False' | 51.102840014017175 |
| Covid-19 vaccines increase the risk of dying from the new Covid-19 variants | Getting the first dose of Covid-19 vaccine increases risk of catching the novel coronavirus | https://factcheck.afp.com/misleading-facebook-posts-claim-covid-19-vaccine-increases-risk-catching-novel-coronavirus | b'Misleading' | 50.774385556493115 |
| Covid-19 vaccines increase the risk of dying from the new Covid-19 variants | People vaccinated against Covid-19 pose a health risk to others by shedding spike proteins | https://factcheck.afp.com/covid-19-vaccine-does-not-make-people-dangerous-others | b'False' | 49.87148066707767 |
| Covid-19 vaccines increase the risk of dying from the new Covid-19 variants | Mass vaccination will cause monster Covid-19 variants | https://factcheck.afp.com/mass-covid-19-vaccination-will-not-lead-out-control-variants | b'False' | 49.756611441979224 |

**Key points**:
- Similarity scores range from 0 to ~100
- You always get 5 results by default
- Higher scores indicate greater similarity
- The top result is most similar but may still be unrelated if no good matches exist

## Customizing Similarity Methods

You can modify the similarity approach by editing the `retrieval_command` list in `main.py`. 

### Default Configuration

SimBa uses:
- **Candidate retrieval**: Top-50 candidates using `all-mpnet-base-v2` embedding model
- **Re-ranking**: Combination of three sentence embedding models plus lexical/referential/string features

### Available Similarity Features

| Type | Options | Description |
|------|---------|-------------|
| **Lexical** | `similar_words_ratio`, `similar_words_ratio_length` | Word overlap (excluding stopwords) |
| **Referential** | `spacy_ne_similarity`, `ne_similarity`, `synonym_similarity` | Based on named entities or synonyms |
| **String** | `levenshtein`, `jaccard_similarity`, `sequence_matching` | Raw text similarity |

Example customization:

```python
retrieval_command = [
    # ... other parameters ...
    "-lexical_similarity_measures", "similar_words_ratio",
    "-referential_similarity_measures", "spacy_ne_similarity", 
    "-string_similarity_measures", "levenshtein",
    "--similarity_measure", "braycurtis"
]
```

## Evaluation and Parameter Tuning

For systematic evaluation of different feature combinations:

### 1. Create a Gold Standard File

Prepare `data/mydata/gold.tsv` in TREC qrels format:
```
<query_id>    0    <correct_claim_id>    1
```

### 2. Run Evaluation

```bash
# Clone CLEF evaluation tools
git clone https://github.com/sshaar/clef2020-factchecking-task2
cd clef2020-factchecking-task2

# Run evaluation
python evaluate.py \
  --scores /path/to/data/mydata/pred_qrels.tsv \
  --gold-labels /path/to/data/mydata/gold.tsv \
  --metrics map --metrics precision --metrics reciprocal_rank \
  --depths 1 --depths 3 --depths 5 \
  -o results.tsv
```

This produces metrics like MAP@1, MAP@3, MAP@5, Precision@k, and MRR@k for optimization.

## Performance Optimization

For faster reruns when working with the same corpus:

```bash
python main.py sample -c
```

The `-c` flag reuses cached embeddings, significantly reducing computation time for repeated experiments.

## Conclusion

You now know how to use SimBa for retrieving similar fact-checked claims and how to customize and evaluate different similarity approaches for your specific use case. This method provides an efficient way to leverage existing fact-checking work and can significantly support verification workflows in social science research.

## References

- Hövelmeyer, Alica, Katarina Boland, and Stefan Dietze. 2022. *SimBa at CheckThat! 2022: Lexical and Semantic Similarity-Based Detection of Verified Claims in an Unsupervised and Supervised Way.* In: CEUR Workshop Proceedings, Vol. 3180, pp. 511–531. [PDF](https://ceur-ws.org/Vol-3180/paper-40.pdf)

- Boland, Katarina, Hövelmeyer, Alica, Fafalios, Pavlos, Todorov, Konstantin, Mazhar, Usama, & Dietze, Stefan. 2023. *Robust and Efficient Claim Retrieval for Online Fact-Checking Applications.* Preprint. [DOI](https://doi.org/10.21203/rs.3.rs-3007151/v1)

## Contact

For questions or support, contact: [katarina.boland@hhu.de](mailto:katarina.boland@hhu.de)