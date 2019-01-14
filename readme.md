# Project 1 - Logs Analysis

This program runs three reports on the news database: the Top 3 Most Popular Articles, The Top Authors by Views, and Days on Which 1% of Requests Results in a 404 Error.

# Program design

A function is defined for each of the reports.  Then, each function is called and outputs the result in plain text.

# Function for Report 1 - Top 3 Most Popular Articles

The function for Report 1 uses a single SQL query that joins the log table and articles table on articles.slug and log.path by using the CONCAT function to add "/articles/" to the articles.slug.  This is because articles.slug needs to have "/article/" appended to the beginning of the entry to match the data stored in log.path.  The query returns article.title and count(log.path) grouped by article.title ordered by views in descending order, limited by top 3.

Full query:

```sh
SELECT articles.title, count(log.path) AS views
    FROM articles
    JOIN log ON CONCAT('/article/', articles.slug) = log.path
    GROUP BY articles.title
    ORDER BY views DESC
    LIMIT 3
```

This function is called by top3articles()

# Function for Report 2 - Top Authors By Views

The function for Report 2 builds on the query in Report 1 by also joining the authors table on authors.id and articles.author.
The query returns author.name and count(log.path) grouped by the author.id ordered by views in descending order.

Full query:

```sh
SELECT authors.name, count(log.path) AS views
    FROM authors
    JOIN articles ON authors.id = articles.author
    JOIN log ON CONCAT('/article/', articles.slug) = log.path
    GROUP BY authors.id
    ORDER BY views DESC;
```

This function is called by topauthors()

# Function for Report 3 - Days with More than 1% 404 Errors

The function for Report 3 uses a single SQL query that uses a subquery to calculate the percentage of 404 requests.  This is done using the CASE expression to set a value of 1.0 (a float) or 0 to each row in the database, grouped by day.  The values are multiplied by 100 to convert from decimals to percentage.  The subquery is then queried to return only those days in which the percentage is greater than 1.

Full query:
```sh
SELECT * FROM (SELECT cast(time as date) AS day,
    (sum(CASE WHEN log.status='404 NOT FOUND'
        THEN 1.0 ELSE 0 END)
        /count(status))*100 AS percent
        FROM log GROUP BY day ORDER BY percent DESC) AS sub
    WHERE sub.percent>1;
```

This function is called by dayswith404errors()

# Calling the Functions

Each function is called and then formatted for output in plain text.


# How to run the program

Launch terminal, cd to the directory where loganalysis.py is located, and run the command

```sh
python loganalysis.py
```

The program will output the results of these reports in plain text.

# Author

Stephen Gnoza
stephen.gnoza@gmail.com
