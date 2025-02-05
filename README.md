### Homework Kestra Docker Compose Setup
###  This project is a Docker Compose configuration for setting up a Kestra instance along with a PostgreSQL database and supporting services like pgAdmin for database management.

## Project Structure
Copy
Edit
homework_kestra/
├── docker-compose.yaml
Services
This configuration includes the following services:

## 1. Postgres
This service sets up a PostgreSQL database for Kestra.

Image: postgres
Volumes:
postgres-data:/var/lib/postgresql/data
Environment Variables:
POSTGRES_DB: kestra
POSTGRES_USER: kestra
POSTGRES_PASSWORD: k3str4
Healthcheck:
Verifies if the PostgreSQL database is ready by running pg_isready.
## 2. Kestra
This service runs Kestra, a powerful workflow orchestration tool.

Image: kestra/kestra:latest
Command: server standalone
Volumes:
kestra-data:/app/storage
/var/run/docker.sock:/var/run/docker.sock
/tmp/kestra-wd:/tmp/kestra-wd
Environment Variables:
Datasources:
PostgreSQL database configuration
Kestra Settings:
Basic Auth disabled
Repository type set to PostgreSQL
Storage set to local
Queue set to PostgreSQL
Ports:
8080:8080
8081:8081
Depends On:
postgres: Ensures PostgreSQL is running before starting Kestra.
## 3. Postgres Zoomcamp
This service sets up another PostgreSQL database for Zoomcamp.

Image: postgres
Environment Variables:
POSTGRES_USER: kestra
POSTGRES_PASSWORD: k3str4
POSTGRES_DB: postgres-zoomcamp
Ports:
5432:5432
Depends On:
kestra: Ensures Kestra is running before starting the database.
## 4. pgAdmin
This service provides a web-based UI for managing PostgreSQL databases.

Image: dpage/pgadmin4
Environment Variables:
PGADMIN_DEFAULT_EMAIL: admin@admin.com
PGADMIN_DEFAULT_PASSWORD: root
Ports:
8085:80
Depends On:
postgres_zoomcamp: Ensures the Zoomcamp PostgreSQL service is running before starting pgAdmin.
Volumes
The following volumes are defined for persistent data storage:

postgres-data: Used by the PostgreSQL database for Kestra.
kestra-data: Used by Kestra to store its data.
zoomcamp-data: Used by the PostgreSQL database for Zoomcamp.
How to Run
Prerequisites:
Docker and Docker Compose installed on your system.
Steps:
Clone or download the repository.
Navigate to the project directory:
bash
Copy
Edit
cd homework_kestra
Run the Docker Compose setup:
bash
Copy
Edit
docker-compose up
Accessing Services:
Kestra UI: http://localhost:8080
pgAdmin UI: http://localhost:8085
Login with:
Email: admin@admin.com
Password: root
Notes
This setup uses a root user for Kestra for development purposes. For a production environment, consider using a rootless setup. For more details, refer to Kestra's installation documentation.
Ensure that your local environment allows Docker to run properly with sufficient permissions.

KESTRA OCHESTRATION WORK FLOW

# Kestra Workflow: 02_postgres_taxi_scheduled

This is a Kestra workflow designed to process NYC TLC taxi data and load it into PostgreSQL databases. It provides two workflows: one for yellow taxis and one for green taxis. The workflow pulls CSV files from a public source, processes them, and loads them into two separate tables for each taxi type.

## Description
This workflow downloads and processes CSV data related to taxi trips and loads it into PostgreSQL. The CSV files are available for both yellow and green taxis, and the data is organized in separate staging and target tables in PostgreSQL. This workflow includes tasks like downloading, extracting, and transforming data before loading it into the database.

### CSV Source
- The data is sourced from the DataTalksClub's GitHub repository: [NYC TLC Data](https://github.com/DataTalksClub/nyc-tlc-data/releases).

### Input Options
- **Taxi Type**: The user can choose between `yellow` or `green` taxi data for processing.

## Workflow Tasks

1. **Set Label**: Adds a label for tracking the execution, including the taxi type and file name.
2. **Extract**: Downloads and unzips the relevant CSV file for the selected taxi type.
3. **If Yellow Taxi**:
   - Creates tables for yellow taxi data in PostgreSQL.
   - Loads data into the staging table and merges it into the target table.
