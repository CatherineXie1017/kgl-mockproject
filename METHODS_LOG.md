
# Institutional Self-Assertion in UN General Debate Discourse
## Methods and Reproducibility Log

**Corpus:** UN General Debate Corpus (UNGDC)  
**Study period:** 2016–2025  
**Research design:** Exploratory computational text analysis  
**Status:** Final analyses completed; 62-term Self-Assertion KGL frozen

This document records the implemented analytical workflow, including data processing, measurement construction, decision rules, model inference, statistical analyses, and reproducibility requirements. It is intended to be used alongside the project's source code, datasets, and manual annotation records.

The final project uses a theory-guided Institutional Self-Assertion Knowledge-Guided Lexicon (KGL) and a pretrained human-value classifier. Earlier plans to construct four additional framing lexicons were not implemented.

---

## 1. Research Design and Corpus

### 1.1 Research Questions

**RQ1:** Is the prevalence of Institutional Self-Assertion associated with autonomy-related value expression in Self-Assertion-matched sentences across selected countries?

**RQ2:** How do model-predicted human-value signals accompanying Institutional Self-Assertion vary across selected regional case pairs?

The study examines UN General Debate speeches from 2016–2025. Years serve as repeated observations of recent national diplomatic discourse rather than as predictors in a longitudinal analysis.

### 1.2 Corpus Construction

The corpus was divided into two nonoverlapping sets:

- **Development corpus:** Used exclusively for KGL development, expansion, purification, and contextual auditing.
- **Focal corpus:** Reserved for substantive cross-country and regional analyses.

The focal corpus was not used to modify lexical terms, weights, or inclusion decisions.

The focal sample comprised 18 purposively selected countries, grouped into nine two-country regional case pairs.

| Regional case pair | Countries (ISO3) |
|---|---|
| Modern West | USA, GBR |
| East Asia | CHN, JPN |
| Arab Region | SAU, JOR |
| Latin America | CHL, COL |
| South Asia | IND, PAK |
| Sub-Saharan Africa | GHA, KEN |
| Southeast Asia | IDN, THA |
| Central Asia | KAZ, TJK |
| Eastern Europe | POL, ROU |

Each focal country contributed one speech annually from 2016 through 2025.

| Corpus | Countries | Speeches |
|---|---:|---:|
| Focal | 18 | 180 |
| Development | 178 country units in coherence analysis | 1,755 |

The regional labels identify selected comparative cases and do not imply statistical representativeness of the broader regions.

### 1.3 Raw Data Parsing and Audit

Raw speech files followed the naming convention:

`ISO3_SESSION_YEAR.txt`

Example: `AFG_71_2016.txt`

Python recursively imported the text files and extracted country codes, UN session numbers, years, filenames, file paths, and speech text.

Corpus auditing verified:

- Complete focal country-year coverage: 18 countries × 10 years.
- Exactly one focal speech per country-year.
- No overlap between focal and development corpora.
- No missing or empty speech texts.
- No duplicate focal country-year combinations.
- Unusually short files contained coherent speech content.

Original raw files were retained unchanged.

### 1.4 Text Preprocessing

Preprocessing was deliberately conservative to preserve lexical information.

The implemented transformations were:

1. Unicode NFC normalization using `unicodedata.normalize("NFC", text)`.
2. Conversion of Unicode line separators, Windows line endings, and carriage returns to standard newlines.
3. Replacement of nonbreaking spaces and tabs with ordinary spaces.
4. Collapsing repeated spaces and excessive blank lines.
5. Removal of leading and trailing whitespace.
6. Creation of a lowercase computational representation.

Three text representations were retained:

- `text`: Original imported speech.
- `text_clean`: Formatting-normalized speech.
- `text_lower`: Lowercase speech.

### 1.5 Sentence Segmentation and Procedural-Text Removal

Cleaned speeches were segmented using spaCy's `en_core_web_sm` model.

The maximum document length was set to:

`nlp.max_length = 2_000_000`

Each sentence retained its original text, lowercase text, speech metadata, and within-speech sentence ID.

