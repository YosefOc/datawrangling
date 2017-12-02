
# # OpenStreetMap Data Case Study (convert and clean only) and upload to database

> This map is of San Jose. In the audit section, I decided to clean or standardize some of the naming convention. Most of my work here I guess falls under the uniformity aspect of Data Quality. I have completed the Audit or step one of Data Quality Process. Here I will be looking at Step two and three. 

                            1. Audit the data (previous script)                                                                                                 
                            2. Create a Data Cleaning Plan                                                                                 
                            3. Execute the Plan                                                                                                          
                            

# Part I: Standardize city name and correct errors 

> What does it mean to correct name? For example, the city of San Jose has a case where it is spelled or has more characters, Sunnyvale also spelled with state name CA, etc.. 
> Below is updating function using mapping and regular expression to standardize the name of cities and a small code within the Shape_element function that makes update before information is writing to CSV file 

> In terms of script a large and significant part is from Data.py from course with proper schema. 

# City Name Harmonization

> For the city name standardization, I basically utilized function with mapping and regular expression to standardize the name and I inserted this function into the shape_element function from course. 
>    this function is called within the Shape_element function to make the change.. The mapping_city is below in script 

             def update_city(name_city, mapping_city):
   
                 m = pattern_city.search(name_city)       # does the name match the regular expression pattern
                 if m and m.group() in mapping_city:  # if pattern matches(1) and value(name) is in the mapping dictionary
                    name_city = re.sub(pattern_city, mapping_city[m.group()], name_city)  # change the name 
                 return name_city   # return the mapping name ( for example if it is "sunnyvale" return "Sunnyvale") """

>  Here is how it is inserted into the Shape_element function  
 
 # in this part I use update_city to standardize cities name:
                if child.attrib['k'] == 'addr:city':                                      
                    tag['value'] = update_city(child.attrib['v'], mapping_city)


> My mine challenge here was unicode. The letter e in San Jose could be spelled with an accent as was the case in some situation. I started this assignment using Python 3.6 and I didn't encounter this problem. However, due the fact that data.py script was done in Python 2 I created an Python 2.7 environment to do this assignment- I have tired to use .encode('utf-8'). It didn't work for me. 

# street type harmonization

> For the street type standardization, I basically utilized function with mapping and regular expression to standardize the name and I inserted this function into the shape_element function from course material. Complete script is below. 
>   Here is function for street type change: 

    def update_street(name_type, mapping_street):
        m = pattern_street.search(name_type)       # does the name match the regular expression pattern
        if m and m.group() in mapping_street:  # if pattern matches(1) and value(name) is in the mapping dictionary
            name_type = re.sub(pattern_street, mapping_street[m.group()], name_type)  # change the name with mapping proper name         return name_type   # return the mapping name ( for example if it is "Blvd" return "Boulevard")

>  Here is how it is inserted into the Shape_element function- it executed below..  

# in this part I use update_street to standardize street types:            
                elif child.attrib["k"] == 'addr:street':
                    tag['value'] = update_street(child.attrib['v'], mapping_street)

> This part was not difficult except accepting that street type naming includes via... or calle... which is hard to audit. 

# state name harmonization
>   Here is function change state name to one standard: 

    def update_state(name_state, mapping_state):
        m = pattern_state.search(name_state)       # does the name match the regular expression pattern
            if m and m.group() in mapping_state:  # if pattern matches(1) and value(name) is in the mapping dictionary
                name_state = re.sub(pattern_state, mapping_state[m.group()], name_state)  # change the name 
            return name_state   # return the mapping name ( for example if it is "ca" return "CA")


>  Here is how it is inserted into the Shape_element function- it executed below..  

# in this part I use update_state to standardize state name:                    
                elif child.attrib["k"] == 'addr:state':
                    tag['value'] = update_state(child.attrib['v'], mapping_state)
> Not much discrepancy... just performed this for uniformity. 

# country name harmonization
>   Here is function change country name to one standard: 

    def update_country(name_country, mapping_country):
        m = pattern_country.search(name_country)       # does the name match the regular expression pattern
            if m and m.group() in mapping_country:  # if pattern matches(1) and value(name) is in the mapping dictionary
                name_country= re.sub(pattern_country, mapping_country[m.group()], name_country)  # change the name  
            return name_country   # return the mapping name ( for example if it is "US" return "USA")



>  Here is how it is inserted into the Shape_element function- it executed below..  

# in this part I use update_country to standardize country name:                    
                elif child.attrib["k"] == 'addr:country':
                    tag['value'] = update_country(child.attrib['v'], mapping_country) 
> Not much discrepancy... just performed this for uniformity. 

# The main script to script to be implemented- step 3 execute plan 

