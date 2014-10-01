---
layout: post
title: NYC Building Perimeters
tags: javascript webgl nyc
---

I love NYC and WebGL. So I married the two by rendering NYC building perimeters with [ThreeJS](http://threejs.org/) using [NYC Open Data](https://nycopendata.socrata.com/).

<!--more-->

Starting with NYC:

![NYC](https://raw.githubusercontent.com/dimroc/nyc_building_perimeters/master/app/assets/images/icons/nyc.png)

We can zoom into a neighborhood:

![LES Buildings](https://github.com/dimroc/nyc_building_perimeters/raw/master/public/readme/NbcLowerEastSideManhattan.png)

Batches were broken up by neighborhood.

![Batch by hood](https://github.com/dimroc/nyc_building_perimeters/raw/master/public/readme/buildingsInNeighborhoods.png)

[See it for yourself](http://www.dimroc.com/nyc_building_perimeters/#/neighborhoods/lower-east-side).

Three takeaways I didn't foresee:

1. The heavy downloads needed to get to all the 3D json files to the browser (it's even more than I had feared).
2. The large amounts of RAM needed to render the models (no fancy LOD given the zoom levels).
3. The poor garbage collection when toggling between neighborhoods exacerbated 2.

See more technical details on the [github page](https://github.com/dimroc/nyc_building_perimeters).

Check out the release of [ViziCities](http://vizicities.com/), an impressive rendering of cities. Check out [NYC](http://vizicities.apps.rawk.es/demo.html#40.71432818342427,-73.98659111120529).
