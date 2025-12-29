# Conflict Resolution Strategies by Group Type
*Based on PUMAS-GR, GTTRM Papers + Your System's Slot Values*

---

## System Slot Values Reference

| Slot | Values |
|------|--------|
| **traveler_type** | Adventure Seeker, Adventure Traveler, Laidback Traveler |
| **travel_purpose** | Cultural exploration, Relaxation, Business, Adventure |
| **spending_preference** | Economical Traveler, Budget-friendly, Moderate, Luxury Traveler |
| **room_type** | entire_home, private_room, shared_room |
| **house_rule** | No children under 10, No parties, No pets, No smoking, No visitors |
| **attraction** | (City-specific: Museums, Parks, Historical sites, Adventure activities, etc.) |
| **cuisine** | (City-specific: Indian, Italian, Chinese, etc.) |

---

## 1. Family with Children

### Typical Slot Conflicts

| Slot | Parent | Child | Resolution |
|------|--------|-------|------------|
| **traveler_type** | Laidback | Adventure Seeker | Use **Laidback** (safety) with adventure **mini-activities** |
| **travel_purpose** | Relaxation | Adventure | Split: AM = adventure activity; PM = relaxation |
| **spending** | Moderate | N/A | Parent's value (HC) |
| **attraction** | Cultural sites | Theme parks | **Alternate days** or include both per day |
| **house_rule** | No parties, No smoking | N/A | Parent's HC (non-negotiable) |
| **room_type** | entire_home | N/A | Prefer entire_home for families (HC) |

### GTTRM Strategy
- **Aggregation**: Weighted average (parent weight: 0.7)
- **HC**: Child-friendly house rules, entire_home preferred
- **Subgroups**: Only for brief moments (museum kids zone vs gallery)

---

## 2. Multigenerational (Young + Elderly)

### Typical Slot Conflicts

| Slot | Young | Elderly | Resolution |
|------|-------|---------|------------|
| **traveler_type** | Adventure Seeker | Laidback Traveler | **Subgroup split** |
| **travel_purpose** | Adventure | Cultural exploration | AM split, PM cultural together |
| **spending** | Budget-friendly | Moderate/Luxury | **Separate bookings** or mid-tier compromise |
| **attraction** | Hiking, Adventure sports | Museums, Heritage sites | **GTTRM subgroups**: young → hike, elderly → heritage |
| **room_type** | shared_room (budget) | private_room (comfort) | Book different room types in same accommodation |

