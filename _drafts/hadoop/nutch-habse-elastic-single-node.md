---
layout: post
title: "nutch-hbase-elastic-single-node"
date: $(DATE)
categories: [Hadoop]
---

Info
----

This guide sets up a **non-clustered** Nutch crawler, which stores its data via HBase. We will not learn how to setup Hadoop et al., but just the bare minimum to crawl and index websites on a single machine.

Terms
-----

* **Nutch** - the crawler (fetches and parses websites)
* **HBase** - filesystem storage for Nutch (Hadoop component, basically)
* **Gora** - filesystem abstraction, used by Nutch (HBase is one of the possible implementations)
* **ElasticSearch** - index/search engine, searching on data created by Nutch (does not use HBase, but its down data structure and storage)

Requirements
------------

* OpenJDK 7 & ant
* [Nutch 2.3 RC](https://github.com/apache/nutch/archive/release-2.3.tar.gz) (yes, you **need** 2.3, 2.2 will not work)
* [HBase 0.94.26](http://mirror.cc.columbia.edu/pub/software/apache/hbase/hbase-0.94.26/hbase-0.94.26.tar.gz) (HBase 0.98 [won't work](https://issues.apache.org/jira/browse/GORA-304))
* [ElasticSearch 1.4.2](https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.deb)

Install OpenJDK, ant and ElasticSearch via your repository manager of choice (ES can be installed by using the ``.deb`` linked above, if you need).

Extract Nutch and HBase somewhere. From now on, we will refer to the Nutch root directory by ``$NUTCH_ROOT`` and the HBase root by ``$HBASE_ROOT``.

Setting up HBase
----------------

1. edit ``$HBASE_ROOT/conf/hbase-site.xml`` and add
  
  ```xml
  <configuration>
    <property>
      <name>hbase.rootdir</name>
      <value>file:///full/path/to/where/the/data/should/be/stored</value>
    </property>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>false</value>
    </property>
  </configuration>
  ```

2. edit ``$HBASE_ROOT/conf/hbase-env.sh`` and enable JAVA_HOME and set it to the proper path:

  ```diff
  -# export JAVA_HOME=/usr/java/jdk1.6.0/
  +export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
  ```

  This step might seem redundant, but even with ``JAVA_HOME`` being set in my shell, HBase just didn't recognize it.

3. kick off HBase:

  ```bash
  $HBASE_ROOT/bin/start-hbase.sh
  ```

Setting up Nutch
----------------

1. enable the HBase dependency in ``$NUTCH_ROOT/ivy/ivy.xml`` by uncommenting the line

  ```xml
  <dependency org="org.apache.gora" name="gora-hbase" rev="0.5" conf="*->default" />
  ```

2. configure the HBase adapter by editing the `$NUTCH_ROOT/conf/gora.properties`:

  ```diff
  -#gora.datastore.default=org.apache.gora.mock.store.MockDataStore
  +gora.datastore.default=org.apache.gora.hbase.store.HBaseStore
  ```

3. build Nutch

  ```shell
  $ cd $NUTCH_ROOT
  $ ant clean
  $ ant runtime
  ```

  This *can take a while* and creates ``$NUTCH_ROOT/runtime/local``.

4. configure Nutch by editing ``$NUTCH_ROOT/runtime/local/conf/nutch-site.xml``:

  ```xml
  <configuration>
    <property>
      <name>http.agent.name</name>
      <value>mycrawlername</value> <!-- this can be changed to something more sane if you like -->
    </property>
    <property>
      <name>http.robots.agents</name>
      <value>mycrawlername</value> <!-- this is the robot name we're looking for in robots.txt files -->
    </property>
    <property>
      <name>storage.data.store.class</name>
      <value>org.apache.gora.hbase.store.HBaseStore</value>
    </property>
    <property>
      <name>plugin.includes</name>
      <!-- do **NOT** enable the parse-html plugin, if you want proper HTML parsing. Use something like parse-tika! -->
      <value>protocol-httpclient|urlfilter-regex|parse-(text|tika|js)|index-(basic|anchor)|query-(basic|site|url)|response-(json|xml)|summary-basic|scoring-opic|urlnormalizer-(pass|regex|basic)|indexer-elastic</value>
    </property>
    <property>
      <name>db.ignore.external.links</name>
      <value>true</value> <!-- do not leave the seeded domains (optional) -->
    </property>
    <property>
      <name>elastic.host</name>
      <value>localhost</value> <!-- where is ElasticSearch listening -->
    </property>
  </configuration>
  ```

5. configure HBase integration by editing ``$NUTCH_ROOT/runtime/local/conf/hbase-site.xml``:

  ```xml
  <configuration>
    <property>
      <name>hbase.rootdir</name>
      <value>file:///full/path/to/where/the/data/should/be/stored</value> <!-- same path as you've given for HBase above -->
    </property>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>false</value>
    </property>
  </configuration>
  ```

That's it. Everything is now setup to crawl websites.

Adding new Domains to crawl with Nutch
--------------------------------------

1. create an empty directory. Add a textfile containing a list of seed URLs.

  ```bash
  $ mkdir seed
  $ echo "https://www.website.com" >> seed/urls.txt
  $ echo "https://www.another.com" >> seed/urls.txt
  $ echo "https://www.example.com" >> seed/urls.txt
  ```
  
2. inject them into Nutch by giving a file URL (!)

  ```bash
  $ $NUTCH_ROOT/runtime/local/bin/nutch inject file:///path/to/seed/
  ```

Actual Crawling Procedure
-------------------------

1. Generate a new set of URLs to fetch. This is is based on both the injected URLs as well as outdated URLs in the Nutch crawl db.

  ```bash
  $ $NUTCH_ROOT/runtime/local/bin/nutch generate -topN 10
  ```

  The above command will create job batches for 10 URLs.

2. Fetch the URLs. We are not clustering, so we can simply fetch all batches:

  ```bash
  $ $NUTCH_ROOT/runtime/local/bin/nutch fetch -all
  ```

3. Now we parse all fetched pages:

  ```bash
  $ $NUTCH_ROOT/runtime/local/bin/nutch parse -all
  ```

4. Last step: Update Nutch's internal database:

  ```bash
  $ $NUTCH_ROOT/runtime/local/bin/nutch updatedb -all
  ```

On the first run, this will only crawl the injected URLs. The procedure above is supposed to be repeated regulargy to keep the index up to date.

Putting Documents into ElasticSearch
------------------------------------

Easy peasy:

```bash
$ $NUTCH_ROOT/runtime/local/bin/nutch index -all
```

Query for Documents
-------------------

The usual ElasticSearch way:

```bash
$ curl -X GET "http://localhost:9200/_search?query=my%20term"
```