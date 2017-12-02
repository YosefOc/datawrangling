
# OpenStreetMap Data Case Study (audit only)

> This map is of San Jose Metro. I lived in San Jose for many years. I am curious what additional things I can learn about city and see what I can improve in terms of mapping information. I would like to know basic mapping things about the city. In this exercise, I am going to look at the data and basically audit for the following: 

                            1. City Name- make sure there is one standard 
                            2. Street Types- I will look at the typical street types and standardize some 
                            3. Check for State information and standardize this to one format 
                            4. Check for country designation and standardize this.. 

> The approach or steps I would like to take similiar to what is recommended from course Data Quality Process 
                            
                            Step one: Audit the data                                                                                                
                            Step Two: Create a Data Cleaning Plan                                                                                 
                            Step Three: Execute the Plan 
                            
                            Step Four: Manual Correction(if necessary) 
                            
                            Step Five: re-audit and repeat process as necessary 
                            

# # Auditing the Map data 

> I would like to audit the map data and see if there any issues or errors with map data: i will look at city, street name, state, and country information for the data. 

# Part I: Audit the city information for accuracy and standard naming 

> I would like to first confirm that the data I got only has city of Oakland or is it inclusive of other cities. 
> I can get this information be looking at element-node's sub-element with "tag" label and basically getting the attribute value for k:addr:city  v value.

> I would like to know the count for each city. 


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import xml.etree.cElementTree as ET
from collections import defaultdict
import pprint
import codecs 
import re
# open the osm_file (will create an file object)
filename = "san-jose_california.osm"
osm_file = open(filename, "r")


# you have dictionary(set) for street_types 
city_name_list = defaultdict(int)


# This is function will place every element that has addr:city attribute in dictionary 
# giving me a list of all "cities" in the map. 

def audit_city_name(city_name_list, city_name):
        
        if city_name not in city_name_list: 
            city_name_list[city_name] =1
        else:
            city_name_list[city_name] += 1 

            
def is_city_name(elem):
     return (elem.attrib['k'] == "addr:city")

# function below works with "way" element which is for streets 
# 1. step one is check to see if attribute has addr:street label 
def audit():
    for event, elem in ET.iterparse(osm_file, events=("start",)): 
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"): 
                if is_city_name(tag): 
                    audit_city_name(city_name_list, tag.attrib['v']) 
    pprint.pprint(dict(city_name_list))
    return city_name_list

```

> I used the above script to find what cities are in the map. Although the label from openstreetmap designates as San Jose. I have discovered that it not only has city of San Jose but other cities like Campbell, Sunnyvale, etc. 

> Discovery of Audit- One thing is unicode with accent for San Jose'. it is getting displayed as u'San Jos\xe9. Some cities attribute include the state. Of course there is lower case and upper case usage of the city name. I will utilize a mapping dictionary and tool to standardize the city names: 

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








> Solution: The script and solution for cleaning and updating is with Clean and Convert script Part I. 

# Part II: Audit street type and standard naming convention 


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import xml.etree.cElementTree as ET
from collections import defaultdict
import pandas as pd
import re
import pprint
# open the osm_file (will create an file object)
osm_file = open("san-jose_california.osm", "r")

# create a pattern for regular expression 
pattern = re.compile(r'\S+\.?$', re.IGNORECASE)

# you have dictionary(set) for street_types 
street_types = defaultdict(int)

#have a list of all proper street types because you will be looking street types with error 
expected = ["Street", "Avenue", "Boulevard", "Drive", "Court", "Place", "Square", "Lane", "Road", "Trail", "Parkway", "Commons"] 


def audit_street_type(street_types, street_name):
    result = pattern.search(street_name)
    if result:
        street_type = result.group()
        if street_type not in street_types: 
            street_types[street_type] =1
        else:
            street_types[street_type] += 1 

            
def is_street_name(elem):
     return (elem.attrib['k'] == "addr:street")

# function below works with "way" element which is for streets 
# 1. step one is check to see if attribute has addr:street label 
def audit():
    for event, elem in ET.iterparse(osm_file, events=("start",)): 
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"): 
                if is_street_name(tag): 
                    audit_street_type(street_types, tag.attrib['v']) 
                    
    pprint.pprint(dict(street_types))
    return street_types

```

> In the process of auditing the street types in San Jose. I have noticed a lot street types that end with street name. This is unusual compared to other cities I looked at like Oakland, etc. What is happening or unique about San Jose Metro. 
If you look at the history of San Jose Metro area there are two phases: 

> Phase I: San Jose up until late 70's was mostly a farmland area and it major stop on the El Camino Real rd during the Spanish days of state of California. For example, I see street type with name of Real ( which spanish word for Royal). El Camino Real( Royal Road) extended all the way to Southern California. So, street types indicate names from San Jose's Spanish past. 

