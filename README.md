The original database and more details can be found at: https://www.kaggle.com/datasets/rikdifos/credit-card-approval-prediction

# Data Structure Analysis

Several data inconsistencies were identified during the initial exploration.

A mismatch was observed between credit_record and application_record: while 404 default-related records were found in credit_record, only 326 corresponding records were present in application_record. This suggests potential issues in the extraction logic, joining rules, or data origins, requiring validation with the data engineering team.

After a broader comparison between both datasets, it was identified that **402,100 out of 438,557 records (91.7%) do not match across the two tables**. This reinforces the hypothesis of structural inconsistencies in the data pipeline or historical availability, and indicates that any downstream analysis should be interpreted with caution until lineage validation is completed.

For unemployed clients, the value **365243** is used in DAYS_EMPLOYED as a sentinel value. However, when cross-checking with OCCUPATION_TYPE, nearly **31% of records are null**, despite having DAYS_EMPLOYED filled with values different from 365243. This suggests either an undocumented business rule or a pipeline inconsistency.

Another relevant inconsistency was found among pensioners. Approximately **75,000 pensioners** were identified, but only **138 of them (0.184%)** had DAYS_EMPLOYED populated. This may indicate:

1. Data leakage or inconsistencies during extraction or transformation.
2. A subset of clients who recently became pensioners, with incomplete historical updates.

Further investigation with the data engineering team would be required to clarify these scenarios.

# Additional Variables for Deep Dive

The dataset does not contain the exact age of clients. Instead, DAYS_BIRTH is provided as a negative value representing the number of days since birth. Age was approximated by multiplying this value by -1, dividing by 365, and rounding the result.

A similar approach was applied to DAYS_EMPLOYED, from which two additional variables were derived:

- Months employed

- Years employed

To support non-linear analysis and potential rule-based decisions, a categorical variable was created: **cluster of months employed**.

# Pattern Investigation and Data Analysis

For an initial assessment of default behavior, MONTHS = 0 was used to represent the current reference period. Clients with STATUS between 1 and 5 were flagged as having default behavior. Under this definition, 404 out of 33,856 clients (1.2%) were identified as currently in default.

An additional flag was created to identify clients with at least one default event in the last 12 months.

After isolating this population, it was observed that **409,223 out of 438,557 clients (93.3%)** did not have active credit exposure in the **last 12 months**. Therefore, subsequent analyses focus on the remaining **6.7%**, representing the effective risk-exposed population. Within the full dataset, **1.2% of clients experienced a 30-day default in the last 12 months**.

## Default Severity Analysis

Within the 6.7% exposed population:

- 72.1% experienced only a 30-day default

- 5.5% experienced a 60-day default

- 1.7% experienced a 90-day default

- 20.6% experienced a 120-day default

Clients aged **36–55 represent 51.4% of this group**, indicating overrepresentation relative to other age bands. A notable pattern emerges when focusing on severe defaults (120 days): clients **aged 46–55 are overrepresented by +11 percentage points** compared to the 30-day default group.

When combining age with employment stability, an even clearer pattern appears. Among clients with a 30-day default, **14.9% have less than two years of employment**, whereas this proportion increases to **25.0%** for clients with a 120-day default — a difference of **+10.1 percentage points**. This suggests that **employment stability plays a more significant role in default severity than age alone**.

# Final Considerations

This analysis prioritizes data integrity, interpretability, and decision relevance. Given the scale of inconsistencies identified between core datasets, any predictive modeling or automated decisioning should only proceed after proper data lineage validation. The findings reinforce the importance of combining stability-related features rather than relying on isolated demographic variables for risk assessment.
