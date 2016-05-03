translating-us-topo-with-gdal
=============================

Translating US Topo 7.5 min GeoPDF's to other custom raster formats with GDAL.

## About

A guide to customizing and translating the US Topo GeoPDF to other raster formats.  Create georeferenced rasters which better fit your project needs from the layers of the US Topo GeoPDF.

## Requirements

- __GDAL >= 1.8.0__ or to create GeoPDF __GDAL >= 1.10.0__
- PDF GDAL Driver
  - [Poppler](http://poppler.freedesktop.org/) (GPL-licensed)
  - [PoDoFo](http://podofo.sourceforge.net/) (LGPL-licensed)
  - [PDFium](https://pdfium.googlesource.com/pdfium/) (New BSD-licensed, supported since GDAL 2.1.0)

Check to verify GeoPDF format is avaiblale with your GDAL installation.

```bash
$ gdalinfo --formats
  [...]
    PDF -raster,vector- (rw+vs): Geospatial PDF
  [...]
```

## Examples

_*Note: Example commands should apply to any platform with GDAL_

__Download the example [Raleigh West NC US Topo](https://prd-tnm.s3.amazonaws.com/StagedProducts/Maps/USTopo/1/15702/5585347.pdf)__

#### Get US Topo GeoPDF Layer information

Retrieve the metadata information for the Raleigh West US Topo and the avaiblale data layers which will be used to customize output rasters.

```bash
$ gdalinfo -mdd LAYERS raleigh_west.pdf
```

#### Create a GeoTiff of only the terrain layers

We will create a GeoTiff of only the terrain data (hillshade and contours) by __explicitly including__ the layers to be used in the command.  This output will also include the US Topo map collar, legend, and grids.

```bash
$ gdal_translate -of "GTiff" -co COMPRESS=DEFLATE -co TILED=YES \
  --config GDAL_PDF_LAYERS "Map_Collar,Map_Frame.Projections_and_Grids,Map_Frame.Terrain" \
  --config GDAL_PDF_BANDS 3 --config GDAL_PDF_DPI 300 \
  raleigh_west.pdf raleigh_west_terrain.tif
```

#### Create a GeoTiff of only the vector layers

We will create a GeoTiff of only the vecotr data by __explicitly excluding__ the raster layers (Orthoimagery and Shaded Relief).  This output will also include the US Topo map collar, legend, and grids.

```bash
$ gdal_translate -of "GTiff" -co COMPRESS=DEFLATE -co TILED=YES \
  --config GDAL_PDF_LAYERS_OFF "Map_Frame.Terrain.Shaded_Relief,Images" \
  --config GDAL_PDF_BANDS 3 --config GDAL_PDF_DPI 300 \
  raleigh_west.pdf raleigh_west_vector.tif
```

#### Clip the GeoTiff of the vector only map to the neatline

We will create a GeoTiff __clipped__ to the topo neatline of only the vector data.  This output could be used to mosaic/stitch adjacent clipped topos to create a larger, seamless map.

_*Note - This example uses a Node.js package [Wellknown](https://github.com/mapbox/wellknown) to convert the topo's neatline Well Known Text geometry to GeoJSON for clipping.  If Node.js is not available on your system, the projected neatline geojson is included in this project._

```bash
# Save the neatline geometry from the GeoPDF topo as a variable
$ NEATLINE=$(gdalinfo raleigh_west.pdf | grep NEATLINE | sed 's/^.*POLYGON/POLYGON/')

# Install wellknown to convert the neatline WKT to unprojected geojson
$ npm install -g wellknown
$ echo $NEATLINE | wellknown > neatline_unprojected.geojson

# Project the geojson to the topo UTM Zone 17N NAD 83
$ ogr2ogr -f GeoJSON -a_srs EPSG:26917 neatline.geojson neatline_unprojected.geojson

# Translate and clip topo to the neatline with only vector data.
$ gdalwarp -of GTiff -dstalpha -cutline neatline.geojson \
  -co COMPRESS=DEFLATE -co TILED=YES --config GDAL_PDF_LAYERS_OFF \ 
  "Map_Collar,Images,Map_Frame.Projections_and_Grids,Map_Frame.Terrain.Shaded_Relief" \
  --config GDAL_PDF_BANDS 3 --config GDAL_PDF_DPI 300 \
  raleigh_west.pdf raleigh_west_vector_clip.tif

```

## Sources

#### US Topo

- US Topo Program [information](http://nationalmap.gov/ustopo/)
- Map based browsing and downloads [Map Store](http://store.usgs.gov)
- Text and attribute based search [ScienceBase](https://www.sciencebase.gov/catalog/)
- Browse direct [ downloads](http://prd-tnm.s3-website-us-west-2.amazonaws.com/?prefix=StagedProducts/Maps/)

#### GDAL/OGR

- CLI Commands' Reference [GDAL Utilities](http://www.gdal.org/gdal_utilities.html)
- GeoPDF format [information](http://www.gdal.org/frmt_pdf.html)
- GeoTiff format [information](http://www.gdal.org/frmt_gtiff.html)
- Command [cheat sheet](https://github.com/dwtkns/gdal-cheat-sheet)

## Contact

Andrew Burnes
apburnes@gmail.com
