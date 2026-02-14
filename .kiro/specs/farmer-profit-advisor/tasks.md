# Implementation Plan: Farmer Profit Advisor

## Overview

This implementation plan breaks down the Farmer Profit Advisor hackathon MVP into discrete coding tasks. The system will be implemented in Python, focusing on core recommendation logic, data handling, and validation of critical properties. The MVP targets 1-2 Indian states (Punjab, Maharashtra) with 5-10 crops, using historical seasonal price proxies rather than true forecasting.

## Tasks

- [ ] 1. Set up project structure and core data models
  - Create Python project with FastAPI backend structure
  - Define data models for User Context, Crop, District, Season, Recommendation
  - Define data models for Profit Range (P10, P50, P90), Confidence Label
  - Set up configuration for supported states, districts, and crops (1-2 states, 5-10 crops)
  - _Requirements: FR-SCP-01, FR-SCP-02, FR-SCP-03_

- [ ] 2. Implement data ingestion and normalization
  - [ ] 2.1 Create data loaders for static datasets
    - Load crop calendar data (CSV/JSON format)
    - Load mandi price data (CSV format with commodity, mandi, date, modal_price)
    - Load yield data (district-level and state-level averages)
    - Load cost data (seed, fertilizer, labor, irrigation costs by crop)
    - _Requirements: FR-DAT-01, FR-DAT-02, FR-DAT-03_
  
  - [ ] 2.2 Implement commodity and geography normalization
    - Create canonical crop name mapping (handle regional variations)
    - Create canonical district name mapping
    - Implement unit normalization (prices to ₹/quintal, land to acres, rainfall to mm)
    - _Requirements: FR-DAT-05, FR-DAT-06, FR-DAT-07_
  
  - [ ] 2.3 Implement data validation
    - Flag missing values in datasets
    - Flag outliers (>3 standard deviations)
    - Compute data quality scores for confidence assessment
    - _Requirements: FR-DAT-08_

- [ ] 3. Implement feasibility filtering
  - [ ] 3.1 Create feasibility checker
    - Implement crop-district-season compatibility check using crop calendar
    - Implement sowing window overlap validation
    - Filter crops with missing feasibility data
    - _Requirements: FR-FEA-01, FR-FEA-02, FR-FEA-03_
  
  - [ ]* 3.2 Write validation test for feasibility invariant
    - **Property 1: Feasibility Invariant**
    - Test with sample districts (Punjab-Ludhiana, Maharashtra-Pune)
    - Test valid combinations (Rice-Punjab-Kharif, Wheat-Punjab-Rabi)
    - Test invalid combinations (Wheat-Punjab-Kharif, Cotton-Punjab-Zaid)
    - **Validates: Requirements FR-FEA-01, FR-FEA-02, FR-FEA-03**
  
  - [ ] 3.3 Implement zero feasibility error handling
    - Return explanation when no crops pass feasibility
    - Suggest input adjustments (try different season, check district)
    - _Requirements: FR-FEA-05, FR-SAF-03_

- [ ] 4. Implement price estimation using historical seasonal proxies
  - [ ] 4.1 Create price estimator
    - Aggregate mandi prices to district-level monthly averages
    - Compute rolling 3-year seasonal quantiles (P10, P50, P90) for harvest month
    - Optional: Apply simple linear trend adjustment if clear trend exists
    - Implement fallback to state-level when district data sparse
    - _Requirements: FR-PRF-02, FR-PRF-06_
  
  - [ ] 4.2 Compute price volatility metrics
    - Calculate coefficient of variation (CV) = std_dev / mean
    - Flag high price risk when CV > 30%
    - _Requirements: FR-UNC-02_

- [ ] 5. Implement yield and cost estimation
  - [ ] 5.1 Create yield estimator
    - Use district historical average as base yield
    - Apply irrigation adjustment factors (none: 0.7×, partial: 0.85×, assured: 1.0×)
    - Apply weather adjustment (rainfall deviation from normal)
    - Apply sowing timing penalty (±10% if outside optimal window)
    - Model uncertainty: P10 = 0.7 × base, P50 = 1.0 × base, P90 = 1.3 × base
    - Implement fallback to state-level when district data unavailable
    - _Requirements: FR-PRF-02, FR-PRF-07, FR-PRF-08_
  
  - [ ] 5.2 Create cost estimator
    - Calculate seed costs (crop-specific rate × land size)
    - Calculate fertilizer costs (standard NPK requirements × prices)
    - Calculate labor costs (person-days × wage rate)
    - Calculate irrigation costs if irrigation used
    - Calculate other costs (pesticides, land prep, transport)
    - Apply ±10% uncertainty range
    - _Requirements: FR-PRF-03, FR-PRF-09_

