# TakeMeter — Planning Document

## Community

**Community:** Counter-Strike 2 (CS2) competitive scene, specifically posts and comments about all teams competing in the most recent CS2 Major tournament. Source: r/GlobalOffensive and r/cs2 on Reddit, primarily match-thread comments and post-match discussion threads.

**Why this community fits:** CS2 tournament discourse is dense with opinion and varies enormously in quality. After a big match, you'll find everything from detailed tactical breakdowns citing round-by-round stats to pure hype reactions to bold unjustified predictions — across all teams and matchups. The community actively distinguishes between "analysts" and "people who just vibe," which means the label distinctions are grounded in norms the community already cares about. Covering all teams gives a broader, more representative dataset and ensures the model learns discourse *style* rather than team-specific vocabulary.

---

## Labels

### `analysis`
**Definition:** The post supports a claim about a team using specific, verifiable evidence — round statistics, player K/D ratios, map win rates, tactical observations tied to specific rounds, or historical team comparisons with numbers.

**Examples:**
- "Vitality's CT-side on Mirage is statistically their worst map this tournament — they're winning only 38% of CT rounds, and their mid control is consistently getting broken by early aggressive pushes."
- "NAVI's in-game leading style favors slow default play, which is why their T-side has struggled against teams that play aggressive early — they've lost 14 of 17 eco rounds this event."

**Uncertain case:** "Falcons' AWPer has been inconsistent this event — 2 great games but went negative in both semis." *(Could be analysis if tied to specific match data, or hot_take if stated as a general pattern without verifiable backing.)*

---

### `hot_take`
**Definition:** A bold, confident opinion about a team stated as fact without supporting evidence. The claim might be true, but the post asserts rather than argues — it tells you what to think rather than showing why.

**Examples:**
- "Falcons are massively overrated — they only got this far because the bracket was soft."
- "Spirit has zero motivation this major, you can see it in their play."

**Uncertain case:** "Falcons' AWPer has been inconsistent — he's had 2 great games but went negative in the semis, and this kind of inconsistency has historically been what holds this team back." *(The word "historically" sounds like a pattern claim but is unsupported — leans hot_take.)*

---

### `reaction`
**Definition:** An immediate emotional response to a specific match moment or result involving any team, with little to no argument. The post is expressing a feeling in real time rather than making a claim about a team.

**Examples:**
- "LETS GOOO FALCONS WINNING THAT CLUTCH ROUND!!!"
- "I can't believe Spirit threw a 15-9 lead. I'm done watching this tournament."

**Uncertain case:** "Oh my god that 1v3 clutch was insane, Falcons might actually win this whole thing." *(Emotional trigger + a mild prediction — label it reaction if the emotional response is the primary content.)*

---

## Hard Edge Case & Decision Rule

**The problem boundary:** `analysis` vs. `hot_take` — posts that cite one specific stat or data point in an otherwise assertion-heavy post.

**Example:** "Falcons' AWPer went negative in the semis and that's proof this roster will never win a major — star players can't go negative in elimination matches." *(applies to any team in the major)*

**Decision rule:** If the post provides specific, verifiable evidence (a stat, a round, a map result) that would support the claim *even if you removed the opinion framing*, label it `analysis`. If the evidence is vague, cherry-picked, or decorative — cited to sound credible but not actually reasoning through the claim — label it `hot_take`. A single cherry-picked stat used to justify a sweeping generalization (`"this roster will never win a major"`) is `hot_take`.

---

## Data Collection Plan

- **Source:** Reddit — r/GlobalOffensive and r/cs2 match threads for all teams across the most recent CS2 Major. Cover a variety of matchups (group stage, playoffs, finals) to get diverse team discourse.
- **Target:** At least 200 labeled posts/comments. Aim for ~70 per label (~35% each) to avoid imbalance.
- **Collection method:** Manual copy-paste into a CSV with columns `text`, `label`, `notes`. Start with Falcons threads to stay grounded, then expand to other team threads.
- **If a label is underrepresented:** Collect additional comments from dedicated post-match discussion threads or from team-specific subreddits (e.g., r/fnatic, r/GlobalOffensive team flairs) where that label type is more common (e.g., `analysis` posts appear more in dedicated discussion threads than in live match threads).

---

## Evaluation Metrics

- **Overall accuracy** — gives a high-level view but is misleading if one class dominates.
- **Per-class F1** — the most important metric here, since the three labels are distinct behaviors and we care about the model learning each boundary, not just majority-class prediction. For example, if 60% of your posts are reaction, a model that just predicts reaction for everything gets 60% accuracy — but its F1 for analysis and hot_take would be 0. Per-class F1 exposes that immediately.
- **Confusion matrix** — needed to understand *which* label pairs are getting confused. A model that perfectly separates `reaction` but conflates `analysis` and `hot_take` is a very different failure mode than one that lumps everything into one bucket.

---

## Definition of Success

A classifier that is genuinely useful for a community moderation or "post quality" tagging tool would need per-class F1 ≥ 0.70 across all three labels on the test set, and must not default to predicting one label for more than 60% of examples. The fine-tuned model should meaningfully outperform the zero-shot Groq baseline — if it doesn't, fine-tuning added nothing and the labels may be too easy or too noisy.

---

## AI Tool Plan

- **Label stress-testing:** Give Claude the three label definitions and edge case description, ask it to generate 10 posts that sit on the boundary between `analysis` and `hot_take`. If it produces posts that can't be cleanly classified, revise the decision rule before annotating 200 examples.
- **Annotation assistance:** Will use an LLM to pre-label batches of 30–40 posts at a time, then manually review and correct every label before committing it. Will track which examples were pre-labeled with a `prelabeled` column in the CSV for disclosure.
- **Failure analysis:** After fine-tuning, paste the wrong predictions into Claude and ask it to identify systematic patterns — specific label pairs being confused, post length effects, sarcasm, or posts where team name appears without any argument. Verify each suggested pattern by re-reading the examples before including it in the evaluation report.
