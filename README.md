
# Customer Enrichment Using Company House and LLM

## 📌 Overview

Customer Enrichment improves client data by adding missing or verified details from external sources like **Companies House (CH)** and Large Language Models (LLMs).

**Companies House** is the official UK government body that keeps records of all registered companies. It stores information such as company name, address, directors, and filing history.

A **Company Registration Number (CRN)** is a unique number given to every company when it is registered with Companies House.  
It is useful because:
- It uniquely identifies each company.
- It helps fetch the right company details from Companies House.
- It avoids mistakes like mixing up companies with similar names.

## Input
* Schema for Input and Output Results can be updated in configs/config Folder
- Table which you upload in Input Schema **{inputSchema}** should contains following columns : 
  - `client_id`  (Optional)
  - `client_name` (Mandatory)  
  - `client_address`  (Optional)
  - `client_post_code`  (Optional)
  - `existing_crn`  (Optional)
  - `existing_vat` (Optional)

Make sure that even if optional field values are missing, the give Input table must still include these column names.  
This ensures the enrichment workflow runs smoothly without errors.

## 📁 Project Structure

.
├── configs/                   # Configuration files
│   └── config
├── utils/                     # Utility functions (including LLM + CH)
│   ├── api_rotation/
│   ├── ch_utils/
│   ├── client_verification/
│   ├── compare_llm_ch_results/
│   ├── enrich_non_entity/
│   ├── enriching_clients/
│   ├── enrich_with_crn/
│   ├── enrichment_info_helper/
│   ├── filter_client_for_enrichment/
│   ├── helper_functions/
│   ├── llm_utils/
│   ├── load_input_data/
│   ├── sparkSession/
│   ├── store_data/
│   └── storing_final_enriched_clients/
├── main.ipynb                # Main notebook
└── README.md                 # Project documentation


## Default variables

| **Variable Name**                    | **Datatype** | **Default Value** | **Purpose**                                                      |
| ------------------------------------ | ------------ | ----------------- | ---------------------------------------------------------------- |
| `threshold_merge`                    | number       | 6                 | Accepted threshold for merge                                     |
| `threshold_possible_merge`           | number       | 5                 | Accepted threshold for possible merge                            |
| `threshold_total_match_score`        | number       | 0.7               | Accepted threshold for total match score                         |
| `avoid_dummy_values_count_threshold` | number       | 20000             | Threshold to avoid dummy record counts during matching           |
| `exclude_column_values`              | boolean      | True              | Whether to exclude columns listed in `exclude_column_values.csv` |


## 🔄 How it Works

### 1. Companies House API Key

* Create API keys via [Companies House Developer Portal](https://developer.company-information.service.gov.uk/).
* Unlimited keys can be created, free to use, no expiry.

### 2. Clients with CRN

* Validate CRN directly on Companies House.
* If valid → **High Confidence**, store in `{inputSchema}.intermediate_enriched_table`.
* If invalid → fallback to VAT/Name-based steps.

### 3. Clients with VAT ID

* Validate VAT ID via [UK Gov VAT service](https://www.tax.service.gov.uk/).
* Use scraping to fetch CRN → verify via Companies House.
* If valid → **High Confidence**, else fallback.

### 4. Companies House Enrichment (Name + Postcode)

* Generate variants of client name → query CH API sequentially.
* Assign **Confidence Levels**:

  * High (Exact Match) → Name >95%, Postcode match
  * High → Name >75%, Postcode match
  * Medium → Name >75%, Postcode mismatch
  * Low → Below threshold
  * None → No CRN

### 5. LLM-Based Enrichment (One-time for Low/None Matches)

* Model Used : "gpt-4o-mini-search-preview-2025-03-11"
* Functions:

  * `generate_entity_type_prompt()`
  * `generate_registration_lookup_prompt()`
* Confidence levels: High / Medium / Low / None (same logic as CH).
* Results stored in `{outputSchema}.intermediate_enriched_table`.

### 6. Non-Registered Entities (Trusts, Charities, Councils, Universities, Individuals)

* Identify non-companies (`Is_Company = False`) with Low/None confidence.
* Classify using regex (`Council`, `Mr/Mrs/Dr`, etc.).
* Map entity → sector via `determine_client_sector()`.
* Map sector → Client sector using `sic_code_mapping`.
* Store results in `{outputSchema}.nonentity_temp_table`.

### 7. Clients Needing Client Verification

* Extract from `{outputSchema}.client_verification_needed`.
* Perform **manual CRN lookup** (ChatGPT/Google).
* Upload enriched CSV back to Lakehouse for processing.

### 8. Retrieving Enriched Columns

* Select clients with **High-Exact/High/Medium Confidence** (CH + LLM).
* Enrich attributes:

* Company Type, Size, SIC Codes, Sector, Client Sector.
* Use YAML mapping (`ch_enriched_mapping.yml`) for standardisation.
* Map SIC (4-digit via YAML, 5-digit via scraping).

### 9. Storing Final Results

* Union enriched entities + non-entities.
* Store in **final table**:

  * `{outputSchema}.enriched_client_information`

---

## 📊 Final Output

* **Enriched attributes:**

  * `client_id`
  * `company_number`
  * `jurisdiction`
  * `company_size`
  * `company_status`
  * `sic_codes`
  * `client_sector`
  * `mapping_client_sector`
  * `client_sector_description`
  * `client_type`
  * `region`

## 🤝 Contact

   * Shubham Lingwal
   * Mugundhan K
   * Pardhu M

## 📊 Improvements

- Create Seperate CSV files of ch_enriched_mapping file . 
