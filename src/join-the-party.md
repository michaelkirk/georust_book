# Join the Party

Imagine living in New York City. Good pizza, Broadway, and one of the country's best collections of [open data](https://data.ny.gov/) have never felt closer to home. What really makes a city though, are the people who live there. Think about the people who help fill the blank spaces in your life. The shape of each relationship is usually a little different, but it's almost always possible to find some common ground. In this New York City life of our imagination, let's imagine some of the [Friends](https://en.wikipedia.org/wiki/Friends) we'd like to spend our time with, and start to think about where we could all hang out.

![Scene from Friends, with everyone inside the crowded Central Perk cafe](images/nyc-friends.jpg)
<span class="photo-credit">Courtesy NBC</span>

## Drink from the Firehose

That cafe looks way too crowded. Instead, let's embrace the great outdoors and start putting together a list of parks in the city. Somewhere in these shaded shapes, we can see the faint outline of a future party beginning to form:

![The 2029 parks of New York City](images/nyc-parks.png)

Fortunately for us, we also have a list of every public drinking fountain that will let us sip some of NYC's ["world-renowned"](https://www1.nyc.gov/site/greenyc/take-action/drink-tap-water.page) tap water whenever the conversation starts to feel a little dry:

![The 3120 drinking fountains of New York City](images/nyc-water-fountains.png)

## Filtered Water

We have these two different data sources — drinking fountains and parks. Solving spatial problems often involves finding ways to stir things up. Let's see how many of these parks contain at least one of these drinking fountains.

```rust
use geojson::de::deserialize_feature_collection;
use geo::algorithm::Contains;

fn reader(filename: &str) -> std::io::BufReader<std::fs::File> {
  let path = format!("src/data/nyc/{filename}");
  std::io::BufReader::new(std::fs::File::open(path).expect("parks file path must be valid"))
}

let parks: Vec<geo::MultiPolygon> = {
  deserialize_feature_collection(reader("parks.geojson"))
    .expect("valid FeatureCollection")
    .map(|park_result: geojson::Result<geojson::Feature>| {
      let park_feature = park_result.expect("valid feature");
      geo::MultiPolygon::try_from(park_feature).expect("valid conversion")
    })
    .collect()
};

let drinking_fountain_points: Vec<geo::Point> = deserialize_feature_collection(reader("drinking_fountains.geojson"))
    .expect("is a FeatureCollection")
    .map(|feature_result: geojson::Result<geojson::Feature>| {
      let feature = feature_result.expect("valid Feature");
      geo::Point::try_from(feature).expect("valid conversion")
    }).collect();

let mut parks_with_drinking_fountains = 0;
for park in &parks {
  for drinking_dountain_point in &drinking_fountain_points {
    if park.contains(drinking_dountain_point) {
      parks_with_drinking_fountains += 1;
      break;
    }
  }
}

assert_eq!(parks_with_drinking_fountains, 902);

let parks_without_drinking_fountains = parks.len() - parks_with_drinking_fountains;
assert_eq!(parks_without_drinking_fountains, 1127);
```

A little less than half of NYC's parks have drinking fountains. Good to know!

## Thirst for Knowledge

Counting is simple, but oftentimes we will need to do something more complex like taking certain attributes from one data source and combining those attributes with another data source based on their spatial relationship.

Let's look at some selected data from two data sources:

**List of parks (same as above)**

| park_name            | park_shape                       |
|----------------------|----------------------------------|
| Gilbert Ramirez Park | `MULTIPOLYGON(((-73.93418260...` |
| Spargo Park          | `MULTIPOLYGON(((-73.89721950...` |
| Turtle Playground    | `MULTIPOLYGON(((-73.82693385...` |
| ...                  | ...                              |

**The five boroughs**

| borough_name | borough_shape                  |
|--------------|--------------------------------|
| Brooklyn     | `MULTIPOLYGON(((-73.86327...)` |
| Queens       | `MULTIPOLYGON(((-73.82644...)` |
| ...          | ...                            |

It's truncated in the tables above, but the park and borough data sources include detailed shape information. The smaller shapes of each *park* can be positioned within the larger shapes that comprise each *borough*. Because our friends live in different areas, a list like this could really help us narrow down where we want to get together:

