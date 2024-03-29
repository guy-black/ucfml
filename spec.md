 # Universal Config file modeling language

##  Goals and Motivation
One of the common pain points of FOSS for casual computer users  that I'm seeking to mitigate here is the textual configuration file.  I have two main goals for this language beyond simply creating a GUI for any configuration file for any program:
1.  This language should not require any changes to the main program's source code, or place any extra maintainance responsibilities on the main program's maintainer.
2.  This language should be very simple to learn, on par with markup languages like HTML.  It should be trivial for experienced developers to use, and be fairly accessible for the persistent and curious casual computer user to learn.

I will consider the goal of creating a GUI for any configuration file for any program achieved if
1.  This can accurately model any possible set of configurable options, including an ideal input format for that option, and validity checks to verify a valid input.
2.  This can accurately model any configuration file template and correctly fill in values in that template from the options modeled, leading to correctly generated configuration file in the end.

## Syntax

With the exception of comments the entire syntax  will be written in a purely declarative `Tag: Value` format.  A value can be another `Tag` or it can be a Number, Text, a Boolean, an Operator, Comparison, Gate, a tuple of values, or a list of values.  Every `Tag` either takes a single `Value`, or several `SubTag`s that can each as well either take a single `Value` or more `SubTag`s.  There are 8 types of tags: Top Layer, Meta, Body, Input, Template, Expression, Validator, Special Operators, and Conditionals.
If the value for a tag is a simple base value (ie. a literal number, text, bool, etc) then it may be written either on the same line directly following the `:` or on the next line two columns over from the start of the tag, else the value must be written on the next line two columns over.  Allowing minification or obfuscation hinders the goal of making UCFML accessible and understandable by any one regardless of their experience or comfort with programming.
In order to keep the language really easy to learn I wanted the language to be verbose without being noisey.  Writing a `ucfml` file shouldn't feel like invoking an arcane old spell word for word to do the right thing, but rather like explaining to your elderly relative how install updates to their computer.

Comments will be the same as in Haskell or Elm:
Single line comments can be prefaced with `--` ex `--this is a single line comment`
Multi Line comments can be wrapped in `{- -}`
```
{- this is
a comment
spanning many
lines -}
```

There are 7 types of base values
1. Text
2. Number
3. Bool
4. Gates
5. Operators
6. Comparisons
7. Base

two parameterized types
1. List
2. Tuples

xx complex tags composed of subtags that in turn take either a base value or another complex one
### Option Tags:
1. OptSection:
  - Title   :: Text
  - About   :: Text
  - Options :: [ Option | OptSection ]
2. Option:
  - Title :: Text
  - About :: Text
  - Input :: inputtags*
### Input Tags
3. Dropdown:
  - OptList  :: [ ( Text, abv*) ]
  - Required :: Bool
4. CheckBox:
  - OptList     :: [ ( Text, abv*) ]
  - Required    :: Bool
  - MinRequired :: Number
  - Maxrequired :: Number
5. Radio:
  - OptList  :: [ ( Text, abv*) ]
  - Required :: Bool
6. NumberInput:
  - Float     :: Bool
  - Signed    :: Bool
  - LRange    :: Number
  - URange    :: Number
  - Base      :: Base
  - Required  :: Bool
  - Default   :: Number
  - Validator :: Validator
7. TextInput
 -  MinLn     :: Number
 -  MaxLn     :: Number
 -  Required  :: Bool
 -  Defualt   :: Text
 -  Validator :: Validator
8. ListInput
 -  MinLn   :: Number
 -  MaxLn   :: Number
 -  Default :: [ x ]
 -  Input   :: inputtags (must generate value of type x)
9. CompoundInput :: [ ( Text, inputtags) ]
### Validator tags
10. MustPass:
 -  Condition :: Bool
 -  Hint      :: Text
11. MustFail:
 -  Condition :: Bool
 -  Hint      :: Text
### Expression tags
12. Expr
13. Length
14. BoolLog
15. BoolExpr
16. Regexp
17. SedExpr
18. Concat
19. Repeat
20. Combine
21. Reverse
22. ForEach
### Conditional tags
23. Cond
24. CondL
### Template tags
24. ConFile

* inputtags = any input tags
        Dropdown | CheckBox | Radio | NumberInput | TextInput | ListInput InputTag | CompoundInput [ InputTag ]
* abv = any base value type
        Text | Number | Bool | Gate | Operator | Comparison | Base

