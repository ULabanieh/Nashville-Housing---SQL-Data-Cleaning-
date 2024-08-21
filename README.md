# Project Overview
---
This is a data cleaning project about a dataset of house listings in the city of Nashville, USA. The project contains different types of data cleaning and transformation procedures that serve to make the data more readable, understandable and easier to work with. I will show you here an overview of the most important queries and parts of the project. For the full detailed project, feel free to download the project files. 

# Import Dataset
---
The dataset is in .xlsx format, so to import it into MS SQL Server, follow these steps:
1. Open SQL Server Import and Export Wizard
2. Select "Microsoft Excel" as data source
3. Browse file path and choose the correct Excel version (in this case it would be Microsoft Excel 2007-2010)
4. In the "Choose a Destination" prompt select "Microsoft OLE DB Driver for SQL Server"
5. Select "Copy data from one or more tables or views"
6. Select "Sheet 1", click Next and then "Run immediately"
7. Click Finish and wait for import process to finish

# SQL Data Cleaning
---
Here I will be outlining the main SQL queries I used to perform the data cleaning necessary on this dataset along with the output of those queries.

## Standardize Date Format
---
```SQL
SELECT SaleDateConverted
FROM Nashville_Housing

ALTER TABLE Nashville_Housing
ADD SaleDateConverted Date; 

UPDATE Nashville_Housing
SET SaleDateConverted = CONVERT(Date, SaleDate)
```

![1446-02-17 19_37_00-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/7afa1b3e-b826-4c53-b7a7-007efd412f77)


## Populate Property Address Data
---
```SQL
SELECT *
FROM Nashville_Housing
--WHERE PropertyAddress IS NULL
ORDER BY ParcelID

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashville_Housing a
JOIN Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashville_Housing a
JOIN Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL
```
![1446-02-17 19_43_39-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/d21a05ba-6aa7-4c8d-9e0a-1da76396b00b)


Output is empty because there are no longer NULL values in the PropertyAddress column


## Breaking out address into individual columns (Address, City, State)
---
```SQL
-- Breaking out Address into individual columns (Address, City, State) - PropertyAddress Column

SELECT PropertyAddress
FROM Nashville_Housing
--WHERE PropertyAddress IS NULL
--ORDER BY ParcelID

SELECT
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) AS Address
, SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) AS City
FROM Nashville_Housing


ALTER TABLE Nashville_Housing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE Nashville_Housing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

ALTER TABLE Nashville_Housing
ADD PropertySplitCity NVARCHAR(255)

UPDATE Nashville_Housing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))


-- Breaking out Address into individual columns (Address, City, State) - OwnerAddress Column

SELECT OwnerAddress
FROM Nashville_Housing

SELECT PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)
, PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)
, PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM Nashville_Housing


ALTER TABLE Nashville_Housing
ADD OwnerSplitAddress NVARCHAR(255);

UPDATE Nashville_Housing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

ALTER TABLE Nashville_Housing
ADD OwnerSplitCity NVARCHAR(255)

UPDATE Nashville_Housing 
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

ALTER TABLE Nashville_Housing
ADD OwnerSplitState NVARCHAR(255)

UPDATE Nashville_Housing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)

SELECT *
FROM Nashville_Housing
```

![1446-02-17 19_47_34-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/60f1f835-fbb1-45ad-90db-bcbf2d0da806)


## Change Y and N to Yes and No in "Sold as Vacant" field
---

```SQL
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM Nashville_Housing
GROUP BY SoldAsVacant

SELECT SoldAsVacant
, CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		WHEN SoldAsVacant = 'N' THEN 'No'
		ELSE SoldAsVacant
		END
FROM Nashville_Housing

UPDATE Nashville_Housing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		WHEN SoldAsVacant = 'N' THEN 'No'
		ELSE SoldAsVacant
		END
```

![1446-02-17 19_49_31-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/47df103b-3f41-42a1-b3aa-3489feb1a1de)


## Remove Duplicates
---

```SQL
WITH RowNumCTE AS (
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY  ParcelID, 
				PropertyAddress,
				SalePrice,
				SaleDate,
				LegalReference
				ORDER BY UniqueID
				) row_num
FROM Nashville_Housing
--ORDER BY ParcelID
)
--DELETE
SELECT *
FROM RowNumCTE
WHERE row_num > 1
--ORDER BY PropertyAddress

SELECT *
FROM Nashville_Housing
```

![1446-02-17 19_51_18-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/b7110534-0cc4-4f45-ac51-3eee0eb0cdb7)
Output is empty because there are no longer duplicate rows in our dataset

## Delete Unused Columns
---

```SQL
SELECT *
FROM Nashville_Housing

ALTER TABLE Nashville_Housing
DROP COLUMN OwnerAddress, PropertyAddress, SaleDate
```

![1446-02-17 19_52_42-Nashville_Housing_Data_Cleaning_Project sql](https://github.com/user-attachments/assets/e9351178-ebbf-43d0-bcb4-b5d28b9a0717)
