# Nashville Housing Data Cleaning

## Introduction

This project aims to clean and prepare the Nashville housing data for further analysis. The dataset contains information about housing sales in Nashville, including details such as sale date, property address, owner address, sale price, and more. The cleaning process involves standardizing date formats, populating missing property addresses, breaking down address fields into individual columns, standardizing categorical values, removing duplicates, and deleting unused columns.

## Table of Contents
1. [Standardize Date Format](#standardize-date-format)
2. [Populate Property Address Data](#populate-property-address-data)
3. [Breaking out Address into Individual Columns](#breaking-out-address-into-individual-columns)
4. [Change Y and N to Yes and No in "Sold as Vacant" Field](#change-y-and-n-to-yes-and-no-in-sold-as-vacant-field)
5. [Remove Duplicates](#remove-duplicates)
6. [Delete Unused Columns](#delete-unused-columns)

---

### 1. Standardize Date Format<a name="standardize-date-format"></a>

```sql
SELECT SaleDate, CONVERT(Date, SaleDate)
FROM PortfolioProject..NashvilleHousing
```
This query selects the `SaleDate` column from the `PortfolioProject..NashvilleHousing` table and converts it to a standardized date format using the `CONVERT` function.

```sql
UPDATE PortfolioProject..NashvilleHousing
SET SaleDate = CONVERT(Date, SaleDate)
```
This update statement applies the converted date format to the `SaleDate` column in the `PortfolioProject..NashvilleHousing` table.

---

### 2. Populate Property Address Data<a name="populate-property-address-data"></a>

```sql
SELECT x.ParcelID, x.PropertyAddress, y.ParcelID, y.PropertyAddress, ISNULL(x.PropertyAddress, y.PropertyAddress)
FROM PortfolioProject..NashvilleHousing AS x
JOIN PortfolioProject..NashvilleHousing AS y
	ON x.ParcelID = y.ParcelID
	AND x.[UniqueID ] <> y.[UniqueID ]
WHERE x.PropertyAddress IS NULL
```
This query identifies records with missing `PropertyAddress` by comparing rows with the same `ParcelID`. It then populates missing addresses with available ones.

---

### 3. Breaking out Address into Individual Columns<a name="breaking-out-address-into-individual-columns"></a>

```sql
ALTER TABLE PortfolioProject..NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE PortfolioProject..NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)
```
These statements add a new column `PropertySplitAddress` to the table and populate it with the part of `PropertyAddress` before the comma.

Similar operations are performed for breaking down the city part of the address, i.e., `PropertySplitCity`.

---

### 4. Change Y and N to Yes and No in "Sold as Vacant" Field<a name="change-y-and-n-to-yes-and-no-in-sold-as-vacant-field"></a>

```sql
UPDATE PortfolioProject..NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
						WHEN SoldAsVacant = 'N' THEN 'No'	
						ELSE SoldAsVacant
						END
```
This update statement replaces 'Y' with 'Yes' and 'N' with 'No' in the `SoldAsVacant` column.

---

### 5. Remove Duplicates<a name="remove-duplicates"></a>

```sql
WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY UniqueID) AS row_num
FROM PortfolioProject..NashvilleHousing
)
SELECT *
FROM RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress
```
This query utilizes a Common Table Expression (CTE) to assign row numbers to duplicate records based on specific columns. Then, it selects and displays duplicate records based on the row numbers assigned.

---

### 6. Delete Unused Columns<a name="delete-unused-columns"></a>

```sql
ALTER TABLE PortfolioProject..NashvilleHousing
DROP COLUMN PropertyAddress, OwnerAddress, TaxDistrict
```
These statements remove the `PropertyAddress`, `OwnerAddress`, and `TaxDistrict` columns from the `PortfolioProject..NashvilleHousing` table, assuming they are no longer needed for analysis.
