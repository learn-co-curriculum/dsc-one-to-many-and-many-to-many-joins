
# One-to-Many and Many-to-Many Joins

## Introduction

Previously, you've learned about the typical case where one joins on a primary or foreign key. In this section, you'll explore other types of joins using One-to-Many and Many-to-many relationships!

## Objectives

You will be able to:

- Explain why join Tables are needed in Many-to-Many relationships

## One-to-Many and Many-to-Many relationships

So far, you've seen a couple different kinds of join statements: left joins and inner joins. Both of these refer to the way in which you would like to define your joins based on the tables and their shared information. Another perspective on this is the number of matches between the tables based on your defined links with the keywords *on* or *using*.
  
You have also seen the typical case where one joins on a primary or foreign key. For example, when you join on customerID or employeeID, this value should be unique to that table. As such, your joins have been very similar to using a dictionary to find additional information associated with that record. In cases where there are multiple entries, in either table, for the field you are joining on, you will similarly be given multiple rows in your resulting view, one for each of these entries.  
  
For example, let's say you have another table 'restaurants' that has many columns including *name*, *city*, and *rating*. If you were to join this table with the offices table using the shared city column, you might get some unexpected behavior. That is, in the office table, there is only one office per city. However, because there is apt to be more then one restaurant for each of these cities in your second table, you will get unique combinations of Offices and Restaurants from your join. If there are 513 restaurants for Boston in your restaurant table and 1 office for Boston, your joined table will have each of these 513 rows, one for each restaurant along with the one office.

If you had 2 offices for Boston, and 513 restaurants, your join would have 1026 rows for boston; 513 for each restuarant along with the first office and 513 for each restaurant with the second office. Three offices in Boston would similarly produce 1539 rows; one for each unique combination of restaurants and offices. This is where you should be particularly careful of many to many joins as the resulting set size can explode drastically potentially consuming vast amounts of memory and other resources.  

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
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 8





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
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Boston</td>
      <td>+1 215 837 0825</td>
      <td>1550 Court Place</td>
      <td>Suite 102</td>
      <td>MA</td>
      <td>USA</td>
      <td>02107</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>NYC</td>
      <td>+1 212 555 3000</td>
      <td>523 East 53rd Street</td>
      <td>apt. 5A</td>
      <td>NY</td>
      <td>USA</td>
      <td>10022</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Paris</td>
      <td>+33 14 723 4404</td>
      <td>43 Rue Jouffroy D'abbans</td>
      <td></td>
      <td></td>
      <td>France</td>
      <td>75017</td>
      <td>EMEA</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Tokyo</td>
      <td>+81 33 224 5000</td>
      <td>4-1 Kioicho</td>
      <td></td>
      <td>Chiyoda-Ku</td>
      <td>Japan</td>
      <td>102-8578</td>
      <td>Japan</td>
    </tr>
  </tbody>
</table>
</div>




```python
cur.execute('select * from employees;')
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 23





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
      <th>employeeNumber</th>
      <th>lastName</th>
      <th>firstName</th>
      <th>extension</th>
      <th>email</th>
      <th>officeCode</th>
      <th>reportsTo</th>
      <th>jobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td>1</td>
      <td></td>
      <td>President</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Marketing</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1088</td>
      <td>Patterson</td>
      <td>William</td>
      <td>x4871</td>
      <td>wpatterson@classicmodelcars.com</td>
      <td>6</td>
      <td>1056</td>
      <td>Sales Manager (APAC)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1102</td>
      <td>Bondur</td>
      <td>Gerard</td>
      <td>x5408</td>
      <td>gbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1056</td>
      <td>Sale Manager (EMEA)</td>
    </tr>
  </tbody>
</table>
</div>



### A One-to-One Join...


```python
cur.execute('select * from offices join employees using(officeCode);')
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 23





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
      <th>employeeNumber</th>
      <th>lastName</th>
      <th>firstName</th>
      <th>extension</th>
      <th>email</th>
      <th>reportsTo</th>
      <th>jobTitle</th>
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
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td></td>
      <td>President</td>
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
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1002</td>
      <td>VP Sales</td>
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
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1002</td>
      <td>VP Marketing</td>
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
      <td>1143</td>
      <td>Bow</td>
      <td>Anthony</td>
      <td>x5428</td>
      <td>abow@classicmodelcars.com</td>
      <td>1056</td>
      <td>Sales Manager (NA)</td>
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
      <td>1165</td>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>x3291</td>
      <td>ljennings@classicmodelcars.com</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
  </tbody>
</table>
</div>



### A One-to-Many Join
Here you join products with product lines. There are only a few product lines that will be matched to each product. As a result, the product line descriptions will be repeated in your resulting view.


