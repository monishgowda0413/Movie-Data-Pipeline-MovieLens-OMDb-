# Movie-Data-Pipeline-MovieLens-OMDb-
Built an end-to-end ETL pipeline using Python to ingest, enrich, and analyze MovieLens data with OMDb API integration. Designed normalized SQL schema and executed analytical queries for movie insights.
Overview:

This project implements an end-to-end data engineering pipeline that extracts movie and rating data from the MovieLens dataset, enriches each movie’s metadata using the OMDb API, and loads the structured, normalized data into a SQLite database for analysis.

The ETL process (Extract–Transform–Load) was designed to demonstrate:

Data ingestion and cleaning

API integration with key rotation

Relational data modeling and normalization

Analytical reporting using SQL

Final analytical outputs include:

Highest-rated movie

Top 5 genres by average rating

Director with the most movies

Year-wise average rating trend
# Movie Data Pipeline (MovieLens + OMDb)

## What this does
- Ingests MovieLens `movies.csv` and `ratings.csv`
- Enriches each movie with OMDb (rotating multiple API keys)
- Loads into SQLite with idempotent inserts
- Provides SQL answers to 4 analytical questions


## Setup
>cmd<
cd %USERPROFILE%\Desktop\movie-pipeline
py -m venv .venv
.\.venv\Scripts ctivate
pip install -r requirements.txt


##Download MovieLens Dataset

Download ml-latest-small from
🔗 https://grouplens.org/datasets/movielens/

Extract it (you’ll have movies.csv and ratings.csv).
Example path:

C:\Users\<YourName>\Downloads\ml-latest-small

##OMDB API Key 
The OMDb API is used to fetch extra movie information such as the director’s name, plot summary, and box office collections.
	-API Website: http://www.omdbapi.com/
	-You will need to generate a free API key from their website.

## Run(your data path, DB, and OMDb keys)
>cmd<
set DATA_DIR=C:\path\to\ml-latest-small
set DBURL=sqlite:///movies.db
set OMDBKEYS=key1,key2,...,key10

python etl.py --data-dir "%DATA_DIR%" --schema "schema.sql" --db-url "%DBURL%" --omdb-keys "%OMDBKEYS%" --limit 50
python etl.py --data-dir "%DATA_DIR%" --schema "schema.sql" --db-url "%DBURL%" --omdb-keys "%OMDBKEYS%"


## Query
Open `movies.db` in **DB Browser for SQLite** → **Execute SQL** → run `queries.sql`.



>>>>Design Choices and Assumptions:

1. SQLite Database

-Chosen for simplicity and ease of local setup (no server needed).

-Allows SQL analytics and easy inspection via DB Browser.

2. Normalized Schema

-Separate tables for movies, genres, directors, and ratings.

-Many-to-many relationships handled via bridge tables (movie_genres, movie_directors).

3. API Key Rotation

-OMDb limits 1000 calls per day per key.

-The script rotates through 20 API keys to maximize throughput (≈20k/day).

4. Idempotent Inserts

-Uses INSERT OR IGNORE so re-running the ETL never duplicates rows.

5. Resumable Design

-Movies already enriched are skipped; pipeline can resume anytime.

6. Retries and Graceful Handling

-Handles transient OMDb connection failures automatically.

-Continues enrichment without breaking the pipeline.


## Challenges and Solutions:
          Challenge	                                                           How It Was Solved
1. API rate limit (1000/day per key)	                      Implemented key rotation across 20 keys, allowing 20k requests/day.
2. Network drops (ConnectionResetError)	                      Added retry mechanism with exponential backoff to handle transient failures.
3. Inconsistent movie titles between MovieLens and OMDb	      Added title cleaning (removed punctuation, retried without year).
4. Duplicate records during re-runs	                      Used INSERT OR IGNORE and primary keys to ensure idempotency.
5. Long enrichment time for 10k movies	                      Added --limit parameter to control batch size and resume in chunks.


##Output Tables:
            Table	                      Description
           1.movies	           Base movie list with cleaned titles and release year
           2.ratings	           User ratings (userId, movieId, rating, timestamp)
           3.genres	           Unique genres
           4.movie_genres	   Movie-genre relationship
           5.omdb	           Enriched movie data from OMDb API
           6.directors	           Unique directors
           7.movie_directors	   Movie-director relationship


##Analytical Queries:
1. Highest average rated movie
SELECT m.title, AVG(r.rating) AS avg_rating
FROM ratings r JOIN movies m ON r.movie_id=m.movie_id
GROUP BY r.movie_id ORDER BY avg_rating DESC LIMIT 1;

 2. Top 5 genres by rating
SELECT g.name, ROUND(AVG(r.rating),2) AS avg_rating
FROM ratings r
JOIN movie_genres mg ON r.movie_id=mg.movie_id
JOIN genres g ON mg.genre_id=g.genre_id
GROUP BY g.name ORDER BY avg_rating DESC LIMIT 5;

3. Director with most movies
SELECT d.name, COUNT(*) AS total_movies
FROM movie_directors md JOIN directors d ON md.director_id=d.director_id
GROUP BY d.name ORDER BY total_movies DESC LIMIT 1;

4. Average rating per year
SELECT m.release_year, ROUND(AVG(r.rating),2) AS avg_rating
FROM movies m JOIN ratings r ON m.movie_id=r.movie_id
GROUP BY m.release_year ORDER BY m.release_year;



##Learning Outcomes:

Working on this project helped me understand how a complete data pipeline actually works from start to finish.
I learned how to extract data from raw CSV files, clean and transform it into a structured format, and load it into a database for analysis.
While integrating the OMDb API, I got hands-on experience with handling API rate limits, rotating multiple keys, and managing connection errors in a safe way.

I also learned how to design a proper SQL data model using normalization techniques and how to write analytical queries to get meaningful insights from the data.
Overall, this project gave me a strong understanding of how data engineering concepts are applied in real-world scenarios — especially in building scalable, reliable ETL workflows.


##
