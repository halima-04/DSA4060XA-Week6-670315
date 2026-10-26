# DSA 4060XA – Recommender Systems
## Week 6 Guided Practical Lab: Evaluating Recommender Systems

**Student name:** Halima Mahdi Mohammed  
**Student ID:** 670315  
**Course:** DSA 4060XA – Recommender Systems  
**Lab topic:** Popularity Baseline, Content-Based Filtering, and Item-Based Collaborative Filtering  
**Top-N cutoff:** \(K = 3\)

---

## 1. Project overview

This repository contains the Week 6 guided practical lab for evaluating recommender systems. The practical compares three approaches to movie recommendation:

1. **Popularity Baseline** – uses average training ratings to predict ratings and to rank unseen movies.
2. **Content-Based Filtering** – represents movies with binary genre features and compares a user's genre-preference profile with candidate movies.
3. **Item-Based Collaborative Filtering (Item-Based CF)** – calculates movie-to-movie similarity from the training users' rating patterns and predicts ratings using similarity-weighted ratings from each user's history.

The goal is not only to generate recommendations, but to evaluate how well the models perform against a common held-out test set and to identify the trade-offs between rating accuracy, ranking quality, catalogue coverage, novelty, and diversity.

## 2. Repository contents

```text
DSA4060XA-Week6-670315/
├── DSA4060XA_Week6_Guided_Evaluation_Lab.ipynb
├── DSA4060_Week6_Movies.csv
├── DSA4060_Week6_Ratings.csv
└── README.md
```

- `DSA4060XA_Week6_Guided_Evaluation_Lab.ipynb` contains detailed Markdown explanations, Python code, tables, charts, evaluation, discussion prompts, limitations, and the extension challenge.
- `DSA4060_Week6_Movies.csv` contains movie metadata, including movie ID, title, and genre labels.
- `DSA4060_Week6_Ratings.csv` contains user–movie rating interactions, including user ID, movie ID, rating, and timestamp.
- `README.md` documents the purpose, methods, findings, recommendation, and limitations.

## 3. Lab objectives

The practical aims to:

- inspect movie metadata and explicit rating interactions;
- split each user's interactions chronologically into training data and a held-out test interaction;
- construct a simple popularity baseline;
- build a genre-based content recommender;
- build an item-based collaborative-filtering recommender;
- compare rating predictions with Mean Absolute Error (MAE) and Root Mean Squared Error (RMSE);
- evaluate Top-3 lists with Precision@3, Recall@3, and NDCG@3;
- calculate catalogue coverage, novelty, and intra-list diversity;
- use the results to recommend a model for a streaming service that wants relevant recommendations without repeatedly showing only the same popular titles.

## 4. Dataset and preparation

The supplied dataset contains **20 catalogue movies and 120 rating interactions**. The data include explicit ratings on a 1–5 scale. The notebook reports the unique user count, number of rated movies, rating distribution, and missing-value checks directly from the CSV files.

Each row in the movies file represents a movie and its metadata. Each row in the ratings file represents one user's rating of one movie at a timestamp.

### Train/test split

The ratings are sorted by `user_id` and `timestamp`. For each user, the latest interaction is held out as test data; earlier interactions are used for training. This is a chronological, per-user holdout.

The split matters because the recommender must be evaluated on interactions that were not used to construct movie averages, user profiles, or item similarities. Evaluating on interactions already used for learning can make the reported performance unrealistically optimistic.

All model calculations use the training set. The test set is used only for evaluation.

## 5. Evaluation metrics

### Rating prediction

- **MAE:** the average absolute difference between actual and predicted ratings. Lower is better.
- **RMSE:** the square root of the average squared prediction error. Lower is better, and larger errors receive more weight than they do under MAE.

### Ranking quality

For ranking, a held-out movie is considered relevant when its actual rating is 4 or 5.

