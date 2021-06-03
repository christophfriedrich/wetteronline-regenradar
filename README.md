# Wetteronline Regenradar

This repository is an attempt to re-engineer [wetteronline.de](https://www.wetteronline.de/)'s Regenradar API that is new as of ca. May 2021.

*Deutschsprachige Leser siehe letzten Abschnitt. (German readers see last section.)*


## Retrieving ground truth image URLs

Using my browser's developer console (Ctrl+K in Firefox, F12 in Chrome) I inspected the network traffic while opening https://www.wetteronline.de/regenradar/nordrhein-westfalen, restricted to just images. Among lots of ads and other stuff, I discovered URLs like the following: 

**URL for Wilhelmshaven area, retrieved 3rd June 2021 ca. 16:30**
https://tiles.wo-cloud.com/composite?format=webp&k=-1426785660&lg=rr&tiles=dG9wb3wxOzswOzB8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9nZW9sYXllci9yYXN0ZXJpbWFnZXMvcnJfdG9wb2dyYXBoeS92MS9aTDgvNTEyLzEzMl84Mi5qcGckcnwyOzswOzE7ZmFsc2V8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9yYWlubGF5ZXJQcm9nLzIwMjEvMDYvMDMvMTQvMTUvdjI0L1pMNy81MjIvc3ByaXRlLzY2XzQwLnBuZzt3ZXR0ZXJyYWRhcmdsb2JhbC9wcm96ZXNzL3RpbGVzL3JhaW5sYXllclByb2cvMjAyMS8wNi8wMy8xNC8xNS92MTMvWkw3LzUyMi9ib3JkZXIvNjZfNDAucG5nJGl8MTs7MDswfGdlby9wcm96ZXNzL2thcnRlbi9wcm9kdWt0a2FydGVuL3dldHRlcnJhZGFyL2dlbmVyYXRlL3Jhc3RlclRpbGVzL3JyX2dlb292ZXJsYXlfY2l0aWVzX2V4Y2x1ZGVkL3YyL1pMOC81MTIvMTMyXzgyLnBuZw==&time=20210603-1415-2

Decoded tiles parameter (see below how to do this):
> topo|1;;0;0|wetterradar/prozess/tiles/geolayer/rasterimages/rr_topography/v1/ZL8/512/132_82.jpg$r|2;;0;1;false|wetterradar/prozess/tiles/rainlayerProg/2021/06/03/14/15/v24/ZL7/522/sprite/66_40.png;wetterradarglobal/prozess/tiles/rainlayerProg/2021/06/03/14/15/v13/ZL7/522/border/66_40.png$i|1;;0;0|geo/prozess/karten/produktkarten/wetterradar/generate/rasterTiles/rr_geooverlay_cities_excluded/v2/ZL8/512/132_82.png

**URL for Münster area, retrieved 3rd June 2021 ca. 16:30**
https://tiles.wo-cloud.com/composite?format=webp&k=-1426785660&lg=rr&tiles=dG9wb3wxOzswOzB8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9nZW9sYXllci9yYXN0ZXJpbWFnZXMvcnJfdG9wb2dyYXBoeS92MS9aTDgvNTEyLzEzMl84NC5qcGckcnwyOzswOzA7ZmFsc2V8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9yYWlubGF5ZXJQcm9nLzIwMjEvMDYvMDMvMTQvMTUvdjI0L1pMNy81MjIvc3ByaXRlLzY2XzQyLnBuZzt3ZXR0ZXJyYWRhcmdsb2JhbC9wcm96ZXNzL3RpbGVzL3JhaW5sYXllclByb2cvMjAyMS8wNi8wMy8xNC8xNS92MTMvWkw3LzUyMi9ib3JkZXIvNjZfNDIucG5nJGl8MTs7MDswfGdlby9wcm96ZXNzL2thcnRlbi9wcm9kdWt0a2FydGVuL3dldHRlcnJhZGFyL2dlbmVyYXRlL3Jhc3RlclRpbGVzL3JyX2dlb292ZXJsYXlfY2l0aWVzX2V4Y2x1ZGVkL3YyL1pMOC81MTIvMTMyXzg0LnBuZw==&time=20210603-1415-2

Decoded tiles parameter:
> topo|1;;0;0|wetterradar/prozess/tiles/geolayer/rasterimages/rr_topography/v1/ZL8/512/132_84.jpg$r|2;;0;0;false|wetterradar/prozess/tiles/rainlayerProg/2021/06/03/14/15/v24/ZL7/522/sprite/66_42.png;wetterradarglobal/prozess/tiles/rainlayerProg/2021/06/03/14/15/v13/ZL7/522/border/66_42.png$i|1;;0;0|geo/prozess/karten/produktkarten/wetterradar/generate/rasterTiles/rr_geooverlay_cities_excluded/v2/ZL8/512/132_84.png

**URL for Münster area, retrieved 3rd June 2021 ca. 19:00**
https://tiles.wo-cloud.com/composite?format=webp&k=-1350085305&lg=rr&tiles=dG9wb3wxOzswOzB8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9nZW9sYXllci9yYXN0ZXJpbWFnZXMvcnJfdG9wb2dyYXBoeS92MS9aTDgvNTEyLzEzMl84NC5qcGckcnwyOzswOzA7ZmFsc2V8d2V0dGVycmFkYXIvcHJvemVzcy90aWxlcy9yYWlubGF5ZXJQcm9nLzIwMjEvMDYvMDMvMTgvMzUvdjI0L1pMNy81MjIvc3ByaXRlLzY2XzQyLnBuZzt3ZXR0ZXJyYWRhcmdsb2JhbC9wcm96ZXNzL3RpbGVzL3JhaW5sYXllclByb2cvMjAyMS8wNi8wMy8xOC8zMC92MTQvWkw3LzUyMi9ib3JkZXIvNjZfNDIucG5nJGl8MTs7MDswfGdlby9wcm96ZXNzL2thcnRlbi9wcm9kdWt0a2FydGVuL3dldHRlcnJhZGFyL2dlbmVyYXRlL3Jhc3RlclRpbGVzL3JyX2dlb292ZXJsYXlfY2l0aWVzX2V4Y2x1ZGVkL3YyL1pMOC81MTIvMTMyXzg0LnBuZw%3D%3D&time=20210603-1835-2

Decoded tiles parameter:
> topo|1;;0;0|wetterradar/prozess/tiles/geolayer/rasterimages/rr_topography/v1/ZL8/512/132_84.jpg$r|2;;0;0;false|wetterradar/prozess/tiles/rainlayerProg/2021/06/03/18/35/v24/ZL7/522/sprite/66_42.png;wetterradarglobal/prozess/tiles/rainlayerProg/2021/06/03/18/30/v14/ZL7/522/border/66_42.png$i|1;;0;0|geo/prozess/karten/produktkarten/wetterradar/generate/rasterTiles/rr_geooverlay_cities_excluded/v2/ZL8/512/132_84.png

=> URLs 1 and 2 show different areas at the same time, URLs 2 and 3 the same area at different times, which is helpful for comparisons. We can use this information to investigate further.


## Parameters

Obviously, there are four parameters:

1. `format`: Image format. Known possible values: `png`, `webp`. Defaults to `png` if it's an unrecognised value (I also tried `gif`, `jpg`, `jpeg`, which all resulted in a PNG being delivered anyway) or not specified at all.

2. `k`: Very mysterious. Being a ten-digit integer starting with a `1`, I was immediately reminded of a Unix timestamp. But the values from the URLs above correspond to days in 2012 and 2015 respectively, so that doesn't make much sense. Also it's a negative number?! Removing this parameter alltogether doesn't seem to change anything visually. But their webapp changes it over time. Yeah, very mysterious.

3. `time`: Obviously a datetime parameter which expects the format `YYYYMMDD-HHMM` with an appended `-2`, e.g. `20210603-1415-2`. I have no idea what that `-2` stands for.

4. `tiles`: Base64-encoded string (recognisable especially with the trailing `==`) which defines the layers of the map. You can decode it in JavaScript with the very weirdly named function `atob`, and encode again with `btoa`. The decoded value is structured into multiple components that are separated by pipes (`|`).


## The `tiles` parameter

Splitting an example value of the `tiles` parameter at the pipes, we get the following components:

1. topo
2. 1;;0;0
3. wetterradar/prozess/tiles/geolayer/rasterimages/rr_topography/v1/ZL8/512/132_84.jpg$r
4. 2;;0;0;false
5. wetterradar/prozess/tiles/rainlayerProg/2021/06/03/18/35/v24/ZL7/522/sprite/66_42.png;wetterradarglobal/prozess/tiles/rainlayerProg/2021/06/03/18/30/v14/ZL7/522/border/66_42.png$i
6. 1;;0;0
7. geo/prozess/karten/produktkarten/wetterradar/generate/rasterTiles/rr_geooverlay_cities_excluded/v2/ZL8/512/132_84.png

Some observations I made about these:
- The `topo` in component 1 obviously defines that this is a map with a topographic background. I don't know of any other possible values (because I didn't know which to try).
- The semicolon-separated components are probably meta properties of the following map layer definition. Maybe things like transparency etc.
  - There are five properties, let's call them `a;b;c;d;e`.
  - `a` seems to accept values between 1 and 9 (above 9 the rain disappears). Altering it in component 2 doesn't seem to have any effect, *altering it in component 4 massively changes the appearance of the rain patches!*
  - `b` is never used at all?!
  - `c` seems to be irrelevant, but doesn't accept strings.
  - `d` must be 0 or 1, not 2 or higher.
  - `e` is only sometimes specified. Visually it doesn't seem to make a difference whether it's `true` or `false` or any other string or left out alltogether.
