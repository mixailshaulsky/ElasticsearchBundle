# What is it?

This is the fork [ONGR Elasticsearch Bundle][1] with ability to customize [Index similarity][2] 

## Step 1: Configure bundle

```yaml

# config/packages/ongr_elasticsearch.yaml
ongr_elasticsearch:
    analysis:
        filter:
            edge_ngram_filter: #-> your custom filter name to use in the analyzer below
                type: edge_ngram 
                min_gram: 1
                max_gram: 20
        analyzer:
            eNgramAnalyzer: #-> analyzer name to use in the document field
                type: custom
                tokenizer: standard
                filter:
                    - lowercase
                    - edge_ngram_filter #that's the filter defined earlier
        similarity:
            scripted_tfidf:
              type: scripted
              script:
                source: "double tf = Math.sqrt(doc.freq); double idf = 1.0; double norm = 1/Math.sqrt(doc.length); return query.boost * tf * idf * norm;"
    indexes:
        App\Document\Product:
            hosts: [elasticsearch:9200] # optional, the default is 127.0.0.1:9200
```

## Step 2: Configure your custom similarity for `Document` field

```php
// src/Document/Product.php

namespace App\Document;

use ONGR\ElasticsearchBundle\Annotation as ES;

/**
 * //alias and default parameters in the annotation are optional. 
 * @ES\Index(alias="products", default=true)
 */
class Product
{
    /**
     * @ES\Id()
     */
    public $id;

    /**
     * @ES\Property(type="text", similarity="scripted_tfidf")
     */
    public $title;

    /**
     * @ES\Property(type="float")
     */
    public $price;
}
```

[1]: https://github.com/ongr-io/ElasticsearchBundle
[2]: https://www.elastic.co/guide/en/elasticsearch/reference/6.8/index-modules-similarity.html
