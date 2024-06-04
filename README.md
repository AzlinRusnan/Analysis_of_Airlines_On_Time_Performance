# <div align="center">ANALYSIS OF AIRLINES PERFORMANCES🛬</div>
## <div align="center">![Intro](images/iStock-498532108-916x517-1.jpg)

## Introduction
This report provides an analysis of flight delays and cancellations for an airline. The goal is to understand the key factors causing these issues and suggest ways to improve. By examining various data points, we aim to provide clear insights and actionable recommendations to enhance flight punctuality and reliability. First, we cover the process of uploading a data file to HDFS, followed by analyzing the data using Pig, and finally, visualizing the results with Python packages.

## Problem Statement
Frequent flight delays and cancellations affect the airline's operations and passenger satisfaction. Identifying the primary reasons behind these disruptions and finding solutions to minimize them is crucial for improving service quality and operational efficiency.

## Objectives:
1) Identify the main causes of flight delays and cancellations.

2) Determine the optimal times of the day, days of the week, and months of the year to minimize delays and cancellations.

3) Provide actionable recommendations to improve flight punctuality and reliability.

## Dataset 📖
The source of the data is from [Airline On Time Data](https://www.kaggle.com/datasets/wenxingdi/data-expo-2009-airline-on-time-data/data?select=1993.csv)
### **There are 4 main datasets that we will use:**
- 2008.csv 
- plane-data 
- carriers 
- airports

### **The main questions of interest in the dataset:**
- What are the optimal times of day, days of the week, and times of the year for minimizing flight delays?
- What are the primary factors contributing to flight delays?
- What factors predominantly lead to flight cancellations?
- Which flight experiences the most frequent and significant delays and cancellations?

We will answer all the questions using Pig

## Methodology 🚀
### **Uploading Datasets to Hadoop File System**

Here are the steps to upload the datasets (2008, plane-data, carriers, and airports) to the Hadoop File System:

**Step 1:** Transfer File to Virtual Machine

**Command Prompt:**
>![Command Prompt](images/commandprompt.png)

Explanation:

In this step, the pscp command (PuTTY Secure Copy) is used to transfer the 2008.csv file from your local Windows machine (specified by the user file path) to the home directory of the user maria_dev on the virtual local machine. The file is now accessible in the virtual machine but not yet in the Hadoop file system.

**Step 2:** Upload File to Hadoop File System

**Putty:**
>![Putty](images/putty.png)

Explanation:

Here, the hdfs dfs -put command is used to move the 2008.csv file from the local file system of the virtual machine (where it resides in /home/maria_dev/) to the Hadoop Distributed File System (HDFS). The file is uploaded to the directory /user/maria_dev/flight_data/ in HDFS, making it available for processing by Hadoop.

### **Learning Data Manipulation Using Pig**

### 1. **Data Loading:**
   
- Load datasets containing flight records, airport information, carrier details, and plane data into PIG.

```pig
------------------ Load all the file -------------

-- Load the flight record data
FLIGHT_RECORD = LOAD '/user/maria_dev/flight_data/2008.csv' 
USING PigStorage(',') 
AS (Year:int, Month:int, DayOfMonth:int, DayOfWeek:int, DepTime:int, 
    CRSDepTime:int, ArrTime:int, CRSArrTime:int, UniqueCarrier:chararray, 
    FlightNum:int, TailNum:chararray, ActualElapsedTime:int, 
    CRSElapsedTime:int, AirTime:int, ArrDelay:int, DepDelay:int, 
    Origin:chararray, Dest:chararray, Distance:int, TaxiIn:int, 
    TaxiOut:int, Cancelled:int, CancellationCode:chararray, 
    Diverted:int, CarrierDelay:int, WeatherDelay:int, 
    NASDelay:int, SecurityDelay:int, LateAircraftDelay:int);
--DUMP FLIGHT_RECORD

-- Load the airports data
AIRPORTS = LOAD '/user/maria_dev/flight_data/airports.csv' 
USING PigStorage(',') 
AS (iata:chararray, airport:chararray, city:chararray, state:chararray, country:chararray, lat:double, longitude:double);
--DUMP AIRPORTS;

-- Load the carriers data
CARRIERS = LOAD '/user/maria_dev/flight_data/carriers.csv' 
USING PigStorage(',') 
AS (Code:chararray, Description:chararray);
--DUMP CARRIERS;

-- Load the plane data
PLANES = LOAD '/user/maria_dev/flight_data/plane-data.csv' 
USING PigStorage(',') 
    AS (tailnum:chararray, type:chararray, manufacturer:chararray, 
    issue_date:chararray, model:chararray, status:chararray, 
    aircraft_type:chararray,engine_type:chararray,year:int);
--DUMP PLANES;
```

### 2. **Data Joining:**
```pig
-- Join FLIGHT_RECORD with AIRPORTS on Origin
FLIGHT_AIRPORTS_ORIGIN = JOIN FLIGHT_RECORD BY Origin LEFT, AIRPORTS BY iata;

-- Project the required fields
FLIGHT_AIRPORTS_ORIGIN_PROJECTED = FOREACH FLIGHT_AIRPORTS_ORIGIN GENERATE
    FLIGHT_RECORD::Year, FLIGHT_RECORD::Month, FLIGHT_RECORD::DayofMonth, FLIGHT_RECORD::DayOfWeek,
    FLIGHT_RECORD::DepTime, FLIGHT_RECORD::CRSDepTime, FLIGHT_RECORD::ArrTime, FLIGHT_RECORD::CRSArrTime,
    FLIGHT_RECORD::UniqueCarrier, FLIGHT_RECORD::FlightNum, FLIGHT_RECORD::TailNum, FLIGHT_RECORD::ActualElapsedTime,
    FLIGHT_RECORD::CRSElapsedTime, FLIGHT_RECORD::AirTime, FLIGHT_RECORD::ArrDelay, FLIGHT_RECORD::DepDelay,
    FLIGHT_RECORD::Origin, FLIGHT_RECORD::Dest, FLIGHT_RECORD::Distance, FLIGHT_RECORD::TaxiIn, FLIGHT_RECORD::TaxiOut,
    FLIGHT_RECORD::Cancelled, FLIGHT_RECORD::CancellationCode, FLIGHT_RECORD::Diverted, FLIGHT_RECORD::CarrierDelay,
    FLIGHT_RECORD::WeatherDelay, FLIGHT_RECORD::NASDelay, FLIGHT_RECORD::SecurityDelay, FLIGHT_RECORD::LateAircraftDelay,
    AIRPORTS::iata AS Origin_iata, AIRPORTS::airport AS Origin_airport, AIRPORTS::city AS Origin_city, AIRPORTS::state AS Origin_state,
    AIRPORTS::country AS Origin_country, AIRPORTS::lat AS Origin_lat, AIRPORTS::long AS Origin_long;

-- Join the result with AIRPORTS on Dest
FLIGHT_AIRPORTS = JOIN FLIGHT_AIRPORTS_ORIGIN_PROJECTED BY Dest LEFT, AIRPORTS BY iata;

-- Project the required fields
FLIGHT_AIRPORTS_PROJECTED = FOREACH FLIGHT_AIRPORTS GENERATE
    Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime, CRSArrTime, UniqueCarrier, FlightNum, TailNum,
    ActualElapsedTime, CRSElapsedTime, AirTime, ArrDelay, DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut, Cancelled,
    CancellationCode, Diverted, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay, Origin_iata,
    Origin_airport, Origin_city, Origin_state, Origin_country, Origin_lat, Origin_long, AIRPORTS::iata AS Dest_iata,
    AIRPORTS::airport AS Dest_airport, AIRPORTS::city AS Dest_city, AIRPORTS::state AS Dest_state, AIRPORTS::country AS Dest_country,
    AIRPORTS::lat AS Dest_lat, AIRPORTS::long AS Dest_long;

-- Join the result with CARRIERS on UniqueCarrier
FLIGHT_AIRPORTS_CARRIERS = JOIN FLIGHT_AIRPORTS_PROJECTED BY UniqueCarrier LEFT, CARRIERS BY Code;

-- Project the required fields
FLIGHT_AIRPORTS_CARRIERS_PROJECTED = FOREACH FLIGHT_AIRPORTS_CARRIERS GENERATE
    Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime, CRSArrTime, UniqueCarrier, FlightNum, TailNum,
    ActualElapsedTime, CRSElapsedTime, AirTime, ArrDelay, DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut, Cancelled,
    CancellationCode, Diverted, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay, Origin_iata,
    Origin_airport, Origin_city, Origin_state, Origin_country, Origin_lat, Origin_long, Dest_iata, Dest_airport,
    Dest_city, Dest_state, Dest_country, Dest_lat, Dest_long, CARRIERS::Code, CARRIERS::Description;

-- Join the result with PLANES on TailNum
FINAL_RESULT = JOIN FLIGHT_AIRPORTS_CARRIERS_PROJECTED BY TailNum LEFT, PLANES BY tailnum;

-- Project the required fields
FINAL_PROJECTED = FOREACH FINAL_RESULT GENERATE
    Year, Month, DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime, CRSArrTime, UniqueCarrier, FlightNum, TailNum,
    ActualElapsedTime, CRSElapsedTime, AirTime, ArrDelay, DepDelay, Origin, Dest, Distance, TaxiIn, TaxiOut, Cancelled,
    CancellationCode, Diverted, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay, Origin_iata,
    Origin_airport, Origin_city, Origin_state, Origin_country, Origin_lat, Origin_long, Dest_iata, Dest_airport,
    Dest_city, Dest_state, Dest_country, Dest_lat, Dest_long, Code, Description, PLANES::tailnum, PLANES::type,
    PLANES::manufacturer, PLANES::issue_date, PLANES::model, PLANES::status, PLANES::aircraft_type, PLANES::engine_type, PLANES::year;
-- Store the final result
STORE FINAL_PROJECTED INTO '/user/maria_dev/flight_data/joined_data3' USING PigStorage(',');

```
### 3. **Data Analysis:**
   
**Q1. What are the optimal times of day, days of the week, and times of the year for minimizing flight delays?**

```pig
-- Extract hour from DepTime
DATA_WITH_HOUR = FOREACH DATA GENERATE *, (DepTime / 100) AS DepHour:int;

-- Group by DepHour and calculate average arrival delay
HOUR_DELAYS = GROUP DATA_WITH_HOUR BY DepHour;
AVG_HOUR_DELAYS = FOREACH HOUR_DELAYS GENERATE group AS DepHour, AVG(DATA_WITH_HOUR.ArrDelay) AS AvgArrDelay;
STORE AVG_HOUR_DELAYS INTO '/user/maria_dev/flight_data/results/hourly_delays' USING PigStorage(',');

-- Group by DayOfWeek and calculate average arrival delay
WEEKDAY_DELAYS = GROUP DATA BY DayOfWeek;
AVG_WEEKDAY_DELAYS = FOREACH WEEKDAY_DELAYS GENERATE group AS DayOfWeek, AVG(DATA.ArrDelay) AS AvgArrDelay;
STORE AVG_WEEKDAY_DELAYS INTO '/user/maria_dev/flight_data/results/weekday_delays' USING PigStorage(',');

-- Group by Month and calculate average arrival delay
MONTH_DELAYS = GROUP DATA BY Month;
AVG_MONTH_DELAYS = FOREACH MONTH_DELAYS GENERATE group AS Month, AVG(DATA.ArrDelay) AS AvgArrDelay;
STORE AVG_MONTH_DELAYS INTO '/user/maria_dev/flight_data/results/monthly_delays' USING PigStorage(',');
```

**Q2. What are the primary factors contributing to flight delays?**
```pig
-- Calculate average delay contributions for each factor
CARRIER_DELAY = FOREACH (GROUP DATA ALL) GENERATE AVG(DATA.CarrierDelay) AS CarrierDelay;
WEATHER_DELAY = FOREACH (GROUP DATA ALL) GENERATE AVG(DATA.WeatherDelay) AS WeatherDelay;
NAS_DELAY = FOREACH (GROUP DATA ALL) GENERATE AVG(DATA.NASDelay) AS NASDelay;
SECURITY_DELAY = FOREACH (GROUP DATA ALL) GENERATE AVG(DATA.SecurityDelay) AS SecurityDelay;
LATE_AIRCRAFT_DELAY = FOREACH (GROUP DATA ALL) GENERATE AVG(DATA.LateAircraftDelay) AS LateAircraftDelay;

-- Combine all delay factors into a single tuple
COMBINED_DELAYS = CROSS CARRIER_DELAY, WEATHER_DELAY, NAS_DELAY, SECURITY_DELAY, LATE_AIRCRAFT_DELAY;
FINAL_DELAYS = FOREACH COMBINED_DELAYS GENERATE
    CARRIER_DELAY.CarrierDelay AS CarrierDelay,
    WEATHER_DELAY.WeatherDelay AS WeatherDelay,
    NAS_DELAY.NASDelay AS NASDelay,
    SECURITY_DELAY.SecurityDelay AS SecurityDelay,
    LATE_AIRCRAFT_DELAY.LateAircraftDelay AS LateAircraftDelay;

STORE FINAL_DELAYS INTO '/user/maria_dev/flight_data/results/primary_delay_factors' USING PigStorage(',');
```

**Q3. What factors predominantly lead to flight cancellations?**
```pig
-- Filter out invalid cancellation codes
VALID_CANCELLATIONS = FILTER DATA BY (CancellationCode IS NOT NULL AND CancellationCode != '' AND CancellationCode != 'CancellationCode');

-- Group by CancellationCode and calculate the number of cancellations
CANCELLATIONS = GROUP VALID_CANCELLATIONS BY CancellationCode;
COUNT_CANCELLATIONS = FOREACH CANCELLATIONS GENERATE group AS CancellationCode, COUNT(VALID_CANCELLATIONS) AS Count;

STORE COUNT_CANCELLATIONS INTO '/user/maria_dev/flight_data/results/cancellation_factors' USING PigStorage(',');
```

**Q4. Which flight experiences the most frequent and significant delays and cancellations?**
```pig
-- Group by FlightNum and calculate total arrival delay
FLIGHT_DELAYS = GROUP DATA BY FlightNum;
TOTAL_FLIGHT_DELAYS = FOREACH FLIGHT_DELAYS GENERATE group AS FlightNum, SUM(DATA.ArrDelay) AS TotalArrDelay, COUNT(DATA) AS FlightCount;

-- Find the flight with the maximum total arrival delay
MAX_DELAY = ORDER TOTAL_FLIGHT_DELAYS BY TotalArrDelay DESC;
MOST_DELAYED_FLIGHT = LIMIT MAX_DELAY 1;
STORE MOST_DELAYED_FLIGHT INTO '/user/maria_dev/flight_data/results/most_delayed_flight' USING PigStorage(',');

-- Group by FlightNum and calculate total cancellations
FLIGHT_CANCELLATIONS = GROUP DATA BY FlightNum;
TOTAL_FLIGHT_CANCELLATIONS = FOREACH FLIGHT_CANCELLATIONS GENERATE group AS FlightNum, SUM(DATA.Cancelled) AS TotalCancellations;

-- Find the flight with the maximum total cancellations
MAX_CANCELLATIONS = ORDER TOTAL_FLIGHT_CANCELLATIONS BY TotalCancellations DESC;
MOST_CANCELLED_FLIGHT = LIMIT MAX_CANCELLATIONS 1;
STORE MOST_CANCELLED_FLIGHT INTO '/user/maria_dev/flight_data/results/most_cancelled_flight' USING PigStorage(',');
```
## Findings and Visualizations 📊

**Q1. What are the optimal times of day, days of the week, and times of the year for minimizing flight delays?**

