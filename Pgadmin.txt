
create table covid_analysis.covid_death(
iso_code char(50) ,
continent char(50),
location char(100),
date date,
population bigint,
total_cases int,
new_cases int,
new_cases_smoothed numeric,
total_deaths int,
new_deaths int,
new_deaths_smoothed numeric,
total_cases_per_million numeric,
new_cases_per_million numeric,
new_cases_smoothed_per_million numeric,
total_deaths_per_million numeric,
new_deaths_per_million numeric,
new_deaths_smoothed_per_million numeric,
reproduction_rate numeric,
icu_patients int,
icu_patients_per_million numeric,
hosp_patients int,
hosp_patients_per_million numeric,
weekly_icu_admissions int,
weekly_icu_admissions_per_million numeric,
weekly_hosp_admissions int,
weekly_hosp_admissions_per_million numeric
	)
	
select * from covid_analysis.covid_death;

create table covid_analysis.covid_vaccinations (
	iso_code char(50),
	continent char(50),
	location char(100),
	date date,
	new_tests bigint ,
	total_tests bigint,
	total_tests_per_thousand numeric ,
	new_tests_per_thousand numeric,
	new_tests_smoothed bigint,
	new_tests_smoothed_per_thousand  numeric,
	positive_rate numeric,
	tests_per_case numeric,
	tests_units char(100) ,
	total_vaccinations bigint ,
	people_vaccinated bigint,
	people_fully_vaccinated bigint ,
	total_boosters bigint ,
	new_vaccinations bigint ,
	new_vaccinations_smoothed bigint,
	total_vaccinations_per_hundred numeric,
	people_vaccinated_per_hundred numeric,
	people_fully_vaccinated_per_hundred numeric,
	total_boosters_per_hundred numeric,
	new_vaccinations_smoothed_per_million numeric,
	new_people_vaccinated_smoothed bigint,
	new_people_vaccinated_smoothed_per_hundred numeric,
	stringency_index numeric,
	population_density numeric,
	median_age numeric,
	aged_65_older numeric,
	aged_70_older numeric,
	gdp_per_capita numeric,
	extreme_poverty numeric,
	cardiovasc_death_rate numeric,
	diabetes_prevalence numeric,
	female_smokers numeric,
	male_smokers numeric,
	handwashing_facilities numeric,
	hospital_beds_per_thousand numeric,
	life_expectancy numeric,
	human_development_index numeric,
	excess_mortality_cumulative_absolute numeric,
	excess_mortality_cumulative numeric,
	excess_mortality numeric,
	excess_mortality_cumulative_per_million numeric
	)

select  * 
from covid_analysis.covid_vaccinations
order by 3,4;


--Exploring Covid Death table

select location, date, total_cases, new_cases, total_deaths, population 
from covid_analysis.covid_death
order by 1,2;

--Looking at Total deaths vs total cases by each location
select location, date, total_cases as total_cases, total_deaths as total_deaths, 
(total_deaths)/(total_cases)*100 as death_percentage
from covid_analysis.covid_death
where location like '%India%' or location like '%States%'
order by 1,2;
--Observation 1: 
-- So in the United States currently, there is 1.5% chances (likelihood) you could die of covid, 
--which is much less than 6% during May
--Observation 2:
--In India currently, there is 1.38% chance (likelihood) you can die of covid,
--which is much less than 3.6% during April (small percentage number due to large population - lets explore this)


--Looking at Total cases vs Population
select location, date, total_cases as Totalcases, population, 
(total_cases)/(population)*100 as population_affected
from covid_analysis.covid_death
where location like '%India%' or location like '%States%'
order by 1,2;

--getting the maximum population affected. 
select location, date, total_cases as Totalcases, population, 
(total_cases)/(population)*100 as population_affected
from covid_analysis.covid_death
where (total_cases)/(population)*100 in (
	select max((total_cases)/(population)*100) 
 	from covid_analysis.covid_death
 	where location like '%India%'
);
--Observations: As per the query above we can see that 16.44% of US was affected and tested positiove
--In India, 2.5% was tested positive, the difference is due to large population in India. 


--which country had the most population affected
select location, date, total_cases as Totalcases, population, 
(total_cases)/(population)*100 as population_affected
from covid_analysis.covid_death
where (total_cases)/(population)*100 in (
	select max((total_cases)/(population)*100) 
 	from covid_analysis.covid_death
);
--Observation:Andorraa had the most population affected i.e. 30.69% has tested positive once for covid-19


--Grouping countries with population affected.
select location, population, max(total_cases) as Totalcases, 
max((total_cases)/(population))*100 as population_affected
from covid_analysis.covid_death
where (total_cases)/(population)*100 is not null and continent is not null
group by 1,2
order by 4 desc;
--Observation: Andorra with the most population affected. 
--Shocking: China was in the bottom 5% of the population affected. It may be due to the cases not reported.

--Locations with highest death count per population
select location, population, max(total_deaths) as total_deaths, 
max((total_deaths)/(population))*100 as death_percentage
from covid_analysis.covid_death
where (total_deaths)/(population)*100 is not null and continent is not null
	and location like '%India%' or location like '%State%' 
