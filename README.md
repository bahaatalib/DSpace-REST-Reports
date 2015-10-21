This project is intended as an optional add-on to DSpace to provide Quality Control Reporting for Collection Managers.

These reports utilize the DSpace REST API to provide a Collection Manager with

* an overview of their collections
* a tool to query metadata for consistency

When deploying the DSpace REST API, and institution may choose to make the API publicly accessible or to restrict access to the API.
If these reports are deployed in a protected manner, the reporting tools can be configured to bypass DSpace authorization when reporting on collections and items.

## Sample Screen Shots
The wiki for this project contains screenshots that illustrate the capabilities of these reports.

[Report Screenshots](https://github.com/DSpace-Labs/DSpace-REST-Reports/wiki)

## Pre-requisites
* Determine the access that you will grant to your REST api since these reporting tools can be configured to run without authentication.
* These tools can use *sorttable.js* which can be found at http://www.kryogenix.org/code/browser/sorttable/
* If you enable sorttable, uncomment sorttable.makeSortable(...) in the js files 
* Install this code into dspace/modules/xmlui/src/main/webapp/static/rest (or to a path of your choosing)
* Update the dspace/config/modules/rest.cfg 

## rest.cfg: Configure the REST Reporting features

#### Enable/Disable object authorization for report calls

```
# Enable/disable authorization for the reporting tools.
# By default, the DSpace REST API will only return communities/collections/items that are accessible to a particular user.
# If the REST API has been deployed in a protected manner, the reporting tools can be configured to bypass authorization checks.
# This will allow all items/collections/communities to be returned to the report user.
# Set the rest-reporting-authenticate option to false to bypass authorization
rest-reporting-authenticate = false
```

#### Configure the report pages that can be requested by name

```
# Configure the report pages that can be requested by name
# Create a map of named reports that are available to a report tool user
# Each map entry should be prefixed with rest-report-url 
#   The map key is a name for a report
#   The map value is a URL to a report page
# A list of available reports will be available with the call /rest/reports.
# If a request is sent to /rest/reports/[report key], the request will be re-directed to the specified URL
# 
# This project currently contains 2 sample reports.  Eventually, additional reports could be introduced through this mechanism.
rest-report-url.collections = ../static/rest/index.html
rest-report-url.item-query = ../static/rest/query.html
```

##### database specific way to format a regex SQL clause #####
```
# The REST Report Tools may pass a regular expression test to the database.  
# The following configuration setting will construct a SQL regular expression test appropriate to your database engine
rest-regex-clause = text_value ~ ?
```

#### Configure the list of filters that are useful to your repository managers.
```
# A filter contains a set of tests that will be applied to an item to determine its inclusion in a particular report.
# Private items and withdrawn items are frequently excluded from DSpace reports.
# Additional filters can be configured to examine other item properties.
# For instance, items containing an image bitstream often have different requirements from a item containing a PDF.
# The DSpace REST reports come with a variety of filters that examine item properties, item bitstream properties, 
# and item authorization policies.  The existing filters can be used as an example to construct institution specific filters
# that will test conformity to a set of institutional policies.
# plugin.sequence.org.dspace.rest.filter points to a list of classes that contain available filters.  
# Each class must implement the ItemFilterList interface.
#   ItemFilterDefs:     Filters that examine simple item and bitstream type properties
#   ItemFilterDefsMisc: Filters that examine bitstream mime types and dependencies between bitstreams
#   ItemFilterDefsMeta: Filters that examine metadata properties
#   ItemFilterDefsPerm: Filters that examine item and bitstream authorization policies
plugin.sequence.org.dspace.rest.filter.ItemFilterList = \
        org.dspace.rest.filter.ItemFilterDefs,\
        org.dspace.rest.filter.ItemFilterDefsMisc,\
        org.dspace.rest.filter.ItemFilterDefsPerm

#     org.dspace.rest.filter.ItemFilterDefsMeta,\
```
#### Configure settings used by REST report filters
_Only define the settings used by the filters that you have enabled_
```
# Define the set of supported bitstream bundle names for your repository as a comma separated list
rest-report-supp-bundles = ORIGINAL,THUMBNAIL,TEXT,LICENSE

# Define the bitstream mime types that will be interpreted as a "document".
# Generally, a "document" should be expected to be searchable and to have a TEXT bitstream.  An institution may expect document types to
# have a thumbnail.
rest-report-mime-document = text/plain,application/pdf,text/html,application/msword,text/xml,application/vnd.openxmlformats-officedocument.wordprocessingml.document,application/vnd.ms-powerpoint,application/vnd.openxmlformats-officedocument.presentationml.presentation,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

# Define the standard/preferred bitstream mime types for document items for your repository
# This setting allows the reporting tools to report on "supported" and "unsupported" document types.
rest-report-mime-document-supported = application/pdf

# Define the standard/preferred bitstream mime types for image items for your repository
# This setting allows the reporting tools to report on "supported" and "unsupported" image types.
rest-report-mime-document-image = image/jpeg,image/jp2

# Minimum size for a supported PDF in the repository
# PDF bitstreams smaller than this size will be highlighted in a report.  
# PDF files smaller than this size are potentially corrupt.
rest-report-pdf-min-size = 20000

# Maximum size for a typical PDF in the repository
# PDF bitstreams larger than this size will be highlighted in a report.  
# PDF files larger than this size may be slow to retrieve.
rest-report-pdf-max-size = 25000000

# Minimum size for a thumbnail - could indicate a corrupted original
# Thumbnail bitstreams smaller than this size will be highlighted in a report.  
# Thumbnail files smaller than this size are potentially corrupt.
rest-report-thumbnail-min-size = 400

# Bitstream descriptor to identify generated thumbnails
# The ImageMagick Thumbnail generator tags the thumbnails it has created with a standard description.
# This description identifies thumbnails that can safely be re-generated.
rest-report-gen-thumbnail-desc = Generated Thumbnail
```

#### Metadata Filtering
_If org.dspace.rest.filter.ItemFilterDefsMeta is enabled, the following regular expressions will be used_

```
# Used by org.dspace.rest.filter.ItemFilterDefsMeta
# This class filters items based on metadata properties.
# These filters are useful for filtering a small set of items.  These filters will be inefficient as a query tool.

# regex to detect compound subjects - detect subject terms that should be split into individual terms
rest-report-regex-compound-subject = .*;.*

# regex to detect compound authors - detect author/creator names taht should be split into individual fields
rest-report-regex-compound-author = .* and .*

# regex to detect unbreaking metadata - detect long unbreaking text that may not render properly on a page
rest-report-regex-unbreaking = ^.*[^ ]{50,50}.*$

# regex to detect url in description - detect description fields that contain URL's
rest-report-regex-url = ^.*(http://|https://|mailto:).*$

# regex to identify full text content from the provenance field
# This test has been used to identfiy "full-text" content when harvesting DSpace content by Summon
rest-report-regex-fulltext = ^.*No\\. of bitstreams(.|\\r|\\n|\\r\\n)*\\.(PDF|pdf|DOC|doc|PPT|ppt|DOCX|docx|PPTX|pptx).*$

# regex to identify very long metadata fields that may be slow to render
rest-report-regex-long = ^[\\s\\S]{6000,}$

# regex to identify partial XML entities within a description field (a frequent problem found in ProQuest ETD's)
rest-report-regex-xml-entity = ^.*&#.*$

# regex to identify non ascii characters in metadata
rest-report-regex-non-ascii = ^.*[^\\p{ASCII}].*$

```

