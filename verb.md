## Quickstart

```js
var mm = require('micromatch');
mm(list, patterns[, options]);
```

The [main export](#micromatch) takes a list of strings and one or more glob patterns:

```js
console.log(mm(['foo', 'bar', 'qux'], ['f*', 'b*'])); 
//=> ['foo', 'bar']
```

Use [.isMatch()](#ismatch) to get true/false:

```js
console.log(mm.isMatch('foo', 'f*'));  
//=> true
```

[Switching](#switching-to-micromatch) from minimatch and multimatch is easy!


## Why use micromatch?

> micromatch is a [drop-in replacement](#switching-to-micromatch) for minimatch and multimatch

- Supports all of the same matching features as [minimatch][] and [multimatch][]
- Micromatch uses [snapdragon][] for parsing and compiling globs, which provides granular control over the entire conversion process in a way that is easy to understand, reason about, and maintain.
- More consistently accurate matching [than minimatch](https://github.com/yarnpkg/yarn/pull/3339), with more than 36,000 [test assertions](./test) to prove it. 
- More complete support for the Bash 4.3 specification than minimatch and multimatch. In fact, micromatch passes _all of the spec tests_ from bash, including some that bash still fails.
- [Faster matching](#benchmarks), from a combination of optimized glob patterns, faster algorithms, and regex caching.
- [Micromatch is safer][braces]{#braces-is-safe}, and is not subject to DoS with brace patterns, like minimatch and multimatch. 
- More reliable windows support than minimatch and multimatch. 

### Matching features

* Support for multiple glob patterns (no need for wrappers like multimatch)
* Wildcards (`**`, `*.js`)
* Negation (`'!a/*.js'`, `'*!(b).js']`)
* [extglobs][extglob] (`+(x|y)`, `!(a|b)`)
* [POSIX character classes][brackets] (`[[:alpha:][:digit:]]`)
* [brace expansion][braces] (`foo/{1..5}.md`, `bar/{a,b,c}.js`)
* regex character classes (`foo-[1-5].js`)
* regex logical "or" (`foo/(abc|xyz).js`)

You can mix and match these features to create whatever patterns you need!

## Switching to micromatch

There is one notable difference between micromatch and minimatch in regards to how backslashes are handled. See [the notes about backslashes](#backslashes) for more information.

### From minimatch

Use [mm.isMatch()](#ismatch) instead of `minimatch()`:

```js
mm.isMatch('foo', 'b*');
//=> false
```

Use [mm.match()](#match) instead of `minimatch.match()`:

```js
mm.match(['foo', 'bar'], 'b*');
//=> 'bar'
```

### From multimatch

Same signature:

```js
mm(['foo', 'bar', 'baz'], ['f*', '*z']);
//=> ['foo', 'baz']
```

## API
{%= apidocs("index.js") %}

## Options

- [basename](#optionsbasename) 
- [bash](#optionsbash) 
- [cache](#optionscache) 
- [dot](#optionsdot) 
- [failglob](#optionsfailglob) 
- [ignore](#optionsignore) 
- [matchBase](#optionsmatchBase) 
- [nobrace](#optionsnobrace) 
- [nocase](#optionsnocase) 
- [nodupes](#optionsnodupes) 
- [noext](#optionsnoext) 
- [noglobstar](#optionsnoglobstar) 
- [nonull](#optionsnonull) 
- [nullglob](#optionsnullglob) 
- [snapdragon](#optionssnapdragon) 
- [sourcemap](#optionssourcemap) 
- [unescape](#optionsunescape) 
- [unixify](#optionsunixify) 

### options.basename

Allow glob patterns without slashes to match a file path based on its basename. Same behavior as [minimatch][] option `matchBase`.

**Type**: `Boolean`

**Default**: `false`

**Example**

```js
mm(['a/b.js', 'a/c.md'], '*.js');
//=> []

mm(['a/b.js', 'a/c.md'], '*.js', {matchBase: true});
//=> ['a/b.js']
```

### options.bash

Enabled by default, this option enforces bash-like behavior with stars immediately following a bracket expression. Bash bracket expressions are similar to regex character classes, but unlike regex, a star following a bracket expression **does not repeat the bracketed characters**. Instead, the star is treated the same as an other star.

**Type**: `Boolean`

**Default**: `true` 

**Example**

```js
var files = ['abc', 'ajz'];
console.log(mm(files, '[a-c]*'));
//=> ['abc', 'ajz']

console.log(mm(files, '[a-c]*', {bash: false}));
```

### options.cache

Disable regex and function memoization. 

**Type**: `Boolean`

**Default**: `undefined`


### options.dot

Match dotfiles. Same behavior as [minimatch][] option `dot`.

**Type**: `Boolean`

**Default**: `false`


### options.failglob

Similar to the `--failglob` behavior in Bash, throws an error when no matches are found.

**Type**: `Boolean`

**Default**: `undefined`


### options.ignore

String or array of glob patterns to match files to ignore.

**Type**: `String|Array`

**Default**: `undefined`


### options.matchBase

Alias for [options.basename](#options-basename).


### options.nobrace

Disable expansion of brace patterns. Same behavior as [minimatch][] option `nobrace`.

**Type**: `Boolean`

**Default**: `undefined`

See [braces][] for more information about extended brace expansion.


### options.nocase

Use a case-insensitive regex for matching files. Same behavior as [minimatch][].

**Type**: `Boolean`

**Default**: `undefined`


### options.nodupes

Remove duplicate elements from the result array.

**Type**: `Boolean`

**Default**: `undefined`

**Example**

Example of using the `unescape` and `nodupes` options together:

```js
mm.match(['a/b/c', 'a/b/c'], 'a/b/c');
//=> ['a/b/c', 'a/b/c']

mm.match(['a/b/c', 'a/b/c'], 'a/b/c', {nodupes: true});
//=> ['abc']
```

### options.noext

Disable extglob support, so that extglobs are regarded as literal characters.

**Type**: `Boolean`

**Default**: `undefined`

**Examples**

```js
mm(['a/z', 'a/b', 'a/!(z)'], 'a/!(z)');
//=> ['a/b', 'a/!(z)']

mm(['a/z', 'a/b', 'a/!(z)'], 'a/!(z)', {noext: true});
//=> ['a/!(z)'] (matches only as literal characters)
```


### options.nonegate

Disallow negation (`!`) patterns, and treat leading `!` as a literal character to match.

**Type**: `Boolean`

**Default**: `undefined`


### options.noglobstar

Disable matching with globstars (`**`).

**Type**: `Boolean`

**Default**: `undefined`

```js
mm(['a/b', 'a/b/c', 'a/b/c/d'], 'a/**');
//=> ['a/b', 'a/b/c', 'a/b/c/d']

mm(['a/b', 'a/b/c', 'a/b/c/d'], 'a/**', {noglobstar: true});
//=> ['a/b']
```

### options.nonull

Alias for [options.nullglob](#options-nullglob).


### options.nullglob

If `true`, when no matches are found the actual (arrayified) glob pattern is returned instead of an empty array. Same behavior as [minimatch][] option `nonull`.

**Type**: `Boolean`

**Default**: `undefined`


### options.snapdragon

Pass your own instance of [snapdragon][], to customize parsers or compilers.

**Type**: `Object`

**Default**: `undefined`


### options.sourcemap

Generate a source map by enabling the `sourcemap` option with the `.parse`, `.compile`, or `.create` methods.

_(Note that sourcemaps are currently not enabled for brace patterns)_

**Examples**

``` js
var mm = require('micromatch');
var pattern = '*(*(of*(a)x)z)';

var res = mm.create('abc/*.js', {sourcemap: true});
console.log(res.map);
// { version: 3,
//   sources: [ 'string' ],
//   names: [],
//   mappings: 'AAAA,GAAG,EAAC,iBAAC,EAAC,EAAE',
//   sourcesContent: [ 'abc/*.js' ] }

var ast = mm.parse('abc/**/*.js');
var res = mm.compile(ast, {sourcemap: true});
console.log(res.map);
// { version: 3,
//   sources: [ 'string' ],
//   names: [],
//   mappings: 'AAAA,GAAG,EAAC,2BAAE,EAAC,iBAAC,EAAC,EAAE',
//   sourcesContent: [ 'abc/**/*.js' ] }

var ast = mm.parse(pattern);
var res = mm.compile(ast, {sourcemap: true});
console.log(res.map);
// { version: 3,
//   sources: [ 'string' ],
//   names: [],
//   mappings: 'AAAA,CAAE,CAAE,EAAE,CAAE,CAAC,EAAC,CAAC,EAAC,CAAC,EAAC',
//   sourcesContent: [ '*(*(of*(a)x)z)' ] }
```

### options.unescape

Remove backslashes from returned matches.

**Type**: `Boolean`

**Default**: `undefined`

**Example**

In this example we want to match a literal `*`:

```js
mm.match(['abc', 'a\\*c'], 'a\\*c');
//=> ['a\\*c']

mm.match(['abc', 'a\\*c'], 'a\\*c', {unescape: true});
//=> ['a*c']
```

### options.unixify

Convert path separators on returned files to posix/unix-style forward slashes.

**Type**: `Boolean`

**Default**: `true` on windows, `false` everywhere else 

**Example**

```js
mm.match(['a\\b\\c'], 'a/**');
//=> ['a/b/c']

mm.match(['a\\b\\c'], {unixify: false});
//=> ['a\\b\\c']
```

## Extended globbing

Micromatch also supports extended globbing features.

### extglobs

Extended globbing, as described by the bash man page:

| **pattern** | **regex equivalent** | **description** |
| --- | --- | --- |
| `?(pattern)` | `(pattern)?` | Matches zero or one occurrence of the given patterns |
| `*(pattern)` | `(pattern)*` | Matches zero or more occurrences of the given patterns |
| `+(pattern)` | `(pattern)+` | Matches one or more occurrences of the given patterns |
| `@(pattern)` | `(pattern)` <sup>*</sup> | Matches one of the given patterns |
| `!(pattern)` | N/A (equivalent regex is much more complicated) | Matches anything except one of the given patterns |

<sup><strong>*</strong></sup> Note that `@` isn't a RegEx character.

Powered by [extglob][]. Visit that library for the full range of options or to report extglob related issues.

### braces

Brace patterns can be used to match specific ranges or sets of characters. For example, the pattern `*/{1..3}/*` would match any of following strings:

```
foo/1/bar
foo/2/bar
foo/3/bar
baz/1/qux
baz/2/qux
baz/3/qux
```

Visit [braces][] to see the full range of features and options related to brace expansion, or to create brace matching or expansion related issues.

### regex character classes

Given the list: `['a.js', 'b.js', 'c.js', 'd.js', 'E.js']`:

* `[ac].js`: matches both `a` and `c`, returning `['a.js', 'c.js']`
* `[b-d].js`: matches from `b` to `d`, returning `['b.js', 'c.js', 'd.js']`
* `[b-d].js`: matches from `b` to `d`, returning `['b.js', 'c.js', 'd.js']`
* `a/[A-Z].js`: matches and uppercase letter, returning `['a/E.md']`

Learn about [regex character classes][charclass].

### regex groups

Given `['a.js', 'b.js', 'c.js', 'd.js', 'E.js']`:

* `(a|c).js`: would match either `a` or `c`, returning `['a.js', 'c.js']`
* `(b|d).js`: would match either `b` or `d`, returning `['b.js', 'd.js']`
* `(b|[A-Z]).js`: would match either `b` or an uppercase letter, returning `['b.js', 'E.js']`

As with regex, parens can be nested, so patterns like `((a|b)|c)/b` will work. Although brace expansion might be friendlier to use, depending on preference.

### POSIX bracket expressions

POSIX brackets are intended to be more user-friendly than regex character classes. This of course is in the eye of the beholder. 

**Example**

```js
mm.isMatch('a1', '[[:alpha:][:digit:]]');
//=> true

mm.isMatch('a1', '[[:alpha:][:alpha:]]');
//=> false
```

See [expand-brackets][] for more information about bracket expressions.

***


## Notes

### Bash 4.3 parity

Whenever possible matching behavior is based on behavior Bash 4.3, which is mostly consistent with minimatch. 

However, it's suprising how many edge cases and rabbit holes there are with glob matching, and since there is no real glob specification, and micromatch is more accurate than both Bash and minimatch, there are cases where best-guesses were made for behavior. In a few cases where Bash had no answers, we used wildmatch (used by git) as a fallback. 

### Backslashes

There is an important, notable difference between minimatch and micromatch _in regards to how backslashes are handled_ in glob patterns.

- Micromatch exclusively and explicitly reserves backslashes for escaping characters in a glob pattern, even on windows. This is consistent with bash behavior.
- Minimatch converts all backslashes to forward slashes, which means you can't use backslashes to escape any characters in your glob patterns. 

We made this decision for micromatch for a couple of reasons:

- consistency with bash conventions. 
- glob patterns are not filepaths. They are a type of [regular language][regular-language] that is converted to a JavaScript regular expression. Thus, when forward slashes are defined in a glob pattern, the resulting regular expression will match windows or POSIX path separators just fine. 

**A note about joining paths to globs**

Note that when you pass something like `path.join('foo', '*')` to micromatch, you are creating a filepath and expecting it to still work as a glob pattern. This causes problems on windows, since the `path.sep` is `\\`.

In other words, since `\\` is reserved as an escape character in globs, on windows `path.join('foo', '*')` would result in `foo\\*`, which tells micromatch to match `*` as a literal character. This is the same behavior as bash.

## Contributing

All contributions are welcome! Please read [the contributing guide](.github/contributing.md) to get started.

**Bug reports**

Please create an issue if you encounter a bug or matching behavior that doesn't seem correct. If you find a matching-related issue, please:

- [research existing issues first](../../issues) (open and closed)
- visit the [GNU Bash documentation][bash] to see how Bash deals with the pattern
- visit the [minimatch][] documentation to cross-check expected behavior in node.js
- if all else fails, since there is no real specification for globs we will probably need to discuss expected behavior and decide how to resolve it. which means any detail you can provide to help with this discussion would be greatly appreciated.

**Platform issues**

It's important to us that micromatch work consistently on all platforms. If you encounter any platform-specific matching or path related issues, please let us know (pull requests are also greatly appreciated).

## Benchmarks

### Running benchmarks

Install dev dependencies:

```bash
npm i -d && npm run benchmark
```

### Latest results

As of {%= date() %} (longer bars are better):

```sh
{%= benchmarks("benchmark/stats.json") %}
```

[regular-language]: https://en.wikipedia.org/wiki/Regular_language
[bash]: https://www.gnu.org/software/bash/manual/
[charclass]: http://www.regular-expressions.info/charclass.html
[extended]: http://mywiki.wooledge.org/BashGuide/Patterns#Extended_Globs
[brackets]: https://github.com/micromatch/expand-brackets
[braces]: https://github.com/micromatch/braces
