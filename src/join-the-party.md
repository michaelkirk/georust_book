# Join the Party

Think about the people who help fill the blank spaces in your life. The shape of each relationship is usually a little different, but it's almost always possible to find some common ground. Even if your friends (incorrectly) think that ["the dress"](https://en.wikipedia.org/wiki/The_dress) is white and gold, they still (correctly) don't like feeling thirsty.

## Drink from the Firehose

Imagine living in New York City. Good pizza, Broadway, and one of the country's best collections of [open data](https://data.ny.gov/) have never felt closer to home. We're looking for a spot to meet up with some friends. Let's start with a published list of every park in the city. Somewhere in these shaded shapes, we can see the faint outline of a future party beginning to form:

![The 2029 parks of New York City](images/nyc-parks.png)

Fortunately for us, we also have a list of every public drinking fountain that will let us sip some of NYC's ["world-renowned"](https://www1.nyc.gov/site/greenyc/take-action/drink-tap-water.page) tap water whenever the conversation starts to feel a little dry:

![The 3120 drinking fountains of New York City](images/nyc-water-fountains.png)

## Filtered Water

We have two different data sources — drinking fountains and parks. Solving spatial problems often involves finding ways to stir things up. Let's see how many of these drinking fountains are inside of parks:

```rust
use geojson::FeatureReader;

use std::fs::File;
use std::io::BufReader;

let parks: Vec<geo::Geometry> = {
  let parks_geojson = BufReader::new(File::open("src/data/nyc/parks.geojson").expect("parks file path must be valid"));
  let parks_reader = FeatureReader::from_reader(parks_geojson);

  parks_reader.features().map(|park_result: geojson::Result<geojson::Feature>| {
    let park_feature = park_result.expect("valid feature");
    geo::Geometry::<f64>::try_from(park_feature).expect("valid conversion")
  }).collect()
};

let drinking_fountains: Vec<geo::Geometry> = {
  let drinking_fountains_geojson = BufReader::new(File::open("src/data/nyc/drinking_fountains.geojson").expect("drinking fountains file path must be valid"));
  let drinking_fountains_reader = FeatureReader::from_reader(drinking_fountains_geojson);

  drinking_fountains_reader.features().map(|drinking_fountain_result: geojson::Result<geojson::Feature>| {
    let drinking_fountain_feature = drinking_fountain_result.expect("valid feature");
    geo::Geometry::<f64>::try_from(drinking_fountain_feature).expect("valid conversion")
  }).collect()
};

let park_has_drinking_water = |park: &geo::Geometry| {
  use geo::algorithm::Contains;
  for drinking_fountain in &drinking_fountains {
    if park.contains(drinking_fountain) {
      return true
    }
  }
  false
};

let mut parks_with_drinking_fountains = 0;
for park in &parks {
  if park_has_drinking_water(park) {
    parks_with_drinking_fountains += 1;
  }
}

assert_eq!(parks_with_drinking_fountains, 902);

let parks_without_drinking_fountains = parks.len() - parks_with_drinking_fountains;
assert_eq!(parks_without_drinking_fountains, 1127);
```

## Thirst for Knowledge

Counting is simple, but oftentimes we will need to do something more complex like taking certain attributes from one data source and combining those attributes with another data source based on their spatial relationship.

Let's look at some selected data from two data sources:

**The Five Boroughs**

