---
title: other-cmd-tips
language: ja
date: 2018-09-27 15:18:46
tags:
---


# Aerospike aql

### default connection with json format output
`aql -ojson`

### show namespaces
`show namespaces`

> output:

```
[
    [
        {
          "namespaces": "rssp"
        },
        {
          "namespaces": "sync"
        },
        {
            "node": "127.0.0.1:3000"
        }
    ],
    [
        {
          "namespaces": "rssp"
        },
        {
          "namespaces": "sync"
        },
        {
            "node": "10.146.0.7:3000"
        }
    ],
    [
        {
          "Status": 0
        }
    ]
]

```

### show sets
`show sets`

### select
`select * from domain.set where PK=<key value>`
