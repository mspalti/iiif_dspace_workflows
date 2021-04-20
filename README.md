# Overview

This IIIF implentation relies on two basic features of DSpace. 

* The `bundle` layer. This implementation uses bundles to identify the bitstreams that will be added to IIIF `canvases` (via the `IIIF` bundle) or to related resource annotation lists (via the `OtherContent` bundle). The `bundle` also assures that bitstreams (and therefore `canvases`) appear in the proper order.
* The `entity.type` feature is new with DSpace 7. It is used here to flag an item as an IIIF resource. Items with an `entity.type` of `IIIF` or `IIIFSearchable` incorporate the Mirador viewer into the item display and intialize the viewer with the `manifest` URL for the item. `IIIFSearchable` entities are also intialized with the IIIF search results when the item is retrieved from a DSpace discovery result list.

To render an item as IIIF both conditions are required.  The `entity.type` must be one of the two IIIF types, and bitstreams must be added to the `IIIF` bundle.

This design has a couple implications that merit consideration.  
* The assets in the IIIF bundle are viewed and accessed only via the IIIF viewer. The IIIF viewer also provides downloads and citation information for these assets. Any bitstreams in the default ORIGINAL bundle continue to be accessed via the DSpace Item UI as before. 
* Existing DSpace records with images in the ORIGINAL bundle need to be modified by transferring assets to the IIIF bundle. There may be reasonable ways to avoid this requirement, possibly by relying on bitstream metadata.  





