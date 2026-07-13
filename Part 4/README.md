"# Capstone Project - Part 4" 

# LLM-Powered Feature Engineering and Extraction Pipeline

    This repository contains the end-to-end implementation of an LLM-powered structured processing pipeline. To meet the project requirements, Track B (Tabular Record Batch Scoring) was selected as the core feature track. The implementation also provides foundational setups for Track A and Track C data flows, complete with data validation schemas, a zero-trust PII guardrail, and precise deterministic configurations.

# 1. System Prompts and User Prompt Templates

Below are the exact templates utilized in the production script to format data packets sent to the language model.

# System Prompt (Track B: Few-Shot Batch Scoring)

You are a precise data analysis assistant. Evaluate the provided transaction record and assign risk assessments according to strict logic metrics.

Provide a structured evaluation with exactly these 5 required fields:
1. risk_tier ("low" | "medium" | "high")
2. flag_for_review (boolean)
3. primary_signal (string summary)
4. confidence ("low" | "medium" | "high")
5. recommended_action (string directive)

### Worked Example:
Input JSON:
{
  "order_date": "05/12/2024 0:00",
  "customer_name": "Jane Doe",
  "gender": "Female",
  "product_name": "Premium Blender",
  "quantity": 2,
  "unit_price": 500.00,
  "discount_pct": 0.25,
  "sales_amount": 750.00,
  "profit": -50.00
}

Output JSON:
{
  "risk_tier": "high",
  "flag_for_review": true,
  "primary_signal": "Negative profit margin coupled with an aggressive 25% discount.",
  "confidence": "high",
  "recommended_action": "Audit discount strategy and flag for immediate manager intervention."
}

You must return ONLY a validated JSON object matching the 5 required scalar fields above. Do not include markdown fences, extra keys, or conversational text.

# User Prompt Template (Track C: Explanation Reference)

Feature values:
  product_name={product_name}, depth={depth}, table={table},
  x={x}, y={y}, z={z},
  cut_encoded={cut_encoded}, color_encoded={color_encoded}, clarity_encoded={clarity_encoded}

Predicted class: {predicted_class}
Predicted probability: {predicted_probability:.4f}

Explain this prediction in the requested JSON format.


# 2. Technical Parameters & Security Configuration

# Deterministic Sampling Configuration

  Temperature: 0.0

  Rationale: Setting temperature=0.0 forces objective determination matching strict validation rules. It minimizes token variance, stabilizes structurally complex JSON syntax output, and ensures that identical corporate data profiles generate identical risk evaluations across concurrent runs.

# Security Architecture

