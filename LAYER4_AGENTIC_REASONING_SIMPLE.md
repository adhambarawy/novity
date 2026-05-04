# Layer 4: Agentic Causal Reasoning - Simple Explanation

## The Problem Layer 4 Solves

Imagine you're a doctor and a patient comes in with:
- Fever (101°F)
- Cough
- Sore throat
- Fatigue

**Question:** What's wrong?

**Possible answers:**
- Cold (80% confidence)
- Flu (60% confidence)
- Strep throat (40% confidence)
- COVID (30% confidence)

**Problem:** Multiple diseases could explain these symptoms. Which one is it really?

**Layer 4's job:** Figure out which one is the ROOT CAUSE.

---

## How Layer 4 Works (Simple Version)

### Step 1: Get the List of Suspects

Layer 3 (Diagnostic Models) gives you a ranked list of possible faults:

```
Suction valve leak: 87% confidence
Flow restriction: 45% confidence
Interstage seal leak: 52% confidence
Bearing wear: 12% confidence
```

**Problem:** Are these 4 separate faults or just one fault showing up in different ways?

### Step 2: Ask "Do These Make Sense Together?"

Layer 4 asks: **"If suction valve leaks, what else should happen?"**

**If suction valve leaks:**
- Suction pressure drops ✓ (we see this)
- Discharge temperature rises ✓ (we see this)
- Flow decreases ✓ (we see this)
- Interstage pressure drops ✓ (we see this)

**Conclusion:** All 4 symptoms point to ONE root cause: **Suction valve leak**

The other "faults" (flow restriction, interstage seal leak) are just side effects of the valve leak.

### Step 3: Check Alternative Explanations

**What if it's actually an interstage seal leak instead?**