- In the paths of components 3, 5 and 7, `ZL` probably stands for "zoom level".
- I don't really know what the `522` or `512` does or which other values work in which combination.
- The filename defines the X and Y position of the tile. The first part is the east-west position, the second the north-south one. They can only be incremented in steps of 2.
- The numbers in the filename of ZL8 paths are double of those in ZL7 paths.
- Without the trailing `$r`, the rain is not displayed on the map. Likewise the trailing `$i` causes the borders and city names to be displayed.
- In component 5, the latter path can be removed without any visual consequences *as long as the trailing `$i` is appended to the first path*.
- Also in component 5, instead of `v24` the values `v21` through `v26` work too but result in somewhat different rain patches. Values above or below cause the rain to disappear.
- If one leaves out the `_cities_excluded` in component 7, the result is a map *with* city names. But caution: Even when using the `excluded` version, the lines of borders and rivers are not drawn where there would be a city label if they were enabled!
- Removal of the topographic features (but keeping of the grey background) can be achieved by misspelling or removing `rr_topography` from the path in 3.


## Interdependence of parameters

To me it appears that the parameters `format` and `k` can be entirely left out. `time` is important to be there, but doesn't define which timestamp is actually displayed; this seems to be ONLY governed by component 5 of the `tiles` parameter.


## Reengineering tool

To be able to play with the parameters somewhat conveniently, I wrote a little Vue.js webapp that lets you modify the parameters and displays the corresponding image on-the-fly. To use it, open the file `tool.html` in a browser.


## Informationen auf Deutsch

Ich habe nicht vor, eine volle Übersetzung dieser Informationen auf Deutsch anzubieten, aber da Englisch für einige eine Hürde sein mag, habe ich das, was ich sowieso schon auf Deutsch getippt hatte (bevor ich mich entschieden habe alles auf Englisch zu schreiben), auch noch mit hochgeladen. Siehe die Datei [deutsch.md](deutsch.md).

I will keep information on this page in English and don't plan to add a full German translation of it, but added my original German notes in the file [deutsch.md](deutsch.md) because reading these may be easier for some.
