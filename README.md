# Web Archive APIs workshop

## Scripts and such for workshops on web archive APIs ##

Scripts and descriptions of what they do, culled from daily practice in my daily practices of work and fun and too-much-time-on-my-hands. Credit to the Archive-It and Internet Archive folks that sometimes help me with the harder parts I can't find on Stack Overflow. Portions delivered in an SAA Web Archiving Section webinar workshop, the WARCshop at Penn State University (both in Spring 2017) and at IPres in Kyoto, Japan in Fall 2017. APIs are mostly Intnernet Archive / Archive-It ones because, duh, that's where I work.

See also https://github.com/vinaygoel/ars-workshop because all this stuff will eventually live together in a harmonious fashion.

## API  Examples ##

## Wayback Machine CDX & Availability APIs ##

The Wayback CDX Server API has good documentation with examples, https://github.com/internetarchive/wayback/blob/master/wayback-cdx-server/README.md

The Availability API is useful to looking up if something has been archived. If a user wants to query it for a set of URLs, they can construct an input file like so:

[
{"url":"http://www.example.com"},
{"url":"http://www.wazoo.com"},
{"url":"http://www.badtaxidermy.com"}
]

and store it in a text file like "wayback_lookup.txt" and run this command

`curl -X POST -d @wayback_lookup.txt http://archive.org/wayback/available --header  "Wayback-Api-Version:2" --header "Content-Type:application/json"`

and get a response in JSON.

### Get a list of all hosts on a certain domain ###

`domain=np; curl "http://wwwb-dedup.us.archive.org:8083/cdx/search?url=$domain&matchType=domain&fl=urlkey,length&from=1996&to=2015" | cut -f1 -d '/' | uniq -c`

Here is a sample python notebook for playing with these APIs and charting results: https://gist.github.com/JaimieMurdock/3e9235128024afa680a7e6fdd0b0b746 

## Host Details API ##

The new Wayback Machine search includes profile information for specific hosts, for example, https://web.archive.org/details/example.com and this visualization is served via an API, https://web-beta.archive.org/__wb/search/metadata?q=host:example.com which can be queried at multiple levels, including TLD, for instance, https://web.archive.org/__wb/search/metadata?q=tld:pizza (and https://web.archive.org/details/pizza) and these stats can be parsed and analyzed.

### Sum the total new captures by mime for a host ###

`curl https://web.archive.org/__wb/search/metadata?q=host:unesco.org | jq '.new_urls | .[] | .["application/pdf"]' | paste -sd+ - | bc`

### Sum the total size of captures by mime for a host and print in sizes ###

`curl https://web.archive.org/__wb/search/metadata?q=tld:collection-eot2016-waybacksummary | jq '.urls_total_compressed_size | .[] | .["application/pdf"]' | paste -sd+ - | bc | awk '{ myfloat = "%.10f"; byte = sprintf(myfloat, $1 /1024) ; print byte " kB";byte = sprintf(myfloat, $1 /1024**2) ; print byte " MB"; byte = sprintf(myfloat, $1 /1024**3) ; print byte " GB"; byte = sprintf(myfloat, $1 /1024**4) ; print byte " TB" }'`

## Media and Hash APIs ##

The new Wayback Search also allows the ability to search for specific media types and via SHA1 checksum.

### Lookup a specific checksum in the Wayback Machine ###

`http://wbgrp-search0-a.us.archive.org:8090/search/waybacksearch?q=digest:sha1:XH6Z6FZHL7T4OIJL3FQX43MFTEWDQ6PX`

### Feed in a list of checksums and get their associated URLs ###

Make a text file of SHA1 checksums with one checksum on each line; in this example we have called the file 'hash_example2.txt'

`cat hash_example2.txt | while read url; do curl -L -s http://wbgrp-search0-a.us.archive.org:8090/search/waybacksearch?q=digest:sha1:${url} | jq -r '.["hits"] | .[] | .["content_digest"] + " " + .["url"]'; done`

### Get puppy media ###

`curl http://ia803501.us.archive.org/api/v1/mediascoresearch?q=puppy | jq -r '.[].link'`

