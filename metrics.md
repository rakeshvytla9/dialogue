W1 — Utilitarian Welfare (Average Satisfaction)
Formula:

$$W_1 = \frac{1}{N} \sum_{i=1}^{N} s_i$$

Where $s_i$ is the per-member constraint satisfaction rate for member $i$ (i.e., what fraction of that member's preferences were satisfied in the final plan), and $N$ is the number of group members.

Intuition: "What's the average happiness?" Maximizes total group happiness. A plan could score well here even if one member gets 0% and another gets 100% (average = 50%).

Note: W1 is mathematically identical to M10 (member_condition_satisfaction_rate) — both compute the mean across members.

W2 — Egalitarian Welfare (Worst-off Member)
Formula:

$$W_2 = \min_{i \in {1..N}} s_i$$

Simply the minimum satisfaction across all members.

Intuition: "How happy is the least happy person?" Based on Rawls' maximin principle — a just plan is one that maximizes the welfare of the worst-off member. This is the harshest metric: if even one member gets 0 satisfaction, W2 = 0.

Why it matters for your paper: W2 shows the clearest C4 > C3 treatment effect because social theory strategies like Least Misery and Elder Stamina explicitly protect the worst-off member from being ignored.

W3 — Nash Welfare (Geometric Mean)
Formula:

$$W_3 = \left(\prod_{i=1}^{N} s_i\right)^{1/N}$$

The geometric mean of all members' satisfaction rates.

Intuition: "Is satisfaction balanced?" The geometric mean naturally penalizes inequality — if one member has very low satisfaction, it drags the whole product down multiplicatively. It's a middle ground between W1 (pure average) and W2 (pure worst-case):

More forgiving than W2 (a single low score doesn't zero it out, unless it's literally 0)
More equity-sensitive than W1 (can't compensate a 0 with a 1)


<img width="722" height="584" alt="image" src="https://github.com/user-attachments/assets/413e9349-cbf1-4e54-b74f-eface95a140e" />
<img width="727" height="538" alt="image" src="https://github.com/user-attachments/assets/785a1043-da79-4998-b9a9-58f35d20f6c5" />






# Metric Reference: Data Sources & Calculations

## Data Source Legend

Every metric reads from one dialogue CSV. That CSV has columns like `speaker`, `intent`, `utterance`, `constraint_state`, `plan`, `ref_info_*`. Here's what each data source means:

| Source | What it is | Where in CSV |
|---|---|---|
| **Dialogue turns** | The sequence of system/user utterances and their intents | `speaker`, `intent`, `utterance` columns |
| **Synthesis turns** | System turns with intent `sys_synthesize_*` — where the system merges preferences | Subset of dialogue turns |
| **Feedback turns** | System/user turns with intent `sys_feedback_*` / `user_feedback_*` | Subset of dialogue turns |
| **Plan payload** | The JSON itinerary: days → meals, accommodations, attractions, events, poi_list | `plan` column (last non-empty turn) |
| **Plan raw output** | The planner's full text output before JSON parsing | `plan.raw_output` field |
| **Reference catalog** | Database records of real places (restaurants, hotels, attractions, events) | `ref_info_accommodations`, `ref_info_restaurants`, etc. |
| **User preferences** | What each member said they want, extracted from `user_provide_*` turns | `utterance` of turns with intent `user_provide_lodging`, etc. |
| **Condition flags** | `multi_user`, `enable_synthesis`, `use_social_theory`, `enable_feedback` | Metadata in first turn or inferred from filename |

---

## Group A: Dialogue-Only Metrics (Do NOT use the plan)

These metrics evaluate the *conversation process* — how well the system conducted the dialogue, detected conflicts, and applied strategies. They never look at the final itinerary.

---

### M1 — Member Voice Coverage

**What it measures:** Did every group member get to speak?

**Data used:** `speaker` field of `user_provide_*` turns + `member_count` from metadata

**Calculation:**
- If `multi_user=True`: count distinct speakers who made preference turns → `distinct_speakers / member_count`
- If `multi_user=False` (C1): only 1 spokesperson → `1 / member_count`

**Example:** 4-member family, all 4 speak → M1 = 4/4 = **1.0**. C1 single-user → M1 = 1/4 = **0.25**

---

### M2 — Conflict Detection Rate

**What it measures:** Of the slots where members actually disagreed, did the synthesis phase recognize the conflict?

**Data used:** User preference turns (to find divergent slots) + synthesis turn metadata (`has_conflict` flag or lexical fallback)

**Calculation:**
1. Find "divergent slots" — slots where members stated different preferences
2. For each divergent slot, check if synthesis metadata says `has_conflict=True`, OR if synthesis text contains words like "conflict", "different", "compromise"
3. M2 = `detected_conflicts / divergent_slots`
4. Returns `None` for C1/C2 (no synthesis phase)

**Example:** Members disagree on cuisine, event, attraction (3 divergent slots). Synthesis detects 2 of them → M2 = 2/3 = **0.667**

---

### M3 — Resolution Explanation Rate

**What it measures:** When the system synthesized preferences, did it explain *why* it made its choice?

**Data used:** Synthesis turn metadata (`resolution_explanation` field) + synthesis utterance text

**Calculation:**
1. Find synthesis slots (slots with `sys_synthesize_*` turns)
2. For each, check if `resolution_explanation` is non-empty, OR if the synthesis text has ≥6 tokens
3. M3 = `explained_slots / synthesis_slots`
4. Returns `None` for C1/C2

**Example:** 4 synthesis slots, all have explanations → M3 = 4/4 = **1.0**

---

### M4 — Theory Strategy Usage Rate

**What it measures:** When social theory is enabled (C4/C5), did the system actually apply a named strategy (e.g., "parent_veto", "Include Both") in its synthesis?

**Data used:** Synthesis turn metadata (`resolution_strategy` field) for **actionable slots only** (lodging, cuisine, event, attraction)

**Calculation:**
1. Find actionable slots with synthesis turns
2. For each, check if `resolution_strategy` is a real strategy (not "N/A" or empty)
3. M4 = `slots_with_strategy / actionable_synthesis_slots`
4. Returns 0.0 for C1/C2/C3 (social theory disabled)

**Example:** C4 family — cuisine="Family Friendly", event="parent_veto", attraction="Include Both", lodging="N/A" → 3/4 = **0.75**

> **Note:** Previously used ALL 7 slots in denominator (including traveler_type, travel_purpose, spending_preference which always get N/A). Now fixed to use only 4 actionable slots.

---

### M4b — Theory Text Leakage Rate

**What it measures:** Does the system's language reveal that it's reading from a theory script? (ideally low — theory should guide decisions, not be quoted)

**Data used:** Utterance text from `sys_synthesize_*`, `sys_feedback_*`, and `sys_present_plan` turns

**Calculation:**
1. Collect all candidate system turn texts
2. Search each for theory-specific phrases (regex: "parental authority", "elder stamina", "subgroup", "generational balance", etc.)
3. M4b = `turns_with_theory_phrases / total_candidate_turns`

**Example:** 8 candidate turns, 1 contains "parental authority" → M4b = 1/8 = **0.125**

---

### M5 — Feedback Activation Rate

**What it measures:** For conflict slots, was the feedback loop actually triggered?

**Data used:** Synthesis metadata (`has_conflict`) + feedback turn counts per slot

**Calculation:**
1. Find slots where synthesis flagged a conflict
2. For each, check if there are any `user_feedback_*` or `sys_feedback_*` turns for that slot
3. M5 = `activated_conflict_slots / total_conflict_slots`
4. Returns 0.0 for C1-C4

**Example:** C5 — 3 conflict slots, all 3 get feedback → M5 = 3/3 = **1.0** (always 1.0 by current design)

---

### M6 — Member Acceptance Rate

**What it measures:** Did members accept the final plan?

**Data used:** `user_accept_plan` or `user_approve_approach` intent turns

**Always 1.0** in current implementation (scripted acceptance). Diagnostic only.

---

### M7 — Dialogue Efficiency

**What it measures:** How many turns did the dialogue take?

**Data used:** Total turn count, slot count, feedback turn count

**Output:** Dictionary: `{total_turns, turns_per_slot, feedback_turn_overhead}` — not a single number

---

## Group B: Plan-Dependent Metrics (USE the final itinerary)

These metrics evaluate the *output quality* — how good the final travel plan is. They read the generated plan JSON and compare it against user preferences and database records.

---

### M8 — Preference Addressing Score (combined)

**What it measures:** Were user preferences *mentioned* in the system's synthesis text OR the plan text?

**Data used:** 🔵 User preferences + 🟡 Synthesis turn text + 🟢 Plan presentation text (sys_present_plan utterances + plan raw output)

**Calculation:**
1. For each member's preference in each slot, extract top-6 informative tokens
2. Check if those tokens appear (lexical) in synthesis text OR plan text, or if semantic similarity ≥ 0.45
3. A preference is "addressed" if: phrase hit, or ≥2 token hits, or 1 token when only 1 informative token, OR semantic similarity above threshold
4. M8 = `addressed_pairs / stated_pairs`

**New splits:**
- **M8a** = Same calculation but searches *only* synthesis text → tells you "did the system acknowledge?"
- **M8b** = Same calculation but searches *only* plan text → tells you "did the plan reflect it?"

**Example:** 28 preference pairs, M8=0.58, M8a=0.54 (synthesis echo), M8b=0.22 (plan reflection) → Most of the M8 score came from the synthesis, not the plan.

---

### M9 — Plan DB Grounding Rate

**What it measures:** Are the places in the plan real places from the reference database?

**Data used:** 🟢 Plan payload (place names extracted from all days) + 🔴 Reference catalog (all known place names)

**Calculation:**
1. Extract all place names from plan's meals, accommodations, attractions, events, poi_list
2. Normalize names (lowercase, strip punctuation)
3. Check each against the reference catalog
4. M9 = `grounded_places / total_plan_places`

**Example:** Plan mentions 15 places, all 15 are in the database → M9 = 15/15 = **1.0**

---

### M10 — Member Slot Satisfaction Rate

**What it measures:** For each member's preference in each actionable slot (lodging/cuisine/event/attraction), did the plan actually satisfy it?

**Data used:** 🔵 User preferences + 🟢 Plan payload + 🔴 Reference catalog (place records with categories, cuisines, room types) + 🟢 Plan raw output

**Calculation (multi-method — any one passing = satisfied):**

| Method | How it works |
|---|---|
| **Room type** (lodging only) | User says "private room" → check if selected accommodation's `roomType` field matches. Uses *semantic intent matching* to detect what user actually wants vs. negated mentions. |
| **Category lookup** | Extract category tokens from preference ("Italian", "Diner", "Sports") and from DB records of selected places. If any overlap → satisfied. |
| **Lexical** | Extract top-6 informative tokens from preference, search in plan places + their DB records + plan raw output. ≥2 hits = satisfied. |
| **Semantic** | Compute sentence-transformer embedding similarity between preference text and plan place records. If max similarity ≥ 0.42 → satisfied. |

Final: `M10 = satisfied_member_slot_pairs / total_member_slot_pairs`

**Example:** 4 members × 4 slots = 16 pairs. 14 satisfied → M10 = 14/16 = **0.875**

---

### M10b — Category Lookup Satisfaction Rate

**What it measures:** Among slots where category matching is applicable (cuisine, event, attraction — NOT lodging), did the DB record's category match the preference?

**Data used:** Same as M10, but only the category-lookup channel

**Calculation:** `category_satisfied_pairs / category_applicable_pairs`

**Example:** 12 pairs have category tokens, 8 match → M10b = 8/12 = **0.667**

---

### M11 — All-Constraints Satisfied Rate

**What it measures:** What fraction of members had ALL their actionable slot preferences satisfied?

**Data used:** Same as M10 (aggregated per member)

**Calculation:**
1. For each member, check if ALL their slots passed M10
2. M11 = `fully_satisfied_members / total_members`

**Example:** 4 members, 2 have all 4 slots satisfied → M11 = 2/4 = **0.50**

> **Interpretation note:** C1 (1 spokesperson) often gets M11=1.0 because it's easier to satisfy 1 person. C2-C5 (4 members) get lower M11 because ALL 4 must be fully satisfied. This is a base-rate effect, not a quality signal.

---

### M12 — Semantic Preference Satisfaction

**What it measures:** How semantically similar is each member's preference to what the plan actually contains for that slot?

**Data used:** 🔵 User preferences + 🟢 Plan slot-specific text (extracted meals/accommodations/attractions) + 🟢 Plan raw output

**Calculation:**
1. For each member-slot pair, extract plan text for that slot using `_plan_slot_text` (e.g., for cuisine: all breakfast/lunch/dinner values from all days)
2. Concatenate with plan raw output
3. Compute semantic similarity (sentence-transformers embedding cosine) between preference text and this target
4. M12 = `mean(all_semantic_scores)`

**Example:** 16 preference pairs, average cosine similarity = 0.39 → M12 = **0.39**

> **Known issue:** Target text includes full plan raw output per slot, adding noise. The slot-specific extraction is correct but diluted.

---

### M13 — Fairness Gini Coefficient

**What it measures:** How fairly distributed are the M12 semantic satisfaction scores across members? (0 = perfectly equal, 1 = maximally unequal)

**Data used:** Per-member average M12 scores

**Calculation:**
1. Compute each member's average semantic satisfaction (from M12 pair scores)
2. Apply Gini coefficient formula: `Σ|xi - xj| / (2n²μ)` over all pairs

**Example:** 4 members with averages [0.38, 0.41, 0.39, 0.40] → very low Gini ≈ **0.02** (fair)

---

### M14 — Concession Rate

**What it measures:** How well does the negotiated resolution align with each member's original preference?

**Data used:** 🔵 User preferences + 🟡 Synthesis `resolution_explanation` text (ONLY — no plan fallback)

**Calculation:**
1. For each member-slot pair, get the synthesis resolution explanation
2. Compute semantic similarity between preference text and resolution text
3. M14 = `mean(all_alignment_scores)`
4. Returns **None** for C1/C2 (no synthesis = no negotiation)

**Example:** C4 — member said "Barbecue", resolution says "family-friendly restaurant" → similarity ~0.35. Average across all pairs → M14 = **0.51**

> **Note:** Previously C1/C2 fell back to plan raw output, making cross-condition comparison invalid. Now fixed.

---

### M15 — Serendipity Score

**What it measures:** What fraction of grounded plan places were NOT explicitly requested by anyone but still semantically relevant?

**Data used:** 🔵 All user preference texts + 🟢 Grounded plan places + 🔴 Reference catalog (place records)

**Calculation:**
1. For each grounded place, check if its normalized name appears in any preference text (explicit mention)
2. If NOT explicitly mentioned, check semantic similarity between the place's DB record and all preference texts
3. If max similarity ≥ 0.30 (SERENDIPITY_THRESHOLD) → serendipitous
4. M15 = `serendipitous_places / grounded_places`

**Example:** 12 grounded places, 2 explicitly mentioned, 8 of remaining 10 are semantically relevant → M15 = 8/12 = **0.67**

---

### M16 — Plan Coherence Score

**What it measures:** Is the plan structurally sound? (meals filled, variety, temporal consistency)

**Data used:** 🟢 Plan payload only

**Checks per day:**

| Check | What it verifies |
|---|---|
| Breakfast filled | Not empty or "-" |
| Lunch filled | Not empty or "-" |
| Dinner filled | Not empty or "-" |
| Meal variety | Lunch ≠ Dinner (different restaurants) |
| POI temporal order | Time slots in poi_list are sequential (start times increase, end > start) |

**Calculation:** `M16 = passed_checks / total_checks`

**Example:** 5-day plan: 5×4 structural checks + 3 temporal checks = 23 total. 21 pass → M16 = 21/23 = **0.91**

---

## Quick Reference: Which Metrics Use What

| Metric | Dialogue | Synthesis | Plan JSON | Plan Text | DB Refs | Condition |
|---|:---:|:---:|:---:|:---:|:---:|---|
| M1 Voice | ✅ | | | | | All |
| M2 Conflict | ✅ | ✅ | | | | C3-C5 only |
| M3 Explanation | | ✅ | | | | C3-C5 only |
| M4 Strategy | | ✅ | | | | C4-C5 only |
| M4b Leakage | | ✅ | | ✅ | | All |
| M5 Feedback | | ✅ | | | | C5 only |
| M6 Acceptance | ✅ | | | | | All (always 1.0) |
| M7 Efficiency | ✅ | | | | | All |
| **M8 Addressing** | | **✅** | | **✅** | | All |
| **M8a Synth-only** | | **✅** | | | | All |
| **M8b Plan-only** | | | | **✅** | | All |
| **M9 Grounding** | | | **✅** | | **✅** | All |
| **M10 Satisfaction** | | | **✅** | **✅** | **✅** | All |
| **M10b Category** | | | **✅** | | **✅** | All |
| **M11 All-Satisfied** | | | **✅** | **✅** | **✅** | All |
| **M12 Semantic** | | | **✅** | **✅** | | All |
| **M13 Fairness** | | | **✅** | **✅** | | All |
| **M14 Concession** | | **✅** | | | | C3-C5 only |
| **M15 Serendipity** | | | **✅** | | **✅** | All |
| **M16 Coherence** | | | **✅** | | | All |


