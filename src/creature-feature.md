# Attribute Data

In the previous example we worked with WKT - a format for encoding geometric shapes like Points, Lines, and Polygons. It should come as no surprise that geospatial fundamentally involves geometry, but it's almost never *only* about the geometry. It's usually about tying some attribute data *to* that geometry. 

One common way to do this, is to embed WKT as one column in a CSV. Here's an excerpt from a CSV of New York City's public parks which includes the geometry of the park as well as some other attributes for each park in the city.

```csv

```

CSV is nice in some ways, because readers and writers are ubiquitous (any spreadsheet app!). However, to be blunt, in many other ways it is an old and crappy format. There's a couple things about the above data which, though might not be obvious at first, will complicate our analysis.

Let's ask some questions!
- Numeric vs string type isn't always clear, escaping/quotes
- No strong convention for how geometry is represented. It might contain:
  - two separate columns for a lon/lat point
  - a column containing some *other* serialization, like WKT (or geojson, etc).

<i>
TODO: Show example of working with CSV directly (e.g. through a hashmap style interface)

Original dataset includes acres, but maybe it'd be fun to compute that field ourselves in order to use a geometric algorithm.
</i>

<i>
TODO: Show the "civilized" way of doing the above, but with a typed struct rather than a Hashmap interface
</i>

## OMG GeoJSON

GeoJSON is a different format which does not have some of these short comings.

It's pretty popular, especially useful on the web, because it's, well JSON. Which web browsers understand natively.

GeoJSON has a built in way of expressing attributes *along side* the geometry.

Beyond web applications, many other geospatial tools can interoperate with geojson - e.g. qgis.

Downsides: It's geometry representation is quite verbose (not efficient for humans to read or store/transmit), it's not super readable. Spreadsheets are efficient for editing CSV's. It lacks a spatial index (future topic!), so certain geometric operations are slow compared to formats like shapefiles, gpx, and flatgeobuf.
