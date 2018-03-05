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
`query([1,2,3,4,5], '[ _>6 ]')`              | `[]`    |

```javascript
jsqry.first(target, query[, ...args])
```

Same as `jsqry.query` described above but returns first item of result list or `null` in case of empty result.

Example                                      | Result
---------------------------------------------|-------
`first([1,2,3,4,5], '[ _>? && _<? ]', 2, 5)` | `3`   
`first([1,2,3,4,5], '[ _>6 ]')`              | `null`   

## Query syntax

Query in general can have a form below

```
field1.field2[ CONDITION or INDEX or SLICE ].field3{ TRANSFORMATION }.field4<< QUERY >>.field5.x( KEYEXPR )
```

Here:

part                      | meaning                                             
--------------------------|-----------------------------------------------
`field1.field2.field3...` | [fields access](#field-access), same as in JS
`[ CONDITION ]`           | [filtering](#filtering)                             
`[ INDEX ]`               | [index access](#indexing), same as in JS            
`[ FROM:TO:STEP ]`        | [slices](#slicing), Python-style                    
`{ TRANSFORMATION }`      | [object transformation](#transformation)            
`.x( KEYEXPR )`           | [call action](#calls)                               
`<< QUERY >>`             | [nested filtering](#nested-filtering)               

*Note: all mentioned query elements are optional and can be combined in arbitrary order.*

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

In fact it's very simple. 
Every expression during execution by jsqry is substituted to a function this way: 

from         | to                                                    
-------------|--------------------------------------
`EXPRESSION` | `function(_,i) { return EXPRESSION }`

(here **_** — the value of item, **i** — it's index).

This function is then applied to the elements being queried.

Examples of filtering:

Example                                                 | Result          | Comment
--------------------------------------------------------|-----------------|---------------------
`query([1,2,3,4,5], '[ _ % 2 == 1 ]')`                  | `[1,3,5]`       | odd elements
`query([{a:1},{a:2},{a:3}], '[_.a>=2]')`                | `[{a:2},{a:3}]` |
`query(["a", "bb", "aaa", "c"], '[_.startsWith("a")]')` | `["a","aaa"]`   |
`query(["a", "bb", "aaa", "c"], '[_.length>1]')`        | `["bb","aaa"]`  |
`query(["",1,null,"B",undefined,333,false], '[_]')`     | `[1, "B", 333]` | only true-like items

Examples using index argument **i**:

Example                             | Result        | Comment               
------------------------------------|---------------|-------------------------
`query([1,2,3,4,5,6], '[ i % 2 ]')` | `[2,4,6]`     | Take every 2nd element *
`query([1,2,3,4,5,6], '[ i > 0 ]')` | `[2,3,4,5,6]` | Omit first element *    

\* - same result is achievable by [slicing](#slicing): `[1::2]` and `[1:]`. 
 
### Indexing

Same to JS with addition: index can be negative (meaning index from the end).

Example                                        | Result    | Comment     
-----------------------------------------------|-----------|-------------
`query(["a", "bb", "aaa", "c"], '[2]')`        | `["aaa"]` |
`first(["a", "bb", "aaa", "c"], '[2]')`        | `"aaa"`   |
`first(["a", "bb", "aaa", "c"], '[2].length')` | `3`       |
`first(["a", "bb", "aaa", "c"], '[-1]')`       | `"c"`     | last element

### Slicing

This is very similar to [Python's slicing](https://www.dotnetperls.com/slice-python).

Has form of `[ FROM:TO:STEP ]`. Any of `FROM` / `TO` / `STEP` is optional. 

Example                        | Result        | Comment     
-------------------------------|---------------|--------------------
`query([1,2,3,4,5], '[::2]')`  | `[1,3,5]`     | Every other element
`query([1,2,3,4,5], '[1:]')`   | `[2,3,4,5]`   | All but 1st item
`query([1,2,3,4,5], '[:-1]')`  | `[1,2,3,4]`   | All but last item
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

At the moment these are supported:

call          | description
--------------|-------------
.s( KEYEXPR ) | sorting
.u( KEYEXPR ) | unique
.g( KEYEXPR ) | grouping

Note that any of the call can accept optional [functional expression](#functional-expression) `KEYEXPR` that will define the behavior of a call.
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

#### Custom calls

You can define your own call handler `myCall` by implementing the function for `jsqry.fn.myCall`.

Let's implement as an example the `partition` function of [lodash](https://lodash.com/) which splits an array into two according to a function

```javascript
_.partition([1, 2, 3, 4], n => n % 2);
// → [[1, 3], [2, 4]]
```

Here is how you do this in jsqry using custom call:

```javascript
jsqry.fn.partition = function (pairs, res) {
    var trueElts = [];
    var falseElts = [];
    res.push(trueElts, falseElts);
    for (var i = 0; i < pairs.length; i++) {
        var pair = pairs[i];
        var e = pair[0]; // input element
        var v = pair[1]; // function result for it
        if (v)
            trueElts.push(e);
        else
            falseElts.push(e);
    }
};

query([1, 2, 3, 4], 'partition( _ % 2 )')
// → [[1, 3], [2, 4]]
```

### Nested filtering

Nested filtering has a form `<< QUERY >>` where `QUERY` should be [jsqry query string](#query-syntax).

Nested filtering can help you in case of a query like <i>select a parent who has a child older 10</i>.
If you try to achieve this using [filtering](#filtering) you realize that you need to implement some sort of a loop in `condition` part: 

```javascript
var parents = [{name:"John", children:[{age:1},{age:5}]}, {name:"Alice", children:[{age:7},{age:12}]}];

first(parents, '[_.children.filter(child=>child.age>10).length].name')
// → "Alice"
``` 

And here is how we can do the same much simpler using nested filtering:

```javascript
first(parents, '<<children[_.age>10]>>.name')
// → "Alice"
```

During nested filtering element is included if nested `query` for it yields a result with at least one true-like element. 

### Flatting

This will help you if you have array of arrays and you want to query the inner elements.

You need to use `*` [path element](#field-access). 

Example                                       | Result                        | Comment     
----------------------------------------------|-------------------------------|--------------------
`query([["a", "bb"], ["cccc"]],'*')`          | `["a", "bb", "cccc"]`         |
`query([["a", "bb"], ["cccc"]],'*.length')`   | `[1, 2, 4]`                   |
`query([["a", "bb"], ["cccc"]],'length')`     | `[2, 1]`                      | Compare with previous
`query([["a", "bb"], ["cccc",["dd"]]],'*')`   | `["a", "bb", "cccc", ["dd"]]` |
`query([["a", "bb"], ["cccc",["dd"]]],'*.*')` | `["a", "bb", "cccc", "dd"]`   |

### More examples

Here are some interesting results you can achieve with jsqry if you twist it a bit :-)

Zip:
```javascript
query(['a', 'b', 'c', 'd'], '{[_,?[i]]}', ['A', 'B', 'C', 'D'])
// → [['a','A'],['b', 'B'],['c', 'C'],['d', 'D']]

query(['a', 'b', 'c', 'd'], '{ [_, ?[i], ?[i]] }', ['A', 'B', 'C', 'D'], ['AA', 'BB', 'CC', 'DD'])
// → [['a','A', 'AA'],['b', 'B', 'BB'],['c', 'C', 'CC'],['d', 'D', 'DD']]
```

Enumerate:
```javascript
query(['a', 'b', 'c', 'd'], '{[i,_]}')
// → [[0, 'a'], [1, 'b'], [2, 'c'], [3, 'd']]

query(Array(26), '{String.fromCharCode(i+97)}').join('')
// → 'abcdefghijklmnopqrstuvwxyz'
```

Difference:
```javascript
query([1, 2, 1, 0, 3, 1, 4], '[?.indexOf(_)<0]', [0, 1])
// → [2, 3, 4]
```

Union:
```javascript
query([[1, 2, 3], [101, 2, 1, 10], [2, 1]], '*.u()')
// → [1, 2, 3, 101, 10]
```

Famous [FizzBuzz](https://rosettacode.org/wiki/FizzBuzz) program:
```javascript
query(Array(20), '{ i++, i % 15 == 0 ?? "FizzBuzz" : i % 3 == 0 ?? "Fizz" : i % 5 == 0 ?? "Buzz" : i }')
// → [1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "FizzBuzz", 16, 17, "Fizz", 19, "Buzz"]             
```

