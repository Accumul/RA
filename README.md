# README TASK 1
Task 1 mainly consists of three parts: collecting raw data, processing the data to obtain a cleaned file, and analyzing data. It provides data support for the overall research point of view.
## 1. Collecting Raw Data
Raw data comes from <http://data.gdeltproject.org/events/index.html>, which is open to the public. It contains over a quarter-billion records from 1979 to today.  
For downloading files in batches, I use **wget** to obtain all compressed files in the URL. The size of raw data is around 234 GB. 
### 1.1 Data overview
The raw data from 1979 to 2005 is collected by year. From 2005 to March 2013, the raw data is collected by month. Then, it is collected by day from April 2013 to now. The data we need is in the first part of raw data, so we could discard the redundant part. For the needed data, there are 57 variables from 1979 to March 2013 and 58 variables from April 2013 to now. The difference is that the last variable **SOURCEURL** was added. Then, 28 variables are selected as the content of cleaned data.
### 1.2 Variable descriptions
- **GlobalEventID:** Globally unique identifier assigned to each event record that uniquely identifies it in the master dataset.
- **Day:** Date the event took place in YYYYMMDD format.
- **Actor1Name:** The complete raw CAMEO code for Actor1.
- **Actor1CountryCode:**  The 3-character CAMEO code for the country affiliation of Actor1.
- **Actor1Type1Code:** The 3-character CAMEO code of the CAMEO “type” or “role” of Actor1, if specified.
- **Actor1Type2Code:** If multiple type/role codes are specified for Actor1, this returns the second code.
- **Actor1Type3Code:** If multiple type/role codes are specified for Actor1, this returns the third code.
- The 5 variables above are repeated for Actor2. The set of fields above is repeated, but each is prefaced with “Actor2” instead of “Actor1”. The definitions and values of each field are the same as above.
- **Actor1Geo_Type** This field specifies the geographic resolution of the match type and holds one of the following values: 1=COUNTRY, 2=USSTATE, 3=USCITY, 4=WORLDCITY, 5=WORLDSTATE.
- **Actor1Geo_Fullname** This is the full human-readable name of the matched location. 
- **Actor1Geo_CountryCode** This is the 2-character FIPS10-4 country code for the location.
- The 3 variables above are repeated for Actor2 and Action, using those prefixes.
- **IsRootEvent** The system codes every event found in an entire document, using an array of techniques to deference and link information together. 
- **EventCode** This is the raw CAMEO action code describing the action that Actor1 performed upon Actor2.
- **EventBaseCode** CAMEO event codes are defined in a three-level taxonomy.
- **EventRootCode** Similar to EventBaseCode, this defines the root-level category the event code falls under.
- **QuadClass** The entire CAMEO event taxonomy is ultimately organized under four primary classifications: Verbal Cooperation, Material Cooperation, Verbal Conflict, and Material Conflict.
- **GoldsteinScale**  Each CAMEO event code is assigned a numeric score from -10 to +10, capturing the theoretical potential impact that type of event will have on the stability of a country.
- **NumMentions** This is the total number of mentions of this event across all source documents.
- **NumSources** This is the total number of information sources containing one or more mentions of this event.
- **NumArticles** This is the total number of source documents containing one or more mentions of this event.
- **AvgTone** This is the average “tone” of all documents containing one or more mentions of this event.
## 2. Processing The Data
Since we focus on the relationships between countries, we need to filter out data without country information. For data which misses the *Actor1CountryCode* or *Actor2CountryCode*, we fill them with corresponding Geo_CountryCode (*Actor1Geo_CountryCode* or *Actor2Geo_CountryCode*).  
```py  
df1.loc[pd.isnull(df1['Actor1CountryCode']), 'Actor1CountryCode'] = df1[pd.isnull(df1['Actor1CountryCode'])]['Actor1Geo_CountryCode']
```
In this process, we have to transform the *ActorXGeo_CountryCode* which is the 2-character FIPS10-4 country code to *ActorXCountryCode* which is the 3-character CAMEO code. For example, we transform CHINA's code from **CH** to **CHN**. 
```py  
df['Actor1CountryCode'] = df['Actor1CountryCode'].replace('CH', 'CHN')
```
Then, we keep events where both Actor1CountryCode and Actor2CountryCode is populated with a country name and drop data missing for *Actor1CountryCode*.  
```py  
df1.dropna(subset=['Actor1CountryCode', 'Actor2CountryCode'], axis=0, how='any', inplace=True)
```
For the following data analysis，we multiply Goldstein scale by -1.  
```py  
df1['GoldsteinScale'] = df1['GoldsteinScale'] * (-1)
```
## 3. Analyzing The Data
Based on the cleaned file, We can study different combinations of variables. For example, we could observe the trend of the *Goldstein Scale* between any two countries over time to analyze the relationship and disputes between them.
