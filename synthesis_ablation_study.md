# Group Travel Planning: Synthesis and Ablation Study

## Overview

This document describes our approach to **fair group travel planning** through transparent preference negotiation. The key innovation is a **per-slot synthesis mechanism** where the system explains how conflicting preferences are resolved using Social Choice Theory principles.

---

## The Problem

When groups travel together, they often have different preferences:
- **Elder + Young Adult**: Grandparent wants comfort food, young adult wants trendy restaurants
- **Family**: Parents prioritize safety, kids want adventure
- **Strangers**: No shared context for compromises

Traditional planners either:
1. **Ignore conflicts** - Just pick one person's preference
2. **Average preferences** - Generic middle-ground that satisfies no one

Our solution: **Transparent synthesis** that explains HOW preferences are balanced.

---

## Two Planning Approaches

### 1. Passive Aggregator (formerly "Approach B")

> System **silently combines** preferences without explanation

**Flow:**
```
User 1: "I want Italian food"
User 2: "I want Chinese food"
System: [internally picks one]
System: "Here's your plan with Italian restaurant..."
```

**Problem:** User 2 doesn't know why their preference was ignored.

---

### 2. Active Mediator (formerly "Approach A")

> System **transparently explains** how it balances preferences

**Flow:**
```
User 1: "I want Italian food"
User 2: "I want Chinese food"
System: "I notice you have different cuisine preferences. 
        Following the 'least misery' principle for your 
        elder+young group, I'll prioritize comfort food 
        for most meals, with one Chinese dinner when 
        everyone is energized. Does this work?"
User 1: "That sounds fair!"
User 2: "I appreciate you including Chinese too!"
```

**Benefit:** Both users feel heard; resolution is transparent.

---

## Per-Slot Synthesis

Synthesis happens **after each preference slot**, not just at the end.

### Slot Order
1. `traveler_type` - What kind of traveler?
2. `travel_purpose` - Adventure, relaxation, culture?
3. `spending_preference` - Budget, moderate, luxury?
4. `lodging` - Entire home, private room?
5. `cuisine` - Italian, Chinese, local?
6. `event` - Music, theatre, sports?
7. `attraction` - Museums, nature, nightlife?

### Per-Slot Flow

```
┌─────────────────────────────────────────────────────────────┐
│ System: "What cuisine do you prefer?"                        │
├─────────────────────────────────────────────────────────────┤
│ Grandparent: "I want Central-Italian, reminds me of home"   │
│ Young Adult: "I want contemporary, something modern"        │
├─────────────────────────────────────────────────────────────┤
│ SYNTHESIS:                                                   │
│ System: "Since we're using 'least misery' for your          │
│         elder+young group, I'll prioritize comfort food..."  │
├─────────────────────────────────────────────────────────────┤
│ Grandparent: "That sounds perfect!"                         │
│ Young Adult: "I get it—maybe we can find modern Italian?"   │
└─────────────────────────────────────────────────────────────┘
                              ↓
              (Move to next slot: event)
```

---

## Social Choice Theory Strategies

Different group types use different resolution strategies:

| Slot | Elder+Young | Family | Spouses | Strangers |
|------|-------------|--------|---------|-----------|
| **traveler_type** | Least Misery | Parent Decides | Alternate | Cluster |
| **cuisine** | Least Misery | Family-friendly | Log-Rolling | Multi-cuisine |
| **attraction** | Subgroup Optional | Include Both | Log-Rolling | Subgroup |
| **event** | Elder Stamina | Age-appropriate | Alternate | Self-select |
| **spending** | Compromise | Parent HC | Higher Pref | Self-select |

### Strategy Definitions

- **Least Misery**: Minimize the worst-case dissatisfaction (prioritize elder's comfort)
- **Log-Rolling**: Trade-offs across slots ("You pick cuisine, I pick events")
- **Subgroup Optional**: Some activities are optional (young adults can hike alone)
- **Parent HC (Hierarchical)**: Parents have final decision authority

---

## Ablation Study Design

We test each component's contribution through **ablation flags**:

### Experimental Conditions

| Condition | enable_synthesis | use_social_theory | Description |
|-----------|-----------------|-------------------|-------------|
| **1. Passive Aggregator** | `False` | - | No synthesis (baseline) |
| **2. Raw LLM Synthesis** | `True` | `False` | LLM explains without explicit strategies |
| **3. Guided Synthesis** | `True` | `True` | LLM uses named Social Choice strategies |

### Research Questions

1. **RQ1**: Does synthesis improve perceived fairness?
   - Compare Condition 1 vs Conditions 2-3
   
2. **RQ2**: Do explicit strategies improve resolution quality?
   - Compare Condition 2 vs Condition 3

3. **RQ3**: Does transparency affect user satisfaction?
   - Analyze member response sentiment

### Evaluation Metrics

| Metric | Description |
|--------|-------------|
| **Fairness Score** | Are all members' preferences partially addressed? |
| **Transparency Score** | Does system explain WHY choices were made? |
| **Satisfaction Proxies** | Member response sentiment (positive/negative) |
| **Turn Efficiency** | Dialogue length vs resolution quality |

---

## Implementation Details

### Key Methods

| Method | Location | Purpose |
|--------|----------|---------|
| `_synthesize_slot()` | `dialogue_generator_MULTICITYV1_FIXED.py:5981` | Per-slot synthesis |
| `simulate_group_approach_b()` | `dialogue_generator_MULTICITYV1_FIXED.py:5373` | Main orchestrator |
| `format_slot_strategies_for_prompt()` | `synthesis_phase.py` | Strategy formatting |

### Ablation Flags

```python
simulator.simulate_group_approach_b(
    trip_row=row,
    group_type="elder_young",
    member_count=2,
    member_ages=[65, 28],
    # Ablation flags:
    enable_synthesis=True,       # Turn synthesis ON/OFF
    use_social_theory=True,      # Use explicit Social Choice strategies
    enable_feedback=False        # (Legacy) End-of-dialogue feedback
)
```

---

## Artifacts Generated

| File | Description |
|------|-------------|
| `data/perslot_synthesis_dialogue.json` | Full dialogue with all turns |
| `data/perslot_synthesis_dialogue.txt` | Human-readable dialogue transcript |
| `logs/synthesis_phase.log` | Structured logging of synthesis decisions |

---

## Sample Output

### Conflict Detection (Cuisine Slot)

```
DEBUG _synthesize_slot(cuisine): conflict=True, strategy=Least Misery
  → Grandparent prefers Central-Italian; Young Adult prefers contemporary
```

### Generated Dialogue

> **System**: "I understand that you'd like to try traditional Central-Italian food, Grandparent, and Young Adult 1, you're interested in something more contemporary. Since we're using the **'least misery' strategy** for cuisine, we'll prioritize the elder's preference for familiar, comforting food like pasta and breadsticks."

> **Grandparent**: "That sounds perfect! I just want to feel at home with a good plate of lasagna."

> **Young Adult**: "Yeah, I get it—comfort is important. Maybe we can find a place that has both? Like a modern Italian spot with a twist?"

---

## Future Work

1. **POI Grounding**: Replace generic descriptions with actual restaurant/attraction names
2. **Feedback Loop**: Allow members to request adjustments within each slot
3. **Multi-city Negotiation**: Handle city-specific preference conflicts
4. **User Study**: Human evaluation of fairness and satisfaction
