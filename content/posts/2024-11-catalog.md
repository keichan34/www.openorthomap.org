+++
title = 'The Catalog'
date = 2024-11-02T06:50:14+09:00
+++

The Catalog serves as the index of all geospatial data managed by an OpenOrthoMap instance. It's structured to function as a set of static files, enabling it to be served from virtually any web server setup - even something as lightweight as a Raspberry Pi. This approach, implemented in the initial proof-of-concept cloud service, maximizes flexibility for both cloud-based and local deployments.

### Core of the Catalog: S2 Index

At the heart of the Catalog lies the [S2 Geometry](http://s2geometry.io/) index, chosen for its efficient subdivisibility and spatial locality. Unlike hexagonal grids, which don’t nest cleanly, the S2 index subdivides cells in a way that’s both hierarchical and precise. Additionally, it leverages a Hilbert curve to ensure spatially close cells are stored close together in the index, improving data locality.

### Structure of Index Files

Each S2 cell with relevant data, from level 0 to the maximum depth of level 12, has a corresponding index file in the form of a CSV. These files contain metadata and references to data files, keeping the structure simple and human-readable. Below is an example of an index file for a level 9 cell (`353ce4.csv`):

```csv
meta,version,1
meta,count,2
file,1,Hello World,hello-world.pmtiles,1.000
file,1,Foo,foo.pmtiles,0.750
```

### Index File Breakdown

1. Version Information:
   The first line specifies the version of the CSV index format (currently `1`). This format prioritizes clarity and simplicity, though it may evolve in future versions.

2. File Count:
   The second line (`meta,count,2`) indicates that two files are associated with this S2 cell. Note that a single file can span multiple cells, typically calculated using a bounding box around the file’s coverage.

3. File References:
   Each `file` line includes the following fields: the line type (`file`), file format (`1`), file display name, file name, and cell intersection area. Currently, format `1` corresponds to PMTiles Tileset (Raster). Cell intersection area is in a range 0 to 1, corresponding to the ratio of intersection area to cell area.

### Handling File Limits

The `count` value can exceed the number of `file` entries. When this occurs, it means either:
- There were too many files to list in one file.
- Some files have insufficient coverage at this cell level to be relevant.

The reference implementation currently excludes files with an intersection area of 0.1 or less. The most detailed level will always contain all files.

Note that when a file completely covers a cell, it may only be in the parent catalog file. For this reason, when requesting coverage for a specific area, make sure to not only request the catalog files for covering cells, but also their parent cells as well.

## About Future Changes

This spec is under heavy development and may change without warning in the future.

Changelog:

* Add display name, intersection area to file metadata
* Remove subcell counts
