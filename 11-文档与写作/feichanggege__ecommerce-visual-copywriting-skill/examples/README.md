# Examples

This folder contains example inputs and expected output patterns for `ecommerce-visual-copywriting-skill`. The examples are intentionally written as reusable reference cases, not final legal advice.

Each example follows the project workflow:

1. Collect product information.
2. Extract compliant selling reasons.
3. Confirm the selling reasons with the user.
4. Draft main image copy and product detail page modules.
5. Self-review the output with the scoring checklist.
6. Deliver only after the quality threshold is reached.

## Example 1: Ordinary Food

### Input

```text
Product type: Ordinary food, not a health food, not a drug
Brand: Xionghua Mumo
Product: Pine pollen solid drink
Ingredients: pine pollen, maltodextrin, polygonatum, hawthorn, mint
SKU: 60g/box
Target user: Office workers who want a convenient daily drink
Main image draft: Highlight quick dissolving, independent small sticks, fresh taste, easy to carry
Evidence: No health-function approval document provided
Platform: Taobao / Douyin Shop
```

### Safe Selling Reasons

```text
1. Independent small-stick packaging: convenient for office, commute, and travel.
2. Quick dissolving drinking experience: suitable for daily beverage scenarios.
3. Pine pollen and botanical ingredients: can describe ingredients and flavor, but cannot imply health effects.
4. Ordinary food boundary: must clearly state that it is not a health food or drug.
```

### Expected Output Pattern

```text
[Image 1 / Main Image]

Scene:
Product box and small sticks centered, with a clean green-and-gold background. Add a cup showing the drink after mixing.

In-image copy:
Title: Pine Pollen Solid Drink
Subtitle: Small sticks, easy to carry
Selling points: Quick dissolve · Fresh taste · Daily drink
Disclaimer: Ordinary food, not health food

Design guidance:
Use a clean ingredient-based style. Keep the disclaimer readable at the bottom. Do not use health-effect words such as immunity, detox, sleep improvement, or body conditioning.
```

## Example 2: Fitness Equipment

### Input

```text
Product type: Fitness equipment, not a medical device
Product: Posture training resistance band
Target user: People who sit for long periods and want home exercise support
Main image draft: Improve posture, fix hunchback, relieve back pain
Evidence: No medical device certificate
Platform: JD / Douyin Shop
```

### Risky Draft

```text
Fix hunchback, relieve back pain, correct spine problems.
```

### Safer Direction

```text
Home posture training, shoulder-back stretching, lightweight resistance support.
```

### Expected Output Pattern

```text
[Image 2 / Use Scenario]

Scene:
Split view: office break stretching, home training, and warm-up before exercise.

In-image copy:
Title: Daily posture training
Subtitle: Shoulder-back stretching support
Selling points: Lightweight · Portable · Easy to start
Disclaimer: Fitness equipment, not a medical device

Design guidance:
Avoid medicalized words such as treatment, correction, spine disease, back pain relief, or rehabilitation. Use training and exercise language instead.
```

## Example 3: Compliance Review

### Input

```text
Product type: Ordinary food
Draft copy:
100% effective for improving sleep. The best natural formula. Drink for 7 days to restore energy and improve immunity.
Evidence: No test report, no health food approval, no clinical evidence
```

### Review Result

```text
High risk. The draft contains multiple issues:

1. "100% effective" is an absolute and unsupported claim.
2. "The best" is an absolute marketing claim.
3. "Improve sleep" and "improve immunity" are health-function claims.
4. "Drink for 7 days" creates an unsupported effect promise.
5. Ordinary food cannot imply treatment, health improvement, or body-function changes.
```

### Safer Rewrite Direction

```text
Product name: Botanical solid drink
Selling points: Ingredient combination, light flavor, portable small sticks, daily drinking scenario
Required disclaimer: Ordinary food, not health food, not medicine
```

### Expected Output Pattern

```text
[Compliance Rewrite]

Scene:
Ingredient flat lay with product packaging and a prepared drink.

In-image copy:
Title: Botanical Solid Drink
Subtitle: Light taste, easy to carry
Selling points: Small sticks · Daily drink · Fresh flavor
Disclaimer: Ordinary food, not health food or medicine

Design guidance:
Remove all health-effect claims. Keep the visual focus on ingredients, taste, packaging, and daily drinking scenarios.
```

## Self-Review Checklist for Examples

Before adding a new example, check:

- Product category is clearly stated.
- Evidence and qualification status are clearly stated.
- Risky words are identified when relevant.
- Output uses the project structure: scene, in-image copy, design guidance.
- Main image copy is short enough to fit on a real mobile e-commerce image.
- Required disclaimer is present for sensitive categories.
- The example does not present legal advice as a final compliance guarantee.

## How to Add More Examples

Open a pull request with:

- A short product brief.
- The risky or raw draft, if available.
- The safer selling reasons.
- One expected output block.
- Notes about which rule section the example tests.

Good examples are small, realistic, and easy to manually review.