Some files contained introductory procedural statements spoken by the UN presiding officer. To avoid attributing these statements to national representatives, a sentence was removed only when both conditions were satisfied:

1. It occurred among the first five sentences of a speech.
2. It contained one of the following procedural strings:

`the assembly will hear an address`  
`i request protocol`  
`escort his excellency`  
`escort her excellency`

Sentence IDs were renumbered consecutively within each speech following removal.

| Processing checkpoint | Focal | Development |
|---|---:|---:|
| Speeches | 180 | 1,755 |
| Cleaned whitespace-delimited words | 390,849 | 3,576,476 |
| Sentences before procedural removal | 18,903 | 157,525 |
| Procedural sentences removed | 21 | 179 |
| Final cleaned sentences | **18,882** | **157,346** |

**Output files:**

- `data/preprocessed/focal_sentences_clean.csv`
- `data/preprocessed/development_sentences_clean.csv`

The whitespace-delimited counts reported above describe corpus size. Later KGL purification and speech-level prevalence calculations used their respective tokenization conventions, specified below.

---

## 2. Institutional Self-Assertion Construct and Seed Development

### 2.1 Construct Definition

**Institutional Self-Assertion** refers to language in which a focal national actor actively positions itself concerning its positions, goals, interests, rights, preferences, autonomy, capabilities, roles, or intended actions.

Relevant expressions may involve advancing, defending, maintaining, claiming, committing to, exercising, or resisting constraints on self-relevant positions or actions.

Three semantic conditions were required:

1. A recoverable focal national actor.
2. Active agentive self-positioning.
3. A self-relevant position, object, goal, or action.

Explicit first-person pronouns or an opposing actor were not required.

Expressions were excluded when they primarily represented:

- Generic action or stance-taking.
- Neutral factual self-description.
- Positive self-evaluation without self-positioning.
- Third-party agency.
- Aggression without relevant self-positioning.
- Unrelated or overly generic lexical senses.

Importantly, the lexical term itself had to provide meaningful evidence of Self-Assertion. Merely appearing in an assertive passage was insufficient.

The construct distinguishes **assertive linguistic form** from its potential cultural, social, or psychological function.

### 2.2 Construct Facets

Eight behavioral facets guided conceptual coverage:

| Facet | Meaning |
|---|---|
| `advance_outcome` | Advancing a desired outcome |
| `claim_position` | Claiming or expressing a position |
| `defend_self_relevant` | Defending a self-relevant object or position |
| `maintain_position` | Maintaining an established position |
| `resist_constraint` | Resisting external constraints |
| `commit_intended_action` | Expressing commitment or intended action |
| `exercise_capability_role` | Exercising capabilities or institutional roles |
| `influence_outcome` | Seeking influence over outcomes |

Direct construct labels were tracked separately.

### 2.3 Initial Candidate Audit

An initial theory-guided pool contained 20 terms:

`self-assertion`, `assertiveness`, `assertive`, `reject`, `resist`, `advocate`, `propose`, `pursue`, `influence`, `defend`, `demand`, `reaffirm`, `insist`, `assert`, `urge`, `refuse`, `oppose`, `uphold`, `claim`, `declare`.

Up to 20 occurrences per candidate were randomly sampled from the development corpus.

**Total manually reviewed: 346 contexts.**

Each target sentence was examined alongside its immediately preceding and following sentences, when available.

Coding considered focal-actor identification, self-relevant agency, lexical sense, overall construct relevance, and recurring diplomatic boilerplate.

Contextual precision informed seed decisions but was not used as a universal numerical inclusion threshold.

### 2.4 Independent Construct-Coverage Audit

An independent sample of **250 development-corpus contexts** was selected without filtering for existing candidate terms.

The audit identified:

| Classification | Contexts |
|---|---:|
| Clear Self-Assertion | 66 |
| Clear non-Self-Assertion | 181 |
| Ambiguous | 3 |

The audit identified insufficient coverage of two facets:

- Commitment and intended action.
- Exercise of capability or institutional role.

A targeted audit subsequently examined:

`commit`, `intend`, `ready`, `continue`, `pledge`, `exercise`, `participate`, `engage`, `mediate`, `lead`.

