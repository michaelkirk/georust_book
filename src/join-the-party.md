# Join the Party

Solving spatial problems often involves combining data from multiple sources. Often we want to relate one spatial data source to another.

As a concrete, albeit simple example, let's say I have a list of all the parks in New York City, including their shapes. I also have a list of public water fountain locations across the city. Let's say I want to know how many of NYC's water fountains can be found within its parks.

```rust
use geo::{GeometryCollection, MultiPoint};
use geojson::FromGeoJson;

let parks = GeometryCollection::from_geojson_str(&parks_geojson).unwrap();
let water_fountains = MultiPoint::from_geojson_str(&water_fountains_geojson).unwrap();

let mut water_fountains_in_parks = 0;
for water_fountain in &water_fountains {
  if parks.contains(&water_fountain) {
    water_fountains_in_parks += 1;
  }
}

assert_eq!(water_fountains_in_parks, 1000);
```

In the above example, we simply counted the points within the polygon. Often you'll want to do something more complex - commonly you'll want to take some attributes from one data source and combine it with those from another data source, based on their spatial relation.

As a further example, let's say I want to prepare an interactive web map showing a pin where each water fountain is. When a user clicks on the pin they should see some details about the water fountain, including it's latitude/longitude and the _name_ of the park that it is inside of, if any.

To be a little more specific, imagine something like these two data sources:

**Water Fountains**

| water fountain location |
|-------------------------|
| `POINT(1.0 2.0)`        |
| `POINT(2.0 3.0)`        |
| `POINT(4.0 4.0)`        |
| `POINT(9.0 9.0)`        |

**Parks**

| park shape                       | park name     |
|----------------------------------|---------------|
| `MULTIPOLYGON((1.0 2.0,...)...)` | Joe's Park    |
| `MULTIPOLYGON((3.0 4.0,...)...)` | Memorial Park |

Ultimately, we want to combine these two data sources in order to create something like:

| water fountain location | park name            |
|-------------------------|----------------------|
| `POINT(1.0 2.0)`        | Joe's Park           |
| `POINT(2.0 3.0)`        | Joe's Park           |
| `POINT(4.0 4.0)`        | Memorial Park        |
| `POINT(4.0 4.0)`        | <blank>              |

If you've worked with SQL before, you're probably thinking this looks an awful lot like a [JOIN clause](https://en.wikipedia.org/wiki/Join_(SQL)). And you'll no doubt be delighted to know that this operation is referred to as a *spatial* join. If you've never worked with SQL before, I'm sure you'll be delighted to know that it doesn't really matter what it's called and that the SQL users probably weren't really all that excited about the preceding revelation anyway.

Here's what that'd look like:
```rust
use geo::types::{MultiPolygon, Point};
use geo::algorithms::{Contains};
use geojson::FromGeoJson;

struct Park {
  geometry: MultiPolygon<f64>,
  name: String
}

// let parks = GeometryCollection::from_geojson_str(&parks_geojson).unwrap();
let parks: Vec<Park> = todo!("parse geojson into Park struct, maybe using the FromGeoJson trait");

struct WaterFountain {
  location: Point<f64>,
  park_name: Option<String>,
}

let water_fountain_locations = MultiPoint::from_geojson_str(&water_fountains_geojson).unwrap();

let mut water_fountains_in_parks = 0;
let water_fountains = water_fountain_locations.iter().map(|water_fountain_location| {
  for park in parks {
    if park.geometry.contains(&water_fountain_location) {
      return WaterFountain {
        location: water_fountain_location,
        park_name: Some(park.name.clone())
      }
    }
  }
  // None of the parks contained this water fountain
  return WaterFountain { location: water_fountain_location, park_name: None }
}).collect();

let output = water_fountains.to_geojson_str();
assert_eq!(output[0..50], r#"{"type": "FeatureCollection", "features":""#);
```
<i>
draft note 1: I'm not sure if this example is worthwhile - it's somewhat redundant with the forthcoming party example. There's nothing fundamentally new about the party example which will follow - it'll just be a more complicated (convoluted?) example. But maybe that is a good way to reinforce after the relatively "nice and neat" example here.
</i>
<i>
draft note 2: it's kind of a lot of code. We might want to break it up better like we did with the previous walk through.
</i>

## My problem

Ok, enough with the motivating context, and onto the problem at hand. I need your help. I'm visiting New York for a little bit, and I'm trying to throw a party for my friends here. I'm going to invite 3 friends (it's a small party), but each friend has their own favorite activity. Bobi likes basketball, Horia likes horses, while Sam prefers swimming. My own favorite past time is geo spatial analysis, so I'm set, but my friends could use a little help.

New York City has a lot of parks. They all have different amenities. Could one of them satisfy these diverse needs? The city has published some data on which ameneties are available where, but it's not organized in such a way that lets you answer this kind of question directly.

Luckily, the tools we need to piece together the perfect party are at hand.

---

## Draft Notes

- Lots of space.
- City that never sleeps; party never ends.
- Friends with different interests.
- Dunk vs. dry (drinking fountains)
- Real horses vs. fake horses.
- Space jam. Jam/jelly (Michael is sure there's a joke here)
- Concessions
- Thirst trap
- NYC public data publishing practices. Possibly the best in the country.
- Dunk on friends (make them take the L train before taking the L)
- Sequence: Basketball (hot), pool (cool), snacks, drinks
- What's within your heart? Different social circles, overlapping?
- Broadway shows.
- Get away from Times Square.
- Can't show off your drip in a place without water.

Point in a polygon (point is contained).
Polygon within polygon, versus polygon that touches the side of another.
"If a property line crosses counties, should it be counted in both (or neither)."
Contains, intersects, "is within"

Data sets:
- Parks
- Basketball
- Concessions
- Drinking fountains

Frank summary of what we're going to talk about (e.g. this is what a spatial join *is*).

Borough column in the Parks dataset is a spatial join, already in the data set. How would we get that if it wasn't already there?

This article introduces at least three new concepts:
1. geojson as a format
2. **attribute** data (the previous example uses *only* geometry, not attributes like "name" or "borough"
3. parsing into structs
4. spatial joining

Arguably the first three concepts might benefit from being separated into a preceding chapter. It should be probably be called https://en.wikipedia.org/wiki/Creature_Features or maybe something like "OMG GEOJSON"
