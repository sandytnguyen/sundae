# sundae
#q1
```mysql
--Top 100 Actors based on frequency in movie rental db
SELECT a.first_name
	,a.last_name
	,count(f.title) AS total_films
FROM actor a
INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
INNER JOIN film f ON fa.film_id = f.film_id
--group by actors
GROUP BY a.actor_id
--desc by frequency of occurances
ORDER BY 3 DESC LIMIT 100;
```
#q2
```mysql
--list of all films by category, desc by # of films, exclude sports and games
SELECT c.name AS category_name
	,count(f.title) AS total_films
FROM category c
INNER JOIN film_category fc ON fc.category_id = c.category_id
INNER JOIN film f ON fc.film_id = f.film_id
--exclude sports and games category
WHERE c.name <> 'Sports'
	AND c.name <> 'Games'
--group by category and then count desc
GROUP BY 1
ORDER BY 2 DESC
```

q3 1) SQL query and plot in R or 2) add a custom sql data connection in Tableau and create viz
```r
{r}
#using R Markdown, RPostgres, ggplot2
library(RPostgres)
#open db connection
con <-dbConnect(RPostgres::Postgres(), host="localhost", dbname="rentals")
```
```mysql
{sql, connection=con, output.var=”rentals_df”}
--list measures
SELECT count(DISTINCT p.customer_id) AS distinct_individuals
	,sum(p.amount) AS total_revenue
	,count(p.payment_ts) AS total_rentals
	,EXTRACT(WEEK FROM payment_ts) AS week
FROM payment p
--join tables
LEFT JOIN rental r ON r.customer_id = p.customer_id
LEFT JOIN inventory i ON i.inventory_id = r.inventory_id
LEFT JOIN store s ON s.store_id = i.store_id
--filter on store_id 1, address table not used
WHERE i.store_id = 1
--filter on p12w, based on latest rental date
	AND payment_ts > (
		SELECT max(payment_ts)
		FROM payment
		) - interval '12 weeks'
--group by week, missing weeks 24, 27, 30 and 33
GROUP BY 4
ORDER BY 4 asc’)
```
```r
{r}
#call plotting package
library(ggplot2)
#plot distinct count of individuals per week
p1<-ggplot(data=rentals_df, aes(x=week, y=distinct_individuals)) +
#since missing four weeks, showing only 8 bins. If 12 weeks, then bins=12
geom_bar(bins=8, fill="steelblue")+
  theme_minimal()
p1

#plot total revenue by week
p2<-ggplot(data=rentals_df, aes(x=week, y=total_revenue)) +
#since missing four weeks, showing only 8 bins. If 12 weeks, then bins=12
  geom_bar(bins=8, fill="steelblue")+
  theme_minimal()
p2

#plot total rentals per week
p3<-ggplot(data=rentals_df, aes(x=week, y=total_rentals)) +
#since missing four weeks, showing only 8 bins. If 12 weeks, then bins=12
  geom_bar(bins=8, fill="steelblue")+
  theme_minimal()
p3

#disconnect
dbDisconnect(con)
```

