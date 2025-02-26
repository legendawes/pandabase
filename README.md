# pandabase
##### pip install pandabase
[![Build Status](https://travis-ci.org/notsambeck/pandabase.svg?branch=master)](https://travis-ci.org/notsambeck/pandabase)

pandabase links pandas DataFrames to SQL databases, supporting read, append, and upsert. 

By default, uses DataFrame.index as the primary key. By using an explicit primary key, pandabase makes rational database schemas the obvious choice, and makes it easy to maintain clean data even when it must be updated frequently. 

Designed for especially for time-series datasets that need to be updated over time and stored to disk, but are used primarily in-memory for computation. All supported types can be a value or Null, nice for ML applications.

Tested under Python 3.6 and 3.7, with new versions of Pandas (>= 0.24) SQLAlchemy (>= 1.3). Works with SQLite and postgres - requires psycopg2 and postgres>=8.

### Features
* pandabase.to_sql replaces df.to_sql
* pandabase.read_sql replaces pd.read_sql
* primary keys: by default, uses df.index (with name != None) as PK
  * alternatively, supports auto_index(with parameter auto_index=True)
* insert modes: 'create_only', 'upsert', and 'append'
* datatypes (nullable): boolean, int, float, datetime (UTC only), and string/object 

### Bonus Features
* moderately smart insertion handles new records that 'almost correspond' with database schema
* tested under SQLite and PostgresQL
* 93% test coverage (pytest)
* includes companda for rich comparisons of DataFrames - super valuable for testing

### Design Considerations
* Minimal dependencies: Pandas (>= 0.24) & SQLAlchemy (>= 1.3) are the only requirements
* Database is the source of truth: will coerce incoming DataFrames to fit existing schema
  * but also is reasonably smart about how new tables are created from DataFrames
* Not horrendously slow (?)

### License
MIT license

### Thanks
Code stolen from:
[Dataset](https://github.com/pudo/dataset) (nice, more general-purpose SQL interaction library) and 
[pandas.sql](https://github.com/pandas-dev/pandas/blob/master/pandas/io/sql.py)

### Installation
From your inside your virtual environment of choice:

```bash
~/$ pip install pandabase
```

For latest version:

```bash
~/$ git clone https://github.com/notsambeck/pandabase
~/$ cd pandabase
~/pandabase/$ pip install -r requirements.txt
~/pandabase/$ pip install .
```

### Usage
```python
# Python >= 3.6
>>> import pandas as pd
>>> import pandabase
>>> my_data = pd.DataFrame(index=range(7, 12), 
                           columns=['some_number'],
                           data=pd.np.random.random((5,1)))
>>> my_data.index.name = 'my_index_name'        # index must be named to use as PK
>>> pandabase.to_sql(my_data, table_name='my_table', con='sqlite:///new_sqlite_db.sqlite', how='create_only')
Table('my_table', ...
>>> exit()
```

Your data is now persistently stored in a SQLite database, using my_data.index as primary key. To append or update data, replace 'create_only' with 'append' or 'upsert'. To store records without an explicit index, use 'autoindex=True'.

```bash
~/pandabase$ ls
new_sqlite_db.sqlite
```

```python
>>> import pandabase
>>> df = pandabase.read_sql('my_table', con='sqlite:///new_sqlite_db.sqlite'))
>>> df
    some_number 
7   0.722416 
8   0.076045 
9   0.213118 
10  0.453716 
11  0.406995
```

### Using Extra Features
Companda - rich comparisons of DataFrames. call companda on two DataFrames, get a Companda object back (that evaluates to True/False).

```python
>>> from pandabse.companda import companda
>>> df = pandabase.read_sql('my_table', con='sqlite:///new_sqlite_db.sqlite'))
>>> companda(df, df.copy())
Companda(True, message='Equal DataFrames')
>>> bool(companda(df, df.copy()))
True

>>> df2 = df.copy
>>> df2.iloc[1, 2] = -1000
>>> companda(df, df2)
Companda(False, message='Columns, indices are equal, but unqual values in columns [col_a]...')
>>> bool(companda(df, df2))
False
```

Table tools: pandabase.
* add_columns_to_db(new_col, table_name, con):
    * """Make new columns as needed with ALTER TABLE (can be run by pandabase.to_sql automatically)"""
* drop_db_table(table_name, con):
    * """Drop table [table_name] from con"""
* get_db_table_names(con):
    * """get a list of table names from database"""
* get_table_column_names(con, table_name):
    * """get a list of column names from database, table"""
* describe_database(con):
    * """get a description of database content: table_name: {table_info_dict}"""
