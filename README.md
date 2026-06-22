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

- **Base model:** `distilbert-base-uncased`
- **Training setup:** Google Colab T4 GPU, 3 epochs, learning rate 2e-5, batch size 16
- **Dataset split:** 70% train / 15% validation / 15% test (140 / 30 / 30)
- **Key hyperparameter decision:** Used the default learning rate of 2e-5, which is standard for fine-tuning BERT-family models. Batch size of 16 was kept as-is since the dataset is small and larger batches would have reduced the number of gradient update steps.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile`, zero-shot (no task-specific training)

**Prompt approach:** The system prompt named the community, defined all three labels in plain language matching `planning.md`, gave one example post per label, and instructed the model to output only the label name — nothing else. This kept responses parseable without post-processing.

**Baseline accuracy: 76.67%** (23/30 test examples correct)

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Groq baseline (zero-shot) | **76.67%** |
| Fine-tuned DistilBERT | **50.00%** |

Fine-tuning made the model worse by 26.67 percentage points. The baseline is the stronger classifier for this task.

### Per-Class Metrics (Fine-Tuned Model)

Derived from the confusion matrix below:

| Label | Precision | Recall | F1 |
|---|---|---|---|
| `analysis` | 0.50 | 1.00 | 0.67 |
| `hot_take` | 0.40 | 0.18 | 0.25 |
| `reaction` | 0.60 | 0.33 | 0.43 |

None of the labels reached the success criterion of F1 ≥ 0.70. Only `analysis` came close, but only because the model predicted `analysis` for almost everything (100% recall = it never missed a true `analysis` example, but that is because it called most things `analysis`).

### Confusion Matrix (Fine-Tuned Model)

| True \ Predicted | `analysis` | `hot_take` | `reaction` |
|---|---|---|---|
| **`analysis`** | **10** | 0 | 0 |
| **`hot_take`** | 7 | **2** | 2 |
| **`reaction`** | 3 | 3 | **3** |

The diagonal represents correct predictions. The model predicted `analysis` on 20 out of 30 examples — nearly everything. It correctly identified all 10 true `analysis` examples but at the cost of misclassifying 7 `hot_take` and 3 `reaction` examples as `analysis` as well.

### Wrong Predictions — 3 Analyzed Failures

**Failure 1 — True: `hot_take`, Predicted: `analysis`**
> "Looks like Falcons stole Vitality's mental fortitude. Damm they're reaching a higher level during OTs"

This was labeled `hot_take` because it asserts a sweeping claim (Falcons absorbed Vitality's mental strength) with no statistical backing — the OT reference is vague, not a count or record. The model likely predicted `analysis` because it detected CS-specific terminology ("Falcons," "Vitality," "OTs") and a comparative framing that stylistically resembles analytical writing. The model learned surface vocabulary, not argument structure.

**Failure 2 — True: `hot_take`, Predicted: `analysis`**
> "Think my biggest takeaway is that Spirit is scary going forward. Old Spirit was just Donk and friends but we're seeing that even with Donk underperforming the pieces around him are very strong. Once Magixx gets more reps as IGL and they have a coach to help control their mental, I see them as favorites for the future."

This was labeled `hot_take` because the prediction ("favorites for the future") is asserted without evidence — the post observes that other players stepped up but converts that observation into a sweeping prediction with no data. The model predicted `analysis` because the post is long, mentions multiple specific players by name (Donk, Magixx), and references a role ("IGL"). Length and player-name density are false proxies for analytical reasoning.

**Failure 3 — True: `reaction`, Predicted: `analysis`**
> "No matter the results, I'm still so proud of Team Spirit for coming this far! They had no coach to call for time outs to strategise and reset their mental, and yet, they've managed pushed their opponents to many game point rounds throughout the tournament."

This was labeled `reaction` because the primary content is emotional pride — the coaching detail is mentioned as context for the feeling, not as evidence for a claim. The model predicted `analysis` because the sentence structure resembles an analytical observation (subject + "had no coach" + evidence of what they achieved). This is a genuine labeling edge case: a reaction post that sounds analytical because the fan is proud enough to back up their emotion with facts.

### Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "karrigan has now performed 8 upsets in CS2 major playoffs in 4/5 majors..." | `analysis` | `analysis` | 0.91 |
| "Falcons under Karrigan: 9 overtimes, 8 wins" | `analysis` | `analysis` | 0.88 |
| "Death, taxes and Karrigan finding a way into major finals" | `hot_take` | `analysis` | 0.79 |
| "WEE FUCKING WOO" | `reaction` | `reaction` | 0.82 |
| "I died and came back to life like twenty different times this series" | `reaction` | `hot_take` | 0.54 |

**Why the first prediction is reasonable:** The karrigan upsets post lists specific events (Copenhagen, Shanghai, Budapest, Cologne) with named opponents and a verifiable count (8 upsets in 4/5 majors). The model correctly identifies this as `analysis` with high confidence because the post structure — claim + enumerated evidence — is the clearest signal in the training data for that label.

### What the Model Learned vs. What Was Intended

**Intended:** The model should learn to distinguish argument structure — does the post reason from evidence, assert without reasoning, or express an emotion?

**What it actually learned:** The model learned a much simpler proxy — post length and CS vocabulary density. Long posts with player names, match references, and technical terms (IGL, OT, CT-side) got classified as `analysis` regardless of whether they were actually reasoning. Short, emotional posts got classified correctly when they were extremely short ("WEE FUCKING WOO"), but failed when they were longer reactions or assertion-heavy `hot_take` posts that used CS-specific language.

The core boundary the model failed to learn is the `analysis` vs. `hot_take` distinction. Both label types use the same vocabulary (player names, team names, match references), but `analysis` uses that vocabulary as evidence while `hot_take` uses it as decoration. Distinguishing these requires understanding whether a post is *arguing* or *asserting* — a semantic distinction that DistilBERT could not reliably extract from only 140 training examples.

The baseline LLM (Llama-3.3-70b) has the language understanding depth to catch this distinction zero-shot, which explains why it outperformed the fine-tuned model by 27 percentage points.

---

## Spec Reflection

**One way the spec helped:** The instruction to define a hard edge case and decision rule before annotating forced a specific, written rule for `analysis` vs. `hot_take` boundaries. Without that, annotation would have been inconsistent across the 200 examples, and the model would have had an even noisier signal to learn from.

**One way implementation diverged from the spec:** The spec assumed fine-tuning would outperform the baseline — the evaluation table in the instructions implies fine-tuning is the goal and baseline is the floor to beat. In practice, the baseline outperformed the fine-tuned model significantly. The task turned out to be one where a large zero-shot LLM has a natural advantage over a small fine-tuned model trained on limited data, because it requires understanding argument structure rather than pattern-matching on vocabulary.

---

## AI Usage

- **Label stress-testing:** Used Claude to generate boundary-case posts between `analysis` and `hot_take` to pressure-test the decision rule before annotating 200 examples.
- **Annotation assistance:** Used Claude to pre-label batches of 30–40 posts at a time using the label definitions from `planning.md`. Every pre-assigned label was manually reviewed and corrected before committing. Pre-labeled examples are noted in the `notes` column of `dataset.csv`.
- **Dataset organization:** Used Claude to extract relevant comments from raw Reddit thread text (pasted into a staging file), clean formatting artifacts, and write them into `dataset.csv` with labels and notes.
- **Evaluation analysis:** Used Claude to help interpret the confusion matrix, compute per-class F1 scores from the raw counts, and identify systematic error patterns (over-prediction of `analysis` due to vocabulary proxy learning). All patterns were verified by re-reading the misclassified examples manually before including them in this report.
- **Polish writing:** Used Claude to polish and edit `README.md` to ensure better clarity to the reader.
