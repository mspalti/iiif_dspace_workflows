# Overview

This IIIF implentation relies on two basic features of DSpace. 

* The `bundle` layer continues to be part of DSpace 7. This implementation uses bundles to identify the bitstreams that will be added to IIIF `canvases` (via the `IIIF` bundle) or to related resource annotation lists (via the `OtherContent` bundle). The `bundle` also assures that bitstreams (and therefore `canvases`) appear in the proper order.
* The `entity.type` feature is new with DSpace 7. It is used to flag an item as an IIIF resource. Items with an `entity.type` of `IIIF` or `IIIFSearchable` incorporate the Mirador viewer into the item display and intialize the viewer with the `manifest` URL for the item. `IIIFSearchable` entities are also intialized with the IIIF search results when the item is retrieved from a DSpace discovery result list.

To render an item as IIIF both conditions are required.  The `entity.type` must be one of the two IIIF types, and bitstreams must be added to the `IIIF` bundle.





