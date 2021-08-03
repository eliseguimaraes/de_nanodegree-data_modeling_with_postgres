# Project: Data Modeling with Postgres


## Purpose

This project consists of an ETL Pipeline that creates and populates a Postgres database optimized for Sparkify's analytics.

Sparkify's current analytics team is particularly interested in understanding songs users are listening to. Although they do hold a history of songplays, this data is spread across several files, which are not optimized for analysis. 

By designing a database from scratch with the analytics goals in mind (and automatically populating it), we were able to optimize table format for the queries the team is interested in running, therefore making this data readily available for the startup's business purposes. 

## Technical choices

### Database schema design

The database was designed in a star schema fashion. Since Sparkify's current analytics goal is to understand which songs were played, a song plays table was created and chosen to be the Fact Table - the central table of our star schema.

* Fact Table
    * songplays - songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent
* Dimension Tables
    * users -user_id, first_name, last_name, gender, level
    * songs - song_id, title, artist_id, year, duration
    * artists - artist_id, name, location, latitude, longitude
    * time - start_time, hour, day, week, month, year, weekday

With this design, the team can define metrics based on the events described by our songplays table (the fact table), while qualifying them (filtering, grouping, etc) with the dimension tables.


### ETL pipeline

The ETL pipeline was created to automate the analytics database population with data from the JSON files.

The pipeline does the following actions:
* Process all song_data files
    * Extract song data and insert on the songs table
    * Extract artist data and insert on the artists table
* Process all log files
    * Filter by "NextSong" logs - records associated with song plays
    * Extract and process time data and insert on the time table
    * Extract user data and insert on the users table
    * Extract songplay records, enriches the data with artist and song ids - useful for future join operations - and insert on the songplays table


This pipeline was entirely built using python. This technology was chosen mostly for leveraging implementation time - with pre-built modules such as psycopg2 and pandas, which make connection with the PostgreSQL database and data processing much easier. The data volume in this project did not represent an impediment for performance.


## Example analysis

1. Total time played

    Query: `SELECT SUM(s.duration) FROM songplays sp INNER JOIN songs s on s.song_id = sp.song_id `
    
    Result: None* 
2. Most frequent user by songs played

    Query: `SELECT user_id, COUNT(songplay_id) FROM songplays sp GROUP BY user_id ORDER BY 2 DESC LIMIT 5`
    
    Result:
    ![](https://i.imgur.com/6u7D4MN.png)

3. Most popular artist

    Query: `SELECT a.name as "Artist", COUNT(songplay_id) FROM songplays sp INNER JOIN artists a ON a.artist_id = sp.artist_id GROUP BY a.name ORDER BY 2 DESC LIMIT 5`
    
    Result: None* 

4. Most active days of week

    Query: `SELECT t.weekday as "weekday", COUNT(songplay_id) FROM songplays sp INNER JOIN time t ON t.start_time = sp.start_time GROUP BY t.weekday ORDER BY 2 DESC LIMIT 5`
    
    Result:
    ![](https://i.imgur.com/B9q84KL.png)
    
    
 * The data presented in the JSON files had no matches between songplay logs and song data files
