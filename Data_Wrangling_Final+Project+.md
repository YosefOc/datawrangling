
# OpenStreetMap Data Case Study

_____________________________________________________________________________________________________________________________

### Map Area

This map is of San Jose Metro. I lived in San Jose for many years. I am curious what additional things I can learn about city and its neighboring cities of cupertino, santa clara, sunnyvale, etc. and see what I can improve in terms of mapping information. 

https://mapzen.com/data/metro-extracts/metro/san-jose_california/


There are two other files in this github folder that can help meet the Data Wrangling Rubric requirement in terms of Code. 

____________________________________________________________________________________________________________________

# Audit Map and improve data quality

I utilized the course material as guideline for what audit. The data quality section discussed five common correction that are made to data to improve quality: 
                 1. Removing typographical errors 
                 2. Validating against known entities 
                 3. Cross-checking with other datasets 
                 4. Data Harmonization  
                 5. Changing Reference Data (for example form two digit to three digit- US to USA)


I chose to work mainly on the Data Harmonization. I decided to look at how the cities name, Street type name, State and Country name. 


         

### Data Harmonization problems: 
               1. City Name: For example, for city of "Los Gatos" 
               We have 'Los Gato', 'Los Gatos, CA',
               
               2. Street Type Name: For example, for street type "Avenue" 
               We have 'Ave', 'Ave.', etc 
               
               3. State Name: For example, for state of "California" 
               We have 'ca', 'CA',
               
               4. Country Name: For example, for country of "United States" 
               We have 'US', 'USA',
               

                              

 ### Code for audit 

In terms of code the course data.py, schema.py were utilized. 
I just focused element attribute I need for each of data I wanted: city name, street type, etc 
The code below is sample for the city name.
Since I am only doing Data Harmonization- the coding from auditing to cleaning was a matter of changing the element type. 

#### Finding elements( "city name")
def audit_city_name(city_name_list, city_name):
        
        if city_name not in city_name_list: 
            city_name_list[city_name] =1
        else:
            city_name_list[city_name] += 1 


def is_city_name(elem):
        return (elem.attrib['k'] == "addr:city")
 

def audit():
    for event, elem in ET.iterparse(osm_file, events=("start",)): 
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"): 
                if is_city_name(tag): 
                    audit_city_name(city_name_list, tag.attrib['v']) 
    pprint.pprint(dict(city_name_list))
    return city_name_list       
#### Cleaning and updating information 

For the city name harmonization, I basically utilized function with mapping and regular expression to standardize the name and I inserted this function into the shape_element function from course. this function is called within the Shape_element function to make the change.. The mapping_city is below in script
One function update and change name- 

def update_city(name_city, mapping_city):

             m = pattern_city.search(name_city)       # does the name match the regular expression pattern
             if m and m.group() in mapping_city:  # if pattern matches(1) and value(name) is in the mapping dictionary
                name_city = re.sub(pattern_city, mapping_city[m.group()], name_city)  # change the name 
             return name_city   # return the mapping name ( for example if it is "sunnyvale" return "Sunnyvale") """
             

if child.attrib['k'] == 'addr:city':                                      
                tag['value'] = update_city(child.attrib['v'], mapping_city)
             
             
### City name (these are major cities in the map) 
SELECT tags.value, COUNT(*) as count 
FROM (SELECT * FROM nodes_tags UNION ALL 
      SELECT * FROM ways_tags) tags
WHERE tags.key= 'city' AND tags.type='addr'
GROUP BY tags.value
ORDER BY count DESC;Sunnyvale|3426
San Jose|1098
Morgan Hill|401
Santa Clara|335
Saratoga|233
Los Gatos|148
San Jos├⌐|129
Milpitas|105
Campbell|80
Cupertino|62
Alviso|11
Mountain View|7
> Although Openstreetmap classifies the map as "San Jose".. a significant part of it is Sunnyvale. 
  So, the map is "Silicon Valley" and not just city of San Jose. Another thing is my script was not able completely Harmonize the city name. The challenge here is San Jose' with accent(spanish form). I only encountered this with python 2.7 and not latest python. 