- [ ] 6. Implement profit calculation and ranking
  - [ ] 6.1 Create profit calculator
    - Compute profit = revenue - costs where revenue = yield × price
    - Generate profit quantiles (P10, P50, P90) using Monte Carlo or analytical methods
    - Ensure P10 ≤ P50 ≤ P90 ordering
    - Scale to total profit when land size provided (total = per-acre × land_size)
    - _Requirements: FR-PRF-01, FR-PRF-04, FR-PRF-05_
  
  - [ ]* 6.2 Write validation tests for profit calculation
    - **Property 3: Profit Calculation Correctness**
    - Test profit = (yield × price) - (seed + fertilizer + labor + irrigation + other)
    - Test P10 ≤ P50 ≤ P90 for all crops
    - Test land size scaling: total_profit = per_acre_profit × land_size
    - **Validates: Requirements FR-PRF-01, FR-PRF-02, FR-PRF-03, FR-PRF-04, FR-PRF-05**
  
  - [ ] 6.3 Implement ranking utility function
    - Define utility weights based on risk preference:
      - Low risk: w_low=0.6, w_med=0.3, w_high=0.1
      - Medium risk: w_low=0.2, w_med=0.6, w_high=0.2
      - High risk: w_low=0.1, w_med=0.3, w_high=0.6
    - Compute utility = w_low × P10 + w_med × P50 + w_high × P90
    - Apply risk penalties (high volatility: -10%, negative P10: -20%)
    - Use price stability (lower CV) as tiebreaker
    - _Requirements: FR-REC-02, FR-REC-03, FR-REC-04, FR-REC-05, FR-REC-07, FR-PRF-10_
  
  - [ ]* 6.4 Write validation tests for risk-based ranking
    - **Property 5: Risk-Based Ranking** (Appendix A1)
    - Test low risk preference: higher P10 ranks higher
    - Test high risk preference: higher P90 ranks higher
    - Test medium risk preference: higher P50 ranks higher
    - Use sample crops with different profit distributions
    - **Validates: Requirements FR-REC-03, FR-REC-04, FR-REC-05, FR-PRF-10**
  
  - [ ] 6.4 Select conservative fallback crop
    - Select crop with highest P10 (best worst-case)
    - Ensure CV < 20% (low volatility)
    - _Requirements: FR-REC-06_

- [ ] 7. Implement confidence assessment
  - [ ] 7.1 Create confidence scorer
    - Start with base score 100
    - Apply penalties:
      - <12 months price data: -40 points
      - 12-36 months price data: -20 points
      - Missing recent data (>30 days): -15 points
      - State-level yield data: -15 points
      - No weather forecast: -10 points
      - Wide profit range (P90/P10 > 3): -10 points
      - Negative P10: -15 points
    - Assign label: ≥70 = high, 40-69 = medium, <40 = low
    - Generate textual explanation for confidence level
    - _Requirements: FR-CNF-01, FR-CNF-02, FR-CNF-03, FR-CNF-04, FR-CNF-05, FR-CNF-06, FR-CNF-07_
  
  - [ ]* 7.2 Write validation tests for confidence rules
    - **Property 7: Confidence Rules**
    - Test <12 months data → low confidence
    - Test 12-36 months data → medium confidence
    - Test >36 months data → high confidence
    - Test state-level yield → confidence reduced by one level
    - **Validates: Requirements FR-CNF-02, FR-CNF-03, FR-CNF-04, FR-CNF-05, FR-CNF-06**

- [ ] 8. Implement explanation generation
  - [ ] 8.1 Create explanation templates
    - Feasibility template: "{Crop} is suitable for {district} during {season}, sowing in {month_range}"
    - Price outlook template: "Prices {trend} over past {months} months. Current: ₹{price}/quintal. Expected: ₹{forecast}/quintal"
    - Cost template: "Estimated costs: ₹{total}/acre (seed: ₹{seed}, fertilizer: ₹{fert}, labor: ₹{labor}, irrigation: ₹{irrig})"
    - Risk template: "Main risks: {risk_list}" (price volatility, weather sensitivity)
    - Ranking template: "Ranked #{rank} due to {reason}. Would rank higher if {improvement}"
    - _Requirements: FR-EXP-02, FR-EXP-03, FR-EXP-04, FR-EXP-05, FR-EXP-06_
  
  - [ ] 8.2 Implement grounding validation
    - Verify all numbers come from actual data (not hallucinated)
    - Verify crop-district-season in crop calendar
    - Verify price trends match historical data direction
    - Verify cost estimates within ±50% of benchmarks
    - Reject explanations with unsupported agronomic advice
    - _Requirements: FR-EXP-08, FR-SAF-08_
  
  - [ ]* 8.3 Write validation tests for explanation grounding
    - **Property 6: Explanation Grounding**
    - Test all explanations reference specific mandis and dates
    - Test no hallucinated numbers (all traceable to data)
    - Test no unsupported agronomic advice (soil treatments, pest control)
    - **Validates: Requirements FR-EXP-08, FR-SAF-08**

