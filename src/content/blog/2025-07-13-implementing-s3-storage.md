---
title: Implementing S3 Storage
pubDate: 2025-06-25
category: tech
published: false
---
## Why?

To this point, [OldInsuranceMaps.net](http://OldInsuranceMaps.net) has functioned by downloading image files from the Library of Congress (or other sources, in certain contexts) and storing those files on the main server. Once the images are available locally, the splitting, georeferencing, and mosaicking operations are performed directly on those files, creating even more files (ultimately, [Cloud Optimized GeoTIFFs](https://cogeo.org/)) on the server. Really, the entire point the platform is to build tools and workflows around the creation and management of these files.

The problem with this is that the server that OIM runs on has limited storage space, let's say about 500gb. This has been plenty for the last few years (especially as [I've implemented better compression methods](https://github.com/ohmg-dev/OldInsuranceMaps/pull/145) over time) but as available space inevitably diminishes, I decided it was finally time to progress to a new storage strategy.

## What is S3 Object Storage?

S3 object storage is a protocol developed by Amazon Web Services (since implemented by many other companies) for handling file storage outside of a normal server environment and "in the cloud" instead. In other words, it's a simplified way of uploading, downloading, or directly accessing files that are stored in remote "buckets". In these buckets, storage is (broadly speaking) very cheap and practically unlimited.

## By the numbers...

[https://github.com/ohmg-dev/OldInsuranceMaps/pull/280](https://github.com/ohmg-dev/OldInsuranceMaps/pull/280)