# Power BI  Data Model Transformation

## Overview

This project demonstrates an end-to-end Power BI data modeling workflow in which a complex and poorly structured dataset is transformed into a clean, scalable analytical model.

The starting dataset intentionally represents many of the problems commonly found in real-world BI environments: inconsistent naming conventions, duplicated information, many-to-many relationships, inappropriate filter directions, mixed table grains, redundant tables, and transactional and descriptive data stored together.

The objective was to redesign the model using dimensional modeling principles and build a reliable foundation for Power BI reporting.

Rather than focusing only on dashboard visualization, this project focuses on the architecture underneath the reports.

---

## Project Objective

Transform a chaotic Power BI model into a structured analytical model that:

* follows star-schema principles
* clearly separates facts and dimensions
* preserves the correct grain of each table
* avoids unnecessary many-to-many relationships
* uses surrogate keys where appropriate
* minimizes redundant data
* maintains consistent naming conventions
* protects key business totals throughout transformations
* supports reusable DAX measures
* implements Row-Level Security (RLS)
* provides a reliable foundation for reporting

---

## The Challenge

The original dataset contained multiple modeling issues frequently encountered in business environments.

These included:

* fragmented customer information across several tables
* fragmented product information
* duplicate and unnecessary tables
* inconsistent entity naming
* transactional and descriptive attributes mixed together
* different grains across related tables
* many-to-many relationships
* inconsistent relationship directions
* repeated geographic information
* yearly order tables stored separately
* inventory stored in a wide monthly format
* campaign/product mappings stored as lists
* source-system technical columns unnecessary for analytics
* inconsistent naming conventions

The goal was not simply to clean individual tables, but to redesign the entire semantic model.

---

# Modeling Approach

The project was completed in four major phases.

## 1. Explore & Understand

Before transforming the data, I analyzed the source tables to understand:

* business entities
* table grain
* candidate dimensions
* candidate facts
* relationships between entities
* redundant information
* potential data-quality issues

Understanding the grain of every table was especially important before performing merges or creating relationships.

---

## 2. Build Dimensions

Related source tables were consolidated into reusable dimensions.

### Customer Dimension

Customer information was distributed across multiple source tables, including:

* customer master
* customer contacts
* user details
* addresses
* cities
* regions

These sources were analyzed and consolidated into a single customer dimension.

One important challenge involved customer contacts.

The customer master had a grain of:

> One row per customer

while the contact table had:

> Multiple contacts per customer

Directly merging these tables would duplicate customer records.

To preserve the dimension grain, only the appropriate primary-contact information was incorporated into the customer dimension.

This was an important example of why grain validation is critical before merging datasets.

---

### Product Dimension

Product and subcategory information was consolidated into a clean product dimension.

Transformations included:

* removing test records
* validating uniqueness
* standardizing attributes
* removing unnecessary source-system columns
* applying consistent naming conventions
* creating modeling keys

---

### Additional Dimensions

As transactional tables were analyzed, additional dimensions were extracted where descriptive attributes were embedded inside facts.

This included dimensions such as:

* geography
* campaign
* date
* other supporting analytical dimensions

A role-playing geographic dimension was also used where the same geography could represent different business contexts such as shipping and billing locations.

---

# Fact Table Development

After the dimensions were established, transactional processes were transformed into fact tables.

The project included fact structures for areas such as:

* sales
* inventory
* marketing campaigns
* order fulfillment
* other business transactions

Fact tables were connected to shared dimensions rather than directly to other fact tables.

This produces a cleaner model and prevents ambiguous filtering behavior.

---

## Sales Fact

The sales process required combining order-header and order-line information.

The final sales fact contains transactional measures such as:

* quantity
* price
* cost
* sales amount

Dimension attributes were replaced with dimension keys wherever appropriate.

Before and after major transformations, important business totals were validated to ensure that merges or relationship changes did not silently alter the results.

---

## Inventory Fact

The original inventory dataset stored months as separate columns.

This wide structure is not ideal for analytical modeling.

The table was reshaped into a normalized fact structure where the grain represents inventory by product and period.

This makes inventory analysis significantly easier and allows the date and product dimensions to filter the fact consistently.

---

## Campaign Modeling

Marketing campaign data required separating descriptive campaign information from daily campaign activity.

Campaign attributes were modeled as a dimension while transactional metrics such as spend, clicks, and impressions were modeled separately.

The campaign-to-product relationship also introduced a **factless fact table**.

A factless fact contains dimension keys but no additive numeric measure. Its purpose is to record that a relationship or event exists.

In this project it is used to represent campaign/product coverage.

---

## Order Fulfillment

The order lifecycle involved multiple business events, including:

* order
* shipment
* invoice
* payment

These processes were modeled to support analysis of the fulfillment lifecycle while maintaining appropriate grains and relationships.

---

# Star / Galaxy Schema

The original model resembled a highly interconnected web of tables.

The redesigned model follows dimensional modeling principles:
<img width="1536" height="1024" alt="ChatGPT Image Sep 15, 2026, 02_53_35 PM" src="https://github.com/user-attachments/assets/e6dfa850-1f43-44e3-b466-6994b0f2ebe3" />


```text
                 dim_customer
                       |
                       |
dim_product ---- fact_sales ---- dim_date
                       |
                       |
                    dim_geo


dim_product ---- fact_inventory ---- dim_date


dim_campaign ---- fact_campaign_spend ---- dim_date

      |
      |
fact_promotion_coverage
      |
      |
 dim_product


dim_customer ---- fact_order_process ---- dim_date
```