> - Option 1: Gilbert Ramirez Park in Brooklyn
> - Option 2: Spargo Park in Queens
> - Option 3: Turtle Playground in Queens
> - ...

In order to produce a list like this, we need to combine the park name from the first data source with the borough that contains it from the second data source.

If you've worked with SQL before, you might be thinking that this sounds a bit like a [JOIN clause](https://en.wikipedia.org/wiki/Join_(SQL)), and you'll no doubt be delighted to know that this kind of operation is indeed referred to as a *spatial join*. If you've never worked with SQL before, don't worry, you have an even bigger reason to be delighted.

Let's start to build a list of venues by combining each park with the name of the borough the park is in.

```rust
use geo::geometry::{MultiPolygon, Point};
use geo::algorithm::Contains;

use geojson::de::{deserialize_feature_collection_to_vec, deserialize_geometry};

# fn reader(filename: &str) -> std::io::BufReader<std::fs::File> {
#   let path = format!("src/data/nyc/{filename}");
#   std::io::BufReader::new(std::fs::File::open(path).expect("parks file path must be valid"))
# }

// Using what we learned in the previous lesson, we'll
// deserialize the input GeoJSON into a structs.

// First Input
#[derive(serde::Deserialize)]
struct Park {
  #[serde(deserialize_with="deserialize_geometry")]
  geometry: geo::MultiPolygon,

  #[serde(rename="signname")]
  name: String
}

let parks = deserialize_feature_collection_to_vec::<_, Park>(reader("parks.geojson")).expect("valid geojson");

// Second Input
#[derive(serde::Deserialize)]
struct Borough {
  #[serde(deserialize_with="deserialize_geometry")]
  geometry: geo::MultiPolygon,

  #[serde(rename="boro_name")]
  name: String
}

let boroughs = deserialize_feature_collection_to_vec::<_, Borough>(reader("boroughs.geojson")).expect("valid geojson");

// Output
struct PartyVenue {
  park_geometry: MultiPolygon,
  park_name: String,
  borough_name: String,
}

let mut venues: Vec<PartyVenue> = parks.into_iter().map(|park| {
  for borough in &boroughs {
    if borough.geometry.intersects(&park.geometry) {
      return PartyVenue {
        park_name: park.name.clone(),
        park_geometry: park.geometry.clone(),
        borough_name: borough.name.clone(),
      };
    }
  }
  panic!("park did not intersect with any borough");
}).collect();

let first_venue = &venues[0];
assert_eq!(first_venue.park_name, "Devoe Park");
assert_eq!(first_venue.borough_name, "Bronx");
```

We've produced a list of parks, spatially joined to their borough name. But there are still so many options. How can we narrow it down?

## Water, Cooler

