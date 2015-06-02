# Reflect.type

This is a stage 0, "strawman" proposal.

## Problem

`typeof` has long been known to have perculiar behaviour. The most famously understood issue is that `typeof null === "object"`, which consistently requires special casing when dispatching on a value's type. It was [proposed] [1] to correct `typeof null` to return `"null"`, but in the interests of the Harmony/"not breaking the web" initiative, it was decided that this behaviour should remain unfixed.

For the purposes of efficient specification, the *ECMAScript* specification actually defines a trivial algorithm `Type(x)` which has a more straightforward behaviour than `typeof`. In particular:
 * `null` has type *Null*; and
 * Functions have no special treatment - they are instances of *Object*.

Though desirable `Type(x)` is not available to *ECMAScript* programs.

## Solution

A stated goal of the `Reflect` module is to ["expose the essential methods that make up JavaScript's object model as defined by the spec"] [2]. As such, it provides an obvious opportunity to make the specification's `Type(x)` algorithm available directly to *ECMAScript* programs.

`Reflect.type(x)` is proposed to accept as its sole argument any value `x` and return a symbol that uniquely corresponds to the type of the value, as defined by the specification. Such values would be exposed in the `Reflect.types` object, keyed by conventional spellings of the specification types, i.e. equivalent to:
```js
Reflect.types = {
    undefined: @@undefinedType,
    null: @@nullType,
    boolean: @@booleanType,
    string: @@stringType,
    symbol: @@symbolType,
    number: @@numberType,
    object: @@objectType,
}
```
where `@@` denotes well-known symbols, referenced by algorithms of the specification.

A possible implementation of `Reflect.type` would be equivalent to:
```js
Reflect.type = function(x) {
    switch(typeof x) {
    case "undefined":
        return @@undefinedType;
    case "boolean":
        return @@booleanType;
    case "string":
        return @@stringType;
    case "symbol":
        return @@symbolType;
    case "number":
        return @@numberType;
    case "object":
    case "function":
    default:   // NB: `typeof` may return arbitrary strings for implementation-defined values!
        if (x === null) {
            return @@nullType;
        }
        return @@objectType;
    }
};
```

## Notes

* Using symbols to represent each type as opposed to short strings as `typeof` does may prove useful in ensuring that the domain values of `typeof` and `Reflect.type` are never accidentally mismatched.
* Symbol descriptions can aid debugging, as the default `Symbol#toString` implementation includes the description.

## Outstanding Issues

* How might new, user-definable value types be exposed? Assuming they become new primitives, `Reflect.type` should probably return an appropriate symbol, accessible from whatever mechanism is used to define the type. Alternatively, greater symmetry could be achieved by introducing global objects `Null` and `Undefined`, that, alongside the existing globals `Boolean`, `String`, `Symbol`, `Number`, and `Object`, and user-defined value objects, would have a `type` property holding the relevant symbol, such that e.g. `Reflect.type(undefined) === Undefined.type`
* Perhaps, for symmetry with `Object.getPrototypeOf`, it could be argued that this should be named `Reflect.getTypeOf`. This is both more verbose and at odds with the current specification language.

 [1]: http://wiki.ecmascript.org/doku.php?id=harmony%3atypeof_null
 [2]: https://esdiscuss.org/topic/maybe-we-need-a-reflect-api-to-iterate-over-instance-members#content-9
