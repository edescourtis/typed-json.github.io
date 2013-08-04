# Typograph

Typograph is a JSON based format for defining structured JSON data, that
can be used by language type systems or contract / guard librarires to
do guarantees about type safety.

## Format

Every type is associtade with a unique URI. While it's recommended to have
type definition under that URI it's not a requirement, but rather a unique
identifier for the given type. All primitive types are also associated with
a specific URIs. This specification definese set of following primitive types:

### null

Primitive type representing absense of value. It can be aliased in a local
typegraph file as follows:


```json
{
  "null": "http://typograph.io/#null"
}
```

*Note: `"null"` is just an identifier that will be used to refer to `null`
type in the rest of the graph, but any different identifier would work just
fine*


Since `null` is valid JSON primitive it can be referenced in type definitions
as `null` which will be equivalent of `http://typograph.io/#null`.


### boolean

Boolean type that can be either `true` or `false`. Booleans can be aliased in
a local typegraph file as:

```json
{
  "bool": "http://typograph.io/#boolean"
}
```


### integer

```json
{
  "int": "http://typograph.io/#integer"
}
```

### float

```json
{
  "float": "http://typograph.io/#float"
}
```

### string

```json
{
  "string": "http://typograph.io/#string"
}
```

## Composite types

Main use of typograph is for defining composite type structures. There are
few type structures that can be expressed:

### Records

Record types represent JSON objects with a specific structure. They are defined
by providing signiture of it's fields:


```json
{
  "point": {
    "x": "http://typograph.io/#integer",
    "y": "http://typograph.io/#integer"
  }
}
```

Above typograph defines `point` type that **Must** have `x` and `y` fields
with an integer values. In this example actual integer type identifiers were
used, but same could have being expressed by aliasing integer type:


```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" }
}
```

More complex structures could be defined by reusing types defined earlier:

```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "line": {
    "color": "string",
    "start": "point",
    "end": "point"
  }
}
```

### Collections

Collections like arrays (different languages could use different collection
types like lists for example) are expressed as follows:

```json
{
  "int": "http://typograph.io/#integer",
  "graph": ["int"]
}
```

Composite types can be used to compose new collection types:


```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "shape": ["point"]
}
```

### Tuples

Tuples are just a records with indexed fields that are represented
via arrays in JSON. They can be defined as follows:

```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "line": { "0": "point", "1": "point" }
}
```

Above defined `line` type defines structure for values like:

```js
[{x: 0, y: 0}, {x:0, y: 10}]
```

Tupeles can also hold different types of values:

```json
{
  "int": "http://typograph.io/#integer",
  "string": "http://typograph.io/#string",
  "color": "string",
  "point": { "x": "int", "y": "int" },
  "pixel": {
    "0": "point",
    "1": "color"
  }
}
```

Above defined `pixel` type defines structure for values like:

```js
[{x:0, y:0}, "red"]
````

*Note: That "color" is just an alias for a string with a different
semantic meaning. It's useful to give semantic meaning to an entities
used in type definitions, that allows changing types of those entities
independently from computed types*


Tuples also be used for defining types for fixed size collections. Note
that there's no need to define types for each individual index on a tuple
as definition implies that type of the lower indexed elements is same
unless defineded otherwise:


```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "square": { "3": "point" }
}
```

Above definition of `square` is identical to:

```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "square": {
    "0": "point",
    "1": "point",
    "2": "point",
    "3": "point"
  }
}
```

Different types can also be mixed in long tuples as simple
as:


```json
{
  "int": "http://typograph.io/#integer",
  "point": { "x": "int", "y": "int" },
  "find-better-example": {
    "4": "point",
    "6": "int",
    "8": "point"
  }
}
```

Above defitinion of `find-better-example` type defines array
structure of `9` elements where `0` to `4` indexed elements
are of `point` type, `5` to `6` integers and `7` to `8` are
of `point` types.


## Union types

## Constants

Constants can be used to define types for the specific values
that can be reused in union types for example:


```json
{
  "int": "http://typograph.io/#string",
  "true": [[true]],
  "false": [[false]],
  "bool": "true|false"
```

# Prior art:

- [Elm records](http://elm-lang.org/learn/Records.elm)
- [MongoDB BSON](http://bsonspec.org/)
- [JSON Schema](http://json-schema.org/)
- [Protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)

[JSON]:http://json.org/
[structural typing]:http://en.wikipedia.org/wiki/Structural_type_system