Up to 20 occurrences per term were inspected; all nine available occurrences of `mediate` were included.

**Total targeted contexts reviewed: 189.**

`intend` and `ready` were identified as useful commitment-related anchors. `exercise` was provisionally retained as a capability/role anchor.

### 2.5 Frozen Theory Seeds

Final seed selection jointly considered theoretical centrality, contextual relevance, semantic specificity, facet coverage, actor ambiguity, polysemy, and morphological redundancy.

To limit redundant weighting of morphological variants, `self-assertion` and `assert` were retained as direct seeds rather than independently retaining all closely related forms.

The resulting **14-term seed set** was frozen before FastText expansion results were inspected.

| Conceptual role | Frozen seeds |
|---|---|
| Direct assertion | `self-assertion`, `assert` |
| Advance/influence outcomes | `urge`, `propose`, `advocate` |
| Institutional positioning | `declare` |
| Defend self-relevant objects | `defend` |
| Maintain positions | `reaffirm`, `insist` |
| Resist constraints | `reject`, `resist` |
| Commitment/intended action | `intend`, `ready` |
| Capability/institutional role | `exercise` |

**Output:** `data/seed_candidates/self_assertion_seed_set_v1.csv`

---

## 3. FastText Lexical Expansion

The project adapted the Knowledge-Guided Lexical Analysis framework of Havaldar et al. (2024). It was not intended as an exact methodological replication.

### 3.1 Embedding Model

**Model:** `fasttext-wiki-news-subwords-300`

**Dimensionality:** 300

All 14 frozen seeds had available FastText representations.

Two expansion strategies were implemented.

### 3.2 Synonym Expansion

For each seed, candidate terms were evaluated using cosine similarity between FastText vectors.

A candidate was retained through the synonym-expansion route when:

\[
\cos(v_s,v_w) \geq 0.75
\]

where \(v_s\) represents the seed vector and \(v_w\) the candidate vector.

### 3.3 Concept-Centroid Expansion

A concept centroid was constructed by averaging the raw FastText vectors of the 14 seeds:

\[
C = \frac{1}{14}\sum_{i=1}^{14}v_i
\]

Candidates were retrieved based on cosine similarity with this centroid.

The following thresholds were examined:

`0.45`, `0.50`, `0.55`, `0.60`, `0.65`, `0.70`.

The retained threshold was:

\[
\cos(v_w,C) \geq 0.60
\]

The selected threshold balanced conceptual coverage against semantic diffusion and the feasibility of subsequent purification.

### 3.4 Lexical Weighting

Weights were assigned as follows:

\[
\operatorname{weight}(w)=
\begin{cases}
1, & \text{original theory seed}\\
\text{cosine similarity}, & \text{expanded term}
\end{cases}
\]

When an expanded term appeared through multiple expansion routes, the maximum applicable similarity weight was retained. Expansion-source metadata were preserved.

### 3.5 Tokenization

KGL purification used lowercase lexical tokens extracted with:

`TOKEN_PATTERN = re.compile(r"[a-z]+(?:-[a-z]+)*")`

This pattern recognizes alphabetic terms and hyphenated alphabetic forms while preventing arbitrary substring matching.

The same tokenization rule was used for development-corpus frequency counts and country-level lexical-coherence calculations.

---

## 4. KGL Purification and Finalization

### 4.1 Sequential Purification

The expanded vocabulary underwent sequential corpus-frequency, grammatical, contextual, statistical-coherence, and semantic screening.

| Stage | Decision rule | Remaining terms |
|---|---|---:|
| Frequency purification | Development-corpus count ≥ 10; original seeds provisionally protected | 543 |
| Hard function-word exclusion | Remove fixed exclusion list | 516 |
| Development-context evidence | Exclude non-seeds with ≥ 5 audited appearances and ≤ 25% Self-Assertion-context occurrence rate | 498 |
| Initial internal coherence | Positive country-level item-rest correlation; seeds provisionally protected | 328 |
| Full contextual semantic review | Retain semantically valid terms after adjudication | 69 |
| Final semantic/coherence rule | Semantic KEEP and (`r > 0` or surviving theory seed) | **62** |

