# Motor-Vehicle-Thefts

Business Case: The New Zealand police department plans to release a public service announcement to encourage citizens to be aware of thefts and stay safe, the requirement is to dig into the stolen vehicles database to find details on when, which and where vehicles are most likely to be stolen.

-- Objective 1: Identify when vehicles are likely to be stolen
-- 1. Find the number of vehicles stolen each year
SELECT YEAR(date_stolen), COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY YEAR(date_stolen);

-- 2. Find the number of vehicles stolen each month
SELECT YEAR(date_stolen), MONTH(date_stolen), COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY YEAR(date_stolen), MONTH(date_stolen)
ORDER BY YEAR(date_stolen), MONTH(date_stolen);

SELECT YEAR(date_stolen), MONTH(date_stolen), DAY(date_stolen), COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
WHERE MONTH(date_stolen) = 4
GROUP BY YEAR(date_stolen), MONTH(date_stolen),DAY(date_stolen)
ORDER BY YEAR(date_stolen), MONTH(date_stolen),DAY(date_stolen);

-- 3. Find the number of vehicles stolen each day of the week
SELECT DAYOFWEEK(date_stolen) AS dow, COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY DAYOFWEEK(date_stolen)
ORDER BY dow;

-- 4. Replace the numeric day of week values with the full name of each day of the week (Sunday, Monday, Tuesday, etc.)
SELECT 	DAYOFWEEK(date_stolen) AS dow,
		CASE 	WHEN DAYOFWEEK(date_stolen) = 1 THEN 'Sunday'
				WHEN DAYOFWEEK(date_stolen) = 2 THEN 'Monday'
				WHEN DAYOFWEEK(date_stolen) = 3 THEN 'Tuesday'
                WHEN DAYOFWEEK(date_stolen) = 4 THEN 'Wenesday'
                WHEN DAYOFWEEK(date_stolen) = 5 THEN 'Thursday'
                WHEN DAYOFWEEK(date_stolen) = 6 THEN 'Friday'
                ELSE 'Saturday' END AS day_of_week,
		COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY DAYOFWEEK(date_stolen), day_of_week
ORDER BY dow;

-- Objective 2: Identify which vehicles are likely to be stolen

-- 1. Find the vehicle types that are most often and least often stolen
SELECT vehicle_type, COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY vehicle_type
ORDER BY num_vehicles DESC
LIMIT 5;

SELECT vehicle_type, COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY vehicle_type
ORDER BY num_vehicles ASC
LIMIT 5;

-- 2. For each vehicle type, find the average age of the cars that are stolen
SELECT vehicle_type, AVG(YEAR(date_stolen) - model_year) AS avg_age
FROM stolen_vehicles
GROUP BY vehicle_type
ORDER BY avg_age DESC;

-- 3. For each vehicle type, find the percent of vehicles stolen that are luxury versus standard
WITH luxury_standard AS
(SELECT 	vehicle_type, 
		CASE WHEN make_type = 'Luxury' THEN 1
        ELSE 0 END AS luxury,
        1 AS all_cars
FROM	stolen_Vehicles sv LEFT JOIN make_details md
		ON sv.make_id = md.make_id)
SELECT vehicle_type, SUM(luxury) / SUM(all_cars) * 100 AS pct_lux
FROM luxury_standard
GROUP BY	vehicle_type
ORDER BY	pct_lux DESC;

-- 4. Create a table where the rows represent the top 10 vehicle types, the columns represent the top 7 vehicle colors (plus 1 column for all other colors) and the values are the number of vehicles stolen
SELECT color, COUNT(vehicle_id) AS num_vehicles
FROM stolen_vehicles
GROUP BY color
ORDER BY num_vehicles DESC
LIMIT 7;

SELECT	vehicle_type, COUNT(vehicle_id) AS num_vehicles,
		SUM(CASE WHEN color = 'Silver' THEN 1 ELSE 0 END) AS silver,
		SUM(CASE WHEN color = 'White' THEN 1 ELSE 0 END) AS white,
        SUM(CASE WHEN color = 'Black' THEN 1 ELSE 0 END) AS black,
        SUM(CASE WHEN color = 'Blue' THEN 1 ELSE 0 END) AS blue,
        SUM(CASE WHEN color = 'Red' THEN 1 ELSE 0 END) AS red,
        SUM(CASE WHEN color = 'Grey' THEN 1 ELSE 0 END) AS grey,
        SUM(CASE WHEN color = 'Green' THEN 1 ELSE 0 END) AS green,
        SUM(CASE WHEN color IN ('Gold','Brown','Yellow','Orange','Purple','Cream','Pink') THEN 1 ELSE 0 END) AS other
FROM	stolen_vehicles
GROUP BY vehicle_type
ORDER BY num_vehicles DESC
LIMIT 10;        
-- Objective 3: Identify where vehicles are likely to be stolen

-- 1. Find the number of vehicles that were stolen in each region
SELECT	region, COUNT(vehicle_id) AS num_vehicles
FROM 	stolen_vehicles 
JOIN locations ON locations.location_id = stolen_vehicles.location_id
GROUP BY region
ORDER BY num_vehicles;
        
-- 2. Combine the previous output with the population and density statistics for each region
SELECT	region, COUNT(vehicle_id) AS num_vehicles, population, density
FROM 	stolen_vehicles 
JOIN locations ON locations.location_id = stolen_vehicles.location_id
GROUP BY region, population, density
ORDER BY num_vehicles;

-- 3. Do the types of vehicles stolen in the three most dense regions differ from the three least dense regions?
SELECT	density, vehicle_type, COUNT(vehicle_id) AS num_vehicles
FROM 	stolen_vehicles 
JOIN locations ON locations.location_id = stolen_vehicles.location_id
GROUP BY density, vehicle_type
ORDER BY 	density DESC,
			num_vehicles DESC
LIMIT 3;

-- RESULT 1
-- 343.09	Saloon	327
-- 343.09	Stationwagon	306
-- 343.09	Hatchback	296

SELECT	density, vehicle_type, COUNT(vehicle_id) AS num_vehicles
FROM 	stolen_vehicles 
JOIN locations ON locations.location_id = stolen_vehicles.location_id
GROUP BY density, vehicle_type
ORDER BY 	density ASC,
			num_vehicles DESC
LIMIT 3;

-- RESULT 2
-- 3.28	Stationwagon	9
-- 3.28	Saloon	7
-- 3.28	Roadbike	2

-- => Nearly the same