Values of any option may be acccesed by prefacing its `RefVar` value with a `$`.
For options with a comound input, `$refvar` will bring a tuple of all values, or you can access a specific value with `$refvar.inputlabel`.
For options with a list input, `$refvar` brings a list of all values.  You can access the first value with `$refvar.0`, second with `$refvar.1` third with `$refvar.2` etc. You can access the last item with `refvar.-1`, second to last with `refvar.-2`, etc.  You an access a specific range of values using `::` ex `refvar.1::3` will bring the second through fourth values, `refvar.2::-1` will bring the third value till the last value.

### Top layer Tags

There are 4 top layer tags denoting the 3 sections of each UCFML file:
1. `Define` is for defining constants and functions to be referenced later in the model
2. `Meta` covers basic information about the UCFML form.
3. `Body` will contain all the tags to represent all of the options of the UCFML form
4. `Template` will contain all the tags to represent the file(s) to be generated by the UCFML form

Example Usage:
```
Define: <definitions go here>1`

Meta: <meta tags here>

Body: <body tags here>

Template: <template tags here>
```

### Definitions

Definitions simply alias a short name for a longer value to write out.  All definitions will be listed in the form of

```
-- for a constant value
label ::= value

longval ::=
  [ "a"
  , "value"
  , "written"
  , "over'
  , "several"
  , "lines"
  ]

-- for function
fnName arg1 arg2 arg3 ::=
  <complex tags to use args in>
```

This can be used to store long values you don't want to write over and over
```
pi ::= 3.141592

breakfast ::= "spam spam spam spam baked beans spam spam spam spam and egg"
```

Or to define custom functions to avoid writing complicated tag structures
```
ftoc f ::=
  Expr:
    Left:
      Expr:
        Left: $f
        Operator: -
        Right: 32
    Operator: *
    Right:
      Expr:
        Left: 5
        Operator: /
        Right: 9
```

that function can later be used with the syntax of

```
ftoc: <farenheit temperature>
```

for a function with multiple arguments defined like

```
greater this that ::=
  Cond:
    [ (BoolExpr:
         Left: $this
         Comparison: >
         Rith: $that
      , $this)
    , (True, $that)
    ]
```

it can be called with this syntax

```
greater:
  this: <first number>
  that: <second number>
```


### Meta tags

Meta takes 4 subtags that are mostly self explanatory

1. `Title` takes a string of text to be the title of this UCFML form
2. `About` takes a text describing the UCFML form
3. `UpstreamDocs` takes a link that should lead to an upstream source of documentation about the file(s) this UCFML form is generating
4. `Author` takes a text naming the author of the file along with contact information

Example Usage:
```
Meta:
  Title: "a sample title of a ucfml form"
  About: "this is just an example of how the syntax should look for a meta tag"
  UpstreamDocs: "github.com/guy-black/"
```

### Body tags

`Body` Tag only takes one value, that is a list of `Option` and/or `OptSection` tags.

Each `OptSection` tag takes 3 subtags
1. `Title` takes a string of text to be the title of this section of options
2. `About` takes a text describing this section of options
3. `Options` takes a list of `Option` tags to represent the options that should be in this `OptSection`

Each `Option` tag takes 4 subtags
1. `Title` of this option itself
2. `About` about this option
3. `RefVar` takes the name that will be used to refer to this value in `Template` or `Expressions`.
4. `Input` takes the input tag to represent the input of this option

Example Usage:
```
Body:
  [ Option:
      Title: "sample option"
      About: "this option is not within any option section, just sitting loose"
      RefVar: fizz
      Input: <input tags here>
  , OptSection:
      Title: "sample option section"
      About: "this is how to group a section of options to gether"
      Options:
        [ Option:
            Title: "sample sectioned option"
            About: "this option sits inside of a section titled 'sample option section'"
            RefVar: buzz
            Input: <input tags here>
        , OptSetion:
            Title: "you can nest sections as much as you like"
            About: "this i a section of options nested inside a section of options
            Options:
              [ Option:
                  Title: "sample sectioned sectioned option"
                  About: "this option is nested in side of an option section thats inside of another option section"
                  RefVar: baz
                  Input: <input tags here>
              ]
        ]
  ]