### Top 20 streets in map
Stevens Creek Boulevard|285
Hollenbeck Avenue|175
South Stelling Road|130
East Estates Drive|123
Johnson Avenue|120
Bollinger Road|117
Miller Avenue|117
South De Anza Boulevard|112
North Santa Cruz Avenue|106
West Remington Drive|103
Rainbow Drive|99
South Blaney Avenue|99
North 1st Street|98
South Tantau Avenue|94
North Murphy Avenue|93
Barnhart Avenue|86
Bittern Drive|84
East El Camino Real|84
West El Camino Real|83
Rodrigues Avenue|81
> All these street are major historical streets or roads that cut thru the map. 
 All names like De Anza or El Camino Real go back more 130 years... 

### Harmonizing the State and Country name 
SELECT tags.value, COUNT(*) as count 
FROM (SELECT * FROM nodes_tags UNION ALL 
      SELECT * FROM ways_tags) tags
WHERE tags.key= 'state' AND tags.type='addr'
GROUP BY tags.value
ORDER BY count DESC;
________________

CA|2138

SELECT tags.value, COUNT(*) as count 
FROM (SELECT * FROM nodes_tags UNION ALL 
      SELECT * FROM ways_tags) tags
WHERE tags.key= 'country' AND tags.type='addr'
GROUP BY tags.value
ORDER BY count DESC;
________________

USA|893



> We can see that the state and country name have one form CA or USA instead of what was existing in the database. 

### Observations from above results 

> I decide to focus on data harmonization for my investigation and I also decided to use openstreetmap 
and input some in location to see how the process works. I believe the database could have better quality if certain field
automatically take a certain format. 

# Data Overview and Additional Ideas

This section contains basic statistics about the dataset, the SQLite3 queries used to gather them, and some additional ideas about the data in context.

 ### File sizes

 ```
san-jose_california.osm ......... 376 MB
sanjose_map.db ...................654 MB
nodes.csv ....................... 147 MB
nodes_tags.csv .................. 3.00 MB
ways.csv ........................ 14.0 MB
ways_tags.csv ................... 23 MB
ways_nodes.cv ................... 50 MB  
```  

### Number of nodes
sqlite> SELECT COUNT(*) FROM nodes;

Numbers of nodes: 5312787
### Number of ways
sqlite> SELECT COUNT(*) FROM ways;

Numbers of ways: 712266
### Number of unique users
sqlite> SELECT COUNT(DISTINCT(e.uid))          
FROM (SELECT uid FROM nodes UNION ALL SELECT uid FROM ways) e;

Numbers of users: 1427

### Top 10 contributing users
andygol......887148 
nmixter......849579
mk408........406593
Bike Mapper..271557
samely.......242739
RichRico.....227739
dannykath....222279
MustangBuyer.194433
karitotp.....187584
Minh Nguyen..148536
# Other ideas about the dataset

> I wanted to know why there are different format for state from "ca" to "CA" to "Ca"... I also wanted know why there 
different format for cities ranging with "los gatos" to "Los gatos" to "Los Gatos,CA", etc.. what was contributing to this lack of data harmonization. 

> I decided to input two business locations that weren't on the map near where I live in Berkeley. When I manually inputted the information by placing pin, it seems city, state, etc information was automatically populated in a nice standard format for example "CA" for state or city Berkeley. I had to manually input the information to change to something like "ca". 

> My theory is that if information is manually inputted the system has some standard default setting at least for California map I was working with.. .
it at least complies with 
https://en.wikipedia.org/wiki/ISO_3166-2:US





> Maybe the "automatic update" approach might be affecting the quality of data and based on the top users numbers above there is a good chance that some of harmonization issue with database might be caused by automatic update at least for some of fields. 

> The users that highest number of changes and inputs to San Jose/Silcon Valley map is user.. 
https://www.openstreetmap.org/user/andygol

### Benefits/Anticipated Issues Portion

> My recommendation is that the database should gray out certain field and they should be automatically updated or populated by tool based on things like ISO 3166- I think this would make it easier for user to input information and reduce data quality issues. 
I think some users might not be happy with situation given that openstreetmap is all about users inputting information.. but I think there has to be balance to improve or maintain a certain quality- I can't see a reason to make all fields editable?  

# Conclusion 

> I found mapping information for San Jose/Silicon Valley interesting. It is such large database and you don't know where to start in terms of investigating its quality. In course, we learned about the five measure of quality: validity, accuracy, completeness, consistency, and finally uniformity- I decided to work on the easiest in this case- uniformity in terms of data harmonization. 

> I believe to maintain certain quality i.e. data harmonization some fields need to be stricted or populated via from information from some ISO source(gold standard) information. 

