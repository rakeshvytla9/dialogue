# Strategy-Locked Dialogue Generation: Full Methodology

## Overview

For each group dialogue, the system generates a **per-slot negotiation** across 7 slots, where the conversation structure *adapts* depending on:
- The **condition** (C1–C5 ablation)
- The **group type** (family / multigenerational / spouses / colleagues / strangers)
- The **canonical strategy** assigned to that slot

The 7-slot negotiation order is fixed:

```
traveler_type → travel_purpose → spending_preference → cuisine → attraction → event → lodging
```

---

## Condition Behaviour (Locked)

| Condition | Synthesis | Strategy Lock | Vote Round | Feedback | Subgroup Assignment |
|-----------|-----------|---------------|------------|----------|---------------------|
| **C1** (single user) | ✗ | ✗ | ✗ | ✗ | ✗ |
| **C2** (approach B) | ✗ | ✗ | ✗ | ✗ | ✗ |
| **C3** (raw synthesis) | ✓ raw | ✗ | ✗ | ✗ | ✗ |
| **C4** (guided synthesis) | ✓ canonical | ✓ | ✓ (if eligible) | ✗ | ✓ (if required) |
| **C5** (active mediator) | ✓ canonical | ✓ | ✓ (if eligible) | ✓ | ✓ (if required) |

---

## Canonical Strategy Mapping (C4/C5 Only)

Each group type × slot combination has a deterministic strategy assignment.

### Family

| Slot | `strategy_id` | `operator` | Rule |
|------|--------------|------------|------|
| traveler_type | `hard_constraint` | `hard_constraint` | Parent-decides behaviour |
| travel_purpose | `weighted_avg` | `weighted_avg` | Blend preferences by weighted support |
| spending_preference | `parent_budget_hc` | `hard_constraint` | Parent budget guardrail is binding |
| cuisine | `include_both` | `include_both` | Include both sides across the itinerary |
| attraction | `include_both` | `include_both` | Include both sides across the itinerary |
| event | `parent_veto_hc` | `hard_constraint` | Parent veto for age-appropriateness |
| lodging (room_type) | `parent_vote_lower_misery` | `parent_vote_lower_misery` | Parent-only vote; tie-break by lower misery |
| lodging (house_rule) | `hard_constraint` | `hard_constraint` | Parent safety constraints are binding |

### Multigenerational (`elder_young`)

| Slot | `strategy_id` | `operator` | Rule |
|------|--------------|------------|------|
| traveler_type | `least_misery` | `least_misery` | Minimize strongest dissatisfaction |
| travel_purpose | `subgroup_optional` | `subgroup_optional` | Shared backbone with optional branch |
| spending_preference | `self_select` | `self_select` | Self-select around a shared budget envelope |
| cuisine | `least_misery` | `least_misery` | Minimize strongest dissatisfaction |
| attraction | `subgroup_required` | `subgroup_required` | Split required; explicit branch assignments |
| event | `elder_stamina_hc` | `hard_constraint` | Elder stamina/accessibility constraints binding |
| lodging (room_type) | `flexible_shared_first` | `flexible_shared_first` | Prefer shared lodging; allow flexibility |
| lodging (house_rule) | `elder_accessibility_hc` | `hard_constraint` | Elder accessibility constraints binding |

### Colleagues

| Slot | `strategy_id` | `operator` | Rule |
|------|--------------|------------|------|
| traveler_type | `self_select` | `self_select` | Members self-select style within group constraints |
| travel_purpose | `weighted_avg` | `weighted_avg` | Weighted blend across members |
| spending_preference | `budget_guardrail_hc` | `hard_constraint` | Budget policy is binding |
| cuisine | `approval_vote_plus_coverage` | `approval_vote_plus_coverage` | Top collective option + minority coverage |
| attraction | `subgroup_required` | `subgroup_required` | Split required; explicit branch assignments |
| event | `subgroup_optional` | `subgroup_optional` | Shared backbone with optional branch |
| lodging (room_type) | `approval_vote_plus_coverage` | `approval_vote_plus_coverage` | Approval vote with minority coverage commitment |
| lodging (house_rule) | `shared_rules_intersection_hc` | `hard_constraint` | Shared rules intersection is binding |

### Spouses

| Slot | `strategy_id` | `operator` | Rule |
|------|--------------|------------|------|
| traveler_type | `alternate` | `alternate` | Alternate preference ownership across days |
| travel_purpose | `alternate` | `alternate` | Alternate preference ownership across days |
| spending_preference | `higher_pref_with_compensation` | `higher_pref_with_compensation` | Higher preference with compensating trade-off |
| cuisine | `mutual_interest` | `mutual_interest` | Select overlap of shared interest |
| attraction | `log_roll` | `log_roll` | Trade slots across days |
| event | `mutual_interest` | `mutual_interest` | Select overlap of shared interest |
| lodging (room_type) | `higher_pref_with_compensation` | `higher_pref_with_compensation` | Higher preference with compensating trade-off |
| lodging (house_rule) | `shared_rules_intersection_hc` | `hard_constraint` | Shared rules intersection is binding |

