
# Operation Data Analysis Project

## Data Sources
- **SQL**: System Data (CRM).
- **Google Sheets**: Additional Data for Marketing Knowledge.

---

## Step 1: Explore the Data 

### Objective
Understand the basic structure of the data and determine the proportion of missing values.

---

## SQL Queries and Analysis

### **1. Opportunities Table**

#### Query:
```sql
SHOW TABLES;
DESCRIBE opportunities;

SELECT 
    COUNT(*) AS "Count Cell", 
    SUM(sales_agent_id IS NULL) AS "Sales agent null",
    (SUM(sales_agent_id IS NULL) * 100.0 / COUNT(*)) AS "Sales agent null %",
    SUM(product_id IS NULL) AS "Product null",
    (SUM(product_id IS NULL) * 100.0 / COUNT(*)) AS "Product null %",
    SUM(account_id IS NULL) AS "Account null",
    (SUM(account_id IS NULL) * 100.0 / COUNT(*)) AS "Account null %",
    SUM(deal_stage_id IS NULL) AS "Status null",
    (SUM(deal_stage_id IS NULL) * 100.0 / COUNT(*)) AS "Status null %",
    SUM(engage_date IS NULL) AS "Engage date null",
    (SUM(engage_date IS NULL) * 100.0 / COUNT(*)) AS "Engage date null %",
    SUM(close_date IS NULL) AS "Close date null",
    (SUM(close_date IS NULL) * 100.0 / COUNT(*)) AS "Close date null %",
    SUM(close_value IS NULL) AS "Close value null",
    (SUM(close_value IS NULL) * 100.0 / COUNT(*)) AS "Close value null %",
    SUM(compain_tag IS NULL) AS "Campaign null",
    (SUM(compain_tag IS NULL) * 100.0 / COUNT(*)) AS "Campaign null %",
    SUM(CASE WHEN deal_stage_id = 1 THEN close_value ELSE 0 END) AS "Total close value"
FROM opportunities;
```

#### Comments:
- **Total Rows**: 8799.
- **Key Insights**:
  - `account_id`: 16.20% null values.
  - `engage_date`: 5.68% null values.
  - `close_date` & `close_value`: 23.74% null values.

- **Management Implications**:
  - Null `account_id` indicates no actions have been taken.
  - Null `engage_date` implies a deal is assigned but not engaged yet.
  - Null `close_date` means the lead is still under processing or negotiation.

---

### **2. Account Table**

#### Query:
```sql
DESCRIBE account;

SELECT 
    COUNT(*) AS "Total ROWs",
    SUM(name IS NULL) AS "Name null",
    SUM(sector_id IS NULL) AS "Sector ID null",
    SUM(year_established IS NULL) AS "Year established null",
    SUM(employees IS NULL) AS "Employees null",
    SUM(office_location_id IS NULL) AS "Office Location ID null",
    SUM(subsidiary_id IS NULL OR subsidiary_id = '') AS "Subsidiary ID null",
    (SUM(subsidiary_id IS NULL OR subsidiary_id = '') * 100.0 / COUNT(*)) AS "Subsidiary null %"
FROM account;
```

#### Comments:
- **Total Rows**: 8799.
- **Key Insights**:
  - `subsidiary_id`: 82.35% null values.
- **Management Implications**:
  - Null `subsidiary_id` signifies that the account is sovereign and not linked as a sub-account.

---

### **3. Employee Table**

#### Query:
```sql
DESCRIBE employee;

SELECT 
    COUNT(*) AS "Total ROWs",
    SUM(name IS NULL) AS "Name null",
    SUM(mgr_id IS NULL) AS "Manager null",
    SUM(regional_office_id IS NULL) AS "Regional Office null",
    SUM(status_id IS NULL) AS "Status null"
FROM employee;
```

#### Comments:
- **Total Rows**: 41.
- **Key Insights**:
  - All columns have complete data (no null values).
