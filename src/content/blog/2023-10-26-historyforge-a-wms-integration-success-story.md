---
title: "HistoryForge: A WMS Integration Success Story"
description: To integrate with HistoryForge, I implemented WMS endpoints for all maps.
pubDate: 2023-10-26
heroImage: ../../assets/Screenshot from 2025-07-13 17-11-50.png
category: tech,collabs
published: true
---
Working with the [HistoryForge](https://historyforge.net) team we have gotten the mosaicked New Orleans 1895, vol. 1 edition into their web map interface! You can find it in the map overlays at [https://sela.historyforge.net/forge](https://sela.historyforge.net/forge). (Also admire the Robinson Atlas and other layers Elizabeth has added while you are there!) Some may recall this edition was the focus of our first georeference-a-thon this past summer. Happy to see it in use outside of [OldInsuranceMaps.net](http://OldInsuranceMaps.net)! The best thing about this integration is that it spurred some development on the HF platform, so now _any_ maps/mosaics from [OldInsuranceMaps.net](http://OldInsuranceMaps.net) can be pulled directly in. No need to move files, etc. I have a feeling there will be more HF collaboration in the future!

On the tech side, I'd like to acknowledge Github user _mindless-bureaucrat_ who [added a WMS endpoint to TiTiler](https://github.com/developmentseed/titiler/pull/572) early this year (and of course endless appreciation for TiTiler's core maintainer, [Vincent Sarago](https://github.com/vincentsarago)). When they did, I knew it would be useful to us sooner or later, and HistoryForge is a wonderful first use. Now any sheets or mosaics from [OldInsuranceMaps.net](http://OldInsuranceMaps.net) are available through this standard (get in touch if using layers that way interests you!).