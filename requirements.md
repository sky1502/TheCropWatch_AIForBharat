# Requirements Document: Farmer Profit Advisor

## Introduction

The Farmer Profit Advisor is a hackathon MVP prototype designed for Indian farmers to recommend which crops to plant in a given season to maximize expected profit. The system provides uncertainty-aware, explainable, and advisory recommendations without guarantees. The MVP targets small and medium farmers across 1-2 Indian states with support for 5-10 crops, focusing on district-level feasibility and mandi-based price estimation using historical seasonal patterns.

## Glossary

- **System**: The Farmer Profit Advisor application including mobile client, API, and backend services
- **User**: A farmer or field agent using the application
- **Crop_Recommendation**: A suggested crop with profit estimates, confidence metrics, and explanations
- **Season**: One of three Indian agricultural seasons: Kharif (monsoon), Rabi (winter), or Zaid (summer)
- **District**: Administrative subdivision within an Indian state used for geographic scoping
- **Mandi**: Government-regulated agricultural market where farmers sell produce
- **Profit_Range**: Distributional profit estimate expressed as P10, P50, P90 quantiles per acre
- **Confidence_Label**: Categorical indicator (low/medium/high) of recommendation reliability
- **Feasibility_Constraint**: Hard requirement based on season, geography, and crop calendar compatibility
- **Agent**: AI orchestrator that manages user interaction and recommendation generation
- **Sowing_Window**: Date range when a crop can be planted
- **Risk_Preference**: User's tolerance for profit volatility (low/medium/high)
- **Modal_Price**: Most frequently occurring price in mandi data
- **Quantile**: Statistical measure representing a percentile of a distribution (P10 = 10th percentile)

## Requirements

### Requirement 1: User Input Collection

**User Story:** As a farmer, I want to provide my location and farming context, so that I receive relevant crop recommendations for my situation.

#### Acceptance Criteria

1. **FR-INP-01**: WHEN a user starts a recommendation session, THE System SHALL request state, district, and season
2. **FR-INP-02**: WHEN a user provides state and district, THE System SHALL validate them against supported regions
3. **FR-INP-03**: WHEN a user provides a season, THE System SHALL validate it as one of Kharif, Rabi, or Zaid
4. **FR-INP-04**: WHEN required inputs are missing, THE System SHALL prompt for state, district, and season before generating recommendations
5. **FR-INP-05**: WHEN a user provides optional inputs (land size, irrigation, storage, risk preference, selling mode), THE System SHALL incorporate them into recommendation logic
6. **FR-INP-06**: WHEN a user provides a sowing window, THE System SHALL validate it against the selected season's typical calendar
7. **FR-INP-07**: WHEN the Agent collects inputs in the default recommendation flow, THE System SHALL ask no more than 5 clarifying questions
8. **FR-INP-08**: WHEN a user provides land size, THE System SHALL accept values in acres or hectares with explicit units
9. **FR-INP-09**: WHEN a user provides irrigation availability, THE System SHALL accept one of: none, partial, or assured
10. **FR-INP-10**: WHEN a user provides risk preference, THE System SHALL accept one of: low, medium, or high

### Requirement 2: Feasibility Filtering

**User Story:** As a farmer, I want to see only crops that are actually feasible for my district and season, so that I don't waste time on impossible options.

#### Acceptance Criteria

1. **FR-FEA-01**: WHEN generating recommendations, THE System SHALL filter crops to only those compatible with the user's district and season
2. **FR-FEA-02**: WHEN a crop's season window does not overlap with the user's sowing window, THE System SHALL exclude it from recommendations
3. **FR-FEA-03**: WHEN feasibility data is unavailable for a district-crop-season combination, THE System SHALL exclude that crop from recommendations
4. **FR-FEA-04**: WHEN ranking crops, THE System SHALL enforce feasibility constraints before any ranking or profit calculation
5. **FR-FEA-05**: WHEN no crops pass feasibility filtering, THE System SHALL inform the user and suggest adjusting inputs

### Requirement 3: Profit Estimation

**User Story:** As a farmer, I want to understand the expected profit range for each recommended crop, so that I can make informed financial decisions.

#### Acceptance Criteria

