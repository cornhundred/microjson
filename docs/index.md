# MicroJSON Specification

## Introduction

MicroJSON is a format, inspired by [GeoJSON](https://geojson.org), for encoding a variety of data structures related to microscopy images, including reference points, regions of interest, and other annotations. These data structures are represented using the widely adopted JSON format, making it easy to work with in various programming languages and applications. It is fully backwards compatible with the [GeoJSON Specification, RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946), since any GeoJSON also is accepted as a MicroJSON. As GeoJSON supports foreign top level properties, a MicroJSON is also a valid GeoJSON. This specification describes briefly the objects that exist in GeoJSON, and then in more detail describes the additional objects that are part of MicroJSON. For a more detailed description of the GeoJSON objects, please see the [GeoJSON Specification, RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946).

## Objects

### MicroJSON Object

A MicroJSON object is a JSON object that represents a geometry, feature, or collection of features, or more precisely, be either of type (having value of top level field `type` as) `"Geometry"`, `"Feature"`, or `"Featurecollection"`, that is, the same as for GeoJSON.

A MicroJSON object may have a `"bbox"` property":

- `"bbox"`: (Optional) Bounding Box of the feature represented as an array of length 4 (2D) or length 6 (3D).

### Geometry Object

A geometry object is a JSON object where the `type` member's value is one of the following strings: `"Point"`, `"MultiPoint"`, `"LineString"`, `"MultiLineString"`, `"Polygon"`, `"Rectangle"`, `"MultiPolygon"`, or `"GeometryCollection"`.

Each geometry object MUST have a `"coordinates"` member with an array value. The structure of the coordinates array varies with the geometry type.  The innermost point coordinates array MUST contain two or three (if 3D) numbers representing the X and Y (and Z) coordinates of the point in the image. These coordinates follow the same order as the axes in [Multiscale object](#multiscale-object). Please note that these coordinates differ from the GeoJSON specification, where the order is longitude, latitude, and optionally altitude. If no multiscale object is defined, the default coordinate system is assumed to be the same as the image coordinate system, using cartesian coordinates and pixels as units, with the origin at the top left corner of the image, and the x-axis pointing to the right and the y-axis pointing down. The z-axis points into the image, with the origin at the top left corner of the image.

- **Point**:  Must be a single set of point coordinates. A “Point” Geometry may have a radius, if representing a circular object, with the value in pixels, specified as a member `“radius”` of the Geometry object.
- **MultiPoint**: The coordinates array must be an array of point coordinates.
- **LineString**: The coordinates array must be an array of two or more point coordinates forming a continuous line. A “LineString” Geometry may have a radius, with the value in pixels, specified as a member “radius” of the Geometry object.
- **MultiLineString**: The coordinates array must be an array of LineString coordinate arrays.
- **Polygon**: The coordinates array must be an array of linear ring point coordinate arrays, where the first linear ring represents the outer boundary and any additional rings represent holes within the polygon.
    - A subtype of “Polygon” is the “Rectangle” geometry: A polygon with an array of four 2D point coordinates representing the corners of the rectangle in a counterclockwise order. It has the property subtype with the value `“Rectangle”`.
- **MultiPolygon**: The coordinates array must be an array of Polygon coordinate arrays.

### GeometryCollection

A GeometryCollection is an array of geometries (Point, multipoint, LinesString, MultiLineString, Polygon, MultiPolygon). It is possible for this array to be empty.

### Feature Object

A feature object represents a spatially bounded entity associated with properties specific to that entity. A feature object is a JSON object with the following members, required unless otherwise noted:

- `"type"`: A string with the value `"Feature"`.
- `"geometry"`: A geometry object as defined in the section above or a JSON null value.
- `"properties"`: (Optional) A JSON object containing properties and metadata specific to the feature, or a JSON null value. It consists of key-value pairs, where the key is a string and the value is a any JSON value. The value may be a string, number, array, object.
- `"id"`: (Optional) A unique identifier for this feature.
- `"ref"`: (Optional) A reference to an external resource, e.g. URI to a zarr strcture, e.g. "s3://zarr-demo/store/my_array.zarr".
- `"parentId"`: (Optional) A reference to the parent feature, e.g. the id of the feature that this feature is a part of.
- `"feeatureClass"`: (Optional) A string indicating the class of the feature, e.g. "cell", "nucleus", "mitochondria", etc.

#### Special Feature Objects

- **Image**: An image MUST have the following key-value pairs in its “properties” object:

    - `"type"`: A string with the value “Image”
    - `"URI"`: A string with the image URI, e.g. “./image_1.tif"
    An Image MUST also have a geometry object (as its “geometry” member) of type "Polygon", subtype “Rectangle”, indicating the shape of the image. An Image may have the following additional key-value pairs in its “properties” object:
    - `"correction"`: A list of coordinates indicating the relative correction of the image, e.g. `[1, 2]` indicating a correction of 1 units in the x direction and 2 units in the y direction, with units as defined by the coordinate system. If the coordinate system is not defined, the units are pixels.

### FeatureCollection Object

A FeatureCollection object is a JSON object representing a collection of feature objects. A FeatureCollection object has a member with the name `"features"`. The value of `"features"` is a JSON array. Each element of the array is a Feature object as defined above. It is possible for this array to be empty. Additionally, it may have the following members:

- `"properties"`: (Optional) A JSON object containing properties and metadata specific to the feature collection, and which apply to all features of the collection, or a JSON null value. It has the same structure as the `"properties"` member of a Feature object.

#### Special FeatureCollection Objects

- **StitchingVector**: Represents a stitching vector, and MUST have the following key-value pairs in its “properties” object:
    - `"type"`: A string with the value “StitchingVector”

    Any object of a StitchingVector “features” array MUST be an “Image” special type of features object.
