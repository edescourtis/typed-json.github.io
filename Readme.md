# Typed JSON

Typed JSON is a format for defining structured [JSON][] data, that
can be used by language type systems or contract / guard librarires
to do some type safety guarantees.

## Format

Every type is associated with a unique URI. It is recommended to have a type
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

String type can be aliased as `string` as follows:

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

Above JSON defines `point` type that **must** have `x` and `y` fields
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

Collections, like arrays (different languages could use different collection
types, lists for example), must contain items of specified type and are defined
as follows:

```json
{
  "int": "http://typed-json.org/#int",
  "point": { "x": "int", "y": "int" },
  "shape": ["point"]
}
```

*Note: The example above defines a `shape` type that is a
collection of an arbitrary number of `point` items*

The following JSON data would conform to the `shape` type:

```js
[{"x":0, "y":0}]
[{"x":0, "y":0}, {"x": 0, "y": 10}]
[{"x":0, "y":0}, {"x": 0, "y": 10}, {"x": 10: "y": 10}]
```

It is also possible to define fixed-size collections:


```json
{
  "int": "http://typed-json.org/#int",
  "point": ["int", 2],
  "line": ["point", 2]
}
```

The following JSON data would conform to the `line` type:

```js
[[0,0], [0,10]]
[[0,0], [10,10]]
```

### Tuples

Tuples are fixed size JS arrays. In contrast to regular fixed
size arrays, they define element types by index and therefore
are preferable for defining mixed-type arrays:

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

The above `pixel` type defines a structure for values like:

```js
[{x:0, y:0}, "red"]
[{x:0, y:12}, "green"]
````

*Note: "color" is just an alias for a string with a different
semantic meaning. It's useful to give semantic meaning to entities
used in type definitions, as it allows changing the types of those entities
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
for the `digit` type. Metadata keys must be mapped to objects whose
fields are not specified by this format. Different environments may
choose to support metadata fields. For example, the `digit` type's metadata
specifies range of ints, but if runtime does not support ranges, it
will still treat `digit` as `int` type.

## Constants

Specification recognizes constants of `string`, `integer`, `float`
and `boolean` types:

```json
{
  "readyStatus": 1,
  "readyState": "'complete'",
  "yes": true,
}
```

Above data structure defines type `readyStatus` constant of `int`
type that will only match `1`. Type `yes` is a boolean that is
`true`. Type `readyState` is a constant primitive that matches
`'complete'` string in JSON although in some languages that could
translate to more appropriate contant values like [keywords][]
in clojure.


## Union types

Composite types can also be defined in form of [union types][] to
allow structures that can contain non-homogeneous types:

```json
{
  "string": "http://typed-json.org/#string",
  "string": "http://typed-json.org/#int",
  "pending": { "pending": "int" },
  "complete": { "data": "string" },
  "status": "pending|complete"
}
```

This is very natural in untyped languages like JavaScript, but it also maps nicely onto [Algebraic Data Types](http://elm-lang.org/learn/Pattern-Matching.elm) (ADT) in functional languages like Elm and Haskell. It also maps onto subclasses in OO languages like Java.

In Haskell, the `"status"` type would be represented as:

```haskell
data Status = Pending { pending :: Int }
            | Complete { data :: String }
```

## Unions with Constant Types

Unions can be defined over constant types as well:

```json
{
  "yes": "'yes'",
  "no": "'no'",
  "show": "yes|no"
}
```

In JavaScript, the values passed along would be a string `'yes'` or `'no'`. A statically-typed functional language like Elm or Haskell would represent this as:

```haskell
data Show = Yes | No
```

It is guaranteed that members of a union type are named, so it is always safe to map onto an ADT.

There is also syntax sugar to express above in more
concise way:

```json
{
  "show": "'yes'|'no'"
}
```

This is allowed because string literals have names. It is still possible to map onto ADTs.

Therefore, this syntactic sugar is not permitted for other type literals. Literal integers and floats must be defined explicitly to ensure there is a name.

```json
{
  "two": 2,
  "three": 3,
  "five": 5,
  "seven": 7,
  "prime-digits": "two|three|five|seven"
}
```

In JavaScript, this would just send an integer along the wire. In Haskell or Elm, this would be represented as:

```haskell
data PrimeDigits = Two | Three | Five | Seven
```

This lets you get the colloquial representation in very different languages.

# Prior art:

- [Elm records](http://elm-lang.org/learn/Records.elm)
- [MongoDB BSON](http://bsonspec.org/)
- [JSON Schema](http://json-schema.org/)
- [Protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)

[JSON]:http://json.org/
[structural typing]:http://en.wikipedia.org/wiki/Structural_type_system
[keywords]:http://clojure.org/data_structures#Data%20Structures-Keywords
[Union_types]:https://en.wikipedia.org/wiki/Union_type
