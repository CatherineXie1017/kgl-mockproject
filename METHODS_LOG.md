# Methodology and Decision Log

## Project Overview

**Working title:**  
*What Does Self-Assertion Mean Across Cultures? A Knowledge-Guided and Exploratory Computational Analysis of Self-Assertion in Global Institutional Discourse*

**Corpus:** UN General Debate Corpus (UNGDC)

**Primary analysis period:** 2016–2025

**General design:**  
The project examines how self-assertion is expressed and framed in global institutional discourse.

The analysis separates the corpus into:

- a **development corpus**, used for construction, quality control, expansion, purification, and validation of Knowledge-Guided Lexica (KGLs); and
- a **focal corpus**, reserved for the eventual substantive cross-country and cross-regional analyses.

This separation is intended to reduce the risk of adapting the measurement instrument to produce desired results in the focal countries.

The project is exploratory rather than preregistered. Nevertheless, methodological decisions, thresholds, quality-control procedures, and deviations from the original plan are documented prospectively wherever possible.

---

# 1. Overall Planned Analytical Framework

The project follows the general Knowledge-Guided Lexicon framework developed by Havaldar et al., adapted to the present corpus and research question.

The broad analytical sequence is:

1. corpus construction and auditing;
2. conservative text preprocessing;
3. sentence segmentation and removal of non-substantive procedural material;
4. theory-guided seed-word construction;
5. FastText-based synonym expansion;
6. FastText-based concept/centroid expansion;
7. similarity-based weighting of expanded terms;
8. corpus-frequency purification;
9. internal-correlation purification;
10. lexicon validation and sensitivity analyses;
11. freezing of the lexica before focal-country analysis;
12. focal self-assertion scoring;
13. extraction of self-assertion contexts;
14. conditional scoring of framing lexica within self-assertion contexts;
15. country- and region-level descriptive aggregation;
16. residual semantic discovery among self-assertion contexts poorly captured by the theory-derived framing lexica.

The five planned lexica are:

1. **Self-Assertion**
2. **Self-Oriented Framing**
3. **Ingroup-Protective Framing**
4. **Collective-Advancement Framing**
5. **Relational/Negotiation Framing**

The Self-Assertion lexicon serves as the focal behavioral retrieval lexicon.

The four framing lexica are intended to characterize the lexical context surrounding self-assertion rather than to measure independent psychological traits.

The project therefore distinguishes:

> **whether self-assertion occurs**

from:

> **how self-assertion is framed when it occurs**

The lexica are interpreted as measures of **lexical framing associated with theoretical functions**, not direct measures of speakers' psychological motives.

---

# 2. Corpus Definition

## 2.1 Analysis period

Only speeches from **2016 through 2025** are included.

Earlier UN General Debate speeches were excluded because the project is designed to characterize relatively recent institutional discourse rather than longitudinal historical change.

Year therefore primarily functions as repeated sampling of each country's recent discourse rather than as a substantive longitudinal predictor.

---

## 2.2 Focal corpus

The focal corpus consists of **18 countries**, with two selected country cases from each of nine cultural regions:

| Region | Country 1 | Country 2 |
|---|---|---|
| Modern West | United States (USA) | United Kingdom (GBR) |
| East Asia | China (CHN) | Japan (JPN) |
| Arab region | Saudi Arabia (SAU) | Jordan (JOR) |
| Latin America | Colombia (COL) | Chile (CHL) |
| South Asia | India (IND) | Pakistan (PAK) |
| Sub-Saharan Africa | Ghana (GHA) | Kenya (KEN) |
| Southeast Asia | Indonesia (IDN) | Thailand (THA) |
| Central Asia | Kazakhstan (KAZ) | Tajikistan (TJK) |
| Eastern Europe | Poland (POL) | Romania (ROU) |

Each focal country contributes one UN General Debate speech per year for 2016–2025.

**Intended focal sample:**

18 countries × 10 years = **180 speeches**

