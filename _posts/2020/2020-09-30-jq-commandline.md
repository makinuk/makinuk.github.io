---		
layout: post		
title: jq command line		
published: true
categories: [json]		
date: 2020-09-30 16:24:35
excerpt: | 
        json, jq, csv

        JQ Command Line
---


# ./jq
jq is a lightweight and flexible command-line JSON processor.

### Json to CSV
```bash
$ jq -r '.data[] | [.media.id, .media.name, .url] | @csv' ytblist.json > fileout2.csv
```
