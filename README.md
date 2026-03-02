# Nashville-Housing-Data 🏘️

## 💡Cleaning Data set

## Table of Contents
1. [Standardize Date Format](https://github.com/MichaelArellanox/Nashville-Housing-Data#1-standardize-date-format)
2. [Populate Property Address](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#2-populate-property-address)
3. [Separate Address into Individual Columns](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#3-separating-addresses-into-individual-columns-address-city-state)
4. 

### 1. Standardize Date Format

````sql
SELECT SaleDate, CONVERT(Date, SaleDate)
FROM dbo.NashvilleHousing;
````

**Result:**

<img width="394" height="171" alt="image" src="https://github.com/user-attachments/assets/26bc8d93-9898-470e-bf01-a75bbc2152b0" />

I want to change the SaleDate field to only show the date and not the time stamp like the column on the right.

````sql
ALTER TABLE NashvilleHousing
ADD SaleDateNew Date;<img width="1821" height="276" alt="image" src="https://github.com/user-attachments/assets/adae0628-e1de-4dd8-b503-89520e3560fc" />

UPDATE NashvilleHousing
SET SaleDateNew = CONVERT(Date, SaleDate);

SELECT *
FROM NashvilleHousing
````

**Result:**

<img width="1821" height="276" alt="image" src="https://github.com/user-attachments/assets/c5f4ec2e-6a44-49aa-abac-27cf1bcfd158" />

**SaleDateNew** column added and we will delete SaleDate column later on.

### 2. Populate Property Address

````sql
-- Checking for NULL values
SELECT COUNT(*) PropAddressNull
FROM NashvilleHousing
WHERE PropertyAddress is NULL;
````

**Result:**

<img width="181" height="54" alt="image" src="https://github.com/user-attachments/assets/3162ec5f-9280-42c7-bcd6-90d59718dc8b" />

29 NULL values that we want to populate.

````sql
SELECT *
FROM NashvilleHousing
ORDER BY ParcelID;
````

**Result:**

<img width="1827" height="256" alt="image" src="https://github.com/user-attachments/assets/1b23603a-6be6-44f6-9d3a-69708ed163e6" />

Take a look at the ParcelID column in row 44 and 45. You can see that the property address is the same for ParcelIDs that are the same. So it is safe to assume if a ParcelID has an property address and then later on the same ParcelID has a null value in the property address field, so I can populate it with the address in the first instance.

````sql
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress
FROM NashvilleHousing a
JOIN NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress is NULL;
````

**Result:**

<img width="784" height="273" alt="image" src="https://github.com/user-attachments/assets/6d153f19-d741-4255-8ca6-b533c8d73a5a" />


This query joins the table with itself based on ParcelIDs. I know that the same ParcelID can have the same property address so I need a unique factor to differentiate these ParcelIDs, that is where uniqueID comes in. Two uniqueIDs can have the same ParcelID and one of their property address can be NULL while the other one has an address. That is how I will populate the NULL values. 


````sql
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress is NULL;
````

**Result:**

<img width="1098" height="299" alt="image" src="https://github.com/user-attachments/assets/33bf4c85-f598-42b2-a498-7afda8e49a10" />

I used `ISNULL()` which will check values in `a.PropertyAddress` and if they are NULL it will replace it with the values in `b.PropertyAddress`. As you can see above the new column that was added will be what with fill our `PropertyAddress` field with which is our next step.

````sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress is NULL;

-- if we run th query again from the step above there should be no NULL Values
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress
FROM NashvilleHousing a
JOIN NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress is NULL;
````

**Result:**

<img width="480" height="170" alt="image" src="https://github.com/user-attachments/assets/5a38abd2-5c61-409a-9026-957c3ace354d" />

### 3. Separating Addresses into Individual Columns (Address, City, State)
### First method (`SUBSTRING` and `CHARINDEX`)

Looking at the `PropertyAddress` column in the previous images above you can see that it has an address and city separated by a comma. My goal is to separate the two so the are in individual columns. 

````sql
-- Separating address from city
SELECT
-- starting at the first position and ending at the comma separating address and city
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) as Address,
-- starting at the position after the comma and ending at the end position
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) as City
FROM NashvilleHousing;
````

**Result:**

<img width="358" height="274" alt="image" src="https://github.com/user-attachments/assets/7167818f-a3b1-424f-ac4e-0674d8d62d7c" />

This method focuses on the positioning of the string and separates the address and city by starting at the beginning of the string and ending before the comma for address and then starting after the comma and ending at the end of the string for city. Now the next step is to alter and update our table to add these two new columns.

````sql
-- create two new columns into our copied table to insert our new separated data
ALTER TABLE dbo.NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1);

ALTER TABLE dbo.NashvilleHousing
ADD PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));

-- check if table updated
SELECT *
FROM NashvilleHousing;
````

**Result:**

<img width="1831" height="276" alt="image" src="https://github.com/user-attachments/assets/1e729d44-91b0-48d3-bbf2-2b1cfb185224" />

As you can see when we look at our updated table the two new columns with the data has been added on the far right. 

### Second Method (`PARSENAME`)