The two countries within each region are treated as selected country cases rather than as exhaustive or statistically representative samples of their broader cultural regions.

The regional labels are therefore used as theoretically informative descriptive groupings rather than as claims that two countries fully represent an entire region.

---

## 2.3 Development corpus

All available 2016–2025 speeches from countries not included among the 18 focal countries were placed in the development corpus.

The development corpus is used for lexicon construction, quality control, expansion, purification, and validation.

The focal and development corpora remain separate during lexicon development.

Conceptually:

`Development corpus`

→ build and refine the measurement instrument

`Focal corpus`

→ apply the frozen measurement instrument

This separation is intended to reduce focal-sample overfitting.

---

# 3. Raw File Structure and Parsing

The UNGD text files follow the filename structure:

`ISO3_SESSION_YEAR.txt`

Example:

`AFG_71_2016.txt`

where:

- `AFG` = three-letter country code;
- `71` = UN General Assembly session;
- `2016` = speech year.

Python recursively searched the focal and development directories for `.txt` files.

For each speech, the following information was extracted:

- corpus membership;
- ISO3 country code;
- UN session;
- year;
- filename;
- original filepath;
- complete speech text;
- preliminary whitespace-based word count.

Each raw `.txt` speech was converted into one row of a pandas DataFrame.

The resulting speech-level DataFrames were:

- `focal_df`
- `dev_df`

---

# 4. Corpus Audit

The resulting dataset sizes were:

- **Focal corpus:** 180 speeches
- **Development corpus:** 1,755 speeches

Corpus audit checks included:

1. confirming that the focal corpus contained exactly the intended 18 country codes;
2. confirming that each focal country contributed exactly 10 speeches;
3. confirming coverage of every year from 2016 through 2025;
4. checking that each focal country-year combination occurred exactly once;
5. checking for duplicate country-year records;
6. confirming that none of the focal country codes remained in the development corpus;
7. checking for missing speech text;
8. checking for empty speech text;
9. examining speech-length distributions; and
10. inspecting unusually short speeches for evidence of truncation or corrupted extraction.

The structured datasets were saved as CSV files for reproducible downstream processing.

Generated processed datasets are retained locally rather than tracked in Git because they can be recreated from the source corpus and notebooks.

---

# 5. Initial Text Preprocessing

Preprocessing was intentionally conservative.

The goal of this stage was to standardize formatting and Unicode representation without removing potentially meaningful lexical information.

The original speech text was preserved in the `text` column.

A separate `text_clean` column was created for preprocessing.

---

## 5.1 Unicode normalization

Text was normalized using Unicode **NFC normalization**:

`unicodedata.normalize("NFC", text)`

This standardizes alternative underlying Unicode representations of visually equivalent characters without removing accents or changing lexical content.

---

## 5.2 Line and paragraph separator normalization

The following Unicode separators were converted to ordinary newline characters:

- `U+2028` LINE SEPARATOR → `\n`
- `U+2029` PARAGRAPH SEPARATOR → `\n`

These were treated as formatting variants of ordinary line breaks.

Visible characters such as:

- `U+2022` BULLET (`•`)

were **not automatically removed or replaced**, because they are visible punctuation/formatting characters rather than line terminators and may carry meaningful structure.

---

## 5.3 Whitespace normalization

The following formatting normalization was performed:

- non-breaking spaces (`U+00A0`) → ordinary spaces;
- Windows line endings (`\r\n`) → `\n`;
- remaining carriage returns (`\r`) → `\n`;
- tab characters (`\t`) → ordinary spaces;
- sequences of two or more ordinary spaces → one space;
- sequences of three or more newline characters → two newline characters;
- leading and trailing whitespace removed with `.strip()`.

Ordinary paragraph structure was retained.

The regular expression used to collapse repeated spaces targeted ordinary spaces rather than all whitespace, thereby avoiding accidental removal of newline structure.

---

## 5.4 Lowercase computational representation

A separate `text_lower` variable was created from `text_clean`.

This allows later lexical matching to treat forms such as:

