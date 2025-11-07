# Hospital-Device-Tender-Margin
This project simulates a real-world MedTech sales analytics pipeline, designed to support strategic decision-making for hospital device tenders. It demonstrates a complete ETL → Data Warehouse → BI Dashboard workflow:
1. Extract messy operational CSV files
2. Transform & clean using SQL (MySQL Workbench)
3. Build dimensional data warehouse (star schema)
4. Prepare data for BI reporting (e.g., Power BI)
5. Create tender profit margin insights for business stakeholders

# Hospital Device Tender Margin (MedTech Analytics)

This project simulates an end-to-end MedTech sales analytics pipeline for hospital device tenders.  
It showcases how I clean messy operational data, model a star schema in SQL, and expose a single
trusted layer for BI tools like Power BI.

##concise project description
## Tech Stack

- MySQL (data cleaning, ETL, star schema)
- Power BI (dashboards & DAX)
- Git / GitHub (version control & documentation)

## Data Model

Source tables (dirty):
- `DimSalesRep_dirty`
- `DimHospital_dirty`
- `DimProduct_dirty`
- `FactSales_dirty`

Cleaned dimensions:
- `DimSalesRep_clean` – standardized IDs, names, rep regions, hire dates
- `DimHospital_clean` – cleaned hospital IDs, names, country/region, type
- `DimProduct_clean` – normalized ProductID (`P###`), consistent Category/SubCategory, SterileFlag → 1/0
- `DimDate_clean` – generated calendar from min/max invoice dates (Year, Quarter, Month, Week, Day)

Cleaned fact:
- `FactSales_clean`
  - Fixed IDs (ProductID / SalesRepID formats)
  - Parsed dates
  - Cleaned quantities (invalid/≤0 → NULL)
  - Normalized list/net prices (EU commas → decimals)
  - TenderFlag → 1 / 0 / NULL
  - Enforced only rows with valid dimension keys

Semantic layer:
- `vw_sales_star`
  - Joins `FactSales_clean` with all dimensions
  - Calculates core metrics (Revenue, PriceDelta)
  - Single source for reporting (Power BI connects here)

## Key Steps

1. **Ingest & Inspect**  
   Load raw CSVs into MySQL (`*_dirty` tables), profile for missing IDs, mixed formats, and inconsistent flags.
2. **Clean Dimensions**  
   Standardize IDs, text casing, booleans; remove duplicates; create `*_clean` dimension tables.
3. **Clean Fact Table**  
   Normalize dates, prices, quantities, and foreign keys; keep only referentially valid rows.
4. **Build Date Dimension**  
   Auto-generate `DimDate_clean` covering the transactional date range.
5. **Star Schema View**  
   Create `vw_sales_star` to expose a tidy, join-ready dataset for BI.
6. **Analytics**  
   Use Power BI to analyze revenue by rep, region, product, tender vs non-tender, and over time.

## How to Run (Local)

1. Create a MySQL database `medtech`.
2. Run scripts in `/sql` (in order):
   - `01_create_dirty_tables.sql`
   - `02_clean_dimensions.sql`
   - `03_clean_factsales.sql`
   - `04_dimdate.sql`
   - `05_star_view.sql`
3. Connect Power BI to MySQL and use `vw_sales_star` (and optionally `DimDate_clean`) for reporting.

##Mermaid data model diagram

```mermaid
erDiagram
    DimSalesRep_clean {
        varchar SalesRepID PK
        varchar RepName
        varchar Region
        date    HireDate
    }

    DimHospital_clean {
        varchar HospitalID PK
        varchar HospitalName
        varchar Country
        varchar Region
        varchar HospitalType
    }

    DimProduct_clean {
        varchar ProductID PK
        varchar Category
        varchar SubCategory
        tinyint SterileFlag
    }

    DimDate_clean {
        int     DateKey PK
        date    FullDate
        int     Year
        char    Quarter
        int     MonthNum
        varchar MonthName
        int     WeekNum
        varchar DayName
    }

    FactSales_clean {
        varchar InvoiceID
        date    InvoiceDate
        varchar HospitalID FK
        varchar ProductID FK
        varchar SalesRepID FK
        int     Qty
        decimal ListPrice
        decimal NetPrice
        tinyint TenderFlag
        char(3) Currency
    }

    FactSales_clean }o--|| DimSalesRep_clean : "SalesRepID"
    FactSales_clean }o--|| DimHospital_clean : "HospitalID"
    FactSales_clean }o--|| DimProduct_clean  : "ProductID"
    FactSales_clean }o--|| DimDate_clean     : "InvoiceDate = FullDate"

