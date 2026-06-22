# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in the CS2 competitive community. Given a Reddit comment about a CS2 Major tournament match, the model predicts whether the comment is analytical reasoning, a hot take, or an emotional reaction.

---

## Community

**Counter-Strike 2 (CS2) competitive scene** — specifically posts and comments about teams competing in the most recent CS2 Major tournament (IEM Cologne 2026). Source: r/GlobalOffensive and r/cs2, primarily match-thread comments and post-match discussion threads.

CS2 tournament discourse is dense with opinion and varies enormously in quality. After a big match, you'll find everything from detailed tactical breakdowns citing round-by-round stats to pure hype reactions to bold unjustified predictions. The community actively distinguishes between "analysts" and "people who just vibe," which means the label distinctions are grounded in norms the community already cares about.

---

## Label Taxonomy

### `analysis`
The post supports a claim about a team using specific, verifiable evidence — round statistics, player ratings, map win rates, tactical observations tied to specific rounds, or historical team comparisons with numbers.

**Examples:**
- "karrigan is inevitable. kyxsan's Falcons this year never won playoffs, kyousuke looked like a bum. Now in this major playoffs, kyousuke is the 2nd highest rated player of the tournament, and Falcons are winning all the playoffs matches."
- "Spirit have never been a team known for mental resilience, but here they showed plenty of it. Six match points saved across two maps."

### `hot_take`
A bold, confident opinion about a team stated as fact without supporting evidence. The claim might be true, but the post asserts rather than argues.

**Examples:**
- "Falcons are massively overrated — they only got this far because the bracket was soft."
- "Spirit has zero motivation this major, you can see it in their play."

### `reaction`
An immediate emotional response to a specific match moment or result, with little to no argument. The post is expressing a feeling in real time rather than making a claim.

**Examples:**
- "I died and came back to life like twenty different times this series"
- "LETS GOOO FALCONS WINNING THAT CLUTCH ROUND!!!"

### Hard Edge Case — `analysis` vs. `hot_take`

**Decision rule:** If the post provides specific, verifiable evidence (a stat, a round, a map result) that would support the claim even if you removed the opinion framing, label it `analysis`. If the evidence is vague, cherry-picked, or decorative — cited to sound credible but not actually reasoning — label it `hot_take`. A single cherry-picked stat used to justify a sweeping generalization is `hot_take`.

**Difficult examples:**
1. "Falcons' AWPer went negative in the semis and that's proof this roster will never win a major." → `hot_take` — one cherry-picked stat used to support an extreme generalization.
2. "Donk always look human when it's vs Niko and monesy man." → `hot_take` — "always" pattern claim with no stats.
3. "m0NESY on anubis was just unfair, 2.13 rating in a major grand final map is insane." → `analysis` — specific verifiable player rating on a specific map in a specific match.

---

## Data Collection

**Source:** Reddit — r/GlobalOffensive match threads and post-match discussion threads for the IEM Cologne Major 2026. Threads covered: Falcons vs Spirit (semifinal), Spirit vs 9z (Swiss round 3), Aurora vs Furia (quarterfinal), and the major finals post-match thread.

**Method:** Manual copy-paste into a staging file, then extracted, labeled, and organized into CSV.

**Label distribution (200 examples):**

| Label | Count | % |
|---|---|---|
| `analysis` | 62 | 31% |
| `hot_take` | 73 | 36.5% |
| `reaction` | 65 | 32.5% |

No label exceeds 70% of the dataset. Distribution is within acceptable range for training.

---

## Fine-Tuning

*To be completed in Milestone 5.*

- **Base model:** `distilbert-base-uncased`
- **Training setup:** Google Colab T4 GPU, 3 epochs, learning rate 2e-5, batch size 16
- **Split:** 70% train / 15% validation / 15% test (140 / 30 / 30)

---

## Baseline

*To be completed in Milestone 4.*

Zero-shot baseline using Groq `llama-3.3-70b-versatile` with label definitions provided in the prompt.

---

## Evaluation Report

*To be completed in Milestone 6.*

---

## AI Usage

- **Label stress-testing:** Used Claude to generate boundary-case posts between `analysis` and `hot_take` to pressure-test the decision rule before annotating 200 examples.
- **Annotation assistance:** Used Claude to pre-label batches of 30–40 posts at a time using the label definitions from `planning.md`. Every pre-assigned label was manually reviewed and corrected before committing. Pre-labeled examples are noted in the `notes` column of `dataset.csv`.
- **Dataset organization:** Used Claude to extract relevant comments from raw Reddit thread text (pasted into `data.md`), clean formatting artifacts, and write them into `dataset.csv` with labels and notes.
