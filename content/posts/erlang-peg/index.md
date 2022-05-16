---
title: "Create decoder for JSON-like format in Erlang. PEG and Neotoma tutorial"
cover: "cover.png"
useRelativeCover: true
date: 2017-09-11T00:00:00Z
draft: false
---

First we need to learn something about [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar). In short PEG is standard to describing a language. But it is still only a formal grammar. We need a parser. Huh, very complicated, isn’t it? So what do we need? A parser generator of course! You can use for example [Neotoma](https://github.com/seancribbs/neotoma), very good, well documented parser generator for PEG grammar. Installation instruction you can find on [project GitHub](https://github.com/seancribbs/neotoma), but you can also use [plugin for Rebar3](https://www.rebar3.org/docs/using-available-plugins#section-neotoma).

## Our Simple Object Notation

To familiarize with PEG and Neotoma we will create simple decoder (of course in last part we will create programming language) for our JSON-like format.

```
(
    some_key : "asda"
    another_key : (
        inner : "uf"
        empty : ()
    )
    not_found : null
)
```

Simple and functional.

## My first grammar

```
oson <- object ~;
 
object <- begin end `
   Node
`;
 
begin <- "(" ~;
end <- ")" ~;
```

Not so clear for first look? Read some explanation, it is really simple.

We are creating grammar for `oson`. `oson` consists of (`<-`) exactly one `object`. Object means `begin` and `end`. Begin is the open bracket character, and `end` is the `)` char.

Now, `~;` means return it as it is. Between `and` and `;` you can write Erlang code (without dot at the end). You have `Node` variable, I will explain it later. Return it as it is (`~;`) and returning `Node` mean the same.

If it is not clear for you now, read more example, I promise, creating grammar is very simple.

### Try it

```
2> neotoma:file("oson.peg").
ok
3> c(oson).
\{ok,oson}
4> oson:parse("()"). 
[<<"(">>,<<")">>]
```

- In line 2 we are creating parser from grammar.
- Line 3 – compile the parser.
- Line 4 – Use our parser. () is a oson empty object.

Hmm, list with two strings. I will try to explain this

0. oson is a object
1. object is a list of begin and end
2. begin and end are strings – brackets chars

So, maybe you can name elements in this list?

```
oson <- object ~;
 
object <- begin_object:begin end_object:end `
    Node
`;
 
begin <- "(" ~;
end <- ")" ~;
```

```
8> oson:parse("()"). 
[\{begin_object,<<"(">>},\{end_object,<<")">>}]
```

Proplist, see?

### Node

In [documentation](https://github.com/seancribbs/neotoma/wiki), you can read that

> `Node` is a list of the results from sub-expressions, which may be raw terminals or the transformations of other nonterminals.
 
So, what node is depends on context. It is hard to explain, you should start to “feel this”. If no, don’t worry, you will soon.

Remember, if you are not sure what Node is you can always print it

```
io:format("\n\n~p\n\n", [Node]),
```

### It’s time to store some data

In our oson format we will sore data as key – value.
We will start from the basics. Key will be a atom. For value only `null` allowed.

```
oson <- object ~;
 
object <- begin k:key ":" v:value end `
    Node
`;
 
key <- atom ~;
value <- null ~;
 
begin <- "(" ~;
end <- ")" ~;
atom <- [a-zA-Z0-9_]+ ~;
null <- "null" `
    null
`;
```

As you can see when someone will type `null` as a `value` we will return null atom (Erlang atom) instead of string.

Key is a similar to atom. As key you can use (I hope you know [regular expression](https://en.wikipedia.org/wiki/Regular_expression)) one or more small or big letter or digits or underscore.

Try

```
10> oson:parse("(first_key:null)").
[<<"(">>,
 \{k,[<<"f">>,<<"i">>,<<"r">>,<<"s">>,<<"t">>,<<"_">>,<<"k">>,
 <<"e">>,<<"y">>]},
 <<":">>,
 \{v,null},
 <<")">>]
```

Oops, this is not the result you can except from good parser. But let’s analyze it.

You can see here list with open object character, tuple with key (but key is not string but list of binaries, `list_to_binary/1` is the solution btw). Some colon, tuple with value (null atom as wanted). And `)`.

It is easy to clean this mess.

```
oson <- object ~;
 
object <- begin k:key ":" v:value end `
    Key = proplists:get_value(k, Node),
    Value = proplists:get_value(v, Node),
    \{object, Key, Value}
`;
 
key <- atom ~;
value <- null ~;
 
begin <- "(" ~;
end <- ")" ~;
atom <- [a-zA-Z0-9_]+ `
    list_to_binary(Node)
`;
null <- "null" `
    null
`;
```

```
17> oson:parse("(first_key:null)").
\{object,<<"first_key">>,null}

```

Much better.

But we have here still one small problem. Can you see it?

### Ignore spaces, new lines, etc...

```
Create file example1.oson

(
second_key : null
)
```

```
19> oson:file("example1.oson").
\{fail,\{expected,\{string,<<"(">>},\{\{line,1},\{column,1}}}}
```

Oops, parser expected “start object char” not new line (this file starts with new line, plugin I am using can’t show it). Solution? Simple again – create set of chars that you want to ignore and add to grammar in places when user can use it. How to ignore? Just don’t use it.

```
oson <- object ~;

object <- white begin white k:key white ":" white v:value white end white `
Key = proplists:get_value(k, Node),
Value = proplists:get_value(v, Node),
\{object, Key, Value}
`;

key <- atom ~;
value <- null ~;

begin <- "(" ~;
end <- ")" ~;
atom <- [a-zA-Z0-9_]+ `
list_to_binary(Node)
`;
null <- "null" `
null
`;
white <- [ \t\n\s\r]* ~;
```

We can use white characters in six places. Remember that better is add too much redundant `white` than forgot in one places.

```
22> oson:file("example1.oson").
\{object,<<"second_key">>,null}
```

### More objects

For now grammar accepts exactly one object with exactly one key – value pair (where value can be only null. We will fix it in next chapter).

So, how to repeat in PEG? Use brackets.

I want word “Hi” one or more time.

```
repeat <- ('Hi')+ ~;
```

```
56> repeat:parse("").
\{fail,\{expected,\{at_least_one,\{string,<<"Hi">>}},
\{\{line,1},\{column,1}}}}
57> repeat:parse("Hi").
[<<"Hi">>]
58> repeat:parse("HiHiHi").
[<<"Hi">>,<<"Hi">>,<<"Hi">>]
```

`+` means one or more, `*` means zero or more and `?` means zero or one.

So, in our object we want zero or n key-value pairs. How to do this?

```
oson <- object ~;

object <- white begin white p:(pair)* white end white `
%io:format("\n\nNode = ~p\n\n", [Node]),
PairsNode = proplists:get_value(p, Node),
%io:format("\n\nPairsNode = ~p\n\n", [PairsNode]),
Pairs = [\{proplists:get_value(k, P), proplists:get_value(v, P)} || P <- PairsNode],
\{object, Pairs}
`;

pair <- white k:key white ":" white v:value white ~;
key <- atom ~;
value <- null ~;

begin <- "(" ~;
end <- ")" ~;
atom <- [a-zA-Z0-9_]+ `
list_to_binary(Node)
`;
null <- "null" `
null
`;
white <- [ \t\n\s\r]* ~;
```

Please pay attention on how I debugged this grammar. Trick with printing Node is very useful.

```
84> oson:file("example2.oson").
\{object,[\{<<"some_key">>,null},\{<<"another_key">>,null}]}
```

### Something more than null?

Value should be null or string or another object. Simplest task in this post. But first we have to learn new symbol `/` – or.

Ok, define string

```
'"' str:[^"]* '"'
```

Every char except `"` between quotation marks. I thinks it’s the simplest way. Not perfect, but for this example enough. In your production application spend more time on this regexp and do it better.

```
oson <- object ~;

object <- white begin white p:(pair)* white end white `
PairsNode = proplists:get_value(p, Node),
Pairs = [P || P <- PairsNode],
\{object, Pairs}
`;

pair <- white k:key white ":" white v:value white `
\{proplists:get_value(k, Node), proplists:get_value(v, Node)}
`;
key <- atom ~;
value <- string / object / null ~;

begin <- "(" ~;
end <- ")" ~;
atom <- [a-zA-Z0-9_]+ `
list_to_binary(Node)
`;
null <- "null" `
null
`;
string <- '"' str:[^"]* '"' `
\{string, list_to_binary(proplists:get_value(str, Node))}
`;
white <- [ \t\n\s\r]* ~;
```

Do you see and understand all changes?

example3.oson

```
(
some_key : "asda"
another_key : (
inner : "uf"
empty : ()
)
not_found : null
)
```

```
90> oson:file("example3.oson").
\{object,[\{<<"some_key">>,\{string,<<"asda">>}},
\{<<"another_key">>,
\{object,[\{<<"inner">>,\{string,<<"uf">>}},
\{<<"empty">>,\{object,[]}}]}},
\{<<"not_found">>,null}]}
```

Perfect.

## Good job!

You made your first decoder! Now you should understand how to to create grammar, how PEG and Neotoma works. To be sure you are ready to create your own programming language you need some practice:

0. As a value you can use only string, another object or null. Add numbers.
1. Storing data without lists is not really useful. Add lists of strings (and digits and objects)
2. Now in file must be exactly one object. Allow files without objects (empty or white spaces)

[All sources available on my GitHub.](https://github.com/mateuszjarzyna/Blog/tree/master/programming%20language/oson)