- **Precision@3:** the proportion of the three recommended items that are relevant.
- **Recall@3:** the proportion of the relevant held-out items that are retrieved in the Top-3 list.
- **NDCG@3:** measures ranking quality and rewards relevant items placed nearer the top of the list.

Higher values are better for these ranking metrics. The ranking metrics in this lab are based on one held-out interaction per user, so they should be interpreted as a teaching exercise rather than a robust estimate of production ranking performance.

### Discovery and catalogue metrics

- **Catalogue coverage:** the proportion of all catalogue movies that appear in at least one user's Top-3 recommendations. Higher coverage means more of the catalogue is exposed.
- **Novelty:** estimates how uncommon recommended items are, based on the proportion of training users who rated each item. Higher novelty means the model tends to recommend less commonly interacted-with movies.
- **Diversity:** measures how different the genres are within a recommendation list. It is calculated as one minus the average pairwise cosine similarity between genre vectors. Higher values indicate greater genre variety.

Coverage, novelty, and diversity are not direct measures of relevance. A model can recommend obscure or varied items that users do not actually want, so these metrics must be considered alongside Precision@3, Recall@3, and NDCG@3.

## 6. Models compared

### 6.1 Popularity Baseline

For rating prediction, the model predicts a movie's average rating in the training data. If a movie has no training rating, the global training mean is used. For ranking, movies are sorted by training mean rating, with rating count used to break ties.

**Strengths:** simple, fast, transparent, and a useful benchmark.  
**Weaknesses:** limited personalisation and a tendency to concentrate recommendations around popular or well-rated items.

### 6.2 Content-Based Filtering

The model converts movie genres into binary features. A user's profile is calculated from the genre vectors of movies in their training history, weighted by the user's ratings. Candidate movies are scored using cosine similarity between the profile and each movie's genre vector.

**Strengths:** can personalise recommendations using a user's own history and can explain recommendations through genre overlap. It does not require other users' ratings for a movie if the movie's metadata are available.  
**Weaknesses:** depends on metadata quality, may over-recommend familiar genres, and can struggle to recommend items outside the user's existing interests. The lab converts cosine similarity into an estimated 1–5 rating with a simple heuristic, so this rating estimate is not calibrated or learned from data.

### 6.3 Item-Based Collaborative Filtering

The model creates a user–item rating matrix and calculates cosine similarity between movie rating vectors. To estimate a user's rating for a target movie, it uses a similarity-weighted average of ratings from the user's training history.

**Strengths:** can capture patterns in users' rating behaviour that are not represented by genres alone.  
**Weaknesses:** requires enough interaction data, may struggle with new or rarely rated movies, and can be affected by sparse data. In this lab, missing ratings are filled with zero before cosine similarity is calculated; zero is not a valid rating on the 1–5 scale, so this simplification can distort similarities.

## 7. Results

The notebook was run from top to bottom using the supplied CSV files. The resulting metrics are shown below. Values are rounded to three decimal places.

### 7.1 Rating prediction

| Model | MAE ↓ | RMSE ↓ |
|---|---:|---:|
| Popularity Baseline | 1.086 | 1.201 |
| Content-Based | 1.094 | 1.381 |
| Item-Based CF | **0.924** | **1.070** |

**Finding:** Item-Based CF has the lowest MAE and RMSE in this run, so it performs best for rating prediction among the three evaluated approaches. The content-based model does not beat the popularity baseline on either rating-error metric. However, the results are based on a small teaching dataset and one held-out interaction per user, so they should not be treated as a general conclusion about the models.

### 7.2 Top-3 ranking

| Model | Precision@3 ↑ | Recall@3 ↑ | NDCG@3 ↑ |
|---|---:|---:|---:|
| Popularity Baseline | 0.067 | 0.200 | 0.100 |
| Content-Based | 0.067 | 0.200 | **0.200** |
| Item-Based CF | 0.000 | 0.000 | 0.000 |

