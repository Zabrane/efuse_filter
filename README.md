efuse_filter
====

They're faster and smaller than Bloom, Cuckoo, and Xor filters.

This library is API compatible with the [exor_filter](https://github.com/mpope9/exor_filter) library, and can be used as a replacement without too much hassle.

## Table Of Contents
* [Benchmarks](#benchmarks)
* [Installation](#installation)
* [Example Usage](#example-usage)
   * [Basic Usage](#basic-usage)
   * [Elixir Example](#elixir-example)
   * [Incremental Initialization](#incremental-initialization)
* [Custom Return Values](#custom-return-values)
* [Custom Hashing](#custom-hashing)
* [Serialization](#serialization)

## Benchmarks
![Benchmark Graph](/images/results.png)

This was benchmarked with the [exor_benchmark](https://github.com/mpope9/exor_bechmark) suite that compares several Erlang and Elixir bloom / xor / fuse filter implementations. The full benchmark and results can be found there.

## Installation

This library requires Erlang version 24+.

### rebar3
```erlang
%% rebar.config

{deps, [
  %% hex.pm
  {efuse_filter, "0.1.0"},

  %% git
  {efuse_filter, {git, "git://github.com/mpope9/efuse_filter.git", {tag, "0.1.0"}}}
]}.
```

### mix
```
## mix.exs

defp deps do
  [
    {:efuse_filter, "~> 0.1.0"}
  ]
end
```

## Example Usage

### Basic Usage
```erlang
Filter = fuse8:new(["cat", "dog", "mouse"]),
true = fuse8:contain(Filter, "cat"),
false = fuse8:contain(Filter, "goose").
```

### Elixir Example
```Elixir
alias :fuse8 as: Fuse8

# ...

true =
    [1, 2, 3, 4]
    |> Fuse8.new()
    |> Fuse8.contain(1)
```

### Incremental Initialization
This builds a filter over multiple calls. Filters are immutable. Once filled, more elements cannot be added.
```erlang
Filter0 = fuse8:new_empty(),
Filter1 = fuse8:add([1, 2]),
Filter2 = fuse8:add([3, 4]),
Filter3 = fuse8:finalize(Filter2),

true = fuse8:contain(Filter3, 1),
false = fuse8:contain(Filter3, 5).
```

## Custom Return Values
`fuse8:contain/3` can return a custom value instead of `false` if the required item isn't present in the filter:

```erlang
Filter = fuse8:new(["Ricky Bobby", "Cal Naughton Jr."]),
true = fuse8:contain(Filter, "Ricky Bobby", {error, not_found}),
{error, not_found} = fuse8:contain(Filter, "Reese Bobby", {error, not_found}).
```

## Custom Hashing
By default this library uses the [`erlang:phash2/1`](https://erlang.org/doc/man/erlang.html#phash2-1) function. If you want to use your own custom hashing, pass `none` to the `fuse8:new/2` function. Values passed to `fuse8:contain/2` need to be pre-hashed as well.

```erlang
PreHashedList = [...],
Filter = fuse8:new(PreHashedList, none),
true = fuse8:contain(Filter, hd(PreHashedList)).
```

## Serialization
Functions are provided to the filter in binary form, instead of a nif reference. This can be useful to interop with other platforms / systems. The bin returned can be used with `fuse:contain` for ease of use. Example usage:

```erlang
Filter     = fuse8:new(["test1", "test2", "test3"]),
BinFilter  = fuse8:to_bin(Filter),
true       = fuse8:contain(BinFilter, "test1").
```

Build
====

```bash
$ rebar3 compile
```

Test
====

```bash
$ rebar3 eunit
$ rebar3 cover
```

Docs
====

```
$ rebar3 edoc
```
