# Prompt Engineering Guide for Subsurface Geoscience & Reservoir/Production Engineers

## 🎯 FUNDAMENTALS

### How LLMs Work
**Key Concept:** LLMs are prediction engines that take sequential text (your prompt) and predict the next token based on training data. A token ≈ ¾ of a word.

**The Prediction Loop:**
1. Model receives your prompt (e.g., "Analyze this well log data...")
2. Predicts the next token
3. Adds that token to the prompt
4. Predicts the next token (considering original prompt + new token)
5. Repeats until it determines the output is complete

---

## ⚙️ MODEL SETTINGS

### 1. Output Length
**What it does:** Sets maximum number of tokens the model can output

**Critical insight:** Reducing output length does NOT make responses more succinct—it just cuts them off when the limit is reached.

**O&G Example:**
- 50 tokens: "The reservoir shows permeability ranges from..." (cuts off mid-analysis)
- 500 tokens: Partial reservoir characterization report
- 5,000 tokens: Complete reservoir characterization with recommendations

**When to adjust:** Set appropriate limits for cost/speed optimization in production workflows, but don't expect stylistic changes.

---

### 2. Temperature (Most Important)
**What it does:** Controls randomness in token selection

**Scale:** 0 (consistent/factual) to 1 (creative/varied)

**High Temperature (0.8-1.0):**
- More creative and unique responses
- Different output each time with same prompt
- **Good for:** Generating alternative development scenarios, brainstorming completion strategies, exploring unconventional solutions

**Low Temperature (0.0-0.2):**
- Consistent, predictable responses
- Similar output each time with same prompt
- **Good for:** Parsing well data, calculating reservoir properties, extracting structured information from reports, regulatory compliance documents

**Reservoir Engineering Example:**
```
Prompt: "Suggest completion strategies for a low-permeability carbonate reservoir"

Temperature 1.0: "Consider an aggressive acid stimulation with diverter stages..."
Temperature 1.0 (again): "Multi-stage hydraulic fracturing with proppant optimization..."
Temperature 0.0: "Standard completion approach: acid wash, perforation density..."
Temperature 0.0 (again): Nearly identical recommendation
```

**Recommended starting points:**
- **Data extraction/calculations:** 0.0-0.2
- **Technical analysis:** 0.4-0.6
- **Strategy development:** 0.7-0.9

---

### 3. Top K & Top P
**Less commonly adjusted** but available for fine-tuning creative vs. deterministic outputs

**Recommended starting points:** Top K = 30, Top P = 0.95

---

## 🔧 PROMPTING TECHNIQUES FOR O&G APPLICATIONS

### 1. Zero-Shot Prompting
**Definition:** Providing only a task description without examples

**When to use:** Simple, straightforward subsurface tasks

**Example - Formation Evaluation:**
```
Classify the following lithology based on the well log description:

"GR: 45 API, Resistivity: 850 ohm-m, Neutron Porosity: 12%, Density: 2.68 g/cc, Sonic: 52 μs/ft"

Lithology:
```

**Output:** Limestone with good reservoir quality ✓

**Example - Reservoir Classification:**
```
Classify this reservoir based on the following properties:

Porosity: 18%, Permeability: 85 mD, Net-to-Gross: 0.72, Water Saturation: 25%, API Gravity: 32°

Reservoir Quality:
```

**Output:** Good quality conventional oil reservoir ✓

---

### 2. Few-Shot Prompting
**Definition:** Providing 2+ examples to establish a clear pattern

**Critical for:** Consistent data parsing, report formatting, calculation standardization

**Recommended:** Use 3-5 examples minimum

**Example - Parsing DST Data to JSON:**
```
Parse drill stem test results into valid JSON format.

Example:
"Zone A tested 1,250 BOPD with 35° API, choke size 32/64", FTHP 2,850 psi, water cut 5%"
{
  "zone": "A",
  "oil_rate_bopd": 1250,
  "api_gravity": 35,
  "choke_size": "32/64",
  "fthp_psi": 2850,
  "water_cut_percent": 5
}

Example:
"Zone C flowed 890 BOPD, 28° API, 24/64" choke, flowing pressure 1,950 psi, 12% water"
{
  "zone": "C",
  "oil_rate_bopd": 890,
  "api_gravity": 28,
  "choke_size": "24/64",
  "fthp_psi": 1950,
  "water_cut_percent": 12
}

Now parse:
"The D sand produced 2,100 barrels per day of 42 degree API crude through a 40/64 inch choke at 3,200 psi flowing tubing head pressure with 3 percent water cut"

JSON response:
```

