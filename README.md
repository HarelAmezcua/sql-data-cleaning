# 🧹 Data Cleaning with SQL: Nashville Housing Dataset

This repository demonstrates a complete **data cleaning workflow using SQL** applied to the **Nashville Housing dataset**. The dataset, sourced from a public property records database, required a series of transformations to make it analysis-ready.

---

## 📦 Dataset Overview

The dataset is stored in the table: `SQLPortfolio.dbo.NashvilleHousing` and contains property sales records in Nashville, including fields such as:

- Parcel ID
- Sale Date
- Sale Price
- Property Address
- Owner Address
- Legal Reference
- And other related attributes

---

## 🎯 Objectives

- Ensure **consistency in data formats**
- Address **missing values**
- **Normalize** and **break down** compound fields
- Improve readability through **column renaming and transformation**
- Remove **duplicate records**
- Drop **irrelevant or redundant columns**

---

## 🧰 SQL Operations Performed

### 1. 📅 Standardizing Date Format

```sql
ALTER TABLE NashvilleHousing ADD UpdatedSaleDate DATE;

UPDATE NashvilleHousing
SET UpdatedSaleDate = CONVERT(DATE, SaleDate);
````

A new `UpdatedSaleDate` column is created using a standardized `DATE` type.

---

### 2. 🏠 Populating Missing Property Addresses

Using self-joins on `ParcelID` to impute missing values:

```sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
  ON a.ParcelID = b.ParcelID AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL;
```

---

### 3. 🔍 Splitting Compound Columns

#### Property Address → Street and City

```sql
-- Example:
SET PropertyAddressStreet = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1);
SET PropertyAddressCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress));
```

#### Owner Address → Street, City, State (using `PARSENAME`)

```sql
-- Example:
SET OwnerAddressStreet = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3);
SET OwnerAddressCity   = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2);
SET OwnerAddressState  = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);
```

---

### 4. ✅ Converting Binary Flags

Convert values in `SoldAsVacant` column from `'Y'/'N'` to `'Yes'/'No'`:

```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
  WHEN SoldAsVacant = 'Y' THEN 'Yes'
  WHEN SoldAsVacant = 'N' THEN 'No'
  ELSE SoldAsVacant
END;
```

---

### 5. 🗑️ Removing Duplicates

Use a CTE with `ROW_NUMBER()` to detect and remove duplicate rows:

```sql
WITH RowNumCTE AS (
  SELECT *,
         ROW_NUMBER() OVER (
           PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
           ORDER BY UniqueID
         ) AS row_num
  FROM NashvilleHousing
)
DELETE FROM RowNumCTE
WHERE row_num > 1;
```

---

### 6. 🧹 Dropping Unnecessary Columns

Final cleanup to remove columns no longer needed:

```sql
ALTER TABLE NashvilleHousing
DROP COLUMN SaleDate, OwnerAddress, TaxDistrict, PropertyAddress;
```

---

## ✅ Final Output

The cleaned dataset includes standardized fields, decomposed address columns, binary fields in human-readable form, and no duplicates—ready for exploratory analysis, visualization, or modeling.

---

## 🧑‍💻 Author

**Harel Hernandez**
*Data Scientist | SQL Enthusiast*
Working at the intersection of data engineering and business intelligence.

---

## 📝 License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

---

## 📬 Feedback & Contributions

Feel free to open issues or submit pull requests if you have ideas or improvements. Let's make data cleaner, together.
