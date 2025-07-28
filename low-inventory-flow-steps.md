# 🧾 Power Automate Flow: Daily Low Inventory Replenishment Report

This Power Automate Cloud Flow automatically retrieves inventory items from Business Central that fall below a specified threshold, calculates the restock quantity, and sends an HTML email report.

---

## ✅ Trigger

### 🔹 Step 1: Recurrence
- **Type**: Scheduled
- **Configuration**:
  - Frequency: `Day`
  - Interval: `1`
  - Time zone: `Eastern Standard Time`
  - Start time: `06:00 AM`

---

## 📦 Data Processing

### 🔹 Step 2: Get Item Records from Business Central
- **Connector**: Business Central (Preview)
- **API Setup**:
  - API Publisher: `custom`
  - API Group: `inventoryreplenishment`
  - API Version: `v1.0`
  - Entity Set Name: `itemInventories`

### 🔹 Step 3: Filter array
- **Input**: `value` from previous step
- **Expression**:
```powerautomate
item()?['Inventory'] < item()?['InventoryThreshold']
```

### 🔹 Step 4: Initialize variable
- **Name**: `ReplenishmentItems`
- **Type**: `Array`
- **Value**:
```json
[]
```

### 🔹 Step 5: Apply to each
- **From**: Output of `Filter array`

#### 🔸 Inside: Append to array variable
- **Variable**: `ReplenishmentItems`
- **Value Expression**:
```powerautomate
json(concat('{',
'"Item No.":"', items('Apply_to_each')?['Number'], '",',
'"Description":"', items('Apply_to_each')?['Description'], '",',
'"Inventory":', string(items('Apply_to_each')?['Inventory']), ',',
'"Threshold":', string(items('Apply_to_each')?['InventoryThreshold']), ',',
'"Max Inventory":', string(items('Apply_to_each')?['MaximumInventory']), ',',
'"Replenish Qty":', string(sub(items('Apply_to_each')?['MaximumInventory'], items('Apply_to_each')?['Inventory'])),
'}'))
```

---

## 🧾 Report Generation

### 🔹 Step 6: Create HTML table
- **From**: `ReplenishmentItems` variable
- **Columns**: `Automatic`

### 🔹 Step 7: Send an email (V2)
- **To**: Email recipient (e.g., admin team)
- **Subject**:
```powerautomate
Low Inventory Report - @{utcNow('yyyy-MM-dd')}
```
- **Body**:
```html
<p>Hello,</p>
<p>The following items are below their inventory threshold and require restocking:</p>
@{body('Create_HTML_table')}
<p>Report generated automatically by Power Automate.</p>
<p>Thank you.</p>
```