`Protect`, `PROTECT`, and `protect`

as equivalent while preserving capitalization in both the original and cleaned text.

The three speech-text representations therefore serve different purposes:

- `text` = original imported text;
- `text_clean` = formatting-normalized text;
- `text_lower` = lowercase computational representation.

---

# 6. Post-Preprocessing Quality Control

After formatting normalization:

- missing focal texts: **0**
- missing development texts: **0**
- empty focal texts: **0**
- empty development texts: **0**

A whitespace-based cleaned word count was computed for each speech.

This measure is used for corpus auditing and descriptive purposes and should not be interpreted as a final linguistic token count.

---

## 6.1 Focal corpus speech length

- N = 180
- Mean = 2,171.38 words
- SD = 951.97
- Minimum = 611
- 25th percentile = 1,554
- Median = 1,978.5
- 75th percentile = 2,571.75
- Maximum = 8,242

**Total cleaned focal corpus size:** 390,849 words

---

## 6.2 Development corpus speech length

- N = 1,755
- Mean = 2,037.88 words
- SD = 702.20
- Minimum = 463
- 25th percentile = 1,580.5
- Median = 1,939
- 75th percentile = 2,398
- Maximum = 7,082

**Total cleaned development corpus size:** 3,576,476 words

---

# 7. Short-Speech Quality Check

The five shortest focal and development speeches were identified for inspection.

The shortest focal speech was:

- Indonesia;
- 2021;
- `IDN_76_2021.txt`;
- 611 cleaned whitespace-delimited words.

The beginning of this speech was manually inspected.

The file contained coherent continuous UN General Debate content rather than evidence of an empty, corrupted, or failed extraction.

Short speech length alone was therefore not treated as an exclusion criterion.

The shortest development speeches were also inspected through sentence-level quality-control procedures described below.

---

# 8. Sentence Segmentation

After speech-level formatting normalization, the cleaned speeches were segmented into sentences using spaCy with the English `en_core_web_sm` model.

The spaCy maximum document length was set to:

`nlp.max_length = 2_000_000`

to prevent document-length errors for unusually long speeches.

A sentence-segmentation function was defined to:

1. pass each complete `text_clean` speech through spaCy;
2. iterate over `doc.sents`;
3. remove leading and trailing whitespace from each detected sentence;
4. exclude empty sentence strings; and
5. preserve sentence order within each speech.

Each sentence was converted into one row of a new sentence-level DataFrame.

The following information was retained for each sentence:

- corpus membership;
- ISO3 country code;
- UN session;
- year;
- filename;
- sequential sentence ID within the speech;
- original-case sentence text;
- lowercase sentence text.

Sentence IDs began at 0 within each speech.

---

## 8.1 Initial sentence-level dataset sizes

Before removal of procedural introductions, the sentence-level datasets contained:

- **18,903 focal sentences**
- **157,525 development sentences**

No empty sentence strings were generated:

- empty focal sentences = **0**
- empty development sentences = **0**

---

## 8.2 Sentences per focal speech

- N = 180 speeches
- Mean = 105.02 sentences
- SD = 58.79
- Minimum = 31
- 25th percentile = 73.75
- Median = 93
- 75th percentile = 125
- Maximum = 635

---

## 8.3 Sentences per development speech

- N = 1,755 speeches
- Mean = 89.76 sentences
- SD = 36.89
- Minimum = 14
- 25th percentile = 66
- Median = 84
- 75th percentile = 106
- Maximum = 388

---

## 8.4 Sentence-segmentation quality control

Sentence segmentation was manually inspected in multiple speeches, including ordinary-length, unusually short, and unusually long cases.

A focal example from Indonesia 2021 showed that spaCy appropriately separated successive questions and declarative sentences while preserving long substantive sentences.

The development speech with the smallest number of detected sentences was:

- Guinea-Bissau;
- 2021;
- `GNB_76_2021.txt`;
- 14 detected sentences.