The exact hard function-word exclusion set was:

`and`, `to`, `that`, `have`, `all`, `will`, `not`, `must`, `can`, `they`, `them`, `would`, `should`, `cannot`, `therefore`, `both`, `still`, `always`, `never`, `could`, `does`, `indeed`, `might`, `either`, `really`, `otherwise`, `please`.

### 4.2 Development-Context Evidence Screen

The independent 250-context audit contained 247 unambiguous observations:

- 66 clear Self-Assertion contexts.
- 181 clear non-Self-Assertion contexts.

For each non-seed candidate, its occurrences in these audited contexts were examined.

The implemented decision rules were:

- Fewer than five audited occurrences: retain provisionally because evidence was insufficient.
- At least five audited occurrences: exclude when no more than 25% occurred in human-coded Self-Assertion contexts.
- Original theory seeds: retain provisionally.

This stage was intentionally permissive and did not substitute for full semantic review.

### 4.3 Country-Level Internal-Correlation Screening

The development corpus contained 178 country-level units for this analysis.

For each candidate term \(w\) and country \(c\), weighted relative frequency was computed as:

\[
WF(w,c)=
\frac{\operatorname{count}(w,c)}
{\operatorname{total\_tokens}(c)}
\times \operatorname{weight}(w)
\]

This normalization accounted for unequal amounts of available country-level text.

For each term, internal coherence was measured using Pearson's correlation between its weighted country-frequency vector and the sum of the weighted vectors for all other candidate terms:

\[
r_i =
\operatorname{cor}_c
\left(
WF(w_i,c),
\sum_{j\ne i}WF(w_j,c)
\right)
\]

The target term was excluded from its own comparison sum.

This statistic measures internal lexical coherence across countries, not contextual precision or psychological validity.

The initial positive-correlation screen, with provisional protection for original seeds, yielded **328 terms** for full semantic review.

**Output:** `data/purification/self_assertion_kgl_candidate_final.csv`

### 4.4 Full Contextual Semantic Review

All 328 candidate terms were subjected to contextual review.

For each term:

- Up to five development-corpus occurrences were sampled.
- Occurrences from different speeches were preferred.
- The target sentence and available adjacent sentences were examined.
- Random state: `20260921`.

The review included **1,636 term-context observations**: five contexts for 327 terms and one available context for `self-assertion`.

Each term was evaluated according to whether its actual usage meaningfully indicated active focal-national self-positioning.

Initial classifications:

| Decision | Terms |
|---|---:|
| KEEP | 59 |
| UNCERTAIN | 23 |
| EXCLUDE | 246 |

All 23 uncertain terms were adjudicated using the original construct definition and contextual evidence.

Ten were retained:

`call`, `confirm`, `decide`, `decided`, `favor`, `favour`, `recommend`, `request`, `requested`, `withdraw`.

Thirteen were excluded:

`accede`, `claim`, `claiming`, `encourage`, `enact`, `exercise`, `negotiate`, `promise`, `ratify`, `seek`, `seeking`, `support`, `supported`.

The final semantic review retained **69 terms** and excluded **259 terms**.

Original seed status did not guarantee inclusion. In particular, `exercise` failed semantic review and was excluded.

**Output:** `data/purification/self_assertion_semantic_term_review_FINAL.csv`

### 4.5 Recomputed Internal Coherence

Internal correlations were recomputed using only the 69 semantically retained terms.

Recalculation was necessary because each term's item-rest correlation depends on the composition of the current lexicon.

Of these 69 terms, 59 had positive internal correlations.

The final inclusion rule was:

\[
\text{KEEP}_{final}
=
\text{Semantic KEEP}
\land
(r>0\ \lor\ \text{surviving theory seed})
\]

Three semantically retained original seeds had non-positive recomputed correlations:

- `self-assertion`
- `ready`
- `insist`

They were retained under the theory-seed exception.

Thus:

\[
59 + 3 = \boxed{62\text{ final terms}}
\]

### 4.6 Frozen Self-Assertion Lexicon

The final instrument was a **62-term weighted Institutional Self-Assertion KGL**.

