# Explore-Covid-19-Data-Set
Explore public data sets for Covid 19 using SQL

/*
Covid 19 Data Exploration 
Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types
*/

--Data Source https://ourworldindata.org/covid-deaths
--Extract data "Covid Deaths"
--Extract data "Covid Vaccinations"
--Multivariate data mining

--Checking CovidDeaths for data integrity --

SELECT *
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE continent is not NULL
ORDER BY 3, 4

--Checking CovidDeaths for data integrity.--
SELECT *
FROM PortfolioProject_Covid_19..CovidVaccinations
ORDER BY 3, 4

--Select data that we are going to be using.--

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject_Covid_19..CovidDeaths
ORDER BY 1, 2 

--Looking at total cases vs total deaths.--
SELECT location, date, total_cases, total_deaths, CAST(total_deaths AS DECIMAL)/total_cases*100 AS death_percentage 
FROM PortfolioProject_Covid_19..CovidDeaths
ORDER BY 1, 2

-- Looking at total cases vs deaths in United States.--
--Shows likelihood of dying if you contract covid in your country as percent.--

SELECT location, date, total_cases, total_deaths, CAST(total_deaths AS DECIMAL)/total_cases*100 AS death_percentage 
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE location like '%states%'
ORDER BY 1, 2

--Looking at total cases vs population.--
--Shows what percentage of population that got covid.__ 

SELECT location, date, population, total_cases, CAST(total_cases AS DECIMAL)/population*100 AS death_percentage 
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE location like '%states%'
ORDER BY 1, 2

--looking at which countries with highest infection rate compared to population--

SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases*1.0))/population*100 AS PercentPopulationInfected
FROM PortfolioProject_Covid_19..CovidDeaths
--WHERE location like '%states%'
GROUP BY location, population
ORDER BY PercentPopulationInfected DESC

--Looking at the countries with the highest death count per population.--

SELECT location, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject_Covid_19..CovidDeaths
--WHERE location like '%states%'
WHERE continent is not NULL
GROUP BY location
ORDER BY TotalDeathCount DESC

--Data broken down by continent.--

SELECT continent,  MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent 
ORDER BY TotalDeathCount DESC


--Remove High income, Upper middle income, Lower middle income, Low income,from results.--
--The below query will include more accurate numbers.--

--SELECT location,  MAX(CAST(total_deaths AS int)) AS TotalDeathCount
--FROM PortfolioProject_Covid_19..CovidDeaths
--WHERE continent IS NULL AND location NOT LIKE '%High income%' AND location NOT LIKE '%Upper middle income%' AND location NOT LIKE '%Lower middle income%'
--AND location NOT LIKE '%Low income%' 
--GROUP BY location 
--ORDER BY TotalDeathCount DESC


--Looking at global numbers.--

SELECT date, 
    SUM(new_cases) AS total_cases, 
    SUM(CAST(new_deaths as DECIMAL)) AS total_deaths, 
    CASE WHEN SUM(new_cases) = 0 THEN 0 ELSE SUM(CAST(new_deaths as DECIMAL)) / SUM(new_cases) * 100 
    END AS DeathPercentage
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY date 
ORDER BY 1, 2 

--TOTAL CASES IN THE WORLD--

SELECT --date 
    SUM(new_cases) AS total_cases, 
    SUM(CAST(new_deaths as DECIMAL)) AS total_deaths, 
    CASE WHEN SUM(new_cases) = 0 THEN 0 ELSE SUM(CAST(new_deaths as DECIMAL)) / SUM(new_cases) * 100 
    END AS DeathPercentage
FROM PortfolioProject_Covid_19..CovidDeaths
WHERE continent IS NOT NULL
--GROUP BY date 
ORDER BY 1, 2 

--JOIN two tables Covid Deaths and CovidVacinations.--
--Inner Join(default) specify location and date.-- 
 
    SELECT *
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date


--Looking at the total population vs vacinations.--
--Add in rolling total to track progress of Covid vaccination campaigns around the world.--

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
    dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
    ORDER BY 1,2,3
