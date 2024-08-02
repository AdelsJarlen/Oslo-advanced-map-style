# Straßenraumkarte aka Street Space Map – MapStyle
The [Straßenraumkarte](https://strassenraumkarte.osm-berlin.org/?map=micromap) is a map style with a special focus on the spatial distribution of urban and street spaces, particularly roadways and objects in public spaces or urban land use. It was developed as a base map for OpenStreetMap projects in Berlin-Neukölln, but can now be created in other locations with some effort.

The appeal of this map style, which is aesthetically inspired by architectural plans, lies in its detailed representation of urban environments. The focus is on features such as road surfaces, street furniture, buildings, land use details, or parked cars in the street space. If these things are to be displayed in a location, it is usually necessary to first record them in OpenStreetMap, which can be a time-consuming process (more on this below).

The map style consists of a QGIS project and a pre-processing script for Python in QGIS. The map data can be easily generated and stored as geojson via Overpass queries. The processing of the map data is designed for experiments and details, not for creating large-scale maps. The goal of the map style is initially to enable map representations for individual urban areas of at most a few square kilometers in size, not for entire countries or the world. The technology of the map is not designed for this and is too amateurishly constructed.
Further information about the map (each in German):

German article in Kartographische Nachrichten / Info und Praxis 3/2022: ["Die Neuköllner Straßenraumkarte – Ein detaillierter Plan des öffentlichen Raumes auf Basis freier OpenStreetMap-Geodaten"](https://static-content.springer.com/esm/art%3A10.1007%2Fs42489-022-00119-1/MediaObjects/42489_2022_119_MOESM1_ESM.pdf) (from page 5 / A-10)

Lightning Talk in German at the FOSSGIS Conference 2022 (5 minutes): ["Die Neuköllner Straßenraumkarte – ein hochaufgelöster OSM-Mikro-Mapping-Kartenstil"](https://media.ccc.de/v/fossgis2022-14180-die-neukllner-straenraumkarte-ein-hochaufgelster-osm-mikro-mapping-kartenstil
)

![grafik](https://github.com/SupaplexOSM/strassenraumkarte-neukoelln/blob/main/images/sample_image.jpg)

---

## How can I generate the Street Space Map for my city?

Unfortunately, this is not so simple, as an optimal map representation of the street space map requires a high degree of micromapping in OSM and a cartographic data basis that is only available in a few places (especially road surfaces and on-street parking). In addition, the Neukölln Street Space Map incorporates manually post-processed data on parking in street space - you can also do this in your city, but it is probably time-consuming. Of course, the Street Space Map can also be rendered where this data is not available, but it will look less appealing. Some of the most important data for optimal rendering of the Street Space Map include:

* Road surfaces (-> [area:highway](https://wiki.openstreetmap.org/wiki/Proposal:Area_highway/mapping_guidelines))
* Parking strips (-> [Street Parking](https://wiki.openstreetmap.org/wiki/Street_parking))
* Sidewalk networks (-> [Mapping-Guide für Berlin](https://wiki.openstreetmap.org/wiki/Berlin/Verkehrswende/Gehwege))
* Buildings
* Land use areas (including small-scale ones, e.g., grass areas and shrubs)
* Trees

In addition, the post-processing of OSM data includes an elaborate generation of road markings based on a detailed recording of lane attributes (e.g., lanes, turn:lanes, lane_markings, width, width:lanes, placement, cycleway:*). If these lane attributes are not sufficiently or accurately recorded, the road markings will be generated inaccurately - or should then be better deactivated completely.

Many other things are displayed in the map representation, e.g., street furniture such as control boxes or street lamps, barriers such as fences and gates, or even overhanging parts of buildings if these are defined as building:parts. Ultimately, the Street Space Map is a map style for precise urban micromapping to make the abundance of things in space and their properties as visible as possible.

## Step-by-step guide to your own Street Space Map

Version note: The current version of the Street Space Map was created with QGIS 3.22.4-Białowieża. With older or newer versions, there might be errors in script execution or map display.

0.) OSM micromapping in the area, see above :)

1.) Download data for the desired area. This currently works somewhat cumbersomely via about 20 different Overpass Turbo queries for various layers, e.g., buildings, streets, street areas, or land use. Each query contains a bounding box at the beginning that can be adjusted to the target area (for Berlin-Neukölln, this bounding box is preset as [bbox:52.4543246009110788,13.3924347464750326,52.5009195009107046,13.4859782]). Execute the queries and save the data under "layer/geojson/" as GeoJSON with the appropriate file name:

| [amenity.geojson](https://overpass-turbo.eu/s/1CCd) | [area_highway.geojson](http://overpass-turbo.eu/s/1erF) | [barriers.geojson](http://overpass-turbo.eu/s/1cZw) | 
| [bridge.geojson](http://overpass-turbo.eu/s/1cTT) | [building_part.geojson](http://overpass-turbo.eu/s/1cZv) |
[buildings.geojson](http://overpass-turbo.eu/s/1cZx) |
| [entrance.geojson](http://overpass-turbo.eu/s/1cTV) | [highway.geojson](http://overpass-turbo.eu/s/1cLL) | [housenumber.geojson](http://overpass-turbo.eu/s/1cTN) |
| [landuse.geojson](http://overpass-turbo.eu/s/1cTF) | [leisure.geojson](http://overpass-turbo.eu/s/1cTL) | [man_made.geojson](https://overpass-turbo.eu/s/1Jwo) |
| [motorway.geojson](http://overpass-turbo.eu/s/1cTO) | [natural.geojson](http://overpass-turbo.eu/s/1cTD) | [path.geojson](http://overpass-turbo.eu/s/1eG0) |
| [place.geojson](http://overpass-turbo.eu/s/1cTR) | [playground.geojson](https://overpass-turbo.eu/s/1iMm) | [railway.geojson](https://overpass-turbo.eu/s/1izr) |
| [routes.geojson](http://overpass-turbo.eu/s/1eG1) | [waterway.geojson](http://overpass-turbo.eu/s/1cTP) | |

2.) Install/start QGIS and open the Python console ("Plugins > Python Console").

3.) If desired, generate and manually post-process parking space data. For this, parking space data must be recorded in OSM in as much detail as possible - information on this can be found in the OSM Wiki and at the OSM Parking Space Project. This script can be used to generate suitable parking space data: street_parking.py (information on required data and execution can be found there). Save the result under "layer/geojson/parking/street_parking_lines.geojson". However, the generated parking space data is not yet very precisely positioned. It is worth manually post-processing this in QGIS, cleaning up errors, and especially aligning it with real curb edges (e.g., by snapping or manual shifting/snapping).

4.) Adjust location-specific layers. The map style contains individual layers whose extent/nature is oriented towards use in the original map area Berlin/district Neukölln. These can be adapted to your own area if needed, but are otherwise dispensable. These include the layers:
"map_extent" (under "layer/geojson/map_extent/map_extent.geojson"): Can be used later to define the area in which map tiles should be generated.
"map_fog_square" ("layer/geojson/fog/map_fog_square.geojson"): A frame around the map area in the background color to conceal areas behind it.
"kerb_street_areas" and "kerbs (ways, from street_areas)" ("layer/geojson/kerb/kerb_street_areas.geojson"): An external dataset with street areas, e.g., from ALKIS. Can be used in addition to area:highway surfaces from OSM for displaying road surfaces.

5.) Open and execute the post-processing script ("post_processing.py") in QGIS. The post-processing script contains a whole series of steps that improve the map display, e.g., geometry adjustments and especially the creation of layers for special representations. The largest part of the script is devoted to generating road markings. Depending on the size of the area and available computing power, carrying out all steps can take some time. The individual processing steps can be (de)activated individually (by setting 0 or 1 right at the beginning of the script) and can thus be executed in smaller "packages" or individually if, for example, memory or display problems occur during generation. If necessary, unnecessary processing steps can also be omitted, e.g., if no parking space data is available (see step 3). Check the coordinate reference system again before executing the script (see next step).

6.) Check the coordinate reference system. The Street Space Map uses metric coordinate reference systems for many layers and geometric operations to be able to display things in space with (centi)meter accuracy. The default setting of the map style assumes that the displayed area is in UTM zone 32, which should be quite suitable for cities in Germany (even if some cities are in the neighboring zone). If the target area is in another world region, a suitable reference system must be chosen. In this case, not only must the script line 'crs_to = "EPSG:25833"' be adjusted before script execution (step 5), but in the next step, the reference systems of all map layers that are in EPSG:25833 may also need to be adjusted. If the target area is at a significantly different latitude than Berlin (52° N), a scaling factor must also be adjusted later, as otherwise the metric representations will be distorted (the map itself is in WGS 84 / Pseudo-Mercator (EPSG:3857) to enable rendering of map tiles). The formula for this scaling factor is: 1 / cos(latitude), for Berlin e.g., 1 / cos(52°) = 1.62. If this factor in your target area deviates significantly from this (e.g., by more than 0.1), adjust the factor in the next step in QGIS: "Project > Properties > Variables > "scale_factor = '1.62'"

7.) Open the map style in QGIS ("strassenraumkarte.qgz"). If everything went well, all layers will be loaded and displayed. Adjust the latitude-dependent scaling factor if necessary (see step 6). The map can now be exported or map tiles can be generated from it. Tip for generating tiles in QGIS: The native renderer may not reproduce the colors of the Street Space Map correctly. There are two good QGIS extensions for generating map tiles: "QTiles" and "QMetaTiles". "QMetaTiles" has the advantage that it can handle metatiling, which significantly improves the display of symbols and labels at tile boundaries. However, this extension seems to have trouble with variables, so the scaling factor is not taken into account and symbols and textures are not loaded. My workaround: Save the QGIS project as a *.qgs file (xml), open it in an editor and replace the variables "@scale_factor" and "@project_folder" with the scaling factor and an absolute file path.

> Translated using AI (2024) - credit: Claude.ai 3.5 Sonnet
