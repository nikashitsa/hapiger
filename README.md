# HapiGER

<img src="./assets/hapiger300x200.png" align="right" alt="HapiGER logo" />

Providing good recommendations can create greater user engagement and directly provide value by recommending items the customer might additionally like. However, many applications don't provide recommendations to users because of the difficulty in implementing a custom engine or the pain of using an off-the-shelf engine.

**HapiGER** is a recommendations service that uses the [Good Enough Recommendations (**GER**)](https://www.npmjs.com/package/ger), a scalable, simple recommendations engine, and the [Hapi.js](http://hapijs.org) framework. It has been developed to be easy to integrate, easy to use and very scalable.

[Project Site](http://www.hapiger.com)

## Quick Start Guide

<br/>
***
#### Install HapiGER

Install with `npm`

```bash
npm install -g hapiger
```

<br/>
***

#### Start HapiGER

By default it will start with an in-memory event store (events are not persisted)

```bash
hapiger
```

*There are also PostgreSQL and RethinkDB event stores for persistence and scaling*

<br/>
***

#### Create a Namespace

A Namespace is a bucket where all the events are put:

```bash
curl -X POST 'http://localhost:3456/namespaces' -d'{
    "namespace": "movies"
  }'
```

<br/>
***

#### Create some Events

An event occurs when a person actions something, e.g. `Alice` `view`s `Harry Potter`:

```bash
curl -X POST 'http://localhost:3456/events' -d '{
    "events": [
    {
      "namespace": "movies"
      "person":    "Alice",
      "action":    "view",
      "thing":     "Harry Potter"
    }
  ]
}'
```

Then, `Bob` also `view`s `Harry Potter` (now `Bob` has similar viewing habits to `Alice`)

```bash
curl -X POST 'http://localhost:3456/events' -d '{
    "events": [
    {
      "namespace": "movies"
      "person":    "Bob",
      "action":    "view",
      "thing":     "Harry Potter"
    }
  ]
}'
```

When a person actions and thing, it serves two purposes in HapiGER:

1. It is used to measure a persons similarity to other people
2. It can be a recommendation of that thing

For example, when `Bob` `buy`s `LOTR`

```bash
curl -X POST 'http://localhost:3456/events' -d '{
    "events": [
    {
      "namespace":  "movies"
      "person":     "Bob",
      "action":     "buy",
      "thing":      "LOTR",
      "expires_at": "2016-10-12"
    }
  ]
}'
```

This is an action that can be used to find similar people **AND** it can be seen as `Bob` recommending  `LOTR`. For an event to be a recommendation as well it must have an expiration date set with `expires_at`, which is how long the recommendation will be available for.

<br/>
***

#### Recommendations

What books should `Alice` `buy`?

```bash
curl -X POST 'http://localhost:3456/recommendations' -d '{
    "namespace": "movies",
    "person": "Alice",
    "configuration": {
      "actions" : {"view": 5, "buy": 10}
    }
  ]
}'
```

```
{
  "recommendations":[
    {
      "thing":"The Hobbit",
      "weight":0.22119921692859512,
      "people":[
        "Bob"
      ],
      "last_actioned_at":"2015-02-05T05:56:42.862Z"
    }
  ],
  "confidence":0.00019020140391302825,
  "similar_people":{
    "Bob":1
  }
}
```

`Alice` should buy `The Hobbit` as it was recommended by `Bob` with a weight of about `0.2`.

The `configuration` defines many variables that can be used to customise the search for recommendations. The object is directly passed to GER and the available variables are listed in the [GER Documentation](https://github.com/grahamjenson/ger).

*The `confidence` of these recommendations is pretty low because there are not many events in the system*

<br/>
***

#### How HapiGER Works (the Quick Version)

The HapiGER API calculates recommendations for `Alice` to `buy` by:

1. Finding people (neighbors) that are like `Alice` by looking at her past events
2. Calculating the similarities between `Alice` and her neighbors
3. Looking at the recent `things` that those similar people recommended
4. Weight those recommendations using the similarity of the people

<br/>
***

#### Event Stores

The "in-memory" memory event store is the default, this will not scale well or persist event so is not recommended for production.

The **recommended** event store is **PostgreSQL**, which can be used with:

```
hapiger --es pg --esoptions '{
    "connection":"postgres://localhost/hapiger"
  }'
```

*Options are passed to [knex](http://knexjs.org/).*

HapiGER also supports a [RethinkDB](http://rethinkdb.com/) event store:

```
hapiger --es rethinkdb --esoptions '{
    "host":"127.0.0.1",
    "port": 28015,
    "db":"hapiger"
  }'
```

*Options passed to [rethinkdbdash](https://github.com/neumino/rethinkdbdash).*

<br/>
***

#### Compacting the Event Store

The event store needs to be regularly maintained by removing old, outdated, or superfluous events; this is called **compacting**

```
curl -X POST 'http://localhost:3456/compact' -d {
  "namespace": "movies"
}
```


<br/>
***

#### Namespaces

In addition to creating namespaces, you can also list and destroy them:

```
curl -X GET 'http://localhost:3456/namespaces'
```

To delete a namespace (**and all its events!**):

```
curl -X DELETE 'http://localhost:3456/namespace/movies'
```


<br/>
***

#### Clients

1. Node.js client [ger-client](https://www.npmjs.com/package/ger-client)

## Changelog

8/02/15 -- Updated readme and bumped version
