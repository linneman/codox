# Codox

A tool for generating API documentation from Clojure or ClojureScript
source code.


## Usage

### Leiningen

Include the following plugin in your `project.clj` file or your global
profile:

```clojure
:plugins [[lein-codox "0.10.3"]]
```

Then run:

```
lein codox
```

This will generate API documentation in the "target/doc" subdirectory
(or wherever your project `:target-path` is set to).

### Boot

Add boot-codox to your build.boot dependencies and require the namespace:
```
(set-env! :dependencies '[[boot-codox "0.10.3" :scope "test"]])
(require '[codox.boot :refer [codox]])
```

You can see the options available on the command line:

```
$ boot codox -h
```

or in the REPL:

```
boot.user=> (doc codox)
```

Remember to output files to the target directory with boot's built-in `target` task:

```
$ boot codox target
```

## Breaking Changes in 0.9

In preparation for a 1.0 release, Codox 0.9 has a number of breaking
changes:

* The Leiningen plugin has been changed from `codox` to `lein-codox`
* The Leiningen task has been changed from `lein doc` to `lein codox`
* The default output path has been changed from `doc` to `target/doc`
* The `:sources` option has been renamed to `:source-paths`
* The `:output-dir` option has been renamed to `:output-path`
* The `:defaults` option has been renamed to `:metadata`
* The `:include` and `:exclude` options have been replaced with `:namespaces`
* All the `:src-*` options have been replaced with `:source-uri`

See the "Source Files" section for information on the `:namespaces`
option, and the "Source Links" section for information on the
`:source-uri` option.


## Examples

Some examples of API docs generated by Codox in real projects:

* [Medley](https://weavejester.github.io/medley/medley.core.html)
* [Compojure](https://weavejester.github.com/compojure/)
* [Hiccup](https://weavejester.github.com/hiccup/)
* [Ring](https://ring-clojure.github.com/ring/)


## AOT Compilation

AOT-compiled namespaces will lose their metadata, which mean you'll
lose documentation for namespaces. Avoid having global `:aot`
directives in your project; instead, place them in a specialized
profile, such as `:uberjar`.


## Project Options

Codox can generate documentation from Clojure or ClojureScript. By
default it looks for Clojure source files, but you can change this to
ClojureScript by setting the `:language` key:

```clojure
:codox {:language :clojurescript}
```

It is also possible to inject externally generated documentation data structures
as edn file. This is e.g. requrired to integrate documentation from 3rd party
languages such as Pixie which is quite similar to clojure but Pixie code
can neither be parsed by Clojure nor ClojureScript.

```clojure
:codox {:inject-ns-edn-file <edn file name>}
```
The following snippet gives an example about the strucute of injected edn data:

```clojure
[{:name 'server.app.core
    :doc "documentation for server.app.tstnamespace"
    :author "ol"
    :publics [{:name 'function
               :file "/Users/ol/Entwicklung/clojure/pub-oss/src/pub_oss/core.clj"
               :line 100
               :type :var
               :arglists [['a 'b 'c]]
               :doc "this is a function"
               :members []}]}
   {:name 'server.app.test
    :doc "documentation for server.app.tstnamespace"
    :author "ol"
    :publics [{:name 'test-function
               :file "/Users/ol/Entwicklung/clojure/pub-oss/src/pub_oss/core-test.clj"
               :line 100
               :type :var
               :arglists [['a 'b 'c 'd]]
               :doc "this is a test function"
               :members []}]}]
```

Furthermore the contents of the installation instructions section can be overwriten
by:

```clojure
:codox {:installation (list [:h2 "Installation] ...}
```

### Source Files

By default Codox looks for source files in the `:source-paths` of your
project, but you can change this just for Codox by placing the
following in your `project.clj` file:

```clojure
:codox {:source-paths ["path/to/source"]}
```

The `:namespaces` option can be used to restrict the documentation to
a specific set of namespaces:

```clojure
:codox {:namespaces [library.core library.io]}
```

Regular expressions can also be used for more general matching:

```clojure
:codox {:namespaces [#"^library\."]}
```

For excluding only internal namespaces, it's sometimes useful to use
negative lookahead:

```clojure
:codox {:namespaces [#"^library\.(?!internal)"]}
```

To override the namespaces list and include all namespaces, use `:all`
(the default):

```clojure
:codox {:namespaces :all}
```

The `:exclude-vars` option can be used to exclude vars that match a
regular expression. Set to `nil` to disable. By default vars generated
by record constructor functions are excluded (such as `->Foo` and
`map->Foo`):

```clojure
:codox {:exclude-vars #"^(map)?->\p{Upper}"}
```

Codox constructs documentation from metadata on vars and namespaces.
You can specify a set of default metadata using the `:metadata` map:

```clojure
:codox {:metadata {:doc "FIXME: write docs"}}
```

### Documentation Files

As well as source files, Codox also tries to include documentation
files as well. By default it looks for these in the `doc` directory,
but you can change this with:

```clojure
:codox {:doc-paths ["path/to/docs"]}
```

Documentation files will appear in the output sorted by their
filename. If you want a particular order, one solution is to prefix
your files with `01`, `02`, etc. Alternatively, you can also define
the documentation files explicitly:

```clojure
:codox {:doc-files ["doc/intro.md", "doc/tutorial.md"]}
```

If `:doc-files` is specifies, then `:doc-paths` is ignored. Currently
only markdown files (`.md` or `.markdown`) are supported. Any links
between markdown files will be converted to their HTML equivalents
automatically.


### Output Files

To write output to a directory other than the default, use the
`:output-path` key:

```clojure
:codox {:output-path "codox"}
```

To use a different output writer, specify the fully qualified symbol of the
writer function in the `:writer` key:

```clojure
:codox {:writer codox.writer.html/write-docs}
```

By default the writer will include the project name, version and
description in the output. You can override these by specifying a
`:project` map in your Codox configuration:

```clojure
:codox {:project {:name "Example", :version "1.0", :description "N/A"}}
```

### Source Links

If you have the source available at a URI and would like to have links
to the function's source file in the documentation, you can set the
`:source-uri` key:

```clojure
:codox {:source-uri "https://github.com/foo/bar/blob/{version}/{filepath}#L{line}"}
```

The URI is a template that may contain the following keys:

* `{filepath}`  - the file path from the root of the repository
* `{basename}`  - the basename of the file
* `{classpath}` - the relative path of the file within the source directory
* `{line}`      - the line number of the source file
* `{version}`   - the version of the project

You can also assign different URI templates to different paths of your
source tree. This is particularly useful for created source links from
generated source code, such as is the case with [cljx][].

For example, perhaps your Clojure source files are generated in
`target/classes`. To link back to the original .cljx file, you could do:

```clojure
:codox
{:source-uri
 {#"target/classes" "https://github.com/foo/bar/blob/master/src/{classpath}x#L{line}"
  #".*"             "https://github.com/foo/bar/blob/master/{filepath}#L{line}"}}
```

[cljx]: https://github.com/lynaghk/cljx

### HTML Transformations

The HTML writer can be customized using [Enlive][]-style
transformations. You can use these to modify the HTML documents
produced in arbitrary ways, but the most common use is to add in new
stylesheets or scripts.

The transforms live in the `:transforms` key, in the `:html` map, and
consist of a vector that matches selectors to transformations, in the
same way that `let` matches symbols to values.

For example, the following code adds a new `<script>` element as the
last child of the `<head>` element:

```clojure
:html {:transforms [[:head] [:append [:script "console.log('foo');"]]]}
```

The selectors follow the [Enlive selector syntax][].

The transformations are a little different. There are five transforms,
`:append`, `:prepend`, `:after`, `:before` and `:substitute`. These
match to the corresponding [Enlive transformations][], but expect
[Hiccup][]-style arguments.

[enlive]: https://github.com/cgrand/enlive
[enlive selector syntax]: https://github.com/cgrand/enlive#selector-syntax
[enlive transformations]: https://github.com/cgrand/enlive#transformations
[hiccup]: https://github.com/weavejester/hiccup

### HTML Output Options

The HTML writer also has one other customization option.

By default the namespace list is nested, unless there is only one
namespace in the library. To override this, set the `:namespace-list`
option in the `:html` map to either `:nested` or `:flat`.

```clojure
:html {:namespace-list :flat}
```

### Themes

Themes are HTML transformations packaged with resources. Because
they're data-driven and based on transformation of the generated
documentation, multiple themes can be applied. The default theme is
`:default`. Themes can be added by changing the `:themes` key:

```clojure
:themes [:my-custom-theme]
```

To create a theme, you'll need to place the following resource in the
classpath, either directly in your project, or via a dependency:

    codox/theme/my-custom-theme/theme.edn

This edn file should contain a map of two keys: `:transforms` and
`:resources`.

The `:transforms` key contains an ordered collection of HTML
transformations. See the previous section for more information on the
syntax.

The `:resources` key contains a list of sub-resources that will be
copied to the target directory when the documentation is compiled. For
example, if you define a sub-resource `css/main.css`, then Codox will
copy the resource `codox/theme/foo/css/main.css` to the file
`css/main.css` in the target directory.

Themes can also take parameters. You can put in a keyword as a
placeholder, and then end users can specify the value that should
replace the keyword. This is achieved by using a vector instead of a
keyword to specify the theme:

```clojure
:themes [[keyword {placeholder value}]]
```

For example:

```clojure
:themes [[:my-custom-theme {:some-value "foobar"}]]
```

Codox will look for the keyword `:some-value` in the theme file, and
replace it with the string `"foobar"`.

If you want to take a look at a complete theme, try the
[default theme][] for Codox.

[default theme]: https://github.com/weavejester/codox/tree/master/codox/resources/codox/theme/default


## Metadata Options

To force Codox to skip a public var, add `:no-doc true`
to the var's metadata. For example:

```clojure
;; Documented
(defn square
  "Squares the supplied number."
  [x]
  (* x x))

;; Not documented
(defn ^:no-doc hidden-square
  "Squares the supplied number."
  [x]
  (* x x))
```

You can also skip namespaces by adding `:no-doc true` to the
namespace's metadata. *This currently only works for Clojure code, not
ClojureScript.* For example:

```clojure
(ns ^:no-doc hidden-ns)
```

To denote the library version the var was added in, use the `:added`
metadata key:

```clojure
(defn square
  "Squares the supplied number."
  {:added "1.0"}
  [x]
  (* x x))
```

Similar, deprecated vars can be denoted with the `:deprecated`
metadata key:

```clojure
(defn square
  "Squares the supplied number."
  {:deprecated "2.0"}
  [x]
  (* x x))
```


## Docstring Formats

By default, docstrings are rendered by Codox as fixed-width plain
text, as they would be on a terminal. However, you can override this
behavior by specifying an explicit format for your docstrings.

Currently there are only two formats for docstrings: plaintext and
markdown. The markdown format includes extensions for code blocks,
tables, and, like the plaintext format, URLs are automatically encoded
as links.

You can specify docstring formats via a var's metadata, using the
`:doc/format` option:

```clojure
(defn foo
  "A **markdown** formatted docstring."
  {:doc/format :markdown}
  [x])
```

Or you can specify the docstring format globally by adding it to the
`:metadata` map in your project.clj file:

```clojure
:codox {:metadata {:doc/format :markdown}}
```

Markdown docstrings also support wikilink-style relative links, for
referencing other vars. Vars in the current namespace will be matched
first, and then Codox will try and find a best match out of all the
vars its documenting.

```clojure
(defn bar
  "See [[foo]] and [[user/square]] for other examples."
  {:doc/format :markdown}
  [x])
```


## Live Documentation

You can make the code in your documentation live and interactive by
using the [Klipse theme][] written by [Yehonathan Sharvit][]. This
third-party theme integrates the generated docs with the [Klipse][]
code evaluator.

[klipse theme]: https://github.com/viebel/codox-klipse-theme
[yehonathan sharvit]: https://github.com/viebel
[klipse]: https://github.com/viebel/klipse


## License

Copyright © 2017 James Reeves

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
