# Join the Party

Think about the people who help fill the blank spaces in your life. The shape of each relationship is usually a little different, but it's almost always possible to find some common ground. Even if your friends (incorrectly) think that ["the dress"](https://en.wikipedia.org/wiki/The_dress) is white and gold, they still (correctly) don't like feeling thirsty.

## Drink from the Firehose

Imagine living in New York City. Good pizza, Broadway, and one of the country's best collections of [open data](https://data.ny.gov/) have never felt closer to home. We're looking for a spot to meet up with some friends. Let's start with a published list of every park in the city. Somewhere in these shaded shapes, we can see the faint outline of a future party beginning to form:

![The 2029 parks of New York City](images/nyc-parks.png)

Fortunately for us, we also have a list of every public water fountain that will let us sip some of NYC's ["world-renowned"](https://www1.nyc.gov/site/greenyc/take-action/drink-tap-water.page) tap water whenever the conversation starts to feel a little dry:

![The 3120 water fountains of New York City](images/nyc-water-fountains.png)

## Filtered Water

We have two different data sources — water fountains and parks. Solving spatial problems often involves finding ways to stir things up. Let's see how many of these water fountains are inside of parks:

```rust
use geo::{GeometryCollection, MultiPoint};
use geojson::FromGeoJson;

let parks_geojson = todo!("get geojson string");
let water_fountains_geojson = todo!("get geojson string);
let parks = GeometryCollection::from_geojson_str(&parks_geojson).unwrap();
let water_fountains = MultiPoint::from_geojson_str(&water_fountains_geojson).unwrap();

let mut water_fountains_in_parks = 0;
for water_fountain in &water_fountains {
  if parks.contains(&water_fountain) {
    water_fountains_in_parks += 1;
  }
}

assert_eq!(water_fountains_in_parks, 1234);
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
use geo::algorithm::{Contains};
use geojson::FromGeoJson;

// First Input
struct Park {
  geometry: MultiPolygon,
  name: String
}
let parks: Vec<Park> = Park::from_geojson("parks.geojson");

// Second Input
struct Borough {
  geometry: MultiPolygon,
  name: String
}
let boroughs: Vec<Borough> = Borough::from_geojson("borough.geojson");

// Output
struct PartyVenue {
  park_geometry: MultiPolygon,
  park_name: String,
  borough_name: String,
}

let mut party_venues: Vec<PartyVenue> = Vec::new();

for park in parks {
  for borough in boroughs {
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

let first_venue = party_venues[0];
assert_eq!(first_venue.park_name, todo!());
assert_eq!(first_venue.borough_name, todo!());
```

## Water, Cooler

If Central Park is as mainstream as it gets, which open spaces are more like the avant-garde Vapor Wave Jazz that your hipster friends just can't get enough of these days? Let's augment our earlier code to filter out the mainstream (largest) parks, while still ensuring we'll be able to get a drink of New York's finest (water).

**Parks, with at least one water fountain, sorted by smallest area**

| park_name                    | borough   | fountains | area     |
|------------------------------|-----------|-----------|----------|
| Tiny Town                    | Queens    | 2         | 100      |
| Polly Pocket Park            | Brooklyn  | 3         | 200      |
| Honey I Shrunk the Esplanade | Bronx     | 1         | 300      |
| ...                          | ...       | ...       | ...      |

```rust
use geo::geometry::{MultiPolygon, MultiPoint, Point};
use geo::algorithm::{Area, Contains};
use geojson::FromGeoJson;

// First Input
struct Park {
  geometry: MultiPolygon,
  name: String
}
let parks: Vec<Park> = Park::from_geojson("parks.geojson");

// Second Input
struct Borough {
  geometry: MultiPolygon,
  name: String
}
let boroughs: Vec<Borough> = Borough::from_geojson("borough.geojson");

// Output
struct PartyVenue {
  park_name: String,
  borough_name: String,
  park_geometry: MultiPolygon,
  water_fountain_count: usize,
  area: f64,
}

let mut party_venues: Vec<PartyVenue> = Vec::new();

for park in parks {
  for borough in boroughs {
    if borough.geometry.contains(&park.geometry) {
      let venue = PartyVenue {
        park_name: park.name,
        borough_name: borough.name.clone(),
        area: park.geometry.unsigned_area(),
        park_geometry: park.geometry,
        water_fountain_count: 0, // we'll populate this field next
      };
      party_venues.push(venue);
    }
  }
}

let water_fountains = MultiPoint::from_geojson("water_fountains.geojson").unwrap();
for water_fountain in &water_fountains {
  for party_venue in &mut party_venues {
    if party_venue.park_geometry.contains(&water_fountain.geometry) {
      party_venue.water_fountain_count += 1;
      break;
    }
  }
}

party_venues = party_venues.into_iter().filter(|venue| venue.water_fountain_count > 0).collect();

party_venues.sort_by(|venue_1, venue_2| venue_1.area.partial_cmp(&venue_2.area).unwrap());

let tiniest_venue = party_venues[0];
assert_eq!(tiniest_venue.park_name, todo!());
assert_eq!(tiniest_venue.borough_name, todo!());
assert_eq!(tiniest_venue.water_fountain_count, todo!());
assert_eq!(tiniest_venue.area, todo!());

let largest_venue = party_venues[party_venues.len() -1];
assert_eq!(largest_venue.park_name, todo!());
assert_eq!(largest_venue.borough_name, todo!());
assert_eq!(largest_venue.water_fountain_count, todo!());
assert_eq!(largest_venue.area, todo!());
```

## Drowning in Data

This code works, but it's slower than necessary. So far we've been analyzing data sets without any consideration for speed. We can sort of justify our laissez-faire approach to CPU usage for one-off calculations that won't take too much time to run either way, but optimization will become increasingly important as we begin to work with larger amounts of data. In particular, the way we're checking if each water fountain is in each park is pretty naive. We can do better!

In the next section we'll explore some techniques to enhance the performance of some common geospatial operations. Quickly moving through the real world may require a jet engine, but that doesn't mean that your laptop needs to sound like one.
