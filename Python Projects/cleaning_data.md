# Cleaning Data in Python (in progress)

### Import CSV with custom Null types
```
na_vals = ['NA', 'Missing', 'N/A', 'none'] #custom null values
d_parser = lambda x: pd.datetime.strptime(x, '%Y-%m-%d %I-%p') #date format
df = pd.read_csv('filepath', na_values=na_vals, index_col='Set Column As Index', parse_date=['Date columns needing formatting'], date_parser=d_parser)
```

### Import Excel/JSON
```
df = pd.read_excel('filepath')
df = pd.read_json('filepath', orient='records', lines=True)
df = pd.read_json('url')
```

### Data Info
```
df.shape
df.columns
```

### Find/Edit NaN values
```
df.isna() / df.isnull()
df.isna().sum()
df['column'].unique()
df.replace('na values', np.nan, inplace=True)
df.fillna(0)
df.dropna() #axis='index' vs 'column', how='any' vs 'all', subset=['column(s)']
```

### Check/Convert Null Datatypes
```
df.dtypes
df['column'].astype(float) 
#type(np.nan) is a float
#convert to float if column includes NaN values
```

### Rename Columns
```
df.rename(columns={'org column': 'new column', ...}, inplace=True)
```

### Convert Datetime
```
df.dtypes
df['Date Column'] = pd.to_datetime(df['Date Column'], format='%Y-%m-%d %I-%p')
```

### Create Datetime Columns
```
df['Day of Week'] = df['Date Column'].dt.day_name()
```

### Sort
```
df.sort_values(by='column(s)')
```

### Count Values
```
df.value_counts() #dropna=False
```

### Filter Values
```
filt = df['column'] == 'value'
df.loc[filt]
df.loc['row', 'column'] #use ":" for multiple
```

### Filter Dates
```
filt = (df['Date column'] >= pd.to_datetime('2019-01-01')) & (df['Date column'] <= pd.to_datetime('2020-01-01'))
df.loc[filt]
```

### Filter Dates When Date is Indexed
```
df.set_index['Date column']
df['2019']
df['2020-01':'2020-02']
```

### Resample Dates
```
df['Column'].resample('D')
```

### Aggregations
```
df.resample('W').agg({'Column 1': 'mean', 'Column 2': 'sum', 'Column 3': 'min'})
```

### Export
```
df.to_csv('filepath')
df.to_csv('filepath', sep='\t') #tsv
df.to_excel('filepath')
df.to_json('filepath', orient='records', lines=True)
```

### Connect and export to SQL-PostgreSQL
```
from sqlalchemy import create_engine
import psycopg2

engine = create_engine('postgresql://dbuser:dbpass@localhost:5432/sampe_db') #database connection; should use environment variable for user/pw
df.to_sql('sample_table', engine) #create new table
df.to_sql('sample_table', engine, if_exists='replace') #replace table
df.to_sql('sample_table', engine, if_exists='append') #append new data to table
```

### Import/Query SQL
```
engine = create_engine('postgresql://dbuser:dbpass@localhost:5432/sampe_db') #database connection; should use environment variable for user/pw
df = pd.read_sql('sample_table', engine, index_col='Column name')
df = pd.read_sql_query('SELECT * FROM sample_table', engine, index_col='')
```

### Necessary Packages
```
pip install xlwt openpyxl xlrd #excel
pip install SQLAlchemy
pip install psycopg2-binary #postgreSQL
```