**Final output:** `data/purification/self_assertion_kgl_FINAL.csv`

The lexicon was frozen before focal-country analysis and was not revised based on focal-country results.

The completed lexicon is a theory-guided and development-corpus-purified measure of Self-Assertion-related vocabulary, not a formally validated sentence-level classifier.

No independent holdout estimate of the final lexicon's precision, recall, or F1 score was obtained.

### 4.7 Essential Purification Artifacts

| File | Purpose |
|---|---|
| `data/purification/self_assertion_frequency_purified.csv` | Frequency-purified candidates |
| `data/purification/self_assertion_kgl_candidate_final.csv` | 328 candidates entering semantic review |
| `data/purification/self_assertion_semantic_term_review.csv` | Sampled contexts for semantic evaluation |
| `data/purification/self_assertion_semantic_term_review_FINAL.csv` | Final human semantic decisions |
| `data/purification/self_assertion_semantic69_recomputed_correlations.csv` | Recomputed correlations for 69 terms |
| `data/purification/self_assertion_kgl_FINAL.csv` | Frozen 62-term KGL and weights |

---

## 5. Focal Self-Assertion Measurement

### 5.1 Sentence-Level Retrieval

The frozen 62-term KGL was applied to the 18,882 cleaned focal sentences.

Matching used case-insensitive, word-boundary regular expressions.

Each matched sentence retained:

`iso3`, `year`, `filename`, `sentence_id`, `sentence_text`, `matched_terms`, `n_matches`.

Retrieval produced:

| Quantity | Result |
|---|---:|
| Matched sentences | 1,230 |
| KGL term occurrences | 1,309 |
| Speeches with matches | 177 |
| Speeches without matches | 3 |

One sentence could contain multiple KGL occurrences. Therefore, sentence counts and term-occurrence counts were distinguished.

**Output:** `data/contexts/focal_self_assertion_contexts.csv`

Only the matched sentence, rather than its surrounding contextual window, was used for subsequent human-value prediction.

### 5.2 Weighted Speech-Level Prevalence

For each focal speech \(d\), Institutional Self-Assertion prevalence was calculated as:

\[
SA_d =
1000\times
\frac{
\sum_{w\in L}f(w,d)\operatorname{weight}(w)
}{
\operatorname{wordcount}(d)
}
\]

where \(f(w,d)\) is the number of occurrences of term \(w\) in speech \(d\), and the weights are taken from the frozen lexicon.

Speech-level word counts used the regular-expression pattern:

`\b\w+\b`

All 180 speeches were included. Speeches without matches received a prevalence score of zero.

**Output:** `data/results/focal_self_assertion_scores.csv`

The file contains 180 observations and seven columns:

`iso3`, `year`, `filename`, `word_count`, `n_matches`, `weighted_matches`, `sa_per_1000`.

The overall mean speech-level prevalence was approximately 2.60 weighted matches per 1,000 words.

---

## 6. Human-Value Detection

### 6.1 Model Selection

To investigate value-related language accompanying Institutional Self-Assertion, the project used a pretrained multi-label classifier rather than developing separate framing lexicons.

**Model:** TUM DeBERTa Human Value Detector

**Model identifier:** `tum-nlp/Deberta_Human_Value_Detector`

**Source:** https://huggingface.co/tum-nlp/Deberta_Human_Value_Detector

The model predicts 20 categories from the ValueEval human-values framework, broadly based on Schwartz's refined basic human-values theory.

The model was used for inference only. No additional training or fine-tuning was conducted.

### 6.2 Sentence-Level Inference

The model was applied to all **1,230 Self-Assertion-matched sentences**.

- Input: `sentence_text`.
- Batch size: 8.
- Output: 20 continuous model-predicted value scores per sentence.
- No conversion to binary classifications.

**Output:** `data/results/focal_self_assertion_values.csv`

The resulting dataset contained:

- 1,230 sentence-level observations.
- Seven metadata columns.
- Twenty predicted value-score columns.
- 27 columns in total.

The complete set of 20 value categories was retained for exploratory analysis.

