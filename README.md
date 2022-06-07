# LimpiezaDatosSQL
Limpieza de datos usando SQL.

## Objetivo
Realizar un proceso de limpieza a un set de datos utilizando la herramienta SQL Server. Efectuando distintas tareas como visualizaciones de datos, correcciones de valores, actualizaciones de tablas, eliminación de espacios innecesarios, etc.

Este es un proyecto que hace parte de la lista de reproducción de proyectos para el desarrollo de un portafolio de Alex The Analyst. 

En los archivos podrán encontrar:

1. [Nashville Housing Data for Data Cleaning.xlsx](https://github.com/FabianPedreros/LimpiezaDatosSQL/blob/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx)--> Archivo de excel con los datos a limpiar.
2. [Limpieza de datos en SQL.pptx](https://github.com/FabianPedreros/LimpiezaDatosSQL/blob/main/Limpieza%20de%20datos%20en%20SQL.pptx)--> Presentación en la que de manera resumida se muestran los pasos ejecutados para la limpieza en SQL.
3. [LimpiezaDeDatos.sql](https://github.com/FabianPedreros/LimpiezaDatosSQL/blob/main/LimpiezaDeDatos.sql)--> Archivo en el que se encuentran las consultas ejecutadas.
4. [Cheat Sheet SQL Data Preparation.jpg](https://github.com/FabianPedreros/LimpiezaDatosSQL/blob/main/Cheat%20Sheet%20SQL%20Data%20Preparation.jpg)--> Imagen que contiene algunas de las consultas más utilizadas para la preparación de datos en SQL.



Link a la lista de reproducción de Alex. https://www.youtube.com/watch?v=8rO7ztF4NtU&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f&index=3&ab_channel=AlexTheAnalyst


## Consulta para primera visualización de datos

	SELECT TOP 1000 *
	FROM Nashville..Houses;
	
![image](https://user-images.githubusercontent.com/32172901/172442192-b04d530a-b7e0-48a4-9f88-f275fa450dd7.png)


## Estandarización de fechas (SaleDate)
   Eliminación de  los valores de horas, ya que solo contiene información de fecha

  ### Visualización de datos SaleDate

	  SELECT SaleDate
	  FROM Nashville..Houses;
![image](https://user-images.githubusercontent.com/32172901/172442464-9e536158-b355-4b3d-9648-71c46e07181b.png)

  ### Visualización de modificación del tipo de dato a Fecha

	  SELECT SaleDate , CONVERT(date, SaleDate)
	  FROM Nashville..Houses;

  ### Actualización del campo SaleDate a el formato date
  
	  UPDATE Nashville..Houses
	  SET SaleDate = CONVERT(date, SaleDate);

  ### Para el caso que no permita la actualización, se puede crear una nueva columna y eliminar la otra

	  ALTER TABLE Nashville..Houses
	  ADD SaleDateConverted Date;

	  UPDATE Nashville..Houses
	  SET SaleDateConverted = CONVERT(date, SaleDate);

	  ALTER TABLE Nashville..Houses
	  DROP COLUMN SaleDate;
![image](https://user-images.githubusercontent.com/32172901/172442597-c2d3627b-0b3c-427c-bbd6-f9c93afd7b3d.png)


  ### Revisión y conteo de valores nulos para SaleDate

	  SELECT SaleDateConverted
	  FROM Nashville..Houses
	  WHERE SaleDateConverted IS NULL;

## Transformación de datos para el campo PropertyAddress
### Visualización de datos para el campo PropertyAddress

	SELECT PropertyAddress
	FROM Nashville..Houses;

### Revisión y conteo de valores nulos para SaleDate

	SELECT PropertyAddress
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;

	SELECT COUNT(*)
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;

### Revisión de valores nulos en PropertyAddress
	SELECT *
	FROM Nashville..Houses
	WHERE PropertyAddress IS NULL;
	
![image](https://user-images.githubusercontent.com/32172901/172442706-18de1a67-4a43-4aea-9d61-7c32e48c3612.png)

![image](https://user-images.githubusercontent.com/32172901/172442772-9b77832e-0aa4-44e1-9ea4-7c2e92f4d46c.png)


## Consulta de registros con más de un ParcelID
	SELECT COUNT(PropertyAddress), ParcelID, PropertyAddress
	FROM Nashville..Houses
	GROUP BY ParcelID, PropertyAddress
	HAVING COUNT (PropertyAddress) > 1
	ORDER BY ParcelID
	
![image](https://user-images.githubusercontent.com/32172901/172443301-6c8a910e-38b1-424f-95c9-ccd2a8f206fa.png)


### Consulta de ParcelID y PropertyAddress de la consulta anterior
	SELECT ParcelID, PropertyAddress
	FROM Nashville..Houses
	WHERE ParcelID IN 
		(SELECT ParcelID
		FROM Nashville..Houses
		GROUP BY ParcelID, PropertyAddress
		HAVING COUNT (PropertyAddress) > 1)
	ORDER BY ParcelID

### Consulta de los ParcelID y PropertyAddress para la asignacióna los nulos, join a la misma tabla

	SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
	FROM Nashville..Houses AS a
	JOIN Nashville..Houses AS b
		ON a.ParcelID = b.ParcelID
		AND a.[UniqueID ]<> b.[UniqueID ]
	WHERE a.PropertyAddress IS NULL;
	
![image](https://user-images.githubusercontent.com/32172901/172443370-6052e63a-807d-4883-8b41-47c241750a89.png)


### Actualización de las direcciones de los ParcelID con direcciones nulas

	UPDATE a
	SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
	FROM Nashville..Houses AS a
	JOIN Nashville..Houses AS b
		ON a.ParcelID = b.ParcelID
		AND a.[UniqueID ]<> b.[UniqueID ]
	WHERE a.PropertyAddress IS NULL;
	
![image](https://user-images.githubusercontent.com/32172901/172443402-a0e73349-e928-4a97-a96c-212492a2051b.png)

![image](https://user-images.githubusercontent.com/32172901/172443464-d3c536f1-d29f-4d9a-b35e-754aedfb04dc.png)
 


## División en columnas de los datos en PropertyAddress

### Consulta de los datos en las direcciones

		SELECT DISTINCT(PropertyAddress)
		FROM Nashville..Houses;
		
![image](https://user-images.githubusercontent.com/32172901/172443724-2357900f-1592-448a-bb0b-3815a47b8192.png)


### Consulta para la división de dirección, valor antes de la coma

		SELECT
		SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) as Direccion
		FROM Nashville..Houses;
		
![image](https://user-images.githubusercontent.com/32172901/172443793-298e29cb-648c-4a8c-8e71-38093be1f287.png)


### Consulta para la división de ciudad, valor despúes de la coma
		SELECT
		SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress)) as Ciudad
		FROM Nashville..Houses;

### Agregar la columna y valores para dirección

		ALTER TABLE Nashville..Houses
		ADD PropertyDireccion NVARCHAR(255)

		UPDATE Nashville..Houses
		SET PropertyDireccion = (SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1));

![image](https://user-images.githubusercontent.com/32172901/172443855-33c8c442-c8c2-4b99-b488-a29d57fd6a8a.png)

### Agregar la columna y valores para ciudad

		ALTER TABLE Nashville..Houses
		ADD PropertyCiudad NVARCHAR(255)

		UPDATE Nashville..Houses
		SET PropertyCiudad = SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) + 2, LEN(PropertyAddress));


		SELECT TOP 100 *
		FROM Nashville..Houses;



## División en columnas de los datos en OwnerAddress

### Consulta de los datos en las direcciones

		SELECT DISTINCT(OwnerAddress)
		FROM Nashville..Houses;
		
![image](https://user-images.githubusercontent.com/32172901/172443889-65ed5a0d-684a-48a0-8991-fa0f44786a31.png)


### Consulta para la división de los valores en OwnerAddress

		SELECT  PARSENAME(REPLACE(OwnerAddress, ',', '.'),3),
				PARSENAME(REPLACE(OwnerAddress, ',', '.'),2),
				PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)
		FROM Nashville..Houses;
		
![image](https://user-images.githubusercontent.com/32172901/172443946-e7603b35-a0c5-4e25-8895-d8434ef5a8e1.png)



### Agregar la columna y valores para la dirección del dueño

		ALTER TABLE Nashville..Houses
		ADD OwnerDireccion NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerDireccion = PARSENAME(REPLACE(OwnerAddress, ',', '.'),3);
		
![image](https://user-images.githubusercontent.com/32172901/172443994-97cf18ee-2333-40b1-8fa1-be05ad9cf23c.png)


### Agregar la columna y valores para la ciudad del dueño

		ALTER TABLE Nashville..Houses
		ADD OwnerCiudad NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerCiudad = PARSENAME(REPLACE(OwnerAddress, ',', '.'),2);


### Agregar la columna y valores para el estado del dueño

		ALTER TABLE Nashville..Houses
		ADD OwnerEstado NVARCHAR(255)

		UPDATE Nashville..Houses
		SET OwnerEstado = PARSENAME(REPLACE(OwnerAddress, ',', '.'),1);


##- Revisión de valores en SoldAsVacant
### Consulta de valores y conteo de frecuencias

		SELECT DISTINCT (SoldAsVacant), COUNT (SoldAsVacant) AS Frecuencia
		FROM Nashville..Houses
		GROUP BY SoldAsVacant
		ORDER BY Frecuencia;
		
![image](https://user-images.githubusercontent.com/32172901/172444054-a8944551-42f5-4b07-bf01-d0bbe7140030.png)

### Consulta de valores permitidos en SoldAsVacant

		SELECT SoldAsVacant, 
		CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
			 WHEN SoldAsVacant = 'N' THEN 'No'
			 ELSE SoldAsVacant
			 END
		FROM Nashville..Houses

### Actualización de valores permitidos en SoldAsVacant

		UPDATE Nashville..Houses
		SET SoldAsVacant = (CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
			 WHEN SoldAsVacant = 'N' THEN 'No'
			 ELSE SoldAsVacant
			 END)
![image](https://user-images.githubusercontent.com/32172901/172444106-e1c50d9b-cdfd-4072-b871-172e114a8a96.png)
![image](https://user-images.githubusercontent.com/32172901/172444149-1cabfa59-7fb7-4e01-a03d-1383a13fbc96.png)


## Eliminación de duplicados

 Consulta de registros duplicados según las columnas:
 ParcelId, PropertyAddres, SalePrice, SaleDate, LegalReference

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


### Eliminación de duplicados

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
		
![image](https://user-images.githubusercontent.com/32172901/172444361-84e0779c-ad56-49f2-b27f-e4ac491acf24.png)


### Eliminar columnas innecesarias ###

		ALTER TABLE Nashville..Houses
		DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress
		
![image](https://user-images.githubusercontent.com/32172901/172444550-f5f823a0-6e0e-4a14-9851-56db4800ad6f.png)


## Eliminar espacios innecesarios en los atributos con strings
### Validación de atributo LandUse con valores con espacios adicionales 

		SELECT LandUse, TRIM(REPLACE(LandUse,'  ','')), LEN(LandUse), LEN(TRIM(REPLACE(LandUse,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(LandUse) <> LEN(TRIM(REPLACE(LandUse,'  ','')))
		
![image](https://user-images.githubusercontent.com/32172901/172444623-30c954c0-a043-4bdd-8c7d-ee988b694573.png)


### Actualización del valor CONDOMINIUM OFC  OR OTHER COM CONDO ###> CONDOMINIUM OFC OR OTHER COM CONDO

		UPDATE Nashville..Houses
		SET LandUse = (CASE WHEN LandUse = 'CONDOMINIUM OFC  OR OTHER COM CONDO' THEN 'CONDOMINIUM OFC OR OTHER COM CONDO'
			 ELSE LandUse
			 END)
			 
![image](https://user-images.githubusercontent.com/32172901/172444663-f6c977dc-241b-40d0-bbf0-845939f588a4.png)


### Validación de actualización

		SELECT LandUse, COUNT(LandUse) AS Frecuencia
		FROM Nashville..Houses
		GROUP BY LandUse
		ORDER BY Frecuencia


### Validación de atributo UniqueId con valores con espacios adicionales, no tiene.

		SELECT [UniqueID], TRIM(REPLACE([UniqueID],'  ','')), LEN([UniqueID]), LEN(TRIM(REPLACE([UniqueID],'  ','')))
		FROM Nashville..Houses
		WHERE LEN([UniqueID]) <> LEN(TRIM(REPLACE([UniqueID],'  ','')))

### Validación de atributo ParcelID con valores con espacios adicionales, no tiene.

		SELECT ParcelId, TRIM(REPLACE(ParcelId,'  ','')), LEN(ParcelId), LEN(TRIM(REPLACE(ParcelId,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(ParcelId) <> LEN(TRIM(REPLACE(ParcelId,'  ','')))

### Validación de atributo LegalReference con valores con espacios adicionales, no tiene.

		SELECT LegalReference, TRIM(REPLACE(LegalReference,'  ','')), LEN(LegalReference), LEN(TRIM(REPLACE(LegalReference,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(LegalReference) <> LEN(TRIM(REPLACE(LegalReference,'  ','')))

### Validación de atributo OwnerName con valores con espacios adicionales, espacios dobles al interior de la cadena.

		SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


### Actualización del valor en OwnerName eliminando espacios dobles
		UPDATE Nashville..Houses
		SET OwnerName = TRIM(REPLACE(OwnerName,'  ',' '));

### Validación de la actualización

		SELECT OwnerName, TRIM(REPLACE(OwnerName,'  ','')), LEN(OwnerName), LEN(TRIM(REPLACE(OwnerName,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerName) <> LEN(TRIM(REPLACE(OwnerName,'  ','')));


### Validación de atributo PropertyDireccion con valores con espacios adicionales, 55788 valores líneas con espacios dobles, triples, al inicio o final.

		SELECT PropertyDireccion, TRIM(REPLACE(PropertyDireccion,'  ','')), LEN(PropertyDireccion), LEN(TRIM(REPLACE(PropertyDireccion,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(PropertyDireccion) <> LEN(TRIM(REPLACE(PropertyDireccion,'  ','')));

### Actualización del valor en PropertyDireccion eliminando espacios triples
		UPDATE Nashville..Houses
		SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'   ',' '));

### Actualización del valor en PropertyDireccion eliminando espacios dobles
		UPDATE Nashville..Houses
		SET PropertyDireccion = TRIM(REPLACE(PropertyDireccion,'  ',' '));


### Validación de atributo PropertyCiudad con valores con espacios adicionales, no hay.

		SELECT PropertyCiudad, TRIM(REPLACE(PropertyCiudad,'  ','')), LEN(PropertyCiudad), LEN(TRIM(REPLACE(PropertyCiudad,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(PropertyCiudad) <> LEN(TRIM(REPLACE(PropertyCiudad,'  ','')));

### Validación de atributo OwnerDireccion con valores con espacios adicionales, 25273 valores líneas con espacios dobles, triples, al inicio o final.

		SELECT OwnerDireccion, TRIM(REPLACE(OwnerDireccion,'  ','')), LEN(OwnerDireccion), LEN(TRIM(REPLACE(OwnerDireccion,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerDireccion) <> LEN(TRIM(REPLACE(OwnerDireccion,'  ','')));

### Actualización del valor en OwnerDireccion eliminando espacios triples
		UPDATE Nashville..Houses
		SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'   ',' '));

### Actualización del valor en OwnerDireccion eliminando espacios dobles
		UPDATE Nashville..Houses
		SET OwnerDireccion = TRIM(REPLACE(OwnerDireccion,'  ',' '));

### Validación de atributo OwnerCiudad con valores con espacios adicionales, 25969 valores líneas con espacios dobles, triples, al inicio o final.

		SELECT OwnerCiudad, TRIM(REPLACE(OwnerCiudad,'  ','')), LEN(OwnerCiudad), LEN(TRIM(REPLACE(OwnerCiudad,'  ','')))
		FROM Nashville..Houses
		WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

		SELECT DISTINCT(OwnerCiudad)
		FROM Nashville..Houses
		WHERE LEN(OwnerCiudad) <> LEN(TRIM(REPLACE(OwnerCiudad,'  ','')));

![image](https://user-images.githubusercontent.com/32172901/172444737-a966af90-63d3-42db-9fc2-c847fed3a5f1.png)

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
 
 ### Actualización del valor en OwnerCiudad eliminando espacios dobles
		
	UPDATE Nashville..Houses
	SET OwnerCiudad = TRIM(REPLACE(OwnerCiudad,'  ',''));
		

 ### Actualización del tipo de valor al atributo YearBuilt de float a int
		
	ALTER TABLE Nashville.dbo.Houses ALTER COLUMN YearBuilt int;  
	GO 
	
![image](https://user-images.githubusercontent.com/32172901/172444817-f2ddeef4-9551-4f95-b3e7-a119d95c336b.png)
