# `cloben` [![Build Status](https://travis-ci.org/sgraf812/cloben.svg?branch=master)](https://travis-ci.org/sgraf812/cloben)

`cloben` is a Haskell shell script transforms `cabal bench`/`stack bench` results into a CSV file readable by `gipeda` for visualization.
Prior to that, it optionally clones a specific commit of a given git repository into a temporary folder in which it then performs the benchmarking.

It parses build warnings and timing data output in the standard `criterion` format.

# Usage

There are two modes of operation:

`cloben` will benchmark the cabal project in the current working directory and output the CSV data on `stdout`. It will try to use `stack` as an optimization, but the fall back mechanism (cabal sandboxing) takes 10 minutes on my laptop for even the simplest dependency tree, so stay calm :).

`cloben <repo> <commit>` will also attempt to recursively clone a remote git `repo` at a specific `commit` into a temporary directory and `cd` into it prior to benchmarking.

For usage with `gipeda`, `stdout` should be redirected into a csv file.

See also `cloben --help`.

# How to build

Simplest way? Don't! Use [`stack`s excellent support for `runghc`](http://docs.haskellstack.org/en/stable/GUIDE.html#script-interpreter):
```
$ stack cloben.hs
```
Or on unixoid systems:
```
$ chmod +x cloben.hs
$ ./cloben.hs
```

Of course, `cloben` can be built both in a `cabal` and in a `stack` environment.
```
$ stack build && stack exec cloben
...
$ cabal install -j && cabal run
...
```

Or even just through hackage:

```
$ stack install cloben && cloben
...
$ cabal install cloben && cloben
...
```

# Example

What running `cloben` on the `Pipes` library yielded:

```
$ ./cloben.hs https://github.com/Gabriel439/Haskell-Pipes-Library 930c834aacfa7bf8ec65d072e0d0a982aa7a2bc1 > logs/930c834aacfa7bf8ec65d072e0d0a982aa7a2bc1.csv
$ cat logs/930c834aacfa7bf8ec65d072e0d0a982aa7a2bc1.csv
build/warnings;81.0
benchmark/prelude-benchmarks/Folds/all;152900.0
benchmark/prelude-benchmarks/Folds/any;163700.0
benchmark/prelude-benchmarks/Folds/find;170800.0
benchmark/prelude-benchmarks/Folds/findIndex;164500.0
benchmark/prelude-benchmarks/Folds/fold;68150.0
benchmark/prelude-benchmarks/Folds/foldM;67690.0
benchmark/prelude-benchmarks/Folds/head;10.84
benchmark/prelude-benchmarks/Folds/index;117000.0
benchmark/prelude-benchmarks/Folds/last;119600.0
benchmark/prelude-benchmarks/Folds/length;53960.0
benchmark/prelude-benchmarks/Folds/null;11.2
benchmark/prelude-benchmarks/Folds/toList;138100.0
benchmark/prelude-benchmarks/Pipes/chain;1351000.0
benchmark/prelude-benchmarks/Pipes/drop;110400.0
benchmark/prelude-benchmarks/Pipes/dropWhile;159000.0
benchmark/prelude-benchmarks/Pipes/filter;585100.0
benchmark/prelude-benchmarks/Pipes/findIndices;397900.0
benchmark/prelude-benchmarks/Pipes/map;324600.0
benchmark/prelude-benchmarks/Pipes/mapM;1276000.0
benchmark/prelude-benchmarks/Pipes/take;346500.0
benchmark/prelude-benchmarks/Pipes/takeWhile;332400.0
benchmark/prelude-benchmarks/Pipes/scan;370900.0
benchmark/prelude-benchmarks/Pipes/scanM;1177000.0
benchmark/prelude-benchmarks/Pipes/concat;159600.0
benchmark/prelude-benchmarks/Zips/zip;1218000.0
benchmark/prelude-benchmarks/Zips/zipWith;1318000.0
benchmark/prelude-benchmarks/enumFromTo.vs.each/enumFromTo;205700.0
benchmark/prelude-benchmarks/enumFromTo.vs.each/each;209000.0
benchmark/lift-benchmarks/ReaderT/runReaderP_B;4791000.0
benchmark/lift-benchmarks/ReaderT/runReaderP_A;266600.0
benchmark/lift-benchmarks/StateT/runStateP_B;4912000.0
benchmark/lift-benchmarks/StateT/runStateP_A;344800.0
benchmark/lift-benchmarks/StateT/evalStateP_B;5534000.0
benchmark/lift-benchmarks/StateT/evalStateP_A;349400.0
benchmark/lift-benchmarks/StateT/execStateP_B;5451000.0
benchmark/lift-benchmarks/StateT/execStateP_A;324300.0
```

# Which `cabal`, `stack` and `ghc` binaries are used?

Short answer: That found on the path at the time of executing this script.

If `cloben` is compiled to an executable which is then called from your shell,
it will use `cabal`, `stack` and `ghc` binaries found on your path.

If however you prefer to run this as a script through `stack` (see above),
note that `stack` modifies the environment, so it will use a 'stack-local' version
of `cabal` when `stack install cabal-install` happened.

Finally, note that the fallback mechanism (try `stack bench`, fall back to `cabal sandbox init && cabal bench`)
might fail if it can't find `ghc` on the path.
