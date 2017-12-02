
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

> This is final cleaning script to get implemented to create CSV files. Most of script for this part is from course. 

             1. Data.py 
             2. Schema 
     


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
After auditing is complete the next step is to prepare the data to be inserted into a SQL database.
To do so you will parse the elements in the OSM XML file, transforming them from document format to
tabular format, thus making it possible to write to .csv files.  These csv files can then easily be
imported to a SQL database as tables.

The process for this transformation is as follows:
- Use iterparse to iteratively step through each top level element in the XML
- Shape each element into several data structures using a custom function
- Utilize a schema and validation library to ensure the transformed data is in the correct format
- Write each data structure to the appropriate .csv files

We've already provided the code needed to load the data, perform iterative parsing and write the
output to csv files. Your task is to complete the shape_element function that will transform each
element into the correct format. To make this process easier we've already defined a schema (see
the schema.py file in the last code tab) for the .csv files and the eventual tables. Using the
cerberus library we can validate the output against this schema to ensure it is correct.

## Shape Element Function
The function should take as input an iterparse Element object and return a dictionary.

### If the element top level tag is "node":
The dictionary returned should have the format {"node": .., "node_tags": ...}

The "node" field should hold a dictionary of the following top level node attributes:
- id
- user
- uid
- version
- lat
- lon
- timestamp
- changeset
All other attributes can be ignored

The "node_tags" field should hold a list of dictionaries, one per secondary tag. Secondary tags are
child tags of node which have the tag name/type: "tag". Each dictionary should have the following
fields from the secondary tag attributes:
- id: the top level node id attribute value
- key: the full tag "k" attribute value if no colon is present or the characters after the colon if one is.
- value: the tag "v" attribute value
- type: either the characters before the colon in the tag "k" value or "regular" if a colon
        is not present.

Additionally,

- if the tag "k" value contains problematic characters, the tag should be ignored
- if the tag "k" value contains a ":" the characters before the ":" should be set as the tag type
  and characters after the ":" should be set as the tag key
- if there are additional ":" in the "k" value they and they should be ignored and kept as part of
  the tag key. For example:

  <tag k="addr:street:name" v="Lincoln"/>
  should be turned into
  {'id': 12345, 'key': 'street:name', 'value': 'Lincoln', 'type': 'addr'}

- If a node has no secondary tags then the "node_tags" field should just contain an empty list.

The final return value for a "node" element should look something like:

{'node': {'id': 757860928,
          'user': 'uboot',
          'uid': 26299,
       'version': '2',
          'lat': 41.9747374,
          'lon': -87.6920102,
          'timestamp': '2010-07-22T16:16:51Z',
      'changeset': 5288876},
 'node_tags': [{'id': 757860928,
                'key': 'amenity',
                'value': 'fast_food',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'cuisine',
                'value': 'sausage',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'name',
                'value': "Shelly's Tasty Freeze",
                'type': 'regular'}]}

### If the element top level tag is "way":
The dictionary should have the format {"way": ..., "way_tags": ..., "way_nodes": ...}

The "way" field should hold a dictionary of the following top level way attributes:
- id
-  user
- uid
- version
- timestamp
- changeset

All other attributes can be ignored

The "way_tags" field should again hold a list of dictionaries, following the exact same rules as
for "node_tags".

Additionally, the dictionary should have a field "way_nodes". "way_nodes" should hold a list of
dictionaries, one for each nd child tag.  Each dictionary should have the fields:
- id: the top level element (way) id
- node_id: the ref attribute value of the nd tag
- position: the index starting at 0 of the nd tag i.e. what order the nd tag appears within
            the way element

The final return value for a "way" element should look something like:

{'way': {'id': 209809850,
         'user': 'chicago-buildings',
         'uid': 674454,
         'version': '1',
         'timestamp': '2013-03-13T15:58:04Z',
         'changeset': 15353317},
 'way_nodes': [{'id': 209809850, 'node_id': 2199822281, 'position': 0},
               {'id': 209809850, 'node_id': 2199822390, 'position': 1},
               {'id': 209809850, 'node_id': 2199822392, 'position': 2},
               {'id': 209809850, 'node_id': 2199822369, 'position': 3},
               {'id': 209809850, 'node_id': 2199822370, 'position': 4},
               {'id': 209809850, 'node_id': 2199822284, 'position': 5},
               {'id': 209809850, 'node_id': 2199822281, 'position': 6}],
 'way_tags': [{'id': 209809850,
               'key': 'housenumber',
               'type': 'addr',
               'value': '1412'},
              {'id': 209809850,
               'key': 'street',
               'type': 'addr',
               'value': 'West Lexington St.'},
              {'id': 209809850,
               'key': 'street:name',
               'type': 'addr',
               'value': 'Lexington'},
              {'id': '209809850',
               'key': 'street:prefix',
               'type': 'addr',
               'value': 'West'},
              {'id': 209809850,
               'key': 'street:type',
               'type': 'addr',
               'value': 'Street'},
              {'id': 209809850,
               'key': 'building',
               'type': 'regular',
               'value': 'yes'},
              {'id': 209809850,
               'key': 'levels',
               'type': 'building',
               'value': '1'},
              {'id': 209809850,
               'key': 'building_id',
               'type': 'chicago',
               'value': '366409'}]}