**Example - Well Log Interpretation Format:**
```
Interpret the following well log data and provide a structured evaluation.

Example 1:
Input: "Depth 8,450-8,470 ft: GR 35 API, RHOB 2.32 g/cc, NPHI 28%, RT 45 ohm-m"
Output:
Zone: Pay Zone 1
Depth: 8,450-8,470 ft
Lithology: Sandstone
Porosity: ~26% (avg NPHI-RHOB)
Interpretation: Good quality reservoir sand, hydrocarbon-bearing

Example 2:
Input: "Depth 9,200-9,215 ft: GR 85 API, RHOB 2.62 g/cc, NPHI 8%, RT 2.5 ohm-m"
Output:
Zone: Non-reservoir
Depth: 9,200-9,215 ft
Lithology: Shale
Porosity: ~8%
Interpretation: Tight shale, non-productive

Now interpret:
"Depth 10,350-10,375 ft: GR 42 API, RHOB 2.28 g/cc, NPHI 32%, RT 125 ohm-m"
```

---

### 3. System Prompting
**Definition:** Sets overall context and purpose for the model

**Location:** Usually in a "System Instructions" field

**O&G Example:**
```
System: You are a senior reservoir engineer with 20 years of experience in carbonate reservoir characterization and waterflood optimization. You provide technically accurate, field-proven recommendations based on industry best practices.
```

---

### 4. Contextual Prompting
**Definition:** Provides specific background information relevant to the analysis

**Structure:**
```
Context: [Field/reservoir background, constraints, historical data]

Task: [Actual technical request]
```

**Example - Production Optimization:**
```
Context: This is a mature waterflood project in the Permian Basin. The XYZ field has been producing for 25 years with current water cut at 78%. We have 45 producing wells and 23 water injection wells. Recent polymer flood pilot showed promising results in the northeast fault block.

Task: Analyze the attached production data and recommend which wells are candidates for workover to improve oil production. Consider economic constraints with oil at $75/bbl.
```

**Example - Drilling Hazard Assessment:**
```
Context: We are planning a horizontal well in the Bakken Formation. Offset wells have experienced lost circulation in the upper Lodgepole section and elevated pore pressures in the Three Forks. The planned TD is 21,500 ft MD with a 10,000 ft lateral.

Task: Review the offset well data and create a drilling hazards summary for the operations team.
```

---

### 5. Role Prompting
**Definition:** Assigns specific petroleum engineering expertise for the model to adopt

**Enhanced Framework:**
- **Role:** Reservoir engineer, geophysicist, production engineer, petrophysicist
- **Goal:** Optimize recovery, reduce costs, ensure safety
- **Backstory:** Years of experience, specific field knowledge

**Example - Petrophysical Analysis:**
```
You are an expert petrophysicist with 15 years of experience in complex carbonate reservoir characterization. You specialize in integrating core data, well logs, and formation testing to build robust rock property models.

I will provide you with well log data and you will interpret lithology, porosity, and fluid saturations. Use industry-standard cutoffs and clearly state your assumptions.

Attached: Well log data from Smith #4 well, 8,200-8,800 ft interval.
```

**Example - Reservoir Simulation:**
```
Act as a reservoir simulation engineer specializing in enhanced oil recovery. You have extensive experience with ECLIPSE and CMG software. I need you to review this history match report and identify which parameters need adjustment to improve the match quality between simulated and actual production data.
```

**Example - Production Engineering:**
```
You are a production engineer responsible for artificial lift optimization in the Eagle Ford play. Analyze the dynamometer cards from 15 rod pump wells and recommend optimal pump speeds and stroke lengths to maximize production while minimizing failures.
```

---

### 6. Step-Back Prompting
**Definition:** Ask the model to consider fundamental principles first, then apply to specific problem

