# jsqry manual

[jsqry](https://github.com/jsqry/jsqry) is a simple JS lib to query objects/arrays.

## API

API of the lib is truly minimalistic and consists only of two functions: 
`query` and `first`.

```js
jsqry.query(target, queryString[,...args])
```

Query target `target` list or object by query defined by `queryString`. Arguments `args` can be used
in case when parameterized query is used.

Example: 

`query([1,2,3,4,5], '[ _>2 && _<5 ]')` gives `[3,4]`.

Same with parameterized query:  

`query([1,2,3,4,5], '[_>?&&_<?]', 2, 5)` gives same result `[3,4]`.

```js
jsqry.first(target, queryString[,...args])
```

Same as `jsqry.query` described above but returns first item of result list.

Example:

`first([1,2,3,4,5], '[_>?&&_<?]', 2, 5)` gives `3`.

## Query syntax

Query in general can have a form below

```
field1.field2[ condition or index or slice ].field3{ transformation }.field4<< query >>.field5.x( func )
```

Here:

| part                      | meaning                                              |
----------------------------|-------------------------------------------------------
| `field1.field2.field3...` | [simple fields access](#field-access), same as in JS |
| `[ condition ]`           | [filtering](#filtering)                              |
| `[ index ]`               | [index access](#indexing), same as in JS             |
| `[ from:to:step ]`        | [slices](#slicing), Python-style                     |
| `{ transformation }`      | [object transformation](#transformation)             |
| `.x( func )`              | [call action](#calls)                                |
| `<< query >>`             | [nested filtering](#nested-filtering)                |

Note: all mentioned query elements are optional and can go in arbitrary order.

### Field access

| Example                                         | Result    |
--------------------------------------------------|------------
| `query([{a:1},{a:2},{a:3}], 'a')`               | `[1,2,3]` |
| `query([{a:{b:1}},{a:{b:2}},{a:{b:3}}], 'a.b')` | `[1,2,3]` |

### Filtering

Let's elaborate a bit how **condition** and **transformation** work.
In fact it's very simple. 
Every expression in square / curly brackets during execution is substituted this way: 

| from                          | to                                                     |
--------------------------------|---------------------------------------------------------
| `condition_or_transformation` | `function(_,i) { return condition_or_transformation }` |

(here **_** — the value of item, **i** — it's index).

| Example                                                 | Result          |
----------------------------------------------------------|------------------
| `query([1,2,3,4,5], '[ _ % 2 == 1 ]')`                  | `[1,3,5]`       |
| `query([{a:1},{a:2},{a:3}], '[_.a>=2]')`                | `[{a:2},{a:3}]` |
| `query(["a", "bb", "aaa", "c"], '[_.startsWith("a")]')` | `["a","aaa"]`   |
| `query(["a", "bb", "aaa", "c"], '[_.length>1]')`        | `["bb","aaa"]`  |

### Indexing

TODO

### Slicing

TODO

### Transformation

TODO

### Calls

TODO

### Nested filtering

TODO

