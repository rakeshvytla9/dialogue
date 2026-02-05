# Group Dialogue Generation: Research Framework

## Research Motivation

Traditional travel planning datasets assume a **single user** provides all preferences. This fails to capture group travel where multiple individuals must negotiate.

---

## The Three Approaches

### 1. Representative Baseline ("The Dictator")

One user provides all inputs. Fast but unfair to silent members.

```
User: "Plan a trip for my family of 4."
System: [Generates plan based on one perspective]
```

❌ **Failure**: Elder's need for rest ignored, child's preferences invisible.

---

### 2. Passive Aggregator

All members provide preferences, but system silently combines them without explanation.

```
Grandparent: "Comfort food"
Young Adult: "Hungarian goulash"
System: [Plan generated without explaining trade-offs]
```

❌ **Failure**: Unexplained choices → user distrust.

---

### 3. Active Mediator (Our Innovation)

System uses **per-slot synthesis** to explain how conflicts are resolved using Social Choice Theory strategies.

```
[Preferences collected]
System: "Using 'least misery' for your elder+young group,
        I'll prioritize comfort food for most meals,
        with one Hungarian dinner when everyone is energized."
Grandparent: "That sounds fair!"
Young Adult: "Works for me!"
```

✅ **Success**: Verifiable fairness + transparent consensus.

---

## Per-Slot Synthesis

After EACH preference slot:
1. **Detect**: Are there conflicting preferences?
2. **Resolve**: Apply group-appropriate strategy
3. **Explain**: System tells users how it balanced
4. **Confirm**: Members respond naturally

---

## Social Choice Strategies

| Strategy | When Used |
|----------|-----------|
| **Least Misery** | Prioritize most constrained member (elder comfort) |
| **Log-Rolling** | Trade slots: "You pick cuisine, I pick events" |
| **Subgroup Optional** | Split activities (young hike alone) |
| **Parent HC** | Parent has final say on budget/safety |

---

## Ablation Study

| Condition | enable_synthesis | use_social_theory |
|-----------|-----------------|-------------------|
| Passive Aggregator | `False` | - |
| Raw LLM | `True` | `False` |
| Guided (Active Mediator) | `True` | `True` |

---

## Key Files

| File | Purpose |
|------|---------|
| `dialogue_generator_MULTICITYV1_FIXED.py` | Main logic |
| `synthesis_phase.py` | Strategies |
| `docs/synthesis_ablation_study.md` | Full documentation |
| `data/perslot_synthesis_dialogue.txt` | Sample output |