```

### Input tags

There are 7 input tags available to use, 5 primary input types and 2 modifiers

#### Base input types

- `Dropdown` Takes two subtags
  1. `OptList` takes a list of tuples containing a text to label the choice, and the actual value.  One tuple may be prefaced with `->` to indicate that it should be selected by default
  2. `Required` takes a `True` or `False` if a value needs to be picked to generate a valid file in the end

  Example usage:
  ```
  Dropdown:
    OptList:
      [ ("Choice1 label", "choice2value")
      , ->("Choice2 label", "choice2value") -- this choice will be preselected by default
      , ("Choice3 label", "choice3value")
      ]
    Required: True
  ```
- `RadioButton` Takes two subtags
  1. `OptList` takes a list of tuples containing a string of text to label the choice, and the actual value.  One tuple may be prefaced with `->` to indicate that it should be selected by default
  2. `Required` takes a `True` or `False` if a value needs to be picked to generate a valid file in the end

  Example usage:
  ```
  RadioButton:
    OptList:
      [ ("Choice1 label", "choice2value")
      , ->("Choice2 label", "choice2value") -- this choice will be preselected by default
      , ("Choice3 label", "choice3value")
      ]
    Required: True
  ```
- `Checkbox` Takes two subtags
  1. `OptList` takes a list of tuples containing a string of text to label the choice, and the actual value. Tuples may be prefaced with `->` to indicate that it should be selected by default
  2. `MinRequired` takes a number for the minimum amount of options that must be selected to make a valid file or may be left blank for no minimum
  3. `MaxAllowed` takes a number or the max amount of options that may be selected or left blank for no max

  Example usage:
  ```
  Checkbox:
    OptList:
      [ ("Choice1 label", "choice2value")
      , ->("Choice2 label", "choice2value") -- this choice will be preselected by default
      , ->("Choice3 label", "choice3value") -- this choice will be preselected by default
      ]
    MinRequired: 1
    MaxRequired:
  ```
- `Number` takes 8 subtags
  1. `Float` takes a boolean, `True` if this is a floating point number, or `False` if this is an integer
  2. `Signed` takes a boolean, `True` if the number can be negative, or `False` if this number must be positive
  3. `LRange` can take a number to indicate a lower bound, or may be left blank to indicate no lower bound
  4. `URange` can take a number to indicate an upper bound, or may be left blank to indicate no upper bound
  5. `Base` can be either `Bin`, `Oct`, `Dec`, `Hex` depending on what sort of number to expect, if left blank then defaults to Dec
  6. `Required` takes a `True` or `False` if a value needs to be picked to generate a valid file in the end
  7. `Default` can take a number for default value or may be left blank for no default
  8. `Validator` can take a list of `validator` tags or can be left blank for no validator

  Example usage:
  A number input that takes a positive integer between 69 and 420 inclusively and doesn't need to have a value to generate a valid file
  ```
  Number:
    Float: False
    Signed: False
    LRange: 69
    URange: 420
    Base: Dec
    Required: False
    Default:
    Validator: <validator tag here>

- `TextInput` takes 5 subtags
  1. `MinLn` can take a positive number to indicate a minimum length the text must be for a valid input or left blank or set to 0 for no minimum
  2. `MaxLn` can take a positive number to indicate a maximum length the text must be for a valid input or left blank for no maximum
  3. `Required` takes a `True` or `False` if a value needs to be set to generate a valid file in the end
  4. `Default` can take a string of text for a default value or can be left blank for no default
  5. `Validator` can take a list of `Validator` tags or can be left blank for no validator

  Example usage:
  A text input that takes only one character, is required to produce a valid file, and has * as the default value
  ```
  TextInput:
    MinLn: 1
    MaxLn: 1
    Required: True
    Defualt: "*"
    Validator:

  ```
#### Input modifiers

- `Compound` combines multiple closely related inputs into one compund input sharing the some option.  The `Compound` tag only takes one value; a list of tuples of labels and Inputs to combine.

   Example usage
   A `TextInput` labled "Name" with no maximum length, a minimum length of 1, and required to have a value; combined with an optional number input labeled "Age" that takes any positive integer
  ```
  Compound:
    [ ( "Name"
      , TexInput:
          MinLn: 1
          MaxLn:
          Required: True
          Default:
          Validator:)
    , ( "Age"
      , Number:
          Float: False
          Signed: False
          LRange:
          URange:
          Base: Dec
          Required: False
          Default:
          Validator:)
    ]
  ```
- `List` is used to denote that an input can be selected from many times and the option should allow adding and removing to the list.  `List` takes 4 subtags
  1. `MinLn` takes a positive integer to indicate the minimum amount of selections the list must have to generate a valid output file, or may be left blank or 0 to indicate no minimum
  2. `MaxLn` takes a positive integer to indicate the maximum amount of selections the list must have to generate a valid output file, or may be left blank to indicate no maximum
  3. `Default` takes a list of values of type produced by `Input` below to list as the default values