### 6.3 Informal Inspection and Validation Scope

Sixty sentences and their predicted scores were inspected informally.

This was not a formal validation study. No independently annotated UNGD test set, inter-rater reliability assessment, or corpus-specific classification accuracy estimate was produced.

The model was originally developed for argument-based value detection rather than UN diplomatic discourse. Predictions were therefore interpreted as model-estimated value-related language rather than verified psychological motives.

---

## 7. Aggregation and Final Analyses

### 7.1 Country-Level Self-Assertion Prevalence

Self-Assertion prevalence was averaged across the ten annual speeches for each country:

\[
SA_c =
\frac{1}{10}
\sum_{y=2016}^{2025}SA_{cy}
\]

All years contributed equally, including years with zero KGL matches.

### 7.2 Country-Level Human-Value Scores

Predicted human-value scores were aggregated in two stages.

**Stage 1 — Sentence to country-year**

For value \(v\), the country-year mean was:

\[
V_{cy,v} =
\frac{1}{n_{cy}}
\sum_{s=1}^{n_{cy}}p_{s,v}
\]

where \(p_{s,v}\) is the predicted score for sentence \(s\).

**Stage 2 — Country-year to country**

Available country-year scores were averaged equally:

\[
V_{c,v} =
\frac{1}{|Y_c|}
\sum_{y\in Y_c}V_{cy,v}
\]

Years without Self-Assertion-matched sentences were treated as missing for conditional value-score aggregation, not assigned zero.

Eligible years were:

| Countries | Eligible years |
|---|---:|
| India | 8 |
| Indonesia | 9 |
| Remaining 16 countries | 10 each |

This approach prevented country-years with more matched sentences from automatically receiving greater weight.

### 7.3 Regional Case-Pair Aggregation

Regional case-pair scores were calculated as the unweighted mean of the two constituent country means:

\[
X_r = \frac{X_{c_1}+X_{c_2}}{2}
\]

This procedure was applied separately to Self-Assertion prevalence and each human-value category.

Regional scores were not calculated by pooling all matched sentences across the two countries.

### 7.4 RQ1: Self-Assertion and Autonomy-Related Values

RQ1 examined the association between Self-Assertion prevalence and two autonomy-related value categories:

- **Self-direction: action:** Primary indicator.
- **Self-direction: thought:** Complementary indicator.

The two scores were analyzed separately rather than combined into a composite.

Power: dominance was not treated as an autonomy indicator because it concerns control over others rather than self-determination.

Spearman rank-order correlations were calculated between Self-Assertion prevalence and each Self-direction score at two aggregation levels:

- Country level: \(N=18\).
- Regional case-pair level: \(N=9\).

Two-sided p-values were calculated. The country-level analysis was primary; regional correlations were secondary descriptive checks using aggregated observations from the same countries.

| Analysis level | Self-direction: action (ρ) | Self-direction: thought (ρ) |
|---|---:|---:|
| Country (N = 18) | −0.040 | −0.098 |
| Regional case pair (N = 9) | +0.167 | +0.267 |

The observed associations were interpreted descriptively rather than generalized to national populations or entire cultural regions.

RQ1 scatterplots displayed country-level observations alongside labeled regional case-pair means.

### 7.5 RQ2: Screening Regional Value Variation

All 20 predicted human-value categories were initially examined.

For each category \(v\), the range of regional case-pair means was calculated:

\[
R_v =
\max_r(V_{r,v})-\min_r(V_{r,v})
\]

The regional pairs with the highest and lowest scores were identified, and constituent-country scores were inspected for within-pair consistency.

A post-hoc exploratory threshold of:

\[
R_v\geq0.10
\]

was used to prioritize categories for visualization.

Ten categories met the threshold:

| Category |
|---|
| Achievement |
| Security: societal |
| Universalism: concern |
| Universalism: nature |
| Tradition |
| Self-direction: action |
| Power: resources |
| Power: dominance |
| Self-direction: thought |
| Universalism: tolerance |

The threshold was a descriptive screening decision, not a preregistered criterion or statistical significance threshold.

All 20 categories remained in the analytical dataset.