1. **FR-PRF-01**: WHEN calculating profit for a crop, THE System SHALL compute profit as expected revenue minus expected costs
2. **FR-PRF-02**: WHEN estimating revenue, THE System SHALL use mandi price forecasts and expected yield estimates
3. **FR-PRF-03**: WHEN estimating costs, THE System SHALL include seed, fertilizer, labor, irrigation, and other input costs
4. **FR-PRF-04**: WHEN presenting profit estimates, THE System SHALL express them as quantile ranges (P10, P50, P90) per acre
5. **FR-PRF-05**: WHEN land size is provided, THE System SHALL also compute total profit ranges across the user's land
6. **FR-PRF-06**: WHEN price data is sparse, THE System SHALL use district-level or state-level aggregates as fallbacks
7. **FR-PRF-07**: WHEN yield data is sparse, THE System SHALL use district-level or state-level historical averages as fallbacks
8. **FR-PRF-08**: WHEN computing yield estimates, THE System SHALL incorporate irrigation availability into calculations
9. **FR-PRF-09**: WHEN computing cost estimates, THE System SHALL incorporate irrigation availability into calculations
10. **FR-PRF-10**: WHEN ranking crops, THE System SHALL incorporate risk preference into the ranking utility function

### Requirement 4: Recommendation Generation

**User Story:** As a farmer, I want to receive a ranked list of crop options, so that I can quickly identify the best opportunities.

#### Acceptance Criteria

1. **FR-REC-01**: WHEN generating recommendations, THE System SHALL return exactly 3 top-ranked crops plus 1 conservative fallback option
2. **FR-REC-02**: WHEN ranking crops, THE System SHALL use a utility function that balances expected profit, risk, and user preferences
3. **FR-REC-03**: WHEN risk preference is low, THE System SHALL weight P10 profit more heavily in ranking
4. **FR-REC-04**: WHEN risk preference is high, THE System SHALL weight P90 profit more heavily in ranking
5. **FR-REC-05**: WHEN risk preference is medium, THE System SHALL weight P50 profit most heavily in ranking
6. **FR-REC-06**: WHEN selecting the conservative fallback, THE System SHALL include at least one low-risk crop
7. **FR-REC-07**: WHEN multiple crops have similar utility scores, THE System SHALL use price stability as a tiebreaker

### Requirement 5: Confidence Assessment

**User Story:** As a farmer, I want to know how reliable each recommendation is, so that I can gauge the trustworthiness of the advice.

#### Acceptance Criteria

1. **FR-CNF-01**: WHEN generating a recommendation, THE System SHALL assign a confidence label of low, medium, or high to each crop
2. **FR-CNF-02**: WHEN mandi price data covers fewer than 12 months, THE System SHALL assign low confidence
3. **FR-CNF-03**: WHEN mandi price data covers 12-36 months with consistent reporting, THE System SHALL assign medium confidence
4. **FR-CNF-04**: WHEN mandi price data covers more than 36 months with consistent reporting and recent data, THE System SHALL assign high confidence
5. **FR-CNF-05**: WHEN yield data is based on state-level aggregates rather than district-level data, THE System SHALL reduce confidence by one level
6. **FR-CNF-06**: WHEN weather forecast data is unavailable, THE System SHALL reduce confidence by one level
7. **FR-CNF-07**: WHEN assigning a confidence label, THE System SHALL provide a short textual reason for the assigned confidence level

### Requirement 6: Explanation Generation

**User Story:** As a farmer, I want to understand why each crop is recommended, so that I can trust the advice and learn about market conditions.

#### Acceptance Criteria

