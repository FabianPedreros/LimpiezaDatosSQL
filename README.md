# LimpiezaDatosSQL
Limpieza de datos usando SQL.

Realizar un proceso de limpieza a un set de datos utilizando la herramienta SQL Server. Efectuando distintas tareas como visualizaciones de datos, correcciones de valores, actualizaciones de tablas, eliminación de espacios innecesarios, etc.

Este es un proyecto que hace parte de la lista de reproducción de proyectos para el desarrollo de un portafolio de Alex The Analyst. 

En los archivos podrán encontrar:

1. Nashville Housing Data for Data Cleaning.xlsx --> Archivo de excel con los datos a limpiar.
2. Limpieza de datos en SQL.pptx --> Presentación en la que de manera resumida se muestran los pasos ejecutados para la limpieza en SQL.
3. LimpiezaDeDatos.sql --> Archivo en el que se encuentran las consultas ejecutadas.
4. Cheat Sheet SQL Data Preparation.jpg --> Imagen que contiene algunas de las consultas más utilizadas para la preparación de datos en SQL.


Link a la lista de reproducción de Alex. https://www.youtube.com/watch?v=8rO7ztF4NtU&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f&index=3&ab_channel=AlexTheAnalyst


------- Limpieza de datos en SQL -------

/* Consulta para primera visualización de datos */

SELECT TOP 1000 *
  FROM Nashville..Houses;

  -- Estandarización de fechas (SaleDate)--
  -- Eliminación de  los valores de horas, ya que solo contiene información de fecha

  -- Visualización de datos SaleDate

  SELECT SaleDate
  FROM Nashville..Houses;

  -- Visualización de modificación del tipo de dato a Fecha

  SELECT SaleDate , CONVERT(date, SaleDate)
  FROM Nashville..Houses;

  -- Actualización del campo SaleDate a el formato date
  
  UPDATE Nashville..Houses
  SET SaleDate = CONVERT(date, SaleDate);

  -- Para el caso que no permita la actualización, se puede crear una nueva columna y eliminar la otra

  ALTER TABLE Nashville..Houses
  ADD SaleDateConverted Date;

  UPDATE Nashville..Houses
  SET SaleDateConverted = CONVERT(date, SaleDate);

  ALTER TABLE Nashville..Houses
  DROP COLUMN SaleDate;

  -- Revisión y conteo de valores nulos para SaleDate

  SELECT SaleDateConverted
  FROM Nashville..Houses
  WHERE SaleDateConverted IS NULL;

-- Transformación de datos para el campo PropertyAddress
-- Visualización de datos para el campo PropertyAddress

SELECT PropertyAddress
FROM Nashville..Houses;

-- Revisión y conteo de valores nulos para SaleDate

SELECT PropertyAddress
FROM Nashville..Houses
WHERE PropertyAddress IS NULL;

SELECT COUNT(*)
FROM Nashville..Houses
WHERE PropertyAddress IS NULL;

-- Revisión de valores nulos en PropertyAddress
SELECT *
FROM Nashville..Houses
WHERE PropertyAddress IS NULL;


-- Consulta de registros con más de un ParcelID
SELECT COUNT(PropertyAddress), ParcelID, PropertyAddress
FROM Nashville..Houses
GROUP BY ParcelID, PropertyAddress
HAVING COUNT (PropertyAddress) > 1
ORDER BY ParcelID

-- Consulta de ParcelID y PropertyAddress de la consulta anterior
SELECT ParcelID, PropertyAddress
FROM Nashville..Houses
WHERE ParcelID IN 
	(SELECT ParcelID
	FROM Nashville..Houses
	GROUP BY ParcelID, PropertyAddress
	HAVING COUNT (PropertyAddress) > 1)
ORDER BY ParcelID

-- Consulta de los ParcelID y PropertyAddress para la asignacióna los nulos, join a la misma tabla

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashville..Houses AS a
JOIN Nashville..Houses AS b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ]<> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL;

-- Actualización de las direcciones de los ParcelID con direcciones nulas

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashville..Houses AS a
JOIN Nashville..Houses AS b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ]<> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL;
   


-- División en columnas de los datos en PropertyAddress

-- Consulta de los datos en las direcciones

SELECT DISTINCT(PropertyAddress)
FROM Nashville..Houses;


-- Consulta para la división de dirección, valor antes de la coma

SELECT
SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) as Direccion
FROM Nashville..Houses;


-- Consulta para la división de ciudad, valor despúes de la coma
SELECT
SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress)) as Ciudad
FROM Nashville..Houses;

-- Agregar la columna y valores para dirección

ALTER TABLE Nashville..Houses
ADD PropertyDireccion NVARCHAR(255)