**Purpose:** Activates relevant subsurface knowledge before tackling specific technical challenges

**Example - Stimulation Design:**

**Standard Prompt (May be too generic):**
```
Design a hydraulic fracture treatment for this tight gas sand well.
```

**Step-Back Method:**

**Step 1 - Fundamental Question:**
```
Based on reservoir engineering principles, what are the five most critical reservoir and rock properties that determine the optimal hydraulic fracture design for tight gas sands? Consider both reservoir characterization and geomechanical factors.
```

**Step 2 - Specific Application:**
```
Context: [Paste the five critical factors from Step 1]

Now design a hydraulic fracture treatment for the following well:
- Formation: Mesaverde tight gas sand
- Permeability: 0.05 mD
- Porosity: 8%
- Depth: 9,500 ft
- Young's Modulus: 4.5 Mpsi
- Stress regime: Normal faulting
- In-situ stress contrast: 850 psi
- Reservoir pressure: 4,200 psi
```

**Best for:** Complex engineering decisions, avoiding cookie-cutter solutions, engaging deep technical reasoning

---

### 7. Chain of Thought (CoT)
**Definition:** Prompting the model to show step-by-step technical reasoning

**Critical for:** Calculations, material balance, decline curve analysis, pressure transient analysis

**Simple Implementation:**
```
[Your technical question]

Show all calculations step by step. Include units in each step.
```

**Example - OOIP Calculation:**
```
Calculate the Original Oil in Place (OOIP) for a reservoir with the following properties. Show your work step by step with units.

Given:
- Area: 640 acres
- Net pay thickness: 45 ft
- Porosity: 22%
- Initial water saturation: 28%
- Formation volume factor (Bo): 1.25 RB/STB

Show all steps and formulas used.
```

**Output:**
```
Step 1: Convert area to square feet
640 acres × 43,560 ft²/acre = 27,878,400 ft²

Step 2: Calculate bulk volume
Bulk Volume = Area × Thickness
= 27,878,400 ft² × 45 ft = 1,254,528,000 ft³

Step 3: Calculate pore volume
Pore Volume = Bulk Volume × Porosity
= 1,254,528,000 ft³ × 0.22 = 275,996,160 ft³

Step 4: Calculate hydrocarbon pore volume
HC Pore Volume = Pore Volume × (1 - Sw)
= 275,996,160 ft³ × (1 - 0.28) = 198,717,235 ft³

Step 5: Convert to barrels (1 bbl = 5.615 ft³)
= 198,717,235 ft³ ÷ 5.615 ft³/bbl = 35,391,289 RB

Step 6: Convert to stock tank barrels
OOIP = 35,391,289 RB ÷ 1.25 = 28,313,031 STB

Answer: 28.3 MMSTB
```

**Example - Decline Curve Analysis:**
```
A well is producing under exponential decline. Initial rate was 850 BOPD, current rate after 2 years is 340 BOPD. Calculate:
1. Decline rate (D)
2. Projected rate after 5 years
3. Cumulative production after 5 years

Show all work step by step.
```

**When to use:**
- Reservoir engineering calculations
- Pressure/volume/temperature (PVT) analysis
- Well test interpretation
- Material balance equations
- Production forecasting

---

### 8. Few-Shot + Chain of Thought (Combined)
**Power technique for technical calculations**

**Example - Permeability Calculation:**
```
Calculate permeability using Darcy's Law. Show all work step by step with units.

Example:
Q = 250 bbl/day, μ = 2.5 cp, L = 150 ft, A = 85,000 ft², ΔP = 650 psi

Solution:
Step 1: Convert Q to ft³/sec
Q = 250 bbl/day × (5.615 ft³/bbl) × (1 day/86,400 sec) = 0.0162 ft³/sec

Step 2: Apply Darcy's Law: k = (Q × μ × L) / (A × ΔP)
k = (0.0162 × 2.5 × 150) / (85,000 × 650)
k = 6.075 / 55,250,000 = 1.10 × 10⁻⁷ ft²

Step 3: Convert to millidarcys (1 darcy = 9.87 × 10⁻⁹ cm²)
k = 110 mD

---

Now calculate:
Q = 420 bbl/day, μ = 1.8 cp, L = 200 ft, A = 120,000 ft², ΔP = 850 psi

Show all steps:
```

