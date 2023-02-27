# SQLite and SQLAlchemy Guide

This notebook shows how to interact with SQLite databases using SQLAlchemy.

Original tutorial source: [SQLAlchemy Tutorial With Examples](https://www.datacamp.com/tutorial/sqlalchemy-tutorial-examples).

See also the SQL guide, in particular the section about SQLite and SQAlchemy: [`../SQL_Guide.md`](../SQL_Guide.md).

Three datasets are used, which are colocated.

- [European Football Database @ Kaggle](https://www.kaggle.com/datasets/groleo/european-football-database?select=european_database.sqlite): [`./european_database.sqlite`](./european_database.sqlite)
- [Stock Exchange Data @ Datacamp](https://www.datacamp.com/workspace/datasets/dataset-python-stock-exchange): [`./stock.sqlite`](./stock.sqlite)
- Custom created students dataset: [`./students.sqlite`](./students.sqlite)

Table of contents:

- [1. Basics](#1-basics)
  - [1.1 Connect to or Open Database](#11-connect-to-or-open-database)
  - [1.2. Access to Tables and Columns](#12-access-to-tables-and-columns)
  - [1.3 Queries](#13-queries)
- [2. Creating a New Database and Working with It](#2-creating-a-new-database-and-working-with-it)
  - [2.1 Create a New Database](#21-create-a-new-database)
  - [2.2 Inserting Rows](#22-inserting-rows)
  - [2.3 Queries: Examples with SQL and with SQLAlchemy](#23-queries-examples-with-sql-and-with-sqlalchemy)
  - [2.4 Output to a Pandas Dataframe and CSV](#24-output-to-a-pandas-dataframe-and-csv)
  - [2.5 Input from CSV and Pandas](#25-input-from-csv-and-pandas)
- [3. SQL Table Management](#3-sql-table-management)
  - [3.1 Update and Delete Rows](#31-update-and-delete-rows)
  - [3.2 Dropping Tables](#32-dropping-tables)
  - [3.3 Complex Queries with SQLAlchemy: Joining Tables and Filtering](#33-complex-queries-with-sqlalchemy-joining-tables-and-filtering)


```python
import sqlalchemy as db
```


```python
db.__version__
```




    '1.4.39'



## 1. Basics

### 1.1. Connect to or Open Database


```python
# Download the sqlite database from the link above
# We open/connect to the database file
# The database has 2 tables: divisions & matchs
engine = db.create_engine("sqlite:///european_database.sqlite")
conn = engine.connect() 
```

### 1.2. Access to Tables and Columns


```python
# Inspect which tables we have in the db
insp = db.inspect(engine)
table_names = insp.get_table_names()
print(table_names)
```

    ['divisions', 'matchs']



```python
# Extract the metadata
metadata = db.MetaData()

# Create table object: we need to create it to further access it
# We use the autoload option because we don't know
# how the table was created
division = db.Table('divisions',
                    metadata,
                    autoload=True,
                    autoload_with=engine)

# Show columns 
#metadata.tables['divisions']
print("Raw 'divisions' table structure:")
print(repr(metadata.tables['divisions']))
print("\nTable 'divisions' columns:")
print(division.columns.keys())
print("\nTable 'divisions' column and type:")
columns = division.columns
for c in columns:
    print(f"- {c.name}, {c.type}")

# Crete the object of the other table
matchs = db.Table('matchs',
                    metadata,
                    autoload=True,
                    autoload_with=engine)

# Another way of displaying data:
print("\nComplete database:")
tables = (division, matchs)
for table in tables:
    columns = table.columns
    print(f"- Table: {table.fullname}")
    for c in columns:
        print(f"  - {c.name}, {c.type}")
```

    Raw 'divisions' table structure:
    Table('divisions', MetaData(), Column('division', TEXT(), table=<divisions>), Column('name', TEXT(), table=<divisions>), Column('country', TEXT(), table=<divisions>), schema=None)
    
    Table 'divisions' columns:
    ['division', 'name', 'country']
    
    Table 'divisions' column and type:
    - division, TEXT
    - name, TEXT
    - country, TEXT
    
    Complete database:
    - Table: divisions
      - division, TEXT
      - name, TEXT
      - country, TEXT
    - Table: matchs
      - Div, TEXT
      - Date, DATE
      - HomeTeam, TEXT
      - AwayTeam, TEXT
      - FTHG, REAL
      - FTAG, REAL
      - FTR, TEXT
      - season, INTEGER


### 1.3. Queries


```python
# Option 1: SQLAlchemy Queries
# Create a query object: it's a SQL statement, but defined as a Python object
# SELECT * FROM division
query = division.select()
```


```python
# We can print the query object and get the SQL statement
print(query)
```

    SELECT divisions.division, divisions.name, divisions.country 
    FROM divisions



```python
# Another way of creating query objects: via db/sqlalchemy
query_ = db.select([division.columns.name])
print(query_)
```

    SELECT divisions.name 
    FROM divisions



```python
# Execute query
exe = conn.execute(query) # executing the query
# Get results: list of tuples; each tuple is a row
result = exe.fetchmany(5) # extracting top 5 results
print(f"\nFive results: \n{result}")
result = exe.fetchone() # extracting top result
print(f"\nTop result: \n{result}")
result = exe.fetchall() # extracting all results
print(f"\nAll results: \n{result}")
```

    
    Five results: 
    [('B1', 'Division 1A', 'Belgium'), ('D1', 'Bundesliga', 'Deutschland'), ('D2', '2. Bundesliga', 'Deutschland'), ('E0', 'Premier League', 'England'), ('E1', 'EFL Championship', 'England')]
    
    Top result: 
    ('E2', 'EFL League One', 'England')
    
    All results: 
    [('E3', 'EFL League Two', 'England'), ('EC', 'National League', 'England'), ('F1', 'Ligue 1', 'France'), ('F2', 'Ligue 2', 'France'), ('G1', 'Superleague', 'Greece'), ('I1', 'Seria A', 'Italy'), ('I2', 'Seria B', 'Italy'), ('N1', 'Eredivisie', 'Netherlands'), ('P1', 'Liga NOS', 'Portugal'), ('SC0', 'Scottish Premiership', 'Scotland'), ('SC1', 'Scottish Championship', 'Scotland'), ('SC2', 'Scottish League One', 'Scotland'), ('SP1', 'LaLiga', 'Spain'), ('SP2', 'LaLiga 2', 'Spain'), ('T1', 'Süper Lig', 'Turkey')]



```python
# Option 2: Simple SQL statements passed as strings
# Another way of creating queries: SQL statements as text/string!
query_text = """SELECT divisions.division, divisions.name, divisions.country
                FROM divisions"""
```


```python
exe = conn.execute(query_text) # executing the query
result = exe.fetchmany(5) # extracting top 5 results
print(result)
```

    [('B1', 'Division 1A', 'Belgium'), ('D1', 'Bundesliga', 'Deutschland'), ('D2', '2. Bundesliga', 'Deutschland'), ('E0', 'Premier League', 'England'), ('E1', 'EFL Championship', 'England')]


## 2. Creating a New Database and Working with It

### 2.1 Create a New Database


```python
# A new database file will be created, if none exists
# Creating and connecting is very similar
engine = db.create_engine('sqlite:///students.sqlite')
conn = engine.connect()
metadata = db.MetaData()

# Create a new table: Students
# This is the Core SQLAlchemy API; there is also the ORM API
# with which tables are created as classes.
# Now, we don't use the autoload option as before, because are creating the table!
# Note column arguments:
# https://docs.sqlalchemy.org/en/20/core/metadata.html#sqlalchemy.schema.Column
# - name
# - type: https://docs.sqlalchemy.org/en/14/core/types.html
#       Integer()
#       String()
#       Boolean()
#       Date()
#       Time()
#       Numeric()
#       LargeBinary() = BLOB, images, etc.
#       PickleType() = PickleType builds upon the Binary type to apply Python’s pickle.dumps()
#       ...
# - kwargs: check docs
#       primary_key
#       nullable
#       default
#       ...
# Autoincrement: if we add 'sqlite_autoincrement=True', the primary key increases
# automatically, thus, we cannot insert its value.
Student = db.Table('Student',
                   metadata,
                   db.Column('Id', db.Integer(), primary_key=True),
                   db.Column('Name', db.String(255), nullable=False),
                   db.Column('Major', db.String(255), default="Math"),
                   db.Column('Pass', db.Boolean(), default=True), sqlite_autoincrement=True
                   )

metadata.create_all(engine)
```

### 2.2 Inserting Rows


```python
# Insert one row
# Note: if we added 'sqlite_autoincrement=True' in the table definition,
# the primary key increases automatically, thus, we cannot insert its value.
# If not, we can manually control the Id, but each inserted value
# must be different!
# WARNING: Every time we run this cell, a new entry/row is added!
query = db.insert(Student).values(#Id=1,
                                  Name='Matthew',
                                  Major="English",
                                  Pass=True)
result = conn.execute(query)

# Check: fetch all rows
output = conn.execute(Student.select()).fetchall()
print(output)

```

    [(1, 'Matthew', 'English', True), (2, 'Matthew', 'English', True), (3, 'Matthew', 'English', True), (4, 'Matthew', 'English', True), (5, 'Matthew', 'English', True), (6, 'Matthew', 'English', True), (7, 'Matthew', 'English', True), (8, 'Matthew', 'English', True), (9, 'Matthew', 'English', True), (10, 'Matthew', 'English', True), (11, 'Matthew', 'English', True), (12, 'Matthew', 'English', True), (13, 'Matthew', 'English', True), (14, 'Matthew', 'English', True), (15, 'Matthew', 'English', True)]



```python
# Insert many
# WARNING: Every time we run this cell, new entries/rows are added!
query = db.insert(Student)
#values_list = [{'Id':'2', 'Name':'Nisha', 'Major':"Science", 'Pass':False},
#              {'Id':'3', 'Name':'Natasha', 'Major':"Math", 'Pass':True},
#              {'Id':'4', 'Name':'Ben', 'Major':"English", 'Pass':False}]
values_list = [{'Name':'Nisha', 'Major':"Science", 'Pass':False},
              {'Name':'Natasha', 'Major':"Math", 'Pass':True},
              {'Name':'Ben', 'Major':"English", 'Pass':False}]
result = conn.execute(query,values_list)

# Check: fetch all rows
output = conn.execute(Student.select()).fetchall()
print(output)
```

    [(1, 'Matthew', 'English', True), (2, 'Matthew', 'English', True), (3, 'Matthew', 'English', True), (4, 'Matthew', 'English', True), (5, 'Matthew', 'English', True), (6, 'Matthew', 'English', True), (7, 'Matthew', 'English', True), (8, 'Matthew', 'English', True), (9, 'Matthew', 'English', True), (10, 'Matthew', 'English', True), (11, 'Matthew', 'English', True), (12, 'Matthew', 'English', True), (13, 'Matthew', 'English', True), (14, 'Matthew', 'English', True), (15, 'Matthew', 'English', True), (16, 'Nisha', 'Science', False), (17, 'Natasha', 'Math', True), (18, 'Ben', 'English', False), (19, 'Nisha', 'Science', False), (20, 'Natasha', 'Math', True), (21, 'Ben', 'English', False)]


### 2.3 Queries: Examples with SQL and with SQLAlchemy


```python
output = conn.execute("SELECT * FROM Student")
print(output.fetchall())
```

    [(1, 'Matthew', 'English', 1), (2, 'Matthew', 'English', 1), (3, 'Matthew', 'English', 1), (4, 'Matthew', 'English', 1), (5, 'Matthew', 'English', 1), (6, 'Matthew', 'English', 1), (7, 'Matthew', 'English', 1), (8, 'Matthew', 'English', 1), (9, 'Matthew', 'English', 1), (10, 'Matthew', 'English', 1), (11, 'Matthew', 'English', 1), (12, 'Matthew', 'English', 1), (13, 'Matthew', 'English', 1), (14, 'Matthew', 'English', 1), (15, 'Matthew', 'English', 1), (16, 'Nisha', 'Science', 0), (17, 'Natasha', 'Math', 1), (18, 'Ben', 'English', 0), (19, 'Nisha', 'Science', 0), (20, 'Natasha', 'Math', 1), (21, 'Ben', 'English', 0)]



```python
output = conn.execute("SELECT Name, Major FROM Student WHERE Pass = True")
print(output.fetchall())
```

    [('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Matthew', 'English'), ('Natasha', 'Math'), ('Natasha', 'Math')]



```python
# SQLAlchemy API: SELECT, WHERE
query = Student.select().where(Student.columns.Major == 'English')
output = conn.execute(query)
print(output.fetchall())
```

    [(1, 'Matthew', 'English', True), (2, 'Matthew', 'English', True), (3, 'Matthew', 'English', True), (4, 'Matthew', 'English', True), (5, 'Matthew', 'English', True), (6, 'Matthew', 'English', True), (7, 'Matthew', 'English', True), (8, 'Matthew', 'English', True), (9, 'Matthew', 'English', True), (10, 'Matthew', 'English', True), (11, 'Matthew', 'English', True), (12, 'Matthew', 'English', True), (13, 'Matthew', 'English', True), (14, 'Matthew', 'English', True), (15, 'Matthew', 'English', True), (18, 'Ben', 'English', False), (21, 'Ben', 'English', False)]



```python
# SQLAlchemy API: SELECT, WHERE, AND
query = Student.select().where(db.and_(Student.columns.Major == 'English',
                                       Student.columns.Pass != True))
output = conn.execute(query)
print(output.fetchall())
```

    [(18, 'Ben', 'English', False), (21, 'Ben', 'English', False)]



```python
# More API examples
# in                Student.select().where(Student.columns.Major.in_(['English','Math']))
# and, or, not      Student.select().where(db.or_(Student.columns.Major == 'English', Student.columns.Pass = True))
# order by          Student.select().order_by(db.desc(Student.columns.Name))
# limit             Student.select().limit(3)
# sum, avg, count   db.select([db.func.sum(Student.columns.Id)]) # min, max, etc.
# group by          db.select([db.func.sum(Student.columns.Id),Student.columns.Major]).group_by(Student.columns.Pass)
# distinct          db.select([Student.columns.Major.distinct()])
```

### 2.4 Output to a Pandas Dataframe and CSV


```python
import pandas as pd
```


```python
# First execute the query we wish and get the results
query = Student.select().where(Student.columns.Major.in_(['English','Math']))
output = conn.execute(query)
results = output.fetchall()
```


```python
# Create a dataframe: pass the results, get the columns names
data = pd.DataFrame(results)
data.columns = results[0].keys()
data
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
      <th>Id</th>
      <th>Name</th>
      <th>Major</th>
      <th>Pass</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>15</th>
      <td>17</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>18</td>
      <td>Ben</td>
      <td>English</td>
      <td>False</td>
    </tr>
    <tr>
      <th>17</th>
      <td>20</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
    <tr>
      <th>18</th>
      <td>21</td>
      <td>Ben</td>
      <td>English</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>






```python
# Dataframe -> CSV
data.to_csv("student_result.csv", index=False)
```

### 2.5 Input from CSV and Pandas


```python
# Create/connect database: if  no file, one is created
engine = db.create_engine("sqlite:///stock.sqlite")
```


```python
# Read CSV: another CSV file is loaded, one with stock prices
df = pd.read_csv('stock_data.csv')
# Convert to SQL table
# `to_sql` function requires connection and table name as an argument.
# You can also use `if_exisits` to replace an existing table 
# with the same name and `index` to drop the index column. 
df.to_sql(con=engine, name="Stock_price", if_exists='replace', index=False)
```


```python
# To validate the results, we need to connect the database and create a table object.
conn = engine.connect()
metadata = db.MetaData()
stock = db.Table('Stock_price', metadata, autoload=True, autoload_with=engine)
```


```python
# Check results with SELECT * query
query = stock.select()
exe = conn.execute(query)
result = exe.fetchmany(5)
for r in result:
    print(r)
```

    ('HSI', '1986-12-31', 2568.300049, 2568.300049, 2568.300049, 2568.300049, 2568.300049, 0.0, 333.87900637)
    ('HSI', '1987-01-02', 2540.100098, 2540.100098, 2540.100098, 2540.100098, 2540.100098, 0.0, 330.21301274)
    ('HSI', '1987-01-05', 2552.399902, 2552.399902, 2552.399902, 2552.399902, 2552.399902, 0.0, 331.81198726)
    ('HSI', '1987-01-06', 2583.899902, 2583.899902, 2583.899902, 2583.899902, 2583.899902, 0.0, 335.90698726000005)
    ('HSI', '1987-01-07', 2607.100098, 2607.100098, 2607.100098, 2607.100098, 2607.100098, 0.0, 338.92301274)


## 3. SQL Table Management


```python
# Connect to Students database, again
engine = db.create_engine('sqlite:///students.sqlite')
conn = engine.connect()
metadata = db.MetaData()

# We don't need to re-define the table, although we could do it
# Instead, we can use autoload
student = db.Table('Student',
                   metadata,
                   autoload=True,
                   autoload_with=engine
                   )

# Check results with SELECT * query
query = student.select()
exe = conn.execute(query)
result = exe.fetchmany(5)
for r in result:
    print(r)
```

    (1, 'Matthew', 'English', True)
    (2, 'Matthew', 'English', True)
    (3, 'Matthew', 'English', True)
    (4, 'Matthew', 'English', True)
    (5, 'Matthew', 'English', True)


### 3.1 Update and Delete Rows


```python
# We can use a SQL statement to UPDATE rows
# or we can do it via the SQLAlchemy API
# Syntax:
# table.update().values(column_1=1, column_2=4,...).where(table.columns.column_5 >= 5)
query = student.update().values(Pass = True).where(student.columns.Name == "Nisha")
results = conn.execute(query)

# Check
output = conn.execute(Student.select()).fetchall()
data = pd.DataFrame(output)
data.columns = output[0].keys()
data
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
      <th>Id</th>
      <th>Name</th>
      <th>Major</th>
      <th>Pass</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>Nisha</td>
      <td>Science</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
    <tr>
      <th>17</th>
      <td>18</td>
      <td>Ben</td>
      <td>English</td>
      <td>False</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>Nisha</td>
      <td>Science</td>
      <td>True</td>
    </tr>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>Ben</td>
      <td>English</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Delete has a similar syntax
# table.delete().where(table.columns.column_1 == 6)
query = Student.delete().where(Student.columns.Name == "Ben")
results = conn.execute(query)

# Check
output = conn.execute(Student.select()).fetchall()
data = pd.DataFrame(output)
data.columns = output[0].keys()
data
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
      <th>Id</th>
      <th>Name</th>
      <th>Major</th>
      <th>Pass</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>Matthew</td>
      <td>English</td>
      <td>True</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>Nisha</td>
      <td>Science</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
    <tr>
      <th>17</th>
      <td>19</td>
      <td>Nisha</td>
      <td>Science</td>
      <td>True</td>
    </tr>
    <tr>
      <th>18</th>
      <td>20</td>
      <td>Natasha</td>
      <td>Math</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



### 3.2 Dropping Tables


```python
# First, we need to close all queries!
# Otherwise, we get the error that the database is locked
results.close()
exe.close()
# Then, we can drop the desired tables; if none specified, all dropped: metadata.drop_all(engine)
metadata.drop_all(engine, [student], checkfirst=True)
```

### 3.3 Complex Queries with SQLAlchemy: Joining Tables and Filtering


```python
# Connect to database
engine = db.create_engine("sqlite:///european_database.sqlite")
conn = engine.connect()
metadata = db.MetaData()
division = db.Table('divisions', metadata, autoload=True, autoload_with=engine)
match = db.Table('matchs', metadata, autoload=True, autoload_with=engine)
```


```python
# Running complex queries - Example:
# 1. We will select both division and match columns.
# 2. Join them using a common column: division.division and match.Div.
# 3. Select all columns where the division is E1 and the season is 2009.
# 4. Order the result by HomeTeam.
query = db.select([division,match]).\
            select_from(division.join(match,division.columns.division == match.columns.Div)).\
            where(db.and_(division.columns.division == "E1", match.columns.season == 2009 )).\
            order_by(match.columns.HomeTeam)
output = conn.execute(query)
results = output.fetchall()

data = pd.DataFrame(results)
data.columns = results[0].keys()
data
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
      <th>division</th>
      <th>name</th>
      <th>country</th>
      <th>Div</th>
      <th>Date</th>
      <th>HomeTeam</th>
      <th>AwayTeam</th>
      <th>FTHG</th>
      <th>FTAG</th>
      <th>FTR</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2008-08-16</td>
      <td>Barnsley</td>
      <td>Coventry</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>A</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>1</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2008-08-30</td>
      <td>Barnsley</td>
      <td>Derby</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2008-09-16</td>
      <td>Barnsley</td>
      <td>Cardiff</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>A</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>3</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2008-09-27</td>
      <td>Barnsley</td>
      <td>Norwich</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>D</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>4</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2008-10-04</td>
      <td>Barnsley</td>
      <td>Doncaster</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>547</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2009-03-10</td>
      <td>Wolves</td>
      <td>Ipswich</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>D</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>548</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2009-03-14</td>
      <td>Wolves</td>
      <td>Charlton</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>549</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2009-04-10</td>
      <td>Wolves</td>
      <td>Southampton</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>550</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2009-04-18</td>
      <td>Wolves</td>
      <td>QPR</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>551</th>
      <td>E1</td>
      <td>EFL Championship</td>
      <td>England</td>
      <td>E1</td>
      <td>2009-05-03</td>
      <td>Wolves</td>
      <td>Doncaster</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>H</td>
      <td>2009</td>
    </tr>
  </tbody>
</table>
<p>552 rows × 11 columns</p>
</div>




