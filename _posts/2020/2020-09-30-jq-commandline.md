---
layout: post	
title: jq Command Line	
published: true	
categories: [Json]	
date: 2020-09-30 09:00
excerpt: | 
     export file list as Json via jq
     
---

 
# ./jq
jq is a lightweight and flexible command-line JSON processor.

### Json to CSV
```bash
$ jq -r '.data[] | [.media.id, .media.name, .url] | @csv' ytblist.json > fileout2.csv
```

# File lists as Json



```
#!/bin/bash

while getopts u:a:f: flag
do
    case "${flag}" in
        u) username=${OPTARG};;
        h) helper=${OPTARG};;
        f) folder=${OPTARG};;
    esac
done

find $folder -ls | jq -nR '
  # Return an object with useful information
  def gather:
    [splits(" +")] as $in
    | { pathname: $in[-1], entrytype: $in[2][0:1], size: ($in[6] | tonumber) };

  reduce (inputs | gather) as $entry ({};
      ($entry.pathname | split("/") ) as $names
      | if ($entry|.entrytype == "-") then
           ($names[0:-1] + ["items"]) as $p
           | setpath($p; getpath($p) + [{name: $names[-1], size: $entry.size}])
        else . end) '
```

parameters :
 - `-f` foldername start folder name
