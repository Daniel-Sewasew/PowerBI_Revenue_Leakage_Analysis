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
- Total Inventory Invoices: count of total products sold where the item type is inventory.
```
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
   'Item Master'[Item Type] = "INVENTORY"
```
- Compliant Invoices: the count of total products sold, where type is inventory, and compliant type is compliant.
```
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
    'Item Master'[Item Type] = "INVENTORY",
    Transaction_Data[CompliantInfo] = "Compliant"
```
- Non-compliant invoices are the count of total products sold, where the type is inventory, and the compliant type is non-compliant.
```
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
    'Item Master'[Item Type] = "INVENTORY",
    Transaction_Data[CompliantInfo] = "Non Compliant"
```
- Total Revenue: the sum of actual price times shipped quantity.
```
  SUMX(
      Transaction_Data,
      Transaction_Data[ACTUAL_PRICE]  * Transaction_Data[QTY_SHIPPED]
        )
```
### Data Modeling 
A star schema was adopted for data model purposes. The SRC table is a brigade table between the transaction and security table with a unique sales representative code, as there are duplicates in the security table. A security table is used for role-level security requirements. 

![image](https://github.com/user-attachments/assets/621b1e6a-791d-466d-8a71-1751a2369ad2)

### Observations
- Yes, there are non-compliant invoices. The total number of invoices was 2328. Among them, 2068 (88.83%) were compliant, whereas 260 (11.17%) were non-compliant. 
- Out of $20.10 million in total revenue collected, about $-442.552.08 (for 260 non-compliant invoices) was observed in revenue leakage or deficit. 
- Dec 2021 was the highest revenue leakage month with $-416,838.50, -50.84% of the deficit for 204 invoice orders. 
- The chart shows that revenue leakage remained relatively stable, nearly zero, from July 2021 to November 2021. A sharp drop in December 2021 indicates a significant revenue leakage, reaching approximately -400K.
- Services companies ($-430,287), food ($-51,273), and beverage ($-19,846) were the top three market industries that showed revenue leakage, respectively. 
- Others ($402,791), Containers ($-30,859), and Caps and Closers ($-18,903) are the top three item types with the highest revenue leakage items category. 
- The chart shows that the most significant revenue leakage comes from "Others" at -402,791, followed by "Containers" with -30,859, "Caps &..." with -18,903, while "Packaging" has negligible or no leakage.
- Among the sales representatives, Tim experienced the highest sales leakage, -438,545, followed by Karen with -29,874, Sarah with -13,641, and Nick with the slightest leakage, -2,138.
- The result indicates that Whitney had the most significant leakage at -458,605, followed by Jon with -6,460. In contrast, Roxanne had no leakage, and Makenzie showed a positive leakage of 12,513 from the account coordinators.
- The top five customers contributing to revenue leakage are Oirko with -434,730, Epicin with -28,157, Pharmacal, Inc. with -24,590, Eouse-Kansas with -12,161, and BcCormick with -9,139.

