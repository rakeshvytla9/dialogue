# Practical Cuisine Conflict Resolution Guide
*Real-World Strategies to Replace Abstract Algorithms*

---

## Scenario 1: The "Deadlock" (3 Indian vs 3 Italian)
*Imagine 6 colleagues: 3 want Indian, 3 want Italian.*

### Strategy A: Log-rolling (Time Trading) - *Best for Team Bonding*
Since the split is equal, no side should "lose." PUMAS suggests **taking turns**.
*   **System Action:** "Since the group is evenly split, I recommend we do **Italian for Lunch** today and **Indian for Dinner** (or tomorrow's lunch). This way, everyone gets their top choice once."
*   **Why:** Fairness (Individual Fairness principle).

### Strategy B: Subgroup Formation (GTTRM) - *Best if Time-Constrained*
Colleagues often have different schedules. GTTRM allows splitting because the "connection value" (V) is lower than family.
*   **System Action:** "It looks like we have two strong preferences. I can book tables at **Curry House** and **Pasta Place**—they are just 5 minutes apart. You can split up for this meal and regroup for the afternoon meeting. Shall I do that?"
*   **Why:** Efficiency and maximizing individual satisfaction (Most Pleasure).

---

## Scenario 2: The "Fragmentation" (5 Colleagues, 5 Cuisines)
*Imagine 5 colleagues want: Indian, Italian, Chinese, Thai, Burger.*

### Strategy A: Bridging (Finding the Common Denominator)
PUMAS "Desires Distance" (DD) seeks the option closest to *everyone*.
*   **System Action:** "Everyone has quite different tastes! To keep the team together, I recommend **[Name of Food Court / Multi-Cuisine Buffet]**. They offer Indian, Asian, and Continental counters, so everyone finds something they like."
*   **Why:** Minimizes "Desires Distance" (aggregate dissatisfaction).

### Strategy B: Borda Count Voting (Finding the "Least Hated")
Plurality voting fails here (1 vote each). You need a ranked vote.
*   **System Action:** "It's a mix! I'm sending a quick poll. Please rank these 3 options (Mediterranean, American, Fusion) from 1 to 3. We'll go with the one that gets the best overall score, even if it's not your #1."
*   **Why:** Finds the consensus "safe" choice that no one hates.

---

## 1. The "Bridging" Strategy (Common Denominator)

Instead of calculating "desires distance," find a venue that **physically bridges** the gap.

### A. Multi-Cuisine Venues
**Best for:** Large groups with polarized tastes (e.g., 3 Indian vs 3 Italian).
- **Food Halls / Food Courts**: High-end food halls allow everyone to grab what they want and sit together.
- **Hotel Buffets**: Usually have Indian, Continental, and Asian sections.
- **Global Fusion Restaurants**: Menus specifically designed to offer "something for everyone."

### B. The "Safe Bridge" Options
Certain cuisines act as neutral ground for diverse palates:
- **Mediterranean/Lebanese**: Often acceptable to both vegetarians and meat-eaters, and less polarizing than spicy cuisines.
- **Modern Café/Bistro**: Offers pasta (Italian-adjacent), burgers (Western), and often curry bowls (Asian-adjacent).

---

## 2. Practical Voting Mechanisms

When you need a decision, use these simple voting structures instead of complex negotiation protocols.

### A. The "Veto" First Vote
Before asking what people *want*, ask what they **hate**.
1. "Is there anything anyone absolutely **cannot** eat?" (Allergies, strong dislikes)
2. Filter those out immediately.
3. Vote on the remaining options.

### B. Approval Voting (The "Okay" List)
Don't ask for the #1 choice. Ask: **"Which of these would you be OK with?"**
- **Scenario**: 5 people, 3 options (Indian, Thai, Pizza).
- **Indian**: 3 Loves, 2 Hates → **Winner? NO (2 unhappy)**
- **Pizza**: 0 Loves, 5 Okays → **Winner? YES (0 unhappy)**

> **Real-World Rule**: Satisfaction is maximized by minimizing misery, not maximizing delight.

---

## 3. Time-Based Compromise (Alternating)

**Best for**: Longer trips (multi-day).
- **Rule**: "Today is Person A's choice, Tomorrow is Person B's choice."
- **Why it works**: Fairness is achieved over *time*, not per meal.
- **System Dialogue**: "Since we have a split preference, let's do Italian for lunch today (Group A's pick) and Indian for dinner (Group B's pick)."

---

## 4. Subgrouping (The "Split")

**Best for**: Quick meals (lunch) or when preferences are irreconcilable (Vegan vs Steakhouse).
- **Strategy**: Find two different restaurants within **5 minutes walking distance**.
- **Execution**: "There's a great Steakhouse and a top-rated Vegan café just across the street from each other. Shall we split for lunch and meet at the coffee shop next door in 90 minutes?"

---

## 5. System Logic Implementation

Here is how to code these practical rules into your `resolve_conflict` function:

```python
def solve_cuisine_conflict(preferences):
    # 1. Check for dietary hard constraints (Veto Rule)
    safe_options = filter_by_dietary_restrictions(preferences)
    
    # 2. Check for "Bridging" venues
    multi_cuisine = find_venues(type="food_court", cities=...)
    if multi_cuisine:
        return suggest(multi_cuisine, reason="Everyone can choose their own meal")
        
    # 3. Check for "Safe Bridge" cuisines
    common_ground = find_venues(cuisine=["Mediterranean", "Continental"])
    if common_ground:
        return suggest(common_ground, reason="Offers variety for both tastes")
        
    # 4. If Time > 1 day, Alternate
    if trip_duration > 1:
        return suggest_alternating_schedule()
        
    # 5. Last Resort: Split
    return suggest_nearby_split_options()
```

---

## Summary for Your Dialogues

| Scenario | System Suggestion |
|----------|-------------------|
| **Split Group (3 vs 3)** | "I recommend [Food Hall Name] which has both Italian and Indian counters." |
| **Polarized (Veg vs Meat)** | "how about [Restaurant Name]? It's highly rated for steaks but has a dedicated vegan menu." |
| **Indecisive Group** | "Let's try 'Approval Voting'. I'll list 3 options, tell me which ones you are 'OK' with, not just your favorite." |
