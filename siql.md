SIQL
====

Preface
-------

SIQL (pronounced like "cycle") stands for **si**phon.io **q**uery **l**anguage.
It is a common language used for querying various [siphon.io](http://siphon.io/)
services.

SIQL has been designed to meet two main goals:

1. Be easy to both create and consume for both humans and computers
2. Be compatible across multiple data source backends

The first goal is achieved by basing the entire language on common data
structures and concepts.

The second is achieved by reducing the language's features to only the subset
required for read access to real time data sources.

Structure
---------

A query is a JSON document with a specific structure. The two fields available
for use are `fields` and `condition`. They control the fields returned to you
and the filters applied to the data source you're querying respectively.

Specifying Fields
-----------------

Specifying which fields to return is done by providing a property called
`fields` in the query. It controls what fields are returned in the response
stream.

This field should be an object consisting of `dot.path` notation field names as
keys and booleans as values.

To include a field in the response stream, set a key with its name to `true`,
as in the following example:

```json
{
  "fields": {
    "title": true
  }
}
```

That configuration would result in only the `title` field being included in the
response stream.

To exclude a field from the response stream, set a key with its name to `false`,
as in the next example:

```json
{
  "fields": {
    "title": false
  }
}
```

That configuration will result in all fields except `title` being included in
the response stream.

> **NOTE**: if a field is specified in the query, but it does not exist in the
> data being returned, it will simply not appear in the output.

If this part of the query is omitted, or is empty, all fields from the data
source will be returned.

Applying Filters
----------------

Filters are applied to the data source via the `condition` property. By 
combining a few simple condition types, rich filters can be created in an easy
to follow manner.

There are two categories of conditions available for use with SIQL.

The first category contains simple comparison conditions, consisting of `eq`
(equal), `ne` (not equal), `gt` (greater than) and `lt` (less than). Note that
there is no "contains" or "partial match" functionality, as this is not
applicable to many data sources.

The second category contains group conditions, consisting of `or` and `and`.
These can be used to create fine-grained queries covering multiple fields and
circumstances.

Here is an example of querying using a comparison condition:

```json
{
  "condition": {
    "type": "eq",
    "left": "published",
    "right": true
  }
}
```

That example applies an `eq` (equal) condition to the `published` field. Only
documents where `published` is equal to `true` will be returned.

> **NOTE**: type safety is *not* guaranteed! Some data sources are more strict
> about types than others. The implication of this is that sometimes `1` will be
> considered as equal to `"1"` or `true` might be considered equal to anything
> "truthy".

The other comparison types operate roughly the same, except with their own
meanings. `ne` is the opposite of `eq`, `lt` tests if a field is less than a
certain value and `gt` does the opposite of `lt`.

Group conditions have different parameters to comparison conditions, but are
just as easy to use. Here's an example of grouping two comparison conditions:

```json
{
  "condition": {
    "type": "and",
    "data": [
      {
        "type": "eq",
        "left": "published",
        "right": true
      },
      {
        "type": "ne",
        "left": "deleted",
        "right": true
      }
    ]
  }
}
```

That example would only return documents where `published` is `true` and
`deleted` is not `true`.

> **NOTE**: there is a subtle difference between "not true" and "false". If a
> document does not contain the field, it's not false, but it's also not true.
> That means it will not pass a "equals false" test, but it will pass a "not
> true" test. Watch out for this!

Both group and comparison conditions can both be included in a group condition,
and you can mix or match them however you like. This allows you to build
arbitrarily complex queries in layers.

Full Example
------------

This is a full example that uses all the features of SIQL.

```json
{
  "fields": {
    "id": true,
    "title": true,
    "time": true,
    "author.name": true
  },
  "condition": {
    "type": "and",
    "data": [
      {"type": "eq", "left": "published", "right": true},
      {"type": "eq", "left": "deleted", "right": false},
      {"type": "gt", "left": "time", "right": "2011-01-01T00:00:00.000Z"},
      {"type": "lt", "left": "time", "right": "2011-02-01T00:00:00.000Z"},
      {
        "type": "or",
        "data": [
          {"type": "eq", "left": "author.name", "right": "rei"},
          {"type": "eq", "left": "author.name", "right": "shinji"}
        ]
      }
    ]
  }
}
```
