# Nashville Housing Data - Data Cleaning

## Introduction
The Nashville Housing Data Cleaning project aims to prepare the dataset sourced from Nashville Housing for further analysis. The dataset contains information about housing properties in Nashville, including sales data, property addresses, owner details, and more. This file provides an overview of the SQL queries used to clean and preprocess the data.

## Table of Contents
1. [Standardize Date Format](#standardize-date-format)
2. [Populate Property Address Data](#populate-property-address-data)
3. [Breaking out Address into Individual Columns](#breaking-out-address-into-individual-columns-address-city-state)
4. [Change Y and N to Yes and No](#change-y-and-n-to-yes-and-no-in-sold-as-vacant-field)
5. [Remove Duplicates](#remove-duplicates)
6. [Delete Unused Columns](#delete-unused-columns)

---

### 1. Standardize Date Format<a name="standardize-date-format"></a>
```sql
-- Standardize Date Format
ALTER TABLE PortfolioProject..NashvilleHousing
ADD SaleDateConverted Date;

UPDATE PortfolioProject..NashvilleHousing
SET SaleDateConverted = CONVERT(Date, SaleDate);
```
Explanation:
- These queries standardize the date format in the 'SaleDate' column by converting it to the `Date` data type from `Datetime` data type using the `CONVERT` function.
- An `ALTER TABLE` statement adds a new column 'SaleDateConverted' to store the standardized date.
- Finally, an `UPDATE` statement populates the new column with the standardized dates.

### 2. Populate Property Address Data<a name="populate-property-address-data"></a>
```sql
- Populate Property Address data
SELECT x.ParcelID, x.PropertyAddress, y.ParcelID, y.PropertyAddress, ISNULL(x.PropertyAddress, y.PropertyAddress)
FROM PortfolioProject..NashvilleHousing AS x
JOIN PortfolioProject..NashvilleHousing AS y
	ON x.ParcelID = y.ParcelID
	AND x.[UniqueID ] <> y.[UniqueID ]
WHERE x.PropertyAddress IS NULL;

UPDATE x
SET PropertyAddress = ISNULL(x.PropertyAddress, y.PropertyAddress)
	FROM PortfolioProject..NashvilleHousing AS x
	JOIN PortfolioProject..NashvilleHousing AS y
		ON x.ParcelID = y.ParcelID
		AND x.[UniqueID ] <> y.[UniqueID ]
	WHERE x.PropertyAddress IS NULL;
```
Explanation:
- The first `SELECT` statement retrieves the records where the 'PropertyAddress' is null, ordered by 'ParcelID'.
- The subsequent queries populate missing 'PropertyAddress' values by matching records with the same 'ParcelID' from different instances of the dataset.

### 3. Breaking out Address into Individual Columns (Address, City, State)<a name="breaking-out-address-into-individual-columns-address-city-state"></a>
```sql
SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) AS PropertySplitAddress,
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) AS PropertySplitCity
FROM PortfolioProject..NashvilleHousing;

ALTER TABLE PortfolioProject..NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE PortfolioProject..NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1);

ALTER TABLE PortfolioProject..NashvilleHousing
ADD PropertySplitCity NVARCHAR(255);

UPDATE PortfolioProject..NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress));
```
Explanation:
- The initial `SELECT` statement breaks down the 'PropertyAddress' into separate columns for 'PropertySplitAddress' and 'PropertySplitCity' using string manipulation functions like `SUBSTRING` and `CHARINDEX`.
- Subsequent `ALTER TABLE` and `UPDATE` statements add and populate new columns respectively for the split address components.

### 4. Change Y and N to Yes and No in "Sold as Vacant" field<a name="change-y-and-n-to-yes-and-no-in-sold-as-vacant-field"></a>
```sql
UPDATE PortfolioProject..NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
						WHEN SoldAsVacant = 'N' THEN 'No'	
						ELSE SoldAsVacant
						END;
```
Explanation:
- Here, the `UPDATE` statement converts 'Y' and 'N' values to 'Yes' and 'No' respectively in the 'SoldAsVacant' field.

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
FROM PortfolioProject..NashvilleHousing)
DELETE
FROM RowNumCTE
WHERE row_num > 1;
```
Explanation:
- This query uses a Common Table Expression (CTE) to assign row numbers based on certain columns' values.
- Rows with duplicate values in specified columns ('ParcelID', 'PropertyAddress', 'SalePrice', 'SaleDate', 'LegalReference') are identified.
- Using DELETE, duplicate rows are removed, keeping only one instance of each unique combination.

### 6. Delete Unused Columns<a name="delete-unused-columns"></a>
```sql
ALTER TABLE PortfolioProject..NashvilleHousing
DROP COLUMN PropertyAddress, SaleDate, OwnerAddress, TaxDistrict;

SELECT *
FROM PortfolioProject..NashvilleHousing;
```
Explanation:
- The ALTER TABLE statement removes the specified columns ('PropertyAddress', 'SaleDate', 'OwnerAddress', 'TaxDistrict') from the dataset which are not needed.
- Following this alteration, a SELECT statement displays the updated structure of the dataset with the dropped columns.

---

By following these SQL scripts, the Nashville Housing Data has been cleaned and prepared for further analysis.