Inspection showed that these were coherent, relatively long sentences rather than evidence that spaCy had collapsed a malformed document.

The focal speech with the largest number of detected sentences was:

- United States;
- 2025;
- `USA_80_2025.txt`;
- 635 detected sentences.

Inspection showed that the unusually high count was not primarily caused by catastrophic sentence segmentation.

However, this inspection revealed that some UNGD files contained procedural statements spoken by the UN presiding officer before the national representative's substantive speech.

This prompted a separate procedural-text audit.

---

# 9. Procedural-Introduction Detection and Removal

## 9.1 Identification of non-substantive procedural material

Sentence-level quality control revealed that some UNGD text files included procedural introductory language spoken by the UN presiding officer immediately before the national representative's statement.

Examples included phrases such as:

- `"the assembly will hear an address"`
- `"i request protocol"`
- `"escort his excellency"`
- `"escort her excellency"`

These sentences are not part of the substantive national statement and should not be attributed to the country in lexical analysis.

A diagnostic search was therefore conducted within the first five sentence positions of each speech.

The following phrase frequencies were observed at the speech level:

- `"the assembly will hear an address"` → 75 speeches
- `"i request protocol"` → 117 speeches
- `"escort his excellency"` → 104 speeches
- `"escort her excellency"` → 10 speeches

Because these phrases appeared systematically across multiple speeches, they were treated as recurring procedural boilerplate rather than isolated anomalies.

---

## 9.2 Conservative procedural-introduction flag

A project-specific procedural-introduction flag was created.

A sentence was eligible for flagging only when:

1. it occurred within the first five sentence positions of a speech; and
2. it contained at least one of the empirically identified procedural phrases.

The positional restriction was used to reduce the risk of removing genuine substantive content elsewhere in a speech.

The flagged sentences occurred only at sentence positions 0–2.

### Focal corpus flagged positions

- sentence 0: 7
- sentence 1: 13
- sentence 2: 1

Total flagged focal procedural sentences:

**21**

### Development corpus flagged positions

- sentence 0: 69
- sentence 1: 107
- sentence 2: 3

Total flagged development procedural sentences:

**179**

The positional distributions were consistent with the interpretation that the flagged material represented introductions preceding the substantive national statements.

---

## 9.3 Removal and preservation of original data

Procedural-introduction sentences were excluded from newly created cleaned sentence-level DataFrames.

The original unfiltered sentence-level DataFrames were retained.

This allowed the procedural filtering procedure to remain auditable and reversible.

The procedure removed:

- **21 focal sentences**
- **179 development sentences**

The resulting cleaned sentence-level corpora contained:

- **18,882 focal sentences**
- **157,346 development sentences**

---

## 9.4 Sentence-ID renumbering

After procedural sentences were removed, sentence IDs were renumbered separately within each speech using sequential counts beginning at 0.

This ensures that later sentence-context extraction can reliably identify adjacent sentences.

For example, if sentence \(i\) contains a self-assertion hit, its local context can later be defined using:

\[
S_{i-1} + S_i + S_{i+1}
\]

without gaps created by previously removed procedural sentences.

---

# 10. Current Data Provenance and Processing Structure

The implemented pipeline currently follows:

`Raw UNGD .txt files`

→ `01_data_audit.ipynb`

→ structured and audited speech-level datasets

→ `02_preprocessing.ipynb`

→ Unicode and whitespace normalization

→ preservation of original, cleaned, and lowercase text representations

→ speech-level preprocessing quality control

→ spaCy sentence segmentation

→ sentence-level quality control

→ detection of presiding-officer procedural introductions

→ conservative procedural-introduction removal

→ within-speech sentence-ID renumbering

→ cleaned sentence-level datasets for downstream KGL development

The following levels of data are intentionally preserved:

1. original raw `.txt` source files;
2. structured speech-level data;
3. normalized speech-level text;
4. unfiltered sentence-level data;
5. cleaned sentence-level data after procedural-introduction removal.

This structure allows preprocessing decisions to be audited, revised, or reproduced without altering the original corpus.

