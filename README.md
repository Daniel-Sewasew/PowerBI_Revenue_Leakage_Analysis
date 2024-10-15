## Revenue Leakage Analysis
The Power BI analysis aims to identify non-compliant activities, track revenue leakage and trends, assess total revenue, highlight key industries and offenders, pinpoint specific products or materials prone to non-compliance, and provide valuable insights to decision-makers.
### Introduction  
In the financial domain, revenue leakage is the unnoticed loss of potential earnings due to process inefficiencies, errors, or mismanagement. It drains profitability when companies provide services or products but fail to capture or record the revenue, often occurring in billing, pricing, contract management, and payments.
### Business User Questions to be answered by Report:
- Do we have a lot of non-compliant activity?
- What is the Leakage amount/status for non-compliant invoices?
- Do the leakage trends change over time?
- How much total revenue is earned by the business? 
- What are the top 5 revenue leakage industries? 
- Who are the biggest offenders (Sales Representative, Account Coordinator, Customers)?
- Is non-compliance more common among specific item groups, materials, etc.?
### Requirements
1. The final report should contain a dashboard and no more than three additional pages of analysis. 
2. The developer should decide what visuals, slicers, date filters, etc., best fulfill the requirements and answer the Business User Questions.
3. Only analyze activity for Item Type = INVENTORY.
4. Implement Row Level Security based on OPRID and Sales Rep Code (based on the Security tab). 
### Data Preparation 
Using power query, the following tasks are performed. 
- Calculate a calculated column to identify invoice types as compliant vs non-compliant. 
- Compute general leakage measures to know the overall leakage status and for subsequent calculations of leakage percentages. 
- Calculate the percentage of the ratio to know the amount of invoice leakage. 
- Create the date table for further calculations to relate to the fact table with the invoice date. 
- Create a bridge table with a unique link between the security and transaction tables. 
### KPIs and Definitions 
Using the Dax formula, the following tasks were accomplished. 
- Compliant = invoices where LIST_PRICE <= ACTUAL_PRICE.
- Non-Compliant = invoices where LIST_PRICE > ACTUAL_PRICE.
- Note: These two KPIs need to be created using the calculated column as they are also input for further calculations.
- Compliant Information =
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
- Leakage General Measure = (LIST_PRICE - ACTUAL_PRICE) * QTY_SHIPPED
  ```
     ( Transaction_Data[LIST_PRICE] - Transaction_Data[ACTUAL_PRICE] ) * Transaction_Data[QTY_SHIPPED]
  ```
- Leakage Actual: Calculation conditions: Leakage should not include transactions that are Compliant, where QTY_SHIPPED <= 0, where INVOICE_GM <= 0, or where the Leakage GM being calculated is > INVOICE_GM of that line. In these instances, Leakage GM should = $0.
```
IF (
    Transaction_Data[CompliantInfo] <> "Compliant", --Leakage should not include transactions that are Compliant,
    IF (
        Transaction_Data[QTY_SHIPPED] <= 0 --where QTY_SHIPPED <= 0,
            || Transaction_Data[INVOICE_GM] <= 0 --where INVOICE_GM <= 0
            || [Leakage GM] > Transaction_Data[INVOICE_GM], --where the Leakage GM being calculated is > INVOICE_GM of that line. In these instances Leakage GM should = $0.
        0,
        [Leakage GM]
    ),
    [Leakage GM]
)
```
- Leakage % = Leakage GM / INVOICE_GM.
 ``` DIVIDE(
    SUM(Transaction_Data[Leakage GM]),SUM(Transaction_Data[INVOICE_GM]))
```
