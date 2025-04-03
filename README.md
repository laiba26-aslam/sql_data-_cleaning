# sql_data-_cleaning
USE world_la_off;

SELECT *
FROM layoffs;

-- SQL Project - Data Cleaning

-- now when we are data cleaning we usually follow a few steps
-- 1. check for duplicates and remove any
-- 2. standardize data and fix errors
-- 3. Look at null values and see what 
-- 4. remove any columns and rows that are not necessary - few ways


-- first thing we want to do is create a staging table. This is the one we will work in and clean the data. We want a table with the raw data in case something happens
CREATE TABLE layoffs_stagin
LIKE layoffs;
SELECT *
FROM layoffs_stagin;

-- 1. Remove Duplicates

# First let's check for duplicates

 SELECT *,
 ROW_NUMBER() OVER( 
PARTITION BY company,industry,total_laid_off,percentage_laid_off,'date') AS row_num
FROM layoffs_stagin;

INSERT  layoffs_stagin
SELECT *
FROM layoffs;

WITH duplicate_cute AS
(
 SELECT *,
 ROW_NUMBER() OVER( 
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_num
FROM layoffs_stagin
)
SELECT *
FROM duplicate_cute
WHERE row_num>1;

 -- let's just look at  Casper to confirm
SELECT * 
FROM layoffs_stagin
WHERE company='Casper';

 -- it looks like these are all legitimate entries and shouldn't be deleted. We need to really look at every single row to be accurate

-- these are our real duplicates
 
 
WITH duplicate_cute AS
(
 SELECT *,
 ROW_NUMBER() OVER( 
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_num
FROM layoffs_stagin
)
DELETE
FROM duplicate_cute
WHERE row_num>1;
 

CREATE TABLE `layoffs_stagin2` (
`company` text,
`location`text,
`industry`text,
`total_laid_off` INT,
`percentage_laid_off` text,
`date` text,
`stage`text,
`country` text,
`funds_raised_millions` int,
row_num INT
);

SELECT *
FROM layoffs_stagin2;

INSERT INTO layoffs_stagin2
 SELECT *,
 ROW_NUMBER() OVER( 
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_num
FROM layoffs_stagin;

 SET SQL_SAFE_UPDATES = 0;
DELETE
FROM layoffs_stagin2
WHERE row_num >1;
SET SQL_SAFE_UPDATES = 1;

SELECT *
FROM layoffs_stagin2
WHERE row_num >1;

-- standarization --
-- if we look at industry it looks like we have some null and empty rows, let's take a look at these

 SELECT company,TRIM(company)
  FROM layoffs_stagin2; 
  
    UPDATE layoffs_stagin2
  SET company=TRIM(company);
 SET SQL_SAFE_UPDATES = 0;
  SET SQL_SAFE_UPDATES = 1;
  
    SELECT  DISTINCT industry
  FROM layoffs_stagin2
  ORDER BY 1;
  
    SELECT * 
  FROM layoffs_stagin2
  WHERE industry LIKE 'Crypto%';
    -- setting crypto currency as only crypto--
    SET SQL_SAFE_UPDATES = 0;
    UPDATE layoffs_stagin2
    SET industry='Crypto'
    WHERE industry LIKE 'Crypto%';
      SET SQL_SAFE_UPDATES = 1;
   SELECT * 
  FROM layoffs_stagin2;

  SELECT DISTINCT Country
  FROM layoffs_stagin2
  ORDER BY 1;
    
    SET SQL_SAFE_UPDATES = 0;
  UPDATE layoffs_stagin2   -- here updatinh the table--
  SET country=TRIM(TRAILING '.'FROM  country)
  WHERE country LIKE 'United States%';
   SET SQL_SAFE_UPDATES = 1;
   
   
   SELECT date
   FROM layoffs_stagin2;
   -- Date formating --
   SELECT date ,
   STR_TO_DATE(date,'%m/%d/%Y')
    FROM layoffs_stagin2;
-- now we are going to update our date column in layoffs_staging2--
 SET SQL_SAFE_UPDATES = 0;
UPDATE layoffs_stagin2
SET date =   STR_TO_DATE(date,'%m/%d/%Y');
 SET SQL_SAFE_UPDATES = 1;
  -- date column represent as text now convert into date format--
ALTER TABLE  layoffs_stagin2
MODIFY COLUMN date  DATE;

SELECT *
FROM layoffs_stagin2;
-- NULL VALUES--
  SELECT * 
  FROM layoffs_stagin2
  WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL ;
  
   SET SQL_SAFE_UPDATES = 0;
   UPDATE layoffs_stagin2
   SET industry = NULL
   WHERE industry ='';
    SET SQL_SAFE_UPDATES = 1;
  
  SELECT * 
  FROM layoffs_stagin2
  WHERE industry IS NULL OR industry ='';
   
SELECT * 
FROM layoffs_stagin2
WHERE company='Airbnb' ;
 
 SELECT t1.industry,t2.industry
 FROM layoffs_stagin2 t1
 JOIN  layoffs_stagin2 t2
 ON t1.company=t2.company
 AND T1.location=t2.location
WHERE (t1.industry IS NULL OR t1.industry ='')
AND   t2.industry IS NOT NULL; 


 SET SQL_SAFE_UPDATES = 0;
UPDATE layoffs_stagin2 t1
JOIN layoffs_stagin2 t2
 ON t1.company=t2.company
 SET t1.industry=t2.industry
 WHERE t1.industry IS NULL 
AND   t2.industry IS NOT NULL;

  SET SQL_SAFE_UPDATES = 1;
  
  SELECT *
FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL;


-- 3. Look at Null Values

-- the null values in total_laid_off, percentage_laid_off, and funds_raised_millions all look normal. I don't think I want to change that
-- I like having them null because it makes it easier for calculations during the EDA phase

-- so there isn't anything I want to change with the null values

-- 4. remove any columns and rows we need to
SELECT *
FROM layoffs_stagin2 
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Delete Useless data we can't really use
 SET SQL_SAFE_UPDATES = 0;
DELETE  FROM layoffs_stagin2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
 SET SQL_SAFE_UPDATES = 1;
 
SELECT * 
FROM layoffs_stagin2;

 
ALTER TABLE layoffs_stagin2
DROP COLUMN row_num;


SELECT    *
FROM  layoffs_stagin2;
