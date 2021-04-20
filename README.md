# Overview

This IIIF proposal relies on two basic features of DSpace. 

* The `bundle` layer. The implementation uses bundles to identify bitstreams that will be added 
  to IIIF `canvases` (the `IIIF` bundle) or to related resource annotation lists (the `OtherContent` bundle). 
  The bundles also assures that bitstreams (and therefore `canvases`) appear in the proper order.
* The `entity.type` feature. This is new with DSpace 7 and is used to flag items as IIIF resources. 
  Items with an `entity.type` of `IIIF` or `IIIFSearchable` incorporate the Mirador viewer into the  
  display and initialize the viewer with the `manifest` URL of the item. Also, `IIIFSearchable` entities are  
  initialized with the search results when the item is retrieved from a DSpace discovery result list.

To render an item as IIIF both conditions are required.  The `entity.type` must be one of the two IIIF types, 
and bitstreams must be added to the `IIIF` bundle.

These requirements have implications that merit consideration.

* The assets in the `IIIF` bundle are viewed and accessed only via the IIIF viewer. The IIIF viewer also provides 
  downloads and citation information for these assets. Any bitstreams in the default `ORIGINAL` bundle continue to 
  be accessed via the DSpace Item UI as before. 
* Existing DSpace records with images in the `ORIGINAL` bundle need to be modified by transferring assets to the 
  `IIIF` bundle. There may of course be ways to avoid this requirement, but those have not yet been explored.  
  
The screenshot below shows a record in which bitstreams in the `IIIF` bundle are rendered in the viewer. Meanwhile, a link to the
PDF version of the item is provided in the DSpace UI. Note that the PDF version can also be downloaded via the Mirador viewer.

![item image](images/sample_rec.png "Item")

Overall, the decision to rely on bundles rather than extending bitstream metadata with IIIF fields or using the new 
entity-relationship features of DSpace probably deserves analysis and discussion. My sense is that the bundle layer 
provides a clean and helpful separation for managing assets and creating views but there may be other ways to look
at this.