| borough_name | borough_shape                 |
|--------------|-------------------------------|
| Brooklyn     | `MULTIPOLYGON(((-73.86327...) |
| Queens       | `MULTIPOLYGON(((-73.82644...) |
| ...          | ...                           |

**List of Parks**

| park_name            | park_shape                       |
|----------------------|----------------------------------|
| Gilbert Ramirez Park | `MULTIPOLYGON(((-73.93418260...` |
| Spargo Park          | `MULTIPOLYGON(((-73.89721950...` |
| Turtle Playground    | `MULTIPOLYGON(((-73.82693385...` |
| ...                  | ...                              |

It's truncated in the tables above, but the borough and park data sources include detailed shape information. The smaller shapes of each *park* can be positioned within the larger shapes that comprise each *borough*. Because our friends live in different areas, a list like this could really help us narrow down where we want to get together:

> - Option 1: Gilbert Ramirez Park in Brooklyn
> - Option 2: Spargo Park in Queens
> - Option 3: Turtle Playground in Queens

In order to produce a list like this, we need to combine the park name from the first data source with the borough that contains it from the second data source.

If you've worked with SQL before, you might be thinking that this sounds a bit like a [JOIN clause](https://en.wikipedia.org/wiki/Join_(SQL)), and you'll no doubt be delighted to know that this kind of operation is indeed referred to as a *spatial join*. If you've never worked with SQL before, don't worry, you have an even bigger reason to be delighted.

```rust
use geo::geometry::{MultiPolygon, Point};
use geo::algorithm::Contains;

use geojson::FeatureReader;

use std::fs::File;
use std::io::BufReader;

// Using what we learned in the previous lesson, we'll
// deserialize the input GeoJSON into a structs.

// First Input
#[derive(serde::Deserialize)]
struct Park {
  #[serde(deserialize_with="geojson::deserialize_geometry")]
  geometry: geo::MultiPolygon,

  #[serde(rename="signname")]
  name: String
}

let parks: Vec<Park> = {
  let parks_geojson = BufReader::new(File::open("src/data/nyc/parks.geojson").expect("parks file path must be valid"));
  let parks_reader = FeatureReader::from_reader(parks_geojson);
  parks_reader.deserialize::<Park>().expect("valid FeatureCollection").collect::<geojson::Result<Vec<_>>>().unwrap()
};

// Second Input
#[derive(serde::Deserialize)]
struct Borough {
  #[serde(deserialize_with="geojson::deserialize_geometry")]
  geometry: geo::Geometry,

  #[serde(rename="boro_name")]
  name: String
}

let boroughs: Vec<Borough> = {
  let boroughs_geojson = BufReader::new(File::open("src/data/nyc/boroughs.geojson").expect("borough file path must be valid"));
  let boroughs_reader = FeatureReader::from_reader(boroughs_geojson);

  // TODO: maybe introduce a geojson::deserialize_iter(boroughs_reader)
  boroughs_reader.deserialize::<Borough>().expect("valid FeatureCollection").collect::<geojson::Result<Vec<_>>>().unwrap()
};

// Output
struct PartyVenue {
  park_geometry: MultiPolygon,
  park_name: String,
  borough_name: String,
}

let mut party_venues: Vec<PartyVenue> = Vec::new();

for park in &parks {
  for borough in &boroughs {
    if borough.geometry.contains(&park.geometry) {
      let venue = PartyVenue {
        park_name: park.name.clone(),
        park_geometry: park.geometry.clone(),
        borough_name: borough.name.clone(),
      };
      party_venues.push(venue);
    }
  }
}

let first_venue = &party_venues[0];
assert_eq!(first_venue.park_name, "Devoe Park");
assert_eq!(first_venue.borough_name, "Bronx");
```

## Water, Cooler

If Central Park is as mainstream as it gets, which open spaces are more like the avant-garde Vapor Wave Jazz that your hipster friends just can't get enough of these days? Let's augment our earlier code to filter out the mainstream (largest) parks, while still ensuring we'll be able to get a drink of New York's finest (water).

**Parks, with at least one drinking fountain, sorted by smallest area**

| park_name                    | borough   | fountains | area     |
|------------------------------|-----------|-----------|----------|
| Tiny Town                    | Queens    | 2         | 100      |
| Polly Pocket Park            | Brooklyn  | 3         | 200      |
| Honey I Shrunk the Esplanade | Bronx     | 1         | 300      |
| ...                          | ...       | ...       | ...      |

```rust
use geo::geometry::{MultiPolygon, MultiPoint, Point};
use geo::algorithm::{Area, Contains};

use geojson::FeatureReader;

use std::fs::File;
use std::io::BufReader;

// First Input
#[derive(serde::Deserialize)]
struct Park {
  #[serde(deserialize_with="geojson::deserialize_geometry")]
  geometry: geo::MultiPolygon,

  #[serde(rename="signname")]
  name: String
}
let parks: Vec<Park> = {
  let parks_geojson = BufReader::new(File::open("src/data/nyc/parks.geojson").expect("parks file path must be valid"));
  let parks_reader = FeatureReader::from_reader(parks_geojson);
  parks_reader.deserialize::<Park>().expect("valid FeatureCollection").collect::<geojson::Result<Vec<_>>>().unwrap()
};

// Second Input
#[derive(serde::Deserialize)]
struct Borough {
  #[serde(deserialize_with="geojson::deserialize_geometry")]
  geometry: geo::Geometry,

  #[serde(rename="boro_name")]
  name: String
}

let boroughs: Vec<Borough> = {
  let boroughs_geojson = BufReader::new(File::open("src/data/nyc/boroughs.geojson").expect("borough file path must be valid"));
  let boroughs_reader = FeatureReader::from_reader(boroughs_geojson);
  boroughs_reader.deserialize::<Borough>().expect("valid FeatureCollection").collect::<geojson::Result<Vec<_>>>().unwrap()
};

// Output
struct PartyVenue {
  park_name: String,
  borough_name: String,
  park_geometry: MultiPolygon,
  square_meters: f64,
  drinking_fountain_count: usize,
}

let mut party_venues: Vec<PartyVenue> = Vec::new();

for park in &parks {
  for borough in &boroughs {
    if borough.geometry.contains(&park.geometry) {
      use proj::Transform;
      // EPSG:32115 - New York Easter (meters)
      let square_meters = park.geometry.transformed_crs_to_crs("WGS84", "EPSG:32115").expect("valid projection").unsigned_area();

      let venue = PartyVenue {
        park_name: park.name.clone(),
        borough_name: borough.name.clone(),
        park_geometry: park.geometry.clone(),
        square_meters,
        drinking_fountain_count: 0, // we'll populate this field next
      };
      party_venues.push(venue);
      break;
    }
  }
}

let drinking_fountain_points: Vec<geo::Point> = {
  let drinking_fountains_geojson = BufReader::new(File::open("src/data/nyc/drinking_fountains.geojson").expect("drinking fountains file path must be valid"));
  let drinking_fountains_reader = FeatureReader::from_reader(drinking_fountains_geojson);

  drinking_fountains_reader.features().map(|drinking_fountain_result: geojson::Result<geojson::Feature>| {
    let drinking_fountain_feature = drinking_fountain_result.expect("valid feature");
    let geometry = geo::Geometry::<f64>::try_from(drinking_fountain_feature).expect("valid conversion");
    geo::Point::<f64>::try_from(geometry).expect("valid conversion")
  }).collect()
};

for drinking_fountain_point in &drinking_fountain_points {
  for party_venue in &mut party_venues {
    if party_venue.park_geometry.contains(drinking_fountain_point) {
      party_venue.drinking_fountain_count += 1;
      break;
    }
  }
}

// We're only interested in parks that have at least one water fountain
party_venues = party_venues.into_iter().filter(|venue| venue.drinking_fountain_count > 0).collect();

// Sort by the size of the park
party_venues.sort_by(|venue_1, venue_2| venue_1.square_meters.partial_cmp(&venue_2.square_meters).expect("valid floating point comparison"));

let tiniest_venue = party_venues.first().unwrap();
assert_eq!(tiniest_venue.park_name, "Glendale Veterans Triangle");
assert_eq!(tiniest_venue.borough_name, "Queens");
approx::assert_relative_eq!(tiniest_venue.square_meters, 26.26352507495168);
assert_eq!(tiniest_venue.drinking_fountain_count, 1);

let largest_venue = party_venues.last().unwrap();
assert_eq!(largest_venue.park_name, "Freshkills Park");
assert_eq!(largest_venue.borough_name, "Staten Island");
approx::assert_relative_eq!(largest_venue.square_meters, 3724056.9377815602);
assert_eq!(largest_venue.drinking_fountain_count, 3);
```

## Drowning in Data

This code works, but it's slower than necessary. So far we've been analyzing data sets without any consideration for speed. We can sort of justify our laissez-faire approach to CPU usage for one-off calculations that won't take too much time to run either way, but optimization will become increasingly important as we begin to work with larger amounts of data. In particular, the way we're checking if each drinking fountain is in each park is pretty naive. We can do better!

In the next section we'll explore some techniques to enhance the performance of some common geospatial operations. Quickly moving through the real world may require a jet engine, but that doesn't mean that your laptop needs to sound like one.