### Strangers

| Slot | `strategy_id` | `operator` | Rule |
|------|--------------|------------|------|
| traveler_type | `cluster` | `cluster` | Cluster similar preferences |
| travel_purpose | `cluster` | `cluster` | Cluster similar preferences |
| spending_preference | `self_select` | `self_select` | Self-select within shared constraints |
| cuisine | `subgroup_required` | `subgroup_required` | Split required; explicit branch assignments |
| attraction | `subgroup_required` | `subgroup_required` | Split required; explicit branch assignments |
| event | `subgroup_optional` | `subgroup_optional` | Shared backbone with optional branch |
| lodging (room_type) | `individual_choice` | `individual_choice` | Individual room choice where feasible |
| lodging (house_rule) | `declare_upfront_hc` | `hard_constraint` | House rules must be declared and respected |

---

## Pipeline: Phase by Phase

### Phase 1 — Preference Collection (All Conditions)

Each member states their preference for each of the 7 slots in order. The system collects these into a per-slot dictionary:

```python
slot_preferences = {
    "traveler_1": "I prefer diners with classic American food",
    "traveler_3": "I love barbecue restaurants",
    "traveler_4": "I want a brew pub experience"
}
```

**Output per member per slot:**
```json
{"speaker": "Young Adult 1", "intent": "user_provide_cuisine", "text": "I love barbecue restaurants"}
```

---

### Phase 2 — Synthesis (C3/C4/C5)

For each slot, the system calls `_synthesize_slot()`.

#### 2a. Strategy Injection (C4/C5)

The synthesis prompt includes the full strategy table plus a per-slot canonical directive:

```
== SOCIAL THEORY STRATEGIES FOR ELDER_YOUNG ==
| slot | strategy_id | operator | rule |
|---|---|---|---|
| cuisine | least_misery | least_misery | minimize strongest dissatisfaction |
| attraction | subgroup_required | subgroup_required | split required; explicit branch assignments |
...

Use this canonical strategy for "cuisine" in this turn:
- strategy_id: least_misery
- operator: least_misery
- operator style: pick option minimizing strongest dissatisfaction
```

For **C3**, the prompt instead says:
> *"Use your best judgment to propose a fair resolution. Do not mention named social-choice theories, strategy names, or principles. Use plain reasoning only."*

#### 2b. LLM Generates Synthesis

The LLM outputs structured JSON:

```json
{
  "has_conflict": true,
  "conflict_description": "Members prefer different cuisine types",
  "resolution_strategy": "least_misery",
  "resolution_explanation": "We'll prioritize elder comfort with a diner that also has barbecue options",
  "dialogue": [
    {"speaker": "System", "intent": "sys_synthesize_cuisine", "text": "Since we want everyone comfortable..."},
    {"speaker": "Grandparent 1", "intent": "user_respond_cuisine", "text": "That sounds great, I love diners."},
    {"speaker": "Young Adult 1", "intent": "user_respond_cuisine", "text": "Works for me if they have good pulled pork."}
  ]
}
```

#### 2c. Strategy Clamping (C4/C5)

After the LLM responds, the `resolution_strategy` is **overwritten** with the canonical ID:

```python
if use_social_theory:
    strategy = canonical_strategy_id  # Eliminates LLM drift
```

This prevents the LLM from outputting vague labels like `"attraction"`, `"Family Friendly"`, or `"policy"`.

#### 2d. Guided System Text (Conversation Contract)

The System turn is rewritten to follow a strict format:

```
Conflict: Conflict is present in cuisine preferences.
Strategy: least_misery.
Decision: We'll prioritize elder comfort with a diner that also has barbecue options.
Reason: Members prefer different cuisine types.
Next step: Collect one ballot from each member, then apply the operator deterministically.
```

This ensures every C4/C5 system turn follows: **Conflict → Strategy → Decision → Reason → Next Step**.

---

### Phase 3 — Vote Round (C4/C5, When Eligible)

#### Trigger Conditions

A vote round runs when ALL of:
- `use_social_theory = True`
- `has_conflict = True`
- `operator ∈ _VOTE_OPERATORS`

The eligible operators are:
- `approval_vote_plus_coverage`
- `weighted_avg`
- `parent_vote_lower_misery`
- `least_misery`
- `mutual_interest`
- `higher_pref_with_compensation`

#### 3a. Ballot Collection (LLM)

The system prompts the LLM to collect structured ballots from each member:

```json
{
  "dialogue": [
    {"speaker": "System", "intent": "sys_request_vote_cuisine", "text": "Let's formalize everyone's position on dining."},
    {"speaker": "Grandparent 1", "intent": "user_vote_cuisine", "text": "top_choice: Diner; acceptable: American; hard_no: Spicy food"}
  ],
  "ballots": [
    {"member": "Grandparent 1", "top_choice": "Diner", "acceptable": ["American"], "hard_no": ["Spicy"]},
    {"member": "Young Adult 1", "top_choice": "Barbecue", "acceptable": ["American", "Diner"], "hard_no": []},
    {"member": "Young Adult 2", "top_choice": "Brew Pub", "acceptable": ["Barbecue"], "hard_no": ["Diner"]}
  ]
}
```

#### 3b. Deterministic Vote Summary

The system scores each candidate option against each member's preference text using **window-based polarity scoring**:

- **Full phrase match** in preference text → `+2`
- **Negation cue** within a 4-word window of the match → `−4`
- **Positive cue** before match → `+2`
- **Weak overlap** (partial word match, no full phrase) → `+1 per word overlap`
- **Strong rejection threshold**: net score `≤ −2`

Negation cues: `no`, `not`, `don't`, `without`, `avoid`, `skip`, `exclude`, `hate`, `dislike`
Positive cues: `want`, `prefer`, `love`, `enjoy`, `like`, `need`

**Output:**
```json
{
  "method": "deterministic_vote_parser",
  "strategy_id": "least_misery",
  "selected_option": "diner",
  "tie_break": "score_then_lexicographic",
  "option_details": [
    {"option": "diner", "total_score": 5, "member_scores": {"t1": 4, "t3": 1, "t4": 0}, "strong_rejections": ["t4"]},
    {"option": "barbecue", "total_score": 4, "member_scores": {"t1": 0, "t3": 4, "t4": 0}, "strong_rejections": []},
    {"option": "brew pub", "total_score": 2, "member_scores": {"t1": 0, "t3": 0, "t4": 2}, "strong_rejections": []}
  ],
  "minority_members": ["traveler_3", "traveler_4"],
  "member_primary_option": {"traveler_1": "diner", "traveler_3": "barbecue", "traveler_4": "brew pub"}
}
```

**Special case — Family lodging (`parent_vote_lower_misery`):**
1. Extract parent member IDs by role
2. Only parents vote — children excluded
3. Each parent ranks options by their score
4. Top option by parent vote count wins
5. Ties broken by fewest strong rejections, then lexicographic

#### 3c. Coverage Commitment (Minority Inclusion)

For members whose preferred option was *not* selected:

```json
{
  "slot": "cuisine",
  "minority_targets": {
    "traveler_3": "barbecue",
    "traveler_4": "brew pub"
  },
  "commitment": "Include at least one minority-supported choice in a later day/phase when feasible."
}
```

---

### Phase 4 — Subgroup Flags + Assignment (C4/C5, Eligible Slots)

#### Subgroup Flag Derivation

Subgroup flags are **strictly derived** from canonical strategy + eligible slot + conflict status.

**Rules:**
- Only eligible slots: `{cuisine, attraction, event}`
- `subgrouping_required = True` iff `strategy_id == "subgroup_required"` AND `has_conflict == True`
- `subgrouping_optional = True` iff `strategy_id == "subgroup_optional"` AND `has_conflict == True`
- Otherwise both `False`
- Never derived from free-text phrases like "split" or "subgroup"
- Never gated by `split_allowed` or group attributes

#### Subgroup Member Assignment (when `subgroup_required = True`)

Uses **preference-nearest-split** with stable tie-break:

1. Take the top-2 candidate options from the vote summary
2. For each member, score preference text against both options
3. Assign to the higher-scoring option's group
4. Tied scores: alternate by member_id index (even → Group A, odd → Group B)
5. Ensure both groups are non-empty (move one member if needed)

```json
{
  "slot": "attraction",
  "method": "preference_nearest_split_with_stable_tiebreak",
  "group_a": {
    "anchor": "nature parks",
    "members": [
      {"user_id": "traveler_1", "name": "Grandparent 1"},
      {"user_id": "traveler_2", "name": "Grandparent 2"}
    ]
  },
  "group_b": {
    "anchor": "outdoor activities",
    "members": [
      {"user_id": "traveler_3", "name": "Young Adult 1"},
      {"user_id": "traveler_4", "name": "Young Adult 2"}
    ]
  }
}
```

---

### Phase 5 — Feedback Round (C5 Only)

When `enable_feedback = True` and `has_conflict = True`, the system runs targeted feedback rounds.

#### Flow per round:
1. One member reacts to the current resolution
2. System responds and potentially adjusts
3. If already acceptable → `resolved = True`, loop exits