**Finding:** Popularity and content-based filtering tie on Precision@3 and Recall@3 in this run. Content-Based has the highest NDCG@3, meaning that among the relevant items it retrieves, its ranking placement is better under this evaluation. Item-Based CF has the strongest rating prediction results but does not retrieve the relevant held-out items in the Top-3 list in this run. This demonstrates that low rating-prediction error does not guarantee strong Top-N recommendation quality.

### 7.3 Catalogue coverage

| Model | Coverage ↑ |
|---|---:|
| Popularity Baseline | 0.350 |
| Content-Based | 0.700 |
| Item-Based CF | **0.850** |

**Finding:** Item-Based CF exposes the largest proportion of the catalogue across users. Content-Based filtering also covers more of the catalogue than the popularity baseline. Popularity's lower coverage is consistent with a more concentrated recommendation strategy.

### 7.4 Novelty

| Model | Novelty ↑ |
|---|---:|
| Popularity Baseline | 1.689 |
| Content-Based | 1.344 |
| Item-Based CF | **1.695** |

**Finding:** Item-Based CF has the highest novelty score, narrowly ahead of the popularity baseline. Content-Based has the lowest novelty score in this run, suggesting its recommendations are more concentrated on movies with higher training-user interaction support under the chosen novelty definition. Novelty should not be interpreted as relevance.

### 7.5 Diversity

| Model | Diversity ↑ |
|---|---:|
| Popularity Baseline | 0.736 |
| Content-Based | 0.542 |
| Item-Based CF | **0.780** |

**Finding:** Item-Based CF has the highest genre-based intra-list diversity, while Content-Based has the lowest. This is a reminder that a model built around genre preferences may recommend several items with similar genre features. The measure captures genre variety only, not every aspect of perceived movie similarity.

## 8. Best model by objective

### Best model for rating prediction: Item-Based CF

Item-Based CF has the lowest MAE (0.924) and RMSE (1.070). If the main objective were to estimate users' numerical ratings for movies, this would be the preferred model based on the current offline results.

### Best model for ranking: Content-Based Filtering, with a qualification

Content-Based filtering has the highest NDCG@3 (0.200). It ties with the popularity baseline on Precision@3 (0.067) and Recall@3 (0.200). Therefore, it is the strongest ranking choice if top-positioning as measured by NDCG is prioritised, but it does **not** outperform popularity on every ranking metric. The ranking results are sparse and should be interpreted cautiously.

### Best model for catalogue discovery: Item-Based CF

Item-Based CF has the highest coverage (0.850), novelty (1.695), and diversity (0.780). However, its ranking metrics are zero in this evaluation. It would therefore be risky to select it solely for discovery without improving or validating its relevance performance.

## 9. Final business recommendation

**Scenario:** A streaming service wants relevant recommendations but does not want the same popular titles to dominate every recommendation list.

**Recommendation for this dataset: start with Content-Based Filtering as the primary Top-3 recommendation approach, while monitoring relevance and discovery metrics.**

The reasons are:

1. Content-Based ties the popularity baseline on Precision@3 and Recall@3.
2. Content-Based has the highest NDCG@3 (0.200), suggesting better placement of relevant items near the top of the list in this run.
3. Content-Based catalogue coverage (0.700) is twice the popularity baseline coverage (0.350), so it exposes more of the available catalogue.
4. It offers a reasonable explanation for recommendations through genre overlap, which can be useful for a new streaming service.

**Trade-off accepted:** Content-Based has worse rating-prediction error than the popularity baseline and lower novelty and diversity than Item-Based CF in this run. It may also reinforce users' existing genre preferences rather than introduce sufficiently different content.

A sensible next step would be to test a **hybrid or re-ranking strategy**: first produce a relevance-oriented candidate list, then adjust the final list to improve coverage, novelty, and diversity without allowing relevance to fall below an agreed threshold. Item-Based CF should not be selected as the only model merely because it wins the discovery metrics, because it retrieved none of the relevant held-out items in the current Top-3 evaluation.

