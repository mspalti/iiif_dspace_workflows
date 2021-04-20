# Overview

This IIIF proposal relies on two basic features of DSpace. 

* The `bundle` layer. The implementation uses bundles to identify bitstreams that will be added 
  to IIIF canvases (the `IIIF` bundle) or to related resource annotation lists (the `OtherContent` bundle). 
  The bundles also assures that bitstreams (and therefore canvases) appear in the proper order.
* The `entity.type` feature. This is new with DSpace 7 and is used to flag items as IIIF resources. 
  Items with an `entity.type` of `IIIF` or `IIIFSearchable` incorporate the Mirador viewer into the  
  display and initialize the viewer with the manifest URL of the item. Also, `IIIFSearchable` entities are  
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


# Additional metadata using info.json

Existing DSpace bundles and bitstreams do not address every requirement. In particular, bitstream objects lack 3 metadata fields
that are important for the IIIF implementation. The ability to add meaningful labels to bitstreams, such as "cover", "chapter",
"side", "back", and other more domain-specific labels is a basic requirement. The IIIF spec also recommends accurate height and width 
dimensions for canvases, based on the size of image to be rendered. These requirements could be accommodated by adding metadata
fields to the bitstream. For now, that is seems beyond the scope of the initial pull request and I opted
instead to provide this metadata via a json file that resides in the `IIIF` bundle. 

The `info.json` file is a work-in-progress. Details are provided in the PR description. This `info.json` file is not required but is 
recommended.  Without the file the IIIF manifest will be rendered using default labels and height/width values.

One caveat is that there is currently no tooling for creating or modifying the `info.json` file. This would not be difficult to
create, whether the for the `info.json` file itself or for additional bitstream metadata if we decide that's the best solution. 

One final note about IIIF metadata.  IIIF `ranges` are a nifty way to add supplemental navigation to a multi-image document. I've
included this in the `info.json sequences` property as an array. It may be possible to create ranges by adding a range property
to bitstream metadata. 





# Requirements