---

### 9. Self-Consistency
**Definition:** Generate multiple solutions and select most consistent answer via majority voting

**Critical for:** High-stakes reservoir decisions, reserve estimates, completion design

**Process:**
1. Run the same analysis prompt 3-5 times
2. Compare results
3. Use most common answer or average

**Example - EUR (Estimated Ultimate Recovery) Estimation:**

**Run 1:** Type curve match → EUR = 385,000 BOE ✓  
**Run 2:** Decline curve analysis → EUR = 390,000 BOE ✓  
**Run 3:** Analog well comparison → EUR = 325,000 BOE ✗  
**Run 4:** Material balance → EUR = 380,000 BOE ✓  
**Run 5:** Reservoir simulation → EUR = 392,000 BOE ✓

**Result:** 4/5 estimates in 380-392 MBOE range → Recommended EUR: ~387,000 BOE

**When to use:**
- Reserves certification
- Multi-million dollar completion decisions
- Field development plan selection
- Production forecast validation

**Trade-off:** Higher cost and latency for higher confidence

---

### 10. Tree of Thoughts
**Definition:** Explore multiple technical approaches simultaneously, evaluating at each decision point

**Best for:** Complex field development decisions with multiple viable strategies

**Example - Field Development Planning:**
```
Input: Evaluate development strategies for offshore gas field

Branch 1: Subsea tiebacks
  → Option 1A: Single subsea tree
  → Option 1B: Dual completion
  → Option 1C: Subsea manifold

Branch 2: Fixed platform
  → Option 2A: Wellhead platform
  → Option 2B: Production platform + satellite wells

Branch 3: FPSO development
  → Option 3A: Converted tanker
  → Option 3B: Purpose-built FPSO

[Evaluate each path considering: CAPEX, OPEX, production profile, technical risk]
```

**Implementation:** Requires agentic frameworks or code—not practical for manual prompting

---

### 11. ReAct (Reason + Act)
**Definition:** Combine LLM reasoning with external tools in a loop

**O&G Tools Integration:**
- **Technical databases:** IHS Markit, Enverus, Wood Mackenzie
- **Calculation engines:** PVT correlations, decline curves
- **Simulation software:** ECLIPSE, CMG, Petrel APIs
- **Real-time data:** SCADA systems, sensor data
- **Document repositories:** Well files, completion reports, production data

**Example Workflow - Offset Well Analysis:**
```
Task: "Find analogs for the Smith #5 well in the same geological formation within 5 miles"

Thought: I need to search the well database for nearby wells in the same formation
Action: Query well database
Action Input: Formation="Wolfcamp A", Location=within 5 miles of Smith #5
Observation: Found 23 offset wells

Thought: Need to filter for producing wells with similar completion design
Action: Filter results
Action Input: Status="Producing", Completion="Horizontal with multistage frac"
Observation: 8 wells match criteria

Thought: Get production history for these wells
Action: Query production database
Action Input: Well_IDs=[list of 8 wells], Data="Cumulative 12-month production"
Observation: Production ranges from 85,000 to 145,000 BOE

Thought: Calculate average and recommend EUR range
Final Answer: Based on 8 analog wells, expect 12-month cumulative of 110,000 BOE (±25%)
```

**Modern Implementation:** Available in platforms like ChatGPT with plugins, Claude with tools

---

### 12. Automatic Prompt Engineering
**Definition:** Use AI to write better technical prompts

**Workflow for Reservoir Engineers:**

**Step 1 - Brief description:**
```
"I need to analyze pressure buildup test data from 5 wells to determine reservoir permeability and skin factor"
```

**Step 2 - Ask for structured requirements:**
```
"Create a detailed technical specification document (TSD) for this pressure transient analysis that includes:
- Required input data and units
- Analysis methodology
- Expected outputs and formats
- Quality control checks
- Assumptions and limitations"
```

**Step 3 - Use TSD as prompt:**
```
"Perform pressure transient analysis according to this TSD: [paste TSD]

Here is the well test data: [attach data]"
```

