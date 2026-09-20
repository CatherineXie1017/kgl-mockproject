from pathlib import Path

PROJECT_ROOT = Path.cwd().parent
METHODS_LOG = PROJECT_ROOT / "METHODS_LOG.md"

methods_text = r"""# Methodology and Decision Log

## Project

**Working title:**  
*What Does Self-Assertion Mean Across Cultures? A Knowledge-Guided and Exploratory Computational Analysis of Self-Assertion in Global Institutional Discourse*

**Corpus:** UN General Debate Corpus (UNGDC)

**Primary analysis period:** 2016–2025

**General design:**  
The project separates a focal corpus from a development corpus. The development corpus is used for construction, quality control, expansion, and purification of Knowledge-Guided Lexica (KGLs). The focal corpus is held apart for the eventual substantive cross-country analysis.

This separation is intended to reduce the risk of adapting the lexica to obtain desired results in the focal countries.

---

# 1. Corpus Definition

## 1.1 Analysis period

Only speeches from **2016 through 2025** are included.

Earlier UN General Debate speeches were excluded because the project is designed to characterize recent institutional discourse rather than longitudinal historical change.

Year therefore primarily functions as repeated sampling of each country's recent discourse rather than as a substantive longitudinal predictor.

---

## 1.2 Focal corpus

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

---

## 1.3 Development corpus

All available 2016–2025 speeches from countries not included among the 18 focal countries were placed in the development corpus.

The development corpus is used for lexicon development and purification and is kept separate from the focal corpus.

---

# 2. Raw File Structure and Parsing

The UNGD text files follow the filename structure:

`ISO3_SESSION_YEAR.txt`

Example:

`AFG_71_2016.txt`

where:

- `AFG` = three-letter country code
- `71` = UN General Assembly session
- `2016` = speech year

Python recursively searched the focal and development directories for `.txt` files.

For each speech, the following information was extracted:

- corpus membership
- ISO3 country code
- UN session
- year
- filename
- original filepath
- complete speech text
- preliminary whitespace-based word count

Each raw `.txt` speech was converted into one row of a pandas DataFrame.

---

# 3. Corpus Audit

The raw text files were converted into two structured datasets:

- `focal_df`
- `dev_df`

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

---

# 4. Initial Text Preprocessing

Preprocessing was intentionally conservative. The goal of this stage was to standardize formatting and Unicode representation without removing potentially meaningful lexical information.

The original speech text was preserved in the `text` column.

A separate `text_clean` column was created for preprocessing.

---

## 4.1 Unicode normalization

Text was normalized using Unicode **NFC normalization**:

`unicodedata.normalize("NFC", text)`

This standardizes alternative underlying Unicode representations of visually equivalent characters without removing accents or changing lexical content.

---

## 4.2 Line and paragraph separator normalization

The following Unicode separators were converted to ordinary newline characters:

- `U+2028` LINE SEPARATOR → `\n`
- `U+2029` PARAGRAPH SEPARATOR → `\n`

These were treated as formatting variants of ordinary line breaks.

Visible characters such as:

- `U+2022` BULLET (`•`)

were **not automatically removed or replaced**, because they may contain meaningful formatting information and are not line terminators.

---

## 4.3 Whitespace normalization

The following formatting normalization was performed:

- non-breaking spaces (`U+00A0`) → ordinary spaces;
- Windows line endings (`\r\n`) → `\n`;
- remaining carriage returns (`\r`) → `\n`;
- tab characters (`\t`) → ordinary spaces;
- sequences of two or more ordinary spaces → one space;
- sequences of three or more newline characters → two newline characters;
- leading and trailing whitespace removed with `.strip()`.

Importantly, ordinary paragraph structure was retained.

---

## 4.4 Lowercase computational representation

A separate `text_lower` variable was created from `text_clean`.

This allows lexical matching to treat forms such as:

`Protect`, `PROTECT`, and `protect`

as equivalent while preserving capitalization in the original and cleaned-text columns.

---

# 5. Post-Preprocessing Quality Control

After formatting normalization:

- missing focal texts: **0**
- missing development texts: **0**
- empty focal texts: **0**
- empty development texts: **0**

A whitespace-based cleaned word count was computed for each speech.

## Focal corpus speech length

- N = 180
- Mean = 2,171.38 words
- SD = 951.97
- Minimum = 611
- 25th percentile = 1,554
- Median = 1,978.5
- 75th percentile = 2,571.75
- Maximum = 8,242

**Total cleaned focal corpus size:** 390,849 words

## Development corpus speech length

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

# 6. Short-Speech Quality Check

The five shortest focal and development speeches were identified for inspection.

The shortest focal speech was:

- Indonesia
- 2021
- `IDN_76_2021.txt`
- 611 cleaned whitespace-delimited words

The beginning of this speech was manually inspected and contained coherent continuous UN General Debate content rather than evidence of an empty, corrupted, or failed extraction.

Short speech length alone was therefore not treated as an exclusion criterion.

---

# 7. Data Provenance Structure

The processing pipeline currently follows:

`Raw UNGD .txt files`

→ `01_data_audit.ipynb`

→ structured/audited CSV datasets

→ `02_preprocessing.ipynb`

→ formatting-normalized computational text

The original imported text remains preserved so preprocessing decisions can be revised without returning to the raw corpus.

---

# 8. Planned KGL Development

The subsequent methodology will implement Knowledge-Guided Lexicon construction following the general Havaldar et al. framework:

1. theory-guided seed selection;
2. FastText-based synonym expansion;
3. FastText-based concept/centroid expansion;
4. similarity-based weighting;
5. corpus-frequency purification;
6. internal-correlation purification;
7. validation and sensitivity analyses;
8. freezing lexica before substantive focal-country analysis.

Five planned lexica are:

1. Self-Assertion
2. Self-Oriented Framing
3. Ingroup-Protective Framing
4. Collective-Advancement Framing
5. Relational/Negotiation Framing

Details and project-specific deviations or thresholds will be documented here when implemented.

---

# 9. Decision-Logging Principle

For every major methodological decision, record:

- what was done;
- why it was done;
- whether it follows the source method directly or is project-specific;
- numerical thresholds used;
- quality-control results;
- sensitivity analyses;
- any deviations from the original plan;
- date or analysis stage when useful.

This file is a methodological working log, not yet manuscript-ready Methods prose.
"""

METHODS_LOG.write_text(methods_text, encoding="utf-8")

print(f"Created methodology log at:\n{METHODS_LOG}")