---

# 11. Planned Seed-Lexicon Construction

Seed construction will begin with formal construct definitions rather than corpus-driven word selection.

For each construct, theoretically relevant papers will be searched for:

- construct labels;
- proposed mechanisms;
- behavioral manifestations; and
- theoretically proximal lexical expressions.

Candidate seeds will be classified as:

- **Type A:** construct labels;
- **Type B:** mechanism words;
- **Type C:** behavioral manifestations;
- **Type D:** consequences or contextual terms.

Types A–C will generally be preferred.

Type D terms will be used only when theoretically justified.

Candidate seeds should:

- directly express the construct;
- plausibly occur in natural UN discourse;
- avoid region labels;
- avoid purely academic jargon where possible;
- avoid words that are only distal consequences of the construct;
- avoid excessively generic or highly polysemous terms when possible.

The preliminary target is approximately **8–15 seeds per KGL**.

Seed evidence and decisions will be documented before embedding expansion.

The confirmatory seed sets will be frozen before inspection of focal-country substantive profiles.

---

# 12. Planned KGL Expansion and Purification

The initial implementation will follow the general Havaldar et al. structure:

## 12.1 Synonym expansion

For each seed term, nearest embedding neighbors will be retrieved using cosine similarity.

Planned primary similarity threshold:

**0.75**

Planned sensitivity thresholds:

- 0.70
- 0.80

---

## 12.2 Concept expansion

Seed embeddings will be averaged to create a construct centroid.

Nearest words to that centroid will then be retrieved using cosine similarity.

Planned primary threshold:

**0.45**

Planned sensitivity thresholds:

- 0.40
- 0.50

---

## 12.3 Lexicon weighting

Seed terms will receive a weight of:

**1.0**

Expanded terms will be weighted by their cosine similarity to the relevant seed or construct centroid.

The final expanded lexicon will consist of the union of:

- original seed terms;
- synonym-expanded terms; and
- concept-expanded terms.

---

## 12.4 Frequency purification

Rare terms will be removed before internal-correlation purification.

The current planned project-specific primary criterion is:

- at least **50 total occurrences** in the relevant development corpus; and
- occurrence in at least **20 development country-year documents**.

If this proves excessively restrictive, the predefined fallback criterion is:

- at least **25 total occurrences**; and
- occurrence in at least **10 development country-year documents**.

These frequency thresholds are project-specific operational choices rather than universal thresholds specified by the original KGL method.

They will be evaluated before substantive focal-country results are inspected.

---

## 12.5 Internal-correlation purification

For each candidate lexicon word \(w_i\), weighted frequency across development units will be correlated with the summed weighted frequency of the remaining lexicon:

\[
r\left(
F(w_i),
\sum_{w \in L \setminus w_i} F(w)
\right)
\]

Planned primary retention threshold:

\[
r \geq .15
\]

Planned sensitivity thresholds:

- \(r > 0\)
- \(r \geq .05\)
- \(r \geq .10\)
- \(r \geq .20\)

---

# 13. Planned Lexicon Overlap Handling

Lexical overlap between Self-Assertion and framing lexica will be treated conservatively.

If a term appears in both:

- Self-Assertion; and
- a framing KGL,

the current primary plan is to retain it in Self-Assertion and remove it from the framing lexicon for primary scoring.

If a term appears in two framing lexica, assignment will be based on relative construct-centroid similarity.

Current planned rule:

- if the cosine-similarity difference is at least **0.05**, assign the term to the closer framing construct;
- if the difference is less than **0.05**, mark the term as ambiguous and exclude it from primary framing scoring.

The `.05` criterion is project-specific and will be documented as such.

---

# 14. Planned Validation and Sensitivity Analyses

Before focal-country interpretation, the lexica will undergo several forms of quality control.

Planned checks include:

- seed-only baseline comparisons;
- expansion-threshold sensitivity analyses;
- purification-threshold sensitivity analyses;
- inspection of candidate terms for semantic plausibility;
- limited blinded inspection of high-scoring development contexts;
- lexical-overlap auditing.

