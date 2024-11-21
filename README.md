# World Layoffs Data Cleaning Project

### Project Overview
This project will show the steps that I take when I start cleaning data in SQL.

### Data Sources
 https://www.kaggle.com/datasets/swaptr/layoffs-2022

 ### Tools
 - Excel
 - MYSQL

### Data Cleaning/Preparation
 1. check for duplicates and remove any
 2. standardize data and fix errors
 3. Look at null values and see what 
 4. remove any columns and rows that are not necessary

```sql
-- Creating a Duplicate of the raw table to work on
create table layoffs_dup
like layoffs;

insert layoffs_dup
select*
from layoffs

--
-- REMOVING DUPLICATES

-- first using row_number to match against all columns and counting duplicates in a new row
select *,
row_number() over
(partition by company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) as row_num
from layoffs_dup

-- filtering the new column with a cte to see if theres a duplicate
with duplicate_cte as
(
select *,
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) as row_num
from layoffs_dup
)
select *
from duplicate_cte
where row_num > 1;

-- now to delete the dupicate row without deleting the original row
 CREATE TABLE `layoffs_dup2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
  

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

insert into layoffs_dup2
  select *,
row_number() over
(partition by company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) as row_num
from layoffs_dup

select *
from layoffs_dup2
where row_num > 1

-- STANDARDIZE DATA

-- trimming away the white space
update layoffs_dup2
set company = trim(company)

-- Standardizing the names
select distinct country
from layoffs_dup2
order by 1

select *
from layoffs_dup2
where industry like 'Crypto%'

update layoffs_dup2
set industry = 'Crypto'
where industry like 'Crypto%';

update layoffs_dup2
set country = 'United States'
where country like 'United States%';

-- Change the "date" column from text to date format

select `date`,
str_to_date(`date`, '%m/%d/%Y')
from layoffs_dup2

update layoffs_dup2
set `date` = str_to_date(`date`, '%m/%d/%Y')

alter table layoffs_dup2
modify column `date` date

-- NULL AND BLANK VALUES

-- finding the null or blank values
select *
from layoffs_dup2
where industry is null
or industry = ""

-- seeing what to populate the null or blank values with
select l1.company, l1.industry, l2.industry
from layoffs_dup2 as l1
join layoffs_dup2 as l2
on l1.company = l2.company
where (l1.industry is null or l1.industry = "")
and l2.industry is not null

-- changing blanks to nulls so that they can be populated
update layoffs_dup2
set industry = null
where industry = ""

-- populate the null values
update layoffs_dup2 l1
join layoffs_dup2 as l2
on l1.company = l2.company
set l1.industry = l2.industry
where (l1.industry is null)
and l2.industry is not null

-- REMOVE UNNECESSARY COLUMNS AND/OR ROWS
select *
from layoffs_dup2
where total_laid_off is null
and percentage_laid_off is null

delete
from layoffs_dup2
where total_laid_off is null
and percentage_laid_off is null

alter table layoffs_dup2
drop column row_num
```