h  4. `Input` takes an input tag for whatever input you want a list of values from

  Example usage
  A list of `Compound` input type that combines a single character text input labeled "hotkey" and another text input labeled "command to run".  The list features the letter "l" bound to "ls" and "c" bound to "cd" by default.  the list has no minimum length but a maximum of 10.
  ```
  List:
    MinLn:
    MaxLn: 10
    Default: [("l", "ls"), ("c", "cd")]
    Input:
      Compound:
        [ ( "hotkey"
          , TexInput:
              MinLn: 1
              MaxLn: 1
              Default:
              Required: True
              Validator:)
        , ( "command to run"
          , TextInput:
              MinLn:
              MaxLn:
              Default:
              Required: True
              Validator:)
        ]
  ```

### Template tags

The `Template` tag takes a list of `ConFile` subtags, one for each file to be generated.

Each `ConFile` tag takes 3 subtags.

  1. `FileName` takes a text value for the default file name for the file to be generated, or may be left blank for no default file name
  2. `NixPath` takes a text value for a unix style file path of where the file should be saved by default.  May be left blank for no default
  3. `WinPath` takes a text value for a windows style file path of where the file should be saved by default.  May be left blank for no default
  4. `Lines` takes a list of text values to be written out to to the output file, in the order they should be written in.

Example
output a file named "simpledoc.md" that writes the values in an option with the `RefVar`s of "title" and "body"
```
Template:
  [ ConFile:
      FileName: "simpledoc.md"
      NixPath:
      WinPath:
      Lines:
        [ ""
        , Concat: ["#" , $title]
        , $body
        ]
  ]
```

### Expression tags

Note if an expression has variables that haven't been assigned yet then the output will also be not assigned yet

Expressions will use special values called `Operator`s, `Comparison`s, and `Gates`

`Operator`s refer to simple arithmetic operations and can be one of:
  - `+` (addition)
  - `-` (subtraction)
  - `*` (multiplication)
  - `/` (division)
  - `//` (integer division)
  - `%` (modulus, or remainder from integer division)
  - `^` (exponent)

`Comparison`s can be one of
  - `=` (equal)
  - `!=` (not equal)
  - `>` (greater than)
  - `<` (less than)
  - `>=` (greater than or equal)
  - `<=` (less than or equal)

`Gates` covers all possible combinations of two input one output boolean logic gates
  - the classics
    - `AND`  : `True` if `Left` and `Right` are `True` otherwise `False`
    - `NAND` : `False` if `Left` and `Right` are `True` otherwise `True`
    - `OR`   : `True` if either `Left` or `Right` are `True`, `False` otherwise
    - `NOR`  : `False` if either `Left` or `Right` are `True`, `True` otherwise
    - `XOR`  : `True` if `Left` and `Right` have different values, `False` if their values are the same
    - `XNOR` : `False` if `Left` and `Right` have different values, `True` if their values are the same
  - niche, trivial, but technically possible
    - `Never`: always returns false no matter the inputs
    - `Alwys`: always returns true no matter the inputs
    - `idL`  : ignores `Right` and returns `Left`
    - `NidL` : ignores `Right` and returns inverse `Left`
    - `idR`  : ignores `Left` and returns `Right`
    - `NidR` : ignores `Left` and returns inverse `Right`
  - the oddballs
    - `LnotR`: `True` if `Left` is `True` and `Right` is `False`, `False` otherwise
    - `NLntR`: `False` if `Left` is `True` and `Right` is `False`, `True` otherwise
    - `RnotL`: `True` if `Right` is `True and `Left` is `False`, `False` otherwise
    - `NRntL`: `False` if `Right` is `True and `Left` is `False`, `True` otherwise

see also, this table

|Left |Right|Never| And |LnotR| idL |RnotL| idR | XOR | OR  | NOR |XNOR |NidR |NRntL|NidL |NLntR|NAND |Alwys|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|False|False|False|False|False|False|False|False|False|False|True |True |True |True |True |True |True |True |
|False|True |False|False|False|False|True |True |True |True |False|False|False|False|True |True |True |True |
|True |False|False|False|True |True |False|False|True |True |False|False|True |True |False|False|True |True |
|True |True |False|True |False|True |False|True |False|True |False|True |False|True |False|True |False|True |

