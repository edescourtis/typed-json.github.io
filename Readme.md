# Typed JSON

Typed JSON is a format for defining structured [JSON][] data, that
can be used by language type systems or contract / guard librarires
to do some type safety guarantees.

## Format

Every type is associtade with a unique URI. It is recommended to have a type
definition under that URI, but it is not a requirement, it is up to implementor
to associate actual definitions with a URI, in other words type URIs are just
a unique identifiers for types.

Format defines some base primitive types that are also associated with specific
URIs. This specification recognizes following primitive types:

### Null

Primitive type representing an absense of value. In the type vocabulary they
can be referenced either by URI or aliased as a local type and referced by
alias name:


```json
{
  "null": "http://typed-json.org/#Null"
}
```

*Note: Above definition defines `"null"` just as an alias to primitive
http://typed-json.org/#null type, so that rest of the type vocabulary
would be able to refer to it by that identifier.*


Since `null` is valid JSON primitive it can be used instead of URI
http://typed-json.org/#null. For example type `empty` can be defined
as an alias to http://typed-json.org/#null as follows:

```json
{
  "empty": null
}
```


### Boolean

Boolean type can be aliased as `bool` as follows:

```json
{
  "bool": "http://typed-json.org/#boolean"
}
```


### Int

Ints type can be aliased as `int` as follows:

```json
{
  "int": "http://typed-json.org/#int"
}
```

*Note: int is JS int*

### Float

Float type can be aliased as `float` as follows:

```json
{
  "float": "http://typed-json.org/#float"
}
```

### String

String type can be aliased as `float` as follows:

```json
{
  "string": "http://typed-json.org/#string"
}
```

## Composite types

Typed JSON is used mainly for defining composite type structures. There are
few type structures that can be expressed:

### Records

Record types represent JSON objects with a specific structure. They are
defined in terms of field type signatures:


```json
{
  "point": {
    "x": "http://typed-json.org/#int",
    "y": "http://typed-json.org/#int"
  }
}
```

Above JSON defines `point` type that **Must** have `x` and `y` fields
of `int` type. This example uses full URIs to for field type definitions,
but that's redundant and could be expressed in more eloquent manner:


```json
{
  "int": "http://typed-json.org/#int",
  "point": { "x": "int", "y": "int" }
}
```

Composite data type definitions can refer to other composite types:

```json
{
  "int": "http://typed-json.org/#int",
  "point": { "x": "int", "y": "int" },
  "line": {
    "start": "point",
    "end": "point"
  }
}
```

### Collections

Collections like arrays (different languages could use different collection
types, lists for example) must contain items of certain type(s) and are defined
as follows:

```json
{
  "int": "http://typed-json.org/#int",
  "point": { "x": "int", "y": "int" },
  "shape": ["point"]
}
```

*Note: Above example above defines `shape` type that is
collection of `point` type items of arbitrary number*

Following JSON data would match `shape` type:

```js
[{"x":0, "y":0}]
[{"x":0, "y":0}, {"x": 0, "y": 10}]
[{"x":0, "y":0}, {"x": 0, "y": 10}, {"x": 10: "y": 10}]
```

It is also possible to define fixed size collections:


```json
{
  "int": "http://typed-json.org/#int",
  "point": [int, 2]
  "line": ["point", 2]
}
```

Following JSON data would match `line` type definition:

```js
[[0,0], [0,10]]
[[0,0], [10,10]]
```

### Tuples

Tuples are fixed size JS arrays, in contrast to regular fixed
size arrays they define element types by index and there for
are preferable for defining mixed type arrays:

```json
{
  "int": "http://typed-json.org/#int",
  "string": "http://typed-json.org/#string",
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
[{x:0, y:12}, "green"]
````

*Note: That "color" is just an alias for a string with a different
semantic meaning. It's useful to give semantic meaning to an entities
used in type definitions, that allows changing types of those entities
independently from computed types*

## Metadata

New primitives can be defined by aliasing existing primitive types
and adding some additional metadata. For example type `digit` can
be defined as:

```json
{
  "digit": "http://typed-json.org/#int",
  "digit:meta": {
    "min": 0,
    "max": 9
  }
}
```

Note that above definition uses "digit:meta" key to define metadata
for the `digit` type. Metadata keys must be mapped to an objects who's
fields are not specified by this format. Different environments may
choose to support metadata fields, for example `digit` type metadata
specifies range of ints, but if runtime does not supports ranges it
will still treat `digit` as `int` type.

## Constants

Specification recognizes constants of `string`, `integer`, `float`
and `boolean` types:

```json
{
  "readyStatus": 1,
  "readyState": "'complete'",
  "yes": "true",
}
```

Above data structure defines type `readyStatus` constant of `int`
type that will only match `1`. Type `yes` is a boolean that is
`true`. Type `readyState` is a constant primitive that matches
`"complete"` string in JSON although in some languages that could
translate to more appropriate contant values like [keywords][]
in clojure.


## Union types

Composite types can also be defined in form of [union types][] to
allow structures that can contain either of listed types:

```json
{
  "string": "http://typed-json.org/#string",
  "pending": {"pending":"true"},
  "complete": {"data": "string"}
  "status": "pending|complete"
}
```


Unions can be defined over constant types as well:


```json
{
  "yes": "'yes'",
  "no": "'no'",
  "show": "yes|no"
}
```

There is also syntax sugar to express above in more
consise way:

```json
{
  "show": "'yes'|'no'"
}
```

*Note: Union types support syntax sugar over string type constants
all others have to be defined as types explicitly*

```json
{
  "two": 2,
  "three": 3,
  "five": 5,
  "seven": 7,
  "primedigits": "two|three|five|seven"
}
```


# Prior art:

- [Elm records](http://elm-lang.org/learn/Records.elm)
- [MongoDB BSON](http://bsonspec.org/)
- [JSON Schema](http://json-schema.org/)
- [Protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)

[JSON]:http://json.org/
[structural typing]:http://en.wikipedia.org/wiki/Structural_type_system
[keywords]:http://clojure.org/data_structures#Data%20Structures-Keywords
[Union_types]:https://en.wikipedia.org/wiki/Union_type