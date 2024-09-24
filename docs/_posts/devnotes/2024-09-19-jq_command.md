---
title: Pretty Print and Search JSON in Your Terminal
date: 2024-09-19 20:22:11 +0200
tags:
   - unix
layout: tag

# hidden: true
---

<!-- # Pretty Print and Search JSON in Your Terminal -->
Using the command-line JSON processor --- `jq`
```
sudo apt update
sudo apt install jq
```


## View the pretty-printed JSON
Use `jq -C` and `less -r` to retain the colorful output.
```
$ cat captions_val2017.json | jq -C | less -r

  "info": {
    "description": "COCO 2017 Dataset",
    "url": "http://cocodataset.org",
    "version": "1.0",
    "year": 2017,
    "contributor": "COCO Consortium",
    "date_created": "2017/09/01"
  },
  "licenses": [
    {
      "url": "http://creativecommons.org/licenses/by-nc-sa/2.0/",
      "id": 1,
      "name": "Attribution-NonCommercial-ShareAlike License"
    },...
  ],
  "annotations": [
    {
      "image_id": 179765,
      "id": 38,
      "caption": "A black Honda motorcycle parked in front of a garage."
    },...
  ]

```

## Search 'annotations' by id

```
$ jq '.annotations[] | select(.id == 38)' captions_val2017.json
{
  "image_id": 179765,
  "id": 38,
  "caption": "A black Honda motorcycle parked in front of a garage."
}
```

##  Search 'annotations->captions' that contains a string

```
$ jq '.annotations[] | select(.caption|contains("lion"))' captions_val2017.json
{
  "image_id": 508602,
  "id": 656200,
  "caption": "Bird standing on car roof near covered pavilion."
}
{
  "image_id": 327890,
  "id": 59110,
  "caption": "A group of park benches under a pavilion."
}
{
  "image_id": 253002,
  "id": 249668,
  "caption": "A logo on a double decker bus with a lion and a sword."
}

```
