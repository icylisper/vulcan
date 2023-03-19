Vulcan
======

Tool to simplify workflow with Git-based Clojure dependencies using
clojure.tools.deps

-   upgrade to latest SHA corresponding to release tags
-   flatten dependencies
-   diff upstream versions of Git deps
-   link to local projects
-   generate the next semantic version tag
-   pack dependencies to run without uberjar
-   find dependencies that have conflicting namespaces
-   run test selectors with proper exit code
-   import new deps in the REPL

Install
=======

Add below alias to \~/.clojure/deps.edn

``` {.clojure}
{:aliases
 {:vulcan
  {:extra-deps
   {icylisper/vulcan
    {:git/url "https://git.sr.ht/~icylisper/vulcan",
     :sha "d7ed0f85893a2b5054720b0e191c606dfaf45c9c"}},
   :main-opts ["-m" "vulcan.main"],
   :jvm-opts
   ["-client"
    "-XX:+UseSerialGC"
    "-XX:CICompilerCount=1"
    "-XX:+TieredCompilation"
    "-Xshare:off"
    "-Xverify:none"
    "-XX:TieredStopAtLevel=1"]}}}
```

Usage
=====

``` {.bash}
clj -Avulcan command args
Available commands:
  upgrade    Upgrade deps to latest tags for given org or prefix
  diff       Show diff of current and upstream tags for repos with given prefix
  pack       Packs Git and Maven dependencies
  classpath  Print the Pack classpath
  conflicts  Find overlapping or conflicting namespaces for given org or prefix
  next-tag   Generate the next semantic version tag
  self-update Update vulcan itself to latest master SHA in ~/.clojure/deps.edn
  test       Run test-runner with given selector
  link       link local git repos
  unlink     unlink local git repos
```

Upgrading Dependencies
----------------------

The assumption is that Git tags are SEMVERs and correspond to releases.
\`-Avulcan upgrade\` upgrades dependencies to the latest release tags
for the given organization or prefix. This is particularly useful in
Continuous integrations and local development.

``` {.bash}
clj -Avulcan upgrade -p <github-org>
e.g clj -Avulcan upgrade -p myorg
"wrote deps.edn"

# where prefix is the Organization prefix

# for a dry run
clj -Avulcan upgrade -p <github-org> --dry-run
```

Flattening dependencies
-----------------------

To upgrade git dependencies to their latest tags and flatten out their
dependencies, do

``` {.bash}
clj -Avulcan upgrade -p <github-org> --flatten
# or short form
clj -Avulcan upgrade -p <github-org> -f
```

This is useful where top-level repos could contain a flattened deps map
for overrides and identifying stale dependencies. Flatten command also
prints the dependents (parents) of the dependencies. For example:

``` {.clojure}
{:deps
 {instaparse/instaparse
  {:mvn/version "1.4.0",
   :exclusions #{org.clojure/clojure},
   :dependents [clout/clout]},
  io.aviso/pretty
  {:mvn/version "0.1.30", :dependents [com.taoensso/timbre]}}}
```

Diffing upstream versions
-------------------------

To find the latest and current versions (tags) for given org

``` {.bash}
clj -Avulcan diff -p <github-org>
# output should be something like this
{foo/bar
 {"0.1.6" "2018-03-05T03:46:54.000+0000",
  "0.1.7" "2018-03-07T01:50:08.000+0000"},
```

Linking to local projects
-------------------------

Sometimes, it is useful to "link" local projects that have local changes
- similar to **lein checkout**. To get the similar behavior do

Link assumes that the project we're linking is one directory above the
vulkan directory.

``` {.bash}
clj -Avulcan link --project <org/project>
# or
clj -Avulcan link -p <org/project>
```

Unlinking is easier too. Make sure you unlink before comitting deps.edn
to git

``` {.bash}
clj -Avulcan unlink -p <org/project>
```

Packing Dependencies
--------------------

To pack all Maven and Git dependencies into a single directory, do

``` {.bash}
clj -Avulcan pack
```

The above command packs all deps to ./lib/{git,jar}. We could tar,
containerize and deploy Also **pack** generates a .classpath file that
contains the resolved classpath string that can be used when invoking
the service

``` {.bash}
java -cp src:`cat .classpath` clojure.main -m my.main $@
```

Finding Conflict
----------------

To find overlapping or conflicting namespaces for given org (or prefix)

``` {.bash}
clj -Avulcan conflicts -p github-org

The following projects duplicate the namespace foo.bar
foo-dep foo.bar
bar-dep foo.bar
```

Generate next-tag
-----------------

``` {.bash}
clj -Avulcan  next-tag
0.1.0
```

For this to work, need to create a RELEASE-0.1.0 tag initially

Test selectors
--------------

``` {.bash}
clj -Avulcan test -s unit
clj -Avulcan test -s integration
```

This is useful to run tests with proper exit codes

Importing libraries in the REPL
-------------------------------

``` {.clojure}
(require '[vulcan.deps :as deps])
;; to import known libs in current deps.edn
(deps/import! :my-git-lib :latest)
(deps/import! :my-git-lib "0.1.40")
;; to try a new library not in deps.edn
(deps/import! '(hiccup {:mvn/version "0.1.0"})
(deps/import! '{org/project
                 {:git/url "git@github.com:org/project.git",
                  :tag "0.1.98"}})
```

Author: Isaac Praveen
