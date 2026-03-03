# Nashville-Housing-Data 🏘️

## 💡Cleaning Data set

### Table of Contents 📖
1. [Standardize Date Format](https://github.com/MichaelArellanox/Nashville-Housing-Data#1-standardize-date-format)
2. [Populate Property Address](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#2-populate-property-address)
3. [Separate Address into Individual Columns](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#3-separating-addresses-into-individual-columns-address-city-state)
4. [Change Y and N to Yes and No in SoldAsVacant Field](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#4-change-y-and-n-to-yes-and-no-in-soldasvacant-field)
5. [Remove Duplicates](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#5-remove-duplicates)
6. [Delete Unnecessary Columns](https://github.com/MichaelArellanox/Nashville-Housing-Data?tab=readme-ov-file#6-delete-unnecessary-columns)

<br>
   
### 1. Standardize Date Format 📅

````sql
SELECT SaleDate, CONVERT(Date, SaleDate)
FROM dbo.NashvilleHousing;
````

**Result:**

<img width="394" height="171" alt="image" src="https://github.com/user-attachments/assets/26bc8d93-9898-470e-bf01-a75bbc2152b0" />

I want to change the `SaleDate` field to only show the date and not the time stamp like the column on the right.

<br>

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

`SaleDateNew` column added and I will delete `SaleDate` column later on.

<br>

### 2. Populate Property Address 📇

````sql
-- Checking for NULL values
SELECT COUNT(*) PropAddressNull
FROM NashvilleHousing
WHERE PropertyAddress is NULL;
````

**Result:**

<img width="181" height="54" alt="image" src="https://github.com/user-attachments/assets/3162ec5f-9280-42c7-bcd6-90d59718dc8b" />

29 NULL values that we want to populate.

<br>

````sql
SELECT *
FROM NashvilleHousing
ORDER BY ParcelID;
````

**Result:**

<img width="1827" height="256" alt="image" src="https://github.com/user-attachments/assets/1b23603a-6be6-44f6-9d3a-69708ed163e6" />

I noticed the `ParcelID` column in row 44 and 45. I saw that `PropertyAddress` is the same for Parcel IDs that are the same. So it is safe to assume if a Parcel ID has a property address and then later on the same Parcel ID has a null value in the property address field, so I can populate it with the address in the first instance.

<br>

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


This query joins the table with itself based on `ParcelID`. I know that the same Parcel ID can have the same property address so I need a unique factor to differentiate these Parcel IDs, that is where `uniqueID` comes in. Two unique IDs can have the same ParcelID and one of their property address can be NULL while the other one has an address. That is how I will populate the `NULL` values. 

<br>

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

I used `ISNULL()` which will check values in `a.PropertyAddress` and if they are `NULL` it will replace it with the values in `b.PropertyAddress`. As you can see above the new column that was added will be what with fill our `PropertyAddress` field with which is my next step.

<br>

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

<br>
<br>
<br>

### 3. Separating Addresses into Individual Columns (Address, City, State) 🗺️
### First method (`SUBSTRING` and `CHARINDEX`)

Looking at the `PropertyAddress` column in the previous images above I can see that it has an address and city separated by a comma. My goal is to separate the two so the are in individual columns. 

<br>

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

This method focuses on the positioning of the string and separates the address and city by starting at the beginning of the string and ending before the comma for address and then starting after the comma and ending at the end of the string for city. Now the next step is to alter and update the table to add these two new columns.

<br>

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

Now when I look at the updated table the two new columns with the data has been added on the far right. 

<br>

### Second Method (`PARSENAME`)

We will take a look at the `OwnerAddress` column now and see what we need to separate.


````sql
SELECT OwnerAddress
FROM NashvilleHousing;
````

**Result:**

<img width="469" height="300" alt="image" src="https://github.com/user-attachments/assets/2c1e7cdc-702d-4673-9e25-c4a737647fb2" />

In this field the address, city, and state need to be separated into individual columns and I will use `PARSENAME` to acheive this. Although, this method is only useful when data is separated with periods so I had to work around that.

<br>

````sql
SELECT
-- onyl useful with periods
-- parsename does everything backwards so it starts from city state then address so in parsename we will go backwards
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM NashvilleHousing;
````

**Result:**

<img width="602" height="374" alt="image" src="https://github.com/user-attachments/assets/d56a708b-e372-46d4-9074-dad4ea7501aa" />

Using `PARSENAME` I have successfully separated the address, city, and state into individual columns and will now use this to update the table.

<br>

````sql
-- update table with new separated data
ALTER TABLE NashvilleHousing
ADD OwnerSplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3);

ALTER TABLE NashvilleHousing
ADD OwnerSplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2);

ALTER TABLE NashvilleHousing
ADD OwnerSplitState NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);

-- check if table updated 
SELECT *
FROM NashvilleHousing;
````

**Result:**

<img width="1820" height="345" alt="image" src="https://github.com/user-attachments/assets/1bea4555-1724-4520-9cf5-8eaf7b81a768" />

The table has been updated with the three new columns shown to the far right.

<br>

### 4. Change Y and N to Yes and No in `SoldAsVacant` field ☑️

First I want to see how many of each are in this field. By looking through the data there is a mix of Y, N, Yes, and No.

````sql
-- Checking how many Y's, N's, Yes, and No's there are
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2;
````

**Result:**

<img width="320" height="131" alt="image" src="https://github.com/user-attachments/assets/33373c07-6b4d-4733-80ea-58e93218a894" />

The 52 Y's and 399 N's will be changed by using case statements.

<br>

````sql
-- Want to change the Y's and N's to Yes and No
SELECT SoldAsVacant,
CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
     WHEN SoldAsVacant = 'N' THEN 'No'
     ELSE SoldAsVacant
     END
FROM NashvilleHousing;
````

**Result:**

<img width="331" height="323" alt="image" src="https://github.com/user-attachments/assets/f642de0f-802b-4cfd-9c05-d6b49af41ea7" />

The query was successful and multiple rows where the data was N is now No. The next step is to just update our table.

<br>

````sql
-- Update our table
UPDATE NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
     WHEN SoldAsVacant = 'N' THEN 'No'
     ELSE SoldAsVacant
     END
FROM NashvilleHousing
````
If I rerun my previous query then only Yes and No should appear.

<br>

**Results:**

<img width="324" height="85" alt="image" src="https://github.com/user-attachments/assets/22c51905-3847-430d-8881-db1c507461e1" />

No Y's or N's show up so our update was successful.

<br>

### 5. Remove Duplicates 2️⃣

````sql
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num

From NashvilleHousing;
````

**Result:**

<img width="1826" height="354" alt="image" src="https://github.com/user-attachments/assets/4fab75d1-4e55-4bd1-a9f6-4cea0e716dff" />

Using `ROW_NUMBER` and `PARTITION BY` we assigned row numbers which can be seen in the new `row_num` column to the far right. If the row number is greater than 1 then that means it is a duplicate. 

<br>

````sql
WITH Row_NumCTE AS(
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
						) row_num

FROM NashvilleHousing
)

-- figuring out how many duplicates there are, if a row number is greater than 1 than it is a duplicate
SELECT *
FROM Row_NumCTE
WHERE row_num > 1
ORDER BY PropertyAddress;
````

**Result:**

<img width="1855" height="409" alt="image" src="https://github.com/user-attachments/assets/32f7e9fb-247b-46c6-b10f-defc5cf7f95a" />

I created a CTE and selected all the rows that have a row number greated than 1 (duplicates). I can see on the bottom right corner there are 104 duplicates and my next goal is to delete them.

<br>

````sql
WITH Row_NumCTE AS(
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
						) row_num

FROM NashvilleHousing
)

-- delete duplicates
DELETE
FROM Row_NumCTE 
WHERE row_num > 1;
````

Since I created a CTE I have to execute these two at the same time for it to be successful. If I rerun the previous `SELECT` statement that collects duplicate rows, no rows should come up now.

<br>

**Result:**

<img width="1846" height="284" alt="image" src="https://github.com/user-attachments/assets/6082a9d7-aa20-4374-84ed-cbadc2809279" />

I have deleted the duplicates from the table.

<br>

<br>

### 6. Delete Unnecessary Columns ✖️

````sql
SELECT *
FROM NashvilleHousing;
````

**Result:**

<img width="1816" height="353" alt="image" src="https://github.com/user-attachments/assets/f7eec413-f8cc-4a30-81a7-9a9ef13fba09" />

Looking at my data set and looking back at what I have completed so far, some unnecessary columns that I can delete are `PropertyAddress`, `OwnerAddress`, and `SaleDate` since these are the ones I have updated. I also looked at other columns and thought that `TaxDistrict` was not important so decided mark that as unnecessary.

<br>

````sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, PropertyAddress, TaxDistrict, SaleDate;
````
If we rerun the previous `SELECT` statement then the columns should be gone.

<br>

**Completed table after cleaning:**

<img width="1838" height="372" alt="image" src="https://github.com/user-attachments/assets/d03c8488-308d-4ee8-9a76-091c96e060a9" />

<img width="1837" height="352" alt="image" src="https://github.com/user-attachments/assets/9cc61d28-6fb4-440e-a49e-6ca7a427ffc4" />