4. **If Green Taxi**:
   - Creates tables for green taxi data in PostgreSQL.
   - Loads data into the staging table and merges it into the target table.
5. **Purge Files**: Cleans up the downloaded files to avoid cluttering the storage.

## PostgreSQL Database

This workflow uses two PostgreSQL databases:
- **Postgres Database for Yellow Taxi Data**: Contains the main table `public.yellow_tripdata` and staging table `public.yellow_tripdata_staging`.
- **Postgres Database for Green Taxi Data**: Contains the main table `public.green_tripdata` and staging table `public.green_tripdata_staging`.

## Scheduling

The workflow is triggered by scheduled cron jobs:
- **Yellow Taxi Data**: Triggered at `10:00 AM` on the 1st of every month.
- **Green Taxi Data**: Triggered at `9:00 AM` on the 1st of every month.

## Environment Variables
The following environment variables are used in the configuration:
- **POSTGRES_USER**: Username for the PostgreSQL database.
- **POSTGRES_PASSWORD**: Password for the PostgreSQL user.
- **POSTGRES_DB**: Database name to use in PostgreSQL.

## Dependencies

- **Kestra**: Orchestrates the data processing workflow.
- **PostgreSQL**: Stores the processed data in tables.
- **Wget**: Used to download the CSV files.
- **Gunzip**: Unzips the downloaded files.

## Usage

To run the workflow, ensure that the following services are up and running:
1. **Kestra**: Ensure the Kestra service is running.
2. **PostgreSQL**: Make sure the PostgreSQL databases are set up and running.

You can trigger the workflow manually or it will be automatically triggered based on the defined schedule.

### Example Trigger:
```yaml
triggers:
  - id: green_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 1 * *"
    inputs:
      taxi: green

  - id: yellow_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 10 1 * *"
    inputs:
      taxi: yellow

###### HOMEWORK QUESTIONS
#### QUESTION 1 
Within the execution for Yellow Taxi data for the year 2020 and month 12: what is the uncompressed file size (i.e. the output file yellow_tripdata_2020-12.csv of the extract task)?
128.3 MB
134.5 MB
364.7 MB
692.6 MB

#### SOLUTION QUESTION 1

NO CODE NEEDED ANSWER  134.5 MB

#### QUESTION 2

What is the rendered value of the variable file when the inputs taxi is set to green, year is set to 2020, and month is set to 04 during execution?
{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv
green_tripdata_2020-04.csv
green_tripdata_04_2020.csv
green_tripdata_2020.csv

#### SOLUTION QUESTION 2 

green_tripdata_2020-04.csv


#### QUESTION 3
#### SOLUTION QUESTION 3 
SELECT COUNT(*)
FROM yellow_tripdata
WHERE lpep_pickup_datetime >= '2020-01-01 00:00:00'
  AND lpep_pickup_datetime < '2021-01-01 00:00:00';

#### QUESTION 4
How many rows are there for the Green Taxi data for all CSV files in the year 2020?
5,327,301
936,199
1,734,051
1,342,034

#### SOLUTION QUESTION 4

SELECT COUNT(*)
FROM green_tripdata
WHERE lpep_pickup_datetime >= '2020-01-01 00:00:00'
  AND lpep_pickup_datetime < '2021-01-01 00:00:00';

#### QUESTION 5

How many rows are there for the Yellow Taxi data for the March 2021 CSV file?
1,428,092
706,911
1,925,152
2,561,031


#### SOLUTION QUESTION 5

SELECT COUNT(*)
FROM yellow_tripdata
WHERE lpep_pickup_datetime >= '2021-03-01 00:00:00'

#### QUESTION 6

How would you configure the timezone to New York in a Schedule trigger?
Add a timezone property set to EST in the Schedule trigger configuration
Add a timezone property set to America/New_York in the Schedule trigger configuration
Add a timezone property set to UTC-5 in the Schedule trigger configuration
Add a location property set to New_York in the Schedule trigger configuration

#### SOLUTION QUESTION 6

Add a timezone property set to America/New_York in the Schedule trigger configuration
