# sqlalchemy-challenge

## Part 1: Analyze and Explore the Climate Data

**Database Setup and Data Retrieval**:
1. Import the necessary libraries and set up the Matplotlib style for visualizations.
2. Use the SQLAlchemy `create_engine()` function to connect to SQLite database.
3.  Use the SQLAlchemy `automap_base()` function to reflect tables into classes.
4.  Save references to the classes `station` and `measurement`.

**Precipitation Analysis**:
Starting from the most recent data point in the database and calculating the date one year from the last date in the data set. Retrieve the precipitation scores for the date range.
```
most_recent_date = session.query(Measurement.date).order_by(Measurement.date.desc()).first()[0]
one_year_ago = (dt.datetime.strptime(most_recent_date, '%Y-%m-%d') - dt.timedelta(days=365)).strftime('%Y-%m-%d')
precipitation_data = session.query(Measurement.date, Measurement.prcp).filter(Measurement.date >= one_year_ago).all()
```
![image](https://github.com/Glowary/sqlalchemy-challenge/assets/141696007/f425c14d-7088-4d4f-b2fd-e2a56237185a)

**Station Analysis**
Find the total number of stations, identify the most active station, and calculate the max/min/average temperature for the most active station.
```
most_active_stations_query = session.query(Measurement.station, func.count(Measurement.station).label('observation_count')).group_by(Measurement.station).order_by(func.count(Measurement.station).desc())
temperature_stats_query = session.query(func.min(Measurement.tobs), func.max(Measurement.tobs), func.avg(Measurement.tobs)).filter(Measurement.station == most_active_station_id)
tobs_data_query = session.query(Measurement.tobs).filter(Measurement.station == most_active_station_id).filter(Measurement.date >= one_year_ago)
```
![image](https://github.com/Glowary/sqlalchemy-challenge/assets/141696007/b3bb1d3c-e1ce-4023-b036-e8f784cbdd4a)

## Part 2: Design Climate App

### Precipitation Data

 - **Route:** `/api/v1.0/precipitation`
 - **Description:** This route provides precipitation data for the last 12 months from the most recent date in the database.
 - **Process:** Calculate the year before the last date, filter, and sort the data based on the date. Then, create a dictionary where the date is the key and the precipitation is the value.
```
begin_date = dt.date(2017, 8, 23) - dt.timedelta(days=365) results = session.query(Measurement.date, Measurement.prcp).filter(Measurement.date >= begin_date).order_by(Measurement.date).all() precipitation = [] for date, prcp in results: precipitation_dict = { date : prcp } precipitation.append(precipitation_dict)
```
### List of Stations

- **Route:** `/api/v1.0/stations`
- **Description:** This route returns a list of weather stations along with their name, latitude, longitude, and elevation.
- **Process:** Create a list of dictionaries from the query station data.

### Temperature Data of the Most Active Station

- **Route:** `/api/v1.0/tobs`
- **Description:** This route retrieves temperature data for the most active weather station for the last 12 months.
- **Process:** Find the most active station and calculate the date one year from the last date. Then, create a list of dictionaries from the query temperature observations of the most active station.
```
most_active_station = session.query(Measurement.station, func.count(Measurement.station)).group_by(Measurement.station).order_by(func.count(Measurement.station).desc()).first()
results = session.query(Measurement.date, Measurement.tobs).filter(Measurement.station == most_active_station_id).filter(Measurement.date >= one_year_ago).all()
```
### Temperature Data on Specific Date or Date Range 

- **Route:** `/api/v1.0/temp/YYYY-MM-DD`
- **Description:** This route calculates temperature statistics for a date or range.
- **Process:** Query the database to calculate the max, min, and average temperature for the specified date range. Then, create a dictionary from the row data and append it to a list of the temperature data.
```
results = session.query(func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)).filter(Measurement.date >= start).filter(Measurement.date <= end).all()
```
**Reference**  
[strftime()](https://www.w3schools.com/python/gloss_python_date_strftime.asp)  
[describe()](https://www.w3schools.com/python/pandas/ref_df_describe.asp)  
[conditionally filter](https://stackoverflow.com/questions/31063860/conditionally-filtering-in-sqlalchemy)