> Final cleaning script to get implemented utilizes the files below from course with slight modification.

             1. Data.py 
             2. Schema 
             
     

# from CSV to SQLITE 3 database

> At this point all the five CSV files have been created and now it is time import the information into SQL database. 
 per course information we are going to use the SQLITE 3 tool

# Connecting to SQLITE3 database (setup)

> Part I: is importing the modules that are required 


```python
import sqlite3
import csv
from pprint import pprint
```

> Part II: is connecting to database


```python
# I decided to call my database below(since it doesn't exist it will be created)

sqlite_file = 'sanjose_map.db'    


# Connect to the database
conn = sqlite3.connect(sqlite_file)
```

> Part III: initiating a cursor- ("cursor is a control structure that enables traversal over the records in a database")


```python
# Get a cursor object
cur = conn.cursor()
```

# Part I: Creating tables in database 

> At this point we are read to create tables in the database. These tables will blank with structure only. 
There are five tables in csv: 
                       1. nodes.csv 
                       2. nodes_tags.csv 
                       3. ways.csv
                       4. ways_nodes.csv 
                       5. ways_tags.csv 


```python
# create the nodes table (id, lat, long, user, uid, version, changeset, timestamp)
cur.execute('''
    CREATE TABLE IF NOT EXISTS nodes(id INTEGER, lat REAL, lon REAL, user TEXT, uid INTEGER, version TEXT, changeset INTEGER, timestamp DATE)
''')

# create the nodes_tag table ( id, key, value, type) 
cur.execute('''
    CREATE TABLE IF NOT EXISTS nodes_tags(id INTEGER, key TEXT, value TEXT,type TEXT)
''')

# create the ways table(id, user, uid, version, changeset, timestamp) 
cur.execute('''
    CREATE TABLE IF NOT EXISTS ways(id INTEGER, user TEXT, uid INTEGER, version TEXT, changeset INTEGER, timestamp DATE)
''')

# create the ways_nodes(id, node_id, position)
cur.execute('''
    CREATE TABLE IF NOT EXISTS ways_nodes(id INTEGER, node_id INTEGER, position INTEGER)
''')

# create the ways_tags table ( id, key, value, type) 
cur.execute('''
    CREATE TABLE IF NOT EXISTS ways_tags(id INTEGER, key TEXT, value TEXT,type TEXT)
''')

# after creating all these tables we need to commit the change 

conn.commit()

```

> Verify the existence of table 


```python
cur.execute('select * from nodes_tags')
r = cur.fetchone()
print (cur.description)
```

    (('id', None, None, None, None, None, None), ('key', None, None, None, None, None, None), ('value', None, None, None, None, None, None), ('type', None, None, None, None, None, None))
    

# Inputting the data from csv to database 

> At this point, we have created a new database(san_jose.map), created five tables and committed the change, next step is inputting data from csv file into tables

> This is a two step process: Reading the files using DictReader(per row) and then inserting the row the the tables 

>> Step One: Open file and read rows 


```python
# opening and reading the nodes.csv file 
with open('nodes.csv','rb') as fin:
    dr = csv.DictReader(fin) 
    to_db = [(i['id'].decode('utf-8'),i['lat'],i['lon'],i['user'].decode('utf-8'),i['uid'],i['version'],i['changeset'],i['timestamp']) for i in dr if i['id']]

# opening and reading the nodes_tags file 
with open('nodes_tags.csv','rb') as fin:
    dr = csv.DictReader(fin) # comma is default delimiter
    to_db2 = [(i['id'].decode("utf-8"), i['key'].decode("utf-8"),i['value'].decode("utf-8"), i['type'].decode("utf-8")) for i in dr if i['key']]


# opening and reading the ways file 
with open('ways.csv','rb') as fin:
    dr = csv.DictReader(fin) 
    to_db3 = [(i['id'].decode('utf-8'),i['user'].decode('utf-8'),i['uid'],i['version'],i['changeset'],i['timestamp']) for i in dr if i['id']]


# opening and reading the ways_tags file 
with open('ways_tags.csv','rb') as fin:
    dr = csv.DictReader(fin) # comma is default delimiter
    to_db4 = [(i['id'].decode("utf-8"), i['key'].decode("utf-8"),i['value'].decode("utf-8"), i['type'].decode("utf-8")) for i in dr if i['key']]

# opening and reading the ways_nodes file
with open('ways_nodes.csv','rb') as fin:
    dr = csv.DictReader(fin) # comma is default delimiter
    to_db5 = [(i['id'].decode("utf-8"), i['node_id'].decode("utf-8"),i['position'].decode("utf-8")) for i in dr if i['id']]


```

>>Step Two: Inserting file information to respective table 


