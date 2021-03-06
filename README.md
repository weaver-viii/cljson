# cljson

Use cljson to send data between Clojure and ClojureScript applications using
JSON as the data transfer format. The cljson library has implementations for
Clojure and ClojureScript and supports all the data types that ClojureScript
supports, including tagged literals and metadata.

![perf chart][2]

## Install [![Build Status][1]][3]

Artifacts are published on [Clojars][4].

![latest version][5]

## Usage

There are two functions exported by this library: `clj->cljson` and
`cljson->clj`.  They convert Clojure data to and from JSON strings.

```clojure
user=> (require '[tailrecursion.cljson :refer [clj->cljson cljson->clj]])
nil

user=> (clj->cljson [1 2 3])
"[\"v\",1,2,3]"

user=> (cljson->clj "[\"v\",1,2,3]")
[1 2 3]

user=> (clj->cljson '(1 2 3))
"[\"l\",1,2,3]"

user=> (cljson->clj "[\"l\",1,2,3]")
(1 2 3)

user=> (clj->cljson [(java.util.Date.) {[1 2 3] :foo 'bar #{"bar"}}])
"[\"v\",[\"inst\",\"2013-06-24T17:34:04.183-00:00\"],[\"m\",[\"v\",1,2,3],[\"k\",\"foo\"],[\"y\",\"bar\"],[\"s\",\"bar\"]]]"

user=> (cljson->clj "[\"v\",[\"inst\",\"2013-06-24T17:34:04.183-00:00\"],[\"m\",[\"v\",1,2,3],[\"k\",\"foo\"],[\"y\",\"bar\"],[\"s\",\"bar\"]]]")
[#inst "2013-06-24T17:34:04.183-00:00" {[1 2 3] :foo, bar #{"bar"}}]

user> (defrecord Person [name])
user.Person
user> (clj->cljson (Person. "Bob"))
"[\"user.Person\",[\"m\",[\"k\",\"name\"],\"Bob\"]]"
user> (binding [*data-readers* {'user.Person map->Person}] (cljson->clj "[\"user.Person\",[\"m\",[\"k\",\"name\"],\"Bob\"]]"))
#user.Person{:name "Bob"}
```

### Tagged Literals

Cljson provides the `EncodeTagged` protocol which can be extended to user types
and records. This protocol is used to transform a Clojure/ClojureScript thing
into JSON-ready data.

If a type does not satisfy this protocol then cljson will use the core printer
to obtain a printed representation of the thing. If the printed representation
is a tagged literal then the data part is reread and converted to JSON-ready
data.

Reading of tagged literals is done via the normal tagged literal mechanisms
built into Clojure and ClojureScript.

Have a look at _cljson.clj_ and _cljson.cljs_ to see examples of this.

### Metadata

Bind `*print-meta*` to `true` to have metadata included in the JSON output.

### ClojureScript and Records

Unlike `clojure.core`, ClojureScript lacks a `*data-readers*` dynamic var. Instead, use [cljs.reader](https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/reader.cljs)'s `register-tag-parser!` function to declare constructors for records and types:

```clojure
(ns example
  (:require [tailrecursion.cljson :refer [clj->cljson cljson->clj]]
            [cljs.reader          :refer [register-tag-parser!]]))

(defrecord Person [name])
(register-tag-parser! "example.Person" map->Person)
(print (cljson->clj (clj->cljson (Person. "Bob")))) ;=> #example.Person{:name Bob} 
```

## License

Copyright © 2013 Alan Dipert and Micha Niskin

Distributed under the Eclipse Public License, the same as Clojure.

[1]: https://travis-ci.org/tailrecursion/cljson.png?branch=master
[2]: https://docs.google.com/a/thefreshdiet.com/spreadsheet/oimg?key=0AveuiOwXIG2PdEFRYXo0RV9YTjIwa1lPaDVNSzU1M1E&oid=5&zx=4oukjhd76v9a
[3]: https://travis-ci.org/tailrecursion/cljson
[4]: http://clojars.org/tailrecursion/cljson
[5]: http://clojars.org/tailrecursion/cljson/latest-version.svg
