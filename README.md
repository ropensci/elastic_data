elastic datasets
================

This is a collection of smallish datasets to use for playing with Elasticsearch.

You can only fit so much data in an R package. The R client for Elasticsearch we maintain
[elastic](https://github.com/ropensci/elastic) comes with some data, but of course 
it's nice to have more, so here it is.

See also [nodbi](https://github.com/ropensci/nodbi) for working with Elasticsearch from R.

## Datasets

* `plos_everything.json`
* `plos_introductions.json`
* `plos_data.json`
* `geonames_elastic_bulk.zip` - too big for gitub, [at dropbox](https://www.dropbox.com/s/8vcrt3g2d0pfw8l/geonames_elastic_bulk.zip?dl=0)
* `gbif_data.json`
* `gbif_geo.json`
* `gbif_geopoint.json`
* `gbif_geoshape.json`
* `gbif_geosmall.json`
* `shakespeare_data.json`
* `omdb.json`

## Loading into ES

These datasets are formatted to be ready for bulk loading into Elasticsearch
via the [bulk API](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-bulk.html)

### geonames

`geonames_elastic_bulk.zip` is about 70 `.json` files in Elasticsearch bulk format. It was prepared from the [Geonames](http://www.geonames.org/) database at [http://download.geonames.org/export/dump/](http://download.geonames.org/export/dump/). The original data from Geonames was licensed under a Creative Commons Attribution 3.0 License, see [http://creativecommons.org/licenses/by/3.0/](http://creativecommons.org/licenses/by/3.0/).

To load the geonames data into Elasticsearch, do as you wish, but e.g., in R you could do:

First, create the index and set the `geo_shape` mapping

```r
body <- '{
 "mappings": {
   "record": {
     "properties": {
         "location" : {"type" : "geo_shape"}
      }
   }
 }
}'
index_create(index='geonames', body=body)
```

should return

```r
#> $acknowledged
#> [1] TRUE
```

Note: the index type is `record`, and the index name is `geonames`. The index and index type were set in the json files.

Then use a for loop to load in each file. AKAIK there is a limit on the file size you can load in (let me know if there's a way to get around it), so that's why theres a bunch of json files instead of one big file.

```r
devtools::install_github("ropensci/elastic")
library("elastic")
files <- list.files("path/to/unzipped/files")
for(i in seq_along(files)){
  invisible(
    docs_bulk(
      sprintf("path/geonames%s.json", files[i])
    )
  )
}
```

The `docs_bulk()` function uses the `/_bulk` endpoint to `POST` data to an index called `geonames` in your ES server. The output of the bulk load call prints info, that's why we use `invisible()` so you don't get thousands of lines printed.

Check that it worked:

```r
Search("geonames")$hits$total
#> [1] 6646030
```

You should have ~ 6.6 million records
