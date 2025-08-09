---
title: Using GDAL's VRTs for Georeferencing, from "A look inside OldInsuranceMaps.net"
description: In a recent presentation I described in more detail how the warping
  process works in OldInsuranceMaps.net.
pubDate: 2025-08-09
heroImage: ../../assets/Screenshot from 2025-07-24 11-11-20.png
category: presentations
published: true
---
Recently I gave a presentation about [OldInsuranceMaps.net](http://OldInsuranceMaps.net) for [I-GUIDE's Virtual Consulting Office](https://i-guide.io/i-guide-vco/) speaker series and I was able to talk a bit more about the technical details of the platform than I have in the past, which was really fun and a great excuse to make some more diagrams. One element of the system I described is the way that the georeferencing process relies on [GDAL's "virtual raster"](https://gdal.org/en/stable/drivers/raster/vrt.html) driver (i.e. file format), and how this is used for, among other things, the live preview feature of the georeferencing interface. Here's a short summary of that part of the talk with a few relevant slides. For more, you can [view the full recording](https://i-guide.io/i-guide-vco/a-look-inside-oldinsurancemaps/), or find the [slides here](https://docs.google.com/presentation/d/1zv23HALxZ_3VP8ox3Mr089qDxOn4mNrO2zOKWKsseiA/edit?usp=sharing) (note, I've updated some of them a bit since the recording).

The basic georeferencing process, which turns a non-spatial image into a geospatial dataset, works by chaining two VRTs together and then writing out a GeoTIFF (specifically, a Cloud-Optimized GeoTIFF, or "COG"). The first VRT references the original image and embeds the user-submitted ground control points (GCPs). The second VRT references the first VRT, and embeds the warping information (which transformation to use). The final step is a GDAL translation process which turns the warped VRT into an actual COG.

![](../../assets/Screenshot%20from%202025-08-09%2008-37-19.png)

Importantly, a VRT is just a small XML file, so creating one is nearly instantaneous and requires virtually no storage space. In OldInsuranceMaps, they are also disposable, as the actual database stores all of the information that gets embedded into them. They can be recreated on demand and then deleted after they are no longer needed.

With this in mind, the live preview that is presented in the georeferencing interface is created by injecting user-generated information directly into the VRT linkage chain, and updating that information on GCP creation or modification using a feedback loop. Creating this preview is actually very simple, because TiTiler, and MapServer which I used in the original implementation of this feature, can read and tile data directly from a VRT--no need to actually generate a GeoTIFF.

![](../../assets/Screenshot%20from%202025-08-09%2008-37-14.png)

Finally, to create the mosaics that are the ultimate end goal of the entire system, even more VRTs are added. First, on top of the warped VRT a new one is created that incorporates the layer's mask, drawn by users through a unified, "multimask" interface. Then, all of the cropped VRTs are wrapped up into a single mosaicked VRT (mosiacking is actually what VRTs are most commonly used for), and that final VRT is then translated into a single (sometimes quite large) COG.

![](../../assets/Screenshot%20from%202025-08-09%2008-11-35.png)

There is even more you can do with this concept of chaining VRTs together, for example enabling the support of custom projections during the georeferencing process. Sanborn Maps are all very large-scale, so warping to a non-Web Mercator projection isn't necessary or useful. But I have implemented and tested this idea out with much smaller-scale maps that were drawn in a non-Mercator projection, and it works flawlessly. I described the concept in more detail in [this old comment on GitHub](https://github.com/cogeotiff/rio-tiler/discussions/565#discussioncomment-5844734), and hope to properly implement it on the platform someday.