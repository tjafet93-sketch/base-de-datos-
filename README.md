# RiwiSupply S.A.S. — Database Centralization & Relational Model

This repository contains the database schema, data loading scripts, and analytical queries designed to centralize and secure the operational records of **RiwiSupply S.A.S.**, a national distributor of industrial supplies.

---

## 1. Project Description
Historically, RiwiSupply S.A.S. managed all purchases, inventory movements, suppliers, and warehouse allocations within a single shared Excel spreadsheet. This operational approach led to severe data integrity issues:
- **Supplier redundancy:** Suppliers recorded multiple times with different name formats (e.g., "Aceros del Norte S.A.S.", "ACEROS NORTE").
- **Product inconsistencies:** Names and categories were inconsistent across records.
- **Geographical data dispersion:** City names were abbreviated or misspelled (e.g., "B/quilla" vs. "Barranquilla").
- **Transactional tracking difficulties:** Difficulty in computing accurate inventory stock, calculating warehouse valuations, and compiling supplier purchasing metrics.

This project implements a robust relational database schema designed to eliminate these anomalies, ensure referential integrity, and provide clean, reliable datasets for business analytics.

---

## 2. Technologies Used
- **Database Engine:** PostgreSQL (v15+ recommended) / MySQL compatible.
- **Modeling Notation:** Mermaid.js ER Diagram.
- **Query Language:** Standard SQL (DDL, DML, DQL) and PL/pgSQL (for PostgreSQL functions/stored procedures).

---

## 3. Explanation of the Normalization Process

The transformation from the raw Excel spreadsheet to a highly structured relational design progressed through the following normalization stages:

### Unnormalized Form (UNF)
The initial Excel file consolidated all attributes into one large flat table:  
`[MovementDate, SupplierName, SupplierCity, Warehouse, WarehouseCity, ProductName, Category, Quantity, UnitPrice, MovementType, PurchaseOrder]`  
*Problems identified:* Massive storage redundancy, update anomalies (updating a supplier name required modifying thousands of records), deletion anomalies (deleting a movement could remove the only record of a product), and lack of data types constraint.

### First Normal Form (1NF)
- **Goal:** Ensure atomicity of columns, eliminate repeating groups, and define a unique primary key.
- **Action:** Cell values were verified to contain only single atomic values. A composite primary key was identified to uniquely identify each transaction: `(riwi_purchase_order, riwi_product_name, riwi_movement_date, riwi_warehouse_name, riwi_movement_type)`. All raw strings were cleaned and standardized (e.g., mapping "B/quilla" to "Barranquilla").

### Second Normal Form (2FN)
- **Goal:** Fulfill 1NF and eliminate partial dependencies. Non-key attributes must depend on the whole composite primary key, not just a subset.
- **Action:** Attributes belonging to master entities were extracted into separate tables, isolating them from transactional tracking. 
  - `riwi_products_2fn`: isolated `riwi_product_name` and `riwi_category_name`.
  - `riwi_suppliers_2fn`: isolated `riwi_supplier_name` and `riwi_supplier_city`.
  - `riwi_warehouses_2fn`: isolated `riwi_warehouse_name` and `riwi_warehouse_city`.
  The transactional table `riwi_inventory_movements_2fn` retained only the foreign key IDs, quantity, unit price, movement type, purchase order, and transaction date.

### Third Normal Form (3FN)
- **Goal:** Fulfill 2NF and eliminate transitive dependencies. Non-key attributes must not depend on other non-key attributes.
- **Action:** 
  - Supplier cities and Warehouse cities depend transitively on their host entities. To remove this, we extracted `riwi_cities` and mapped supplier and warehouse tables to it via foreign keys.
  - Product categories depend transitively on the product ID. We extracted `riwi_categories` to centralize categories and mapped products to it.
  - This step eliminated text-entry errors (e.g., preventing a supplier city from being written as "Ctg" while a warehouse city is written as "Cartagena").

---

## 4. Database Structure

The schema design utilizes the mandatory prefix `riwi_` for all tables and attributes. Below is the structural overview:

### 1. `riwi_cities`
Stores unique geographical locations.
- `riwi_city_id` (SERIAL, PK): Unique identifier.
- `riwi_city_name` (VARCHAR, UNIQUE, NOT NULL): Standardized city name.

### 2. `riwi_categories`
Stores product classifications.
- `riwi_category_id` (SERIAL, PK): Unique identifier.
- `riwi_category_name` (VARCHAR, UNIQUE, NOT NULL): Category description.

### 3. `riwi_suppliers`
Contains standardized profiles of commercial vendors.
- `riwi_supplier_id` (SERIAL, PK): Unique identifier.
- `riwi_supplier_name` (VARCHAR, NOT NULL): Legal name of vendor.
- `riwi_city_id` (INT, FK): References `riwi_cities`.

### 4. `riwi_warehouses`
Defines operational storage centers.
- `riwi_warehouse_id` (SERIAL, PK): Unique identifier.
- `riwi_warehouse_name` (VARCHAR, UNIQUE, NOT NULL): Warehouse name.
- `riwi_city_id` (INT, FK): References `riwi_cities`.

### 5. `riwi_products`
Centralizes the inventory catalog.
- `riwi_product_id` (SERIAL, PK): Unique identifier.
- `riwi_product_name` (VARCHAR, NOT NULL): Product description.
- `riwi_category_id` (INT, FK): References `riwi_categories`.

### 6. `riwi_inventory_movements`
The central transaction table logging all purchase transactions and physical stock allocations.
- `riwi_movement_id` (SERIAL, PK): Unique transaction log identifier.
- `riwi_movement_date` (DATE, NOT NULL): Transaction timestamp.
- `riwi_supplier_id` (INT, FK): References `riwi_suppliers`.
- `riwi_warehouse_id` (INT, FK): References `riwi_warehouses`.
- `riwi_product_
