#  Nashville Housing Data Cleaning Project (PostgreSQL)

This project demonstrates step-by-step data cleaning on a raw Nashville Housing dataset using PostgreSQL. It includes handling null values, splitting columns, fixing inconsistent values, removing duplicates, and dropping unnecessary columns.

##  Dataset
The dataset was downloaded as a `.xlsx` file converted into .csv:   
`Nashville Housing Data.xlsx.csv`

##  Table Creation

```sql
CREATE TABLE Nashville_housing (
  UniqueID numeric,
  ParcelID varchar(255),
  LandUse varchar(255),
  PropertyAddress varchar(255),
  SaleDate date,
  SalePrice text,
  LegalReference varchar(255),
  SoldAsVacant char(50),
  OwnerName varchar(255),
  OwnerAddress varchar(255),
  Acreage float,
  TaxDistrict varchar(255),
  LandValue numeric,
  BuildingValue numeric,
  TotalValue numeric,
  YearBuilt int,
  Bedrooms int,
  FullBath int,
  HalfBath int
);
```
--------------------------------------------------------------------------------------------------------------------------------
##  LOAD DATA

```sql
COPY Nashville_housing
FROM 'C:\Users\sneha\Downloads\Nashville Housing Data.xlsx.csv'DELIMITER ','CSV HEADER;
```
-----------------------------------------------------------------------------------------------------------------------------------

## VIEW DATA
```sql
SELECT * FROM Nashville_housing;
```
-------------------------------------------------------------------------------------------------------------------------------


## FILL NULL VALUES IN PROPERTY ADDRESS USING SELF JOIN

```sql
select nh.parcelid, nh.propertyaddress, nash.parcelid, nash.propertyaddress,
COALESCE(nh.propertyaddress, nash.propertyaddress) AS New_propertyaddress
from Nashville_housing nh
join Nashville_housing nash
  on nh.parcelid = nash.parcelid
 and nh.uniqueid != nash.uniqueid
where nh.propertyaddress IS NULL;

update Nashville_housing AS nh
set propertyaddress = COALESCE(nh.propertyaddress, nash.propertyaddress)
from Nashville_housing nash
where nh.parcelid = nash.parcelid
  and nh.uniqueid != nash.uniqueid
  and nh.propertyaddress IS NULL;
```

--------------------------------------------------------------------------------------------------------------------------

## SPLIT OWNER ADDRESS INTO STREET, CITY AND STATE

```sql
SELECT 
  owneraddress,
  SPLIT_PART(owneraddress, ',', 1) AS owner_street_address,
  TRIM(SPLIT_PART(owneraddress, ',', 2)) AS owner_city,
  TRIM(SPLIT_PART(owneraddress, ',', 3)) AS owner_state
FROM Nashville_Housing;

ALTER TABLE Nashville_Housing 
add column owner_street_address varchar(255),
add column owner_city varchar(255),
add column  owner_state varchar(255);

UPDATE Nashville_Housing
SET
  owner_street_address = SPLIT_PART(owneraddress, ',', 1),
  owner_city = TRIM(SPLIT_PART(owneraddress, ',', 2)),
  owner_state = TRIM(SPLIT_PART(owneraddress, ',', 3));
```

------------------------------------------------------------------------------------------------------------

## SPLIT PROPERTY ADDRESS INTO STREET AND CITY

```sql
SELECT 
  propertyaddress,
  SPLIT_PART(propertyaddress, ',', 1) AS street_address,
  TRIM(SPLIT_PART(propertyaddress, ',', 2)) AS city
FROM Nashville_Housing;

ALTER TABLE Nashville_Housing 
add column  street_address varchar(255),
add column  city varchar(255);

UPDATE Nashville_Housing
SET
  street_address = SPLIT_PART(propertyaddress, ',', 1),
  city = TRIM(SPLIT_PART(propertyaddress, ',', 2));

ALTER TABLE Nashville_housing 
RENAME COLUMN street_address TO Property_Street_Address;

ALTER TABLE Nashville_housing 
RENAME COLUMN city TO Property_City;
```

--------------------------------------------------------------------------------------------------------

## STANDARDIZE SOLDASVACANT VALUES

```sql
UPDATE Nashville_housing
SET soldasvacant =
  CASE
    WHEN soldasvacant = 'Y' THEN 'Yes'
    WHEN soldasvacant = 'N' THEN 'No'
    ELSE soldasvacant
  END;
```

------------------------------------------------------------------------------------------------

## REMOVE DUPLICATES

```sql
-- Convert SaleDate to date type if needed
ALTER TABLE Nashville_housing 
ALTER COLUMN SaleDate TYPE date;
```
-----------------------------------------------------------------------------------------------------------------

## Identify duplicates

```sql
WITH row_num_cte AS (
  SELECT *,
         ROW_NUMBER() OVER (
           PARTITION BY parcelid, propertyaddress, saledate, saleprice, legalreference
           ORDER BY uniqueid
         ) AS row_num
  FROM Nashville_housing
)
SELECT *
FROM row_num_cte
WHERE row_num > 1;
```
-------------------------------------------------------------------------------------------------------------------

## Delete Duplicates

```sql
WITH row_num_cte AS (
  SELECT uniqueid,
         ROW_NUMBER() OVER (
           PARTITION BY parcelid, propertyaddress, saledate, saleprice, legalreference
           ORDER BY uniqueid
         ) AS row_num
  FROM Nashville_housing
)
DELETE FROM Nashville_housing
WHERE uniqueid IN (
  SELECT uniqueid
  FROM row_num_cte
  WHERE row_num > 1
);
```

----------------------------------------------------------------------------------------------------------

## DROP UNUSED COLUMNS

```sql
ALTER TABLE Nashville_housing
DROP COLUMN propertyaddress;

ALTER TABLE Nashville_housing
DROP COLUMN owneraddress;

ALTER TABLE Nashville_housing
DROP COLUMN taxdistrict;

ALTER TABLE Nashville_housing
DROP COLUMN saledate;
```

------------------------------------------------------------------------------------------------------

## FINAL CHECK

```sql
SELECT * FROM Nashville_housing;
```

--------------------------------------------------------------------------------------------------