### 7.6 Region-Centered Value Profiles

The first RQ2 figure summarized the relative value profile of each regional case pair.

For each value category, regional case-pair means were standardized across the nine pairs:

\[
Z_{r,v} =
\frac{V_{r,v}-\mu_v}{\sigma_v}
\]

where \(\mu_v\) and \(\sigma_v\) are the mean and standard deviation of the nine regional scores for value \(v\), calculated with `ddof=0`.

The figure contained nine panels, one per regional case pair.

Within each panel:

- Ten screened value categories were ranked by their standardized scores.
- Values with \(z\geq1\) were highlighted in blue.
- Values with \(z\leq-1\) were highlighted in red.
- All remaining values were displayed in gray.

The color thresholds served only to highlight relative positions and did not indicate statistical significance.

Standardization was performed separately for each value category. Therefore, z-scores describe a region's relative standing within a category, not the relative psychological importance of different values.

**Figure:** `outputs/figures/rq2_region_centered_profiles_FINAL.png`

### 7.7 Value-Centered Regional Comparisons

A complementary figure compared regional means within six selected value categories:

1. Achievement.
2. Security: societal.
3. Universalism: concern.
4. Self-direction: action.
5. Power: dominance.
6. Universalism: tolerance.

These categories were selected for focused presentation based on exploratory regional contrasts, interpretability, constituent-country patterns, and conceptual breadth.

The selection was data-informed rather than confirmatory.

The figure displayed nine regional case-pair means for each category.

Within each panel:

- Blue identified the highest regional mean.
- Orange identified the lowest regional mean.
- Gray represented all remaining regional means.
- Highest and lowest scores were numerically annotated.

Each value category used an independent horizontal scale.

Comparisons were therefore interpreted within categories rather than as directly comparable effect sizes across categories.

**Figure:** `outputs/figures/rq2_value_centered_comparison_FINAL.png`

The region-centered and value-centered figures were complementary presentations of the same observations, not independent evidence.

---

## 8. Reproducibility and Project Files

### 8.1 Notebook Execution Order

The analysis was implemented in Python using Jupyter notebooks.

| Notebook | Function |
|---|---|
| `01_data_audit.ipynb` | Corpus parsing and completeness checks |
| `02_preprocessing.ipynb` | Text normalization and sentence segmentation |
| `03_seed_lexica.ipynb` | Construct development, seed audits, and seed freezing |
| `04_expansion.ipynb` | FastText expansion and weighting |
| `05_purification.ipynb` | Corpus-based and contextual KGL purification |
| `06_self_assertion_contexts.ipynb` | Focal Self-Assertion sentence extraction |
| Notebook 07 | Speech-level Self-Assertion prevalence |
| Notebook 08 | Descriptive country comparisons |
| Notebook 09 | Human-value model inference |
| Notebook 10 | Value-score aggregation and exploratory heatmap |
| Notebook 11 | RQ1 and RQ2 analyses and final figures |

The exact filenames for Notebooks 07–11 should be taken from the repository.

Notebooks should be executed in numerical order, preserving dependencies on previously generated datasets.

### 8.2 Core Data Artifacts

| File | Content |
|---|---|
| `data/preprocessed/focal_sentences_clean.csv` | 18,882 cleaned focal sentences |
| `data/preprocessed/development_sentences_clean.csv` | 157,346 cleaned development sentences |
| `data/seed_candidates/self_assertion_seed_set_v1.csv` | Frozen 14-term seed set |
| `data/purification/self_assertion_kgl_FINAL.csv` | Frozen weighted 62-term KGL |
| `data/contexts/focal_self_assertion_contexts.csv` | 1,230 matched sentences |
| `data/results/focal_self_assertion_scores.csv` | Speech-level Self-Assertion scores |
| `data/results/focal_self_assertion_values.csv` | Sentence-level human-value predictions |

Intermediate purification and manual annotation artifacts are documented in Section 4.7.

### 8.3 Final Figures

| Figure | File |
|---|---|
| RQ1 autonomy association | `outputs/figures/rq1_autonomy_association.png` |
| RQ2 region-centered profiles | `outputs/figures/rq2_region_centered_profiles_FINAL.png` |
| RQ2 value-centered comparisons | `outputs/figures/rq2_value_centered_comparison_FINAL.png` |

