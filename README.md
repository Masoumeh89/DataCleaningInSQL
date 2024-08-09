# DataCleaningInSQL
Data Cleaning Project on Nashville Housing Data


## Table of Contents

1. [Introduction](#introduction)
2. [Data Source](#data-source)
3. [Standardizing Date Format](#standardizing-date-format)
4. [Populating Missing Property Address Data](#populating-missing-property-address-data)
5. [Breaking Out Address into Individual Columns](#breaking-out-address-into-individual-columns)
6. [Correcting Categorical Values](#correcting-categorical-values)
7. [Removing Duplicates](#removing-duplicates)
8. [Deleting Unused Columns](#deleting-unused-columns)
9. [Conclusion](#conclusion)




## Introduction

In this project, the primary objective was to clean and prepare the Nashville Housing Data for analysis. Data cleaning is a crucial step in the data science process, ensuring that the dataset is accurate, consistent, and ready for downstream tasks such as analysis and modeling. The dataset used in this project contained various fields related to property sales in Nashville, including addresses, sale dates, prices, and ownership information. The cleaning process involved standardizing formats, populating missing data, breaking out address information into separate columns, correcting categorical values, removing duplicates, and deleting unnecessary columns.

## Data Source 

Data Link:

https://www.kaggle.com/datasets/tmthyjames/nashville-housing-data



# Data Cleaning Process

## Standardizing Date Format

The dataset's SaleDate field contained dates in different formats, which could lead to inconsistencies in analysis. The date format was standardized to a uniform YYYY-MM-DD format using the CONVERT function in SQL.


```sql

UPDATE NashvilleHousing
SET SaleDate = CONVERT(Date, SaleDate);

```

## Populating Missing Property Address Data

Some records in the dataset were missing property addresses. To populate these missing values, a self-join was performed on the ParcelID, matching records with the same ParcelID but different UniqueID. The missing PropertyAddress values were then filled in with the corresponding values from the matched records.


```sql

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
ON a.ParcelID = b.ParcelID AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL;

```


## Breaking Out Address into Individual Columns

The PropertyAddress and OwnerAddress fields contained both address and city information in a single string. These fields were split into separate columns (Address, City, State) using the SUBSTRING, CHARINDEX, and PARSENAME functions.

```sql

ALTER TABLE NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255), PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1),
    PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));

```


The same process was applied to the OwnerAddress field.

## Correcting Categorical Values

The SoldAsVacant field used 'Y' and 'N' to indicate whether a property was sold as vacant. To make the data more readable, these values were converted to 'Yes' and 'No'.

```sql

UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
                    WHEN SoldAsVacant = 'Y' THEN 'Yes'
                    WHEN SoldAsVacant = 'N' THEN 'No'
                    ELSE SoldAsVacant
                    END;

```



## Removing Duplicates

Duplicate records can skew analysis results, so it was essential to remove them. A Common Table Expression (CTE) was used to identify and remove duplicates based on key fields such as ParcelID, PropertyAddress, SalePrice, SaleDate, and LegalReference.

```sql

WITH RowNumCTE AS (
SELECT *,
    ROW_NUMBER() OVER (
    PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference 
    ORDER BY UniqueID
    ) row_num
FROM NashvilleHousing
)
DELETE FROM NashvilleHousing
WHERE UniqueID IN (
    SELECT UniqueID
    FROM RowNumCTE
    WHERE row_num > 1
);

```



## Deleting Unused Columns

After the data cleaning operations, some columns were no longer needed, such as OwnerAddress, TaxDistrict, and the original PropertyAddress. These columns were removed to streamline the dataset.

```sql

ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;

```



## Conclusion
The data cleaning process significantly improved the quality and usability of the Nashville Housing Data. By standardizing date formats, filling in missing values, breaking out addresses into individual components, correcting categorical values, removing duplicates, and deleting unnecessary columns, the dataset was transformed into a well-structured and clean format. This cleaned dataset is now ready for further analysis, allowing for more accurate insights and decision-making. The steps and methods used in this project are applicable to a wide range of datasets, showcasing the importance and impact of thorough data cleaning in any data science project.



