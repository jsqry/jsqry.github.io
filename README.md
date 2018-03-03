# jsqry manual

[jsqry](https://github.com/jsqry/jsqry) is a simple JS lib to query objects/arrays.

## API

API of jsqry is truly minimalistic and consists only of two functions: 
`query` and `first`.

```javascript
jsqry.query(target, query[, ...args])
```

Query `target` list or object by `query`. Arguments `args` can be used
for parameterized queries.

Example                                      | Result  | Comment              
---------------------------------------------|---------|----------------------
`query([1,2,3,4,5], '[ _>2 && _<5 ]')`       | `[3,4]` |                      
`query([1,2,3,4,5], '[ _>? && _<? ]', 2, 5)` | `[3,4]` | same using parameters

```javascript
jsqry.first(target, query[, ...args])
```

Same as `jsqry.query` described above but returns first item of result list.

Example                                          | Result
-------------------------------------------------|-------
`first([1,2,3,4,5], '[ _>? && _<? ]', 2, 5)`     | `3`   

## Query syntax

Query in general can have a form below

```
field1.field2[ CONDITION or INDEX or SLICE ].field3{ TRANSFORMATION }.field4<< QUERY >>.field5.x( EXPRESSION )
```

Here:

part                      | meaning                                             
--------------------------|-----------------------------------------------------
`field1.field2.field3...` | [simple fields access](#field-access), same as in JS
`[ CONDITION ]`           | [filtering](#filtering)                             
`[ INDEX ]`               | [index access](#indexing), same as in JS            
`[ FROM:TO:STEP ]`        | [slices](#slicing), Python-style                    
`{ TRANSFORMATION }`      | [object transformation](#transformation)            
`.x( EXPRESSION )`        | [call action](#calls)                               
`<< QUERY >>`             | [nested filtering](#nested-filtering)               

Note: all mentioned query elements are optional and can go in arbitrary order.

### Field access

Example                                                | Result   
-------------------------------------------------------|----------
`query([{a:1},{a:2},{a:3}], 'a')`                      | `[1,2,3]`
`query([{a:{b:1}},{a:{b:2}},{a:{b:3}}], 'a.b')`        | `[1,2,3]`
`query([{name:"John"},{name:"Peter"}], 'name.length')` | `[4,5]`

### Filtering

Filtering has a form `[ CONDITION ]` where `CONDITION` should be [functional expression](#functional-expression).

#### Functional expression

Let's elaborate a bit how this works.
<br>In fact it's very simple. 
Every expression during execution by jsqry is substituted to a function this way: 

from         | to                                                    
-------------|--------------------------------------
`EXPRESSION` | `function(_,i) { return EXPRESSION }`

(here **_** — the value of item, **i** — it's index).

This function is then applied to the elements being queried.

Examples of filtering:

Example                                                 | Result         
--------------------------------------------------------|----------------
`query([1,2,3,4,5], '[ _ % 2 == 1 ]')`                  | `[1,3,5]`      
`query([{a:1},{a:2},{a:3}], '[_.a>=2]')`                | `[{a:2},{a:3}]`
`query(["a", "bb", "aaa", "c"], '[_.startsWith("a")]')` | `["a","aaa"]`  
`query(["a", "bb", "aaa", "c"], '[_.length>1]')`        | `["bb","aaa"]` 
`query(["",1,null,"B",undefined,333,false], '[_]')`     | `[1, "B", 333]` 

Examples using index:

Example                                  | Result        | Comment               
-----------------------------------------|---------------|-----------------------
`query([1,2,3,4,5,6], '[ i % 2 == 0 ]')` | `[2,4,6]`     | Take every 2nd element *
`query([1,2,3,4,5,6], '[ i > 0 ]')`      | `[2,3,4,5,6]` | Omit first element *    

\* - same result is achievable by [slicing](#slicing) `[::2]` and `[1:]` 
 
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

Transformation has a form `{ TRANSFORMATION }` where `TRANSFORMATION` should be [functional expression](#functional-expression).

Example                                           | Result                  | Comment     
--------------------------------------------------|-------------------------|------------------
`query([1,2,3,4,5], '{_*100}')`                   | `[100,200,300,400,500]` | 
`query([1,2,3,4,5], '{_*?}', 100)`                | `[100,200,300,400,500]` | Same using parameter 
`query(['a', 'bB', 'Ccc'], '{_.toUpperCase()}')`  | `["A", "BB", "CCC"]`    |
`query(Array(5), '{i}')`                          | `[0, 1, 2, 3, 4]`       | Generate number sequence  
`query([1,2,3,5],'{?(_,?)}', Math.pow, 2)`        | `[1, 4, 9, 25]`         | squares
`query([{f:'John',l:'Doe'},{f:'Bill',l:'Smith'}], '{_.f + " " + _.l}')` | `["John Doe","Bill Smith"]` |

### Calls

Calls are used to apply some transformation to collection as a whole.
<br>At the moment these are supported

call             | description
-----------------|-------------
.s( EXPRESSION ) | sorting
.u( EXPRESSION ) | unique
.g( EXPRESSION ) | grouping

Note that any of the call can accept optional [functional expression](#functional-expression) `EXPRESSION` that will define the behavior of a call.
If omitted the default is used which is identity (`_`). 

Example                                            | Result                       | Comment     
---------------------------------------------------|------------------------------|----------
`query([2,3,1,5,4], 's()')`                        | `[1,2,3,4,5]`                | sort
`query([2,3,1,5,4], 's(-_)')`                      | `[5,4,3,2,1]`                | sort desc
`query([{age:5},{age:1},{age:3}],'s(_.age)')`      | `[{age:1},{age:3},{age:5}]`  | sort by age
`first([{age:5},{age:1},{age:3}],'s(-_.age).age')` | `5`                          | max age
`first([{age:20,name:'John'},{age:30,name:"Peter"},'s(-_.age).name')` | `"Peter"` | oldest
`query([1,2,1,1,3,2], 'u()')`                      | `[1,2,3]`                    | unique
`query(["aa", "b", "a", "bbb", "c"], 'u(_[0])')`   | `["aa", "b", "c"]`           | unique by 1st letter
`query([1,2,1,1,3,2], 'g()')`                      | `[[1,[1,1,1]],[2,[2,2]],[3,[3]]]` | group
`first([1,2,1,1,3,2], 'g().s(-_[1].length).0')`    | `1`                          | the most popular digit
`query(["aa", "b", "a", "bbb", "c"], 'g(_[0])')`   | `[["a",["aa","a"]],["b",["b","bbb"]],["c",["c"]]]` | group by 1st letter

### Nested filtering

Nested filtering has a form `<< QUERY >>` where `QUERY` should be [jsqry query string](#query-syntax).

Nested filtering can help you in case of a query like <i>select a man who has a son older 10</i>.
If you try to achieve this using [filtering](#filtering) you realize that you need to implement some sort of a loop in `condition` part: 

```javascript
var men = [{name:"Ivan", sons:[{age:1},{age:5}]}, {name:"Peter", sons:[{age:7},{age:20}]}];

first(men, '[_.sons.filter(son=>son.age>10).length].name')
// "Peter"
``` 

Here is how the same achieved by nested filtering:
```javascript
first(men, '<<sons[_.age>10]>>.name')
// "Peter"
```

During nested filtering element is included if nested `query` for it yields a result with at least one true-like element. 