1. **FR-EXP-01**: WHEN presenting a crop recommendation, THE System SHALL provide 3 to 5 bullet-point explanations
2. **FR-EXP-02**: WHEN generating explanations, THE System SHALL include an explanation of feasibility (why this crop works for the district and season)
3. **FR-EXP-03**: WHEN generating explanations, THE System SHALL include an explanation of price outlook (recent trends, expected demand)
4. **FR-EXP-04**: WHEN generating explanations, THE System SHALL include an explanation of cost considerations (major input costs, irrigation needs)
5. **FR-EXP-05**: WHEN generating explanations, THE System SHALL include an explanation of key risks (price volatility, weather sensitivity)
6. **FR-EXP-06**: WHEN generating explanations, THE System SHALL include an explanation of ranking factors (what would change this crop's position)
7. **FR-EXP-07**: WHEN generating explanations, THE System SHALL use simple language appropriate for users with limited technical literacy
8. **FR-EXP-08**: WHEN generating explanations, THE System SHALL ground statements in actual data sources (mandi prices, weather patterns, crop calendars)

#### Grounding and Explanation Enforcement

**LLM Allowed Actions:**
- THE System SHALL allow the LLM to summarize structured model outputs (profit ranges, confidence scores, ranking positions)
- THE System SHALL allow the LLM to reference computed features and metadata fields (price trends, volatility metrics, yield estimates)
- THE System SHALL allow the LLM to format and present data in natural language

**LLM Prohibited Actions:**
- THE System SHALL NOT allow the LLM to introduce agronomic advice not derived from datasets (e.g., specific fertilizer recommendations, pest control methods)
- THE System SHALL NOT allow the LLM to suggest soil treatments, pest management practices, or farming techniques beyond what's computed
- THE System SHALL NOT allow the LLM to make causal or prescriptive claims not computed by the model (e.g., "this will definitely increase yield")
- THE System SHALL NOT allow the LLM to invent data points or statistics not present in the model output

**Enforcement:**
- WHEN the LLM generates an explanation containing unsupported claims, THE System SHALL reject and regenerate the explanation
- WHEN presenting explanations, THE System SHALL ensure all factual statements are traceable to specific data fields or computed metrics

### Requirement 7: Uncertainty Communication

**User Story:** As a farmer, I want to understand the risks and uncertainties in each recommendation, so that I can make decisions appropriate to my risk tolerance.

#### Acceptance Criteria

1. **FR-UNC-01**: WHEN presenting profit estimates, THE System SHALL display the full quantile range (P10, P50, P90) not just a single number
2. **FR-UNC-02**: WHEN price volatility for a crop exceeds 30% coefficient of variation, THE System SHALL explicitly flag it as high price risk
3. **FR-UNC-03**: WHEN weather sensitivity is high for a crop, THE System SHALL explicitly mention weather risk in the explanation
4. **FR-UNC-04**: WHEN presenting recommendations, THE System SHALL display a disclaimer stating that recommendations are advisory only with no profit guarantees
5. **FR-UNC-05**: WHEN confidence is low, THE System SHALL prominently display the low confidence label and recommend caution

### Requirement 8: What-If Analysis

**User Story:** As a farmer, I want to explore how changing my inputs affects recommendations, so that I can understand my options better.

#### Acceptance Criteria

1. **FR-WIF-01**: WHEN a user requests a what-if scenario, THE System SHALL accept changes to irrigation availability (none/partial/assured), sowing date (+2 weeks delay), or storage access (yes/no)
2. **FR-WIF-02**: WHEN a user changes irrigation from none to partial or assured, THE System SHALL recompute recommendations with updated yield and cost assumptions
3. **FR-WIF-03**: WHEN a user delays sowing by +2 weeks, THE System SHALL recompute feasibility and recommendations for the shifted window
4. **FR-WIF-04**: WHEN a user toggles storage access, THE System SHALL recompute recommendations with updated selling assumptions
5. **FR-WIF-05**: WHEN returning what-if results, THE System SHALL return them within 5 seconds
6. **FR-WIF-06**: WHEN presenting what-if results, THE System SHALL highlight which recommendations changed and why
7. **FR-WIF-07**: WHEN generating a what-if scenario, THE System SHALL create a deterministic scenario ID (hash of inputs) and cache results
8. **FR-WIF-08**: WHEN identical what-if inputs are provided, THE System SHALL produce identical outputs (deterministic behavior)

### Requirement 9: Data Ingestion and Normalization

**User Story:** As a system operator, I want the system to ingest and normalize public Indian agricultural data, so that recommendations are grounded in real data.

#### Acceptance Criteria

1. **FR-DAT-01**: WHEN ingesting data, THE System SHALL ingest government-published crop calendars for supported states and districts
2. **FR-DAT-02**: WHEN ingesting data, THE System SHALL ingest mandi price data including modal, minimum, and maximum prices where available
3. **FR-DAT-03**: WHEN ingesting data, THE System SHALL ingest historical rainfall and temperature data from official sources
4. **FR-DAT-04**: WHEN weather forecast data is available, THE System SHALL ingest near-term weather forecast data
5. **FR-DAT-05**: WHEN normalizing data, THE System SHALL normalize commodity names across data sources to a canonical crop list
6. **FR-DAT-06**: WHEN normalizing data, THE System SHALL normalize geographic identifiers (state names, district names) to a canonical format
7. **FR-DAT-07**: WHEN normalizing data, THE System SHALL normalize units (prices per quintal, land in acres/hectares, rainfall in mm) to standard units
8. **FR-DAT-08**: WHEN validating data, THE System SHALL validate data quality and flag records with missing or anomalous values
9. **FR-DAT-09**: WHEN updating data, THE System SHALL refresh mandi price data at least weekly
10. **FR-DAT-10**: WHEN updating data, THE System SHALL refresh weather data at least daily

### Requirement 10: Safety and Guardrails

**User Story:** As a system operator, I want the system to fail safely and avoid harmful recommendations, so that farmers are not misled.

#### Acceptance Criteria

1. **FR-SAF-01**: WHEN presenting recommendations, THE System SHALL never claim guaranteed profit or assured returns
2. **FR-SAF-02**: WHEN data quality is insufficient for a reliable recommendation, THE System SHALL return conservative fallback options with explicit low confidence warnings
3. **FR-SAF-03**: WHEN no feasible crops are found, THE System SHALL explain why and suggest input adjustments rather than forcing a recommendation
4. **FR-SAF-04**: WHEN receiving user inputs, THE System SHALL validate all user inputs and reject out-of-range or nonsensical values
5. **FR-SAF-05**: WHEN profit estimates are negative, THE System SHALL clearly warn the user and explain cost drivers
6. **FR-SAF-06**: WHEN filtering crops, THE System SHALL not recommend crops outside the MVP-supported crop list even if data exists
7. **FR-SAF-07**: WHEN filtering crops, THE System SHALL not recommend crops outside the MVP-supported states even if data exists
8. **FR-SAF-08**: WHEN the Agent generates explanations, THE System SHALL validate that claims are grounded in actual data
9. **FR-SAF-09**: WHEN generating recommendations, THE System SHALL log all recommendations and inputs for audit and quality monitoring

### Requirement 11: Performance and Reliability

**User Story:** As a farmer with limited connectivity, I want the app to respond quickly and work offline when possible, so that I can use it in rural areas.

#### Acceptance Criteria

1. **NFR-LAT-01**: WHEN a user requests initial recommendations with all required inputs, THE System SHALL return results within 8 seconds
2. **NFR-LAT-02**: WHEN a user requests a what-if scenario, THE System SHALL return results within 5 seconds
3. **NFR-OFF-01**: WHEN caching results, THE System SHALL cache the most recent recommendation results on the mobile client
4. **NFR-OFF-02**: WHEN the mobile client is offline, THE System SHALL allow viewing of cached recommendation results
5. **NFR-OFF-03**: WHEN the mobile client is offline, THE System SHALL display a clear indicator that data may be stale
6. **NFR-REL-01**: WHEN API failures occur, THE System SHALL handle them gracefully and display user-friendly error messages
7. **NFR-REL-02**: WHEN measuring uptime during peak usage hours (6 AM to 10 PM IST), THE System SHALL have 99% uptime
8. **NFR-REL-03**: WHEN the backend is unavailable, THE System SHALL serve cached district-level recommendations if available

### Requirement 12: Interpretability and Transparency

**User Story:** As a field agent helping farmers, I want to understand how the system arrived at its recommendations, so that I can explain and defend the advice.

#### Acceptance Criteria

1. **FR-INT-01**: WHEN presenting a recommendation, THE System SHALL cite specific data sources (mandi name, date range, weather station)
2. **FR-INT-02**: WHEN presenting price forecasts, THE System SHALL show recent historical prices as context
3. **FR-INT-03**: WHEN presenting yield estimates, THE System SHALL show the basis (district average, state average, or specific source)
4. **FR-INT-04**: WHEN presenting recommendations, THE System SHALL provide a summary of the ranking logic (which factors weighted most heavily)
5. **FR-INT-05**: WHEN a crop is excluded due to feasibility, THE System SHALL explain which constraint was violated

### Requirement 13: MVP Scope Constraints

**User Story:** As a product manager, I want the MVP to have clear scope boundaries, so that we can deliver quickly and iterate.

#### Acceptance Criteria

1. **FR-SCP-01**: WHEN defining supported regions, THE System SHALL support exactly 1 to 2 Indian states in the hackathon MVP
2. **FR-SCP-02**: WHEN defining supported crops, THE System SHALL support exactly 5 to 10 crops in the hackathon MVP
3. **FR-SCP-03**: WHEN processing geographic data, THE System SHALL use district-level geographic resolution (not block or village level)
4. **FR-SCP-04**: WHEN aggregating price data, THE System SHALL use mandi-level price data aggregated to district level
5. **FR-SCP-05**: WHEN validating seasons, THE System SHALL support only the three major seasons (Kharif, Rabi, Zaid) not minor seasons
6. **FR-SCP-06**: WHEN filtering products, THE System SHALL not include livestock, horticulture, or non-crop agricultural products in the MVP
7. **FR-SCP-07**: WHEN generating recommendations, THE System SHALL not include market linkage or buyer matching features in the MVP
8. **FR-SCP-08**: WHEN generating recommendations, THE System SHALL not include financial product recommendations (loans, insurance) in the MVP
9. **FR-SCP-09**: WHEN generating recommendations, THE System SHALL not include soil health or pest management advice in the MVP