## 10. Limitations

At least the following limitations affect the interpretation of the results:

1. **Small dataset:** the catalogue contains only 20 movies and the ratings file contains 120 interactions. A small sample can produce unstable metrics and may not generalise to a real streaming platform.
2. **One held-out interaction per user:** each user contributes only one test interaction. This makes the ranking evaluation sparse and gives limited evidence about a user's broader set of relevant movies.
3. **Relevance threshold:** relevance is defined as a rating of 4 or 5. A different threshold, such as rating exactly 5, could change ranking results.
4. **Limited content features:** content-based filtering uses only genre labels. It ignores descriptions, actors, directors, language, release year, user context, and other useful information.
5. **Heuristic content rating conversion:** the conversion from cosine similarity to a 1–5 score is a simple teaching approximation, not a calibrated rating prediction model.
6. **Missing ratings encoded as zero in CF:** zero is outside the 1–5 rating scale, so filling missing entries with zero can influence cosine similarity in unintended ways.
7. **Simplified novelty definition:** novelty uses interaction support as a proxy for popularity. A less common movie is not necessarily relevant or valuable to the user.
8. **Genre-only diversity:** diversity is based on genre-vector similarity and does not fully represent how different movies may feel to viewers.
9. **No significance testing:** the lab does not test whether differences between metric values are statistically meaningful.
10. **Offline evaluation only:** no user study, online A/B test, satisfaction measure, watch time, retention, or business outcome is measured.
11. **Cold-start and sparse-data issues:** new users and movies with few interactions may not have enough data for reliable collaborative filtering or preference profiles.
12. **Popularity tie-breaking:** average rating and count are simple ranking signals; the model does not use confidence intervals or shrinkage to account for low-support items.

## 11. Reproducibility and how to run

### Requirements

- Python 3
- Jupyter Notebook or JupyterLab
- pandas
- NumPy
- scikit-learn
- matplotlib

Install the libraries if needed:

```bash
pip install pandas numpy scikit-learn matplotlib notebook
```

### Run instructions

1. Download or clone this repository.
2. Keep the notebook and both CSV files in the same folder.
3. Open `DSA4060XA_Week6_Guided_Evaluation_Lab.ipynb` in Jupyter Notebook, JupyterLab, or VS Code.
4. Ensure the selected Python environment has the required libraries installed.
5. Select **Restart Kernel and Run All Cells**.
6. Confirm that all output tables and charts appear and that there are no errors.
7. If you change the dataset or code, rerun the notebook from the beginning so the results remain consistent.

The notebook reads the supplied CSV files using their relative filenames. If they are stored elsewhere, either move them into the notebook folder or update the file paths in Step 2.

## 12. Suggested next experiments

The lab extension implemented in the notebook compares Top-3 recommendations for two users (Option C). Further experiments could include:

- changing K to 1, 3, and 5 and comparing Precision@K, Recall@K, and NDCG@K;
- changing the relevance threshold from rating ≥ 4 to rating = 5;
- using multiple held-out interactions per user instead of one;
- calculating item similarity using only co-rated users or mean-centred ratings;
- learning a calibration function for content-based rating predictions;
- building a hybrid recommender that combines collaborative and content-based scores;
- applying a diversity-aware re-ranking step and measuring whether ranking relevance remains acceptable.

## 13. Conclusion

The experiment demonstrates that recommender evaluation is multi-dimensional. Item-Based CF performs best on rating prediction and discovery metrics in this run, but it fails to retrieve relevant held-out items in the Top-3 ranking evaluation. Content-Based filtering has the highest NDCG@3 and broader catalogue coverage than the popularity baseline, making it the most defensible starting point for the stated business objective in this small dataset. The final model choice should be validated with more interactions, stronger evaluation design, and user-centred outcomes before deployment.
