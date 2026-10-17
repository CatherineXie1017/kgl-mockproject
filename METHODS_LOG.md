# Methodology and Decision Log

## Project Overview

**Working title:**  
*What Does Self-Assertion Mean Across Cultures? A Knowledge-Guided and Exploratory Computational Analysis of Self-Assertion in Global Institutional Discourse*

**Corpus:** UN General Debate Corpus (UNGDC)

**Analysis period:** 2016–2025

**Current completed measurement phase:** Institutional Self-Assertion KGL

The project examines how national actors express **Institutional Self-Assertion** in UN General Debate discourse and how instances of Self-Assertion are framed across cultural contexts.

The project separates the corpus into:

- a **development corpus**, used to construct, audit, expand, and purify the lexical measurement instruments; and
- a **focal corpus**, reserved for the substantive cross-country and cross-regional analysis.

The focal corpus is not used to construct or modify the lexical measures. This separation reduces the risk of adapting the measurement instrument in response to patterns observed in the focal countries.

The broader project is organized around five Knowledge-Guided Lexica (KGLs):

1. **Self-Assertion**
2. **Self-Oriented Framing**
3. **Ingroup-Protective Framing**
4. **Collective-Advancement Framing**
5. **Relational/Negotiation Framing**

The Self-Assertion KGL functions as the primary retrieval lexicon. The four framing KGLs will ultimately characterize the lexical context surrounding detected Self-Assertion.

The project therefore distinguishes:

> **whether Self-Assertion occurs**

from:

> **how Self-Assertion is framed when it occurs**

The lexical measures are interpreted as indicators of discourse patterns rather than direct measures of speakers' internal psychological states or motives.

As of the current Method Log, the **Self-Assertion KGL has been fully constructed and frozen**. The framing KGLs have not yet been constructed.

---

# 1. Corpus Definition

## 1.1 Analysis period

The analysis is restricted to speeches from **2016 through 2025**.

The project is intended to characterize relatively recent institutional discourse rather than long-run historical change.

Year therefore primarily provides repeated observations of each country's recent discourse rather than functioning as the project's main substantive longitudinal predictor.

---

## 1.2 Focal corpus

The focal corpus consists of **18 countries**, with two selected country cases from each of nine cultural regions.

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

Each focal country contributes one UN General Debate speech per year from 2016 through 2025.

Thus:

\[
18 \text{ countries} \times 10 \text{ years}
=
180 \text{ speeches}
\]

**Focal corpus size: 180 speeches**

The regional labels are used as theoretically informative descriptive groupings. The two selected countries are not assumed to exhaustively or statistically represent the broader region.

---

## 1.3 Development corpus

All available 2016–2025 speeches from countries outside the 18-country focal set were assigned to the development corpus.

**Development corpus size: 1,755 speeches**

The development corpus was used for:

- construct-grounded seed evaluation;
- contextual seed auditing;
- conceptual coverage auditing;
- FastText expansion;
- corpus-frequency purification;
- contextual quality control;
- semantic purification;
- internal-correlation purification.

The focal corpus remained separate throughout Self-Assertion KGL construction.

Conceptually:

```text
Development corpus
        ↓
construct and purify measurement instrument
        ↓
freeze measurement instrument
        ↓
Focal corpus
        ↓
substantive analysis
```

---

# 2. Raw Data Structure and Parsing

UN General Debate text files followed the filename structure:

```text
ISO3_SESSION_YEAR.txt
```

For example:

```text
AFG_71_2016.txt
```

where:

- `AFG` identifies the country;
- `71` identifies the UN General Assembly session;
- `2016` identifies the speech year.

Python recursively identified the raw `.txt` files and imported each speech into one row of a pandas DataFrame.

For each speech, the following information was retained:

- corpus membership;
- ISO3 country code;
- UN session;
- year;
- filename;
- filepath;
- complete speech text;
- preliminary speech length.

Separate structured DataFrames were created for the focal and development corpora.

---

# 3. Corpus Audit

Before substantive preprocessing, the corpus was audited to verify its structure and completeness.

The audit confirmed:

1. the focal corpus contained exactly the intended 18 country codes;
2. each focal country contributed exactly 10 speeches;
3. all years from 2016 through 2025 were represented for each focal country;
4. every focal country-year combination appeared exactly once;
5. no focal country remained in the development corpus;
6. no focal speech had missing text;
7. no development speech had missing text;
8. no focal speech had empty text;
9. no development speech had empty text;
10. unusually short speeches reflected coherent substantive UNGD content rather than failed extraction.

The final speech-level corpus sizes were therefore:

- **Focal corpus: 180 speeches**
- **Development corpus: 1,755 speeches**

---

# 4. Text Preprocessing

Preprocessing was intentionally conservative.

The goal was to standardize formatting and computational representation while avoiding unnecessary removal or alteration of lexical content that might later be relevant to KGL construction.

---

## 4.1 Unicode normalization

Speech text was normalized using Unicode NFC normalization:

```python
unicodedata.normalize("NFC", text)
```

This standardizes alternative underlying Unicode representations of visually equivalent characters without removing accents or changing lexical content.

---

## 4.2 Formatting normalization

The following transformations were applied:

- Unicode line separators were normalized to standard newline characters;
- non-breaking spaces were converted to ordinary spaces;
- Windows line endings were converted to `\n`;
- remaining carriage returns were converted to `\n`;
- tabs were converted to ordinary spaces;
- repeated ordinary spaces were collapsed;
- excessive blank lines were collapsed;
- leading and trailing whitespace was removed.

Ordinary paragraph structure and visible lexical content were retained.

---

## 4.3 Text representations

Three text representations were retained:

- `text` = original imported speech text;
- `text_clean` = formatting-normalized text;
- `text_lower` = lowercase computational representation.

This allows lexical matching to treat forms such as:

```text
Protect
PROTECT
protect
```

as equivalent while preserving original capitalization for contextual inspection.

---

# 5. Corpus Size After Preprocessing

Using cleaned whitespace-delimited word counts:

## Focal corpus

- N = 180 speeches
- Mean = 2,171.38 words
- SD = 951.97
- Minimum = 611
- Median = 1,978.5
- Maximum = 8,242

**Total cleaned focal corpus size: 390,849 words**

## Development corpus

- N = 1,755 speeches
- Mean = 2,037.88 words
- SD = 702.20
- Minimum = 463
- Median = 1,939
- Maximum = 7,082

**Total cleaned development corpus size: 3,576,476 words**

These word counts were used for corpus auditing and description rather than as final linguistic token counts.

---

# 6. Sentence Segmentation

The cleaned speeches were segmented into sentences using spaCy with:

```text
en_core_web_sm
```

The spaCy maximum document length was increased to:

```python
nlp.max_length = 2_000_000
```

to prevent errors for unusually long speeches.

