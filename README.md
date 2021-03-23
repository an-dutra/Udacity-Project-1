# Sparkify Songs Analysis

Sparkify is a music streaming startup, and currently have songs and logs data into json files.
Their analytics team is interested in understanding what songs users are listening to but since data is in json files the don't have an easy way to query data. This project was model to attend this demand and make Sparkify analysis easier 

## Source data
The data is stored in json format and distributed in 2 datasets Songs and Logs.

### Song Dataset
Song dataset are stored in `data/song_data`, in the following layout

```javascript
{
    "num_songs": 1,
    "artist_id": "AR7G5I41187FB4CE6C",
    "artist_latitude": null,
    "artist_longitude": null,
    "artist_location": "London, England",
    "artist_name": "Adam Ant",
    "song_id": "SONHOTT12A8C13493C",
    "title": "Something Girls",
    "duration": 233.40363,
    "year": 1982
}
```

#### Logs dataset
Logs dataset are stored in `data/log_data` will contain a list of files with following layout:

```javascript
{
    "artist": null,
    "auth": "Logged In",
    "firstName": "Walter",
    "gender": "M",
    "itemInSession": 0,
    "lastName": "Frye",
    "length": null,
    "level": "free",
    "location": "San Francisco-Oakland-Hayward, CA",
    "method": "GET",
    "page": "Home",
    "registration": 1540919166796.0,
    "sessionId": 38,
    "song": null,
    "status": 200,
    "ts": 1541105830796,
    "userAgent": "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36\"",
    "userId": "39"
}
```

## ETL Flow
Our solution will extract data using python and save it in a Star Schema in Postgre, to create tables and database you can execute `create_tables.py`, it will drop and recreate tables, the tables layout are below.

### Schema

#### Table Songplays
```sql
songplays (
	songplay_id,
	start_time,
	user_id,
	level,
	song_id,
	artist_id,
	session_id,
	location,
	user_agent
);
```


#### Table Artists
```sql
artists (
	artist_id,
	name,
	location,
	latitude,
	longitude
);
```

#### Table Songs
```sql
songs (
    song_id,
    title,
    artist_id,
    year,
    duration
);

```
#### Table Time
```sql
time (
    time_id,
    start_time,
    hour,
    day,
    week,
    month,
    year,
    weekday
);

```
#### Table Users
```sql
users (
    user_id,
    first_name,
    last_name,
    gender,
    level
);
```

### Processing
The process run two flows one to read each dataset and save data on tables. All data are processed in `etl.py` script.

#### Song Dataset
To load the song dataset in our schema it will execute the function `process_song_file` on each .json file located on the `data/song_data` folder. This function will extract data for the the `artists` and `songs` data, and load it into artists and songs tables.

##### Logs dataset
The `process_log_file` function will extract data from json in `data/log_data`, split information to load remaining tables from schema.
It will filter the column `page == 'Next Song'` to guarantee to load only listening sound data, then save data into `users` and based on timestamps in ts it will generate data to load `time` table.
Finally `songplays` table it will be loaded with data joined for all tables, the process will join currently dataframe (generated by extract from log_data) with `artists` and `songs` to generate our fact data, and load it.
