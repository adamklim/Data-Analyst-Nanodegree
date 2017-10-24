# Project: Wrangle OpenStreetMap Data (with SQL)


#### Area


I live in Vienna, Austria. I know the city quite well, for this project wanted to select a far away place.


I looked at [http://www.antipodr.com/](http://www.antipodr.com/) and chose [Auckland, New Zealand.](https://mapzen.com/data/metro-extracts/metro/auckland_new-zealand/)

- you can download compressed map file [here](https://s3.amazonaws.com/metro-extracts.mapzen.com/auckland_new-zealand.osm.bz2).

#### Technical notes:

I used [Jupyter notebook](http://jupyter.org/) for code documentation, code can be viewed [here](openstreetmap.ipynb). For dependencies and environment management I use [Conda](https://conda.io/docs/), and the environment file can be found [here](env_dependencies.yaml).



### Data Audit

First step I took was to look at the XML file tags:

* `'bounds': 1`
* `'member': 21117`
* `'nd': 3308115`
* `'node': 2868725`
* `'osm': 1`
* `'relation': 1367`
* `'tag': 1602646,`
* `'way': 316599`

The file contains 1 602 646 unique tags, in the next step I checked the count of tags containing lowercase characters only, lowercase characters with colon, problematic characters, and other types.

* `{'lower': 852693, 'lower_colon': 19124, 'other': 730829, 'problemchars': 0}`



#### Problems in the Map

Initial exploration of the map revealed that there are inconsistencies with how the street names are entered:

* Street names are inconsitently abbreviated eg. names like `"St."`, `"Rd"` together with `"Street"` or `"Road"` are used.
* Lowercase characters are used eg. `"way"` instead of `"Way"`.
    * in the next step I am going to clean up street names.

* Postal codes have correct 4 digit format, but there were postal codes not [listed for Auckland](https://www.google.com/search?q=postal+code+list+auckland) - it's possible that there are other neighboring cities that are included in the map.
    * We'll have a closer look at it once we load the map into database.


### Cleaning street type inconsistencies

In the file expected.csv, I am storing expected values for street types in first row and for postcodes in second row.
the list of expected street types contains: [Avenue, Boulevard, Commons, Court, Crescent, Drive, Lane, Parkway, Place, Road, Square, Street, Way]. We can see a lot of unexpected values, some are abbreviated street types like `"Ave"`, others can be correct types that were not included in the expected list.


```
{'0632': set(['15 Arrenway Dr, Rosedale, Auckland 0632']),
 '16': set(['State Highway 16']),
 '2': set(['State Highway 2']),
 '22': set(['State Highway 22']),
 '26': set(['26']),
 'Auckland': set(['Exmouth Road, Northcote, Auckland']),
 'Ave': set(['Brennan Ave',
             'Delta Ave',
             'Erson Ave',
             'Gillies Ave',
             'Vitasovich Ave',
             'Waverley Ave']),
 'Broadway': set(['Broadway']),
 'Close': set(['Challen Close', 'Court Town Close', 'Regia Close']),
 'Coronation': set(['Coronation']),
 'Cove': set(['Clearwater Cove']),
 'Cr': set(['Marjorie Jayne Cr']),
 'Cresent': set(['Tawa Cresent']),
 'East': set(['Customs Street East',
              'Durham Street East',
              'Greenlane East',
              'Pakenham Street East',
              'Sylvan Avenue East',
              'Victoria Street East',
              'Virginia Avenue East',
              'Wellesley Street East']),
 'Esplanade': set(['The Esplanade']),
 'Expressway': set(['Albany Expressway']),
 'Gardens': set(['Westminster Gardens']),
 'Highway': set(['676 Mount Wellington Highway',
                 'Albany Highway',
                 'Coatesville-Riverhead Highway',
                 'Dairy Flat Highway',
                 'Hibiscus Coast Highway',
                 'Kaipara Coast Highway',
                 'Mount Wellington Highway']),
 'Hill': set(['College Hill']),
 'Howick)': set(['Fisher Parade (Howick)']),
 'Hurstmere': set(['Hurstmere']),
 'Hwy': set(['Main Hwy']),
 'Mall': set(['Onehunga Mall']),
 'Motorway': set(['Auckland Southern Motorway']),
 'Newmarket': set(['Gillies Avenue, Newmarket']),
 'Parade': set(['Fisher Parade', 'King Edward Parade', 'The Parade']),
 'Plc,': set(['Cnr St Lukes Rd & Wagener Plc,']),
 'Ponsonby': set(['Ponsonby']),
 'Rd': set(['Botany Rd',
            'Coster Rd',
            'New North Rd',
            'Ponsonby Rd',
            'Richmond Rd']),
 'Redwood': set(['Redwood']),
 'Rise': set(['Bampton Rise', 'Pacific Rise']),
 'School': set(['Red Beach Primary School']),
 'South': set(['Grange Road South']),
 'St': set(['Balm St', 'Queen St', 'hardinge St']),
 'St.': set(['Queen St.']),
 'Strand': set(['The Strand']),
 'Strret': set(['Cracroft Strret']),
 'Subritzky': set(['Subritzky']),
 'Tai': set(['Ara Tai']),
 'Terrace': set(['Kohia Terrace',
                 'Poynton Terrace',
                 'Quest Terrace',
                 'Scarborough Terrace',
                 'Seaview Terrace',
                 'Shea Terrace',
                 'Sunset Terrace',
                 'Taunton Terrace']),
 'W': set(['Victoria St W']),
 'Wellington': set(['Waipuna Road, Mt Wellington']),
 'West': set(['Crayford Street West',
              'Customs Street West',
              'Green Lane West',
              'Sylvan Avenue West',
              'Tironui Station Road West',
              'Victoria Street West',
              'Wellesley St West',
              'Wellesley Street West',
              'Westmoreland Street West']),
 'Wharf': set(['Princes Wharf']),
 'Zealand': set(['New Zealand']),
 'beach': set(['Little Oneroa beach']),
 'ln': set(['Fort ln']),
 'road': set(['hekerua road']),
 'st': set(['Queen st']),
 'street': set(['Durham street', 'Gore street', 'Haughey street']),
 'way': set(['Kanona way'])}
```

To fix street types I used dictionary that maps changes that will be made in the dataset.
```
mapping = {'Ave': 'Avenue',
           'beach':'Beach',
           'Cr':'Crove',
           'Cresent':'Crescent',
           'ln':'Lane',
           'Hwy':'Highway',
           'Rd':'Road',
           'road':'Road',
           'St':'Street',
           'st':'Street',
           'St.':'Street',
           'street':'Street',
           'Strret':'Street',
           'W':'West',
           'way':'Way',
           'Subritzky':'Subritzky Avenue',
           'Tai':'Tai Road',
           'Hurstmere':'Hurstmere Road'
          }
```

The following changes were being made:

```
('Queen St.', '=>', 'Queen Street')
('Richmond Rd', '=>', 'Richmond Road')
('Botany Rd', '=>', 'Botany Road')
('Ponsonby Rd', '=>', 'Ponsonby Road')
('Coster Rd', '=>', 'Coster Road')
('New North Rd', '=>', 'New North Road')
('Tawa Cresent', '=>', 'Tawa Crescent')
('Haughey street', '=>', 'Haughey Street')
('Gore street', '=>', 'Gore Street')
('Durham street', '=>', 'Durham Street')
('Subritzky', '=>', 'Subritzky Avenue')
('Hurstmere', '=>', 'Hurstmere Road')
('Fort ln', '=>', 'Fort Lane')
('Cracroft Strret', '=>', 'Cracroft Street')
('Kanona way', '=>', 'Kanona Way')
('Main Hwy', '=>', 'Main Highway')
('Little Oneroa beach', '=>', 'Little Oneroa Beach')
('hardinge St', '=>', 'hardinge Street')
('Queen St', '=>', 'Queen Street')
('Balm St', '=>', 'Balm Street')
('Victoria St W', '=>', 'Victoria St West')
('Erson Ave', '=>', 'Erson Avenue')
('Brennan Ave', '=>', 'Brennan Avenue')
('Vitasovich Ave', '=>', 'Vitasovich Avenue')
('Gillies Ave', '=>', 'Gillies Avenue')
('Delta Ave', '=>', 'Delta Avenue')
('Waverley Ave', '=>', 'Waverley Avenue')
('Queen st', '=>', 'Queen Street')
('Marjorie Jayne Cr', '=>', 'Marjorie Jayne Crove')
('Ara Tai', '=>', 'Ara Tai Road')
('hekerua road', '=>', 'hekerua Road')

31 corrections made
```

### Postcodes audit

* `0 postcodes with different format`
* `51 unknown postcodes`


51 postcodes that do not belong to Auckland were identified. We will have a closer look at it while exploring data in SQL database.


### Data preparation for SQL database

In the first step I transformed XML map data to be correct data structure and stored it in .csv files. Those data files will be used for easy data import to SQL database.

The process for this transformation is as follows:
* Use iterparse to iteratively step through each top level element in the XML
* Shape each element into several data structures using a custom function
* Utilize a schema and validation library to ensure the transformed data is in the correct format
* Write each data structure to the appropriate .csv files

### Data exploration in SQL database


After creating empty database and tables to store data I used pandas to import data from csv files to SQL database.

```
df = pandas.read_csv('nodes_tags.csv')
df.to_sql('nodes_tags', conn, if_exists='append', index=False)
df = pandas.read_csv('nodes.csv')
df.to_sql('nodes', conn, if_exists='append', index=False)
df = pandas.read_csv('ways_nodes.csv')
df.to_sql('ways_nodes', conn, if_exists='append', index=False)
df = pandas.read_csv('ways_tags.csv')
df.to_sql('ways_tags', conn, if_exists='append', index=False)
df = pandas.read_csv("ways.csv")
df.to_sql('ways', conn, if_exists='append', index=False)
```

### Data overview

To execute SQL queries in Jupyter I am using [ipython-sql](https://github.com/catherinedevlin/ipython-sql)

Some basic statistics about the data:

#### File sizes:

```
Auckland map      file size: 660.0 MB
nodes.csv         file size: 246.0 MB
nodes_tags.csv    file size: 4.0 MB
ways.csv          file size: 19.0 MB
ways_nodes.csv    file size: 80.0 MB
ways_tags.csv     file size: 77.0 MB
openmap.db        file size: 376.0 MB
```

#### Number of nodes and ways:

```
num_nodes = %sql SELECT COUNT(*) FROM nodes
print 'Number of nodes: {}'.format(num_nodes[0][0])
num_ways = %sql SELECT COUNT(*) FROM ways
print 'Number of ways: {}'.format(num_ways[0][0])
```

* `Number of nodes: 2868725`
* `Number of ways: 316599`

#### Number of unique users:

```
%%sql
SELECT COUNT(DISTINCT(uid)) as 'Unique Users'
FROM (SELECT uid
      FROM nodes
      UNION SELECT uid FROM ways) as e;
```

There are **937 unique users**.

Now, let's look at some othe numbers:
#### How many users contributed only once:

```
%%sql
SELECT COUNT(*) as Users_1_contrib
FROM
    (SELECT e.user, COUNT(*) as num
     FROM (SELECT user FROM nodes UNION ALL SELECT user FROM ways) e
     GROUP BY e.user
     HAVING num=1) a;
```

#### What is the most popular sport in Auckland:

```
%%sql
SELECT value as Sports, COUNT(value) as Popularity
FROM (SELECT * FROM nodes_tags
      UNION ALL SELECT * FROM nodes_tags) as tags
WHERE key = 'sport'
GROUP BY value
ORDER BY Popularity DESC
LIMIT 10;
```

|Sports|Popularity|
|:-----|:---------|
|cricket|62|
|swimming|18|
|basketball|12|
|cricket_nets|12|
|rugby|12|
|soccer|12|
|skateboard|8|
|bowls|4|
|golf|4|
|rugby_union|4|

### Closer look at the postcode issue

As we could see earlier, there were many postcodes in the dataset that do not belong to Auckland. I suspect that the map area covers more than just Auckland, there may be other neighboring cities included, also, some postcodes may be entered incorrectly. Let's check.

#### Cities

```
%%sql
SELECT tags.value as 'City name', COUNT(*) as Count
FROM (SELECT * FROM nodes_tags UNION ALL
      SELECT * FROM ways_tags) tags
WHERE tags.key LIKE '%city'
GROUP BY tags.value
ORDER BY count DESC
LIMIT 10;
```
|City name| Count|
|:--------|:----|
|Auckland | 1005|
|Orewa | 79|
|North Shore City | 69|
|Manukau | 61|
|Papatoetoe | 51|
|Manurewa |  46|
|Otahuhu | 41|
|Waitakere | 32|
|Epsom | 31|
|Takanini | 31|

Some of those names, like `"Epsom"` are actually districts of Auckland, let's check if there are inconsistencies in how city name for Auckland is entered:

```
%%sql
SELECT tags.value as 'City name', COUNT(*) as Count
FROM (SELECT * FROM nodes_tags UNION ALL
      SELECT * FROM ways_tags) tags
WHERE tags.key LIKE '%city'
AND value like '%Auckland%'
GROUP BY tags.value
ORDER BY count DESC
LIMIT 10;
```

|City name |Count|
|:---------|:---|
|Auckland | 1005|
|Hauraki, Auckland | 25|
|auckland  |  6|
|Auckland Central |   4|
|Greenlane, Auckland| 4|
|Mt. Eden, Auckland | 4|
|New Lynn, Auckland | 3|
|Otahuhu, Auckland  | 3|
|Auckland Airport |   2|
|Epsom, Auckland |2|


We can see that there is inconsistency in how the city name is being entered, sometimes city name is replaced by the district name, sometimes city name is entered together with district name.

I am going to check if there is there is inconsistency in entering postcode for entries where city name is entered as  `"Auckland"`.

```
%%sql
SELECT COUNT(*) as Count
FROM(
    SELECT tags.id
    FROM (
        SELECT * FROM nodes_tags
        UNION
        SELECT * FROM ways_tags) tags
    WHERE value like 'Auckland'

    INTERSECT

    SELECT tags.id
    FROM (
        SELECT * FROM nodes_tags
        UNION
        SELECT * FROM ways_tags) tags
    WHERE key = 'postcode'
    and value not in (0600,2024,1010,2022,2102,0632,1023,1025,1071,2016,1011,
                      2104,1061,0626,1072,2023,1026,1022,0610,1051,2025,1042,
                      1062,1041,1021,0612,0620,1052,1060,0624,2105,2103,0602,
                      0622,0627,0630,2571,0629,0618,1081,2576,0604,0614)
);
```
* `Count: 313`

In 313 entries where city name is specified as `"Auckland"` a postcode that does not belong to Auckland is assigned.


# Conclusions
Only basic data audit, cleanup and exploration has been performed. Each of those areas can be further addressed:
* We only looked at the tag type formats, street types and postcodes in data wrangling. Each of those steps can be done in further detail, for example we have 730829 tags that have format `"other"` that can be further investigated.
* Data cleanup was also rudimentary and focused on most common cases of inconsistencies.
* With abundance of different amenities included in the map data, we could keep exploring the dataset, eg. we could look at spatial distribution of points of interest.

It is very difficult to avoid inconsistencies in project like [OpenStreetMap](https://www.openstreetmap.org), where data is entered manually, and also having human cleanup the data seems unrealistic. However, some data checks and triggers could be implemented in the data entry process, for example detection of potential abbreviation, that would warn contributor if it suspects abbreviation is used, but it should allow to enter the value regardless, since the algorithm cannot predict all cases.