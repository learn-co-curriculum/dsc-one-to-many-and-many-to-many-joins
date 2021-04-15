# One-to-Many and Many-to-Many Joins

## Introduction

Previously, you learned about the typical case where one joins on a primary or foreign key. In this section, you'll explore other types of joins using one-to-many and many-to-many relationships!

## Objectives

You will be able to:

* Explain one-to-many and many-to-many joins as well as implications for the size of query results
* Query data using one-to-many and many-to-many joins

## One-to-Many and Many-to-Many Relationships

So far, you've seen a couple of different kinds of join statements: `LEFT JOIN` and `INNER JOIN` (aka, `JOIN`). Both of these refer to the way in which you would like to define your join based on the tables and their shared information. Another perspective on this is the number of matches between the tables based on your defined links with the keywords `ON` or `USING`.
  
You have also seen the typical case where one joins on a primary or foreign key. For example, when you join on `customerID` or `employeeID`, this value should be unique to that table. As such, your joins have been very similar to using a dictionary to find additional information associated with that record. In cases where there are multiple entries in either table for the field (column) you are joining on, you will similarly be given multiple rows in your resulting view, one for each of these entries.

### Restaurants Case Study

We'll start with this familiar ERD:

<img src='images/Database-Schema.png' width=550>

For example, let's say you have another table `restaurants` that has many columns including `name`, `city`, and `rating`. If you were to join this `restaurants` table with the offices table using the shared city column, you might get some unexpected behavior. That is, in the office table, there is only one office per city. However, because there will likely be more than one restaurant for each of these cities in your second table, you will get unique combinations of offices and restaurants from your join. If there are 513 restaurants for Boston in your restaurant table and 1 office for Boston, your joined table will have each of these 513 rows, one for each restaurant along with the one office.

If you had 2 offices for Boston and 513 restaurants, your join would have 1026 rows for Boston; 513 for each restaurant along with the first office and 513 for each restaurant with the second office. Three offices in Boston would similarly produce 1539 rows; one for each unique combination of restaurants and offices. This is where you should be particularly careful of many to many joins as the resulting set size can explode drastically, potentially consuming vast amounts of memory and other resources.  

## Connecting to the Database


```python
import sqlite3
import pandas as pd
```


```python
conn = sqlite3.connect('data.sqlite')
```

## One-to-One Relationships

Sometimes, a `JOIN` does not increase the number of records at all. For example, in our database, each employee is only associated with one office. So if our original query included information about all employees with a `jobTitle` of "Sales Rep", then our joined query also added the city location of their offices, we would get the same number of results both times.

### Sales Rep Employees


```python
q = """
SELECT firstName, lastName, email
FROM employees
WHERE jobTitle = 'Sales Rep'
;
"""
df = pd.read_sql(q, conn)
print("Number of results:", len(df))
```

    Number of results: 17



```python
# Displaying only 5 for readability
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>firstName</th>
      <th>lastName</th>
      <th>email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Leslie</td>
      <td>Jennings</td>
      <td>ljennings@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Leslie</td>
      <td>Thompson</td>
      <td>lthompson@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Julie</td>
      <td>Firrelli</td>
      <td>jfirrelli@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Steve</td>
      <td>Patterson</td>
      <td>spatterson@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Foon Yue</td>
      <td>Tseng</td>
      <td>ftseng@classicmodelcars.com</td>
    </tr>
  </tbody>
</table>
</div>



### Cities for Sales Rep Employees

Now we'll join with the `offices` table in order to display the city name as well.


```python
q = """
SELECT firstName, lastName, email, city
FROM employees
JOIN offices
    USING(officeCode)
WHERE jobTitle = 'Sales Rep'
;
"""
df = pd.read_sql(q, conn)
print("Number of results:", len(df))
```

    Number of results: 17



```python
# Displaying only 5 for readability
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>firstName</th>
      <th>lastName</th>
      <th>email</th>
      <th>city</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Leslie</td>
      <td>Jennings</td>
      <td>ljennings@classicmodelcars.com</td>
      <td>San Francisco</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Leslie</td>
      <td>Thompson</td>
      <td>lthompson@classicmodelcars.com</td>
      <td>San Francisco</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Julie</td>
      <td>Firrelli</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>Boston</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Steve</td>
      <td>Patterson</td>
      <td>spatterson@classicmodelcars.com</td>
      <td>Boston</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Foon Yue</td>
      <td>Tseng</td>
      <td>ftseng@classicmodelcars.com</td>
      <td>NYC</td>
    </tr>
  </tbody>
</table>
</div>



As you can see, we got the same number of results as the original query, we just have more data now.

## One-to-Many Relationships

When there is a one-to-many relationship, the number of records will increase to match the number of records in the larger table.

### Product Lines

Let's start with selecting the product line name and text description for all product lines.


```python
q = """
SELECT productLine, textDescription
FROM productlines
;
"""
df = pd.read_sql(q, conn)
print("Number of results:", len(df))
```

    Number of results: 7