- [ ] 9. Implement recommendation generation
  - [ ] 9.1 Create main recommendation engine
    - Accept user context (state, district, season, sowing_window, land_size, irrigation, risk_preference)
    - Apply feasibility filtering
    - Compute profit estimates for all feasible crops
    - Rank crops by utility function
    - Select top 3 crops + 1 conservative fallback
    - Generate confidence labels and explanations for each crop
    - Return structured recommendation response
    - _Requirements: FR-REC-01_
  
  - [ ]* 9.2 Write validation tests for recommendation structure
    - **Property 2: Recommendation Structure**
    - Test exactly 4 crops returned when feasible crops exist
    - Test each crop has profit range (P10, P50, P90)
    - Test each crop has confidence label {low, medium, high}
    - Test each crop has 3-5 bullet explanations
    - **Validates: Requirements FR-REC-01, FR-PRF-04, FR-UNC-01, FR-CNF-01, FR-EXP-01**
  
  - [ ] 9.3 Add advisory disclaimer and safety checks
    - Add disclaimer: "Recommendations are advisory only. No profit guarantees."
    - Validate no "guaranteed", "assured", "certain" in profit claims
    - Enforce MVP crop list (5-10 crops)
    - Enforce MVP state list (1-2 states)
    - _Requirements: FR-UNC-04, FR-SAF-01, FR-SAF-06, FR-SAF-07_
  
  - [ ]* 9.4 Write validation tests for safety
    - **Property 9: Advisory Disclaimer and Safety**
    - Test disclaimer present in all outputs
    - Test no "guaranteed" or "assured" in profit claims
    - Test all crops from MVP-supported list
    - Test all districts from MVP-supported states
    - **Validates: Requirements FR-UNC-04, FR-SAF-01, FR-SAF-06, FR-SAF-07**

- [ ] 10. Checkpoint - Core recommendation engine complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 11. Implement input validation
  - [ ] 11.1 Create input validators
    - Validate state-district pairs against supported regions
    - Validate season in {Kharif, Rabi, Zaid}
    - Validate irrigation in {none, partial, assured}
    - Validate land size has units and is in range [0.1, 1000]
    - Validate sowing window within season boundaries
    - _Requirements: FR-INP-02, FR-INP-03, FR-INP-06, FR-INP-08, FR-INP-09, FR-SAF-04_
  
  - [ ]* 11.2 Write validation tests for input validation
    - **Property 4: Input Validation**
    - Test valid state-district pairs accepted (Punjab-Ludhiana, Maharashtra-Pune)
    - Test invalid state-district pairs rejected (Punjab-Mumbai)
    - Test valid seasons accepted (Kharif, Rabi, Zaid)
    - Test invalid seasons rejected ("Summer", "Monsoon")
    - Test valid irrigation accepted (none, partial, assured)
    - Test invalid irrigation rejected ("medium", "full")
    - Test land size with units accepted (5 acres, 2 hectares)
    - Test land size without units rejected
    - **Validates: Requirements FR-INP-02, FR-INP-03, FR-INP-08, FR-INP-09, FR-SAF-04**

- [ ] 12. Implement what-if analysis
  - [ ] 12.1 Create what-if scenario handler
    - Accept changes to irrigation (none/partial/assured)
    - Accept sowing date shift (+2 weeks)
    - Accept storage access toggle (yes/no)
    - Generate deterministic scenario ID (hash of inputs)
    - Recompute recommendations with updated assumptions
    - Cache results by scenario ID
    - Highlight which recommendations changed and why
    - _Requirements: FR-WIF-01, FR-WIF-02, FR-WIF-03, FR-WIF-04, FR-WIF-06, FR-WIF-07, FR-WIF-08_
  
  - [ ]* 12.2 Write validation tests for what-if determinism
    - **Property 10: What-If Determinism**
    - Test identical inputs produce identical outputs
    - Test scenario ID is deterministic (hash of inputs)
    - Test irrigation change affects yield and costs
    - Test sowing date shift affects feasibility
    - **Validates: Requirements FR-WIF-07, FR-WIF-08**

