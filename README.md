PROJECT BRIEF – 
Netflix data need to be imported from kaggle api to pandas framework and load the data to MySQL and perform data cleaning and finding below objectives.
Requirements
1.	Remove Duplicates
2.	Handling  foreign characters
3.	New table for listed in director, country, cast
4.	Populate missing values in country, duration columns
5.	Populate the rest of the nulls as not_available
6.	Drop column director, listed_in country, cast

Objective
1.	For each director count the no of movies and tv shows created by then in separate columns for directors who have  created tv shows and movies both
2.	Which country has highest  number of comedy movies
3.	For each year ( as per date added to netflix), which director has max number of movies released
4.	What is the average duration of movies in each genre
5.	Find the list of directors who have created both comedy and horror movies. Display director names along with the no. of comedy and horror movies directed by them


Solution Approach:-
1.	Generated new token and saved in the user file in C drive under .kaggle folder
2.	Workings in Jupyter notebook where we use python libraries to fetch the data from kaggle.
a.	Install kaggle (!pip install kaggle )
b.	import kaggle
c.	Download dataset using kaggle api
d.	Extract file from zip file
e.	Import pandas data frame, load & read the data
f.	Export the data into MySQL
