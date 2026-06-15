
# Generative AI use & student outcomes

**Research question:** How do the *extent* and *type* of generative-AI use relate
to student performance — specifically academic achievement, exam-related anxiety,
and skill retention?

A key theme of the project: it is **not whether students use AI, but how** they
use it that matters for their outcomes.

## Repository contents

| File | What it is |
|------|------------|
| `ai-student-impact-analysis.Rmd` | Main analysis notebook (EDA, models, figures) |
| `proposal.Rmd` | Original project proposal |
| `README.md` | This file |
| `.gitignore` | Excludes the data file from the repo |

## Data

The dataset is **not** included in this repository. Obtain
`ai_student_impact_dataset.csv` (50,000 students, one row per student per
semester) and place it in the project root, then load it as `students`.

The analysis uses these columns: `Pre_Semester_GPA`, `Post_Semester_GPA`,
`Weekly_GenAI_Hours`, `Traditional_Study_Hours`, `Primary_Use_Case`,
`Prompt_Engineering_Skill`, `Tool_Diversity`, `Paid_Subscription`,
`Perceived_AI_Dependency`, `Institutional_Policy`, `Major_Category`,
`Year_of_Study`, `Anxiety_Level_During_Exams`, and `Skill_Retention_Score`.
`Student_ID` is dropped (identifier only).

## Requirements

- R >= 4.1, RStudio
- Packages: `tidyverse` (data handling + ggplot2) and `broom` (tidy model output)

```r
install.packages(c("tidyverse", "broom"))
```

## How to run

Open `ai-student-impact-analysis.Rmd` in RStudio, make sure the CSV is in the
project root, and Knit (or run the chunks top to bottom). All figures are
generated inline; no manual steps are required.

## Engineered variables

The two raw "hours" columns are correlated and don't separate two different
ideas, so we replace them with two near-independent dimensions:

```r
students <- students %>%
  mutate(
    AI_Ratio    = Weekly_GenAI_Hours / (Weekly_GenAI_Hours + Traditional_Study_Hours),
    Total_Hours = Weekly_GenAI_Hours + Traditional_Study_Hours,
    GPA_change  = Post_Semester_GPA - Pre_Semester_GPA
  )
```

- **`AI_Ratio`** — the *mix*: what share of study time relies on AI (reliance).
- **`Total_Hours`** — the *volume*: total weekly study effort (intensity).
- **`GPA_change`** — improvement over the semester (used for inspection; models
  control for `Pre_Semester_GPA` directly — see below).

## Analysis overview

1. **Data familiarization** — column meanings, types, missingness (none), and
   sanity checks (noted the 4.0 GPA ceiling and right-skewed AI hours).
2. **Exploratory analysis** — GPA change by major (negligible differences),
   by AI use case (clear differences), exam anxiety, and institutional policy.
3. **GPA model** — a hierarchical linear model:
   `GPA_change ~ Pre_Semester_GPA + AI_Ratio + Total_Hours + Tool_Diversity +
   Perceived_AI_Dependency + Paid_Subscription + Primary_Use_Case +
   Prompt_Engineering_Skill + Major_Category + Year_of_Study + Institutional_Policy`.
   `Pre_Semester_GPA` is included on the right-hand side (ANCOVA style) to control
   for regression to the mean rather than differencing it away.
   Results shown as a coefficient plot via `broom::tidy(model, conf.int = TRUE)`.
4. **Subjective vs. objective comparison** — parallel models with identical
   controls, swapping only the AI measure: `Weekly_GenAI_Hours` (objective) vs.
   `Perceived_AI_Dependency` (subjective), compared on adjusted R-squared and AIC
   across outcomes.
5. **Grade vs. skill** — examined how weakly grade improvement tracks skill
   retention.

## Key findings

- **Type beats amount.** Use cases that require thinking (Debugging) are linked to
  larger GPA gains; passive use (Direct Answer Generation) to smaller gains.
  Higher `AI_Ratio` (over-reliance) is the single strongest negative predictor.
- **Prompt skill matters.** Advanced prompt-engineering skill is among the
  strongest positive predictors of both GPA gain and skill retention.
- **The right measure depends on the outcome.** Felt dependency (subjective)
  predicts *anxiety* slightly better; actual hours (objective) predict *skill
  retention* slightly better. Differences are small but directionally consistent.
- **Grades != ability.** Grade improvement correlates only weakly with skill
  retention (r ~ 0.20, < 4% shared variance), so a grade gain is a poor indicator
  of skills actually retained.

## Important caveats

- All relationships are **correlational, not causal** — the data are
  observational, so we describe associations, not effects.
- With 50,000 rows almost everything is statistically significant; we judge
  importance by effect size, not p-values alone.
- The full GPA model's R-squared ~ 0.22, so AI use explains part — not most — of
  the variation in outcomes.
- `Perceived_AI_Dependency` and `Skill_Retention_Score` arrived pre-computed in
  the dataset; their exact measurement is not documented.

## Reproducibility notes
- The CSV is git-ignored on purpose; do not commit it.