"""

import csv   # this modification is remove b' symbol in csv files
import codecs
import pprint
import re
import xml.etree.cElementTree as ET

import cerberus  # this tool used to validate the schema- was data contract met?

import schema  #This is schema is basically the contract for data structure or format

OSM_PATH5 = "oakland_california.osm"
OSM_PATH2 = "example.osm"
OSM_PATH3 = "city_of_alameda.osm"
OSM_PATH = "san-jose_california.osm"
NODES_PATH = "nodes.csv"
NODE_TAGS_PATH = "nodes_tags.csv"
WAYS_PATH = "ways.csv"
WAY_NODES_PATH = "ways_nodes.csv"
WAY_TAGS_PATH = "ways_tags.csv"

lower_colon = re.compile(r'^([a-z]|_)+:([a-z]|_)+')
problemchars = re.compile(r'[=\+/&<>;\'"\?%#$@\,\. \t\r\n]')
pattern_city2 = re.compile(r'^\b\w+.\w+$', re.IGNORECASE)
pattern_city = re.compile(r'^\b\w+\s?....\w+$', re.IGNORECASE)
pattern_street = re.compile(r'\S+\.?$', re.IGNORECASE)
pattern_state = re.compile(r'^\w+$', re.IGNORECASE) 
pattern_country = re.compile(r'^\w+$', re.IGNORECASE)
SCHEMA = schema.schema

# Make sure the fields order in the csvs matches the column order in the sql table schema
NODE_FIELDS = ['id', 'lat', 'lon', 'user', 'uid', 'version', 'changeset', 'timestamp']
NODE_TAGS_FIELDS = ['id', 'key', 'value', 'type']
WAY_FIELDS = ['id', 'user', 'uid', 'version', 'changeset', 'timestamp']
WAY_TAGS_FIELDS = ['id', 'key', 'value', 'type']
WAY_NODES_FIELDS = ['id', 'node_id', 'position']


# this mapping dictionary I will use standardize the city naming convention
mapping_city = { 'Campbelll' : 'Campbell',
                 'Los Gato' : 'Los Gatos',
                 'Los Gatos, CA' : 'Los Gatos',
                 'San Jos\xe9' : 'San Jose',
                 'San jose' : 'San Jose',
                 'Santa clara' : 'Santa Clara',
                 'Sunnyvale, CA' : 'Sunnyvale',
                 'campbell' : 'Campbell',
                 'cupertino' : 'Cupertino',
                 'los gatos':'Los Gatos',
                 'San jose' : 'San Jose',
                 'san Jose' : 'San Jose',
                 'santa Clara' : 'Santa Clara',
                 'santa clara' : 'Santa Clara',
                 'sunnyvale' : 'Sunnyvale',
                 'SUnnyvale' : 'Sunnyvale',
                 'San José'  : 'San Jose',
                 'san jose'  : 'San Jose'               
                    }

mapping_street = {  'Ave' : 'Avenue',
                    'Blvd' : 'Boulevard',
                    'Blvd.' : 'Boulevard',
                    'Boulvevard' : 'Boulevard',
                    'Cir' : 'Circle',
                    'Ct' : 'Court',
                    'Dr' : 'Drive',
                    'Hwy' : 'Highway',
                    'Ln' : 'Lane',
                    'Rd' : 'Road',
                    'St' : 'Street',
                    'ave' : 'Avenue',
                    'street' : 'Street'        
                    }

mapping_state =  {  'Ca' : 'CA',
                    'California' : 'CA',
                    'ca' : 'CA'            
                    }

mapping_country = {   'US' : 'USA'
                                
                    }

def update_city(name_city, mapping_city):
   
    m = pattern_city.search(name_city)       # does the name match the regular expression pattern
    if m and m.group() in mapping_city:  # if pattern matches(1) and value(name) is in the mapping dictionary
        name_city = re.sub(pattern_city, mapping_city[m.group()], name_city)  # change the name with mapping proper name
        
    return name_city   # return the mapping name ( for example if it is "sunnyvale" return "Sunnyvale")

def update_street(name_type, mapping_street):
    m = pattern_street.search(name_type)       # does the name match the regular expression pattern
    if m and m.group() in mapping_street:  # if pattern matches(1) and value(name) is in the mapping dictionary
        name_type = re.sub(pattern_street, mapping_street[m.group()], name_type)  # change the name with mapping proper name   
    return name_type   # return the mapping name ( for example if it is "Blvd" return "Boulevard")

def update_state(name_state, mapping_state):
    m = pattern_state.search(name_state)       # does the name match the regular expression pattern
    if m and m.group() in mapping_state:  # if pattern matches(1) and value(name) is in the mapping dictionary
        name_state = re.sub(pattern_state, mapping_state[m.group()], name_state)  # change the name with mapping proper name
    return name_state   # return the mapping name ( for example if it is "ca" return "CA")

def update_country(name_country, mapping_country):
    m = pattern_country.search(name_country)       # does the name match the regular expression pattern
    if m and m.group() in mapping_country:  # if pattern matches(1) and value(name) is in the mapping dictionary
        name_country= re.sub(pattern_country, mapping_country[m.group()], name_country)  # change the name with mapping proper name
    return name_country   # return the mapping name ( for example if it is "US" return "USA")




""" This function above should standardize the city name i.e. from Oakland, CA to Oakland"""

def shape_element(element, node_attr_fields=NODE_FIELDS, way_attr_fields=WAY_FIELDS,
                  problem_chars=problemchars, default_tag_type='regular'):
    """Clean and shape node or way XML element to Python dict"""

    node_attribs = {}
    way_attribs = {}
    way_nodes = []
    tags = []
     # Handle secondary tags the same way for both node and way elements
# getting the node attributes via dictionary node_attribs = {}

    # this for main branch: "node"

    if  element.tag == 'node':  # this will look at Tree and find element with label of "node"

        try:

            for attrib in NODE_FIELDS:   # this is to get the "node" attributes ( file called nodes.csv will store this)
                node_attribs[attrib] = element.attrib[attrib]

        except:
                # see what the issue is, remove when done
                print(attrib)
                # if the attribute doesn't exist, fill it with something
                node_attribs[attrib] = '000'


        for child in element.iter("tag"):   # child of element labelled "node" and child has label "tag"
            tag = {}   # dictionary to hold sub-element(child)'s attributes '
            tag['id'] = element.attrib['id']  # this id is also node id

            if problemchars.search(child.attrib["k"]):  # find unsual characters in child's attribute key
                continue

            elif lower_colon.search(child.attrib['k']):
                tag['key'] = child.attrib['k'].split(':',1)[1]
                tag['value'] =child.attrib['v']
                tag['type'] = child.attrib['k'].split(':',1)[0]

# in this part I use update_city to standardize cities name:
                if child.attrib['k'] == 'addr:city':                                      
                    tag['value'] = update_city(child.attrib['v'], mapping_city)
                   
# in this part I use update_street to standardize street types:            
                elif child.attrib["k"] == 'addr:street':
                    tag['value'] = update_street(child.attrib['v'], mapping_street)
                    
# in this part I use update_state to standardize state name:                    
                elif child.attrib["k"] == 'addr:state':
                    tag['value'] = update_state(child.attrib['v'], mapping_state)
            
# in this part I use update_country to standardize country name:                    
                elif child.attrib["k"] == 'addr:country':
                    tag['value'] = update_country(child.attrib['v'], mapping_country)   
                   
    # otherwise:
                else:
                    tag['value'] = child.attrib['v']

            else:

                tag['key'] = child.attrib['k']
                tag['value'] =child.attrib['v']
                tag['type'] = "regular"


            if tag:
                tags.append(tag)


    elif element.tag == "way": # if no element with label "node" found- logic will come here
    # this is for second branch: "way"
        way_attribs={}
        tagway = {}
        way_nodes = []
        counter = 0


        for attrib in WAY_FIELDS:
            way_attribs[attrib] = element.attrib[attrib]

        for child in element.iter():   #this is for children of way element- it has two children "nd" and "tag"

            if  child.tag == "nd": # if child is labelled "nd"
                way_attrib={}
                way_attrib['id'] = element.attrib['id']
                way_attrib['node_id'] = child.attrib['ref']
                way_attrib['position'] = counter
                way_nodes.append(way_attrib)
                counter+= 1

            elif child.tag == "tag":
                tag = {}
                tag['id'] = element.attrib['id']

                if  problemchars.search(child.attrib['k']):
                    continue

                elif lower_colon.search(child.attrib['k']):
                    tag['key'] = child.attrib['k'].split(':',1)[1]
                    tag['value'] =child.attrib['v']
                    tag['type'] = child.attrib['k'].split(':',1)[0]

                else:
                    tag['key'] = child.attrib['k']
                    tag['value'] =child.attrib['v']
                    tag['type'] = "regular"

                if tag:
                    tags.append(tag)



    if element.tag == 'node':  # for element with label "node" function will return dictionary
        return {'node': node_attribs, 'node_tags': tags}
    elif element.tag == 'way':
        return {'way': way_attribs, 'way_nodes': way_nodes, 'way_tags': tags}


# ================================================== #
#               Helper Functions                     #
# ================================================== #
def get_element(osm_file, tags=('node', 'way', 'relation')):
    """Yield element if it is the right type of tag"""

    context = ET.iterparse(osm_file, events=('start', 'end'))
    _, root = next(context)
    for event, elem in context:
        if event == 'end' and elem.tag in tags:
            yield elem
            root.clear()


def validate_element(element, validator, schema=SCHEMA):
    """Raise ValidationError if element does not match schema"""
    if validator.validate(element, schema) is not True:
        field, errors = next(validator.errors.iteritems())
        message_string = "\nElement of type '{0}' has the following errors:\n{1}"
        error_string = pprint.pformat(errors)

        raise Exception(message_string.format(field, error_string))
class UnicodeDictWriter(csv.DictWriter, object):
    """Extend csv.DictWriter to handle Unicode input"""

    def writerow(self, row):
        super(UnicodeDictWriter, self).writerow({
            k: (v.encode('utf-8') if isinstance(v, unicode) else v) for k, v in row.iteritems()
        })

    def writerows(self, rows):
        for row in rows:
            self.writerow(row)


# ================================================== #
#               Main Function                        #
# ================================================== #
def process_map(file_in, validate):
    """Iteratively process each XML element and write to csv(s)"""

    with codecs.open(NODES_PATH, 'w') as nodes_file, \
         codecs.open(NODE_TAGS_PATH, 'w') as nodes_tags_file, \
         codecs.open(WAYS_PATH, 'w') as ways_file, \
         codecs.open(WAY_NODES_PATH, 'w') as way_nodes_file, \
         codecs.open(WAY_TAGS_PATH, 'w') as way_tags_file:

        nodes_writer = UnicodeDictWriter(nodes_file, NODE_FIELDS)
        node_tags_writer = UnicodeDictWriter(nodes_tags_file, NODE_TAGS_FIELDS)
        ways_writer = UnicodeDictWriter(ways_file, WAY_FIELDS)
        way_nodes_writer = UnicodeDictWriter(way_nodes_file, WAY_NODES_FIELDS)
        way_tags_writer = UnicodeDictWriter(way_tags_file, WAY_TAGS_FIELDS)


        nodes_writer.writeheader()
        node_tags_writer.writeheader()
        ways_writer.writeheader()
        way_nodes_writer.writeheader()
        way_tags_writer.writeheader()

        validator = cerberus.Validator()

        for element in get_element(file_in, tags=('node', 'way')):
            el = shape_element(element)
            if el:
                if validate is True:
                    validate_element(el, validator)

                if element.tag == 'node':
                    nodes_writer.writerow(el['node'])
                    node_tags_writer.writerows(el['node_tags'])
                elif element.tag == 'way':
                    ways_writer.writerow(el['way'])
                    way_nodes_writer.writerows(el['way_nodes'])
                    way_tags_writer.writerows(el['way_tags'])


if __name__ == '__main__':
    # Note: Validation is ~ 10X slower. For the project consider using a small
    # sample of the map when validating.
    process_map(OSM_PATH, validate=True)

```

    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Morgan Hill
    Morgan Hill
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Campbell
    Campbell
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    San José
    San José
    Morgan Hill
    Morgan Hill
    Campbell
    Campbell
    Saratoga
    Saratoga
    Campbell
    Campbell
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    Morgan Hill
    Morgan Hill
    san jose
    San Jose
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Santa Clara
    Santa Clara
    Campbell
    Campbell
    Sunnyvale
    Sunnyvale
    Saratoga
    Saratoga
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Campbell
    Campbell
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Campbell
    Campbell
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Saratoga
    Saratoga
    Mt Hamilton
    Mt Hamilton
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Milpitas
    Milpitas
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    San José
    San José
    Sunnyvale
    Sunnyvale
    Milpitas
    Milpitas
    Sunnyvale
    Sunnyvale
    San José
    San José
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Morgan Hill
    Morgan Hill
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Cupertino
    Cupertino
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Campbell
    Campbell
    Campbell
    Campbell
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Saratoga
    Saratoga
    Sunnyvale
    Sunnyvale
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San José
    San José
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Los Gatos
    Los Gatos
    Felton
    Felton
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    Milpitas
    Milpitas
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Morgan Hill
    Morgan Hill
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Campbell
    Campbell
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Cupertino
    Cupertino
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Campbell
    Campbell
    Campbell
    Campbell
    San Jose
    San Jose
    Cupertino
    Cupertino
    Sunnyvale
    Sunnyvale
    san jose
    San Jose
    Milpitas
    Milpitas
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    New Almaden
    New Almaden
    Campbell
    Campbell
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    San Jose
    San Jose
    Los Gatos, CA
    Los Gatos, CA
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Cupertino
    Cupertino
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Cupertino
    Cupertino
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Morgan Hill
    Morgan Hill
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    San José
    San José
    San José
    San José
    San José
    San José
    san jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    san Jose
    San Jose
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San José
    San José
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Milpitas
    Milpitas
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    Milpitas
    Milpitas
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Campbell
    Campbell
    San José
    San José
    San Jose
    San Jose
    Campbell
    Campbell
    Milpitas
    Milpitas
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    san Jose
    San Jose
    San Jose
    San Jose
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    Milpitas
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Sunnyvale
    Sunnyvale
    Milpitas
    Milpitas
    sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    Sunnyvale
    Sunnyvale
    Campbell
    Campbell
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San José
    San José
    San José
    San José
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    Cupertino
    San Jose
    San Jose
    San Jose
    San Jose
    San José
    San José
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Milpitas
    Milpitas
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Saratoga
    Saratoga
    Saratoga
    Saratoga
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Coyote
    Coyote
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Saratoga
    Saratoga
    Santa Clara
    Santa Clara
    Milpitas
    Milpitas
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Saratoga
    Saratoga
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    Morgan Hill
    Morgan Hill
    Fremont
    Fremont
    Los Gatos
    Los Gatos
    Morgan Hill
    Morgan Hill
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San José
    San José
    Saratoga
    Saratoga
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San Jose
    San Jose
    Alviso
    Alviso
    Cupertino
    Cupertino
    Sunnyvale
    Sunnyvale
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Morgan Hill
    Morgan Hill
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Santa Clara
    Santa Clara
    San José
    San José
    Santa Clara
    Santa Clara
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    San Jose
    San Jose
    San José
    San José
    San Jose
    San Jose
    Sunnyvale
    Sunnyvale
    Santa Clara
    Santa Clara
    Santa Clara
    Santa Clara
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    Los Gatos
    San Jose
    San Jose
    San José
    San José
    San Jose
    San Jose
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San José
    San Jose
    San Jose
    Santa Clara
    Santa Clara
    

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