**Quick Enhancement:**
```
"Here's my basic prompt: 'Calculate reservoir pressure from shut-in data'

Rewrite this using chain of thought methodology with proper petroleum engineering terminology and unit specifications."
```

---

### 13. Code Execution Prompting
**Definition:** Explicitly tell the model to write and execute code for complex calculations

**When to use:** PVT calculations, nodal analysis, type curves, statistical analysis of production data

**Example - Material Balance Calculation:**

**Direct approach (may have errors):**
```
Calculate cumulative water influx from aquifer using van Everdingen-Hurst method for the following data...
```

**Code execution approach (more reliable):**
```
Write Python code to calculate cumulative water influx using the van Everdingen-Hurst method with the following inputs:
- Time steps: [0, 180, 365, 545, 730] days
- Reservoir pressure: [5,250, 5,100, 4,950, 4,800, 4,650] psi
- Aquifer size: infinite acting
- Water compressibility: 3.2e-6 psi⁻¹
- Aquifer permeability: 250 mD
- Aquifer porosity: 0.18

Execute the code and show results in a formatted table.
```

**Other applications:**
- Black oil PVT correlations
- Gas deviation factor (z-factor) calculations
- IPR curve generation
- Production data normalization
- Decline curve fitting
- Volumetric calculations with Monte Carlo uncertainty

---

## 📋 BEST PRACTICES FOR O&G APPLICATIONS

### 1. Always Specify Units
**Critical in engineering calculations** where unit confusion causes major errors

**Instead of:**
```
Calculate the flow rate given pressure of 2500 and viscosity of 2.5
```

**Say:**
```
Calculate the flow rate in barrels per day (BOPD) given:
- Pressure differential: 2,500 psi
- Oil viscosity: 2.5 cp
- Permeability: 85 mD
- Net pay: 45 ft
```

### 2. Provide Formation/Field Context
Specify:
- Geographic location (basin, field)
- Formation name and age
- Lithology (sandstone, carbonate, shale)
- Drive mechanism
- Development stage (exploration, appraisal, production, mature)

### 3. Include Relevant Constraints
**Economic:**
- Oil/gas prices
- Operating costs
- CAPEX limits

**Technical:**
- Regulatory requirements
- Equipment limitations
- Safety factors

**Operational:**
- Available rig time
- Supply chain constraints
- Environmental restrictions

### 4. Request References to Industry Standards
```
"Provide recommendations following SPE best practices and cite relevant SPE papers where applicable"

"Use API RP standards for tubular design calculations"

"Apply industry-standard porosity-permeability transforms for this formation"
```

### 5. Specify Output Format for Integration
```
Output as CSV for import into Petrel:
X_coord, Y_coord, TVD, Porosity, Sw, Lithology

Output as LAS file format for well log data

Generate JSON compatible with our production database schema
```

### 6. Use Domain-Specific Terminology
Don't say: "rock spaces"  
Say: "porosity" or "pore space"

Don't say: "oil amount"  
Say: "oil saturation", "OOIP", or "reserves"

Don't say: "rock hardness"  
Say: "Young's modulus", "unconfined compressive strength", or "rock mechanical properties"

### 7. Request QC Checks
```
"After calculating EUR, perform a sanity check against typical values for Permian Basin horizontal wells"

"Validate that calculated permeability falls within expected range for Cardium sandstones"

"Check that proposed injection rates don't exceed fracture gradient"
```

### 8. Handling Uncertainty
```
"Provide P10, P50, and P90 estimates for recoverable reserves"

"Include uncertainty analysis using Monte Carlo with 1,000 iterations"

"Show sensitivity of NPV to oil price ($60, $75, $90/bbl) and well costs (±20%)"
```

---

## 🎯 QUICK REFERENCE: TECHNIQUE SELECTION FOR O&G

