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
meta,subcell-count,0,2
meta,subcell-count,1,2
meta,subcell-count,2,2
meta,subcell-count,3,2
file,1,hello-world.pmtiles
file,1,foo.pmtiles
```

### Index File Breakdown

1. Version Information:
   The first line specifies the version of the CSV index format (currently `1`). This format prioritizes clarity and simplicity, though it may evolve in future versions.

2. File Count:
   The second line (`meta,count,2`) indicates that two files are associated with this S2 cell. Note that a single file can span multiple cells, typically calculated using a bounding box around the file’s coverage.

3. Subcell Counts:
   The subsequent `meta,subcell-count` lines show how many files are contained within each subcell. Here, each subcell of this level 9 cell contains two files. Importantly, these files may not be the same for every subcell, and subcell counts are updated progressively from the most detailed cells to the root cells.

4. File References:
   Each `file` line includes three fields: the line type (`file`), file format (`1`), and file name. Currently, format `1` corresponds to PMTiles Tileset (Raster).

### Handling File Limits

The `count` value can exceed the number of `file` entries. When this occurs, it means either:
- There were too many files to list in one file.
- Some files have insufficient coverage at this cell level to be relevant.

Although the index format has no upper limit on files, the reference implementation restricts each cell’s list to a maximum of 10 files, prioritized by coverage. Files covering less than 10% of a cell may be omitted, though these limits do not apply at the maximum level (12).
