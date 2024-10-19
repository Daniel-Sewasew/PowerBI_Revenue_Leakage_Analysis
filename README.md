## `Revenue Leakage Analysis`
### Table of Content 
- [Introduction](#introduction )
- [Business Question](#business-question)
- [ Requirement](#requirement)
- [Data Preparation](#data-preparation)
- [KPI and Definition](#kpi-and-definition)
- [Data Modeling](#data-modeling)
- [ Dashboard Graphics](#dashboard-graphics)
- [Row Level Security](#row-level-security )
- [Observation](#observation)
- [Recommendation](#recommendation)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Introduction 
In the financial domain, revenue leakage is the unnoticed loss of potential earnings due to process inefficiencies, errors, or mismanagement. It drains profitability when companies provide services or products but fail to capture or record the revenue, often occurring in billing, pricing, contract management, and payments.
This Power BI analysis aims to identify non-compliant activities, track revenue leakage and trends, assess total revenue, highlight key industries and offenders, pinpoint specific products or materials prone to non-compliance, and provide valuable insights to decision-makers.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Business Question 
- Do we have a lot of non-compliant activity?
- What is the Leakage amount/status for non-compliant invoices?
- Do the leakage trends change over time?
- How much total revenue is earned by the business? 
- What are the top 5 revenue leakage industries? 
- Who are the biggest offenders (Sales Representative, Account Coordinator, Customers)?
- Is non-compliance more common among specific item groups, materials, etc.?
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Requirement
1. The final report should contain a dashboard and, at most, three additional pages of analysis. 
2. The developer should decide what visuals, slicers, date filters, etc., best fulfill the requirements and answer the Business User Questions.
3. Only analyze activity for Item Type = INVENTORY.
4. Implement Row Level Security based on OPRID and Sales Rep Code (based on the Security tab).
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Data Preparation 
Using power query, the following tasks are performed. 
- Calculate a calculated column to identify invoice types as compliant vs non-compliant. 
- Compute general leakage measures to know the overall leakage status and for subsequent calculations of leakage percentages. 
- Calculate the percentage of the ratio to know the amount of invoice leakage. 
- Create the date table for further calculations related to the fact table with the invoice date. 
- Create a bridge table with a unique link between the security and transaction tables.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### KPI and Definition 
Using the Dax formula, the following tasks were accomplished. 
- Compliant = invoices where LIST_PRICE <= ACTUAL_PRICE.
- Non-Compliant = invoices where LIST_PRICE > ACTUAL_PRICE.
- Note: These two KPIs need to be created using the calculated column as they are also input for further calculations.
- Compliant Information =
```sql
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
  ```sql
     ( Transaction_Data[LIST_PRICE] - Transaction_Data[ACTUAL_PRICE] ) * Transaction_Data[QTY_SHIPPED]
  ```
- Leakage Actual: Calculation conditions: Leakage should not include Compliant transactions, where QTY_SHIPPED <= 0, where INVOICE_GM <= 0, or where the Leakage GM being calculated is > INVOICE_GM of that line. In these instances, Leakage GM should = $0.
```sql
IF (
    Transaction_Data[CompliantInfo] <> "Compliant", --Leakage should not include transactions that are Compliant,
    IF (
        Transaction_Data[QTY_SHIPPED] <= 0 --where QTY_SHIPPED <= 0,
            || Transaction_Data[INVOICE_GM] <= 0 --where INVOICE_GM <= 0
            || [Leakage GM] > Transaction_Data[INVOICE_GM], --where the Leakage GM being calculated is > INVOICE_GM of that line. In these instances, Leakage GM should = $0.
        0,
        [Leakage GM]
    ),
    [Leakage GM]
)
```
- Leakage % = Leakage GM / INVOICE_GM.
 ```sql
DIVIDE(
    SUM(Transaction_Data[Leakage GM]),SUM(Transaction_Data[INVOICE_GM]))
```
- Total Inventory Invoices: count total products sold where the item type is inventory.
```sql
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
   'Item Master'[Item Type] = "INVENTORY"
```
- Compliant Invoices: the count of total products sold, where type is inventory, and compliant type is compliant.
```sql
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
    'Item Master'[Item Type] = "INVENTORY",
    Transaction_Data[CompliantInfo] = "Compliant"
```
- Non-compliant invoices are the count of total products sold, where the type is inventory, and the compliant type is non-compliant.
```sql
CALCULATE(
    COUNT(Transaction_Data[PRODUCT_ID]),
    'Item Master'[Item Type] = "INVENTORY",
    Transaction_Data[CompliantInfo] = "Non Compliant"
```
- Total Revenue: the sum of actual price times shipped quantity.
```sql
  SUMX(
      Transaction_Data,
      Transaction_Data[ACTUAL_PRICE]  * Transaction_Data[QTY_SHIPPED]
        )
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Data Modeling 
A star schema was adopted for data model purposes. The SRC table is a brigade table between the transaction and security table with a unique sales representative code, as there are duplicates in the security table. A security table is used for role-level security requirements. 

![image](https://github.com/user-attachments/assets/621b1e6a-791d-466d-8a71-1751a2369ad2)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Dashboard Graphics
##### Main Page
![Main_Page](https://github.com/user-attachments/assets/be495f26-699b-4c69-acce-244ad437b6bf)

##### Main Page with Filter Pane
![Main_Page_FilterPane](https://github.com/user-attachments/assets/2ae724f3-5b42-4556-ac2a-d2be8fe1d66b)

##### Detail View 
![LinkPic2](https://github.com/user-attachments/assets/f44b98e7-169f-4bdc-936e-cc8288b2b7c8)

### Row Level Security 
RLS – Requirements:  
A dynamic role level security must be confirmed based on the security table and email address column information. 
For dynamic RLS, the user principal name Dax function dynamically tracks the email addresses of the report users who verify the username and open the report.  

![image](https://github.com/user-attachments/assets/6c83ab8b-16bf-41d5-8c9f-6df532e1cdda)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Observation
- Yes, there are non-compliant invoices. The total number of invoices was 2328. Among them, 2068 (88.83%) were compliant, whereas 260 (11.17%) were non-compliant. 
- Out of the $20.10 million in total revenue collected, about $442.552.08 (for 260 non-compliant invoices) was observed in revenue leakage or deficit. 
- Dec 2021 was the highest revenue leakage month with $-416,838.50, -50.84% of the deficit for 204 invoice orders. 
- The chart shows that revenue leakage remained relatively stable, nearly zero, from July 2021 to November 2021. A sharp drop in December 2021 indicates a significant revenue leakage, reaching approximately -400K.
- Services companies ($-430,287), food ($-51,273), and beverage ($-19,846) were the top three market industries that showed revenue leakage, respectively. 
- Others ($402,791), Containers ($-30,859), and Caps and Closers ($-18,903) are the top three item types with the highest revenue leakage items category. 
- The chart shows that the most significant revenue leakage comes from "Others" at -402,791, followed by "Containers" with -30,859 and "Caps &..." with -18,903, while "Packaging" has negligible or no leakage.
- Among the sales representatives, Tim experienced the highest sales leakage, -438,545, followed by Karen with -29,874, Sarah with -13,641, and Nick with the slightest leakage, -2,138.
- The result indicates that Whitney had the most significant leakage at -458,605, followed by Jon with -6,460. In contrast, Roxanne had no leakage, and Makenzie showed a positive leakage of 12,513 from the account coordinators.
- The top five customers contributing to revenue leakage are Oirko with -434,730, Epicin with -28,157, Pharmacal, Inc. with -24,590, Eouse-Kansas with -12,161, and BcCormick with -9,139.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Recommendation 
Here are recommendations based on the observations:
1. Address Non-Compliance Issues:
Since 11.17% of invoices are non-compliant, stricter compliance protocols should be established, and automated validation processes should be introduced to reduce non-compliance and associated revenue leakage.

2. Investigate Revenue Leakage:
The significant revenue leakage of $-442,552.08 (particularly in December 2021) warrants a deep investigation into the causes. Identify and rectify process inefficiencies or errors that led to such a sharp drop.

3. Focus on December 2021:
Given that December 2021 saw the highest revenue leakage, a detailed analysis of operations, invoicing errors, and external factors that may have caused the sharp drop in revenue should be conducted, and preventive measures should be developed for the future.

4. Target Industry-Specific Solutions:
The highest leakage occurs in service companies and the food and beverage industries. To reduce leakage and tighten revenue management, implement customized compliance measures and review contractual terms in these sectors.

5. Optimize Item Categories:
The "Others," "Containers," and "Caps and Closures" item categories have the highest revenue leakage. Review the pricing strategies, procurement processes, and inventory management for these items to minimize the financial loss.

6. Evaluate Sales Team Performance:
I'd like you to focus on improving the performance of sales representatives, particularly Tim, who experienced the highest leakage. Training, better sales tracking, and closer monitoring of high-risk transactions can help mitigate future losses.

7. Strengthen Account Coordination:
With Whitney showing the highest leakage among account coordinators, it may be necessary to reassess their handling of accounts and introduce training or process improvements to reduce errors.

8. Prioritize High-Risk Customers:
The top five customers with the highest leakage (Oirko, Epicin, Pharmacal, Inc., Eouse-Kansas, and McCormick) should be closely monitored. Strengthen contract compliance and negotiate better terms to prevent future losses from these customers.

### Tools Used
- Excel
- PowerBI
- Power Query 

