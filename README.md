# mvcgi
BioThings Studio data plugin demo

A data plugin is defined by at least a manifest file and a parser.

manifest.json
=============

It contains different sections:

* "dumper": tells the studio where to get data from

  - "data_url" : a string, or a list of strings, containins URLs.
                 Currently supported protocols are: http, https, and ftp.
                 http/https must not be mixed with ftp (only one protocol supported)

  - "uncompress": true|false. tells the studio to try to uncompress downloaded data.
                  Currently supports zip, gz, bz2 and xz format.

  - "release": Optiona. Format: "module:function", refers to a function returning
               a string as a release.
               
               By default, if not provided, dumper will try to
               find release information from data_url itself. If http is used,
               Last-Modified or ETag HTTP headers will be considered to set a release.
               If ftp, MDTM will be used. Both will result in a timestamp-like format,
               such as YYYY-mm-dd. If a list of URLs is specified, this logic will be applied
               to the latest one in the list.
               
               If no such information is available, this field can be specified as a hook
               returning a release number (as a string). The signature takes "self" as the
               only argument and will represent the active dumper instance (function will
               actually be monkey-patched to the dumper instance). Any method/property is
               available to determine the release number, specifically, self.__class__.SRC_URLS
               containing the URLs defined abobe. self.client can be used to inspect these URLs
               further, and corresponds to either an ftplib.FTP or request.Session instance.

* "upload": tells the studio how to parse and upload data once it's been dumped locally

  - "parser" : Format "module:fuction", where function takes a data folder path as argument
               (ie. where the data was downloaded). While parsing data, it should return or yield
               dict with at least "_id" key present.
               
  - "on_duplicates" : what to do if duplicates are found (parser returns dict with same _id).
                      Can be either error|ignore|merge. (merge will merge dict together, see
                      biothings.utils.dataload.merge_struct function

* "__metadata__": describes the datasource, exposed through /metadata endpoint.

  - "license_url": points to license page
  
  - "license": license name if any
  
  - "url": datasource main web page