If interstage seal leaks:
- Interstage pressure drops ✓ (we see this)
- But discharge temperature should stay normal ✗ (we see it's high)
- But suction pressure should stay normal ✗ (we see it's low)

**Conclusion:** Interstage seal leak doesn't explain all the symptoms. Less likely.

### Step 4: Generate Root Cause Hypotheses

Layer 4 produces:

```
PRIMARY DIAGNOSIS (Most Likely):
├─ Suction valve leak
├─ Confidence: 87%
├─ Evidence:
│  ├─ High discharge temp (180°F vs normal 160°F)
│  ├─ Low suction pressure (85 psi vs normal 100 psi)
│  ├─ Low flow (450 CFM vs normal 500 CFM)
│  └─ Interstage pressure drop
└─ RUL: 12-20 days

SECONDARY DIAGNOSIS (Less Likely):
├─ Interstage seal leak
├─ Confidence: 52%
├─ Evidence:
│  └─ Low interstage pressure
└─ But doesn't explain high discharge temp

TERTIARY DIAGNOSIS (Unlikely):
├─ Bearing wear
├─ Confidence: 12%
├─ Evidence:
│  └─ Slight bearing temp rise
└─ But doesn't explain pressure/flow changes
```

---

## Real-World Example

### Scenario: Compressor C-1402 Showing Multiple Alarms

**Raw Data from Sensors:**
- Discharge temperature: 180°F (HIGH)
- Suction pressure: 85 psi (LOW)
- Flow: 450 CFM (LOW)
- Interstage pressure: 240 psi (LOW)
- Bearing temperature: 155°F (NORMAL)

**Layer 3 Output (Diagnostic Models):**
```
Model 1 (Loss of Efficiency): 78% confidence
Model 2 (Suction Valve Leak): 87% confidence
Model 3 (Flow Restriction): 45% confidence
Model 4 (Interstage Seal Leak): 52% confidence
Model 5 (Bearing Wear): 12% confidence
```

**Problem:** 5 different models flagging different faults. Which one is real?

### Layer 4 Analysis

**Step 1: Analyze Causal Chains**

```
Suction valve leaks
    ↓
Suction pressure drops
    ↓
Less air enters compressor
    ↓
Flow decreases
    ↓
Compressor works harder to maintain discharge pressure
    ↓
Discharge temperature rises
    ↓
Interstage pressure drops (because less air is flowing)
```

**All 5 symptoms explained by ONE root cause!**

**Step 2: Check Physical Consistency**

- If suction valve leaks, discharge temp should rise: ✓ YES (180°F is high)
- If suction valve leaks, suction pressure should drop: ✓ YES (85 psi is low)
- If suction valve leaks, flow should decrease: ✓ YES (450 CFM is low)
- If suction valve leaks, bearing temp should stay normal: ✓ YES (155°F is normal)

**All checks pass!**

**Step 3: Evaluate Alternatives**

**Could it be interstage seal leak instead?**
- Interstage seal leak → Low interstage pressure ✓
- But interstage seal leak → Discharge temp should stay normal ✗ (we see 180°F)
- But interstage seal leak → Suction pressure should stay normal ✗ (we see 85 psi)

**Doesn't fit. Less likely.**

**Could it be bearing wear?**
- Bearing wear → Bearing temp rises ✗ (we see 155°F, which is normal)
- Bearing wear → Doesn't explain pressure/flow changes ✗

**Doesn't fit. Very unlikely.**

### Layer 4 Output

```
DIAGNOSIS: Suction valve leak (throw 4)
CONFIDENCE: 87%
RUL: 12-20 days

SUPPORTING EVIDENCE:
├─ Discharge temperature: 180°F (+20°F above normal)
├─ Suction pressure: 85 psi (-15 psi below normal)
├─ Flow: 450 CFM (-50 CFM below normal)
├─ Interstage pressure: 240 psi (low)
└─ Bearing temperature: 155°F (normal - rules out bearing wear)

ALTERNATIVE HYPOTHESES:
├─ Interstage seal leak (52% confidence)
│  └─ Explains: Low interstage pressure
│  └─ Doesn't explain: High discharge temp, low suction pressure
└─ Bearing wear (12% confidence)
   └─ Doesn't explain: Pressure/flow changes, normal bearing temp

RECOMMENDED ACTION:
"Inspect suction valves with ultrasound, vibration, and PV analysis"
(Source: Ariel Compressor Manual, Section 4.2)

TIMING: Schedule within 7 days
```

---

## Why Layer 4 Matters

### Without Layer 4 (Just Layer 3):
Engineer sees 5 different alarms and doesn't know which one to trust:
- "Is it efficiency loss?"
- "Is it valve leak?"
- "Is it flow restriction?"
- "Is it seal leak?"
- "Is it bearing wear?"

**Result:** Confusion, delayed action, or wrong maintenance

### With Layer 4 (Agentic Reasoning):
Engineer sees ONE clear diagnosis with supporting evidence:
- "It's a suction valve leak"
- "Here's why all the symptoms point to this"
- "Here's what to do about it"

**Result:** Clear action, fast response, correct maintenance

---

## The Three Key Questions Layer 4 Asks

### Question 1: "Do These Symptoms Fit Together?"

**Example:**
- High discharge temp + Low suction pressure + Low flow = YES, they fit together
- High bearing temp + Normal pressures + Normal flow = NO, they don't fit together

### Question 2: "What's the Root Cause?"

**Example:**
- Suction valve leak → Causes all 3 symptoms
- Bearing wear → Doesn't cause pressure/flow changes

### Question 3: "Are There Alternative Explanations?"

**Example:**
- Could it be interstage seal leak? (Check: Does it explain all symptoms? NO)
- Could it be bearing wear? (Check: Does it explain all symptoms? NO)
- Could it be suction valve leak? (Check: Does it explain all symptoms? YES)

---

## Simple Algorithm

```
Layer 4 Algorithm (Pseudocode):

1. Get list of fault candidates from Layer 3
   candidates = [suction_valve_leak, flow_restriction, seal_leak, bearing_wear]

2. For each candidate:
   a. Ask: "If this fault exists, what symptoms should we see?"
   b. Check: "Do we actually see those symptoms?"
   c. Score: How many symptoms match?

3. Rank candidates by score:
   suction_valve_leak: 5/5 symptoms match (100%)
   seal_leak: 2/5 symptoms match (40%)
   bearing_wear: 1/5 symptoms match (20%)
   flow_restriction: 3/5 symptoms match (60%)

4. Pick the winner:
   PRIMARY: suction_valve_leak (100% match)
   SECONDARY: flow_restriction (60% match)
   TERTIARY: seal_leak (40% match)

5. Output:
   "Suction valve leak is the root cause"
   "Flow restriction is a side effect"
   "Seal leak is unlikely"
```

---

## Key Insight

**Layer 4 doesn't just list faults. It explains WHY.**

Instead of:
- "Fault A: 87%"
- "Fault B: 45%"
- "Fault C: 52%"

Layer 4 says:
- "Fault A is the root cause (87% confidence)"
- "Faults B and C are side effects of Fault A"
- "Here's the evidence"
- "Here's what to do"

---

## Summary

**Layer 4: Agentic Causal Reasoning**

**Input:** List of possible faults from Layer 3

**Process:**
1. Analyze causal relationships (if X fails, what happens?)
2. Check physical consistency (do all symptoms fit?)
3. Evaluate alternatives (could it be something else?)
4. Generate root cause hypotheses (what's really wrong?)

**Output:** 
- Primary diagnosis with confidence
- Alternative hypotheses ranked by likelihood
- Supporting evidence for each
- Recommended action

**Result:** Engineers get ONE clear answer instead of multiple confusing alarms.

**Analogy:** Layer 4 is like a detective solving a mystery. Instead of listing all the clues, it figures out which clues point to the real culprit.
