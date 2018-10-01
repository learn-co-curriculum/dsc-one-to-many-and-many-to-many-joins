
# One to Many and Many to Many Joins

We've looked at a couple kinds of different join statements: left joins and inner joins. Both of these refer to the way in which we would like to define our joins based on the tables and their shared information. Another perspective on this is the number of matches between the tables based on our defined links with the keywords *on* or *using*.
  
We've investigated the typical case where one joins on a primary or foreign key. For example, when we join on customerID or employeeID, this value should be unique to that table. As such, our joins have been very similar to using a dictionary to find additional information associated with that record. In cases where there are multiple entries, in either table, for the field you are joining on, you will similarly be given multiple rows in your resulting view, one for each of these entries.  
  
For example, let's say you have another table 'restaurants' that has many columns including *name*, *city*, and *rating*. If you were to join this table with the offices table using the shared city column, you might get some unexpected behavior. That is, in the office table, there is only one office per city. However, because there is apt to be more then one restaurant for each of these cities in our second table, we will get unique combinations of Offices and Restaurants from our join. If there are 513 restaurants for Boston in our restaurant table and 1 office for Boston, our joined table will have each of these 513 rows, one for each restaurant along with the one office.

If we had 2 offices for Boston, and 513 restaurants, our join would have 1026 rows for boston; 513 for each restuarant along with the first office and 513 for each restaurant with the second office. Three offices in Boston would similarly produce 1539 rows; one for each unique combination of restaurants and offices. This is where you should be particularly careful of many to many joins as the resulting set size can explode drastically potentially consuming vast amounts of memory and other resources.  

<img src='Database-Schema.png' width=550>

# Connecting to the Database


```python
import sqlite3
import pandas as pd
```


```python
conn = sqlite3.connect('data.sqlite', detect_types=sqlite3.PARSE_COLNAMES)
cur = conn.cursor()
```

# Checking Sizes of Resulting Joins...

##  A One to One Join

### The original tables...


```python
cur.execute('select * from offices;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

    Number of results: 7





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
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
print('Number of results:', len(df))
df.head()
```

    Number of results: 23





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
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



### The one-to-one Join...


```python
cur.execute('select * from offices join employees using(officeCode);')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

    Number of results: 23





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
      <th>13</th>
      <th>14</th>
      <th>15</th>
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



## A one to many join
Here we join products with product lines. There are only a few product lines that will be matched to each product. As a result, the product line descriptions will be repeated in our resulting view.


```python
cur.execute('select * from products;')
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

    Number of results: 110





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
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
print('Number of results:', len(df))
df.head()
```

    Number of results: 7





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
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
print('Number of results:', len(df))
df.head()
```

    Number of results: 110





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
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



## A many to many join

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

    Number of results: 45





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>18</th>
      <th>19</th>
      <th>20</th>
      <th>21</th>
      <th>22</th>
      <th>23</th>
      <th>24</th>
      <th>25</th>
      <th>26</th>
      <th>27</th>
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
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>...</td>
      <td>Murphy</td>
      <td>Julie</td>
      <td>6505555787</td>
      <td>5557 North Pendale Street</td>
      <td></td>
      <td>CA</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>64600.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td>1</td>
      <td></td>
      <td>President</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>...</td>
      <td>Brown</td>
      <td>Julie</td>
      <td>6505551386</td>
      <td>7734 Strong St.</td>
      <td></td>
      <td>CA</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>105000.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>...</td>
      <td>Murphy</td>
      <td>Julie</td>
      <td>6505555787</td>
      <td>5557 North Pendale Street</td>
      <td></td>
      <td>CA</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>64600.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>...</td>
      <td>Brown</td>
      <td>Julie</td>
      <td>6505551386</td>
      <td>7734 Strong St.</td>
      <td></td>
      <td>CA</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>105000.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Marketing</td>
      <td>San Francisco</td>
      <td>+1 650 219 4782</td>
      <td>...</td>
      <td>Murphy</td>
      <td>Julie</td>
      <td>6505555787</td>
      <td>5557 North Pendale Street</td>
      <td></td>
      <td>CA</td>
      <td>94217</td>
      <td>USA</td>
      <td>1165</td>
      <td>64600.00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 28 columns</p>
</div>




```python
cur.execute("""select * from employees;""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

    Number of results: 23





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
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




```python
cur.execute("""select * from customers;""")
df = pd.DataFrame(cur.fetchall())
print('Number of results:', len(df))
df.head()
```

    Number of results: 122





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103</td>
      <td>Atelier graphique</td>
      <td>Schmitt</td>
      <td>Carine</td>
      <td>40.32.2555</td>
      <td>54, rue Royale</td>
      <td></td>
      <td>Nantes</td>
      <td></td>
      <td>44000</td>
      <td>France</td>
      <td>1370</td>
      <td>21000.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>112</td>
      <td>Signal Gift Stores</td>
      <td>King</td>
      <td>Jean</td>
      <td>7025551838</td>
      <td>8489 Strong St.</td>
      <td></td>
      <td>Las Vegas</td>
      <td>NV</td>
      <td>83030</td>
      <td>USA</td>
      <td>1166</td>
      <td>71800.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>114</td>
      <td>Australian Collectors, Co.</td>
      <td>Ferguson</td>
      <td>Peter</td>
      <td>03 9520 4555</td>
      <td>636 St Kilda Road</td>
      <td>Level 3</td>
      <td>Melbourne</td>
      <td>Victoria</td>
      <td>3004</td>
      <td>Australia</td>
      <td>1611</td>
      <td>117300.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>119</td>
      <td>La Rochelle Gifts</td>
      <td>Labrune</td>
      <td>Janine</td>
      <td>40.67.8555</td>
      <td>67, rue des Cinquante Otages</td>
      <td></td>
      <td>Nantes</td>
      <td></td>
      <td>44000</td>
      <td>France</td>
      <td>1370</td>
      <td>118200.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>121</td>
      <td>Baane Mini Imports</td>
      <td>Bergulfsen</td>
      <td>Jonas</td>
      <td>07-98 9555</td>
      <td>Erling Skakkes gate 78</td>
      <td></td>
      <td>Stavern</td>
      <td></td>
      <td>4110</td>
      <td>Norway</td>
      <td>1504</td>
      <td>81700.00</td>
    </tr>
  </tbody>
</table>
</div>


