
The document attempts to describe implementation and ingest procedures as they relate to 
a proposed IIIF feature for DSpace 7 that is currently under review. Feedback and questions
are appreciated.

* https://github.com/DSpace/dspace-angular/pull/1079
* https://github.com/DSpace/DSpace/pull/3210


# Overview

This IIIF proposal relies on two core features of DSpace. 

* The `bundle` layer: The implementation uses bundles to identify bitstreams that will be added 
  to IIIF canvases (the `IIIF` bundle) or to related resource annotation lists (the `OtherContent` bundle). 
  The bundles also assure that bitstreams (and therefore canvases) appear in the proper order in the manifest.
  
* The `entity type` feature: This is new with DSpace 7 and is used to flag items as IIIF resources. 
  Items with a `dspace.entity.type` metadata field with a value of `IIIF` or `IIIFSearchable` adds the Mirador 
  viewer to the display. `IIIFSearchable` entities are initialized with the search results when the item has 
  been selected from a DSpace discovery result list.

To render an item as IIIF _both_ conditions are required.  The `dspace.entity.type` must be one of the IIIF types 
and bitstreams must be in the `IIIF` bundle.

Note that only the `dspace.entity.type` metadata field is required. There's no need to configure a `relationship-type` for IIIF entities.

These requirements have implications.

* The assets in the `IIIF` bundle are viewed and accessed only via the IIIF viewer. The IIIF viewer also provides 
  downloads and citation information for these assets. Any bitstreams in the default `ORIGINAL` bundle continue to 
  be accessed via the DSpace Item UI as before. 
  
* Existing DSpace records with images in the `ORIGINAL` bundle need to be modified by transferring assets to the 
  `IIIF` bundle. There may of course be ways to avoid this requirement, but those have not yet been explored.  
  
The screenshot below shows a record in which bitstreams in the `IIIF` bundle are rendered in the viewer. Meanwhile, a link to the
PDF version of the item is provided in the DSpace UI. (Note that the PDF version can also be downloaded via the Mirador viewer.)

Overall, the decision to rely on bundles probably deserves analysis and discussion. My sense is that the bundle layer
provides a clean and helpful separation for managing assets and creating views but there may be other ways to look
at this.

![item image](images/sample_rec.png "Item")



# Additional metadata using info.json

Bitstream objects lack 3 metadata fields
that are important for the IIIF implementation. The ability to add meaningful labels to bitstreams, such as "cover", "chapter",
"side", "back", and other more domain-specific labels is a basic requirement. The IIIF spec also recommends accurate height and width 
dimensions for canvases, based on the size of image to be rendered. These requirements could be addressed by adding metadata
fields to the bitstream. For now, that's beyond the scope of this initial pull request. I opted
instead to provide this metadata via a json file that resides in the `IIIF` bundle. 

The `info.json` file is a work-in-progress. Details are [provided in the PR description](https://github.com/DSpace/DSpace/pull/3210). 
Note that this `info.json` file is not required ... but is recommended.  Without the file the IIIF manifest will be 
rendered using default labels and height/width values.

One caveat is that there's currently no tooling for creating or modifying the `info.json` file. Such tooling would not be 
difficult to create in a separate PR, either for the `info.json` file itself or for additional bitstream metadata if we decide that's 
the best solution. 

One final note about metadata.  IIIF `ranges` are a nifty way to create navigation within a multi-image document. I've
included this in the `info.json sequences` property as an array. Alternatively, it may be possible to create manifest ranges 
by adding a 4th additional property for `ranges` to bitstream metadata. 

So to sum up, I added the `info.json` file as a way to provide metadata needed by IIIF without making additions to the existing DSpace data model.
I think those additions would be reasonable since they involve just a few additional properties for bitstreams. But meanwhile, you'll
need to provide an `info.json` file if you want to create IIIF views with custom properties.


# Additional Requirements

When you create a record that has images in the `IIIF` bundle and an IIIF `entity.type`, the Mirador viewer is embedded in the
Angular UI and the REST API is queried for the Item manifest. In IIIF lingo, these steps rely on the IIIF Presentation API. 
To render images the viewer needs to know how to request images from the image server (IIIF Image API). For searchable 
items, the REST endpoint also needs to know the location of the Solr index (IIIF Search API). This configuration information is provided
in new DSpace configuration properties.

## Image Server

Here's a quick description of how the cantaloupe image server is integrated with DSpace.  

The path to the server is provided in an IIIF configuration property in `local.cfg`.

`iiif.image.server = https://digitalcollections.willamette.edu/image-server/cantaloupe-4.1.7/iiif/2/`

This configuration property is used in the image service definition that gets embedded in the IIIF manifest
and returned by the DSpace REST API to Mirador:

```
service: {
  @context: "http://iiif.io/api/image/2/context.json",
  @id: "https://digitalcollections.willamette.edu/image-server/cantaloupe-4.1.7/iiif/2/181c0dd2-e661-43d8-8bbc-648705bfe490",
  profile: "http://iiif.io/api/image/2/level1.json",
}

```

Here is an example of the Image API request URL from Mirador to the cantaloupe server:

`https://digitalcollections.willamette.edu/image-server/cantaloupe-4.1.7/iiif/2/181c0dd2-e661-43d8-8bbc-648705bfe490/full/674,/0/default.jpg`

Cantaloupe is configured to use an `HttpSource` and a `ScriptLookupStrategy` defined in `delegates.rb`. It retrieves the image 
from DSpace and returns the processed image.

```
def httpsource_resource_info(options = {})
        identifier = context['identifier']
        identifier.gsub(/^(.{36})/, "http://dspace-test.willamette.edu:8080/server/api/core/bitstreams/\\1/content")
  end
  ```

## Solr Search Index

The IIIF Search API uses a Solr index.  Details are [described in the pull request](https://github.com/DSpace/DSpace/pull/3210).

The location of the Solr service is defined in `local.cfg`. The REST backend will use this when responding to IIIF Search API requests
sent by the Mirador viewer.

`iiif.solr.search.url = http://localhost:8983/solr/word_highlighting`



# Import Process

Because I have a large data migration in mind, I am using the Simple Archive Format to batch import items to DSpace. Here's 
an abbreviated contents file that shows files imported into the bundles described above. 

```
199701.pdf
alto_1.xml  bundle:OtherContent
alto_2.xml  bundle:OtherContent
001.jp2 bundle:IIIF
002.jp2	bundle:IIIF
info.json	bundle:IIIF
```

For the Search API and Solr index, I am currently populating Solr with separate Python load process.  I think there is a 
way to integrate this more closely with the DSpace ingest process by requiring inclusion of ALTO files in the `OtherContent` 
bundle. That would end the need for an entirely separate load process. Instead, a process running on the Solr server 
could use the DSpace IIIF API to retrieve manifests and ALTO files via the `seeAlso` AnnotationList and use this information to 
maintain the Solr index. 
