# Attribute Data

In the [previous example](./youre-projecting.html) we worked with geometry stored in Well Known Text (WKT) format. It should come as no surprise that solving geospatial problems fundamentally involves geometry, but it's rarely about *exclusively* geometry. Usually there is some other attribute data associated with that geometry that we want want to explore. This combination of a geometry with its associated attribute data is often referred to as a _feature_, and a group of features can be called a _feature collection_.

![Many spidering waterways consolidating into a major river](images/philly-waterways.png)

Like many cities, Philadelphia is fundamentally a city of water. As we saw in the last article, we can describe shapes using Well Known Text. We could represent all these waterways as a long list of WKT declarations:

- `MULTIPOLYGON(....)`
- `MULTIPOLYGON(....)`
- `MULTIPOLYGON(....)`
- ...

I grew up in a town with one river, so we'd say "we're going down to _the_ river."  In Philadelphia you have to be more specific. We need to associate some information with our shapes - giving us a dataset like this:

| creek_name        |  geometry          |
|-------------------|--------------------|
| Wissahickon Creek | MULTIPOLYGON(....) |
| Schuylkill River  | MULTIPOLYGON(....) |
| Delaware River    | MULTIPOLYGON(....) |

![Many rivers of Philadelphia again, but with labels this time - the Delaware on the eats, the Schuylkill from the North West](images/philly-waterways-labeled.png)

Now we're talking specifics! The Delaware River defines Philadelphia's eastern boundary, the Schuylkill River runs through the city from the North West. The Wissahickon is a favorite for local walkers, so let's amble over that way.

![Detail of the Wissahickon Creek](images/philly-wissahickon-detail.png)

Walking near water is neat and all, but walking over water — now that's an infrastructural thrill! So how do we find which segments of the Wissahickon have a bridge? Combining a geometry with other associated data into a _feature_ allows us to solve these kinds of problems.

There are a lot of ways of storing geospatial information.  Recall that WKT is only concerned with shape — it can't store whether that geometry represents a bridge or has a name. One approach then, is to embed WKT into *another* more flexible format. Commonly, a CSV file (comma separated value) will have one column containing the WKT to describe the shape of the feature and will use additional columns for the attribute data - i.e. the name of the waterway and whether or not the segment is bridged.

![Zoomed in segment of a bending waterway, with one narrow segment highlighted](images/philly-bridge-selected.png)

This [CSV of Philadelphia water way segments](philly_waterways.csv) does just that. Here's an excerpt:

| creek_name        | inf1    | geometry           |
|-------------------|---------|--------------------|
| Cobbs Creek       |         | MULTIPOLYGON(....) |
| Cobbs Creek       | Bridged | MULTIPOLYGON(....) |
| Wise's Mill       |         | MULTIPOLYGON(....) |
| Wissahickon Creek | Bridged | MULTIPOLYGON(....) |
| Wissahickon Creek |         | MULTIPOLYGON(....) |
| Wissahickon Creek | Bridged | MULTIPOLYGON(....) |

Note that in that in this data set, a single creek is broken into many small segments. A "Bridged" segment indicates precisely where on the waterway a bridge exists - highlighted in the image above.

For my Wissahickon Walkabout, I want to know how many bridges I'll encounter. For posterity, I also want to know where the largest bridge I'll encounter is, so I can snap a photo when I get there.

```rust
use csv;
use geo::algorithm::Area;
use geo::geometry::MultiPolygon;
use wkt;

let mut csv_reader = {
  use std::fs::File;
  let file = File::open("philly_waterways.csv").expect("invalid file");
  csv::Reader::from_reader(file)
};

let mut max_bridge_area = None;
let mut max_bridge_location = None;
let mut bridge_count = 0;

for row in csv_reader.records() {
  let creek_segment = row.expect("unable to read row from CSV");

  let creek_name = waterway_segment.get(0).expect("missing 'creek_name' field");

  if creek_name != "Wissahickon Creek" {
    continue;
  }

  let infrastructure_label = waterway_segment.get(1).expect("missing 'inf1' field");

  if infrastructure_label != "Bridged" {
    continue;
  }
  bridge_count += 1;

  let geometry_str = waterway_segment.get(2).expect("missing `geometry` field");
  use wkt::TryFromWkt;
  let geometry = MultiPolygon::try_from_wkt_str(geometry_str).expect("invalid wkt");

  let bridge_area = geometry.unsigned_area();

  if let Some(ref mut previous_max) = max_bridge_area {
      if bridge_area > previous_max {
        *previous_max = bridge_area;
        max_bridge_location = Some(geometry.bounding_rect().center());
      }
  } else {
    // This is the first bridge - so by definition it's the
    // biggest one we've seen so far.
  max_bridge_area = Some(bridge_area);
  max_bridge_location = Some(geometry.bounding_rect().center());
  }
}

assert_eq!(max_bridge_area, Some(todo!()));
assert_eq!(max_bridge_location, Some(todo!()));
```