## Full Text Search APIs ##

Archive-It does full-text indexing of  its entire corpus (well over 1PB total). More on Archive-It FTS is here

and the parameters for using this API via Open Search are at https://webarchive.jira.com/wiki/spaces/ARIH/pages/13468101/OpenSearch+Overview

### View full text search results in JSON via the API ###

`http://wbgrp-svc060.us.archive.org:8091/ait/opensearch?&q=ukraine&i=4399&fmt=json`

## Archive-It Partner Data API ##
This API provides access to collection-specific data within Archive-It (but a similar API could provide such collection-related information held in other systems).

### Get all of seeds in a collection ###

`curl 'https://partner.archive-it.org/api/seed?&collection=2950&format=json' | jq '.[] | .canonical_url' | cut -d '"' -f2`

Using the collection identifier 2950 (for the Occupy Movement Collection, https://archive-it.org/collections/2950) it parses out the seed URLs from the JSON response. Change the colleciton identifier to get seed lists of other collections (collections IDs can be found by searching in Archive-It)

### Get collection description ###

`curl https://partner.archive-it.org/api/collection/7994 | jq '.metadata.Description | .[].value'`

Similar to the above, this query returns the curator-added description field for a specific collection, as can be seen in the public interface at 

## R&D APIs ##

### GifCities term frequency count ###

Our GifCities project (https://gifcities.org/) is built on top of an open API. This is an example of a collection extracted from a web archive collection and recreated in a new portal with additional information. The API powers the search and replay but can also be used for some funtime poking around, such as Kittens vs. Puppies!

`curl https://gifcities.archive.org/api/v1/gifsearch?q=kitten | jq -r .[].url_text | tr -s ' ' | tr ' ' '\n' | tr '[A-Z]' '[a-z]' | sort | uniq -c | sort -nr | head -20`

`curl https://gifcities.archive.org/api/v1/gifsearch?q=puppy | jq -r .[].url_text | tr -s ' ' | tr ' ' '\n' | tr '[A-Z]' '[a-z]' | sort | uniq -c | sort -nr | head -20`

## WAT API examples ## 

A WAT file is a derivative file created from a WARC but more lightweight and containing only key data and not the full resource information (such as page text). More on WATs at: https://webarchive.jira.com/wiki/spaces/ARS/pages/90997503/WAT+Overview+and+Technical+Details 

### Get all anchor text from a resource ###

`curl http://vinay-dev.us.archive.org:8080/eot2016/20161215135244/https://www.justice.gov/publications/usdoj-resources-publications-alphabetical-list | jq '.["Envelope"]["Payload-Metadata"]["HTTP-Response-Metadata"]["HTML-Metadata"]["Links"][]["text"]' | grep -v null | less`

### Get page title from seed ###

`curl http://vinay-dev.us.archive.org:8080/eot2016/20161215135244/https://www.justice.gov/publications/usdoj-resources-publications-alphabetical-list | jq -r '.["Envelope"]["Payload-Metadata"]["HTTP-Response-Metadata"]["HTML-Metadata"]["Head"]["Title"]'`

### Extract titles from a list of seeds ###

`cat eot_pr.txt | while read url; do curl -L -s http://vinay-dev.us.archive.org:8080/eot2016/2016/${url} | jq '.["Envelope"] | .["Payload-Metadata"] | .["HTTP-Response-Metadata"] | .["HTML-Metadata"] | .["Head"] | .["Title"]' 2> /dev/null ; done`

### Anchor  terms (with frequencies) in links from these pages ###

`cat eot_pr.txt | while read url; do curl -L -s http://vinay-dev.us.archive.org:8080/eot2016/2/${url} | jq '.["Envelope"] | .["Payload-Metadata"] | .["HTTP-Response-Metadata"] | .["HTML-Metadata"] | .["Links"][] | .["text"]' 2> /dev/null | sed "s@\"@@g"  ;done | tr -s ' ' | tr ' ' '\n' | tr '[A-Z]' '[a-z]' | sort | uniq -c | sort -nr | grep -v null | head -50`
