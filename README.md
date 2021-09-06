# Implemented ETL for Data Collection from ACS APIs 

Name: Cathy Chen

Andrew ID: qiyangch

## Introduction

This document helps with data extraction, transform, and load of American Community Survey (ACS)  from [U.S. Census Bureau’s API](https://www.census.gov/data/developers.html). The ACS was built in 2005, which produces survey-based estimates and provides a great variety of social, economic, demographic, and housing information to the public. 



## Overview

- Get data: select a US state and download the latest ACS data for every block group in that state.
- Transform data: clean the data and get it ready to be loaded into a postgres database.
- Load data: connect the postgres database and load the data into a postgres database table.



### Get Data

In order to pull data from ACS data,  the package ***census*** is used, which allows the users to contain an easy access to  [U.S. Census Bureau’s API.](https://www.census.gov/data/developers.html) Specifically, the package provides a class for census geographies and functions to obtain information about different variables and tables. You can learn more about this package from [here.](https://jtleider.github.io/censusdata/)

- **Census Geographies**: representing the targeted census geographies.

  *class* `censusdata.censusgeo.censusgeo`(*geo*, *name=''*)

  ```python
  censusgeo([('state', '19'), ('county', '*'), ('block group', '*')], 'all block group and all county in Iowa') # Represents the Census geography for Iowa.
  ```

-  **Census Variables**；showing variable information about census variables. 

  *You can learn more about the census variables from [here.](https://api.census.gov/data/2019/acs/acs5/variables.html)*

  `censusdata.variable_info.censustable`(*src*, *year*, *table*)

  ```python
  censusdata.censustable('acs5', 2019, 'B23025_002E') # Returns information on table B23025 (Employment Status for Labor Force) from the ACS 2019 5-year estimates.
  
  ```

- **Download Data**: downloading lists of geographies and data from the Census API.

  `censusdata.download.download`(*src*, *year*, *geo*, *var*, *key=None*, *tabletype='detail'*, *endpt=''*)

  ```python
  censusdata.download('acs5', 2019,censusdata.censusgeo([('state', '19'), ('county', '*'),
  ('block group', '*')]), ['B23025_002E', 'B15002_011E', 'B15002_016E', 'B25075_002E', 'B25075_010E'])
  # Download ACS 2015-2019 5-year estimates for Iowa on labor force, male high school graduate over 25 years old, male master graduates over 25 years old, total value less than $10,000, and total value between $50,000 and $59,999. 
  
  ```

  To summarize, I collect the ACS data in Iowa and explore the variables related to labor force, male education status over 25 years old (high school graduates and master graduates), and the total value (less than $10,000 and $50,000-$59,999). I wonder if the male education status, total value, and identification of labor force varies in Iowa's different block groups or counties. If the male education status and total value will affect the estimates of labor force?

  ![image-20210905232302660](C:\Users\Cathy Chen\AppData\Roaming\Typora\typora-user-images\image-20210905232302660.png)

---

### Transform Data

The main tasks in transforming data:

1. Clean the data of census geographies
   - convert the index containing census geographies information in to a column
   - change the data type of census geographies to string
   - split the census geographies in to *block group, census tract, county, and state*
   - remove the unnecessary information in *block group, census tract*
   - remove the space in *block group, census tract, county, and state*

2. rename the variables and reorder the columns to be more understandable 

---

### Load Data

The package *psycopg2* is adopted to load the collected data into postgres database. This package enables the users to pass python parameters to SQL. You can learn more about this package from [here](https://www.psycopg.org/docs/usage.html#passing-parameters-to-sql-queries).

- `connect()`: creates a new database session and returns a new connection 
- `cursor()`: execute database commands and queries
- `execute()`: send commands to the database
- `commit()`: terminate the transactions

```python
conn = psycopg2.connect(host = 'xxxxxx',
                        port = 'xxxx',
                        database = 'xxxxxx',
                        user = 'xxxxxx',
                        password = 'xxxxx')

cur = conn.cursor()
cur.execute(xxx) # create the table and insert the information
conn.commit()
cur.close()
conn.close()
```

The main tasks in loading data:

1. create two variables *create_table* and *insert_data* for the execute commands.
   - *create_table* helps with building the table in the postgres database
   - *insert_data* includes each line of the data, which loops through each row and each column in the dataset
2. build the connection to the postgres database by psycopg2
   - use `connect()`, `cursor()`, `execute()`, `commit()` to connect, load data into, and disconnect the database