- [ ] 13. Implement conversational agent
  - [ ] 13.1 Create agent orchestrator
    - Implement question policy (max 5 questions)
    - Collect required inputs (state, district, season)
    - Collect optional inputs (land_size, irrigation, risk_preference, storage, selling_mode)
    - Call recommendation engine with collected context
    - Format and present recommendations
    - Handle what-if requests
    - _Requirements: FR-INP-01, FR-INP-04, FR-INP-05, FR-INP-07_
  
  - [ ] 13.2 Implement error handling
    - Handle zero feasible crops (explain constraints, suggest adjustments)
    - Handle negative profits (warn user, explain cost drivers)
    - Handle missing data (use fallbacks, reduce confidence)
    - Handle invalid inputs (provide helpful error messages)
    - _Requirements: FR-FEA-05, FR-SAF-02, FR-SAF-03, FR-SAF-05_

- [ ] 14. Create FastAPI endpoints
  - [ ] 14.1 Implement REST API
    - POST /api/v1/recommendations - Generate recommendations
    - POST /api/v1/what-if - Run what-if scenario
    - GET /api/v1/supported-regions - List supported states and districts
    - GET /api/v1/supported-crops - List supported crops
    - _Requirements: FR-REC-01, FR-WIF-01_
  
  - [ ] 14.2 Add request/response models
    - Define Pydantic models for request validation
    - Define Pydantic models for structured responses
    - Add error response models

- [ ] 15. Implement data loading and caching
  - [ ] 15.1 Create data loader on startup
    - Load all static datasets (crop calendars, prices, yields, costs) on application startup
    - Cache in memory for fast access
    - _Requirements: FR-DAT-01, FR-DAT-02, FR-DAT-03_
  
  - [ ] 15.2 Implement recommendation caching
    - Cache recommendation results by user context hash
    - Cache what-if scenario results by scenario ID
    - Set reasonable TTL (e.g., 24 hours)
    - _Requirements: NFR-OFF-01, FR-WIF-07_

- [ ] 16. Create sample datasets
  - [ ] 16.1 Prepare crop calendar data
    - Create CSV with columns: state, district, crop, season, sowing_start, sowing_end, harvest_month
    - Include 3-5 districts from Punjab and Maharashtra
    - Include 5-8 crops (rice, wheat, cotton, soybean, chickpea, maize, mustard)
    - _Requirements: FR-SCP-01, FR-SCP-02_
  
  - [ ] 16.2 Prepare mandi price data
    - Create CSV with columns: state, district, mandi, crop, date, modal_price, min_price, max_price
    - Include 2-3 years of historical data
    - Include monthly data for harvest months
    - _Requirements: FR-DAT-02_
  
  - [ ] 16.3 Prepare yield and cost data
    - Create CSV for yields: state, district, crop, year, yield_quintals_per_acre
    - Create CSV for costs: state, crop, seed_cost, fertilizer_cost, labor_cost, irrigation_cost, other_cost
    - Use realistic values from CACP reports or state agriculture departments
    - _Requirements: FR-DAT-01, FR-DAT-03_

- [ ] 17. Manual validation and testing
  - [ ] 17.1 Execute manual test cases
    - Test 3 different districts with valid inputs
    - Test 2 invalid district-season combinations
    - Verify profit calculations for 5 crops manually
    - Verify P10 < P50 < P90 for all recommendations
    - Verify risk preference changes ranking
    - Verify confidence labels match data quality
    - Verify all explanations cite specific data sources
    - Verify disclaimer present in all outputs
    - Test what-if scenario (irrigation change)
    - Test zero feasibility scenario (invalid inputs)
    - _Requirements: All core properties_
  
  - [ ] 17.2 Document test results
    - Create test_results.md with inputs, outputs, and validation status
    - Include screenshots or JSON outputs for demo

- [ ] 18. Final checkpoint - System integration complete
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional test tasks that can be skipped for faster MVP delivery
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Focus on 7 core properties for MVP validation
- Use handcrafted test cases with sample data (3-5 districts, 5-8 crops)
- Property-based testing framework (Hypothesis) is designed but not implemented in hackathon MVP
- Full CI/CD pipeline is designed but not implemented in hackathon MVP
