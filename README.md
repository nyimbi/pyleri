Python Left-Right Parser
========================

Why Pyleri?
-----------
Pyleri is an easy-to-use parser create for SiriDB. We first used [lrparsing](http://lrparsing.sourceforge.net/doc/html/) and wrote [jsleri](https://github.com/transceptor-technology/jsleri) for auto-completion and suggestions in our web console. Later we found small issues in lrparsing and also had difficulties keeping the language the same in both projects. That's when we decided to create Pyleri which can export a created language to JavaScript.


Quick usage
-----------
```python
# Imports, note that we skip the imports in other examples...
from pyleri import (
    Grammar,
    Keyword,
    Regex,
    Sequence)

# Create a Grammar Class to define your language
class MyGrammar(Grammar):
    r_name = Regex('(?:"(?:[^"]*)")+')
    k_hi = Keyword('hi')
    START = Sequence(k_hi, r_name)

# Compile your grammar by creating an instance of the Grammar Class.
my_grammer = MyGrammar()

# Use the compiled grammar to parse 'strings'
print(my_grammer.parse('hi "Iris"').is_valid) # => True
print(my_grammer.parse('bye "Iris"').is_valid) # => False
```

Grammar
-------
When writing a grammar you should subclass Grammar. A Grammar expects at least a `START` property so the parser knowns where to start parsing. Grammar has some default properties which can be overwritten like `RE_KEYWORDS` and `RE_WHITESPACE`, which are both explained later. Grammer also has two methods: `parse()` and `export_js()` which are explained below.

Grammar.parse()
---------------
syntax:
```python
Grammar.parse(string)
```
The `parse()` method returns a `NodeResult` object which has the following properties:
- `expecting`: A Python set() containing elements which pyleri expects at `pos`
- `is_valid`: Boolean value, `True` when the given string is valid, `False` when not valid.
- `pos`: Position where the parser had to stop. (when `is_valid` is `True` this value will be equal to the length of the given string)
- `tree`: Contains the parse tree

Let's take the example from Quick usage.
```python
node_result = my_grammer.parse('bye "Iris"')
print(node_result.is_valid) # => False
print(node_result.expecting) # => {hi} => We expected Keyword 'hi' instead of bye 
print(node_result.pos) # => 0 => Position in the string where we are expecting the above
print(node_result.tree) # => Node object containing the parse tree
```

Grammar.export_js()
-------------------
syntax:
```python
Grammar.export_js(
    js_module_name='jsleri', 
    js_template=Grammar.JS_TEMPLATE, 
    js_identation=' ' * 4)
```
Optional keyword arguments:
- `js_module_name`: Name of the JavaScript module. (default: 'jsleri')
- `js_template`: Template String used for the export. You might want to look at the default string which can be found at Grammar.JS_TEMPLATE.
- `js_identation`: identation used in the JavaScript file. (default: 4 spaces)

For example when using our Quick usage grammar, this is the output when running `my_grammar.export_js()`:
```javascript
/* jshint newcap: false */

/*
 * This grammar is generated using the Grammar.export_js() method and
 * should be used with the jsleri JavaScript module.
 *
 * Source class: MyGrammar
 * Created at: 2015-11-04 10:06:06
 */

'use strict';

(function (
            Regex,
            Sequence,
            Keyword,
            Grammar
        ) {
    var r_name = Regex('^(?:"(?:[^"]*)")+');
    var k_hi = Keyword('hi');
    var START = Sequence(
        k_hi,
        r_name
    );

    window.MyGrammar = Grammar(START, '^\w+');

})(
    window.jsleri.Regex,
    window.jsleri.Sequence,
    window.jsleri.Keyword,
    window.jsleri.Grammar
);
```

Choice
------
syntax:
```python
Choice(Element, Element, ..., most_greedy=True)
```
The parser needs to choose between one of the given elements. Choice accepts one keyword argument `most_greedy` which is `True` by default. When `most_greedy` is set to `False` the parser will stop at the first match. When `True` the parser will try each element and returns the longest match. Settings `most_greedy` to `False` can provide some extra performance. Note that the parser will try to match each element in the exact same order they are parsed to Choice.

Example: let's use `Choice` to modify the Quick usage example to allow the string 'bye "Iris"'
```python
class MyGrammar(Grammar):
    r_name = Regex('(?:"(?:[^"]*)")+')
    k_hi = Keyword('hi')
    k_bye = Keyword('bye')
    START = Sequence(Choice(k_hi, k_bye), r_name)

my_grammer = MyGrammar()
print(my_grammer.parse('hi "Iris"').is_valid)  # => True
print(my_grammer.parse('bye "Iris"').is_valid)  # => True    
```

Sequence
--------
syntax:
```python
Sequence(Element, Element, ...)
```
The parser needs to match each element in a sequence.

Example:
```python
class TicTacToe(Grammar):
    START = Sequence(Keyword('Tic'), Keyword('Tac'), Keyword('Toe'))

my_grammer = MyGrammar()
print(my_grammer.parse('Tic Tac Toe').is_valid)  # => True
```

Keyword
-------
syntax:
```python
Keyword(string, ign_case=Fasle)
```
The parser needs to match the string. When matching keywords we need to tell the parser what characters are allowed in keywords. By default Pyleri uses `\w` which is both in Python and JavaScript equal to `[A-Za-z0-9_]`

```python
RE_KEYWORDS = re.compile('^\w+')
```