There are four main groups of expression types based on whether they resolve to a number, bool, text., or list of value.

#### Expressions that resolve to numbers

`Expr` takes 3 subtags and resolves to a new number.  An `Expr` can be given as a value to any tag that takes a number value

  1. `Left` takes a number value to be on the left side of the expression
  2. `Operator` takes an operator to use on the two numbers
  3. `Right` takes a number value to be on the right side of the expression

  Example usage:
  add the numbers from two options, one with the `refVar` baz and the other boz
  ```
  Expr:
    Left: $baz
    Operator: +
    Right: $boz
  ```

`Length` takes only one value that is either a text value or a List of values, and resolves to the length of that value

  Example usage:
  ```
  Length: "a text value" -- resolves to 12
  Length:
    [ "a"
    , "list"
    , "of"
    , "values"
    ] -- resolves to 4
  ```

#### Expressions that resolve to boolean

`BoolExpr` takes 3 subtags and resolves to a new bool.  A `BoolExpr` can be given as a value to any tag that takes a bool value

  1. `Left` takes a number value to be on the left side of the expression
  2. `Comparison` take a comparison to use to compare the two values
  3. `Right` takes a number value to be on the right side of the expression

  Example usage:
  check to see if the value labeled "height" from the compound input of an option with the `refVar` of defSize is greater than 12
  ```
  BoolExpr:
    Left: $defSize."height"
    Comparison: >
    Right: 12
  ```

`BoolLog` takes 3 subtags and resolves to a new bool.  A `BoolLog` can be given as a value to any tag that takes a bool value

  1. `Left` takes a bool value to be on the left side of the expression
  2. `Gate` take a gate to use to compare the two values
  3. `Right` takes a bool value to be on the right side of the expression

  Example usage:
  checks the input on options with the `refVar`s foo and bar to  make sure one but not both are true.
  ```
  BoolLog:
    Left: $foo
    Gate: XOR
    Right: $bar
  ```
`RegExp` takes 2 subtags and resolves to a new bool.  A `RegExp` can be given as a value to any tag that takes a bool value

  1. `Source` takes the string of text being checked
  2. `Regex` takes a string of text for the regular expression the text is being checked against

  Example usage:
  checks that the text in an option with the `refVar` name starts with a capitalized letter
  ```
  RegExp:
    Source: $name
    Regex: "/(^[A-Z])/"
  ```


#### Expressions that resolve to text

`SedExp` takes 2 subtags and resolves to a new text.  A `SedExp` can be given as a value to any tag that takes a text value

  1. `Source` takes the string of text being modified
  2. `SedCmd` takes a string of text for the Sed commands to process the text with

  Example usage
  ensure that there is only one white space separating each word in the text value in the option with the `refVar` startcmds
  ```
  SedExp:
    Source: $startcmds
    SedCmd: "s/ +/ /g"
  ```

`Concat` takes a list of text values and resolves to the combined text value

Example usage
a `Lines` tag with a list of two text values, one has the value of the option with `RefVar` of "width" embedded, and the other with the value of "height"
```
Lines:
  [ Concat: ["width: ", $width," pixels"]
  , Concat: ["height: ", $height, " pixels"]
  ]
```

#### Expressions that resolve to a list of values

`Repeat` can be used anywhere that expects a list giving a list of a given length of a given value and takes two subtags
  1. `Len` takes a number value for how many of `Val` to have in the list
  2. `Val` takes any value to fill the list with

Example usage
A `Default` tag for a `List` input that takes a list of the word "yes" 5 times
```
Default:
  Repeat:
    Len: 5
    Val: "yes"
```

`Combine` takes a list of lists and smashes them down to a single list

Example usage 1
Concatenating multiple lists together
```
Lines:
  Combine:
    [ $listone
    , $listtwo]
```
Example usage 2
Inserting a list of values into a list a certain point
```
Lines:
  Combine:
    [ ["user names:"]
    , $usernames
    , ["user ids"]
    , $userids ]
```

`Reverse` reverses a list.  It can be used anywhere a list is expected, and takes one list value
Example usage:
```
Lines:
  Reverse:
    [ "Third"
    , "Second"
    , "First" ]
```