>Phase II: 
During the hi-tech boom of 80's and 90's companies moved to area... a lot of residential development started appearing for some reason those new residential development complexes started using streets with latin or italian sounding name or naming convention. For example of via or calle - Via Portofino, Via Marino, Via Palamos, Calle de Barcelona, etc..   

> Discovery of Audit- The history of San Jose shows in the street type. The early spanish influence are still present with major streets like El Camino Real (Royal Road in spanish) and San Jose's recent adoption of italiante sounding street type or name is present with streets with name like Via... or Calle ... 

> The same approach as in case of part with mapping functionality and class example was used to standardize the type of streets. 
  Of course street with italianate or spanish naming convention were left alone. Most of emphasis was on standard street type naming convention- For example from Ln to Lane from Dr to Drive from ave to Avenue.... 
  
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

  
  
  
  

Result- look at "convert and clean" script for coding information 

# Part III: Audit state information for accuracy

This part is about sticking to one format to designate the state which in this case is CA. Other than use of lower case lettering there wasn't much difference. 


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import xml.etree.cElementTree as ET
from collections import defaultdict
import pprint
# open the osm_file (will create an file object)
osm_file = open("san-jose_california.osm", "r")

# you have dictionary(set) for street_types 
state_list = defaultdict()



def audit_state_name(state_list, state_name):
    result = pattern.search(state_name)
    if result:
        state_name = (result.group())    
        if state_name not in state_list: 
            state_list[state_name] =1
        else:
            state_list[state_name] +=1
        
            
def is_state_name(elem):
     return (elem.attrib['k'] == "addr:state")

# function below works with "way" element which is for streets 
# 1. step one is check to see if attribute has addr:street label 
def audit():
    for event, elem in ET.iterparse(osm_file, events=("start",)): 
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"): 
                if is_state_name(tag): 
                    audit_state_name(state_list, tag.attrib['v']) 
    pprint.pprint(dict(state_list))
    return state_list
audit()
```

    {'CA': 2018, 'Ca': 1, 'California': 16, 'ca': 61}
    




    defaultdict(None, {'CA': 2018, 'Ca': 1, 'California': 16, 'ca': 61})



> The state information is minimial. 

mapping_state =  {  'Ca' : 'CA',
                    'California' : 'CA',
                    'ca' : 'CA'            
                    }

# Part V: Audit country information  

> Probably out of all information to audit this one probably the least useful. I just did it to standardize the process. 
> I don't I learned match except that streetMap either shows US or USA for country information. 

mapping_country = {   'US' : 'USA'
                                
                    }




```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import xml.etree.cElementTree as ET
from collections import defaultdict
import pprint
# open the osm_file (will create an file object)
osm_file = open("san-jose_california.osm", "r")

# you have dictionary(set) for street_types 
country_list = defaultdict()

def audit_country_name(country_list, country_name):
    result = pattern.search(country_name)
    if result:
        country_name = (result.group())    
        if country_name not in country_list: 
            country_list[country_name] =1
        else:
            country_list[country_name] +=1
        
            
def is_state_name(elem):
     return (elem.attrib['k'] == "addr:country")

# function below works with "way" element which is for streets 
# 1. step one is check to see if attribute has addr:street label 
def audit():
    for event, elem in ET.iterparse(osm_file, events=("start",)): 
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"): 
                if is_state_name(tag): 
                    audit_country_name(country_list, tag.attrib['v']) 
    pprint.pprint(dict(country_list))
    return country_list
audit()
```

    {'US': 869, 'USA': 1}
    




    defaultdict(None, {'US': 869, 'USA': 1})



# Summary: Audit Process and lessons learned 

> Lesson Learned: 

Information/Data Quality: 

 1. Most user inputted data will have error 
 
 2. I have discovered that city name either misspelled or different format, the street types as demostrated in class material       was also not standardized, I looked state and country info for just standardization 
 
 3. I have discovered for the San Jose metro area- there is adoption of italianate or spanish naming convention for  street    
    which are a recent phenomenon instead of historical spanish influence in the area. I assuming that giving a lot of those   
    streets or areas are post-70s'.  
    
 
 

Data Processing: 

1. Mapping data size is large and most effective way to handle the large XML format in this case is using incremental parsing. Incremental parsing does it at element level instead of whole file. 

 

Possible application: 

There is many things you do with this type of mapping data. I can see a situation where you get information from city of san jose or its surrounding metro about every new street in the city and provide a mapping timeline based on new streets. You can use this information to map the cities or metro growth over several decades and get an idea of possible future growth area for urban planning. 







# Please go the cleaning script for step 2 and 3 of Data Quality Process

                            Step one: Audit the data                                                                                                
                            Step Two: Create a Data Cleaning Plan                                                                                 
                            Step Three: Execute the Plan                                                                                                          