OK that works. But we can tidy things up a bit. One thing you may have noticed is the repetitive nature of `get`ting the fields and `expect`ing no errors:

```rust,ignore
let creek_name = waterway_segment.get(0).expect("missing 'creek_name' field");
let infrastructure_label = waterway_segment.get(1).expect("missing 'inf1' field");
let geometry = waterway_segment.get(2).expect("missing `geometry` field");
```

For each row in the CSV, getting fields by number in an ad-hoc fashion, is simple, but a bit loosey goosey, and requires us to remember the field ordering. It also requires some rote error checking boiler plate.

If we want to add a little more *struct*ure to our world, we can parse each row into a rigidly defined `struct`. Recall our schema:

| creek_name        | inf1    | geometry           |
|-------------------|---------|--------------------|
| Wissahickon Creek | Bridged | MULTIPOLYGON(....) |

We could describe this schema as a Rust struct like this:

```rust
struct CreekSegment {
  creek_name: String,
  inf1: String,
  geometry: geo::geometry::MultiPolygon
}
```

Notice how each field of the `CreekSegment` struct corresponds to a column in our CSV input. We could then write code to populate these fields, but rather than write this code from scratch, we turn to the wisdom of those who've deserialized before.

### Serde, slayer of boilerplate
The excellent [`serde`](https://serde.rs) crate is a framework for serializing and deserializing data across a lot of different formats, and gives us a concise way to annotate the above struct declaration in order to automatically populate the struct with data read from our CSV.

```rust
#[derive(serde::Deserialize)]
struct CreekSegment {
  creek_name: String,

  // serde offers some customiziations so that we can use sensible
  // names in our code without having to modify our source data, whose
  // names we might not control
  #[serde(rename = "inf1" )]
  infrastructure_label: String,

  // serde has built-in support for common data types like numbers and strings,
  // and it also allows other crates (like `wkt`) to build custom deserializers
  // so that we can create complext data types (like this `MultiPolygon`)
  // directly from our input data.
  #[serde(deserialize_with = "wkt::deserialize_wkt")]
  geometry: geo::geometry::MultiPolygon
}
```

Finally, before we return to our example, a struct like this is also the perfect place to hang some little helper methods:

```rust
# #[derive(serde::Deserialize)]
# struct CreekSegment {
#   creek_name: String,
#
#
#   #[serde(rename = "inf1" )]
#   infrastructure_label: String,
#
#   #[serde(deserialize_with = "wkt::deserialize_wkt")]
#   geometry: geo::geometry::MultiPolygon
# }
#
impl CreekSegment {
  fn is_bridge(&self) -> bool {
    self.infrastructure_label == "Bridged"
  }

  fn area(&self) -> f64 {
    use geo::algorithm::Area;
    self.geometry.unsigned_area()
  }

  fn centroid(&self) -> geo::Point {
    use geo::algorithm::BoundingRect;
    self.geometry.boudning_rect().center()
  }
}
```

Let's see how we can use the above code to clean up our earlier implementation:
```rust
# use csv;
# use geo::algorithm::Area;
# use geo::geometry::MultiPolygon;
# use wkt;
#
# let mut csv_reader = {
#   use std::fs::File;
#   let file = File::open("philly_waterways.csv").expect("invalid file");
#   csv::Reader::from_reader(file)
# };
#
# let mut max_bridge_area = None;
# let mut max_bridge_location = None;
# let mut bridge_count = 0;
# #[derive(serde::Deserialize)]
# struct CreekSegment {
#   creek_name: String,
#
#
#   #[serde(rename = "inf1" )]
#   infrastructure_label: String,
#
#   #[serde(deserialize_with = "wkt::deserialize_wkt")]
#   geometry: geo::geometry::MultiPolygon
# }
#
# impl CreekSegment {
#   fn is_bridge(&self) -> bool {
#     self.infrastructure_label == "Bridged"
#   }
#
#   fn area(&self) -> f64 {
#     use geo::algorithm::Area;
#     self.geometry.unsigned_area()
#   }
#
#   fn centroid(&self) -> geo::Point {
#     use geo::algorithm::BoundingRect;
#     self.geometry.boudning_rect().center()
#   }
# }
#
for row in csv_reader::records<CreekSegment>() {

  // All of our error checking and field parsing is replaced by this
  // line, and inferred from our serde-annotated struct declaration.
  let creek_segment: CreekSegment = row.expect("invalid creek segment");

  // At this point we know all the fields of creek_segment
  // have been populated.

  if creek_segment.creek_name != "Wissahickon Creek" {
    continue;
  }

  if !creek_segment.is_bridge() {
    continue;
  }
  bridge_count += 1;

  let bridge_area = creek_segment.area();

  if let Some(ref mut previous_max) = max_bridge_area {
      if bridge_area > previous_max {
        *previous_max = bridge_area
        max_bridge_location =  Some(creek_segment.center());
      }
  } else {
    // This is the first bridge - so by definition it's the
    // biggest one we've seen so far.
    max_bridge_area = Some(bridge_area);
    max_bridge_location = Some(creek_segment.center());
  }
}
#
# assert_eq!(bridge_count, todo!());
# assert_eq!(max_bridge_area, Some(todo!()));
# assert_eq!(max_bridge_location, Some(todo!()));
```

Using serde and structs like this can help keep your code tidy, especially as programs get more complex, but doing so is completely optional. If you prefer the ad-hoc style of the original example, accessing fields by number, and you don't care about adding cute little helper methods, that's totally fine. Skip it and stick with the earlier approach.

## CSV C U L8R 

CSV is a nice format in some ways. For one, tons of programs can read and write CSV — for example any spreadsheet app! In many other ways however, it is an old and crappy format, which lacks in some not always obvious ways.

For example, if someone sends you a CSV, how does it represent its geometry? There is no strong convention for what geometry columns in a CSV are named or where they are positioned. As for formats, though WKT is common, it's far from universal. For example, if it is a CSV of points, sometimes you'll have a `latitude` and `longitude` column instead of a WKT column.

Another problem with CSV's is that it's not always clear what type is in a column. For example, consider this data:

| phone                 | description |
|-----------------------|-------------|
| 311                   | info        |
| 911                   | emergency   |
| ...                   | ...         |
| 1-818-912-8200 ext. 4 | office      |

Unless you've examined the entire list, you might not realize that `phone` should be a text column, not a numeric one. Some formats are always clear about the distinction between numbers and text, but CSV isn't necessarily.

This lack of standards means whenever you encounter geographic data stored as CSV you have to dig around a bit to orient yourself to the author's conventions. 

## OMG GeoJSON
GeoJSON is another format for representing geospatial features (geometry + data) with a different set of tradeoffs. Just to give you an example, here's what our previous data would look like as geojson:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [0, 1, 2, ...]
      }
      "properties": {
        "name": "Wissahickon Creek"
        "inf1": "Bridged"
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [4, 5, 6, ...]
      }
      "properties": {
        "name": "Wissahickon Creek"
        "inf1": ""
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [9, 10, 11, ...]
      }
      "properties": {
        "name": "Cobbs Creek"
        "inf1": ""
      }
    }
  ]
}
```

GeoJSON is pretty popular, especially for mapping and other geospatial applications on the web. This is largely because it's, well, JSON, which is used by web browsers natively for all kinds of information. Consequently, it's easy for web programmers to manipulate GeoJSON using JavaScript in the browser.

GeoJSON has long since left the domain of "web only", and now many other geospatial tools know how to read and write GeoJSON. Tools like [QGIS](https://qgis.org) and language tools like GEOS, JTS, GDAL, and Shapely all know how to speak GeoJSON.

```rust
use geojson;
use geo::algorithm::Area;
use geo::geometry::MultiPolygon;
use wkt;

