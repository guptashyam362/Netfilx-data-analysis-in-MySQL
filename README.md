PROJECT BRIEF – 
Netflix data need to be imported from kaggle api to pandas framework and load the data to MySQL and perform data cleaning and finding below objectives.
Requirements
1.	Remove Duplicates
2.	Handling  foreign characters
3.	New table for listed in director, country, cast
4.	Populate missing values in country, duration columns
5.	Populate the rest of the nulls as not_available
6.	Drop column director, listed_in country, cast



Solution Approach:-
1.	Generated new token and saved in the user file in C drive under .kaggle folder
2.	Workings in Jupyter notebook where we use python libraries to fetch the data from kaggle.
a.	Install kaggle (!pip install kaggle )
b.	import kaggle
c.	Download dataset using kaggle api
d.	Extract file from zip file
e.	Import pandas data frame, load & read the data
f.	Export the data into MySQL

OBJECTIVE & ANALYSIS

-- Importing data with optimised space from Python
Create table netflix(
Show_id varchar(10),
Type varchar(10),
Title varchar(200),
Director varchar(250),
Cast varchar(1000),
Country varchar (150),
Date_added varchar(20),
Release_year int,
Rating varchar(10),
Duration varchar(10),
Listed_in varchar(100),
Description varchar(500)
)
select * from netflix;

-- checking the duplicacy in title column

select * from netflix
where title in
(select title from netflix
group by title,type
having count(*)> 1);


-- Handling the duplicacy
create table netflix_rest as (
with cte as 
(select *, 
row_number() over (partition by title, type order by show_id) as rn 
from netflix)

select show_id,type,title,
str_to_date(date_added, '%M %D %Y') as date_added,release_year,rating,
case when duration is null then rating else duration end as duration,
description
from cte);

select * from netflix_rest
-- where rn = 1;



-- new table for listed in, director, country , cast
create table Netflix_director as (
SELECT show_id, trim(value) AS N_direct
from Netflix,
     JSON_TABLE(
         CONCAT('["', REPLACE(REPLACE(director, '"', '\\"'), ',', '","'), '"]'),
         '$[*]' COLUMNS(value VARCHAR(255) PATH '$')
     ) AS jt
     );

create table Netflix_genre as (
SELECT show_id, trim(value) AS genre
from Netflix,
     JSON_TABLE(
         CONCAT('["', REPLACE(REPLACE(listed_in, '"', '\\"'), ',', '","'), '"]'),
         '$[*]' COLUMNS(value VARCHAR(255) PATH '$')
     ) AS jt
     );
     
     
create table Netflix_country as (
SELECT show_id, trim(value) AS country
from Netflix,
     JSON_TABLE(
         CONCAT('["', REPLACE(REPLACE(country, '"', '\\"'), ',', '","'), '"]'),
         '$[*]' COLUMNS(value VARCHAR(255) PATH '$')
     ) AS jt
     );
     
create table Netflix_cast as (
SELECT show_id, trim(value) AS cast
from Netflix,
     JSON_TABLE(
         CONCAT('["', REPLACE(REPLACE(cast, '"', '\\"'), ',', '","'), '"]'),
         '$[*]' COLUMNS(value VARCHAR(255) PATH '$')
     ) AS jt
     );
     
-- populate missing values in country, duration column
select * from netflix
where country is null;

select * from netflix where director='Julien Leclercq';

Insert into netflix_country (
select show_id,M.country from netflix as N
inner join
(select n_direct,country from netflix_country as nc
inner join netflix_director as nd on 
nd.show_id=nc.show_id 
group by n_direct,nc.country) as M on
M.n_direct=N.director
where N.country is null
);


select * from netflix where duration is null;

-- 1.	For each director count the no of movies and tv shows created by them in separate columns for directors who have  created tv shows and movies both
select nd.N_direct, count(distinct nr.type) as cnt, 
count(distinct case when nr.type = 'movie' then nr.show_id end) as movie_cnt,
count(distinct case when nr.type = 'tv show' then nr.show_id end) as tv_cnt
 from netflix_rest as nr
inner join netflix_director as nd on
nd.show_id=nr.show_id
group by nd.N_direct
having cnt > 1
order by cnt desc;

-- 2.	Which country has highest  number of comedy movies 
select country, genre, count(ng.show_id) as no_of_movies, nr.type from netflix_country as nc
inner join netflix_genre as ng on 
nc.show_id =ng.show_id 
inner join netflix_rest as nr on
ng.show_id = nr.show_id
where ng.genre = 'comedies' and nr.type = 'movie'
group by country, genre, type 
order by no_of_movies desc limit 1

3.	For each year ( as per date added to netflix), which director has max number of movies released
with cte as
(
select distinct year(date_added) as add_year, nd.n_direct, count(nd.show_id) as no_of_movies from netflix_rest as nr
inner join netflix_director as nd on
nd.show_id=nr.show_id
where type = 'movie'
group by add_year,nd.n_direct
), 
cte2 as 
(
select *, row_number() over (partition by add_year order by no_of_movies desc,n_direct) as rn from cte
-- order by no_of_movies desc
)
select * from cte2 
where rn = 1
order by no_of_movies desc
;
4. What is the average duration of movies in each genre
with cte as
(
select nr.show_id,nr.type,ng.genre,
cast(replace(duration,' min', '') as unsigned) as duration_int from netflix_rest as nr
inner join netflix_genre as ng on
nr.show_id=ng.show_id
where nr.type = 'movie'
order by genre
)
select genre, avg(duration_int) from cte
group by genre


-- 5.	Find the list of directors who have created both comedy and horror movies. Display director names along with the no. of comedy and horror movies directed by them
select distinct genre from netflix_genre
where genre like '%comed%' -- '%horror%' 

select n_direct,
count(case when genre = 'comedies' then nr.show_id end) as no_comedy,
count(case when genre ='horror movies' then nr.show_id end) as no_horror from netflix_director as nd 
inner join netflix_genre as ng on
nd.show_id = ng.show_id
inner join netflix_rest as nr on 
nr.show_id = nd.show_id
where ng.genre in ('comedies', 'horror movies') and nr.type = 'movie'
group by nd.n_direct
having count( distinct ng.genre) =2
order by no_comedy desc, no_horror desc
