---
layout: post
title:  "Elasticsearch"
date:   2015-01-10 13:21:50
author: white clover
categories: database
tags: database
---


## Installing Elasticsearch

```bash
curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip
unzip elasticsearch-$VERSION.zip
cd elasticsearch-$VERSION
```

## Installing Marvel

Marvel is a management and monitoring tool for Elasticsearch

Marvel is available as a plug-in. To download and install it, run this command in the
Elasticsearch directory:

    ./bin/plugin -i elasticsearch/marvel/latest

You probably don’t want Marvel to monitor your local cluster, so you can disable data
collection with this command:

    echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml

## Running Elasticsearch

Elasticsearch is now ready to run. You can start it up in the foreground with this:
    ./bin/elasticsearch

Add -d if you want to run it in the background as a daemon.
Test it out by opening another terminal window and running the following:
    curl 'http://localhost:9200/?pretty'

You should see a response like this:

```json
{
"status": 200 ,
"name": "Shrunken Bones" ,
"version": {
"number": "1.4.0" ,
"lucene_version" : "4.10"
},
"tagline": "You Know, for Search"
}
```