| Technique | O&G Application | Complexity |
|-----------|-----------------|------------|
| **Zero-Shot** | Lithology classification, basic log interpretation | Low |
| **Few-Shot** | DST parsing, production data formatting, standardized reports | Low-Medium |
| **Chain of Thought** | Reservoir calculations, material balance, well test analysis | Medium |
| **Step-Back** | Completion design, stimulation strategy, EOR screening | Medium |
| **Self-Consistency** | Reserves estimation, field development planning | Medium-High |
| **Tree of Thoughts** | Multi-scenario development planning | High |
| **ReAct** | Offset well analysis with database queries, real-time optimization | High |
| **Role Prompting** | Domain expertise (petrophysics, drilling, production) | Low-Medium |
| **Code Execution** | PVT calculations, decline curves, statistical analysis | Medium |

---

## 🛠️ PRACTICAL EXAMPLES BY DISCIPLINE

### Reservoir Engineering
```
Temperature: 0.2 (calculations), 0.6 (recommendations)
Best techniques: Chain of Thought, Few-Shot, Code Execution
Key outputs: OOIP, EUR, pressure maintenance, simulation inputs
```

### Production Engineering
```
Temperature: 0.3 (optimization), 0.7 (troubleshooting alternatives)
Best techniques: ReAct, Self-Consistency, Role Prompting
Key outputs: Artificial lift design, nodal analysis, production forecasts
```

### Petrophysics
```
Temperature: 0.1 (log analysis), 0.5 (interpretation)
Best techniques: Few-Shot, Chain of Thought, Step-Back
Key outputs: Porosity, saturation, lithology, net pay
```

### Geophysics
```
Temperature: 0.4 (seismic interpretation), 0.7 (play concepts)
Best techniques: Step-Back, Role Prompting, Contextual
Key outputs: Structural maps, prospect identification, AVO analysis
```

### Drilling Engineering
```
Temperature: 0.2 (calculations), 0.5 (hazard assessment)
Best techniques: Chain of Thought, Self-Consistency, Few-Shot
Key outputs: Casing design, hydraulics, wellbore stability
```

---

## 🔑 KEY TAKEAWAYS FOR PETROLEUM ENGINEERS

1. **Always specify units** in all inputs and request units in outputs
2. **Use temperature 0.0-0.2** for calculations and data extraction
3. **Use temperature 0.6-0.8** for strategy development and recommendations
4. **Apply Chain of Thought** for any multi-step calculation
5. **Use Few-Shot** to standardize report formats and data parsing
6. **Leverage Code Execution** for complex PVT and reservoir calculations
7. **Request industry standards** (SPE, API) in responses
8. **Include uncertainty** analysis for reserves and economics
9. **Provide geological context** for better recommendations
10. **Validate outputs** against field experience and offset analogues

---

## 📚 SAMPLE PROMPTS BY WORKFLOW

### Well Planning
```
System: You are a drilling engineer with expertise in horizontal well design in unconventional reservoirs.

Context: Planning a 10,000 ft lateral in the Wolfcamp B formation. Offset wells show average 8.5% drilling time in sliding mode. Target landing zone is 15 ft thick at 10,850 ft TVD.

Task: Create a directional drilling plan with survey station recommendations, build rate suggestions, and motor selection. Include contingency for geo-steering adjustments. Show calculations step by step.
```

### Production Optimization
```
You are a production engineer analyzing rod pump performance. 

Attached: Dynamometer cards from 12 wells showing varying pump efficiencies.

Analyze each card, identify failure modes, and recommend:
1. Optimal stroke length and pump speed
2. Wells requiring immediate workover
3. Expected production increase from optimization

Format output as CSV: Well_ID, Current_Efficiency_%, Recommended_SPM, Recommended_Stroke_in, Expected_Gain_BOPD
```

### Reserves Evaluation
```
Calculate proved developed producing (PDP) reserves using Arps decline curve analysis. Show all work step by step.

Given data:
- Initial rate: 1,250 BOPD
- Current rate: 485 BOPD (after 18 months)
- Decline type: Hyperbolic
- b-factor: 0.8
- Economic limit: 15 BOPD
- Working interest: 75%

Provide:
1. Decline rate calculation
2. Remaining reserves (gross and net)
3. Production forecast table (monthly for 5 years)
4. EUR comparison to type curve
```

---

This guide provides petroleum engineers with structured approaches to maximize AI effectiveness in technical workflows while maintaining engineering rigor and industry standards.