```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>productLine</th>
      <th>textDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Motorcycles</td>
      <td>Our motorcycles are state of the art replicas ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Planes</td>
      <td>Unique, diecast airplane and helicopter replic...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ships</td>
      <td>The perfect holiday or anniversary gift for ex...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Trains</td>
      <td>Model trains are a rewarding hobby for enthusi...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Trucks and Buses</td>
      <td>The Truck and Bus models are realistic replica...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Vintage Cars</td>
      <td>Our Vintage Car models realistically portray a...</td>
    </tr>
  </tbody>
</table>
</div>



### Joining with Products

Now let's join that table with the products table, and select the vendor and product description.


```python
q = """
SELECT productLine, textDescription, productVendor, productDescription
FROM productlines
JOIN products
    USING(productLine)
;
"""
df = pd.read_sql(q, conn)
print("Number of results:", len(df))
```

    Number of results: 110



```python
# Displaying only 5 for readability
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>productLine</th>
      <th>textDescription</th>
      <th>productVendor</th>
      <th>productDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td>Autoart Studio Design</td>
      <td>Hood, doors and trunk all open to reveal highl...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td>Carousel DieCast Legends</td>
      <td>Features include opening and closing doors. Co...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td>Carousel DieCast Legends</td>
      <td>The operating parts of this 1958 Chevy Corvett...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td>Carousel DieCast Legends</td>
      <td>This diecast model of the 1966 Shelby Cobra 42...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td>Classic Metal Creations</td>
      <td>1957 die cast Corvette Convertible in Roman Re...</td>
    </tr>
  </tbody>
</table>
</div>



As you can see, the number of records has increased significantly, because the same product line is now appearing multiple times in the results, once for each actual product.

## Many-to-Many Relationships

A many-to-many join is as it sounds; there are multiple entries for the shared field in both tables. While somewhat contrived, we can see this through the example below, joining the offices and customers table based on the state field. For example, there are 2 offices in MA and 9 customers in MA. Joining the two tables by state will result in 18 rows associated with MA; one for each customer combined with the first office, and then another for each customer combined with the second option. This is not a particularly useful join without applying some additional aggregations or pivots, but can also demonstrate how a poorly written query can go wrong. For example, if there are a large number of occurrences in both tables, such as tens of thousands, then a many-to-many join could result in billions of resulting rows. Poorly conceived joins can cause a severe load to be put on the database, causing slow execution time and potentially even tying up database resources for other analysts who may be using the system.

### Just Offices


```python
q = """
SELECT *
FROM offices
;
"""

df = pd.read_sql(q, conn)
print('Number of results:', len(df))
```

    Number of results: 8


### Just Customers


```python
q = """
SELECT *
FROM customers
;
"""

df = pd.read_sql(q, conn)
print('Number of results:', len(df))
```

    Number of results: 122


### Joined Offices and Customers


```python
q = """
SELECT *
FROM offices
JOIN customers
    USING(state)
;
"""

df = pd.read_sql(q, conn)
print('Number of results:', len(df))
```

    Number of results: 254



```python
# Displaying only 5 for readability
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>officeCode</th>
      <th>city</th>
      <th>phone</th>
      <th>addressLine1</th>
      <th>addressLine2</th>
      <th>state</th>
      <th>country</th>
      <th>postalCode</th>
      <th>territory</th>
      <th>customerNumber</th>
      <th>...</th>
      <th>contactLastName</th>
      <th>contactFirstName</th>
      <th>phone</th>
      <th>addressLine1</th>
      <th>addressLine2</th>
      <th>city</th>
      <th>postalCode</th>
      <th>country</th>
      <th>salesRepEmployeeNumber</th>
      <th>creditLimit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>100 Market Street</td>
      <td>Suite 300</td>
      <td>CA</td>
      <td>USA</td>
      <td>94080</td>
      <td>NA</td>
      <td>124</td>
      <td>...</td>
      <td>Nelson</td>
      <td>Susan</td>
      <td>4155551450</td>
      <td>5677 Strong St.</td>
      <td></td>
      <td>San Rafael</td>
      <td>97562</td>
      <td>USA</td>
      <td>1165</td>
      <td>210500</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>100 Market Street</td>
      <td>Suite 300</td>
      <td>CA</td>
      <td>USA</td>
      <td>94080</td>
      <td>NA</td>
      <td>129</td>
      <td>...</td>
      <td>Murphy</td>
      <td>Julie</td>
      <td>6505555787</td>
      <td>5557 North Pendale Street</td>
      <td></td>
      <td>San Francisco</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>64600</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>100 Market Street</td>
      <td>Suite 300</td>
      <td>CA</td>
      <td>USA</td>
      <td>94080</td>
      <td>NA</td>
      <td>161</td>
      <td>...</td>
      <td>Hashimoto</td>
      <td>Juri</td>
      <td>6505556809</td>
      <td>9408 Furth Circle</td>
      <td></td>
      <td>Burlingame</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>84600</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>100 Market Street</td>
      <td>Suite 300</td>
      <td>CA</td>
      <td>USA</td>
      <td>94080</td>
      <td>NA</td>
      <td>205</td>
      <td>...</td>
      <td>Young</td>
      <td>Julie</td>
      <td>6265557265</td>
      <td>78934 Hillside Dr.</td>
      <td></td>
      <td>Pasadena</td>
      <td>90003</td>
      <td>USA</td>
      <td>1166</td>
      <td>90700</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>100 Market Street</td>
      <td>Suite 300</td>
      <td>CA</td>
      <td>USA</td>
      <td>94080</td>
      <td>NA</td>
      <td>219</td>
      <td>...</td>
      <td>Young</td>
      <td>Mary</td>
      <td>3105552373</td>
      <td>4097 Douglas Av.</td>
      <td></td>
      <td>Glendale</td>
      <td>92561</td>
      <td>USA</td>
      <td>1166</td>
      <td>11000</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>



Whenever you write a SQL query, make sure you understand the unit of analysis you are trying to use. Getting more data from the database is not always better! The above query might make sense as a starting point for something like "what is the ratio of customers to offices in each state", but it's not there yet. Many-to-many joins can be useful, but it's important to be strategic and understand what you're really asking for.

## Summary

In this section, you expanded your join knowledge to one-to-many and many-to-many joins!