Because multiple fact tables share common dimensions, the final architecture can also be described as a **galaxy schema / fact constellation**.

The main modeling principle is:

> Dimensions filter facts. Facts do not directly filter other facts.

---

# Power Query Transformations

Power Query was heavily used to reshape the raw source data.

Key transformations included:

* creating references from staging queries
* merging related sources
* appending yearly transaction tables
* promoting headers
* filtering test records
* removing duplicates
* splitting columns
* expanding nested data
* grouping records
* unpivoting monthly data
* standardizing text
* renaming columns
* removing unnecessary attributes
* generating surrogate keys
* replacing source attributes with dimension keys
* disabling load for staging queries

Queries were organized into logical groups such as:

```text
01_stage
02_dimensions
03_facts
04_support
```

This keeps raw source queries separate from analytical tables.

---

# Naming Standards

Consistent naming conventions were applied throughout the model.

### Tables

Dimensions:

```text
dim_customer
dim_product
dim_date
dim_geo
dim_campaign
```

Facts:

```text
fact_sales
fact_inventory
fact_campaign_spend
fact_order_process
```

### Columns

Snake case was used consistently:

```text
customer_id
customer_key
product_id
product_key
order_date
line_total
```

Surrogate keys use the `_key` suffix to distinguish them from natural source-system IDs.

---

# Data Quality & Validation

A major focus of this project was ensuring that transformations did not silently change business results.

Important totals were captured before major transformations.

After merges, relationship changes, or restructuring, those totals were checked again.

Validation included:

* checking row counts
* checking uniqueness
* detecting duplicate keys
* validating table grain
* validating relationship cardinality
* comparing business totals before and after transformations
* testing dimension filtering
* testing final report results

This principle was followed throughout the project:

> Never assume a transformation is correct simply because Power BI accepts it.

---

# DAX Measures

A dedicated collection of reusable measures was created on top of the semantic model.

Examples include concepts such as:

```DAX
Total Sales =
SUM ( fact_sales[line_total] )
```

```DAX
Active Customers =
DISTINCTCOUNT ( fact_sales[customer_key] )
```

```DAX
Total Customers =
COUNTROWS ( dim_customer )
```

The purpose of the measures layer is to centralize business logic instead of repeatedly creating calculations inside individual visuals.

---

# Date Dimension

A dedicated date dimension was introduced to provide consistent time intelligence across the model.

The date table can be reused by multiple facts and supports analysis by attributes such as:

* date
* year
* quarter
* month
* month number
* other calendar hierarchies

This creates a common time-analysis layer across business processes.

---

# Row-Level Security

The project implements dynamic Row-Level Security based on user and region.

A security mapping associates users with the regions they are authorized to access.

The security logic filters the customer dimension, which then propagates the filter to connected fact tables.

The model uses the current Power BI user identity to determine which regional data should be visible.

The RLS implementation was tested using Power BI's **View As** functionality to verify that users only see the appropriate regional information.

---

# Key Modeling Principles Applied

### Understand the grain

Before merging or connecting tables, determine exactly what one row represents.

### Prefer star schemas

Dimensions surround fact tables and provide filtering context.

### Avoid fact-to-fact relationships

Shared dimensions should provide the analytical connection between business processes.

### Use single-direction filtering

Filters should generally flow:

```text
Dimension → Fact
```

### Remove unnecessary data

Columns that do not support analytics, relationships, security, or required business logic should not remain in the semantic model.

### Protect business totals

Important metrics should be validated throughout model development.

### Standardize the model

Consistent table names, column names, keys, data types, and formats make the model easier to understand and maintain.

---

# Skills Demonstrated

This project demonstrates practical experience with:

**Power BI**

* Semantic modeling
* Model relationships
* Cardinality
* Filter propagation
* Row-Level Security
* DAX measures

**Power Query**

* Data transformation
* Merge
* Append
* Group By
* Unpivot
* Data cleaning
* Query references
* Query organization

**Dimensional Modeling**

* Star schema
* Galaxy schema
* Fact tables
* Dimension tables
* Factless facts
* Surrogate keys
* Role-playing dimensions
* Grain analysis

**Data Quality**

* Duplicate detection
* Referential integrity
* Validation controls
* Reconciliation of business totals

---

# Before vs. After

## Before

The source model contained:

* inconsistent relationships
* many-to-many relationships
* duplicated information
* mixed grains
* redundant tables
* unclear filtering paths
* source-system attributes exposed directly to reporting

## After

The final model provides:

* clearly defined facts and dimensions
* consistent relationship direction
* shared dimensions
* reusable surrogate keys
* standardized naming
* cleaner Power Query architecture
* validated business totals
* centralized DAX measures
* dynamic Row-Level Security
* a scalable analytical foundation

---

# Repository Structure

```text
power-bi-nightmare-data-model/
│
├── README.md
│
├── power-bi/
│   └── power-bi-nightmare-project.pbix
│
├── data/
│   └── README.md
│
├── documentation/
│   ├── data-model.md
│   ├── transformations.md
│   └── security.md
│
└── images/
    ├── original-model.png
    ├── final-model.png
    └── dashboard-preview.png
```

---

# Project Takeaways

The most important lesson from this project is that a successful Power BI solution starts with the data model rather than the dashboard.

A visually impressive report cannot compensate for an incorrect semantic model.

Correctly defining table grain, separating dimensions from facts, controlling relationships, removing unnecessary data, and continuously validating business totals creates a model that is easier to understand, more reliable, and more scalable.

---

## Tools

* Microsoft Power BI Desktop
* Power Query
* DAX
* Dimensional Modeling
* Star Schema Design
* Row-Level Security

---


