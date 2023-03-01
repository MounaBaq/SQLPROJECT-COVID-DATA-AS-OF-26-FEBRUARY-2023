Select *
From SQLPROJECT1..CovidDeaths23
Order By Location

--Select *
--From SQLPROJECT1..CovidVaccinations23
--Order By Location


-- Country Morocco
-- Selecting Data that we are going to be using 

Select Continent,Location,Date, Population,total_cases,new_cases,total_deaths
From SQLPROJECT1..CovidDeaths23
Where Continent Is Not NULL
And Location = 'Morocco'
Order by Location,date

-- Looking at Total Cases Vs Total Deaths in Morocco
--Likelyhood of death if Covid Infected in Morocco

Select Location,Date, Population,total_cases,new_cases,total_deaths, (Convert(Int,Total_deaths)/Total_cases)*100 As DeathPercentage
From SQLPROJECT1..CovidDeaths23
Where Continent Is Not NULL
And Location = 'Morocco'
Order by Location,date

-- Looking at Total Cases vs Total Deaths WorldWide

Select Location,Date, Population,total_cases,new_cases,total_deaths, (Convert(Int,Total_deaths)/Total_cases)*100 As DeathPercentage
From SQLPROJECT1..CovidDeaths23
Where Continent Is Not NULL
--And Location = 'Morocco'
Order by Location,date

--Looking at Total Cases vs Population in Morocco

Select Location,Date, Population,total_cases,new_cases,total_deaths, (Convert(Int,Total_deaths)/Population)*100 As InfectedPopulationPercentageInMorocco
From SQLPROJECT1..CovidDeaths23
Where Continent Is Not NULL
And Location = 'Morocco'
Order by Location,date

--Looking at Total Cases vs Population WorldWide

Select Location,Date, Population,total_cases,new_cases,total_deaths, (Convert(Int,Total_deaths)/Population)*100 As InfectedPopulationPercentageWorldWide
From SQLPROJECT1..CovidDeaths23
Where Continent Is Not NULL
--And Location = 'Morocco'
Order by Location,date

-- Looking at countries with highest infection rates compared to poulation

Select Location,Population,Max(Total_cases) As Max_Infections, Max((Total_cases/Population))*100 As HighestInfectionRate
From SQLPROJECT1..CovidDeaths23
Where Continent is not Null 
--And Location = 'Morocco'
Group By Location,Population
Order By HighestInfectionRate DESC


-- Showing countries with highest death counts per population

Select location, Max(Convert(Int,total_deaths))As Max_DeathsCounts
From SQLPROJECT1..CovidDeaths23
Where Continent is not Null 
--And Location = 'Morocco'
Group By location
Order By Max_DeathsCounts DESC

-- Showing Continent with highest death counts per population

Select Continent, Max(Convert(Int,total_deaths))As Max_DeathsCounts
From SQLPROJECT1..CovidDeaths23
Where Continent is not Null 
--And Location = 'Morocco'
Group By Continent
Order By Max_DeathsCounts DESC

--Showing death rate globaly in an other way : SUm of new cases and New deaths.
-- By Date
Select Date,Sum(Convert(Int,New_deaths)) As SumofNewDeaths, Sum(New_cases) As SumofNewCases, Sum(Convert(Int,New_deaths))/Sum(New_cases)*100 As GlobalDeathPercent
From SQLPROJECT1..CovidDeaths23
Where Continent is not Null
Group By Date
Order by GlobalDeathPercent DESC

-- With no Date

Select Sum(Convert(Int,New_deaths)) As SumofNewDeaths, Sum(New_cases) As SumofNewCases, Sum(Convert(Int,New_deaths))/Sum(New_cases)*100 As GlobalDeathPercent
From SQLPROJECT1..CovidDeaths23
Where Continent is not Null
Order by GlobalDeathPercent DESC


--Select Max(Convert(Int,Total_deaths)) As MaxofTotalDeaths, Max(Total_cases) As MaxofTotalCases, Max(Convert(Int,Total_deaths))/Max(Total_cases)*100 As GlobalDeathPercent
--From SQLPROJECT1..CovidDeaths23
--Where Continent is not Null
--Order by GlobalDeathPercent DESC

-- We can't use the above querry because The columns 'Total deaths' and ' Total cases' 
--represents the sum of everyday cases weather or not there are new cases or new deaths

-- USE OF JOINS

Select *
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date

-- Looking at Total Population vs Vaccination

Select Deaths.Continent, Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null
Order by Location,date


-- Breaking up by location using PARTITION BY and using ROLLING COUNT

Select Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations, Sum(Convert(BigInt,Vacci.New_vaccinations)) 
Over ( Partition By Deaths.Location Order By Deaths.Date) As RollingCountVacci
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null
Order by Location,date 

--New vaccine percentage

Select Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations, Sum(Convert(BigInt,Vacci.New_vaccinations))
Over ( Partition By Deaths.Location Order By Deaths.Date) As RollingCountVacci, (Sum(Convert(BigInt,Vacci.New_vaccinations))
Over ( Partition By Deaths.Location Order By Deaths.Date)/deaths.population)*100 As TotalNewVaccinPercentage
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null
Order by Location,date 

-- Using CTE (Common Table Expression)

With PopVsVacci (Location, Date, Population, new_vaccinations, RollingCountVacci)
As
(
Select Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations, Sum(Convert(BigInt,Vacci.New_vaccinations)) 
Over ( Partition By Deaths.Location Order By Deaths.Date) As RollingCountVacci
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null
--Order by Location,date 
)
Select *, (RollingCountVacci/Population)*100
From PopVsVacci

--TEMP TABLE

DROP TABLE IF EXISTS #Percent_Of_People_Vaccinated
Create Table #Percent_Of_People_Vaccinated
(
Location Varchar(255),
Date Datetime,
Population numeric,
New_Vaccinations numeric,
RollingCountVacci numeric,
)

INSERT INTO #Percent_Of_People_Vaccinated
Select Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations, Sum(Convert(BigInt,Vacci.New_vaccinations)) 
Over ( Partition By Deaths.Location Order By Deaths.Date) As RollingCountVacci
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null
--Order by Location,date 

Select * , (RollingCountVacci/Population)*100
From #Percent_Of_People_Vaccinated
--Where Location='Morocco'

-- Creating View to store data for later visualizations

Create View Percent_Of_People_Vaccinated 
AS
Select Deaths.Location, Deaths.Date, Deaths.Population, Vacci.New_Vaccinations, Sum(Convert(BigInt,Vacci.New_vaccinations)) 
Over ( Partition By Deaths.Location Order By Deaths.Date) As RollingCountVacci
From SQLPROJECT1..CovidDeaths23 As Deaths
Join SQLPROJECT1..CovidVaccinations23 As Vacci
on Deaths.Location=Vacci.Location
And Deaths.Date=Vacci.Date
Where Deaths.Continent is not null

