# Cleaning Data in Python (in progress)

### Import CSV with custom Null types
'''
na_vals = ['NA', 'Missing', 'N/A', 'none']
df = pd.read_csv('csv.file', na_values=na_vals, index_col='Set Column As Index')
'''

### Find/Edit NaN values
```
df.isna() / df.isnull()
df.isna().sum()
df['column'].unique()
df.replace('na values', np.nan, inplace=True)
df.fillna(0)
df.dropna() #axis='index' vs 'column', how='any' vs 'all', subset=['column(s)']
```

### Check/Convert Datatypes
```
df.dtypes
df['column'].astype(float) 
#type(np.nan) is a float
#convert to float if column includes NaN values
```

### Sort
```
df.sort_values(by='column(s)')
```

### Count Values
```
df.value_counts() #dropna=False
```

### Locate Values
```
filt = df['column'] == 'value'
df.loc[filt]
df.loc['row', 'column'] #use ":" for multiple
```