let mut geojson_feature_reader = {
  use std::fs::File;
  let file = File::open("philly_waterways.geojson").expect("invalid file");
  geojson::FeatureCollectionReader::from(file)
};

# let mut max_bridge_area = None;
# let mut max_bridge_location = None;
# let mut bridge_count = 0;
# #[derive(serde::Deserialize)]
# struct CreekSegment {
#   creek_name: String,
#
#
#   #[serde(rename = "inf1" )]
#   infrastructure_label: String,
#
#   #[serde(deserialize_with = "wkt::deserialize_wkt")]
#   geometry: geo::geometry::MultiPolygon
# }
#
# impl CreekSegment {
#   fn is_bridge(&self) -> bool {
#     self.infrastructure_label == "Bridged"
#   }
#
#   fn area(&self) -> f64 {
#     use geo::algorithm::Area;
#     self.geometry.unsigned_area()
#   }
#
#   fn centroid(&self) -> geo::Point {
#     use geo::algorithm::BoundingRect;
#     self.geometry.boudning_rect().center()
#   }
# }
#
for feature in geojson_feature_reader {

  // All of our error checking and field parsing is replaced by this
  // line, and inferred from our serde-annotated struct declaration.
  let creek_segment: CreekSegment = feature.expect("invalid creek segment");

  // Thanks to the magic of serde, the rest of this example is exactly
  // the same as the serde CSV example.

#  // At this point we know all the fields of creek_segment
#  // have been populated.
#
#   if creek_segment.creek_name != "Wissahickon Creek" {
#     continue;
#   }
#
#   if !creek_segment.is_bridge() {
#     continue;
#   }
#   bridge_count += 1;
#
#   let bridge_area = creek_segment.area();
#
#   if let Some(ref mut previous_max) = max_bridge_area {
#       if bridge_area > previous_max {
#         *previous_max = bridge_area
#         max_bridge_location =  Some(creek_segment.center());
#       }
#   } else {
#     // This is the first bridge - so by definition it's the
#     // biggest one we've seen so far.
#     max_bridge_area = Some(bridge_area);
#     max_bridge_location = Some(creek_segment.center());
#   }
# }
#
# assert_eq!(bridge_count, todo!());
# assert_eq!(max_bridge_area, Some(todo!()));
# assert_eq!(max_bridge_location, Some(todo!()));
```

If its ubiquity is its biggest upside, GeoJSON does have some downsides. Its geometry representation is quite verbose. Unlike WKT it's not as easy for humans to read at a glance, and compared to some other formats, it's not very efficient for computers to store or transmit. JSON editors exist, but they aren't nearly as powerful or ubiquitous as spreadsheet programs for CSVs. GeoJSON also lacks a _spatial index_ (future topic!), so certain operations on complex geometries are slow.

There's an entire world of alternative formats — all with their own trade-offs. Luckily, Rust has support for pretty much all of them at this point. Other than [WKT](https://docs.rs/wkt) and [GeoJSON](https://docs.rs/geojson), some other popular choices include:

* [Shapefiles (.shp)](https://docs.rs/shapefile) - venerable (and often maligned) all-purpose format.
* [Geopackage (.gpx)](https://docs.rs/geozero) - the "preferred" format for lots of desktop GIS applications these days, built on top of sqlite.
* [Flatgeobuf (.fgb)](https://docs.rs/flatgeobuf) - a newer format, well suited for efficient random read-only access.

## Up Next

Combining attributes across multiple data sources using spatial joins.

### Draft Notes

"The City Of Philadelphia" - The City Of The City Of Brotherly Love

The most populous conterminous city/county (Phila. city and Phila. county cover the same area, but have separate governments entities).

Sink holes: I cut them out of the draft since I couldn't really fit them into the narrative, but would be happy to have them fit in somehow because they are neat.
Asphalt truck on way to fix streets falls into sinkhole: https://twitter.com/orentalks/status/1070372166867320832 (though this was not in a neighborhood near mill creek, it was allegedly in fish town)

Primer on Philadelphia sink holes:
https://www.phillymag.com/news/2019/08/02/sinkhole-philadelphia/