UPDATE Nashville..Houses
SET PropertyDireccion = (SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1));

-- Agregar la columna y valores para ciudad

ALTER TABLE Nashville..Houses
ADD PropertyCiudad NVARCHAR(255)

UPDATE Nashville..Houses
SET PropertyCiudad = SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress));


SELECT TOP 100 *
FROM Nashville..Houses;



-- División en columnas de los datos en OwnerAddress

-- Consulta de los datos en las direcciones

SELECT DISTINCT(OwnerAddress)
FROM Nashville..Houses;

-- Consulta para la división de los valores en OwnerAddress

SELECT  PARSENAME(REPLACE(OwnerAddress, ',', '.'),3),
		PARSENAME(REPLACE(OwnerAddress, ',', '.'),2),
		PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)
FROM Nashville..Houses;


-- Agregar la columna y valores para la dirección del dueño

ALTER TABLE Nashville..Houses
ADD OwnerDireccion NVARCHAR(255)

UPDATE Nashville..Houses
SET OwnerDireccion = PARSENAME(REPLACE(OwnerAddress, ',', '.'),3);


-- Agregar la columna y valores para la ciudad del dueño

ALTER TABLE Nashville..Houses
ADD OwnerCiudad NVARCHAR(255)

UPDATE Nashville..Houses
SET OwnerCiudad = PARSENAME(REPLACE(OwnerAddress, ',', '.'),2);


-- Agregar la columna y valores para el estado del dueño

ALTER TABLE Nashville..Houses
ADD OwnerEstado NVARCHAR(255)

UPDATE Nashville..Houses
SET OwnerEstado = PARSENAME(REPLACE(OwnerAddress, ',', '.'),1);


--- Revisión de valores en SoldAsVacant --
--Consulta de valores y conteo de frecuencias--

SELECT DISTINCT (SoldAsVacant), COUNT (SoldAsVacant) AS Frecuencia
FROM Nashville..Houses
GROUP BY SoldAsVacant
ORDER BY Frecuencia;

-- Consulta de valores permitidos en SoldAsVacant

SELECT SoldAsVacant, 
CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	 WHEN SoldAsVacant = 'N' THEN 'No'
	 ELSE SoldAsVacant
	 END
FROM Nashville..Houses

-- Actualización de valores permitidos en SoldAsVacant

UPDATE Nashville..Houses
SET SoldAsVacant = (CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	 WHEN SoldAsVacant = 'N' THEN 'No'
	 ELSE SoldAsVacant
	 END)


-- Eliminación de duplicados --

-- Consulta de registros duplicados según las columnas:
-- ParcelId, PropertyAddres, SalePrice, SaleDate, LegalReference

WITH TablaTem AS(
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelId,
				 PropertyAddress,
				 SalePrice,
				 SaleDateConverted,
				 LegalReference
				 ORDER BY
					UniqueID
					) AS DUPLICADO
FROM Nashville..Houses
)
SELECT *
FROM TablaTem
WHERE DUPLICADO >1


-- Eliminación de duplicados

WITH TablaTem AS(
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelId,
				 PropertyAddress,
				 SalePrice,
				 SaleDateConverted,
				 LegalReference
				 ORDER BY
					UniqueID
					) AS DUPLICADO
FROM Nashville..Houses
)
DELETE
FROM TablaTem
WHERE DUPLICADO >1

-- Eliminar columnas innecesarias --

ALTER TABLE Nashville..Houses
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress

-- Eliminar espacios innecesarios en los atributos con strings
-- Validación de atributo LandUse con valores con espacios adicionales 

SELECT LandUse, TRIM(REPLACE(LandUse,'  ','')), LEN(LandUse), LEN(TRIM(REPLACE(LandUse,'  ','')))
FROM Nashville..Houses
WHERE LEN(LandUse) <> LEN(TRIM(REPLACE(LandUse,'  ','')))

-- Actualización del valor CONDOMINIUM OFC  OR OTHER COM CONDO --> CONDOMINIUM OFC OR OTHER COM CONDO

UPDATE Nashville..Houses
SET LandUse = (CASE WHEN LandUse = 'CONDOMINIUM OFC  OR OTHER COM CONDO' THEN 'CONDOMINIUM OFC OR OTHER COM CONDO'
	 ELSE LandUse
	 END)

-- Validación de actualización

SELECT LandUse, COUNT(LandUse) AS Frecuencia
FROM Nashville..Houses
GROUP BY LandUse
ORDER BY Frecuencia


-- Validación de atributo UniqueId con valores con espacios adicionales, no tiene.

