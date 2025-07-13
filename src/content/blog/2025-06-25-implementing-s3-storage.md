---
title: Implementing S3 Object Storage
description: Recently I updated the code base and transferred all content to S3
pubDate: 2025-06-29
heroImage: ../../assets/mackerel-sky.png
category: tech
published: false
---
## What is S3 Object Storage?

S3 object storage is a protocol developed by Amazon Web Services for handling file storage outside of a normal server environment and "in the cloud" instead. In other words, it's a simplified way of uploading, downloading, or directly accessing files that are stored in remote "buckets". In these buckets, storage is (broadly speaking) very cheap and practically unlimited. Many other companies have since adopted the S3 API so their own cloud storage offerings are compatible with it.

## Implementing a new storage backend

To this point, [OldInsuranceMaps.net](http://OldInsuranceMaps.net) has functioned by downloading image files from the Library of Congress (or other sources, in certain contexts) and storing those files on the main server. Once the images are available locally, the splitting, georeferencing, and mosaicking operations are performed directly on those files, creating even more files (ultimately, [Cloud Optimized GeoTIFFs](https://cogeo.org/)) on the server. Really, the entire point the platform is to build tools and workflows around the creation and management of these files.

One downside to this approach is that the server has limited storage space, let's say about 500gb. This has been plenty for the last few years (especially as [I've implemented better compression methods](https://github.com/ohmg-dev/OldInsuranceMaps/pull/145) over time) but as available space inevitably diminishes, I decided it was finally time to progress to a new storage strategy.

To implement S3 storage for all content in OIM, there were two general tasks:

1.  Update the code base to work with remotely stored content instead of relying on files on disk
    
2.  Upload all existing content to an S3 bucket
    

How much existing content? Well, quite a lot. After a big server cleanup effort (this was the first time I had comprehensively dealt with all kinds of orphan files and remnants from past experiments) there were _roughly_ 130,000 files using about 430gb of disk space.

There were also a couple of important constraints to work around:

1.  S3 storage needs to be optional, so that future installations of the code base can run just using local file storage
    
2.  Gotta keep the site running! There is a bit of operational inertia that must be respected on the platform, now that there is a lot of content on it and people using it everyday. Practically speaking, this means making slow, incremental steps.
    

### Code compatibility

OIM is build with the Django web framework, and there is a solid, standardized way for implementing S3 storage using [django-storages.](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html) In theory then, the storages plugin could be installed, a switch could be flipped, and violá, Django will read and write to an S3 bucket instead of the local file system.

However, it wasn't that simple. Due to the fact that a lot of splitting, georeferencing, and mosaicking operations had been built with the expectation of local file storage, some of them failed once S3 got involved. Luckily, GDAL (on which OIM has relied from the start) has a [fantastic virtual file system](https://gdal.org/en/stable/user/virtual_file_systems.html) that allows it to read directly from S3 urls, and combining that with another GDAL feature, [virtual rasters or VRTs](https://gdal.org/en/stable/drivers/raster/vrt.html), offered plenty of flexibility to work with. Ultimately, the implementation required rewriting and improving some basic functions at the core of the app, but produced a better code base in the end. If you are interested, you can read all the gory details in [this pull request](https://github.com/ohmg-dev/OldInsuranceMaps/pull/280).

### Syncing the content

With the code updated, I had only to transfer all files to an S3 bucket and then point the application to it. I chose to use [Wasabi](http://wasabisys.com), a very cheap S3 storage solution, but the beauty of this upgrade is that the code is provider agnostic--in the future switching to Amazon or Linode S3 Object Storage should be fairly painless.

In the week before the final deployment, I configured [aws cli](https://aws.amazon.com/cli/) to point to a new backup bucket in Wasabi, and used it to periodically sync the entire set of files from the production server to that bucket. About 100 new files get created everyday, so the `sync` operation (which only updates new or altered files) was crucial for seamlessly keeping everything up-to-date.

## The result

It went well! Since the switch all new files have gone straight to S3, and all of the existing interfaces and viewers on the site now run from COGs in S3 instead of through `nginx` on the server.

One hiccup I encountered was that the setting `AWS_QUERYSTRING_AUTH` in Django was automatically set to `True` , causing the URLs that the backend returned to have embedded expiration dates in them. Everything would be fine when I re-generated the lookups (which include saved URLs for things like thumbnails) but then the next day these same URLs would be "unauthorized" and no longer work. Setting `AWS_QUERYSTRING_AUTH=False` fixed this issue.

Wasabi is _very_ cheap, just $7/month per terabyte, meaning that the 440gb I have in the bucket (doubled if you include the backup bucket) falls within the lowest tier of usage. A very quick back-of-the-envelope calculation shows that if the _entire_ LOC Sanborn Map collection were georeferenced on OIM, storage would be under 7tb ($49/month) or double that with a backup. That seems like a pretty good deal.

I've talked about making this upgrade for a while, so it feels good to finally do it. Technically, I could delete all old content from the server and then downgrade it to reduce the monthly bill, but storage was just one of the elements in play, because the splitting and warping processes benefit from a powerful server, not to mention I still run TiTiler directly on the server as well. All in good time...