Each detected sentence was converted into one row of a sentence-level DataFrame.

The following information was retained:

- corpus;
- ISO3 country code;
- UN session;
- year;
- filename;
- sentence ID;
- original-case sentence text;
- lowercase sentence text.

Sentence IDs initially began at 0 within each speech.

Before procedural-text removal, the sentence-level corpora contained:

- **18,903 focal sentences**
- **157,525 development sentences**

No empty sentence strings were generated.

Manual inspection of ordinary, unusually short, and unusually long speeches indicated that the sentence segmentation was sufficiently reliable for the planned contextual lexical analyses.

---

# 7. Procedural-Introduction Detection and Removal

Sentence-level inspection revealed that some UNGD files contained procedural introductory statements spoken by the UN presiding officer immediately before the national representative's substantive speech.

Recurring procedural phrases included:

```text
the assembly will hear an address
i request protocol
escort his excellency
escort her excellency
```

These statements should not be attributed to the focal national actor.

A conservative procedural-introduction rule was therefore implemented.

A sentence was eligible for removal only when:

1. it occurred within the first five sentence positions of a speech; and
2. it contained one of the empirically identified procedural phrases.

The positional restriction reduced the risk of deleting legitimate substantive language that happened to contain similar vocabulary later in a speech.

The procedure removed:

- **21 focal procedural sentences**
- **179 development procedural sentences**

The resulting cleaned sentence corpora contained:

- **18,882 focal sentences**
- **157,346 development sentences**

After removal, sentence IDs were renumbered sequentially within each speech.

This allows contextual extraction around a target sentence \(S_i\) using:

\[
S_{i-1} + S_i + S_{i+1}
\]

without gaps created by removed procedural material.

Primary cleaned sentence files:

```text
data/preprocessed/focal_sentences_clean.csv
data/preprocessed/development_sentences_clean.csv
```

---

# 8. Data Provenance

The implemented data-processing pipeline is:

```text
Raw UNGD .txt files
        ↓
speech-level parsing and corpus audit
        ↓
Unicode and whitespace normalization
        ↓
preservation of original, cleaned, and lowercase text
        ↓
spaCy sentence segmentation
        ↓
sentence-level quality control
        ↓
procedural-introduction removal
        ↓
within-speech sentence-ID renumbering
        ↓
cleaned sentence-level corpora
```

The original source files remain unchanged.

The cleaned sentence-level datasets form the basis of downstream KGL development.

---

# 9. Institutional Self-Assertion Construct Development

Institutional Self-Assertion was defined **before final seed selection and before embedding expansion**.

This order was used so that the lexical measure would be grounded in a substantive theoretical definition rather than defined retrospectively by the vocabulary returned by an embedding model.

---

## 9.1 Construct definition

The final definition is:

> **Institutional Self-Assertion is language in which a focal national actor actively positions itself by expressing or enacting its own agency with respect to its positions, goals, interests, rights, preferences, autonomy, capabilities, roles, or intended courses of action. This can include advancing, defending, maintaining, claiming, exercising, committing to, or seeking to influence outcomes concerning these self-relevant positions or actions, as well as resisting external constraints on them.**

The corresponding scope restriction is:

> **The construct concerns agentive self-positioning rather than general action, positive self-evaluation, aggression, generic stance-taking, or factual self-description. Self-assertion may be cooperative, competitive, protective, self-promotive, or relational in function; these functions are conceptually separate from the assertive form itself.**

The conceptual core is therefore:

> **focal national actor + active self-positioning + self-relevant agency**

---

## 9.2 Focal actor rule

The relevant position, goal, right, interest, intended action, capability, or role must belong to the focal country or government giving the speech.

Explicit first-person language such as `we` is not required if the focal actor can be recovered from nearby discourse context.

An explicit opposing actor is also not required.

---

## 9.3 Agentive self-positioning

The focal national actor must actively indicate one or more of the following:

- where it stands;
- what it wants;
- what it intends to do;
- what right or interest it advances;
- what position it maintains;
- what capability or role it claims or exercises;
- what outcome it seeks to influence;
- what external pressure or constraint it resists.

---

## 9.4 Exclusions

The following are insufficient by themselves:

- factual self-description;
- general action;
- generic stance-taking;
- generic linguistic assertion;
- positive self-evaluation;
- self-enhancement;
- aggression.

The relevant lexical expression must indicate **active self-positioning** rather than merely describe the focal country.

---

## 9.5 Correct lexical sense

A candidate lexical item is relevant only when it is being used in the semantic sense corresponding to Institutional Self-Assertion.

This rule is particularly important for polysemous terms.

---

## 9.6 Form/function separation

Self-Assertion is conceptually separated from its hypothesized cultural framing or function.

Self-Assertion may later be framed as:

- self-oriented;
- ingroup-protective;
- collective-advancement;
- relational/negotiative.

However, vocabulary corresponding to one of these functions does not qualify as Self-Assertion unless active focal-national self-positioning is also present.

---

# 10. Initial Self-Assertion Candidate Seeds

An initial theory-guided candidate pool contained 20 terms:

```text
self-assertion
assertiveness
assertive
reject
resist
advocate
propose
pursue
influence
defend
demand
reaffirm
insist
assert
urge
refuse
oppose
uphold
claim
declare
```

These were treated as candidate seeds rather than automatically accepted lexical indicators.

---

# 11. Contextual Audit of Initial Candidate Seeds

Up to 20 occurrences of each initial candidate were randomly sampled from the cleaned development corpus.

This produced:

**346 manually evaluated contexts**

The target sentence was examined together with its immediately preceding and following sentence when available.

The contextual unit was therefore:

\[
S_{i-1}+S_i+S_{i+1}
\]

Coding considered:

- overall construct relevance;
- actor type;
- whether agency was present;
- whether a focal self-relevant position or object was involved;
- whether the lexical sense was appropriate;
- whether the expression represented recurring diplomatic boilerplate.

Actor type was distinguished as:

- focal;
- coalition;
- global collective;
- third party;
- unclear.

Contextual precision was treated as **diagnostic evidence rather than as an automatic seed inclusion threshold**.

Poor performance could reflect different problems:

- unrelated lexical sense;
- polysemy;
- third-party rather than focal-country agency;
- excessively generic usage;
- or genuine theoretical relevance accompanied by contextual ambiguity.

No universal numerical seed-level precision rule was imposed.

---

# 12. Conceptual Facet Framework

To evaluate whether the candidate vocabulary represented the full construct, Institutional Self-Assertion was decomposed into behavioral facets.

The working facets were:

1. `advance_outcome`
2. `claim_position`
3. `defend_self_relevant`
4. `maintain_position`
5. `resist_constraint`
6. `commit_intended_action`
7. `exercise_capability_role`
8. `influence_outcome`

Direct construct terms were tracked separately as:

```text
construct_label
```

This facet framework was used to identify whether conceptually important types of Self-Assertion were absent from the initial candidate vocabulary.

---

# 13. Independent Construct-Coverage Audit

Auditing only sentences containing candidate vocabulary can estimate how candidates behave but cannot identify Self-Assertion expressions that the current candidate vocabulary completely misses.

An independent sample of:

**250 development-corpus contexts**

was therefore randomly selected without filtering on the existing seed list.

Each context was coded for:

- whether clear Institutional Self-Assertion occurred;
- primary behavioral facet;
- key lexical expression;
- whether the current seed families would capture it.

The audit identified:

- **66 clear Self-Assertion contexts**
- **181 non-Self-Assertion contexts**
- **3 ambiguous contexts**

The initial seed families provided good coverage of observed examples involving:

- advancing outcomes;
- claiming or expressing positions;
- defending self-relevant objects;
- maintaining positions;
- resisting constraints;
- influencing outcomes.

Two systematic gaps were identified.

---

## 13.1 Commitment and intended action

The existing candidate pool weakly represented statements involving intended future action or commitment.

Recurring expressions included:

```text
intend to
stands ready
readiness to
committed to
will continue
```

---

## 13.2 Exercise of capability or institutional role

The initial vocabulary also weakly represented enactment of:

- institutional role;
- authority;
- autonomy;
- participation;
- mediation;
- national capability.

A targeted candidate-development stage was therefore conducted for these two facets.

---

# 14. Gap-Targeted Candidate Audit

Candidates examined for **commitment/intended action** were:

```text
commit
intend
ready
continue
pledge
```

Candidates examined for **exercise of capability/role** were:

```text
exercise
participate
engage
mediate
lead
```

Up to 20 occurrences per term were sampled and manually inspected.

Because `mediate` occurred only nine times, all available occurrences were examined.

The resulting targeted audit contained:

**189 contexts**

The audit identified `intend` and `ready` as relatively clean anchors for commitment/intended action.

No comparably clean unigram emerged for the capability/role facet.

`exercise` was nevertheless retained provisionally as the broadest theoretically relevant anchor for that facet, subject to later purification.

---

# 15. Final Seed-Selection Criteria

Final seed decisions considered jointly:

1. theoretical centrality;
2. contextual relevance;
3. semantic distinctiveness;
4. coverage of the construct definition;
5. polysemy;
6. actor ambiguity;
7. morphological redundancy;
8. form/function separation.

Seed selection was therefore not determined by one empirical precision threshold.

---

# 16. Morphological Redundancy

The direct construct family initially contained:

```text
self-assertion
assertiveness
assertive
assert
```

Retaining all four would give closely related morphological variants disproportionate influence during construction of the concept centroid.

The final seed set therefore retained:

```text
self-assertion
assert
```

and did not retain:

```text
assertiveness
assertive
```

as independent seeds.

---

# 17. Frozen Self-Assertion Seed Set

The frozen Self-Assertion seed set contained **14 terms**.

| Conceptual role | Seeds |
|---|---|
| Direct construct / assertion | `self-assertion`, `assert` |
| Advance or influence outcome | `urge`, `propose`, `advocate` |
| Explicit institutional positioning | `declare` |
| Defend self-relevant position/object | `defend` |
| Maintain position | `reaffirm`, `insist` |
| Resist external constraint | `reject`, `resist` |
| Commit/intend action | `intend`, `ready` |
| Exercise capability/role | `exercise` |

Full frozen set:

```text
self-assertion
assert
urge
propose
advocate
declare
defend
reaffirm
insist
reject
resist
intend
ready
exercise
```

The seed set was finalized **before FastText expansion results were inspected**.

Seed file:

```text
data/seed_candidates/self_assertion_seed_set_v1.csv
```

---

# 18. FastText Knowledge-Guided Expansion

The Self-Assertion vocabulary was expanded following the general Knowledge-Guided Lexical Analysis logic of Havaldar et al. (2024), adapted to the present actor-dependent construct.

The pretrained embedding model used was:

```text
fasttext-wiki-news-subwords-300
```

with 300-dimensional word vectors.

All 14 seeds had available FastText representations.

Two parallel expansion routes were implemented:

1. synonym expansion;
2. concept expansion.

---

# 19. Synonym Expansion

For each seed \(s\), nearby FastText terms were identified using cosine similarity:

\[
\cos(v_s,v_w)
\]

The retained synonym-expansion threshold was:

\[
\boxed{.75}
\]

Thus, candidate terms were retained through the synonym-expansion route when:

\[
\cos(v_s,v_w)\geq .75
\]

The similarity between each expanded term and its associated seed was retained for weighting.

---

# 20. Concept Expansion

Concept expansion was intended to retrieve terms related to the overall semantic meaning of the seed set rather than to any one individual seed.

The **raw FastText vectors** of all 14 seeds were averaged:

\[
C=
\frac{1}{n}
\sum_{i=1}^{n}v_i
\]

where \(C\) represents the Self-Assertion concept centroid.

Vocabulary terms were then ranked by cosine similarity with \(C\).

Concept-expansion sensitivity was examined across:

\[
.45,\ .50,\ .55,\ .60,\ .65,\ .70
\]

Lower thresholds produced extremely broad vocabularies with substantial semantic diffusion.

The retained concept-expansion threshold was:

\[
\boxed{.60}
\]

At this threshold, concept expansion produced a sufficiently broad candidate vocabulary while remaining tractable for corpus-based purification.

---

# 21. Lexical Weighting

Original theory seeds received:

\[
weight=1
\]

Expanded terms received:

\[
weight=\text{cosine similarity}
\]

When a word was generated by multiple expansion routes, its source information was retained and the maximum applicable similarity weight was used.

The expanded vocabulary was then subjected to corpus-based purification.

---

# 22. Tokenization for KGL Purification

Lexical matching used lowercase tokenized text.

The tokenizer was:

```python
TOKEN_PATTERN = re.compile(r"[a-z]+(?:-[a-z]+)*")
```

This matches alphabetic words and hyphenated alphabetic forms while avoiding substring matching.

For example:

```text
assert
self-assertion
reaffirm
```

are treated as lexical tokens.

The same tokenization logic was used throughout the purification procedures to maintain consistency.

---

# 23. Self-Assertion Purification Strategy

Embedding proximity alone does not establish that a word is a usable lexical indicator of Institutional Self-Assertion.

The expanded vocabulary was therefore purified sequentially using:

1. corpus-frequency purification;
2. conservative hard function-word exclusion;
3. development-context evidence screening;
4. country-level internal-correlation screening;
5. full term-level contextual semantic purification;
6. recalculation of internal correlations after semantic purification;
7. final combined semantic/coherence inclusion rule.

The final implemented path was:

```text
FastText-expanded vocabulary
        ↓
frequency purification
        ↓
hard function-word exclusion
        ↓
development-context evidence screen
        ↓
country-level internal-correlation screen
        ↓
328 candidate terms
        ↓
full contextual semantic purification
        ↓
69 semantically retained terms
        ↓
recompute internal correlations
        ↓
final inclusion rule
        ↓
62-term final Self-Assertion KGL
```

---

# 24. Frequency Purification

Candidate terms that are extremely rare in the target corpus provide unstable evidence and contribute little to aggregate lexical measurement.

Expanded terms were therefore required to occur at least:

\[
\boxed{10}
\]

times in the development corpus.

Theory-derived seeds were initially protected from exclusion at this stage.

Using:

- synonym threshold = `.75`;
- concept threshold = `.60`;
- development token count ≥ 10;
- protected seeds;

produced:

\[
543
\]

candidate terms before hard function-word exclusion.

---

# 25. Hard Function-Word Exclusion

Embedding expansion produced a small number of high-frequency grammatical or discourse-functional words with no plausible substantive interpretation as Institutional Self-Assertion indicators.

A conservative hard exclusion list was therefore applied:

```text
and
to
that
have
all
will
not
must
can
they
them
would
should
cannot
therefore
both
still
always
never
could
does
indeed
might
either
really
otherwise
please
```

Only terms judged clearly grammatical or function-like were included in this exclusion set.

This reduced:

\[
543 \rightarrow 516
\]

candidate terms.

---

# 26. Development-Context Evidence Screen

The independent 250-context audit provided a development set that had not been generated by searching for particular lexical candidates.

Of these contexts:

- 66 were clear Self-Assertion;
- 181 were clear non-Self-Assertion;
- 3 were ambiguous.

The three ambiguous contexts were excluded from quantitative retrieval diagnostics.

Thus:

\[
N=247
\]

clear development contexts were used in this stage.

The contextual unit was:

\[
S_{i-1}+S_i+S_{i+1}
\]

because actor identity and self-relevant agency can be established by adjacent sentences rather than the target sentence alone.

For each candidate term, occurrences within these 247 contexts were compared with the human Self-Assertion coding.

A conservative evidence screen was applied to **non-seed terms**:

- terms with fewer than 5 appearances in the audited contexts were retained because evidence was insufficient for reliable exclusion;
- terms with at least 5 appearances were excluded when no more than 25% of those appearances occurred in human-coded Self-Assertion contexts;
- terms exceeding this criterion were retained;
- theory-derived seeds remained protected at this stage.

This stage was deliberately permissive.

Its purpose was to remove candidates with clear empirical evidence of poor contextual specificity rather than to define the final lexicon.

The vocabulary was reduced from:

\[
516 \rightarrow 498
\]

terms.

---

# 27. Country-Level Internal-Correlation Screening

The remaining 498 terms were then evaluated for internal lexical coherence across geographic units.

---

## 27.1 Geographic units

The development corpus represented:

\[
178
\]

countries in the country-level analysis.

Each country constituted one geographic unit for internal-correlation calculation.

---

## 27.2 Country-size normalization

Countries contributed unequal amounts of text.

Raw term counts would therefore partly capture corpus volume rather than relative lexical usage.

For each country \(c\) and term \(w\), relative frequency was calculated as:

\[
RF(w,c)=
\frac{\text{count}(w,c)}
{\text{total development tokens}(c)}
\]

The normalized relative frequency was then multiplied by the term's KGL weight:

\[
WF(w,c)=RF(w,c)\times weight_w
\]

This country-token normalization is a project-specific adaptation.

It was used to reduce confounding between lexical frequency and unequal amounts of available country text.

---

## 27.3 Internal-correlation statistic

For each term \(w_i\), Pearson's correlation was calculated across countries between:

1. its own weighted normalized country frequency; and
2. the summed weighted normalized frequency of all remaining candidate terms.

Formally:

\[
r_i=
cor
\left(
WF(w_i),
\sum_{w_j\in L,\ j\neq i}WF(w_j)
\right)
\]

This statistic evaluates whether a word tends to be more frequent in countries where the broader candidate Self-Assertion vocabulary is also more frequent.

It should be interpreted as an **internal lexical coherence statistic**.

It is not:

- semantic similarity;
- contextual precision;
- a probability that an occurrence represents Self-Assertion;
- or a direct measure of psychological validity.

At this stage, a positive correlation was used as the primary internal-coherence screen, while theory seeds remained provisionally protected.

This produced:

\[
328
\]

candidate terms for full contextual semantic review.

Candidate file:

```text
data/purification/self_assertion_kgl_candidate_final.csv
```

This 328-term vocabulary was explicitly treated as an **intermediate candidate set**, not the final KGL.

---

# 28. Full Term-Level Contextual Semantic Purification

Internal correlation cannot by itself establish whether a lexical item actually expresses Institutional Self-Assertion.

A term may correlate with the broader lexical vocabulary because of:

- topic;
- institutional register;
- rhetorical style;
- general diplomatic language;
- or association with assertive passages,

without itself functioning as an indicator of focal-national self-positioning.

A stringent contextual semantic review was therefore conducted for **all 328 candidate terms**.

---

## 28.1 Context sampling

For each candidate term:

- up to five development-corpus occurrences were sampled;
- sampling preferentially selected occurrences from different speeches;
- the target sentence was examined with the immediately preceding and following sentences where available.

The contextual unit was:

\[
S_{i-1}+S_i+S_{i+1}
\]

A fixed random state was used:

```text
20260921
```

The resulting semantic-review dataset contained:

\[
1,636
\]

term-context observations.

Of the 328 candidate terms:

- 327 had five sampled contexts;
- `self-assertion` had one available occurrence in the development corpus.

Review file:

```text
data/purification/self_assertion_semantic_term_review.csv
```

---

# 29. Semantic Purification Decision Rule

The central question for each term was:

> **Does the target lexical item itself provide meaningful evidence of active focal-national self-positioning in its observed UNGD usages?**

This is intentionally stricter than asking whether the surrounding passage contains Self-Assertion.

A term could be retained when its lexical usage itself meaningfully indicated that the focal country was:

- advancing a position or desired outcome;
- asserting or expressing a position;
- maintaining or reaffirming a position;
- resisting or rejecting an external constraint;
- defending a self-relevant object;
- committing to intended action;
- or otherwise actively positioning itself.

Terms were excluded when they functioned primarily as:

- generic actions;
- generic verbs;
- descriptive vocabulary;
- topical vocabulary;
- discourse-management language;
- highly context-dependent expressions;
- semantically diffuse terms;
- or words whose dominant UNGD use did not itself provide evidence of Self-Assertion.

Theory seeds were **not automatically protected during this semantic stage**.

A theory seed therefore had to survive the same substantive semantic review as expanded terms.

---

