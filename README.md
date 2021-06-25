# Datawarehouse Project
## Project description
Sparkify is a music streaming startup with a growing user base and song database.

Their user activity and songs metadata data resides in json files in S3. The goal of the project is to build an ETL pipeline that will extracts their data from S3, stages them in Redshift, and transforms data into a set of dimensional tables for their analytics team to continue finding insights in what songs their users are listening to.

## How to run?

1. To run this project you will need to fill the following information, and save it as dwh.cfg in the project root folder.

```
[CLUSTER]
HOST= 
DB_NAME=dwh
DB_USER=dwhuser
DB_PASSWORD=Passw0rd
DB_PORT=5439

[IAM_ROLE]
ARN= 

[S3]
LOG_DATA='s3://udacity-dend/log_data'
LOG_JSONPATH='s3://udacity-dend/log_json_path.json'
SONG_DATA='s3://udacity-dend/song_data'

[AWS]
KEY=
SECRET=

[DWH] 
DWH_CLUSTER_TYPE       =multi-node
DWH_NUM_NODES          =4
DWH_NODE_TYPE          =dc2.large
DWH_IAM_ROLE_NAME      =dwhRole
DWH_CLUSTER_IDENTIFIER =dwhCluster
DWH_DB                 =dwh
DWH_DB_USER            =dwhuser
DWH_DB_PASSWORD        =Passw0rd
DWH_PORT               =5439

```

2. Follow the steps in **IaC** notebook to set up the needed infrastructure for this project.

> You have to stop once reaching before deleting the cluster, you can come back to the deleting step once you finish from the project.

> This notebook is Taken from the IaC lesson in the course.

3. Run the create_tables script to set up the database staging and analytical tables

`$ python create_tables.py`

4. Finally, run the etl script to extract data from the files in S3, stage it in redshift, and finally store it in the dimensional tables.

`$ python etl.py`


## Database schema design

**State and justify your database schema design and ETL pipeline.**

### Staging Tables
* staging_events

```
CREATE TABLE staging_events(
        artist              VARCHAR,
        auth                VARCHAR,
        firstName           VARCHAR,
        gender              VARCHAR,
        itemInSession       INTEGER,
        lastName            VARCHAR,
        length              FLOAT,
        level               VARCHAR,
        location            VARCHAR,
        method              VARCHAR,
        page                VARCHAR,
        registration        FLOAT,
        sessionId           INTEGER,
        song                VARCHAR,
        status              INTEGER,
        ts                  TIMESTAMP,
        userAgent           VARCHAR,
        userId              INTEGER 
    )
```

**Copying the Staging events table way**

("""
    copy staging_events from {data_bucket}
    credentials 'aws_iam_role={role_arn}'
    region 'us-west-2' format as JSON {log_json_path}
    timeformat as 'epochmillisecs';
""").format(data_bucket=config['S3']['LOG_DATA'], role_arn=config['IAM_ROLE']['ARN'], log_json_path=config['S3']['LOG_JSONPATH'])



* staging_songs

```
 CREATE TABLE staging_songs(
        num_songs           INTEGER,
        artist_id           VARCHAR,
        artist_latitude     FLOAT,
        artist_longitude    FLOAT,
        artist_location     VARCHAR,
        artist_name         VARCHAR,
        song_id             VARCHAR,
        title               VARCHAR,
        duration            FLOAT,
        year                INTEGER
    )
```
**Copying the Staging songs table way**

("""
    copy staging_songs from {data_bucket}
    credentials 'aws_iam_role={role_arn}'
    region 'us-west-2' format as JSON 'auto';
""").format(data_bucket=config['S3']['SONG_DATA'], role_arn=config['IAM_ROLE']['ARN'])

### Fact Table
* songplays - records in event data associated with song plays i.e. records with page NextSong - songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent

```
 CREATE TABLE songplays(
        songplay_id         INTEGER         IDENTITY(0,1)   PRIMARY KEY,
        start_time          TIMESTAMP,
        user_id             INTEGER ,
        level               VARCHAR,
        song_id             VARCHAR,
        artist_id           VARCHAR ,
        session_id          INTEGER,
        location            VARCHAR,
        user_agent          VARCHAR
    )
```


### Dimension Tables
* users - users in the app - user_id, first_name, last_name, gender, level

```
    CREATE TABLE users(
        user_id             INTEGER PRIMARY KEY,
        first_name          VARCHAR,
        last_name           VARCHAR,
        gender              VARCHAR,
        level               VARCHAR
    )
```

* songs - songs in music database - song_id, title, artist_id, year, duration

```
    CREATE TABLE songs(
        song_id             VARCHAR PRIMARY KEY,
        title               VARCHAR ,
        artist_id           VARCHAR ,
        year                INTEGER ,
        duration            FLOAT
    )
```
* artists - artists in music database - artist_id, name, location, lattitude, longitude

```
CREATE TABLE artists(
        artist_id           VARCHAR  PRIMARY KEY,
        name                VARCHAR ,
        location            VARCHAR,
        latitude            FLOAT,
        longitude           FLOAT
    )
```
* time - timestamps of records in songplays broken down into specific units - start_time, hour, day, week, month, year, weekday

```
CREATE TABLE time(
        start_time          TIMESTAMP       NOT NULL PRIMARY KEY,
        hour                INTEGER         NOT NULL,
        day                 INTEGER         NOT NULL,
        week                INTEGER         NOT NULL,
        month               INTEGER         NOT NULL,
        year                INTEGER         NOT NULL,
        weekday             VARCHAR(20)     NOT NULL
    )
```