# jsqry manual

[jsqry](https://github.com/jsqry/jsqry) is a simple JS lib to query objects/arrays.

## API

API of the lib is truly minimalistic and consists only of two functions: 
`query` and `first`.

<br>

```js
jsqry.query(target, queryString[,...args])
```

Query target `target` list or object by query defined by `queryString`. Arguments `args` can be used
in case when parameterized query is used.

Example: 

`query([1,2,3,4,5], '[ _>2 && _<5 ]')` gives `[3,4]`.

Same with parameterized query:  

`query([1,2,3,4,5], '[_>?&&_<?]', 2, 5)` gives same result `[3,4]`.

<br>

```js
jsqry.first(target, queryString[,...args])
```

Same as `jsqry.query` described above but returns first item of result list.

Example:

`first([1,2,3,4,5], '[_>?&&_<?]', 2, 5)` gives `3`.

## Query syntax
