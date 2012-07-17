# Google BigQuery + Github Archive

[Google BigQuery](https://developers.google.com/bigquery/) is a web service that lets you do interactive analysis of massive datasets—up to billions of rows.

The Github Activity stream is automatically uploaded to BigQuery sevice to enable interactive analysis. Follow the [instructions to access the dataset](http://www.githubarchive.org/).

## Sample Queries

Have a clever query you would like to share? Fork the project, add it to the project under **queries/name.sql** and send a pull request!

```sql
/* distribution of different events on GitHub */
SELECT type, count(type) as cnt
FROM [githubarchive:github.timeline]
GROUP BY type
ORDER BY cnt DESC

/* distribution of different events on GitHub for Ruby */
SELECT type, count(type) as cnt
FROM [githubarchive:github.timeline]
WHERE repository_language="Ruby"
GROUP BY type
ORDER BY cnt DESC

/* watches for a specific language + date range */
SELECT repository_name, count(repository_name) as watches, repository_description, repository_url
FROM [githubarchive:github.timeline]
WHERE type="WatchEvent"
	AND repository_language="Ruby"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_name, repository_description, repository_url
ORDER BY watches DESC

/* top 100 repos for Ruby by number of pushes */
SELECT repository_name, count(repository_name) as pushes, repository_description, repository_url
FROM [githubarchive:github.timeline]
WHERE type="PushEvent"
	AND repository_language="Ruby"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_name, repository_description, repository_url
ORDER BY pushes DESC
LIMIT 100

/* push events by language */
SELECT repository_language, count(repository_language) as pushes
FROM [githubarchive:github.timeline]
WHERE type="PushEvent"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_language
ORDER BY pushes DESC

/* show recent push events for Go, sorted by time */
SELECT repository_name, repository_watchers, url, PARSE_UTC_USEC(created_at) as date
FROM [githubarchive:github.timeline]
WHERE type="PushEvent"
	AND repository_language="Go"
	AND repository_watchers > 1
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
ORDER BY date DESC

/* fork events source repos to target repos, sorted by time */
SELECT CONCAT(repository_owner, CONCAT('/', repository_name)) as Source, 
	CONCAT(actor_attributes_login, CONCAT('/', repository_name)) Target, 
	created_at as Date
FROM [githubarchive:github.timeline]
WHERE repository_private="false"
	AND type="ForkEvent"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-07-15 00:00:00')
GROUP BY source, target, date
ORDER BY date;

/* Time elapsed from last push to the fork of the repos,
   for repos not pushed after the fork,
   sorted by elapsed time. */
SELECT CONCAT(repository_owner, CONCAT('/', repository_name)) as source, 
	CONCAT(actor_attributes_login, CONCAT('/', repository_name)) target,
	repository_language as lang,
	(PARSE_UTC_USEC(created_at) - PARSE_UTC_USEC(repository_pushed_at)) as elapsed_time
FROM [githubarchive:github.timeline]
WHERE repository_private="false"
  AND type="ForkEvent"
GROUP BY source, target, lang, elapsed_time
HAVING elapsed_time > 0
ORDER BY elapsed_time;

/* Repos migrated to github, sorted by time elapsed from the last push */
SELECT CONCAT(repository_owner, CONCAT('/', repository_name)) as repos,
	repository_created_at, 
	repository_pushed_at,
	repository_language as lang,
	MAX((PARSE_UTC_USEC(repository_created_at) - PARSE_UTC_USEC(repository_pushed_at))) as deathtime
FROM [githubarchive:github.timeline]
WHERE repository_private="false" AND repository_fork="false" AND repository_language="Ruby"
GROUP BY repos, repository_created_at, repository_pushed_at, lang
HAVING deathtime > 0
ORDER BY deathtime;
```

For full schema of available fields to select, order, and group by, see schema.js.
