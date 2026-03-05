# Oracle-Apex-ORDS-API-Examples
Oracle Apex ORDS API Examples

# E-Commerce Prep & FBA Inventory Management System

## 📖 Overview

This project is an end-to-end inventory and financial management system designed to bridge a Third-Party Logistics (3PL) Prep Center with Amazon FBA operations. It utilizes a **Python middleware script** to ingest external API data and an **Oracle APEX/PL/SQL backend** to handle complex FIFO (First-In-First-Out) inventory tracking, Cost of Goods Sold (COGS) calculations, and consolidated profit analysis.

## 🛠️ Technology Stack

* **Integration Middleware:** Python (`requests`, `json`, `datetime`)
* **Database:** Oracle Database (PL/SQL, Table Constraints)
* **API Gateway:** Oracle REST Data Services (ORDS)
* **Frontend/UI:** Oracle APEX (Interactive Grids, Dashboards, Dynamic Actions)

## ⚙️ Core Workflow & System Modules

### 1. Prep Center Integration & Product Sync (Python Middleware)

The system continuously synchronizes outbound shipping orders and inventory data from the Prep Center (`prepby.me`) to the Oracle database using a robust two-step Python script.

* **Step 1: Header Sync (Delta Load):** The script queries the APEX database for the `max_date` of the last synced order. It then pulls only the new order headers from the Prep Center API, significantly reducing network payload.
* **Step 2: Line Item & Batch Sync:** The script fetches detailed JSON data for pending orders. Crucially, it parses the `consumed_received_orders` array to handle split batches. If an order fulfills 50 units by taking 10 from "Batch A" and 40 from "Batch B," the script dynamically splits this into multiple database rows to ensure pinpoint cost accuracy.
* **Fault Tolerance:** The script is equipped with automatic retry logic, HTTP 429 (Rate Limit) handling with 15-minute sleep cycles, and 30-second timeouts to prevent infinite hangs.

### 2. Warehouse & Prep Setup (Cost Tracking)

Once the data hits the Oracle database, the system calculates the exact cost of every outbound shipment.

* **Batch-Level Costing:** Because the Python script splits inbound sources, the database links each shipped item back to its specific original purchase cost (`cost_price`) and the specific prep fee charged by the warehouse (`prep_cost`).
* **Dynamic Order Costs:** The database dynamically calculates the total order cost by summing the exact batch-level item costs, plus overarching fees like shipping, packaging, and platform commissions (e.g., an 18% fee for FBM orders).

### 3. Amazon FBA Inventory & FIFO Engine

The system treats Amazon FBA fulfillment centers as a secondary warehouse.

* **Stock Transfer:** When the Prep Center ships an order marked as "FBA," the items are virtually transferred into FBA Inbound stock, tied to the exact date they left the prep center.
* **Smart FIFO Deduction (PL/SQL):** When an FBA sale is recorded via the APEX Interactive Grid, a PL/SQL procedure triggers. It loops through available FBA inbound batches, sorted by the oldest prep shipment date, and automatically deducts the `QTY_SOLD`. It calculates the exact COGS based on the specific batches consumed.
* **Auto-Reversal Engine:** If a user accidentally enters a wrong sale and deletes the row, a reverse "LIFO" (Last-In-First-Out) PL/SQL trigger fires. It perfectly refills the exact batches that were just deducted, ensuring accounting remains flawlessly balanced without manual database intervention.

### 4. Financial Dashboard & Profit Analysis

The frontend features an APEX dashboard that provides a consolidated view of company health, completely separating Prep/FBM revenue from FBA revenue to prevent double-counting.

* **Data Aggregation:** A massive `FULL OUTER JOIN` SQL query aligns FBA Sales data (from the FBA table) with Prep/FBM data (from the Shipping Orders table) on a shared chronological timeline.
* **Strict Boundary Logic:** FBA inbound shipments are strictly excluded from the "Prep Profit" columns. Their costs and quantities sit silently in inventory until they are sold on Amazon, at which point they appear in the "FBA Profit" columns.
* **Granular Metrics:** The dashboard breaks down metrics by Month or Year, displaying:
* Total Volume (FBA Qty vs. Prep Qty)
* Gross Sales (FBA Sales vs. Prep Sales)
* Isolated Prep Center Fees
* Total Costs (COGS + Amazon Fees + Shipping)
* **Net Profit** (Calculated dynamically for both channels and combined into a Grand Total).

## 🚀 How to Run the Sync Script

1. Ensure you have Python 3.x installed.
2. Install the required libraries: `pip install requests`
3. Configure your API Keys and Oracle ORDS endpoint URLs in the `CONFIGURATION` section of the script.
4. Run the script

*(Note: Ensure your Oracle APEX tables have the proper Unique Constraints configured—e.g., `SHIPPING_ORDER_NO`, `SKU`, and `INBOUND_SOURCE_NUMBER`—to allow for split batch insertions).*

<img width="1901" height="871" alt="Image" src="https://github.com/user-attachments/assets/0542fc60-eed4-4e47-b529-08a9c64cf94a" />

<img width="1868" height="842" alt="image" src="https://github.com/user-attachments/assets/2c137dbe-3c07-47c5-a033-dab41e945cee" />

<img width="1885" height="771" alt="image" src="https://github.com/user-attachments/assets/1337f503-9312-4d68-9a9c-30131e697345" />

<img width="1875" height="790" alt="image" src="https://github.com/user-attachments/assets/b1ca2416-779c-49b4-9ce3-81d7de094023" />

<img width="1891" height="838" alt="image" src="https://github.com/user-attachments/assets/65fe8a8e-2f55-447d-a9fa-02b68ab6a466" />

<img width="1891" height="835" alt="image" src="https://github.com/user-attachments/assets/6437dfa8-c7cd-44ee-a96b-4c182d55d0f8" />

<img width="1903" height="823" alt="image" src="https://github.com/user-attachments/assets/32466394-1e6d-4e4f-866e-5b26fa84ef21" />

<img width="1899" height="660" alt="image" src="https://github.com/user-attachments/assets/d696091b-b9e1-46a8-9ff6-ff5e53359b22" />

<img width="1888" height="764" alt="image" src="https://github.com/user-attachments/assets/b20ed71e-157a-48a3-860e-9be2cd6a64f8" />

<img width="1903" height="728" alt="image" src="https://github.com/user-attachments/assets/8d1d09f1-62ad-4af9-a573-55daa81862a2" />

<img width="1889" height="772" alt="image" src="https://github.com/user-attachments/assets/91cd71a0-b55d-42d8-a955-6b23e6f5a877" />

<img width="1871" height="847" alt="image" src="https://github.com/user-attachments/assets/16257ea2-e67c-4804-a7d0-1b2ac3af554b" />

<img width="1878" height="801" alt="image" src="https://github.com/user-attachments/assets/602ddafc-9369-442d-9482-6657b3ca0200" />