# 30. Semantic Review and Adjudication

The initial term-level contextual review categorized each of the 328 terms as:

- **KEEP**
- **UNCERTAIN**
- **EXCLUDE**

The initial distribution was:

- **59 KEEP**
- **23 UNCERTAIN**
- **246 EXCLUDE**

The 23 uncertain terms were then adjudicated using the same construct definition and contextual evidence.

Ten uncertain terms were ultimately retained:

```text
call
confirm
decide
decided
favor
favour
recommend
request
requested
withdraw
```

Thirteen uncertain terms were excluded:

```text
accede
claim
claiming
encourage
enact
exercise
negotiate
promise
ratify
seek
seeking
support
supported
```

The final semantic decisions were therefore:

\[
69\ \text{KEEP}
\]

and

\[
259\ \text{EXCLUDE}
\]

for all:

\[
328
\]

reviewed terms.

Final semantic-review file:

```text
data/purification/self_assertion_semantic_term_review_FINAL.csv
```

---

# 31. Removal of `exercise`

`exercise` was one of the original 14 theory seeds because it was intended to anchor the capability/role facet identified in the independent coverage audit.

However, theory-seed status did not guarantee final inclusion.

During the full contextual semantic purification, `exercise` did not provide sufficiently reliable lexical evidence of focal-national self-positioning in its sampled UNGD uses.

It was therefore excluded.

Consequently:

- original frozen seed set = 14 terms;
- seeds surviving contextual semantic purification = 13 terms.

This illustrates the distinction between:

> a theoretically useful **expansion anchor**

and:

> a sufficiently specific **final lexical indicator**.

---

# 32. Recalculation of Internal Correlations After Semantic Purification

The internal correlations calculated for the earlier candidate vocabulary could not be reused after removing 259 terms.

For each term, the internal-correlation statistic depends on the sum of the **other terms currently in the lexicon**.

Once the vocabulary changes, the comparison vector also changes.

Internal correlations were therefore recomputed from scratch using only the:

\[
69
\]

semantically retained terms.

The same procedure was retained:

- 178 development countries;
- country-token normalization;
- lexical weighting;
- Pearson correlation of each term with the sum of all other retained terms.

The resulting correlation distribution was:

| Threshold | Number of terms |
|---|---:|
| \(r > 0\) | 59 |
| \(r > .05\) | 46 |
| \(r > .10\) | 35 |
| \(r > .15\) | 26 |
| \(r > .20\) | 18 |

Distribution summary:

- N = 69
- Mean = 0.126
- SD = 0.134
- 10th percentile = -0.044
- 25th percentile = 0.025
- Median = 0.105
- 75th percentile = 0.224
- 90th percentile = 0.301
- Minimum = -0.075
- Maximum = 0.466

File:

```text
data/purification/self_assertion_semantic69_recomputed_correlations.csv
```

---

# 33. Final Self-Assertion Inclusion Rule

After contextual semantic purification, internal correlation was treated as a measure of **lexical coherence**, not as the sole measure of semantic validity.

The final inclusion rule was:

\[
\boxed{
\text{semantic KEEP}
\land
\left(
r>0
\ \text{OR surviving original theory seed}
\right)
}
\]

Operationally:

1. every final term had to survive contextual semantic purification;
2. expanded/non-seed terms additionally had to show positive internal correlation;
3. an original theory seed that survived contextual semantic review could remain even if its internal correlation was non-positive.

This decision recognizes that Institutional Self-Assertion is intentionally multifaceted.

For example, countries may use:

- resistance/rejection vocabulary;
- proposal/advocacy vocabulary;
- commitment vocabulary;

in different combinations.

A semantically valid theoretical anchor therefore need not necessarily covary positively with every other lexical facet across countries.

---

# 34. Theory Seeds Retained Despite Non-Positive Internal Correlation

Three original theory seeds survived contextual semantic purification but had non-positive recomputed internal correlations:

```text
self-assertion
ready
insist
```

These were retained under the final theory-seed rule.

They were not retained merely because they were original seeds.

Each had already survived the full contextual semantic review.

`exercise`, by contrast, did not survive semantic purification and therefore received no final seed protection.

---

# 35. Final Self-Assertion KGL

Among the 69 semantically retained terms:

\[
59
\]

had positive recomputed internal correlations.

Adding the three semantically validated theory seeds with non-positive correlations produced:

\[
\boxed{62\text{ final terms}}
\]

The final Self-Assertion KGL is therefore a:

> **62-term weighted Knowledge-Guided Lexicon**

Final file:

```text
data/purification/self_assertion_kgl_FINAL.csv
```

The final lexicon should be interpreted as:

> **a theory-guided and corpus-purified lexical approximation of Institutional Self-Assertion in UN General Debate discourse**

It should not be interpreted as:

- a perfect sentence-level classifier;
- proof that every individual lexical occurrence constitutes Self-Assertion;
- a direct measure of national personality;
- a direct measure of a speaker's psychological motivation.

Context remains important because Institutional Self-Assertion is inherently actor-dependent.

---

# 36. Final Self-Assertion Pipeline

The completed Self-Assertion KGL pipeline is:

```text
1. Define Institutional Self-Assertion theoretically
        ↓
2. Generate theory-guided candidate seeds
        ↓
3. Contextually audit initial seed candidates
        ↓
4. Conduct independent 250-context coverage audit
        ↓
5. Identify underrepresented construct facets
        ↓
6. Audit gap-targeted candidate seeds
        ↓
7. Freeze 14 theory seeds
        ↓
8. FastText synonym expansion
   threshold = .75
        ↓
9. FastText concept-centroid expansion
   threshold = .60
        ↓
10. Assign lexical weights
    seed = 1
    expanded term = cosine similarity
        ↓
11. Corpus-frequency purification
    development token count ≥ 10
        ↓
12. Hard function-word exclusion
    543 → 516 terms
        ↓
13. Conservative development-context evidence screen
    516 → 498 terms
        ↓
14. Country-normalized internal-correlation screening
        ↓
15. Positive-r screen + provisional seed protection
    498 → 328 candidate terms
        ↓
16. Sample up to 5 real development usages per candidate
        ↓
17. Full contextual semantic purification
    328 → 69 semantically valid terms
        ↓
18. Recompute country-level internal correlations
    using only the 69 retained terms
        ↓
19. Apply final rule:
    semantic KEEP AND
    (r > 0 OR surviving theory seed)
        ↓
20. Freeze final Self-Assertion KGL
    62 terms
```

---

# 37. Relationship to Havaldar et al. (2024)

The implemented procedure follows the central Knowledge-Guided Lexical Analysis logic of Havaldar et al.:

```text
expert/theory-guided seeds
        ↓
synonym expansion
        +
concept expansion
        ↓
similarity weighting
        ↓
frequency purification
        ↓
internal-correlation purification
        ↓
final lexicon
```