Focal-country or regional labels will not be used to tune the lexica toward theoretically preferred results.

Lexica will be frozen before substantive focal-country profile inspection.

---

# 15. Planned Self-Assertion and Framing Scoring

## 15.1 Speech-level self-assertion

Self-assertion will be scored using weighted lexical frequency.

A general form is:

\[
1000
\times
\frac{
\sum_w
\text{weight}_w
\times
\text{count}_{wd}
}{
\text{tokens}_d
}
\]

where:

- \(w\) indexes lexicon terms;
- \(d\) indexes a speech or analytic text unit;
- `weight` represents the KGL weight assigned to the term.

---

## 15.2 Self-assertion context extraction

Sentences containing one or more Self-Assertion KGL hits will serve as focal sentences.

The planned local context is:

\[
S_{i-1} + S_i + S_{i+1}
\]

where available.

Overlapping context windows will be merged so the same text is not redundantly scored multiple times.

---

## 15.3 Conditional framing analysis

The four framing KGLs will be scored **within self-assertion contexts**, rather than indiscriminately across entire speeches.

This operationalizes the primary question:

> When self-assertion occurs, how is it lexically framed?

The four planned dimensions are:

- Self-Oriented Framing;
- Ingroup-Protective Framing;
- Collective-Advancement Framing;
- Relational/Negotiation Framing.

---

# 16. Planned Aggregation

The intended hierarchy is:

`context`

→ `speech`

→ `country`

→ `region`

Self-assertion prevalence will be calculated at the speech level.

Framing scores will be calculated only for speeches containing at least one self-assertion context.

A speech with no self-assertion contexts will therefore contribute information to self-assertion prevalence but will **not** automatically receive zero scores on all framing dimensions.

Country-level scores will summarize the available speech-level estimates.

The number of contributing speeches out of ten will be reported for framing measures.

Regional estimates will average the two selected constituent-country scores and will be treated descriptively.

Because there are only two countries per region, regional aggregates will not be interpreted as exhaustive estimates of entire cultural regions.

---

# 17. Planned Cross-Framing Visualization

Raw KGL magnitudes may not be directly comparable across constructs because lexica can differ in:

- number of terms;
- similarity weights;
- corpus frequency.

Therefore, cross-frame profile visualizations will not interpret raw differences between KGL scales as directly equivalent quantities.

For profile visualization, framing dimensions may be standardized across the 18 country scores before comparison.

Within-KGL comparisons across countries remain the primary interpretable comparisons.

---

# 18. Planned Residual Discovery Analysis

The project includes a bounded exploratory residual-discovery stage after all five main KGLs have been frozen.

The purpose is to identify recurring semantic patterns in self-assertion contexts that are poorly captured by the four theory-derived framing lexica.

---

## 18.1 Residual definition

For each self-assertion context, coverage by the four framing KGLs will be evaluated.

A strict residual context will be defined as:

- Self-Assertion detected; and
- zero detected terms from all four framing KGLs.

Conceptually:

\[
\text{FrameCoverage}_p =
I(\text{Self-Oriented}>0)
+
I(\text{Protective}>0)
+
I(\text{Collective}>0)
+
I(\text{Relational}>0)
\]

A strict residual passage satisfies:

\[
\text{FrameCoverage}_p = 0
\]

---

## 18.2 Residual sample-size rule

The current planned decision rule is:

- residual N ≥ 40 → conduct clustering;
- residual N = 20–39 → conduct cautious small-sample exploratory clustering;
- residual N < 20 → do not force clustering; report insufficient residual material.

---

## 18.3 Residual embedding and clustering

Residual passages will be embedded using a contextual sentence-embedding model such as:

`all-MiniLM-L6-v2`

Clustering will use HDBSCAN.

Country and region labels will **not** be included as clustering features.

The planned primary parameterization is approximately:

\[
\text{min\_cluster\_size}
=
\max(5,\text{round}(.05N))
\]

and:

\[
\text{min\_samples}
=
\max(3,\text{round}(.5 \times \text{min\_cluster\_size}))
\]

Sensitivity analyses will examine minimum-cluster-size settings of approximately:

- 5% of N;
- 7.5% of N;
- 10% of N.

Cluster stability across specifications will be evaluated using Jaccard overlap.

A cluster with approximately:

\[
J \geq .50
\]

under an alternative specification will be treated as reasonably stable for exploratory purposes.

---

## 18.4 Residual cluster interpretation

For each candidate cluster:

1. cluster-specific TF-IDF terms will be identified;
2. approximately 10–15 high-weight lexical terms will be inspected;
3. approximately five passages nearest the cluster centroid or medoid will be examined;
4. country, region, and year labels will initially be hidden during semantic interpretation;
5. geographic composition will be examined only after a tentative semantic label has been assigned.

A cluster will be considered a candidate novel framing only when it is:

- semantically interpretable;
- reasonably stable across clustering specifications;
- represented by more than one speech;
- not primarily UN procedural boilerplate;
- not simply a disguised version of one of the four theory-derived framing dimensions.

Where possible, region-level interpretation will require evidence from both selected country cases rather than a single country.

Residual clusters will be treated as **candidate constructs for future research**, not added post hoc as new confirmatory KGLs in the present analysis.

---

# 19. Methodological Interpretation Rules

Several interpretation constraints apply throughout the project.

## 19.1 Lexical framing is not psychological motive

A KGL score indicates the presence of language associated with a construct.

It does not establish the speaker's underlying psychological motive.

Preferred interpretation:

> Self-assertion was more strongly surrounded by ingroup-protective framing.

Avoid:

> The country asserted itself because it psychologically wanted to protect the ingroup.

---

## 19.2 Institutional discourse is not individual behavior

The unit of discourse is a national representative speaking in an institutional setting.

Self-assertion therefore refers to **institutional or national self-assertive discourse**, not direct measurement of individual-level assertiveness.

---

## 19.3 Country cases do not exhaustively represent regions

The two selected countries within each region provide theoretically informative comparative cases.

They should not be described as statistically representative of all societies within the corresponding region.

---

## 19.4 Exploratory discovery remains separate from confirmatory measurement

Theory-derived KGL construction and residual semantic discovery are treated as distinct stages.

Residual clusters will not be retroactively incorporated into the primary KGL system in a way that changes the already-completed focal analyses.

---

# 20. Software and Reproducibility

The project is implemented in Python.

Current environment:

- Python 3.11.9
- project-specific `.venv`
- Visual Studio Code
- Jupyter notebooks
- Git/GitHub for version control

Current major packages include:

- pandas
- numpy
- scipy
- scikit-learn
- gensim
- spaCy

Later stages are expected to use:

- sentence-transformers
- hdbscan

The notebook structure currently includes:

- `01_data_audit.ipynb`
- `02_preprocessing.ipynb`
- `03_seed_lexica.ipynb`
- `04_expansion.ipynb`
- `05_purification.ipynb`
- `06_validation.ipynb`
- `07_focal_scoring.ipynb`
- `08_figures.ipynb`

A later residual-discovery notebook may be added if needed.

Generated data files are excluded from Git version control where they can be reproducibly recreated from the notebooks.

Code, notebooks, methodological documentation, and other reproducible project materials are version controlled.

---

# 21. Decision-Logging Principle

For every major methodological decision, record:

- what was done;
- why it was done;
- whether it follows the source method directly or is project-specific;
- numerical thresholds used;
- quality-control results;
- sensitivity analyses;
- unexpected corpus issues;
- deviations from the original plan;
- any decision made after inspecting data and whether focal results had already been viewed.

This document is a methodological working log rather than manuscript-ready Methods prose.

The purpose is to preserve the complete analytical history so that the eventual manuscript Methods section can be written from an accurate record rather than reconstructed from memory.