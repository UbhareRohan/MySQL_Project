# MySQL_Project

use portfolio_projects;
Select * From portfolio_projects.coviddeaths_csv;
Select * From portfolio_projects.covidvaccinated_csv;

-- Q1  Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract covid in your country

Select Location, date, total_cases,total_deaths, round((total_deaths/total_cases)*100,3) as DeathPercentage
From portfolio_projects.coviddeaths_csv
Where location like 'India'
and continent is not null;


-- Q2 Total Cases vs Population
-- Shows what percentage of population infected with Covid in your country

Select Location, date, Population, total_cases, (total_cases/population)*100 as PercentPopulationInfected
From portfolio_projects.coviddeaths_csv
Where location like 'India'
and continent is not null;


-- Q3 Countries with Highest Infection Rate compared to Population

select Location, Population, Date, MAX(total_cases) as HighestInfectionCount,  (Max(total_cases)/population)*100 as PercentPopulationInfected
From portfolio_projects.coviddeaths_csv
Group by Location, Population
order by PercentPopulationInfected desc;

-- Q4 Countries with Highest Death Count per Population
Select Location,Population,Date, MAX(Total_deaths)  as TotalDeathCount
From portfolio_projects.coviddeaths_csv
Where continent is not null 
Group by Location, Population
order by TotalDeathCount desc;

-- highest death Percentage countries

Select Location,Population,Date, MAX(Total_deaths)  as TotalDeathCount, (MAX(Total_deaths)/population)*100 as PercentPopulationdeaths
From portfolio_projects.coviddeaths_csv
Where continent is not null 
Group by Location, Population
order by PercentPopulationdeaths desc;

-- highest death Percentage countries in India 

Select Location,Population,Date, MAX(Total_deaths)  as TotalDeathCount, (MAX(Total_deaths)/population)*100 as PercentPopulationdeaths
From portfolio_projects.coviddeaths_csv
Where Location like "India"
order by PercentPopulationdeaths desc;

-- Q5 BREAKING THINGS DOWN BY CONTINENT
-- Showing contintents with the highest death count per population

Select continent, MAX(Total_deaths)as TotalDeathCount
From portfolio_projects.coviddeaths_csv
Where continent is not null 
Group by continent
order by TotalDeathCount desc;

-- Q6) GLOBAL NUMBERS

Select SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, SUM(new_deaths)/SUM(New_Cases)*100 as DeathPercentage
From portfolio_projects.coviddeaths_csv
where continent is not null 
order by 1,2;


-- Q7)  Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date , dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated,
(SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date)/population)*100 as PercentageRollingPeopleVaccinated
From portfolio_projects.coviddeaths_csv dea
Join portfolio_projects.covidvaccinated_csv vac
	On dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
order by 2,3;

-- Using CTE to perform Calculation on Partition By in previous query

With PopVsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From portfolio_projects.coviddeaths_csv dea
Join portfolio_projects.covidvaccinated_csv vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
)
Select *, (RollingPeopleVaccinated/Population)*100 as PecentageRollingPeopleVaccinated
From PopVsVac;


-- Using View (previous query)

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From portfolio_projects.coviddeaths_csv dea
Join portfolio_projects.covidvaccinated_csv vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null;

select *, (RollingPeopleVaccinated/population)*100 from percentpopulationvaccinated;
