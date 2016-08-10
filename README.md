# cavia

**cavia** is a manager library of test resources in a Clojure project.

[![Clojars Project](https://img.shields.io/clojars/v/cavia.svg)](https://clojars.org/cavia)
[![Build Status](https://travis-ci.org/totakke/cavia.svg?branch=master)](https://travis-ci.org/totakke/cavia)
[![Dependency Status](https://www.versioneye.com/user/projects/54f98ad74f3108b7d2000231/badge.svg)](https://www.versioneye.com/user/projects/54f98ad74f3108b7d2000231)

In some cases, tests of a project require large-size files. Codes for parsing,
I/O, etc. should be tested by various kinds of files. But generally, SCM is not
good for controlling such large test files. One of the solutions is using other
tools like git-annex. Some Clojurians, however, may think that they want to
solve it in the Clojure ecosystem. cavia is useful for such developers. cavia is
written by Clojure so that it can be directly used in a project and source codes.
cavia downloads test resources from remotes and checks their hash before tests
and provides convenience functions to access the resources.

## Installation

cavia is available as a Maven artifact from [Clojars](http://clojars.org/cavia).

With Leiningen/Boot:

```clojure
[cavia "0.2.3"]
```

## Usage

### Define resources profile

First, load `cavia.core` and prepare resources' information with `defprofile` macro.

```Clojure
(require '[cavia.core :as cavia :refer [defprofile]])

(defprofile prof
  {:resources [;; Simple HTTP
               {:id :resource1
                :url "http://example.com/resource1"
                :sha1 "1234567890abcdefghijklmnopqrstuvwxyz1234"}
               ;; Basic authorization
               {:id :resource2
                :url "http://example.com/resource2"
                :sha1 "234567890abcdefghijklmnopqrstuvwxyz12345"
                :auth {:type :basic, :user "user", :password "password"}}
               ;; FTP
               {:id :resource3
                :url "ftp://example.com/resource3"
                :sha1 "34567890abcdefghijklmnopqrstuvwxyz123456"
                :auth {:user "user", :password "password"}}
               ;; Compressed source
               {:id :resource4
                :url "http://example.com/resource4.gz"
                :sha1 "4567890abcdefghijklmnopqrstuvwxyz1234567"
                :packed :gzip}]
   :download-to ".cavia"})
```

Resources are defined in `:resources` as a vector including some maps.
Each resource map must have `:id :url :sha1` fields. These fields are mandatory.
`:id` should be specified as keyword or string. It is used for resource access
and downloading file name.
`:auth` field is optional. It can be used for password authentication.
cavia is now supporting HTTP/HTTPS/FTP/FTPS protocols and Basic/Digest authentications.
A resource that `:packed` specified will be uncompressed after downloading.
Only gzip (`:gzip`) format is supported.

cavia downloads resources to `:download-to` directory. The default location is
`./.cavia`. Thus maybe you should add `/.cavia` to your SCM ignore list.

### Resource management

cavia provides some functions for managing resources.

```Clojure
(cavia/get! prof)   ; downloads missing resources

(cavia/verify prof) ; checks the downloaded resources' hash

(cavia/clean! prof) ; removes the download directory
```

To call cavia functions without the profile specification, use `with-profile` macro.

```Clojure
(with-profile prof
  (cavia/clean!)
  (cavia/get!))
```

`get!` and other functions output progress and logs' print to stdout.
To call the above functions quietly, use `without-print` macro.

```Clojure
(without-print
  (cavia/get! prof))
```

### Resource access

You do not need to remember the downloaded resources' paths any more.
`resource` returns the absolute path to the resource from the specified resource id.
It returns `nil` when the id is not defined.

```Clojure
(cavia/resource prof :resource1) ; returns "/home/totakke/cavia-example/.cavia/resource1"

(cavia/resource prof :undefined) ; returns nil
```

## Example usage with test frameworks

cavia is a library for management of test resources.
It is good to use cavia with test frameworks like clojure.test, [Midje](https://github.com/marick/Midje), etc.

### with clojure.test

```Clojure
(ns foo.core-test
  (:require [clojure.test :refer :all]
            [cavia.core :as cavia :refer [defprofile]]))

(defprofile prof
  {:resources [{:id :resource1
                :url "http://example.com/resource1"
                :sha1 "1234567890abcdefghijklmnopqrstuvwxyz1234"}]})

(defn fixture-cavia [f]
  (cavia/get! prof)
  (f))

(use-fixtures :once fixture-cavia)

(deftest your-test
  (testing "tests with the cavia's resource"
    (is (= (slurp (cavia/resource prof :resource1)) "resource1's content")))
```

### with Midje

```Clojure
(ns foo.t-core
  (:require [midje.sweet :refer :all]
            [cavia.core :as cavia :refer [defprofile with-profile]]))

(defprofile prof
  {:resources [{:id :resource1
                :url "http://example.com/resource1"
                :sha1 "1234567890abcdefghijklmnopqrstuvwxyz1234"}]})

(with-profile prof

  (with-state-changes [(before :facts (cavia/get!))]
    (fact "tests for a large file" :slow
      (slurp (cavia/resource :resource1) => "resource1's content")))

  )
```

## License

Copyright © 2014-2016 Toshiki Takeuchi

Distributed under the Eclipse Public License version 1.0.

## Special thanks

cavia was developed for tests of [Chrovis](https://chrov.is/).
Chrovis is a cloud service of genome analysis and visualization for researchers.
Chrovis is directed by [Xcoo, Inc.](https://xcoo.jp/)

* Xcoo: https://xcoo.jp/
* Chrovis: https://chrov.is/
