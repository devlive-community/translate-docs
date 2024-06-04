[TOC]

### clone

-------------------------------------------------------------------------------------------------------------

`{*} → {*}`

Parameters

*   valueThe object or array to clone

> Returns * A deeply cloned copy of `val`

Added in v0.1.0

Creates a deep copy of the source that can be used in place of the source object without retaining any references to it. The source object may contain (nested) `Array`s and `Object`s, `Number`s, `String`s, `Boolean`s and `Date`s. `Function`s are assigned by reference rather than copied.

Dispatches to a `clone` method if present.

Note that if the source object has multiple nodes that share a reference, the returned object will have the same structure, but the references will be pointed to the location within the cloned value.

```js
const objects = [{}, {}, {}];
const objectsClone = R.clone(objects);
objects === objectsClone; //=> false
objects[0] === objectsClone[0]; //=> false
```

### values

----------------------------------------------------------------------------------------------------------------

`{k: v} → [v]`

Parameters

*   objThe object to extract values from

> Returns Array An array of the values of the object's own properties.

Added in v0.1.0

Returns a list of all the enumerable own properties of the supplied object. Note that the order of the output array is not guaranteed across different JS platforms.

See also [valuesIn](#valuesIn), [keys](#keys), [toPairs](#toPairs).

```js
R.values({a: 1, b: 2, c: 3}); //=> [1, 2, 3]
```

### eqProps

-------------------------------------------------------------------------------------------------------------------

`k → {k: v} → {k: v} → Boolean`

Parameters

*   propThe name of the property to compare
*   obj1
*   obj2

> Returns Boolean

Added in v0.1.0

Reports whether two objects have the same value, in [`R.equals`](#equals) terms, for the specified property. Useful as a curried predicate.

```js
const o1 = { a: 1, b: 2, c: 3, d: 4 };
const o2 = { a: 10, b: 20, c: 3, d: 40 };
R.eqProps('a', o1, o2); //=> false
R.eqProps('c', o1, o2); //=> true
```

### keys

----------------------------------------------------------------------------------------------------------

`{k: v} → [k]`

Parameters

*   objThe object to extract properties from

> Returns Array An array of the object's own properties.

Added in v0.1.0

Returns a list containing the names of all the enumerable own properties of the supplied object. Note that the order of the output array is not guaranteed to be consistent across different JS platforms.

See also [keysIn](#keysIn), [values](#values), [toPairs](#toPairs).

```js
R.keys({a: 1, b: 2, c: 3}); //=> ['a', 'b', 'c']
```

### omit

----------------------------------------------------------------------------------------------------------

`[String] → {String: *} → {String: *}`

Parameters

*   namesan array of String property names to omit from the new object
*   objThe object to copy from

> Returns Object A new object with properties from `names` not on it.

Added in v0.1.0

Returns a partial copy of an object omitting the keys specified.

See also [pick](#pick).

```js
R.omit(['a', 'd'], {a: 1, b: 2, c: 3, d: 4}); //=> {b: 2, c: 3}
```

### pick

----------------------------------------------------------------------------------------------------------

`[k] → {k: v} → {k: v}`

Parameters

*   namesan array of String property names to copy onto a new object
*   objThe object to copy from

> Returns Object A new object with only properties from `names` on it.

Added in v0.1.0

Returns a partial copy of an object containing only the keys specified. If the key does not exist, the property is ignored.

See also [omit](#omit), [props](#props).

```js
R.pick(['a', 'd'], {a: 1, b: 2, c: 3, d: 4}); //=> {a: 1, d: 4}
R.pick(['a', 'e', 'f'], {a: 1, b: 2, c: 3, d: 4}); //=> {a: 1}
```

### pickAll

-------------------------------------------------------------------------------------------------------------------

`[k] → {k: v} → {k: v}`

Parameters

*   namesan array of String property names to copy onto a new object
*   objThe object to copy from

> Returns Object A new object with only properties from `names` on it.

Added in v0.1.0

Similar to `pick` except that this one includes a `key: undefined` pair for properties that don't exist.

See also [pick](#pick).

```js
R.pickAll(['a', 'd'], {a: 1, b: 2, c: 3, d: 4}); //=> {a: 1, d: 4}
R.pickAll(['a', 'e', 'f'], {a: 1, b: 2, c: 3, d: 4}); //=> {a: 1, e: undefined, f: undefined}
```

### project

---

`[k] → [{k: v}] → [{k: v}]`

Parameters

*   propsThe property names to project
*   objsThe objects to query

> Returns Array An array of objects with just the `props` properties.

Added in v0.1.0

Reasonable analog to SQL `select` statement.

See also [pluck](#pluck), [props](#props), [prop](#prop).

```js
const abby = {name: 'Abby', age: 7, hair: 'blond', grade: 2};
const fred = {name: 'Fred', age: 12, hair: 'brown', grade: 7};
const kids = [abby, fred];
R.project(['name', 'grade'], kids); //=> [{name: 'Abby', grade: 2}, {name: 'Fred', grade: 7}]
```

### prop

---

`Idx → {s: a} → a | Undefined`

`Idx = String | Int | Symbol`

Parameters

*   pThe property name or array index
*   objThe object to query

> Returns * The value at `obj.p`.

Added in v0.1.0

Returns a function that when supplied an object returns the indicated property of that object, if it exists.

See also [path](#path), [props](#props), [pluck](#pluck), [project](#project), [nth](#nth).

```js
R.prop('x', {x: 100}); //=> 100
R.prop('x', {}); //=> undefined
R.prop(0, [100]); //=> 100
R.compose(R.inc, R.prop('x'))({ x: 3 }) //=> 4
```

### props

---

`[k] → {k: v} → [v]`

Parameters

*   psThe property names to fetch
*   objThe object to query

> Returns Array The corresponding values or partially applied function.

Added in v0.1.0

Acts as multiple `prop`: array of keys in, array of values out. Preserves order.

See also [prop](#prop), [pluck](#pluck), [project](#project).

```js
R.props(['x', 'y'], {x: 1, y: 2}); //=> [1, 2]
R.props(['c', 'a', 'b'], {b: 2, a: 1}); //=> [undefined, 1, 2]

const fullName = R.compose(R.join(' '), R.props(['first', 'last']));
fullName({last: 'Bullet-Tooth', age: 33, first: 'Tony'}); //=> 'Tony Bullet-Tooth'
```