```python
#inserting information to nodes table 
cur.executemany("INSERT INTO nodes(id, lat, lon, user, uid, version, changeset, timestamp) VALUES (?, ?, ?, ?, ?, ?, ?, ?);", to_db)

#inserting information to ways table 
cur.executemany("INSERT INTO ways(id, user, uid, version, changeset, timestamp) VALUES (?, ?, ?, ?, ?, ?);", to_db3)

#inserting information to nodes_tags table 
cur.executemany("INSERT INTO nodes_tags(id, key, value,type) VALUES (?, ?, ?, ?);", to_db2)

#inserting inforamtion to ways_tags table 
cur.executemany("INSERT INTO ways_tags(id, key, value,type) VALUES (?, ?, ?, ?);", to_db4)

#inserting information to ways_nodes table 
cur.executemany("INSERT INTO ways_nodes(id, node_id, position) VALUES (?, ?, ?);", to_db5)

# committing the changes to tables 
conn.commit()
```

> At this point all the information is inputted into the tables

> Can we can verify this by looking at couple lines from tables with limit 5 since some files are very large

>> nodes table verification 


```python
cur.execute('SELECT * FROM nodes LIMIT 5')
all_rows = cur.fetchall()
print('1):')
pprint(all_rows)
```

    1):
    [(25457954,
      37.1582245,
      -121.6574737,
      u'KindredCoda',
      14293,
      u'10',
      11686320,
      u'2012-05-24T03:24:59Z'),
     (25457955,
      37.1648535,
      -121.6653013,
      u'KindredCoda',
      14293,
      u'10',
      11721513,
      u'2012-05-27T23:43:57Z'),
     (25457957,
      37.186653,
      -121.6871419,
      u'KindredCoda',
      14293,
      u'15',
      11721513,
      u'2012-05-27T23:43:57Z'),
     (25457960,
      37.2273856,
      -121.7433767,
      u'KindredCoda',
      14293,
      u'15',
      11721596,
      u'2012-05-28T00:02:33Z'),
     (25457961,
      37.2578667,
      -121.7968539,
      u'adbrown',
      54598,
      u'22',
      755914,
      u'2008-11-16T05:13:41Z')]
    

>> nodes_tags table verification 


```python
cur.execute('SELECT * FROM nodes_tags LIMIT 5')
all_rows = cur.fetchall()
print('1):')
pprint(all_rows)
```

    1):
    [(26027688, u'highway', u'traffic_signals', u'regular'),
     (26027690, u'highway', u'traffic_signals', u'regular'),
     (26028102, u'highway', u'motorway_junction', u'regular'),
     (26028102, u'noref', u'yes', u'regular'),
     (26028723, u'exit_to', u'Arques / Fair Oaks', u'regular')]
    

>> ways table verification 


```python
cur.execute('SELECT * FROM ways LIMIT 5')
all_rows = cur.fetchall()
print('1):')
pprint(all_rows)
```

    1):
    [(4307859, u'Brian@Brea', 416346, u'6', 17935551, u'2013-09-20T08:06:34Z'),
     (4311278, u'oba510', 933797, u'78', 41607115, u'2016-08-22T06:16:47Z'),
     (4336093,
      u'ashleyannmathew',
      4994101,
      u'14',
      51137796,
      u'2017-08-15T11:11:51Z'),
     (4336096, u'RichRico', 2219338, u'22', 49512265, u'2017-06-13T21:14:53Z'),
     (4336229, u'TorieRob', 3821658, u'17', 40191384, u'2016-06-21T21:28:18Z')]
    

>> ways_tags table verification 


```python
cur.execute('SELECT * FROM ways_tags LIMIT 5')
all_rows = cur.fetchall()
print('1):')
pprint(all_rows)
```

    1):
    [(4307859, u'highway', u'service', u'regular'),
     (4307859, u'service', u'driveway', u'regular'),
     (4311278, u'ref', u'G6', u'regular'),
     (4311278, u'name', u'Central Expressway', u'regular'),
     (4311278, u'lanes', u'2', u'regular')]
    

>> ways_nodes table verification 


```python
cur.execute('SELECT * FROM ways_nodes LIMIT 5')
all_rows = cur.fetchall()
print('1):')
pprint(all_rows)
```

    1):
    [(4307859, 519533963, 0),
     (4307859, 2465053159L, 1),
     (4307859, 26066107, 2),
     (4307859, 26066111, 3),
     (4307859, 767478544, 4)]
    

# Conclusion (cleaning and database process)

> I have been able to get data from map file in xml format and audit the map and come up with data quality plan to clean and update the information. 
> With the updated information and using course script, I was able to create csv files

> I created a database and inputted all the csv information to tables in the database 
