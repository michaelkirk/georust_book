# Data Sources

## Parks_Properties.csv

Description: NYC Public Parks
Download: https://nycopendata.socrata.com/api/views/enfh-gkve/rows.csv?accessType=DOWNLOAD
From Page: https://nycopendata.socrata.com/Recreation/Parks-Properties/enfh-gkve

## DrinkingFountains_20190417.csv

Description: NYC Drinking Water Fountains CSV
Download: https://data.cityofnewyork.us/api/views/bevm-apmm/rows.csv?accessType=DOWNLOAD
From Page: https://data.cityofnewyork.us/Environment/NYC-Parks-Drinking-Fountains/622h-mkfu

## NYC-Parks-Drinking-Fountains.geojson

Description: NYC Drinking Water Fountains
From Page: https://data.cityofnewyork.us/Environment/NYC-Parks-Drinking-Fountains/622h-mkfu

## Borough Boundaries.geojson 

From Page: https://data.cityofnewyork.us/City-Government/Borough-Boundaries/tqmj-j8zm

## DPR_Concessions_001.json

Description: NYC Directory of Concessions
Download: https://www.nycgovparks.org/bigapps/DPR_Concessions_001.json
From Page: https://data.cityofnewyork.us/Housing-Development/Directory-of-Concessions/ac9y-je94

## DPR_Concessions_001.geojson

Transformed  DPR_Concessions_001.json into a GeoJSON FeatureCollection via:

    cat DPR_Concessions_001.json| jq '{type: "FeatureCollection", features: [.[]|{type: "Feature", geometry: {type: "Point", coordinates: [.locations[]|.[]|.lng,.lat|tonumber]}, properties: . }]}' > DPR_Concessions_001.geojson

### Data Dictionary

Name: the name of the concession
Location: address or text description of the location

Park_ID: A unique identifier for the property. The first character is a abbreviation of the borough,
followed by a 3 digit number. Anything after the first 4 characters represents a subproperty or inspection zone.
Boroughs:
X - Bronx
B - Brooklyn
M - Manhattan
Q - Queens
R - Staten Island
See http://www.nycgovparks.org/bigapps/DPR_Parks_001.xml or http://www.nycgovparks.org/bigapps/DPR_Parks_001.json

Permit start date: the date the permit starts
Permit end date: the date the permit expires

Description: HTML description of the concession
Type: the type of the concession
Emails: contact email addresses
Phones: contact phone numbers
Websites: the official site of the concession
Locations: the latitude and longitude of the concession.

## Philly Waterways

Downloaded Hydrographic_Features_Poly.geojson from https://www.opendataphilly.org/dataset/hydrology
converted to csv:

    geozero source/Hydrographic_Features_Poly.geojson philly_waterways.csv

*NOTE*: Be sure to get the "Poly" version, not the "Line" version of that file.

*NOTE*: There is an existing .csv provided on that same page, but it lacks
a geomtry column, hence why we download the geojson and convert it.

Opened in spreadsheet app (Numbers.app):
  - deleted columns except: geometry, creek_name, and inf1
  - reordered geometry to be last column
  - lowercased remaining columns

convert csv back to geojson (to get a geojson which has only our necessary columns)

  geozero philly_waterways.csv --csv-geometry-column=geometry philly_waterways_unformatted.geojson

  # format geojson
  jq . < philly_waterways_unformatted.geojson > philly_waterways.geojson