#### Targeted feedback question:
The system identifies members whose preference tokens have **zero overlap** with the resolution text and asks them directly.

#### Output per round:
```json
{
  "resolved": true,
  "updated_resolution_explanation": "Adjusted to include a barbecue lunch on Day 2",
  "targeted_feedback_question": "Young Adult 1, does adding a barbecue lunch on Day 2 address your concern?",
  "final_revised_decision": "Diner for dinner, barbecue for Day 2 lunch",
  "dialogue": [
    {"speaker": "Young Adult 1", "intent": "user_feedback_cuisine", "text": "I'd feel better if we could at least do barbecue for one lunch."},
    {"speaker": "System", "intent": "sys_feedback_cuisine", "text": "We'll add a barbecue lunch on Day 2 to cover that preference."}
  ]
}
```

---

### Phase 6 — Planner Handoff

All slot results are compiled into a structured JSON payload sent to the TripCraft planner.

#### Per-slot payload:

```json
{
  "cuisine": {
    "has_conflict": true,
    "source": "feedback",
    "feedback_rounds": 1,
    "resolution_strategy": "least_misery",
    "strategy_id": "least_misery",
    "operator": "least_misery",
    "resolution_text": "Diner for dinner, barbecue for Day 2 lunch",
    "subgrouping_required": false,
    "subgrouping_optional": false,
    "vote_summary": {
      "method": "deterministic_vote_parser",
      "selected_option": "diner",
      "option_details": [...]
    },
    "coverage_commitment": {
      "minority_targets": {"traveler_3": "barbecue"},
      "commitment": "Include minority choice in later day/phase."
    },
    "subgroup_assignment": null
  },
  "attraction": {
    "has_conflict": true,
    "strategy_id": "subgroup_required",
    "operator": "subgroup_required",
    "subgrouping_required": true,
    "subgroup_assignment": {
      "group_a": {"anchor": "nature parks", "members": [...]},
      "group_b": {"anchor": "outdoor activities", "members": [...]}
    },
    "vote_summary": {...},
    "coverage_commitment": null
  }
}
```

#### Planner instructions:

```
1. Constraint State is authoritative and contains only user preferences.
2. SLOT DECISION STATE is authoritative for strategy lock fields.
3. feedback_overrides > slot_decision_state > constraint_state
4. Fill Breakfast/Lunch/Dinner with concrete restaurant names from references.
5. When subgrouping_required=true, output explicit:
   - Shared Backbone: common timeline all members follow
   - Group A: explicit branch assignments
   - Group B: explicit branch assignments
   - Rejoin: point where both groups merge back
6. Slot-only split — meals/lodging stay shared unless explicit user override.
```

#### Subgroup signals summary (fed to planner):

```json
{
  "required_slots": ["attraction"],
  "optional_slots": ["travel_purpose"]
}
```

---

## Operator-Specific Conversation Styles

Each operator triggers a distinct conversation style in the synthesis system turn:

| Operator | Style directive |
|----------|----------------|
| `hard_constraint` | Constraint violations are removed first |
| `approval_vote_plus_coverage` | Pick top collective option and commit minority inclusion later |
| `weighted_avg` | Use weighted support and include minority-friendly allocations |
| `least_misery` | Pick option minimizing strongest dissatisfaction |
| `subgroup_required` | Split is required with explicit branch assignments |
| `subgroup_optional` | Shared backbone with optional split branch |
| `alternate` | Trade preference ownership across days/slots |
| `log_roll` | Trade wins across days/slots |
| `parent_vote_lower_misery` | Parents decide; ties use lower-misery tie-break |

---

## Key Design Decisions

1. **Strategy clamping eliminates drift.** The LLM can output any strategy name it wants — the code overwrites it with the canonical ID. This makes the `resolution_strategy` field deterministic and auditable.

2. **Subgroup flags are strict.** Only derived from `strategy_id ∈ {subgroup_required, subgroup_optional}` + eligible slot + conflict. Never from free-text phrases.

3. **Vote rounds are deterministic.** After the LLM collects ballots, the scoring is algorithmic (window-based polarity with negation detection). No LLM judge for final selection.

4. **Coverage commitment is structural.** Minority members are identified by comparing their primary option against the selected option. The commitment to include their preference later is explicit metadata, not a vague prompt hint.

5. **C3 purity is preserved.** No strategy names, no theory terms, subgroup flags forced False, all theory lexicon stripped from resolution text.

---

## Files Involved

| File | Responsibility |
|------|---------------|
| `synthesis_phase.py` | Canonical strategy definitions per group × slot; prompt formatting |
| `dialogue_generator_MULTICITYV1_FIXED.py` | Full pipeline: synthesis, voting, feedback, subgroup assignment, planner handoff |
| `group_models.py` | Group type definitions, member weights, social theory descriptions |