`ForEach can be used anywhere a list is expected to modify the values of a list before using it, it takes 3 subtags:
  1. `Label` will be used similarly to `RefVar` to refer to each value in the list given in the final value
  2. `Input` will takes a list of values to be modified
  3. `NewVal` will take the value or tag to be generated from the input value.  `$<Label>` will reference the list value here

Example usage 1
Take a list of number values and generate a list of the squares of all the number values
```
ForEach:
  Label: val
  Input: $listOfNumbers
  NewVal:
    Expr:
      Left: $val
      Operator: ^
      Right: 2
```

Example usage 2
Take a list of users and create an `OptSection` with `Options` for each one
```
OptSection:
  Title: "user info"
  About: "info for all the previously listed users"
  Options:
    ForEach:
      Label: name
      Input: $ListOfNames
      NewVal:
        Option:
          Title: $name
          About:
            Concat: [ "Information about ", $name]
          RefVar:
            Concat: [ $name, "info"]
          Input:
            TextInput:
              MinLn:
              MaxLn:
              Required: False
              Default:
              Validator:
```
The values for the generated options can then be accessed with
```
ForEach:
  Label: name
  Input: $ListOfNames
  NewVal: $(Concat: [$name, "info"])
```



### Validator tags

The `Validator` subtag in `Number` and `TextInput` takes a list of subtags `MustBe` or `MustNot`, which each take two subtags

   1. `Condition` takes a bool value, either directly or from `BoolExpr`, `BoolLog`, or `RegExp`
   2. `Hint` takes a text value to show the user if the validator fails to help them fix it

Example usage
An option with the `refVar` of "user" with a compound input combining a `Number` labeled "id" and a `TextInput` labeled "name" where "id" must be divisible by 6 but can't be 18, and name must not start with a lower case letter.
```
Option:
  Title: "user info"
  About: "username and id number
  RefVar: user
  Input:
    Compound:
      [ ( "name"
        , TextInput:
            MinLn: 1
            MaxLn:
            Required: True
            Default:
            Validator:
              [ MustFail:
                  Condition:
                    RegExp:
                      Source: $user."name"
                      Regex: "/([^a-z])/"
                  Hint: "name must not start with a lower case letter"
              ])
      , ( "id"
        , Number:
            Float: False
            Signed: False
            Lrange: 0
            Urange: 1000
            Base:
            Required:True
            Default:
            Validator
              [ MustPass:
                  Condition:
                    BoolExpr:
                      Left:
                        Expr:
                          Left: $user."id"
                          Operator: %
                          Right: 6
                      Comparison: =
                      Right: 0
                  Hint: "id must be divisible by 6"
              , MustFail:
                  Condition:
                    BoolExpr:
                      Left: $user."id"
                      Comparison: =
                      Right: 18
                  Hint: "id must not be 18"
              ])
      ]
```

### Conditionals

Conditionals may be used anywhere within a `Meta:`, `Body:`, or `Template:` tag or their subtags, wheneve the correct value or tag is dependent on another value.  There is `Cond` for a single determined value and `CondL` to generate a list of values.  They both take a single value, a list of tuples of `Bool` and the value to resolve to if the Bool is true.  `Cond` will resolve to the first value with Bool of True, if no values are True then no value will be picked and the tag will this is a value to will be considered unassigned.  `CondL` will resolve to a list of all values paired with a Bool of True, or an empty list if there are no values paired with True.

Example usage 1
A Cond that will resolve down to a single text value of "Kid", "Teen", or "Adult" based on the value of an `Option` with a `refVar` of "age"
```
Cond:
  [ ( BoolExpr:
        Left: $age
        Comparison: <
        Right: 13
    , "Kid" )
  , ( BoolExpr:
        Left: $age
        Comparison: >=
        Right: 18
    , "Adult" )
  , (True, "Teen") ]
```
Example usage 2
A CondL that will resolve to a list of text values of true statements about the number value of an option with the`refVar` of "num"
```
CondL:
  [ ( BoolExpr:
        Left:
          Expr:
            Left: $num
            Operator: %
            Right: 2
        Comparison: =
        Right: 0
    , "num is even" )
  , ( BoolExpr:
        Left:
          Expr:
            Left: $num
            Operator: %
            Right: 2
        Comparison: !=
        Right: 0
    , "num is odd" )
  , ( BoolExpr:
        Left: $num
        Comparison: =
        Right: 0
    , "num is zero" )
  , ( BoolExpr:
        Left: $num
        Comparison: >
        Right: 0
    , "num is a positive nummber greater than zero" )
  , ( BoolExpr:
        Left: $num
        Comparison: <
        Right: 0
    , "num is a negative number" ) ]
```