```python
cur.execute('select * from products;')
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 110





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
      <th>productCode</th>
      <th>productName</th>
      <th>productLine</th>
      <th>productScale</th>
      <th>productVendor</th>
      <th>productDescription</th>
      <th>quantityInStock</th>
      <th>buyPrice</th>
      <th>MSRP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S10_1678</td>
      <td>1969 Harley Davidson Ultimate Chopper</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Min Lin Diecast</td>
      <td>This replica features working kickstand, front...</td>
      <td>7933</td>
      <td>48.81</td>
      <td>95.70</td>
    </tr>
    <tr>
      <th>1</th>
      <td>S10_1949</td>
      <td>1952 Alpine Renault 1300</td>
      <td>Classic Cars</td>
      <td>1:10</td>
      <td>Classic Metal Creations</td>
      <td>Turnable front wheels; steering function; deta...</td>
      <td>7305</td>
      <td>98.58</td>
      <td>214.30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>S10_2016</td>
      <td>1996 Moto Guzzi 1100i</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Highway 66 Mini Classics</td>
      <td>Official Moto Guzzi logos and insignias, saddl...</td>
      <td>6625</td>
      <td>68.99</td>
      <td>118.94</td>
    </tr>
    <tr>
      <th>3</th>
      <td>S10_4698</td>
      <td>2003 Harley-Davidson Eagle Drag Bike</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Red Start Diecast</td>
      <td>Model features, official Harley Davidson logos...</td>
      <td>5582</td>
      <td>91.02</td>
      <td>193.66</td>
    </tr>
    <tr>
      <th>4</th>
      <td>S10_4757</td>
      <td>1972 Alfa Romeo GTA</td>
      <td>Classic Cars</td>
      <td>1:10</td>
      <td>Motor City Art Classics</td>
      <td>Features include: Turnable front wheels; steer...</td>
      <td>3252</td>
      <td>85.68</td>
      <td>136.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
cur.execute('select * from productlines;')
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 7





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
      <th>htmlDescription</th>
      <th>image</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Classic Cars</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>Motorcycles</td>
      <td>Our motorcycles are state of the art replicas ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>Planes</td>
      <td>Unique, diecast airplane and helicopter replic...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ships</td>
      <td>The perfect holiday or anniversary gift for ex...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>Trains</td>
      <td>Model trains are a rewarding hobby for enthusi...</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
cur.execute("""select * from products
                      join productlines
                      using(productLine);""")
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 110





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
      <th>productCode</th>
      <th>productName</th>
      <th>productLine</th>
      <th>productScale</th>
      <th>productVendor</th>
      <th>productDescription</th>
      <th>quantityInStock</th>
      <th>buyPrice</th>
      <th>MSRP</th>
      <th>textDescription</th>
      <th>htmlDescription</th>
      <th>image</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>S10_1678</td>
      <td>1969 Harley Davidson Ultimate Chopper</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Min Lin Diecast</td>
      <td>This replica features working kickstand, front...</td>
      <td>7933</td>
      <td>48.81</td>
      <td>95.70</td>
      <td>Our motorcycles are state of the art replicas ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>S10_1949</td>
      <td>1952 Alpine Renault 1300</td>
      <td>Classic Cars</td>
      <td>1:10</td>
      <td>Classic Metal Creations</td>
      <td>Turnable front wheels; steering function; deta...</td>
      <td>7305</td>
      <td>98.58</td>
      <td>214.30</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>S10_2016</td>
      <td>1996 Moto Guzzi 1100i</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Highway 66 Mini Classics</td>
      <td>Official Moto Guzzi logos and insignias, saddl...</td>
      <td>6625</td>
      <td>68.99</td>
      <td>118.94</td>
      <td>Our motorcycles are state of the art replicas ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>S10_4698</td>
      <td>2003 Harley-Davidson Eagle Drag Bike</td>
      <td>Motorcycles</td>
      <td>1:10</td>
      <td>Red Start Diecast</td>
      <td>Model features, official Harley Davidson logos...</td>
      <td>5582</td>
      <td>91.02</td>
      <td>193.66</td>
      <td>Our motorcycles are state of the art replicas ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>S10_4757</td>
      <td>1972 Alfa Romeo GTA</td>
      <td>Classic Cars</td>
      <td>1:10</td>
      <td>Motor City Art Classics</td>
      <td>Features include: Turnable front wheels; steer...</td>
      <td>3252</td>
      <td>85.68</td>
      <td>136.00</td>
      <td>Attention car enthusiasts: Make your wildest c...</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



### A Many-to-Many Join

A many-to-many join is as it sounds; there are multiple entries for the shared field in both tables. While somewhat contrived, we can see this through the example below, joining the offices and customers table based on the state field. For example, there are 2 offices in MA and 9 customers in MA. Joining the two tables by state will result in 18 rows associated with MA; one for each customer combined with the first office, and then another for each customer combined with the second option. This is not a particularly useful join without applying some additional aggregations or pivots, but can also demonstrate how a poorly written query can go wrong. For example, if there are a large number of occurences in both tables, such as tens of thousands, then a many-to-many join could result in billions of resulting rows. Such ill conceived joins can cause severe load can be put on the database causing slow execution time, and potentially even tying up database resources for other analysts who may be using the system.


```python
cur.execute("""select * from offices
                        join customers
                        using(state);""")
df = pd.DataFrame(cur.fetchall())
df.columns = [i[0] for i in cur.description]
print('Number of results:', len(df))
df.head()
```

    Number of results: 254





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
      <td>210500.00</td>
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
      <td>64600.00</td>
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
      <td>84600.00</td>
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
      <td>90700.00</td>
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
      <td>11000.00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
len(df[df.state=='MA'])
```




    18



## Summary

In this section, you expanded your join knowledge to One-to-Many and Many-to-many joins!
