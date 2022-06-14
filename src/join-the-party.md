# Join the Party

Think about the people who help fill the blank spaces in your life. The shape of each relationship is usually a little different, but it's almost always possible to find some common ground. Even if your friends (incorrectly) think that ["the dress"](https://en.wikipedia.org/wiki/The_dress) is white and gold, they still (correctly) don't like feeling thirsty.

## Drink from the Firehose

Imagine living in New York City. Good pizza, Broadway, and one of the country's best collections of [open data](https://data.ny.gov/) have never felt closer to home. We're looking for a spot to meet up with some friends. Let's start with a published list of every park in the city. Somewhere in these shaded shapes, we can see the faint outline of a future party beginning to form:

![The 2029 parks of New York City](images/nyc-parks.png)

Fortunately for us, we also have a list of every public water fountain that will let us sip some of NYC's ["world-renowned"](https://www1.nyc.gov/site/greenyc/take-action/drink-tap-water.page) tap water whenever the conversation starts to feel a little dry:

![The 3120 water fountains of New York City](images/nyc-water-fountains.png)

## Filtered Water

Right now we're dealing with two different data sources. Solving spatial problems often involves finding ways to stir things up. Let's see how many of these parks have water fountains:

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

todo!("There are probably more than 1000 parks with water fountains.")
assert_eq!(water_fountains_in_parks, 1000);
```

## Thirst for Knowledge

We simply skipped any parks that don't have drinking fountains in the example above, but oftentimes we will need to do something more complex like taking some attributes from one data source and combining those attributes with another data source based on their spatial relationship.

Let's look at some selected data from two data sources:

**The Five Boroughs**

| borough name | borough shape                    |
|--------------|----------------------------------|
| Brooklyn     | `MULTIPOLYGON((1.0 2.0,...)...)` |
| Queens       | `MULTIPOLYGON(((1.0 2.0))...)`   |
| ...          | ...                              |

**List of Parks**

| park name         | park shape                       |
|-------------------|---------------------------------|
| Prospect Park     | `MULTIPOLYGON((1.0 2.0,...)...)` |
| Gantra Plaza      | `MULTIPOLYGON((3.0 4.0,...)...)` |
| Forte Greene Park | `MULTIPOLYGON((3.0 4.0,...)...)` |
| ...               | ...                              |

Notice how the borough and park data sources include detailed shape information. The smaller shapes of each *park* can be positioned within the larger shapes that comprise each *borough*. Because our friends live in different areas, a list like this could really help us narrow down where we want to get together:

 - Option 1: Prospect Park in Brooklyn
 - Option 2: Gantry Plaza in Queens
 - Option 3: Forte Green Park in Brooklyn

In order to generate this list, we need to combine the park name from the first data source with the borough from the second data source that contains it.

If you've worked with SQL before, you might be thinking that this sounds a bit like a [JOIN clause](https://en.wikipedia.org/wiki/Join_(SQL)), and you'll no doubt be delighted to know that this kind of operation is indeed referred to as a *spatial join*. If you've never worked with SQL before, don't worry, you have an even bigger reason to be delighted.

```rust
use geo::types::{MultiPolygon, Point};
use geo::algorithms::{Contains};
use geojson::FromGeoJson;

// First Input
struct Park {
  geometry: MultiPolygon<f64>,
  name: String
}
let parks: Vec<Park> = Park::from_geojson("parks.geojson");

// Second Input
struct Borough {
  geometry: MultiPolygon<f64>,
  name: String
}
let boroughs: Vec<Borough> = Park::from_geojson("borough.geojson");

// Output
struct PartyVenue {
  park_geometry: MultiPolygon<f64>,
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
      }
      party_venues.push(venue);
    }
  }
  todo!("This park wasn't contained in a single borough")
}
```

## Water Cooler

We can build upon our new list a bit further by looking for small parks in each borough that are less likely to be crowded. If Central Park is as mainstream as it gets, which open spaces are more like the avant-garde Jazz that your hipster friends just can't get enough of these days? One theory is that busy parks would need to have more drinking fountains, so let's bring them back into the mix and find a place where it's possible to keep our heads above water in an ocean of people:

**Parks Sorted by Fewest Drinking Fountains**

| park name                | borough   | fountains |
|--------------------------|-----------|-----------|
| Unpopular Park           | Queens    | 1         |
| Dry Zone Memorial Grove  | Brooklyn  | 3         |
| Will you laugh again?    | Bronx     | 1         |
| ...                      | ...       |           |

## Drowning in Data

So far we've been analyzing these data sets without any consideration for speed. We can sort of justify our laissez-faire approach to CPU usage for one-off calculations that won't take too much time to run either way, but optimization will become increasingly important as we begin to work with larger amounts of data.

In the next section we'll explore some common techniques to enhance the performance of geospatial operations. Quickly moving through the real world may require a jet engine, but that doesn't mean that your laptop needs to sound like one.

---

## Draft Notes

- Space jam. Jam/jelly (Michael is sure there's a joke here)

Point in a polygon (point is contained).
Polygon within polygon, versus polygon that touches the side of another.
"If a property line crosses counties, should it be counted in both (or neither)."
Contains, intersects, "is within"