The pipeline strictly enforces a zero-trust architecture regarding environment variables.

   API Key Exposure Mitigation: No private keys are hardcoded in source control. The orchestration layer extracts credentials explicitly at runtime using os.environ.get('MY_API_KEY') 

   PII Guardrail Mitigation: Before executing an external inference thread, raw text strings pass through an active pre-processing layer that regex-scans for common email format patterns ([a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+) and 10-digit phone number blocks. If matching sequences are identified, the pipeline halts execution for that record instantly, avoiding upstream PII leaks.



# Comparative Analysis
At temperature=0.0, the system achieves total predictability, ensuring that structural validations never fail due to stylistic drift. Conversely, at temperature=0.7, the model frequently introduces formatting flourishes or unrequested fields, breaking the downstream operational schema parser (jsonschema.ValidationError) and forcing the pipeline onto defensive fallback behaviors.

# 4. Production Demonstration Tables

The execution traces of the pipeline process datasets flawlessly across all defined tracks, adhering to strict schema patterns. On schema structural collapse or typing violations, an explicit catching framework intercepts the error code and provisions an automated FALLBACK_DICT.

# Python Code
 # Output verification schema applied dynamically via jsonschema.validate()
OUTPUT_SCHEMA = {
    "type": "object",
    "properties": {
        "customer_sentiment": {"type": "string"},
        "is_high_value_order": {"type": "boolean"},
        "total_discount_impact": {"type": "number"},
        "estimated_delivery_weeks": {"type": "number"},
        "risk_flag": {"type": "boolean"}
    },
    "required": ["customer_sentiment", "is_high_value_order", "total_discount_impact", "estimated_delivery_weeks", "risk_flag"]
}


# 5. Defensive Exception Pipeline Mechanics
The notebook defines an unambiguous fallback model ensuring structural data storage reliability:

 # python
  try:
    parsed_json = json.loads(stripped_response)
    jsonschema.validate(instance=parsed_json, schema=OUTPUT_SCHEMA)
    return parsed_json, "pass"
except jsonschema.ValidationError as ve:
    logger.error(f"Validation Error Encountered: {ve.message}")
    return FALLBACK_DICT, f"fail ({ve.message})"
except json.JSONDecodeError as jde:
    logger.error(f"JSON Structure Corrupted: {jde.msg}")
    return FALLBACK_DICT, f"fail ({jde.msg})"

Whenever a payload breaks compliance rules or errors out during model prediction processes (joblib.load('best_model.pkl') downstream operations or schema checks), fields automatically map safely to a default state (None values inside FALLBACK_DICT), securing system availability throughout big data runtime dependencies.

# Structured LLM Pipeline Deployment Guide

This project establishes a highly deterministic pipeline for data extraction, quality evaluation, and model explanation using Large Language Models (LLMs). 

## 🛠 Prompt Blueprints

### Track A: Extraction
* **System Prompt:** `You are a structured data extractor. Your sole task is to extract product order data from unstructured text and format it into a valid JSON object matching the provided schema. Output only valid JSON. Do not include any conversational filler, explanations, or Markdown formatting blocks.`
* **User Prompt Template:**
    ```text
    Extract the order data from the following text blocks into the required JSON format.

    ### Example 1
    Input: On August 15, 2024, Aarav Mehta (Male) purchased 8 units of Sugar...
    Output: {"order_date": "08/15/2024", ...}

    ### Example 2
    Input: We processed an order on 08/29/2021 for Pooja Sharma...
    Output: {"order_date": "08/29/2021", ...}

    ### Actual Input
    Input: {{INPUT_TEXT}}
    Output:
    ```

### Track B: Quality Evaluation
* **System Prompt:** `You are an automated data quality auditor. Evaluate the provided dataset record against the strict quality criteria below and output a completed JSON assessment object. Output only valid JSON... [Rubric: data_completeness, format_compliance, business_logic_validity]`
* **User Prompt Template:** ```text
    Evaluate this record: {{RECORD_JSON}}
    ```

### Track C: Explainable AI
* **System Prompt:** `You are an Explainable AI (XAI) agent. Your role is to interpret machine learning predictions and generate structured, human-readable explanations... Output a valid JSON containing prediction_summary, top_contributing_feature, and rationale.`
* **User Prompt Template:**
    ```text
    Features: {{FEATURES}}
    Predicted Class: {{PREDICTED_CLASS}}
    Predicted Probability: {{PREDICTED_PROBABILITY}}
    ```

### Why Temperature = 0?
Setting `temperature=0` forces the LLM to operate deterministically. For programmatic tasks like data extraction and JSON parsing, any behavioral variance can break downstream code. A temperature near 0 ensures that the model always yields predictable, consistent, and syntactically sound outputs suitable for processing pipelines.

---

## 🔬 Temperature A/B Comparison

| Track / Input | Output at temp=0 | Output at temp=0.7 | Key Difference |
| :--- | :--- | :--- | :--- |
| **Track A** (Text: *Aditya Patel (Female) bought 5 Fiction Novels on 04/24/2022 for 610.98 each...*) | `{"order_date": "04/24/2022", "customer_name": "Aditya Patel", "gender": "Female", "product_name": "Fiction Novel", "quantity": 5, "unit_price": 610.98...}` | 
