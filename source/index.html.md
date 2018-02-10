---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='mailto:api-support@intact.design?subject=Sign me up for an API key!'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction to the Intact API

Intact is a web API for physics. At the moment, we support structural mechanics (think bridges and load-bearing mechanical parts). Our mission is to bring physics to any designer, engineer, or maker on the internet. To do that, we need your feedback! Always feel free to <a href="mailto:api-support@intact.design">email us</a>.

You can use our API to create simulation scenarios.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: Token token=abc1234"
```

> Make sure to replace `abc1234` with your API key.

Intact uses API keys to allow access to the API. You can register a new API key by [sending us a message](mailto:api-support@intact.design?subject=Sign me up for an API key!).

Intact expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Token token=abc1234`

<aside class="notice">
You must replace <code>abc1234</code> with your personal API key.
</aside>

# Data Model

Our data model for geometric data consists of:

- Model families: designs that are related together as a design study, which contains references to many...
- Assemblies: an assembly is a composite geometric model, which have many simulations and are made up of one or more instances of...
- Components: a single geometric model.

For now we're focusing on delivering value on a single-assembly level (i.e. we don't really leverage the abstraction of a model family yet). Eventually we will develop sampling procedures across an entire design study, but for now we're focusing on delivering seamless interoperability and full automation of physics.

# Models

#

## List model families

```shell
curl "https://intact.design/api/models"
  -H "Authorization: Token token=abc1234"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "title": "Design 1",
    "metadata": {...},
    "unit": "m",
    "created_at": "2018-02-08T00:37:10.938Z",
    "models": {...}
  },
  {
    ....
  }
]
```

This endpoint retrieves all model families for your user.

# Simulations

### Create design study

```shell
curl "https://intact.design/api/models/{id}/model.json"
  -d {
      "loads": [
      {
        "force": {
          "direction": [
            0,
            0,
            1
          ],
          "magnitude": 100
        },
        "selector": {
          "cells": [
            [0, 1, 2]
          ],
          "positions": [
            [0, 1, 2],
            [2, 3, 4],
            [4, 5, 6]
          ]
        },
      }
    ],
    "restraints": [
      {
        "selector": {
          "cells": [
            [0, 1, 2]
          ],
          "positions": [
            [0, -1, 2],
            [-2, 3, -4],
            [-4, -5, 6]
          ]
        }
      }
    ],
    "units": "m",
    "model": "https://s3.aws.com/path/to/model.stl",
    "material": "ABS"
  }
  -X POST
  -H "Authorization: Token token=abc1234"
  -H "Content-Type: application/json"
```

### Body Parameters

Parameter | Format | Description
--------- | ------- | -----------
loads | Array of objects containing `force` and `selector` keys | Loads to apply to model
restraints | Array of objects containing `selector` keys | Restraints to apply to model
selector | Object containing `cells` and `positions` keys | Represents mesh on which to apply boundary condition
units | One of "m", "mm" , "cm", "in", "ft" | Units that the model (and force) is measured in.
material | Name of material (same format as in our interface) | Material name for the material properties
model | URL to a STL, PLY, or OBJ file | model file

This method returns an id and a tokenized URL that you can use to find the simulation status as follows.

## Find simulation status

```shell
curl "https://intact.design/api/simulations/TOKEN/status"
  -H "Authorization: Token token=abc1234"
```

> The above command returns JSON structured like this:

```json
{
  "status": "Completed|Failed|Queued"
}
```

This endpoint retrieves simulation status for a submitted simulation.