However, the present implementation is an **adapted KGL procedure rather than an exact replication**.

Core KGL components retained include:

- theory-guided seed construction;
- synonym expansion;
- concept-centroid expansion;
- cosine-similarity weighting;
- corpus-frequency purification;
- internal-correlation analysis;
- sensitivity examination of alternative thresholds.

The principal project-specific adaptations are described below.

---

## 37.1 Actor-dependent construct

Institutional Self-Assertion depends on whether the **focal national actor** is actively positioning itself.

A word can therefore be semantically related to assertion while failing to measure the construct when:

- the relevant actor is a third party;
- the expression is merely descriptive;
- the word is used in an unrelated sense.

This motivated stronger contextual review than would be necessary for a more actor-independent lexical construct.

---

## 37.2 Country-size normalization

Internal-correlation calculations used relative frequencies normalized by each country's total development-corpus token count.

This differs from a raw-count implementation.

The normalization was introduced because the amount of available discourse differed across countries and raw lexical counts could therefore partially reflect country corpus volume.

---

## 37.3 Contextual semantic purification

The final procedure included manual contextual review of all 328 statistically surviving candidate terms.

This step was added because embedding similarity and geographic internal correlation cannot establish by themselves that the lexical item is functioning as Self-Assertion in UN discourse.

---

## 37.4 Final semantic-first decision rule

Final inclusion required semantic validity first.

Only after a term had survived semantic purification was internal coherence used for final inclusion.

Thus:

```text
semantic validity
        ↓
internal coherence
```

rather than:

```text
internal correlation alone
        ↓
assumed semantic validity
```

---

## 37.5 Conditional seed protection

Theory seeds were not permanently protected.

A seed had to survive contextual semantic purification before it could receive protection from a non-positive final internal correlation.

This allowed theoretically motivated but empirically unsuitable seeds, such as `exercise`, to be removed.

---

# 38. Comparison of Final Internal-Correlation Sensitivity

After semantic purification, the 69 retained terms showed the following internal-correlation sensitivity:

| Internal-correlation threshold | Self-Assertion terms retained |
|---|---:|
| \(r>0\) | 59 |
| \(r>.05\) | 46 |
| \(r>.10\) | 35 |
| \(r>.15\) | 26 |
| \(r>.20\) | 18 |

This threshold pattern was used as a sensitivity diagnostic rather than as evidence that one universal purification threshold applies across constructs.

The final rule used positive correlation for non-seed terms because semantic purification had already supplied a stronger construct-relevance screen.

---

# 39. Validation Scope and Interpretation Limits

The Self-Assertion development process included substantial within-development-corpus validation through:

- contextual seed auditing;
- independent construct-coverage auditing;
- contextual retrieval diagnostics;
- full candidate-term semantic review;
- country-level internal-coherence analysis.

No additional independent holdout validation was conducted for the current mock project.

Accordingly, the final report should **not claim independently estimated out-of-sample precision, recall, sensitivity, specificity, or classification accuracy** for the final 62-term lexicon.

The defensible claim is narrower:

> The Self-Assertion KGL was theory-guided, constructed in a development corpus separate from the focal analysis corpus, expanded using FastText, and purified through corpus frequency, contextual semantic evidence, and country-level internal coherence.

The KGL should therefore be treated as a:

> **validated lexical approximation of Institutional Self-Assertion for the purposes of the present exploratory analysis**

rather than as a formally validated universal classifier.

---

# 40. Core Self-Assertion Output Files

The primary files retained for the completed purification pipeline are:

```text
data/purification/self_assertion_frequency_purified.csv

data/purification/self_assertion_kgl_candidate_final.csv

data/purification/self_assertion_semantic_term_review.csv

data/purification/self_assertion_semantic_term_review_FINAL.csv

data/purification/self_assertion_semantic69_recomputed_correlations.csv

data/purification/self_assertion_kgl_FINAL.csv
```

Their roles are:

### `self_assertion_frequency_purified.csv`

Records the candidate vocabulary after primary corpus-frequency purification.

### `self_assertion_kgl_candidate_final.csv`

Contains the 328 candidate terms entering full contextual semantic purification.

### `self_assertion_semantic_term_review.csv`

Contains sampled real-world development-corpus contexts used to evaluate the semantic behavior of the 328 candidates.

### `self_assertion_semantic_term_review_FINAL.csv`

Records the final KEEP/EXCLUDE semantic decisions.

### `self_assertion_semantic69_recomputed_correlations.csv`

Contains recomputed internal correlations for the 69 semantically retained terms.

### `self_assertion_kgl_FINAL.csv`

Contains the frozen 62-term Self-Assertion KGL used for subsequent analysis.

---

# 41. Notebook Structure

The current project notebook structure is:

```text
01_data_audit.ipynb
02_preprocessing.ipynb
03_seed_lexica.ipynb
04_expansion.ipynb
05_purification.ipynb
```

The implemented responsibilities are:

### `01_data_audit.ipynb`

- corpus parsing;
- speech-level structure checks;
- country/year coverage;
- duplicate checks;
- missing-data checks;
- raw corpus audit.

### `02_preprocessing.ipynb`

- Unicode normalization;
- formatting normalization;
- lowercase computational representation;
- sentence segmentation;
- sentence-level quality control;
- procedural-introduction removal;
- cleaned sentence-level outputs.

### `03_seed_lexica.ipynb`

- Self-Assertion construct definition;
- candidate-seed development;
- contextual seed audit;
- independent coverage audit;
- gap-targeted candidate evaluation;
- frozen 14-seed set.

### `04_expansion.ipynb`

- FastText model loading;
- synonym expansion;
- concept-centroid expansion;
- similarity weighting;
- expansion-threshold diagnostics.

### `05_purification.ipynb`

- tokenization;
- corpus-frequency purification;
- hard function-word exclusion;
- development-context evidence screening;
- country-level normalized frequency calculation;
- internal-correlation analysis;
- candidate semantic review;
- final semantic adjudication;
- recomputation of internal correlations;
- final 62-term KGL construction.

---

# 42. Completed Self-Assertion Phase Status

The following stages are complete:

- [x] Corpus definition
- [x] Focal/development separation
- [x] Raw-data parsing
- [x] Corpus audit
- [x] Conservative preprocessing
- [x] Sentence segmentation
- [x] Procedural-text identification
- [x] Procedural-text removal
- [x] Self-Assertion theoretical definition
- [x] Semantic boundary rules
- [x] Initial candidate-seed development
- [x] Contextual seed audit
- [x] Independent 250-context coverage audit
- [x] Gap-targeted candidate development
- [x] Gap-targeted contextual audit
- [x] Frozen 14-seed set
- [x] FastText synonym expansion
- [x] FastText concept-centroid expansion
- [x] Similarity weighting
- [x] Frequency purification
- [x] Hard function-word filtering
- [x] Development-context evidence screen
- [x] Country-normalized internal-correlation screening
- [x] Full 328-term contextual semantic review
- [x] Uncertain-term adjudication
- [x] 69-term semantic lexicon
- [x] Recalculation of internal correlations
- [x] Final semantic + coherence inclusion rule
- [x] Frozen 62-term Self-Assertion KGL