- **Management Implications**:
  - `mgr_id` indicates reporting structure. For example, a value of `1` implies direct reporting to the CCO.

---

### **4. Product Table**

#### Query:
```sql
DESCRIBE product;

SELECT 
    COUNT(*) AS "Total ROWs",
    SUM(name IS NULL) AS "Name null",
    SUM(series_id IS NULL) AS "Series ID null",
    SUM(price IS NULL) AS "Price null"
FROM product;
```

#### Comments:
- **Total Rows**: 7.
- **Key Insights**:
  - No null values in this table.

---

### **5. Auxiliary Tables**

#### Description:
The following tables have two columns (`id` and `name`), with no null values:
- **sector**: Categorizes account industries.
- **office_location**: Tracks the country of account origin.
- **regional_office**: Tracks the regional association of employees.
- **deal_stage**: Monitors client interaction stages.
- **series**: Associates products with a specific series.
- **emp_status**: Indicates whether an employee is current or former.


---

### **6. Complex Query for Dashboard**

#### **Objective**
Extract all the data required to design a dashboard covering sales performance, geographic analysis, and KPIs.

#### **Query**:
```sql
SELECT 
    opportunities.id AS "opportunities_id",
    a.name AS "Sales_name",
    CASE 
        WHEN a.mgr_id = 1 THEN "team leader"
        ELSE "agent"
    END AS "employee_type",
    product.name AS "product",
    COALESCE(product.price, 0) AS "product_price",
    COALESCE(acc.name, "not assigned") AS "account",
    CASE 
        WHEN COALESCE(acc.name, "not assigned") = "not assigned" THEN "not assigned account"
        ELSE COALESCE(sup_acc.name, "main account")
    END AS "sup_account",
    deal_stage.name AS 'deal_stage',
    opportunities.engage_date,
    opportunities.close_date,
    CASE 
        WHEN MONTH(close_date) BETWEEN 1 AND 3 THEN 'Q1'    
        WHEN MONTH(close_date) BETWEEN 4 AND 6 THEN 'Q2'  
        WHEN MONTH(close_date) BETWEEN 7 AND 9 THEN 'Q3'  
        WHEN MONTH(close_date) BETWEEN 10 AND 12 THEN 'Q4'
        WHEN engage_date IS NOT NULL THEN 'negotiation'
        ELSE 'no action yet'
    END AS "quarter",
    (opportunities.close_date - opportunities.engage_date) AS "SLA",
    COALESCE(opportunities.close_value, "0") AS "close_value",
    COALESCE(office_location.name, "not assigned account") AS "office_location",
    regional_office.name AS "regional_office",
    sector.name AS "sector",
    series.name AS "series_name",
    COALESCE(b.name, "CCO") AS "manager"
FROM 
    opportunities
LEFT JOIN  
    product ON opportunities.product_id = product.id 
LEFT JOIN  
    account acc ON opportunities.account_id = acc.id 
LEFT JOIN 
    account sup_acc ON acc.subsidiary_id = sup_acc.id 
LEFT JOIN  
    employee a ON opportunities.sales_agent_id = a.id 
LEFT JOIN  
    office_location ON acc.office_location_id = office_location.id 
LEFT JOIN  
    regional_office ON a.regional_office_id = regional_office.id
LEFT JOIN  
    deal_stage ON opportunities.deal_stage_id = deal_stage.id
LEFT JOIN  
    sector ON acc.sector_id = sector.id
LEFT JOIN  
    series ON product.series_id = series.id
LEFT JOIN
    employee b ON a.mgr_id = b.id;
```

---

## **7. Dashboard Design Suggestions**

### **Components**:
1. **Sales Performance**:
   - Analyze sales by employee and location.
2. **Geographical Analysis**:
   - Visualize deal distribution by region.
3. **KPIs**:
   - Track revenue and closed deals.
4. **Pipeline Status**:
   - Monitor client interaction stages.

---