### GTTRM Strategy
- **Aggregation**: **Least Misery** (protect elderly's physical limits)
- **HC**: Elderly accessibility needs = absolute veto
- **Subgroups**: Standard mode - split by traveler_type, merge for meals

### Slot-Specific Rules
```
IF elderly.traveler_type == "Laidback" AND young.traveler_type == "Adventure Seeker":
    → Auto-suggest subgroup formation for attractions
    → Shared activities only for: meals, cultural sites, scenic viewpoints
```

---

## 3. Colleagues/Work Groups

### Typical Slot Conflicts

| Slot | Employee A | Employee B | Resolution |
|------|------------|------------|------------|
| **traveler_type** | Adventure Traveler | Laidback Traveler | **Optional activities** - no forced participation |
| **travel_purpose** | Business | Relaxation | Business hours = work; after = individual choice |
| **spending** | Economical | Luxury | **Expense policy HC** or separate personal bookings |
| **cuisine** | Indian | Italian | **PUMAS DD**: find multi-cuisine or alternate meals |
| **attraction** | Nightlife | Cultural | Subgroups post-work hours |

### PUMAS Strategy
- **MCP Negotiation**: Democratic voting for group dinners
- **Desires Distance (DD)**: Pick restaurants closest to everyone's cuisine preference
- **No Dictatorship**: Even senior members don't override (unless expense policy)

### Slot-Specific Rules
```
IF spending preferences differ by >2 tiers:
    → Suggest: Company pays base (Moderate), individual upgrades optional
    
IF attraction preferences conflict:
    → Evening activities = optional subgroups
    → Team activity = consensus-based (PUMAS voting)
```

---

## 4. Spouses/Couples

### Typical Slot Conflicts

| Slot | Partner A | Partner B | Resolution |
|------|-----------|-----------|------------|
| **traveler_type** | Adventure Seeker | Laidback | **Alternating days** or compromise activities |
| **travel_purpose** | Adventure | Relaxation | Day 1: Adventure; Day 2: Spa; Day 3: Both choose together |
| **cuisine** | Indian | Italian | **Multiplicative**: find restaurants both rate >0.5 |
| **attraction** | Museums | Shopping | **Log-rolling**: "I'll do museum if you come hiking tomorrow" |
| **room_type** | private_room | entire_home | Go with higher preference (entire_home) |

### GTTRM Strategy
- **Social Value**: V[A,B] = 0.95 (almost never split)
- **Aggregation**: **Multiplicative** - both must like activity somewhat
- **Subgroups**: Only if explicitly requested ("I want solo spa time")

### Slot-Specific Rules
```
IF traveler_type conflict (Adventure vs Laidback):
    → Propose: "leisurely adventure" activities
    → Examples: scenic hike (not intense), cultural walking tour
    
IF cuisine conflict:
    → Find restaurants serving BOTH cuisines
    → OR alternate: lunch = A's choice, dinner = B's choice
```

---

## 5. Strangers (Tour Groups, Meetups)

### Typical Slot Conflicts

| Slot | Stranger 1 | Stranger 2 | Stranger 3 | Resolution |
|------|------------|------------|------------|------------|
| **traveler_type** | Adventure | Laidback | Adventure | **Cluster**: Subgroup by type |
| **spending** | Budget | Luxury | Moderate | Offer **tier options** per activity |
| **attraction** | Museums | Adventure | Nightlife | **Subgroup formation** by attraction type |
| **cuisine** | Vegetarian | Non-veg | Vegan | Pick restaurants with **all options** (HC) |

### GTTRM Strategy
- **User Aggregation (UA)**: Keep individual profiles, don't merge
- **Social Value**: V[strangers] = 0.2 (low; splitting is fine)
- **Clustering**: Form subgroups by traveler_type similarity

### Slot-Specific Rules
```
ALWAYS:
    → Cluster by traveler_type first
    → "Adventure Seekers" subgroup, "Laidback Travelers" subgroup
    
FOR shared meals:
    → Dietary restrictions = HC (must satisfy all)
    → Cuisine preference = SC (majority vote or multi-cuisine venue)
    
FOR spending conflicts:
    → Offer activity at multiple price points
    → Let individuals self-select
```

---

## Aggregation Strategy by Slot Type

| Slot Category | Family | Multigenerational | Colleagues | Spouses | Strangers |
|---------------|--------|-------------------|------------|---------|-----------|
| **traveler_type** | Parent decides | Least Misery | Self-select | Alternate | Cluster |
| **travel_purpose** | Weighted avg | Subgroup | Business HC | Alternate | Cluster |
| **spending** | Parent HC | Separate bookings | Policy HC | Higher pref | Self-select |
| **cuisine** | Family-friendly | Least Misery | DD/Vote | Multiplicative | Multi-cuisine |
| **attraction** | Include both | Subgroup | Subgroup | Log-roll | Subgroup |
| **room_type** | entire_home HC | Flexible | Per policy | Higher pref | Individual |
| **house_rule** | Parent HC | Elder's needs | Policy | Shared HC | Declare upfront |

---

## Quick Reference: Traveler Type Conflicts

| Combination | Strategy |
|-------------|----------|
| Adventure + Adventure | Easy - shared activities |
| Laidback + Laidback | Easy - relaxed pace for all |
| Adventure + Laidback | **Hybrid activities** (scenic hike, cultural tours) OR **subgroup split** |
| Mixed group (3+) | **Cluster by type** → form subgroups |

---

## References

1. **PUMAS-GR**: MCP, Desires Distance (DD), Zeuthen WRC
2. **GTTRM**: HC/SC, Social Value V[uz,uo], GA vs UA, Subgroups
3. **Your System**: traveler_type, travel_purpose, spending_preference, attractions