SELECT [UniqueID], TRIM(REPLACE([UniqueID],'  ','')), LEN([UniqueID]), LEN(TRIM(REPLACE([UniqueID],'  ','')))
FROM Nashville..Houses
WHERE LEN([UniqueID]) <> LEN(TRIM(REPLACE([UniqueID],'  ','')))

-- Validación de atributo ParcelID con valores con espacios adicionales, no tiene.

SELECT ParcelId, TRIM(REPLACE(ParcelId,'  ','')), LEN(ParcelId), LEN(TRIM(REPLACE(ParcelId,'  ','')))
FROM Nashville..Houses
WHERE LEN(ParcelId) <> LEN(TRIM(REPLACE(ParcelId,'  ','')))

-- Validación de atributo LegalReference con valores con espacios adicionales, no tiene.

SELECT LegalReference, TRIM(REPLACE(LegalReference,'  ','')), LEN(LegalReference), LEN(TRIM(REPLACE(LegalReference,'  ','')))
FROM Nashville..Houses
WHERE LEN(LegalReference) <> LEN(TRIM(REPLACE(LegalReference,'  ','')))

-- Validación de atributo OwnerName con valores con espacios adicionales, espacios dobles al interior de la cadena.

SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
FROM Nashville..Houses
WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


-- Actualización del valor en OwnerName eliminando espacios dobles
UPDATE Nashville..Houses
SET OwnerName = TRIM(REPLACE(OwnerName,'  ',' '));

-- Validación de la actualización

SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
FROM Nashville..Houses
WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


-- Validación de atributo PropertyDireccion con valores con espacios adicionales, 55788 valores líneas con espacios dobles, triples, al inicio o final.

SELECT PropertyDireccion, TRIM(REPLACE(PropertyDireccion,'  ','')), LEN(PropertyDireccion), LEN(TRIM(REPLACE(PropertyDireccion,'  ','')))
FROM Nashville..Houses
WHERE LEN(PropertyDireccion) <> LEN(TRIM(REPLACE(PropertyDireccion,'  ','')));

-- Actualización del valor en PropertyDireccion eliminando espacios triples
UPDATE Nashville..Houses
SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'   ',' '));

-- Actualización del valor en PropertyDireccion eliminando espacios dobles
UPDATE Nashville..Houses
SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'  ',' '));


-- Validación de atributo PropertyCiudad con valores con espacios adicionales, no hay.

SELECT PropertyCiudad, TRIM(REPLACE(PropertyCiudad,'  ','')), LEN(PropertyCiudad), LEN(TRIM(REPLACE(PropertyCiudad,'  ','')))
FROM Nashville..Houses
WHERE LEN(PropertyCiudad) <> LEN(TRIM(REPLACE(PropertyCiudad,'  ','')));

-- Validación de atributo OwnerDireccion con valores con espacios adicionales, 25273 valores líneas con espacios dobles, triples, al inicio o final.

SELECT OwnerDireccion, TRIM(REPLACE(OwnerDireccion,'  ','')), LEN(OwnerDireccion), LEN(TRIM(REPLACE(OwnerDireccion,'  ','')))
FROM Nashville..Houses
WHERE LEN(OwnerDireccion) <> LEN(TRIM(REPLACE(OwnerDireccion,'  ','')));

-- Actualización del valor en OwnerDireccion eliminando espacios triples
UPDATE Nashville..Houses
SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'   ',' '));

-- Actualización del valor en OwnerDireccion eliminando espacios dobles
UPDATE Nashville..Houses
SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'  ',' '));

-- Validación de atributo OwnerCiudad con valores con espacios adicionales, 25969 valores líneas con espacios dobles, triples, al inicio o final.

SELECT OwnerCiudad, TRIM(REPLACE(OwnerCiudad,'  ','')), LEN(OwnerCiudad), LEN(TRIM(REPLACE(OwnerCiudad,'  ','')))
FROM Nashville..Houses
WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

SELECT DISTINCT(OwnerCiudad)
FROM Nashville..Houses
WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

/*
Ciudades con error que deberían corregirse en la base de datos
 OLD HICKORY
 WHITES CREEK
 MOUNT JULIET
 JOELTON
 GOODLETTSVILLE
 ANTIOCH
 BELLEVUE
 MADISON
 NASHVILLE
 NOLENSVILLE
 HERMITAGE
 BRENTWOOD
 */
 
 -- Actualización del valor en OwnerCiudad eliminando espacios dobles
UPDATE Nashville..Houses
SET OwnerCiudad = TRIM(REPLACE(OwnerCiudad,'  ',''));


 -- Actualización del tipo de valor al atributo YearBuilt de float a int
ALTER TABLE Nashville.dbo.Houses ALTER COLUMN YearBuilt int;  
GO 
