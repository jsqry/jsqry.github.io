# jsqry manual

[jsqry](https://github.com/jsqry/jsqry) is a simple JS lib to query objects/arrays.

## API

API of jsqry is truly minimalistic and consists only of two functions: 
`query` and `first`.

```js
jsqry.query(target, query[, ...args])
```

Query `target` list or object by `query`. Arguments `args` can be used
for parameterized queries.

Example                                      | Result  | Comment              
---------------------------------------------|---------|----------------------
`query([1,2,3,4,5], '[ _>2 && _<5 ]')`       | `[3,4]` |                      
`query([1,2,3,4,5], '[ _>? && _<? ]', 2, 5)` | `[3,4]` | same using parameters

```js
jsqry.first(target, query[, ...args])
```

Same as `jsqry.query` described above but returns first item of result list.

Example                                          | Result
-------------------------------------------------|-------
`first([1,2,3,4,5], '[ _>? && _<? ]', 2, 5)`     | `3`   

## Query syntax

Query in general can have a form below

```
field1.field2[ condition or index or slice ].field3{ transformation }.field4<< query >>.field5.x( func )
```

Here:

part                      | meaning                                             
--------------------------|-----------------------------------------------------
`field1.field2.field3...` | [simple fields access](#field-access), same as in JS
`[ condition ]`           | [filtering](#filtering)                             
`[ index ]`               | [index access](#indexing), same as in JS            
`[ from:to:step ]`        | [slices](#slicing), Python-style                    
`{ transformation }`      | [object transformation](#transformation)            
`.x( func )`              | [call action](#calls)                               
`<< query >>`             | [nested filtering](#nested-filtering)               

Note: all mentioned query elements are optional and can go in arbitrary order.

### Field access

Example                                                | Result   
-------------------------------------------------------|----------
`query([{a:1},{a:2},{a:3}], 'a')`                      | `[1,2,3]`
`query([{a:{b:1}},{a:{b:2}},{a:{b:3}}], 'a.b')`        | `[1,2,3]`
`query([{name:"John"},{name:"Peter"}], 'name.length')` | `[4,5]`

### Filtering

Let's elaborate a bit how **condition** and **transformation** work.
<br>In fact it's very simple. 
Every expression in square / curly brackets during execution is substituted this way: 

from                          | to                                                    
------------------------------|-------------------------------------------------------
`condition_or_transformation` | `function(_,i) { return condition_or_transformation }`

(here **_** — the value of item, **i** — it's index).

Example                                                 | Result         
--------------------------------------------------------|----------------
`query([1,2,3,4,5], '[ _ % 2 == 1 ]')`                  | `[1,3,5]`      
`query([{a:1},{a:2},{a:3}], '[_.a>=2]')`                | `[{a:2},{a:3}]`
`query(["a", "bb", "aaa", "c"], '[_.startsWith("a")]')` | `["a","aaa"]`  
`query(["a", "bb", "aaa", "c"], '[_.length>1]')`        | `["bb","aaa"]` 

Examples using index:

Example                                  | Result        | Comment               
-----------------------------------------|---------------|-----------------------
`query([1,2,3,4,5,6], '[ i % 2 == 0 ]')` | `[2,4,6]`     | Take every 2nd element
`query([1,2,3,4,5,6], '[ i > 0 ]')`      | `[2,3,4,5,6]` | Omit first element    
 
 
### Indexing

Example                                        | Result    | Comment     
-----------------------------------------------|-----------|-------------
`query(["a", "bb", "aaa", "c"], '[2]')`        | `["aaa"]` |
`first(["a", "bb", "aaa", "c"], '[2]')`        | `"aaa"`   |
`first(["a", "bb", "aaa", "c"], '[2].length')` | `3`       |
`first(["a", "bb", "aaa", "c"], '[-1]')`       | `"c"`     | last element

### Slicing

This is very similar to [Python's slicing](https://www.dotnetperls.com/slice-python).
<br>Has form of `[ from:to:step ]`. Any of `from` / `to` / `step` is optional. 

Example                        | Result        | Comment     
-------------------------------|---------------|------------------
`query([1,2,3,4,5], '[::2]')`  | `[1,3,5]`     | Every 2nd element
`query([1,2,3,4,5], '[1:]')`   | `[2,3,4,5]`   | All but 1st item
`query([1,2,3,4,5], '[::-1]')` | `[5,4,3,2,1]` | Reverse
`query([1,2,3,4,5], '[:3]')`   | `[1,2,3]`     | First 3 

### Transformation

TODO

### Calls

TODO

### Nested filtering

TODO

