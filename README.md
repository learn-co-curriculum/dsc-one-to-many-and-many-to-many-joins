
# One-to-Many and Many-to-Many Joins

## Introduction

Previously, you've learned about the typical case where one joins on a primary or foreign key. In this section, you'll explore other types of joins using One-to-Many and Many-to-many relationships!

## Objectives

You will be able to:

- Explain why Join Tables are needed in Many-to-Many relationships

## One-to-Many and Many-to-Many relationships

We've looked at a couple kinds of different join statements: left joins and inner joins. Both of these refer to the way in which we would like to define our joins based on the tables and their shared information. Another perspective on this is the number of matches between the tables based on our defined links with the keywords *on* or *using*.
  
We've investigated the typical case where one joins on a primary or foreign key. For example, when we join on customerID or employeeID, this value should be unique to that table. As such, our joins have been very similar to using a dictionary to find additional information associated with that record. In cases where there are multiple entries, in either table, for the field you are joining on, you will similarly be given multiple rows in your resulting view, one for each of these entries.  
  
For example, let's say you have another table 'restaurants' that has many columns including *name*, *city*, and *rating*. If you were to join this table with the offices table using the shared city column, you might get some unexpected behavior. That is, in the office table, there is only one office per city. However, because there is apt to be more then one restaurant for each of these cities in our second table, we will get unique combinations of Offices and Restaurants from our join. If there are 513 restaurants for Boston in our restaurant table and 1 office for Boston, our joined table will have each of these 513 rows, one for each restaurant along with the one office.

If we had 2 offices for Boston, and 513 restaurants, our join would have 1026 rows for boston; 513 for each restuarant along with the first office and 513 for each restaurant with the second office. Three offices in Boston would similarly produce 1539 rows; one for each unique combination of restaurants and offices. This is where you should be particularly careful of many to many joins as the resulting set size can explode drastically potentially consuming vast amounts of memory and other resources.  

<img src='Database-Schema.png' width=550>

## Connecting to the Database


```python
import sqlite3
import pandas as pd
```


```python
conn = sqlite3.connect('data.sqlite', detect_types=sqlite3.PARSE_COLNAMES)
cur = conn.cursor()
```

## Checking Sizes of Resulting Joins...

### The original tables...


```python
cur.execute('select * from offices;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```


```python
cur.execute('select * from employees;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

### A One-to-One Join...


```python
cur.execute('select * from offices join employees using(officeCode);')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

### A One-to-Many Join
Here we join products with product lines. There are only a few product lines that will be matched to each product. As a result, the product line descriptions will be repeated in our resulting view.


```python
cur.execute('select * from products;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```


```python
cur.execute('select * from productlines;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```


```python
cur.execute("""select * from products
                      join productlines
                      using(productLine);""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

### A Many-to-Many Join

If we join the employees and offices table, we will have a view with repeat cities listed.
(Recall this was 23 rows, one for each employee. Joining this with the customer table on the cities column will cause us to have a huge number of rows, one for each employee and customer combination for a given city.)


```python
cur.execute("""select * from employees
                        join offices
                        using(officeCode)
                        join customers
                        using(city);""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```


```python
cur.execute("""select * from employees;""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```


```python
cur.execute("""select * from customers;""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

## Summary

In this section, you expanded your Join knowledge to One-to-Many and Many-to-many Joins!