---

# 43. Current Methodological Status

**Institutional Self-Assertion KGL construction is complete.**

The frozen instrument is:

```text
data/purification/self_assertion_kgl_FINAL.csv
```

with:

\[
\boxed{62\text{ terms}}
\]

The focal 18-country corpus has not been used to revise the Self-Assertion lexicon.

This marks the end of the **Self-Assertion measurement-development phase**.

# 44. Development-Corpus Self-Assertion Retrieval

The frozen 62-term Self-Assertion KGL was applied to the cleaned development corpus.

Lexical matching used the same lowercase tokenization procedure used during KGL purification:

```python
TOKEN_PATTERN = re.compile(r"[a-z]+(?:-[a-z]+)*")


# 45. Application of the Frozen Self-Assertion KGL to the Focal Corpus

**Notebook 06 — Focal Self-Assertion Context Extraction**

The frozen 62-term Institutional Self-Assertion KGL was applied to the focal UN General Debate corpus.

The focal corpus contained:

- 18 purposively selected countries from nine predefined cultural regions.
- One speech per country per year, 2016–2025.
- 180 speeches in total.
- 18,882 cleaned sentences.

## 45.1 Lexical Matching

The frozen KGL terms were matched against cleaned focal sentences using case-insensitive, word-boundary regular expressions.

For each matched sentence, the procedure retained:

- `iso3`
- `year`
- `filename`
- `sentence_id`
- `sentence_text`
- `matched_terms`
- `n_matches`

Word-boundary matching reduced unintended matches inside longer words.

The focal corpus was not used to revise the KGL or its weights.

## 45.2 Retrieval Results

The procedure identified:

- **1,230 matched sentences**.
- **1,309 KGL term occurrences**.
- **177 speeches** containing at least one match.
- **3 speeches** containing no matches.

A sentence could contain multiple term occurrences. Consequently, matched-sentence counts and term-occurrence counts were distinguished.

The resulting dataset was saved as:

`data/contexts/focal_self_assertion_contexts.csv`

**Decision:** Matched sentences, rather than expanded neighboring-sentence windows, were used for subsequent human-value prediction.

---

# 46. Speech-Level Institutional Self-Assertion Measurement

**Notebook 07 — Self-Assertion Frequency Scoring**

The frozen weighted KGL was used to calculate Self-Assertion prevalence for all 180 focal speeches.

## 46.1 Weighted Lexical Frequency

For each speech:

1. Count occurrences of each KGL term.
2. Retrieve the corresponding frozen term weight.
3. Sum weighted term occurrences.
4. Calculate the cleaned speech word count.
5. Normalize weighted matches per 1,000 words.

The resulting measure was:

$$
SA_d =
\frac{
1000 \times \sum_{w \in L_{SA}} f(w,d)\,\mathrm{weight}(w)
}{
\mathrm{wordcount}(d)
}
$$

where $f(w,d)$ is the number of occurrences of KGL term $w$ in speech $d$.

Cleaned word counts used the regular-expression token pattern `\b\w+\b`.

## 46.2 Zero-Match Speeches

All 180 speeches were retained.

Speeches containing no KGL matches received a Self-Assertion prevalence score of zero.

## 46.3 Output

The results were saved as:

`data/results/focal_self_assertion_scores.csv`

The file contained 180 rows and seven columns:

- `iso3`
- `year`
- `filename`
- `word_count`
- `n_matches`
- `weighted_matches`
- `sa_per_1000`

The overall mean speech-level score was approximately **2.600 weighted occurrences per 1,000 words**.

**Interpretation:** The measure represents weighted lexical prevalence, not directly observed psychological assertiveness or independence.

---

# 47. Initial Cross-National Description of Self-Assertion Prevalence

**Notebook 08 — Descriptive Country Comparisons**

Speech-level Self-Assertion prevalence scores were aggregated by country.

For each country, scores were averaged across its ten annual speeches, giving each year equal weight.

$$
SA_c = \frac{1}{10}\sum_{y=2016}^{2025}SA_{cy}
$$

The resulting country-level scores were examined descriptively.

Examples included:

| Country | Mean SA per 1,000 words |
|---|---:|
| China | 4.440 |
| Saudi Arabia | 4.205 |
| Colombia | 3.539 |
| United States | 3.362 |
| India | 1.354 |

These values describe Institutional Self-Assertion lexical prevalence in the selected diplomatic speeches, not national psychological characteristics.

---

# 48. Revised Strategy for Measuring Human Values

The next objective was to identify human-value language accompanying Institutional Self-Assertion.

Earlier plans considered developing separate framing lexicons or fine-tuning a transformer.

To keep the exploratory mock project computationally feasible, an existing pretrained human-value classifier was selected instead.

**Selected model: TUM DeBERTa Human Value Detector**

Model identifier:

`tum-nlp/Deberta_Human_Value_Detector`

Source:

https://huggingface.co/tum-nlp/Deberta_Human_Value_Detector

The model predicts 20 human-value categories from the ValueEval framework, based largely on Schwartz's refined theory of basic human values.

**Rationale:** Using a pretrained multi-label classifier allowed the project to examine a theoretically established range of values without developing separate lexical instruments for every category.

**Decision:** The pretrained model was used for inference only. No fine-tuning or additional model training was performed.

---

# 49. Transformer-Based Human-Value Prediction

**Notebook 09 — Human Value Detection**

The TUM DeBERTa model was applied to the **1,230 Self-Assertion-matched sentences**.

The `sentence_text` field was used as model input.

Inference was conducted in batches of eight sentences.

## 49.1 Continuous Value Scores

For each sentence, the classifier generated 20 continuous value prediction scores.

The scores were retained without converting them into binary classifications.

This avoided discarding information through a fixed classification threshold.

The scores were interpreted as **model-predicted value-related signals**, not as independently verified human annotations or calibrated measures of psychological motivation.

## 49.2 Output

The results were saved as:

`data/results/focal_self_assertion_values.csv`

The output contained:

- 1,230 rows (one per matched sentence).
- 7 metadata columns.
- 20 predicted human-value score columns.
- 27 columns in total.

All 20 values were retained for subsequent exploratory analysis.

---

# 50. Preliminary Inspection of Human-Value Predictions

A set of 60 sentences and their predictions was examined informally during the analysis.

The examination was exploratory rather than a formal validation study.

Specifically:

- No independent annotation dataset was created.
- No formal inter-rater reliability was calculated.
- No held-out UNGD test set was constructed.
- The pretrained model was not modified or retrained.

Consequently, this inspection cannot establish corpus-specific classification accuracy or F1 scores.

**Limitation:** The model was originally developed for argument-based human-value classification rather than UN diplomatic speeches. Domain differences may affect prediction validity.

---

# 51. Country-Level Human-Value Profiles

**Notebook 10 — Value Aggregation and Exploratory Heatmap**

The 20 predicted value scores were aggregated using a two-stage procedure.

## 51.1 Sentence-to-Country-Year Aggregation

For each country-year, the mean score for each value was calculated across matched sentences.

$$
V_{cy,v} =
\frac{1}{N_{cy}}
\sum_{s=1}^{N_{cy}}p_{s,v}
$$

where:

- $p_{s,v}$ is the predicted score for value $v$ in sentence $s$.
- $N_{cy}$ is the number of matched sentences in country $c$, year $y$.

## 51.2 Country-Year-to-Country Aggregation

The country-year scores were then averaged across available years.

$$
V_{c,v} =
\frac{1}{Y_c}
\sum_{y \in \mathcal{Y}_c}V_{cy,v}
$$

where $Y_c$ is the number of years with at least one matched sentence.

Each eligible year received equal weight.

Years without matches were treated as missing for value-score aggregation rather than assigned a value score of zero.

The number of eligible years was:

- India: 8.
- Indonesia: 9.
- All other countries: 10.

## 51.3 Exploratory Heatmap

An initial **18-country × 20-value heatmap** was generated.

Each cell represented the country-level mean predicted score for one value.

The heatmap was intended for exploratory inspection of cross-national patterns rather than as the sole final results visualization.

**Interpretive constraint:** Comparisons were made across countries within the same value category. Raw scores across different value categories were not interpreted as directly comparable measures of value priority.

---

# 52. Region-Centered Analysis and Autonomy-Related Values

**Notebook 11 — Final Results (Initial Blocks Completed)**

The final analysis was organized around two research questions.

**RQ1:** Does Institutional Self-Assertion prevalence correspond to greater autonomy-related value expression across selected country and regional cases?

**RQ2:** How do model-predicted human-value profiles in Institutional Self-Assertion contexts vary across selected regional case pairs?

## 52.1 Country-Level Analysis Dataset

A country-level dataset was created by combining:

- Mean Self-Assertion prevalence across ten annual speeches.
- Mean scores for all 20 values.
- Number of speeches.
- Number of matched sentences.
- Number of years with value predictions.

The dataset retained all 18 focal countries.

Matched-sentence counts (1,230) and KGL term occurrences (1,309) were treated as distinct quantities.

## 52.2 Regional Aggregation

The 18 countries were organized into nine predefined regional case pairs.

| Region | Country 1 | Country 2 |
|---|---|---|
| Modern West | USA | GBR |
| East Asia | CHN | JPN |
| Arab Region | SAU | JOR |
| Latin America | CHL | COL |
| South Asia | IND | PAK |
| Sub-Saharan Africa | GHA | KEN |
| Southeast Asia | IDN | THA |
| Central Asia | KAZ | TJK |
| Eastern Europe | POL | ROU |

Each regional score was calculated as the equal-weighted mean of the two selected country scores.

$$
X_r = \frac{X_{c_1}+X_{c_2}}{2}
$$

This approach prevented countries with more matched sentences from automatically dominating regional averages.

**Interpretation:** Regional case-pair scores are descriptive summaries of purposively selected countries, not statistically representative estimates of entire cultural regions.

## 52.3 Selection of Autonomy-Related Values

Two values from Schwartz's refined framework were selected for RQ1:

- **Self-direction: action:** Primary autonomy-related indicator.
- **Self-direction: thought:** Complementary autonomy-related indicator.

Self-direction: action concerns freedom in determining one's actions.

Self-direction: thought concerns freedom in developing one's ideas and abilities.

Power: dominance was not selected as an autonomy indicator because its motivational emphasis is control over others rather than self-determination.

The two Self-direction values were analyzed separately rather than combined into an unvalidated composite.

## 52.4 Spearman Correlation Analysis

Descriptive Spearman correlations were calculated between Institutional Self-Assertion prevalence and each autonomy-related value.

Two aggregation levels were examined:

- Country level: $N=18$.
- Regional case-pair level: $N=9$.

Spearman correlation was used to summarize monotonic associations without assuming linear relationships.

Results:

| Analysis level | SD-action (rho) | SD-thought (rho) |
|---|---:|---:|
| Country (N = 18) | -0.040 | -0.098 |
| Regional pair (N = 9) | +0.167 | +0.267 |

The coefficients were treated as descriptive because the countries were purposively selected and the number of regional pairs was small.

**Status:** Data preparation, regional aggregation, and RQ1 correlation analysis are complete.

Final RQ1 visualization and systematic RQ2 analysis remain in progress.

---

# 53. Current Methodological Scope and Remaining Work

The following stages are complete:

- [x] Frozen 62-term Institutional Self-Assertion KGL.
- [x] Focal Self-Assertion sentence retrieval.
- [x] Speech-level weighted Self-Assertion scoring.
- [x] Initial country-level Self-Assertion comparisons.
- [x] Pretrained human-value model selection.
- [x] Transformer inference on 1,230 matched sentences.
- [x] Country-year and country-level value aggregation.
- [x] Exploratory 18-country × 20-value heatmap.
- [x] Country-level dataset preparation for final analysis.
- [x] Nine-region case-pair aggregation.
- [x] RQ1 Spearman correlations.

Remaining stages:

- [ ] RQ1 final visualization.
- [ ] Systematic RQ2 exploration across all 20 values.
- [ ] Selection and visualization of interpretable regional patterns.
- [ ] Final Results and Discussion.
- [ ] Final research brief.

## 53.1 Methods Not Included in the Final Project

The following procedures were considered but not implemented:

- NMF or unsupervised topic clustering.
- Training or fine-tuning a new human-value classifier.
- Separate KGL construction for every human-value category.
- Non-Self-Assertion baseline comparisons.
- Complex multilevel statistical models.
- Formal large-scale human annotation.

These were excluded to maintain a focused, feasible exploratory mock project.

## 53.2 Interpretation Boundaries

The final project adopts the following restrictions:

1. Self-Assertion scores represent weighted lexical prevalence, not national psychological characteristics.
2. DeBERTa outputs represent predicted value-related language, not verified psychological motivations.
3. Without baseline comparisons, value patterns cannot be claimed to be exclusive to Self-Assertion contexts.
4. Regional case-pair means cannot be generalized to entire cultural regions.
5. Exploratory associations and value-profile differences do not establish causal or motivational mechanisms.

**Current status:** Notebook 11 is active. The next step is Block 4, RQ1 visualization.
