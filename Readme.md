# Typed JSON

Typed JSON is a format for defining structured [JSON][http://www.json.org/]
data, that can be used by type systems or contract / guard librarires
to allow cross-language type-safety guarantees.

Typed JSON enables type information to be preserved between typed
and untyped languages. It can be used by developer tools and compilers
to decrease the burden of validating and using data.

Typed JSON supports:

* Primitive types
* Basic data structures (arrays, records, tuples)
* Union types (which can express Algebraic Data Types and sub-types)

Every type is associated with a unique URI. This can be used simply as
a unique identifier, or it can be associated with a type definition.


## Primitive Types

Typed JSON defines base primitive types that are common to almost all
languages. Each primitive type is associated with specific URIs.

```json
{
  "bool"  : "http://typed-json.org/#boolean",
  "int"   : "http://typed-json.org/#int",
  "float" : "http://typed-json.org/#float",
  "string": "http://typed-json.org/#string",
  "null"  : "http://typed-json.org/#null"
}
```

Primitive types can be referenced by their URI or aliased as a local type.
In the example above, we created a local type alias for all of our primitive
types. These aliases can be used later to make more complex types easier to read.

The `null` primitive type represents the absense of value. It is actually
a type of its own, not a possible value for all objects.

Since `null` is valid JSON primitive it can be used instead of URI
http://typed-json.org/#null. For example type `empty` can be defined
as an alias to http://typed-json.org/#null as follows: `{ "empty": null }`


## Composite types

Typed JSON is great for defining composite types. The key composite types are:

 * Records: a group of named fields, each associated with a type (similar to objects)
 * Collections: arrays or lists of a single type
 * Tuples: fixed sized containers with mixed types

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

Collections, like arrays (or lists, depending on the language),
must contain items of specified type and are defined
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
[]
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

Tuples are fixed size containers with *mixed* types.
In contrast to regular fixed size arrays, they define element
types by index.

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
```

*Note: "color" is just an alias for a string with a different
semantic meaning. It's useful to give semantic meaning to entities
used in type definitions, as it allows changing the types of those entities
independently from computed types. This makes it easy to replace color with
a record of RGB values at some point.*

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

In languages like Elm and Haskell, the name of the constant type
could be used to create a simple Algebraic Data Type:

```haskell
data ReadyStatus = ReadyStatus
data ReadyState = ReadyState
data Yes = Yes
```

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

It is still possible to map this onto ADTs because strings implicitly have names.
Unlike strings, constant integers and floats must be defined explicitly to ensure
they have a name.

```json
{
  "two": 2,
  "three": 3,
  "five": 5,
  "seven": 7,
  "prime-digits": "two|three|five|seven"
}
```

In JavaScript, this would just send an integer over the wire.
In Haskell or Elm, this would be represented as:

```haskell
data PrimeDigits = Two | Three | Five | Seven
```

This lets you work with the colloquial representation in very different languages.

# Prior art:

- [Elm records](http://elm-lang.org/learn/Records.elm)
- [MongoDB BSON](http://bsonspec.org/)
- [JSON Schema](http://json-schema.org/)
- [Protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)

[JSON]:http://json.org/
[structural typing]:http://en.wikipedia.org/wiki/Structural_type_system
[keywords]:http://clojure.org/data_structures#Data%20Structures-Keywords
[Union_types]:https://en.wikipedia.org/wiki/Union_type