The final RQ2 figures were also exported as vector PDF files.

### 8.4 Reproducibility Checkpoints

A successful rerun should reproduce the following quantities, allowing minor numerical differences from floating-point computations where appropriate.

| Stage | Expected result |
|---|---:|
| Focal speeches | 180 |
| Development speeches | 1,755 |
| Cleaned focal sentences | 18,882 |
| Cleaned development sentences | 157,346 |
| Frozen seeds | 14 |
| Frequency-purified terms | 543 |
| After function-word removal | 516 |
| After contextual evidence screening | 498 |
| Candidates entering semantic review | 328 |
| Semantically retained terms | 69 |
| Final KGL terms | 62 |
| Focal matched sentences | 1,230 |
| Matched KGL occurrences | 1,309 |
| Zero-match focal speeches | 3 |
| Human-value categories | 20 |
| Regional case pairs | 9 |
| Values in region-centered figure | 10 |
| Values in value-centered figure | 6 |

### 8.5 Requirements for Exact Reproduction

Exact computational reproduction requires more than this written log.

The repository or its documentation should provide:

1. **Corpus provenance:** The precise UNGDC source/release, acquisition procedure, and source-file inventory.
2. **Software environment:** Version-pinned Python dependencies and required spaCy/FastText resources.
3. **Model provenance:** Pretrained model identifiers, relevant revisions, and inference settings.
4. **Manual annotation records:** Sampled contexts and human decisions from seed audits and semantic purification.
5. **Random sampling settings:** Recorded random seeds and sampled observations for the manual-audit stages.
6. **Executable notebooks:** The exact matching, aggregation, prediction, and figure-generation implementations.
7. **Frozen artifacts:** The final weighted KGL and intermediate data necessary to verify lexical decisions.

The semantic review used random state `20260921`; random states for all earlier manual-sampling stages are not fully specified in the original log.

The original log also does not fully document model revision identifiers, all dependency versions, or every low-level inference setting. These should be recovered from the actual notebooks or environment rather than inferred.

The incomplete development-retrieval description in the original Section 44 should also be verified against the corresponding source code if that operation is required for full end-to-end reproduction.

---

## 9. Methodological Scope and Interpretation Boundaries

The project supports exploratory analysis of Self-Assertion-related lexical prevalence and accompanying model-predicted value signals in selected UN diplomatic speeches.

Interpretation is restricted by the following considerations.

**Lexical construct validity.** The 62-term KGL is a theory-guided, context-audited lexical approximation. It does not guarantee that every matched occurrence constitutes Institutional Self-Assertion in its full discourse context.

**Classifier validity.** The pretrained DeBERTa model was not formally validated on an independently labeled sample of UNGD speeches. Its outputs are model-predicted textual signals rather than verified value expressions or psychological motivations.

**Conditional sampling.** Human-value predictions were calculated only for Self-Assertion-matched sentences. No non-Self-Assertion baseline was analyzed, so value patterns cannot be established as exclusive to Self-Assertion contexts.

**Cross-category comparability.** Predicted scores were compared across regional cases within the same value category. Different value categories were not assumed to share a calibrated measurement scale.

**Regional representativeness.** The 18 countries and nine regional case pairs were purposively selected. Regional means describe the selected pairs, not representative estimates of broader cultural regions.

**Statistical inference.** Spearman correlations and regional value contrasts were exploratory. Aggregated regional observations were derived from the same country sample and did not constitute independent replication. No causal or motivational mechanisms were established.

**Exploratory selection.** The RQ2 range threshold, ten screened categories, six focal values, and visualization thresholds were data-informed decisions, not confirmatory statistical tests.

No new classifier training, unsupervised topic modeling, non-Self-Assertion baseline analysis, multilevel explanatory modeling, or formal large-scale independent annotation validation was conducted.

The analytical workflow was completed after the final RQ1 and RQ2 comparisons and figures were generated. The frozen KGL was not subsequently revised in response to focal results.