If [Central Park](https://www.nycgovparks.org/parks/central-park) is as mainstream as it gets, which open spaces are more like the avant-garde Vapor Wave Jazz that your hipster friends just can't get enough of these days? Let's augment our earlier code to filter out the mainstream (largest) parks, while still ensuring we'll be able to get a drink of New York's finest (water).

We want to produce something like this:

| park                         | borough   | fountains | square_meters |
|------------------------------|-----------|-----------|---------------|
| Tiny Town                    | Queens    | 2         | 100           |
| Polly Pocket Park            | Brooklyn  | 3         | 200           |
| Honey I Shrunk the Esplanade | Bronx     | 1         | 300           |
| ...                          | ...       | ...       | ...           |

```rust
# use geo::geometry::{MultiPolygon, MultiPoint, Point};
# use geo::algorithm::{Area, Contains, Intersects};
#
# use geojson::de::{deserialize_feature_collection_to_vec, deserialize_feature_collection, deserialize_geometry};
#
# fn reader(filename: &str) -> std::io::BufReader<std::fs::File> {
#   let path = format!("src/data/nyc/{filename}");
#   std::io::BufReader::new(std::fs::File::open(path).expect("file path must be valid"))
# }
#
# // First Input
# #[derive(serde::Deserialize)]
# struct Park {
#   #[serde(deserialize_with="deserialize_geometry")]
#   geometry: geo::MultiPolygon,
#
#   #[serde(rename="signname")]
#   name: String
# }
# let parks = deserialize_feature_collection_to_vec::<_, Park>(reader("parks.geojson")).expect("valid geojson");
# 
# // Second Input
# #[derive(serde::Deserialize)]
# struct Borough {
#   #[serde(deserialize_with="deserialize_geometry")]
#   geometry: geo::MultiPolygon,
#
#   #[serde(rename="boro_name")]
#   name: String
# }
# let boroughs = deserialize_feature_collection_to_vec::<_, Borough>(reader("boroughs.geojson")).expect("valid geojson");
#
// Output
struct PartyVenue {
  park: String,
  borough: String,
  geometry: MultiPolygon,
  square_meters: f64,
  fountains: usize,
}

let mut venues: Vec<PartyVenue> = parks.map(|park| {
  for borough in &boroughs {
    if borough.geometry.intersects(&park.geometry) {
      use proj::Transform;
      // Project the geometry in order to calculate a useful area.
      // See our first lesson for more on projections.
      //
      // EPSG:32115 - New York Eastern (meters)
      let square_meters = park.geometry
        .transformed_crs_to_crs("WGS84", "EPSG:32115")
        .expect("valid projection")
        .unsigned_area();

      return PartyVenue {
        park: park.name.clone(),
        borough: borough.name.clone(),
        geometry: park.geometry.clone(),
        square_meters,
        fountains: 0, // we'll populate this field later
      };
      break;
    }
  }
}).collect();

let fountains: Vec<geo::Point> = deserialize_feature_collection(reader("drinking_fountains.geojson"))
    .expect("is a FeatureCollection")
    .map(|feature_result: geojson::Result<geojson::Feature>| {
      let feature = feature_result.expect("valid Feature");
      geo::Point::try_from(feature).expect("valid conversion")
    }).collect();

for fountain in &fountains {
  for venue in &mut venues {
    if venue.geometry.contains(fountain) {
      venue.fountains += 1;
      break;
    }
  }
}

// We're only interested in parks that have at least one water fountain
venues = venues.into_iter().filter(|venue| venue.fountains > 0).collect();

// Sort by the size of the park
venues.sort_by(|venue_1, venue_2| venue_1.square_meters.partial_cmp(&venue_2.square_meters).expect("valid floating point comparison"));

let tiniest_venue = venues.first().unwrap();
assert_eq!(tiniest_venue.park, "Glendale Veterans Triangle");
assert_eq!(tiniest_venue.borough, "Queens");
approx::assert_relative_eq!(tiniest_venue.square_meters, 26.26352507495168);
assert_eq!(tiniest_venue.fountains, 1);

let largest_venue = venues.last().unwrap();
assert_eq!(largest_venue.park, "Freshkills Park");
assert_eq!(largest_venue.borough, "Staten Island");
approx::assert_relative_eq!(largest_venue.square_meters, 3724056.9377815602);
assert_eq!(largest_venue.fountains, 3);
```

We have found some excellent candidates for a party. These off the beaten path spots will let us spend hours of quality time with our friends, all while staying hydrated. What more could you possibly ask for on a lazy day in New York City?

## Drowning in Data

One thing you could definitely ask for are quicker results. The code above works, but *my goodness* is it slow! Until now we've been brute forcing our solutions without much consideration for speed. We can sort of justify our laissez-faire approach to CPU usage for one-off calculations like this, but optimization will become increasingly important as we begin to work with larger amounts of data, longer processing pipelines, or if we need to recompute results frequently.

Generally speaking, Rust "is fast", but that's not the whole story. Let's say you're trying to get to [Harlem](https://en.wikipedia.org/wiki/Sugar_Hill,_Manhattan) in a hurry. If you're lucky, you could have been born [Tatyana McFadden](https://en.wikipedia.org/wiki/Tatyana_McFadden), five time winner and course record holder for the New York City Marathon women's weelchair division. Alternatively, you could just recall Duke Ellington and [take the A train](https://www.youtube.com/watch?v=1ggcQk67Mco&t=86s). In the same way, Rust's speedy physique, paired with a little strategy on how you approach your particular problem, will get you where you're going in record time.

In the next section we'll explore some of these tricks of the trade to speed up common geospatial operations. Quickly moving through the real world may require a jet engine, but that doesn't mean that your laptop needs to sound like one.
