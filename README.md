# TheCropWatch_AIForBharat

## Farmer Profit Advisor (Hackathon MVP)

An uncertainty-aware crop recommendation prototype for Indian farmers.  
The system ranks feasible crops for a selected district and season to help maximize expected profit, while showing risk and confidence.

## What It Does

- Collects required inputs: state, district, season, and sowing window
- Accepts optional inputs: land size, irrigation, storage, risk preference, selling mode
- Filters crops by feasibility (district + season + sowing window)
- Estimates profit ranges per crop as `P10/P50/P90` (and total profit if land size is provided)
- Returns `3` top-ranked crops plus `1` conservative fallback
- Adds confidence labels (`low/medium/high`) with short reasons
- Generates simple, data-grounded explanations and supports what-if scenarios

## MVP Scope

- Geography: district-level in `1-2` Indian states
- Crops: `5-10` major crops
- Seasons: `Kharif`, `Rabi`, `Zaid`
- Data focus: mandi prices, crop calendars, weather, yield/cost references

## Safety and Product Guardrails

- Recommendations are advisory only (no guaranteed returns)
- Crops outside supported states/crop list are excluded
- If data quality is weak, confidence is reduced and conservative outputs are preferred

## Architecture (High Level)

`Mobile App -> API Gateway -> Agent Orchestrator -> Recommendation Engine -> Data Services/Pipeline`

Core recommendation engine modules:
- Feasibility Filter
- Profit Estimator
- Ranking Engine
- Explanation Generator
- Confidence Assessor

## Specs

- Requirements: `.kiro/specs/farmer-profit-advisor/requirements.md`
- Design: `.kiro/specs/farmer-profit-advisor/design.md`
