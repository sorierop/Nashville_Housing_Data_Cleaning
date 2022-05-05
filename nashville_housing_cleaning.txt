-- Select table data --

SELECT *
FROM PortfolioProject.dbo.nashville_housing_data

-----------------------------------------------------

-- Populate Property Address NULL Values

-- ParcelIDs that match will be populated with the same PropertyAddress values 

SELECT *
FROM PortfolioProject.dbo.nashville_housing_data
--WHERE PropertyAddress is null
ORDER BY ParcelID

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM PortfolioProject.dbo.nashville_housing_data a
JOIN PortfolioProject.dbo.nashville_housing_data b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress is null

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM PortfolioProject.dbo.nashville_housing_data a
JOIN PortfolioProject.dbo.nashville_housing_data b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress is null

----------------------------------------------------------------------------------

-- Breaking out Addresses into Individual Columns (Address, City, State)

SELECT PropertyAddress
FROM PortfolioProject.dbo.nashville_housing_data
--WHERE PropertyAddress is null
--ORDER BY ParcelID

SELECT
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) AS Address
, SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) AS City


FROM PortfolioProject.dbo.nashville_housing_data

ALTER TABLE nashville_housing_data
ADD Property_Split_Address NVARCHAR(255);

UPDATE nashville_housing_data
SET Property_Split_Address = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) 

ALTER TABLE nashville_housing_data
ADD Property_Split_City NVARCHAR(255);

UPDATE nashville_housing_data
SET Property_Split_City = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) 


-- Owner Address

SELECT *
FROM PortfolioProject.dbo.nashville_housing_data

SELECT OwnerAddress
FROM PortfolioProject.dbo.nashville_housing_data

SELECT
PARSENAME(REPLACE(OwnerAddress,',', '.') ,3),
PARSENAME(REPLACE(OwnerAddress,',', '.') ,2),
PARSENAME(REPLACE(OwnerAddress,',', '.') ,1)
FROM PortfolioProject.dbo.nashville_housing_data


-- Table Alters

ALTER TABLE nashville_housing_data
ADD Owner_Split_Address NVARCHAR(255);

UPDATE nashville_housing_data
SET Owner_Split_Address = PARSENAME(REPLACE(OwnerAddress,',', '.') ,3) 

ALTER TABLE nashville_housing_data
ADD Owner_Split_City NVARCHAR(255);

UPDATE nashville_housing_data
SET Owner_Split_City = PARSENAME(REPLACE(OwnerAddress,',', '.') ,2)

ALTER TABLE nashville_housing_data
ADD Owner_Split_State NVARCHAR(255);

UPDATE nashville_housing_data
SET Owner_Split_State = PARSENAME(REPLACE(OwnerAddress,',', '.') ,1) 



----------------------------------------------------------------------------------


-- Change 1 and 0 to Yes and No in SoldAsVacant


SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM PortfolioProject.dbo.nashville_housing_data
GROUP BY SoldAsVacant
ORDER BY 2


SELECT SoldAsVacant
,	CASE WHEN SoldAsVacant = '1' THEN 'Yes'
		 WHEN SoldAsVacant = '0' THEN 'No'
		 END
FROM PortfolioProject.dbo.nashville_housing_data


ALTER TABLE nashville_housing_data
ADD SoldAsVacant_YN NVARCHAR(255);

UPDATE nashville_housing_data
	SET SoldAsVacant_YN = CASE WHEN SoldAsVacant = '1' THEN 'Yes'
							   WHEN SoldAsVacant = '0' THEN 'No'
						  END

--------------------------------------------------------------------------

-- Removing Duplicates


WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
	ORDER BY UniqueID) row_num



FROM PortfolioProject.dbo.nashville_housing_data
--ORDER BY ParcelID
)

SELECT *
FROM RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress


SELECT *
FROM PortfolioProject.dbo.nashville_housing_data

------------------------------------------------------------------

--Remove Unused Columns 

SELECT *
FROM PortfolioProject.dbo.nashville_housing_data

ALTER TABLE PortfolioProject.dbo.nashville_housing_data
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress

ALTER TABLE PortfolioProject.dbo.nashville_housing_data
DROP COLUMN SoldAsVacant

---------------------------------------------------------------------

-- Finished Table


SELECT *
FROM PortfolioProject.dbo.nashville_housing_data