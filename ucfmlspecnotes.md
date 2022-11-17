# universal config file modeling language

## goals/motivation/why?

One of the scary and difficult part of using Linux and FOSS software for casual
computer users is editing text based configuration files for programs they want
to customize.  It would be wildly impractical to try to write a grafical configuration
tool for literally every open source program out there without one.  There is also
no universal configuration file format every program can use, and I don't know if
I need to explain why [it's a bad idea to try to make one](https://xkcd.com/927/)..

Instead, I hope for ucfml to be sort of a meta config file format.  A ucfml file
should:

- Be able to adequately represent any configuration file, in any format, even
custom hacked together formats a maintainer made for a specific project and never
used elsewhere
- Not require the maintainer of any program to make **any** changes to the way
the program reads configuration files, or the way those files are formatted.
- Have a fairly simple syntax on par with markdown, html, or python, making it
a fairly easy task for the maintainer, or any user of the program familiar with
it's configuration format and options to make
- Be able to be ran with any valid and working implementation of ucfml

I also don't plan for my example implementations to be treated as a **One True
UCFML implementation**.  I want anyone to be able to make their own implementations
of UCFML with whatever bells and whistles they'd like to add, and to run on any
platform they want.  A valid ucfml implementation should be able to at the very
least

- Read a ufcml file
- Properly display all configuration options
- Produce a correct configuration file in the end

## ucfml language features and syntax

### Tag: Value based syntax
All tags in ucfml will be written in the format of

`Tag: Value`

Subtags can be written as

`Tag :
    Subtag1: "subtag value"
    Subtag2:
        Subsubtag1: "subsubtag value1"
        Subsubtag2: "subsubtag value2:`

or

`Tag : Subtag1: "subtag value"
       Subtag2: Subsubtag1: "subsubtag value1"
                Subsubtag2: "subsubtag value2:`

or more compactly as

`Tag:~:Subtag1:"subtag value":~:Subtag2:(Subsubtag1:"subsubtag value1":~:Subsubtag2:"subsubtag value2")`

or even

`Tag:~:"subtag value":~:("subsubtag value1":~:"subsubtag value2")`

the `:~:` operator takes the place of a newline and optionally allows you to omit
the tag so long as it's being used in a Tag with a known schema.  For example `
.....actually nvm I think I can make this feature work everywhere..... idk maybe

this operator doesn't have an effect within lists as thoses can be written on one
or many lines anyway, however if a list contains complicated values, then this
operator can be used to keep them on one line

### List and list preprocessing

lists can be written on one line `[ 1, 2, 3]`

or spread out
`[ 1
 , 2
 , 3
 ]`

list preprocessing

`(rep x v)` in a list will be expanded to x copies of v, for example
`[ "b", (rep 4 "i"), "g"]`
 is converted into
 `[ "b", "i", "i", "i", "i", "g"]

`(ins <List>)` in a list will inset a list into the list, for example
`[ 1, (ins [ 2, 3, 4]), 4]`
is converted into
`[ 1, 2, 3, 4, 5]`

these can also be combined remembering that `rep` changes are made before `ins` changes
`[ (rep 3 (ins [ "hi", "hello"]))]`
is converted to
`[ (ins [ "hi", "hello"]), (ins [ "hi", "hello"]), (ins [ "hi", "hello"])]
which then converts to
`[ "hi", "hello", "hi", "hello", "hi", "hello"]`

`rep` must be used in a list.  `ins` can be used in either a list or a `rep`
`ins` must take a list as input.  `rep` can take a list or a value

### conditionals

You can use conditionnals in your code by wrapping them in `{| ... |}`.
Conditionals in `ucfml` don't necessarily have to return a specific tag or value
but instead resolve to a snippet of `ucfml` code to run with in place of the entire
conditional statement. The snippets to use will be wrapped in `[| ... |]`.

There are two types of conditionals, `if` will pick only one snippet to resolve
to, and `ifmany` will pick many snippets to resolve to. All individual conditions
must be either a `Bool`, `BoolExpr`, or a `BoolLog`. Newlines inside any condition
block serve no syntactic purpose outside of the snippets, conditions can be layed
out on separate lines for clarity, or placed on one long line for brevity.  Either
kind of conditional can be nested inside of each other indefinitely if need be.

#### if

`if` statements can be written as a series of conditions followed by a snippet
to return if that condition is true and then a final else snippet if none of the
conditions are true.  If there is no final else option and none of the cases are
true then no code snippet is put in place of the condition block.  If a variable
inside one of the conditions before the one that is picked changes then the if
conditional is checked again, and the produced code snippet is changed if necessary.

`{|if <condition> then
       [|<snippet>|]
   elseif <anothercondition> then
       [|<snippet>|]
   else
       [|<snippet>|]|}`

####ifmany

`ifmany` is the same as `if`, except no `elseif` or `else`.  Since everthing is
`if` then labeling is optional and down to preference.  If any of the variable
values in the conditions, the whole condition will be checked again and the final
snippet of code is changed if need be.  Every snippet by a true condition will be
all written out in order from top to bottom.

`{|<condition>
       [|<snippet>|]
   <condition>
       [|<snippet>|]
   <condition>
       [|<snippet>|]|}`

### Expressions

#### Numeric expressions

`Expr` can be written as
`Expr:
    Left: <Expr or Num>
    Oper: <operand>
    Right: <Expr or Num>`

All `Expr` should be able to be fully evaluated to a number when all it's variables
have values assigned.

`number` can be either literally a number, or `$<refVar>` to refer to a a number
value in the config.  See  template for more information on accessing `refVar`.

`Oper` can be either +, -, *, ^, / (floating point division),
// (int dviision), % (modulo)

#### Boolean expressions

`BoolExpr` takes two `Expr` or `Num` and a comparison.  If both `Expr` can be
fully evaluated then the `BoolExpr` can be fully evaluated to a `True` or `False`.

`BoolExpr:
    Left: <Expr or Num>
    Comp: <comparison>
    Right: <Expr or Num>`

`BoolLogic` takes two `BoolExpr` or `Bool` and a `Gate`.  If both both `BoolExpr`
can be fully evaluated or are just Bool, then `BoolLogic` can be fully evaluated
to a `True` or `False`

`BoolLog:
    Left: <BoolExpr or Bool>
    Gate: <logic gate>
    Rigth: <BoolExpr or Bool>`

There are 16 possible binary boolean logic operators, all labeled as:
    - niche, trivial, but technically possible
        - `Never`: always returns false no matter the inputs
        - `Alwys`: always returns true no matter the inputs
        - idL : ignores `Right` and returns `Left`
        - NidL : ignores `Right` and returns inverse `Left`
        - idR : ignores `Left` and returns `Right`
        - NidR : ignores `Left` and returns inverse `Right`
    - the classics
        - AND : `True` if `Left` and `Right` are `True` otherwise `False`
        - NAND : `False` if `Left` and `Right` are `True` otherwise `True`
        - OR : `True` if either `Left` or `Right` are `True`, `False` otherwise
        - NOR : `False` if either `Left` or `Right` are `True`, `True` otherwise
        - XOR : `True` if `Left` and `Right` have different values, `False` if their values are the same
        - XNOR : `False` if `Left` and `Right` have different values, `True` if their values are the same
    - the oddballs
        - LnotR : `True` if `Left` is `True` and `Right` is `False`, `False` otherwise
        - NLntR : `False` if `Left` is `True` and `Right` is `False`, `True` otherwise
        - RnotL : `True` if `Right` is `True and `Left` is `False`, `False` otherwise
        - NRntL : `False` if `Right` is `True and `Left` is `False`, `True` otherwise

See also, the table below
|L|R|Never|_And_|LnotR|_idL_|RnotL|_idR_|_XOR_|_OR__|_NOR_|XNOR_|NidR_|NRntL|NidL_|NLntR|NAND_|Alwys|
|F|F|__F__|__F__|__F__|__F__|__F__|__F__|__F__|__F__|__T__|__T__|__T__|__T__|__T__|__T__|__T__|__T__|
|F|T|__F__|__F__|__F__|__F__|__T__|__T__|__T__|__T__|__F__|__F__|__F__|__F__|__T__|__T__|__T__|__T__|
|T|F|__F__|__F__|__T__|__T__|__F__|__F__|__T__|__T__|__F__|__F__|__T__|__T__|__F__|__F__|__T__|__T__|
|T|T|__F__|__T__|__F__|__T__|__F__|__T__|__F__|__T__|__F__|__T__|__F__|__T__|__F__|__T__|__F__|__T__|

LnotR is L∧¬R
RnotL is R∧¬L

### Comments

single line comments can be prefaced with two dashe (--)

`-- this is a single line comment`

multi line comics can be wrapped in {-  -}

`{- this is
 a comment that
 up multiple lines-}`

## file formatting and layout

Every `ucfml` file will have three main sections.
- `Meta`
- `Body`
- `Template`

### Meta

The header contains mostly optional general metadata about this ucfml file like:
- title to be displayed at the top of the configuration menu or to be what the
ucfml implementation sends to a windowmanager as the window title.
    - `Title: <title>`
- openining text to be rendered above the configuration options
    - `About: <intruductory text>`
- upstream documentation url
    - `UpstreamDocs: <url>`

### Body

All inputs will be represented as one of a finite set of types

- dropdown list
    - single choice finite options
    -  `Dropdown:
            OptList:
                [("Option1 label", "Option1 value")
                ,->("Option2 label", "Option2 value")
                ,("Option3 label", "Option3 value")]
            Required: <True or False>`
    - one option may be optionally prefaced with -> to indicate it as default option
- radio button
    - single choice finite options
    -  `RadioButton:
            OptList:
                [("Option1 label", "Option1 value")
                ,->("Option2 label", "Option2 value")
                ,("Option3 label", "Option3 value")]
            Required: <True or False>`
    - one option may be optionally prefaced with -> to indicate it as default option
- check boxes
    - multiple choice finite options
    -  `CheckBox:
            OptList:
                [("Option1 label", "Option1 value")
                ,->("Option2 label", "Option2 value")
                ,->("Option3 label", "Option3 value")]
            Required: <True or False>`
    - options may be optionally prefaced with -> to indicate it should be checked by default
- numerical
    - integer or floating
    - signed or unsiged
    - single choice infinite options
    -  `Number:
            IntFlo: <Floating or Int>
            Sign: <Signed or Unsigned>
            LRange: <optional LowerBound>
            URange: <optional UpperBound>
            Requited: <True or False>
            Default: <optional Default Value>
            Validator:
                <numvalidator>`
- text input
    - single choice infiinte options
    -  `TextInput:
            Require: <Ture or False>
            Default: <optional Default Value>
            Validator:
                <textvalidator>`
- compound
`   - a combination of input types ex:
        - (unsiged integer numerical, checkboxes)
        - (dropdown list, text input, check boxes)
    - note, for purpose of list inputs, a compound input type is that of its
      constituent inputs in order, ex:
        - (dropdown list, text input) is not the same as
            - (text input, dropdown list)
            - or (numerical signed floating, dropdown list)
         - `Compound:
               [ ("input one label", <type of input one>)
               , ("input two label", <type of input two>)
               ]
- list
    - a collection of inderterminate size of the same input type
        - for example, a list of keybindings or aliases
    - after selecting a value, the value is added to an adjacent list of valuess
    - more values can be added to that list,or values can be deleted, moved, edited
    - `List:
           Type: <Type>
           Default: [list, of, values, to, use, by, default]`
    - all default values can be removed from the list just like any user added values
    - default values should either match the types Validity check or be in its `OptList`

Options can be organized into labeled sections with an options title or header text

`OptSection:
    Title: <optional title text>
    About:  <options section description
    Options:
        <list of options>`

Options can also be listed without being in a OptSection with a top level tag of

`Body:
    <list of options or optsections>`

if they're relatively simple and won't benefit from sections.  If there are multiple
ungrouped `Options` tag then they should all be treated as one big option tag

Each individual option  can be written as;

`Option:
    Title: <title of option>
    About:  <description of option>
    RefVar: <a string of characters to refer to this value in template>
    Input: <type tag and values>`



#### validator

For text inputs the validator takes a list of
    `MustMatch: <regex pattern>`
    or
    `MustNotMath: <regex pattern>`

for numerical input validator takes a list of
    `MustBeTrue:
         Left: <Expr or Num>
         Comp: <Comparison sign>
         Right: <Expr or Num>`
         or
     `MustBeFalse:
         Left: <Expr or Num>
         Comp: <Comparison sign>
         Right: <Expr or Num>`

comparison sign can be either =, <,  >, <=, or >=

### template

The template at the end will consist of a list of `ConfFile`s to write the configs
to.  All `ConfFile`s can access the value of any option input with `$<options refvar>`

Individual input values in a compound or list type can be accessed with dot index
notation, ex:
    - for a compound (Number, Text, Checkbox) input with a `refVar: foo`
        - access all 3 values in parenthesis with comma separation
            - `$foo`
        - access Number, the first field
            - `$foo.0`
        - access Text, the second field
            - `$foo.1`
        - access checkbox, the last field
            - `$foo.2`
    - for a List of options with a `refVar: bar`
        - access entire list with `$bar`
        - access first value with `$bar.0`
        - access second value with `$bar.1`
        - access third value with `$bar.2`
        - access the first 5 values with `$bar.0~4`
        - access the last 3 values with `$bar.-3~-1`
    - for a list of compound (Number, Text) with `refVar: baz`
        - access entire first compound with `$baz.0`
        - access just text (second input) of 3rd compound with `$baz.2.1`

All values in a list with `refVar: biz` can be used the same way with

    `ForAll opt in $biz:
        <lines>`
The lines after ForAll can use each option with `$Opt` (or whatever label you gave)

Most programs read all their configuration from one file, and will only need one
`ConfFile`, but for the really complicated programs that read their configuration
from multiple files, you can specify multiple `ConfFile`s.

Each `ConfFile` can be written as

`ConfFile:
    FileName: <name for conf file>
    FilePath: -- optional default filepath for where to save file
        UnixStyle: </the/good/style/file/path>
        WinStyle: <C:\the\bad\style\file\path>
    Template:
        [<Line]`

The list of `Lines` under template will be written to config file from top to
bottom, a `Lines` tag can be either:


hehe fourtwenty ehehehhe