--SELECT location,  MAX(CAST(total_deaths AS int)) AS TotalDeathCount
--FROM PortfolioProject_Covid_19..CovidDeaths
--WHERE continent IS NULL AND location NOT LIKE '%High income%' AND location NOT LIKE '%Upper middle income%' AND location NOT LIKE '%Lower middle income%'
--AND location NOT LIKE '%Low income%' 
--GROUP BY location 
--ORDER BY TotalDeathCount DESC

---USE CTE or common table expression

WITH PopvsVac (continent, location, date, population, new_vaccinations, rolling_vaccination_totals)
AS
(
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
    dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL 
    --ORDER BY 1,2,3
)
SELECT *, (CAST(rolling_vaccination_totals as DECIMAL))/(CAST(population AS DECIMAL)*100)
FROM PopvsVac;

--- Clean it up--

WITH PopvsVac (continent, location, date, population, new_vaccinations, rolling_vaccination_totals) AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
    dea.date) AS rolling_vaccination_totals
    FROM PortfolioProject_Covid_19..CovidDeaths dea
    JOIN PortfolioProject_Covid_19..CovidVaccinations vac
        ON dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (CAST(rolling_vaccination_totals as DECIMAL))/(CAST(population AS DECIMAL)*100) As total_percent_Vaccinated
FROM PopvsVac;

-- to add the clarity and accuracy to query removing additional data categories that grouped countries by region.--
-- High Income, Upper income, Middle income, Lower middle income, Low income.--

WITH PopvsVac (continent, location, date, population, new_vaccinations, rolling_vaccination_totals)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
AND dea.location NOT LIKE '%High income%'
AND dea.location NOT LIKE '%Upper middle income%'
AND dea.location NOT LIKE '%Middle income%'
AND dea.location NOT LIKE '%Lower middle income%'
AND dea.location NOT LIKE '%Low income%'
)
SELECT
PopvsVac.continent,
PopvsVac.location,
PopvsVac.date,
PopvsVac.population,
PopvsVac.new_vaccinations,
PopvsVac.rolling_vaccination_totals,
(CAST(PopvsVac.rolling_vaccination_totals AS DECIMAL))/(CAST(PopvsVac.population AS DECIMAL)*100) AS vaccination_rate
FROM PopvsVac;

--Note that I also added aliases for the table in the SELECT clause for clarity.




--Option to use temp table.--

--DROP TABLE IF EXISTS #percent_population_vaccinated
-- To make alterations, run multiple times. 
Create Table #percent_population_vaccinated
(
    continent NVARCHAR(100), 
    location NVARCHAR(100), 
    date DATETIME, 
    population NUMERIC,
    new_vaccinations NUMERIC,
    rolling_vaccination_totals NUMERIC
)
INSERT INTO #percent_population_vaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
AND dea.location NOT LIKE '%High income%'
AND dea.location NOT LIKE '%Upper middle income%'
AND dea.location NOT LIKE '%Middle income%'
AND dea.location NOT LIKE '%Lower middle income%'
AND dea.location NOT LIKE '%Low income%'


--A few queries to check table.--
--SELECT COUNT(*) FROM #percent_population_vaccinated;
--SELECT TOP 100 * FROM #percent_population_vaccinated;

--SELECT location, date, population, new_vaccinations,rolling_vaccination_totals
--FROM #percent_population_vaccinated
--WHERE continent like '%South America%'
--ORDER BY 1, 2

-- Create view to store data for later visualizations.--

CREATE VIEW percent_population_vaccinatedSELECT AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
    dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
    --ORDER BY 1,2,3

--DROP VIEW IF EXISTS percent_population_vaccinated;

CREATE VIEW dbo.percent_population_vaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION by dea.location ORDER BY dea.location,
    dea.date) AS rolling_vaccination_totals
FROM PortfolioProject_Covid_19..CovidDeaths dea
JOIN PortfolioProject_Covid_19..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
