# PowerBI_Revenue_Leakage_Analysis
Power BI analysis aims to identify non-compliant activities, track revenue leakage and trends, assess total revenue, highlight key industries and offenders, and pinpoint specific products or materials prone to non-compliance.
```
IF (
    Transaction_Data[LIST_PRICE] <= Transaction_Data[ACTUAL_PRICE],
    "Compliant",
    IF (
        Transaction_Data[LIST_PRICE] > Transaction_Data[ACTUAL_PRICE],
        "Non Compliant"
    )
)
```
