# pegjs-backtrace [![npm version](https://badge.fury.io/js/pegjs-backtrace.svg)](https://badge.fury.io/js/pegjs-backtrace)
<img src="https://nodei.co/npm/pegjs-backtrace.png?downloads=true&stars=true" alt=""/>

A tracer module for [PEG.js](http://pegjs.org/) which generates a visual backtrace tree.

This module also supports normal tracing output. See [showTrace](#showTrace) option for detail.

# Example
The following example grammar recognizes arithmetic expressions like `2*(3+4)`.

```
start = additive

additive = multiplicative plus additive / multiplicative

multiplicative = primary mult multiplicative / primary

primary = integer / "(" additive ")"

integer = digits:[0-9]+

plus = "+"

mult = "*"
```

If you give the `2*(3/4)` which the grammar does not recognize, pegjs-backtrace shows backtrace tree from the maximum failure positions to the start rule as follows.

<img src="img/backtrace.png" width="400">

# Install

```
npm install pegjs-backtrace
```

# Usage

pegjs-backtrace is implemented as a [Tracer](https://github.com/pegjs/pegjs/commit/da57118a43a904f753d44d407994cf0b36358adc). When calling the `parse` function, pass the pegjs-backtrace instance to the `tracer` option.
Then, after parser fails, you can obtain the backtrace string from `getBacktraceString()`.

```
var Parser = require('./parser'); // parser generated by pegjs --trace
var Tracer = require('pegjs-backtrace');

var text = '2*(3/4)';
var tracer = new Tracer(text); // input text is required.

try {
    Parser.parse(text,{tracer:tracer});
} catch(e) {
	console.log(tracer.getBacktraceString());
}
```
Note that `--trace` option is required to generated parser.js with pegjs command. If the option is not supplied, tracer feature is disabled.

# Options
When creating pegjs-backtrace instance, you can provide some options as follows.

```
var Tracer = require('pegjs-backtrace');
var tracer = new Tracer(text,{
  parent:null,
  hiddenPaths:[],
  useColor:true,
  showTrace:false,
  maxSourceLines:6,
  maxPathLength:72,
});
```

## parent
This option specifies a parent Tracer instance. Once the option is given, the parent's `trace` method is also called during parsing. The default value is `null`.
```
var Parser = require('./parser');
var Tracer = require('pegjs-backtrace');
var tracer = new Tracer(text,{
  parent:new Parser.DefaultTracer(),
});
```

## useColor
If true, the output is colored with ANSI escape sequence. Otherwise no escape sequence is used. The default value is `true`.

## showSource
If true, quoted source are shown on the trace nodes. Otherwise source is suppressed. The default value is `true`.

## maxSourceLines
The maximum number of lines shown as the quoted source on each trace node. The default value is `6`.

## <a name="showTrace"></a>showTrace 
If true, trace log is printed while parsing. The default value is `false`.

<img src="img/trace.png" width="400">

## showFullPath
If the options is true, the rule path is shown instead of the rule name. The default value is `false`.

<img src="img/path.png" width="90%">

## maxPathLength
The limit length of the displaying path name. When the path length exeeds to the limit, the path is truncated.

## hiddenPaths
This option specifies path patterns to hide. Any node that matches one of these patterns will be hidden from the trace log.
**Note that this option works only with trace log, not for backtrace.** The default value is `[]`.

```
var Tracer = require('pegjs-backtrace');
var tracer = new Tracer(text,{
  hiddenPaths:["integer", "primary/.*"]
});
```
The type of the pattern must be `string` or `RegExp`. Even the pattern is `string`, it may also contain RegExp meta characters.
The `/` character can be used to represent the hierarchcal path of grammar rules. If the pattern is string like `"FOO"`, it is treated as the regular expression `/(^|\/)FOO(\/|$)/`.

# Limitation
The cursor symbol `^` may points the wrong position if the prefix of the source line contains East Asian full-width Characters.

The generating tree is based on the observable trace events from the pegjs parser. Any local failure inside a grammar rule is ignored. For example, if parser fails while
reading `"+"` character in the following `additive` rule, we can observe the event that `additive` fails but cannot get any information of `"+"` fails.
```
additive = multiplicative "+" additive / multiplicative
```
If you would like to see the event on `"+"` failure, the `"+"` should be an explicit rule like:
```
plus = "+"
additive = multiplicative plus additive / multiplicative
```