group by 1,2
order by 4 desc;
--Peru had the highest death rate due to covid of 0.60%
--India has a death percentage of 0.03% and United States has a death percentage of 0.24% until 31st December 2021


--Location with the highest death
select location, population, max(total_deaths) as total_deaths 
from covid_analysis.covid_death
where continent is not null and total_deaths is not null
group by 1,2
order by 3 desc;
--We see that United States,Brazil and India had the highest deaths due to pandemic

--Breaking by continents
--Trying different ways
select continent, max(total_deaths) as total_deaths
from covid_analysis.covid_death
where continent is not null
group by 1
order by 2 desc;
--select distinct continent from covid_analysis.covid_death; #Cross-check that we got everything

--Overall death percentage
select sum(new_deaths) as new_death, sum(new_cases) as new_cases,
sum(new_deaths)/sum(new_cases)*100 as death_percentage
from covid_analysis.covid_death
where continent is not null;
 --From this we can see that 1.88% of the total population globally were died due to covid


--Joining tables to compare vaccinations information
select * from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
order by 3,4;

--Looking at how many percentage of people got vaccinated in each location
select d.location,d.date, d.population, v.people_vaccinated , 
(v.people_vaccinated/d.population)*100 as population_vaccinated
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
where (v.people_vaccinated/d.population)*100 is not null
order by 1,2;

--Looking at rolling sum of vaccintions that have been given to people by different countries by date
--using 'over' to look at non aggregated values along with aggregated values
select d.location,d.date, d.population, v.new_vaccinations, 
sum(v.new_vaccinations) 
	over (partition by d.location order by d.location, d.date) as total_rolling_vaccinated
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
	on d.location = v.location and d.date = v.date
--where d.location like '%States%'
order by 1,2;


--Using CTE's to demonstrate the population vaccinated (incremental ~ rolling)
WITH pop_vac (continent, location, date, population, new_vaccinations,total_rolling_vaccinated)
as
(
select d.continent, d.location,d.date, d.population, v.new_vaccinations, 
sum(v.new_vaccinations) 
	over (partition by d.location order by d.location, d.date) as total_rolling_vaccinated
	--(total_rolling_vaccinated/population) *100
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
	on d.location = v.location and d.date = v.date
where d.continent is not null
)
select *,(total_rolling_vaccinated/population) *100 AS population_vaccinated
from pop_vac
where total_rolling_vaccinated is not null;
--Observation: we see the incremental incresase in the percentage of population vaccinated. This CTE
--would be helpful in analysing population vaccinated by date.


--creating view so that we can use this views to help with our analysis
drop view if exists covid_analysis.PercentPopulationVaccinated;
create view covid_analysis.PercentPopulationVaccinated as 
select d.continent, d.location,d.date, d.population, v.new_vaccinations, 
sum(v.new_vaccinations) 
	over (partition by d.location order by d.location, d.date) as total_rolling_vaccinated
	--(total_rolling_vaccinated/population) *100
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
	on d.location = v.location and d.date = v.date
where d.continent is not null;

--using the views we created to analyse the data.
select * 
from covid_analysis.PercentPopulationVaccinated;



--using the people_vaccinated column from covid_vaccinations table
--Percentage vaccinations per country that have already been done until 31st December 2021.
select d.location, d.continent, max(d.population) as population, max(v.people_vaccinated) as people_vaccinated, --v.new_vaccinations
max(v.people_vaccinated)/max(d.population)*100 as vaccination_percentage
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
where d.continent is not null --and d.population = 1444216102
--where d.population = 7874965730
group by 1,2;
-- by this we can see that atleast 73% of US population and 60% of Indian popultaion have atleast 
--received their one dose.



--how many people have been vaccinated globally
----------------------------------------trying sub queries
select *
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
where sum(people_vaccinated)/sum(population)*100 in (
select max(d.population) as population, max(v.people_vaccinated) as people_vaccinated, 
	max(v.people_vaccinated)/max(d.population)*100
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
	where d.continent is not null
--where d.location like '%India%' or d.location like '%tates%'
);
-------------------------------not working


drop view if exists covid_analysis.PopulationandPeopleVaccinated; 
create view covid_analysis.PopulationandPeopleVaccinated as 
(
select d.location, max(d.population) as population, max(v.people_vaccinated) as people_vaccinated, 
	max(v.people_vaccinated)/max(d.population)*100 as percentage_vaccinated --bylocation
from covid_analysis.covid_death as d
join covid_analysis.covid_vaccinations  as v
on d.location = v.location and d.date = v.date
where d.continent is not null 
group by 1
	);
--select * from covid_analysis.PopulationandPeopleVaccinated;	
select sum(people_vaccinated)/sum(population)*100 as total_world_vaccinated_percentage
from covid_analysis.PopulationandPeopleVaccinated;
--From the query above we can see that around 58.4% of the world population has been vaccinated (atleast 1 dose)

