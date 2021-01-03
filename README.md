Fast Ruby [![Build Status](https://travis-ci.org/JuanitoFatas/fast-ruby.svg?branch=travis)](https://travis-ci.org/JuanitoFatas/fast-ruby)
=======================================================================================================================================================================

In [Erik Michaels-Ober](https://github.com/sferik)'s great talk, 'Writing Fast Ruby': [Video @ Baruco 2014](https://www.youtube.com/watch?v=fGFM_UrSp70), [Slide](https://speakerdeck.com/sferik/writing-fast-ruby), he presented us with many idioms that lead to faster running Ruby code. He inspired me to document these to let more people know. I try to link to real commits so people can see that this can really have benefits in the real world. **This does not mean you can always blindly replace one with another. It depends on the context (e.g. `gsub` versus `tr`). Friendly reminder: Use with caution!**

Each idiom has a corresponding code example that resides in [code](code).

All results listed in README.md are running with Ruby 2.7.2p137, 3.0.0p0 and 3.0.0p0 + JIT on OS X 11.1 Machine information: Mac mini (2018), 3.2 GHz 6 Core Intel Core i7, 64 GB 2667 MHz DDR4. Your results may vary, but you get the idea. : )

You can checkout [the travis build](https://travis-ci.org/JuanitoFatas/fast-ruby) for these benchmark results ran against different Ruby implementations.

**Let's write faster code, together! <3**

Analyze your code
-----------------

Checkout the [fasterer](https://github.com/DamirSvrtan/fasterer) project - it's a static analysis that checks speed idioms written in this repo.

Measurement Tool
-----------------

Use [benchmark-ips](https://github.com/evanphx/benchmark-ips) (2.0+).

### Template

```ruby
require "benchmark/ips"

def fast
end

def slow
end

Benchmark.ips do |x|
  x.report("fast code description") { fast }
  x.report("slow code description") { slow }
  x.compare!
end
```

Idioms
------

### Index

- [General](#general)
- [Array](#array)
- [Date](#date)
- [Enumerable](#enumerable)
- [Hash](#hash)
- [Proc & Block](#proc--block)
- [String](#string)
- [Time](#time)
- [Range](#range)

### General

##### Parallel Assignment vs Sequential Assignment [code](code/general/assignment.rb)

[Read the rationale here](https://github.com/JuanitoFatas/fast-ruby/pull/50#issue-98586885).

```sh
$ ruby -v --jit code/general/assignment.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
 Parallel Assignment     1.183M i/100ms
Sequential Assignment
                         1.208M i/100ms
Calculating -------------------------------------
 Parallel Assignment     19.889M (± 3.1%) i/s -    100.534M in   5.059583s
Sequential Assignment
                         19.847M (± 3.0%) i/s -    100.244M in   5.055527s

Comparison:
 Parallel Assignment: 19889443.1 i/s
Sequential Assignment: 19846591.9 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/general/assignment.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
 Parallel Assignment     1.282M i/100ms
Sequential Assignment
                         1.327M i/100ms
Calculating -------------------------------------
 Parallel Assignment     12.673M (± 2.2%) i/s -     64.099M in   5.060571s
Sequential Assignment
                         13.252M (± 3.0%) i/s -     66.367M in   5.012856s

Comparison:
Sequential Assignment: 13252013.1 i/s
 Parallel Assignment: 12672578.4 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/general/assignment.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
 Parallel Assignment     1.279M i/100ms
Sequential Assignment
                         1.295M i/100ms
Calculating -------------------------------------
 Parallel Assignment     13.236M (± 3.3%) i/s -     66.505M in   5.030204s
Sequential Assignment
                         12.761M (± 4.6%) i/s -     64.740M in   5.084423s

Comparison:
 Parallel Assignment: 13236017.1 i/s
Sequential Assignment: 12760999.8 i/s - same-ish: difference falls within error
```

##### `attr_accessor` vs `getter and setter` [code](code/general/attr-accessor-vs-getter-and-setter.rb)

> https://www.omniref.com/ruby/2.2.0/files/method.h?#annotation=4081781&line=47

```sh
$ ruby -v --jit code/general/attr-accessor-vs-getter-and-setter.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
   getter_and_setter   456.302k i/100ms
       attr_accessor   494.615k i/100ms
Calculating -------------------------------------
   getter_and_setter      5.966M (± 3.0%) i/s -     30.116M in   5.052444s
       attr_accessor      5.700M (± 3.1%) i/s -     28.688M in   5.037694s

Comparison:
   getter_and_setter:  5966373.4 i/s
       attr_accessor:  5700138.9 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/general/attr-accessor-vs-getter-and-setter.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
   getter_and_setter   475.872k i/100ms
       attr_accessor   510.277k i/100ms
Calculating -------------------------------------
   getter_and_setter      4.762M (± 2.7%) i/s -     23.794M in   5.000509s
       attr_accessor      5.042M (± 3.0%) i/s -     25.514M in   5.065417s

Comparison:
       attr_accessor:  5041513.7 i/s
   getter_and_setter:  4761946.0 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/general/attr-accessor-vs-getter-and-setter.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
   getter_and_setter   439.101k i/100ms
       attr_accessor   445.426k i/100ms
Calculating -------------------------------------
   getter_and_setter      4.261M (± 4.3%) i/s -     21.516M in   5.059714s
       attr_accessor      4.502M (± 4.4%) i/s -     22.717M in   5.055727s

Comparison:
       attr_accessor:  4502436.3 i/s
   getter_and_setter:  4260788.8 i/s - same-ish: difference falls within error
```

##### `begin...rescue` vs `respond_to?` for Control Flow [code](code/general/begin-rescue-vs-respond-to.rb)

```sh
$ ruby -v --jit code/general/begin-rescue-vs-respond-to.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
      begin...rescue    73.233k i/100ms
         respond_to?   759.402k i/100ms
Calculating -------------------------------------
      begin...rescue    737.320k (± 3.3%) i/s -      3.735M in   5.071359s
         respond_to?      9.255M (± 3.1%) i/s -     46.324M in   5.010160s

Comparison:
         respond_to?:  9254995.8 i/s
      begin...rescue:   737320.3 i/s - 12.55x  (± 0.00) slower
```

```sh
$ ruby -v code/general/begin-rescue-vs-respond-to.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
      begin...rescue    78.659k i/100ms
         respond_to?   831.216k i/100ms
Calculating -------------------------------------
      begin...rescue    783.395k (± 2.2%) i/s -      3.933M in   5.022975s
         respond_to?      8.137M (± 1.9%) i/s -     40.730M in   5.007601s

Comparison:
         respond_to?:  8136591.9 i/s
      begin...rescue:   783394.5 i/s - 10.39x  (± 0.00) slower
```

```sh
$ ruby -v code/general/begin-rescue-vs-respond-to.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
      begin...rescue    79.631k i/100ms
         respond_to?   924.187k i/100ms
Calculating -------------------------------------
      begin...rescue    795.697k (± 3.0%) i/s -      3.982M in   5.008373s
         respond_to?      9.152M (± 1.9%) i/s -     46.209M in   5.051147s

Comparison:
         respond_to?:  9151841.6 i/s
      begin...rescue:   795696.9 i/s - 11.50x  (± 0.00) slower
```

##### `define_method` vs `module_eval` for Defining Methods [code](code/general/define_method-vs-module-eval.rb)

```sh
$ ruby -v --jit code/general/define_method-vs-module-eval.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
module_eval with string
                       238.000  i/100ms
       define_method   313.000  i/100ms
Calculating -------------------------------------
module_eval with string
                          2.238k (±16.3%) i/s -     10.948k in   5.084874s
       define_method      3.285k (±19.1%) i/s -     15.337k in   5.047939s

Comparison:
       define_method:     3285.1 i/s
module_eval with string:     2238.3 i/s - 1.47x  (± 0.00) slower
```

```sh
ruby -v code/general/define_method-vs-module-eval.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
module_eval with string
                       248.000  i/100ms
       define_method   344.000  i/100ms
Calculating -------------------------------------
module_eval with string
                          2.353k (±14.2%) i/s -     11.408k in   5.007922s
       define_method      3.403k (±17.0%) i/s -     16.168k in   5.052979s

Comparison:
       define_method:     3402.7 i/s
module_eval with string:     2353.1 i/s - 1.45x  (± 0.00) slower
```

```sh
$ ruby -v code/general/define_method-vs-module-eval.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
module_eval with string
                       273.000  i/100ms
       define_method   366.000  i/100ms
Calculating -------------------------------------
module_eval with string
                          2.390k (±17.4%) i/s -     11.466k in   5.006727s
       define_method      3.843k (±14.6%) i/s -     18.666k in   5.055002s

Comparison:
       define_method:     3843.3 i/s
module_eval with string:     2390.3 i/s - 1.61x  (± 0.00) slower
```

##### `raise` vs `E2MM#Raise` for raising (and defining) exeptions  [code](code/general/raise-vs-e2mmap.rb)

Ruby's [Exception2MessageMapper module](http://ruby-doc.org/stdlib-2.2.0/libdoc/e2mmap/rdoc/index.html) allows one to define and raise exceptions with predefined messages.

```sh
$ ruby -v code/general/raise-vs-e2mmap.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
<internal:/Users/xxxxxx/.rbenv/versions/3.0.0/lib/ruby/3.0.0/rubygems/core_ext/kernel_require.rb>:85:in `require': cannot load such file -- e2mmap (LoadError)
	from <internal:/Users/xxxxxx/.rbenv/versions/3.0.0/lib/ruby/3.0.0/rubygems/core_ext/kernel_require.rb>:85:in `require'
	from code/general/raise-vs-e2mmap.rb:2:in `<main>'
```

```sh
$ ruby -v code/general/raise-vs-e2mmap.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Traceback (most recent call last):
	2: from code/general/raise-vs-e2mmap.rb:2:in `<main>'
	1: from /Users/moritasatoshi/.rbenv/versions/2.7.2/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require'
/Users/moritasatoshi/.rbenv/versions/2.7.2/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require': cannot load such file -- e2mmap (LoadError)
```

##### `loop` vs `while true` [code](code/general/loop-vs-while-true.rb)

```sh
$ ruby -v --jit code/general/loop-vs-while-true.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
          While Loop     1.000  i/100ms
         Kernel loop     1.000  i/100ms
Calculating -------------------------------------
          While Loop      0.901  (± 0.0%) i/s -      5.000  in   5.552331s
         Kernel loop      0.303  (± 0.0%) i/s -      2.000  in   6.596543s

Comparison:
          While Loop:        0.9 i/s
         Kernel loop:        0.3 i/s - 2.97x  (± 0.00) slower
```

```sh
$ ruby -v code/general/loop-vs-while-true.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
          While Loop     1.000  i/100ms
         Kernel loop     1.000  i/100ms
Calculating -------------------------------------
          While Loop      0.904  (± 0.0%) i/s -      5.000  in   5.532168s
         Kernel loop      0.245  (± 0.0%) i/s -      2.000  in   8.156238s

Comparison:
          While Loop:        0.9 i/s
         Kernel loop:        0.2 i/s - 3.69x  (± 0.00) slower
```

```sh
$ ruby -v code/general/loop-vs-while-true.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
          While Loop     1.000  i/100ms
         Kernel loop     1.000  i/100ms
Calculating -------------------------------------
          While Loop      0.866  (± 0.0%) i/s -      5.000  in   5.771580s
         Kernel loop      0.262  (± 0.0%) i/s -      2.000  in   7.634135s

Comparison:
          While Loop:        0.9 i/s
         Kernel loop:        0.3 i/s - 3.31x  (± 0.00) slower
```

##### `ancestors.include?` vs `<=` [code](code/general/inheritance-check.rb)

```sh
$ ruby -vW0 --jit code/general/inheritance-check.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  less than or equal   768.207k i/100ms
  ancestors.include?   127.250k i/100ms
Calculating -------------------------------------
  less than or equal      9.614M (± 3.3%) i/s -     48.397M in   5.039452s
  ancestors.include?      1.427M (± 3.1%) i/s -      7.126M in   5.000115s

Comparison:
  less than or equal:  9614457.2 i/s
  ancestors.include?:  1426613.1 i/s - 6.74x  (± 0.00) slower
```

```sh
$ ruby -vW0 code/general/inheritance-check.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  less than or equal   589.574k i/100ms
  ancestors.include?   126.164k i/100ms
Calculating -------------------------------------
  less than or equal      5.856M (± 2.8%) i/s -     29.479M in   5.038227s
  ancestors.include?      1.244M (± 2.7%) i/s -      6.308M in   5.077044s

Comparison:
  less than or equal:  5855755.5 i/s
  ancestors.include?:  1243526.7 i/s - 4.71x  (± 0.00) slower
```

```sh
$ ruby -vW0 code/general/inheritance-check.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  less than or equal   623.069k i/100ms
  ancestors.include?   117.678k i/100ms
Calculating -------------------------------------
  less than or equal      6.100M (± 2.7%) i/s -     30.530M in   5.008685s
  ancestors.include?      1.161M (± 2.1%) i/s -      5.884M in   5.070769s

Comparison:
  less than or equal:  6100109.1 i/s
  ancestors.include?:  1160881.0 i/s - 5.25x  (± 0.00) slower
```

### Method Invocation

##### `call` vs `send` vs `method_missing` [code](code/method/call-vs-send-vs-method_missing.rb)

```sh
$ ruby -v --jit code/method/call-vs-send-vs-method_missing.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
                call   597.543k i/100ms
                send   500.935k i/100ms
      method_missing   359.738k i/100ms
Calculating -------------------------------------
                call      7.957M (± 3.5%) i/s -     40.035M in   5.037966s
                send      5.447M (± 3.5%) i/s -     27.551M in   5.065155s
      method_missing      4.309M (± 3.1%) i/s -     21.584M in   5.013623s

Comparison:
                call:  7956613.5 i/s
                send:  5446584.4 i/s - 1.46x  (± 0.00) slower
      method_missing:  4309345.5 i/s - 1.85x  (± 0.00) slower
```

```sh
$ ruby -v code/method/call-vs-send-vs-method_missing.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
                call   622.263k i/100ms
                send   479.194k i/100ms
      method_missing   383.571k i/100ms
Calculating -------------------------------------
                call      6.277M (± 2.8%) i/s -     31.735M in   5.059745s
                send      4.641M (± 2.2%) i/s -     23.481M in   5.062467s
      method_missing      3.803M (± 2.7%) i/s -     19.179M in   5.047433s

Comparison:
                call:  6277034.5 i/s
                send:  4640505.8 i/s - 1.35x  (± 0.00) slower
      method_missing:  3802528.2 i/s - 1.65x  (± 0.00) slower
```

```sh
$ ruby -v code/method/call-vs-send-vs-method_missing.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
                call   616.613k i/100ms
                send   494.148k i/100ms
      method_missing   402.864k i/100ms
Calculating -------------------------------------
                call      5.917M (± 4.4%) i/s -     29.597M in   5.011819s
                send      4.822M (± 3.9%) i/s -     24.213M in   5.029582s
      method_missing      3.946M (± 3.4%) i/s -     19.740M in   5.008892s

Comparison:
                call:  5917275.2 i/s
                send:  4821949.7 i/s - 1.23x  (± 0.00) slower
      method_missing:  3945873.5 i/s - 1.50x  (± 0.00) slower
```

##### Normal way to apply method vs `&method(...)` [code](code/general/block-apply-method.rb)

```sh
$ ruby -v --jit code/general/block-apply-method.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
              normal   305.290k i/100ms
             &method    91.520k i/100ms
Calculating -------------------------------------
              normal      3.752M (±10.6%) i/s -     18.623M in   5.042571s
             &method    901.751k (± 9.5%) i/s -      4.484M in   5.032685s

Comparison:
              normal:  3752497.2 i/s
             &method:   901751.1 i/s - 4.16x  (± 0.00) slower
```

```sh
$ ruby -v code/general/block-apply-method.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
              normal   321.646k i/100ms
             &method    96.244k i/100ms
Calculating -------------------------------------
              normal      3.194M (± 1.4%) i/s -     16.082M in   5.036074s
             &method    945.800k (± 2.2%) i/s -      4.812M in   5.090436s

Comparison:
              normal:  3194097.6 i/s
             &method:   945800.2 i/s - 3.38x  (± 0.00) slower
```

```sh
$ ruby -v code/general/block-apply-method.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
              normal   339.914k i/100ms
             &method    96.728k i/100ms
Calculating -------------------------------------
              normal      3.409M (± 1.7%) i/s -     17.336M in   5.086629s
             &method    952.460k (± 2.2%) i/s -      4.836M in   5.080383s

Comparison:
              normal:  3409040.0 i/s
             &method:   952459.8 i/s - 3.58x  (± 0.00) slower
```

##### Function with single Array argument vs splat arguments [code](code/general/array-argument-vs-splat-arguments.rb)

```sh
$ ruby -v --jit code/general/array-argument-vs-splat-arguments.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Function with single Array argument
                       825.026k i/100ms
Function with splat arguments
                        14.985k i/100ms
Calculating -------------------------------------
Function with single Array argument
                          9.040M (± 7.1%) i/s -     45.376M in   5.047131s
Function with splat arguments
                        158.733k (±13.5%) i/s -    779.220k in   5.001859s

Comparison:
Function with single Array argument:  9040320.5 i/s
Function with splat arguments:   158732.8 i/s - 56.95x  (± 0.00) slower
```

```sh
$ ruby -v code/general/array-argument-vs-splat-arguments.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Function with single Array argument
                       897.066k i/100ms
Function with splat arguments
                        19.546k i/100ms
Calculating -------------------------------------
Function with single Array argument
                          8.878M (± 2.3%) i/s -     44.853M in   5.055116s
Function with splat arguments
                        209.608k (± 2.8%) i/s -      1.055M in   5.039524s

Comparison:
Function with single Array argument:  8877887.5 i/s
Function with splat arguments:   209608.1 i/s - 42.35x  (± 0.00) slower
```

```sh
$ ruby -v code/general/array-argument-vs-splat-arguments.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Function with single Array argument
                       887.728k i/100ms
Function with splat arguments
                        11.704k i/100ms
Calculating -------------------------------------
Function with single Array argument
                          8.929M (± 3.1%) i/s -     45.274M in   5.075466s
Function with splat arguments
                        200.708k (±11.1%) i/s -    994.840k in   5.035744s

Comparison:
Function with single Array argument:  8929298.6 i/s
Function with splat arguments:   200708.1 i/s - 44.49x  (± 0.00) slower
```

##### Hash vs OpenStruct on access assuming you already have a Hash or an OpenStruct [code](code/general/hash-vs-openstruct-on-access.rb)

```sh
$ ruby -v --jit code/general/hash-vs-openstruct-on-access.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
                Hash   945.061k i/100ms
          OpenStruct   473.333k i/100ms
Calculating -------------------------------------
                Hash     10.968M (± 3.9%) i/s -     54.814M in   5.005838s
          OpenStruct      5.399M (± 2.2%) i/s -     27.453M in   5.087524s

Comparison:
                Hash: 10968444.9 i/s
          OpenStruct:  5398976.2 i/s - 2.03x  (± 0.00) slower
```

```sh
$ ruby -v code/general/hash-vs-openstruct-on-access.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
                Hash   983.613k i/100ms
          OpenStruct   493.441k i/100ms
Calculating -------------------------------------
                Hash      9.685M (± 3.1%) i/s -     49.181M in   5.083119s
          OpenStruct      4.801M (± 4.4%) i/s -     24.179M in   5.046756s

Comparison:
                Hash:  9685315.8 i/s
          OpenStruct:  4801378.3 i/s - 2.02x  (± 0.00) slower
```

```sh
$ ruby -v code/general/hash-vs-openstruct-on-access.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
                Hash     1.001M i/100ms
          OpenStruct   509.244k i/100ms
Calculating -------------------------------------
                Hash     10.277M (± 2.2%) i/s -     52.076M in   5.069727s
          OpenStruct      5.107M (± 2.9%) i/s -     25.971M in   5.089827s

Comparison:
                Hash: 10277399.8 i/s
          OpenStruct:  5107192.0 i/s - 2.01x  (± 0.00) slower
```

##### Hash vs OpenStruct (creation) [code](code/general/hash-vs-openstruct.rb)

```sh
$ ruby -v --jit code/general/hash-vs-openstruct.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
                Hash     1.229M i/100ms
          OpenStruct     9.722k i/100ms
Calculating -------------------------------------
                Hash     12.678M (± 5.2%) i/s -     63.928M in   5.056420s
          OpenStruct     99.263k (± 5.3%) i/s -    495.822k in   5.010216s

Comparison:
                Hash: 12678351.2 i/s
          OpenStruct:    99262.8 i/s - 127.73x  (± 0.00) slower
```

```sh
$ ruby -v code/general/hash-vs-openstruct.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
                Hash     1.078M i/100ms
          OpenStruct    10.610k i/100ms
Calculating -------------------------------------
                Hash     11.167M (± 4.0%) i/s -     56.065M in   5.029384s
          OpenStruct    104.733k (± 2.0%) i/s -    530.500k in   5.067360s

Comparison:
                Hash: 11166626.2 i/s
          OpenStruct:   104733.1 i/s - 106.62x  (± 0.00) slower
```

```sh
$ ruby -v code/general/hash-vs-openstruct.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
                Hash     1.187M i/100ms
          OpenStruct   179.138k i/100ms
Calculating -------------------------------------
                Hash     11.468M (± 3.2%) i/s -     58.165M in   5.077360s
          OpenStruct      1.753M (± 2.7%) i/s -      8.778M in   5.012052s

Comparison:
                Hash: 11467778.3 i/s
          OpenStruct:  1752630.7 i/s - 6.54x  (± 0.00) slower
```

##### Kernel#format vs Float#round().to_s [code](code/general/format-vs-round-and-to-s.rb)

```sh
$ ruby -v --jit code/general/format-vs-round-and-to-s.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
         Float#round   229.172k i/100ms
       Kernel#format   156.958k i/100ms
            String#%   141.804k i/100ms
Calculating -------------------------------------
         Float#round      2.549M (± 2.0%) i/s -     12.834M in   5.036877s
       Kernel#format      1.676M (± 2.9%) i/s -      8.476M in   5.062040s
            String#%      1.542M (± 2.2%) i/s -      7.799M in   5.060264s

Comparison:
         Float#round:  2548944.9 i/s
       Kernel#format:  1675899.2 i/s - 1.52x  (± 0.00) slower
            String#%:  1542045.6 i/s - 1.65x  (± 0.00) slower
```

```sh
$ ruby -v code/general/format-vs-round-and-to-s.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
         Float#round   236.991k i/100ms
       Kernel#format   164.318k i/100ms
            String#%   146.865k i/100ms
Calculating -------------------------------------
         Float#round      2.371M (± 1.8%) i/s -     12.087M in   5.099460s
       Kernel#format      1.654M (± 1.7%) i/s -      8.380M in   5.068616s
            String#%      1.502M (± 1.6%) i/s -      7.637M in   5.084835s

Comparison:
         Float#round:  2370943.4 i/s
       Kernel#format:  1653851.0 i/s - 1.43x  (± 0.00) slower
            String#%:  1502325.0 i/s - 1.58x  (± 0.00) slower
```

```sh
$ ruby -v code/general/format-vs-round-and-to-s.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
         Float#round   217.695k i/100ms
       Kernel#format   168.968k i/100ms
            String#%   152.228k i/100ms
Calculating -------------------------------------
         Float#round      2.216M (± 1.8%) i/s -     11.102M in   5.011712s
       Kernel#format      1.662M (± 3.0%) i/s -      8.448M in   5.088415s
            String#%      1.573M (± 2.6%) i/s -      7.916M in   5.034429s

Comparison:
         Float#round:  2216031.4 i/s
       Kernel#format:  1661913.2 i/s - 1.33x  (± 0.00) slower
            String#%:  1573460.5 i/s - 1.41x  (± 0.00) slower
```

### Array

##### `Array#bsearch` vs `Array#find` [code](code/array/bsearch-vs-find.rb)

**WARNING:** `bsearch` ONLY works on *sorted array*. More details please see [#29](https://github.com/JuanitoFatas/fast-ruby/issues/29).

```sh
$ ruby -v --jit code/array/bsearch-vs-find.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
                find     1.000  i/100ms
             bsearch    96.401k i/100ms
Calculating -------------------------------------
                find      0.294  (± 0.0%) i/s -      2.000  in   6.798231s
             bsearch      1.020M (± 4.1%) i/s -      5.109M in   5.016886s

Comparison:
             bsearch:  1020307.4 i/s
                find:        0.3 i/s - 3468142.14x  (± 0.00) slower
```

```sh
$ ruby -v code/array/bsearch-vs-find.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
                find     1.000  i/100ms
             bsearch    88.173k i/100ms
Calculating -------------------------------------
                find      0.262  (± 0.0%) i/s -      2.000  in   7.629079s
             bsearch    869.645k (± 3.5%) i/s -      4.409M in   5.076293s

Comparison:
             bsearch:   869644.8 i/s
                find:        0.3 i/s - 3317277.66x  (± 0.00) slower
```

```sh
$ ruby -v code/array/bsearch-vs-find.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
                find     1.000  i/100ms
             bsearch    91.196k i/100ms
Calculating -------------------------------------
                find      0.296  (± 0.0%) i/s -      2.000  in   6.760499s
             bsearch    914.904k (± 2.3%) i/s -      4.651M in   5.086308s

Comparison:
             bsearch:   914904.0 i/s
                find:        0.3 i/s - 3092553.00x  (± 0.00) slower
```

##### `Array#length` vs `Array#size` vs `Array#count` [code](code/array/length-vs-size-vs-count.rb)

Use `#length` when you only want to know how many elements in the array, `#count` could also achieve this. However `#count` should be use for counting specific elements in array. [Note `#size` is an alias of `#length`](https://github.com/ruby/ruby/blob/f8fb526ad9e9f31453bffbc908b6a986736e21a7/array.c#L5817-L5818).

```sh
$ ruby -v --jit code/array/length-vs-size-vs-count.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
        Array#length     2.386M i/100ms
          Array#size     2.404M i/100ms
         Array#count     1.723M i/100ms
Calculating -------------------------------------
        Array#length     24.956M (± 2.9%) i/s -    126.454M in   5.071820s
          Array#size     24.715M (± 3.6%) i/s -    124.982M in   5.064406s
         Array#count     17.981M (± 2.5%) i/s -     91.323M in   5.081982s

Comparison:
        Array#length: 24955547.6 i/s
          Array#size: 24715333.3 i/s - same-ish: difference falls within error
         Array#count: 17981427.7 i/s - 1.39x  (± 0.00) slower
```

```sh
$ ruby -v code/array/length-vs-size-vs-count.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
        Array#length     2.366M i/100ms
          Array#size     2.406M i/100ms
         Array#count     1.659M i/100ms
Calculating -------------------------------------
        Array#length     23.847M (± 2.3%) i/s -    120.683M in   5.063504s
          Array#size     23.764M (± 2.2%) i/s -    120.304M in   5.064909s
         Array#count     16.480M (± 1.5%) i/s -     82.928M in   5.033069s

Comparison:
        Array#length: 23846934.7 i/s
          Array#size: 23764167.6 i/s - same-ish: difference falls within error
         Array#count: 16480215.2 i/s - 1.45x  (± 0.00) slower
```

```sh
$ ruby -v code/array/length-vs-size-vs-count.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
        Array#length     2.596M i/100ms
          Array#size     2.560M i/100ms
         Array#count     1.777M i/100ms
Calculating -------------------------------------
        Array#length     25.435M (± 2.8%) i/s -    127.205M in   5.005299s
          Array#size     25.269M (± 2.3%) i/s -    127.989M in   5.067811s
         Array#count     17.736M (± 3.1%) i/s -     88.872M in   5.015894s

Comparison:
        Array#length: 25435172.6 i/s
          Array#size: 25269049.9 i/s - same-ish: difference falls within error
         Array#count: 17735534.5 i/s - 1.43x  (± 0.00) slower
```

##### `Array#shuffle.first` vs `Array#sample` [code](code/array/shuffle-first-vs-sample.rb)

> `Array#shuffle` allocates an extra array. <br>
> `Array#sample` indexes into the array without allocating an extra array. <br>
> This is the reason why Array#sample exists. <br>
> —— @sferik [rails/rails#17245](https://github.com/rails/rails/pull/17245)

```sh
$ ruby -v --jit code/array/shuffle-first-vs-sample.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
 Array#shuffle.first    39.137k i/100ms
        Array#sample   629.472k i/100ms
Calculating -------------------------------------
 Array#shuffle.first    416.282k (± 4.1%) i/s -      2.113M in   5.085984s
        Array#sample      7.802M (± 2.4%) i/s -     39.027M in   5.005181s

Comparison:
        Array#sample:  7802166.0 i/s
 Array#shuffle.first:   416282.3 i/s - 18.74x  (± 0.00) slower
```

```sh
$ ruby -v code/array/shuffle-first-vs-sample.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
 Array#shuffle.first    41.581k i/100ms
        Array#sample   697.885k i/100ms
Calculating -------------------------------------
 Array#shuffle.first    417.075k (± 2.4%) i/s -      2.121M in   5.087423s
        Array#sample      6.893M (± 3.2%) i/s -     34.894M in   5.067868s

Comparison:
        Array#sample:  6893325.4 i/s
 Array#shuffle.first:   417075.2 i/s - 16.53x  (± 0.00) slower
```

```sh
$ ruby -v code/array/shuffle-first-vs-sample.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
 Array#shuffle.first    58.948k i/100ms
        Array#sample     1.273M i/100ms
Calculating -------------------------------------
 Array#shuffle.first    580.951k (± 3.1%) i/s -      2.947M in   5.078244s
        Array#sample     12.909M (± 2.3%) i/s -     64.930M in   5.032926s

Comparison:
        Array#sample: 12908556.1 i/s
 Array#shuffle.first:   580950.6 i/s - 22.22x  (± 0.00) slower
```

##### `Array#[](0)` vs `Array#first` [code](code/array/array-first-vs-index.rb)

```sh
$ ruby -v --jit code/array/array-first-vs-index.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
           Array#[0]     1.751M i/100ms
         Array#first     1.282M i/100ms
Calculating -------------------------------------
           Array#[0]     21.457M (± 2.5%) i/s -    108.582M in   5.063720s
         Array#first     17.061M (± 3.9%) i/s -     85.918M in   5.043743s

Comparison:
           Array#[0]: 21457036.5 i/s
         Array#first: 17061234.2 i/s - 1.26x  (± 0.00) slower
```

```sh
$ ruby -v code/array/array-first-vs-index.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
           Array#[0]     1.782M i/100ms
         Array#first     1.442M i/100ms
Calculating -------------------------------------
           Array#[0]     18.309M (± 2.4%) i/s -     92.690M in   5.065621s
         Array#first     14.511M (± 1.7%) i/s -     73.560M in   5.070792s

Comparison:
           Array#[0]: 18309320.8 i/s
         Array#first: 14510612.9 i/s - 1.26x  (± 0.00) slower
```

```sh
$ ruby -v code/array/array-first-vs-index.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
           Array#[0]     1.847M i/100ms
         Array#first     1.534M i/100ms
Calculating -------------------------------------
           Array#[0]     18.406M (± 2.9%) i/s -     92.335M in   5.020851s
         Array#first     15.146M (± 2.1%) i/s -     76.720M in   5.067686s

Comparison:
           Array#[0]: 18406468.9 i/s
         Array#first: 15146260.0 i/s - 1.22x  (± 0.00) slower
```

##### `Array#[](-1)` vs `Array#last` [code](code/array/array-last-vs-index.rb)

```sh
$ ruby -v --jit code/array/array-last-vs-index.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
          Array#[-1]     1.699M i/100ms
          Array#last     1.256M i/100ms
Calculating -------------------------------------
          Array#[-1]     21.164M (± 4.4%) i/s -    107.042M in   5.069541s
          Array#last     17.539M (± 3.0%) i/s -     87.933M in   5.018236s

Comparison:
          Array#[-1]: 21164021.9 i/s
          Array#last: 17539017.0 i/s - 1.21x  (± 0.00) slower
```

```sh
$ ruby -v code/array/array-last-vs-index.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
          Array#[-1]     1.688M i/100ms
          Array#last     1.455M i/100ms
Calculating -------------------------------------
          Array#[-1]     17.091M (± 2.7%) i/s -     86.110M in   5.042505s
          Array#last     14.483M (± 1.3%) i/s -     72.766M in   5.024972s

Comparison:
          Array#[-1]: 17090591.9 i/s
          Array#last: 14483337.4 i/s - 1.18x  (± 0.00) slower
```

```sh
$ ruby -v code/array/array-last-vs-index.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
          Array#[-1]     1.940M i/100ms
          Array#last     1.456M i/100ms
Calculating -------------------------------------
          Array#[-1]     19.193M (± 3.2%) i/s -     96.989M in   5.058932s
          Array#last     15.262M (± 2.2%) i/s -     77.169M in   5.058836s

Comparison:
          Array#[-1]: 19193245.1 i/s
          Array#last: 15261520.9 i/s - 1.26x  (± 0.00) slower
```

##### `Array#insert` vs `Array#unshift` [code](code/array/insert-vs-unshift.rb)

```sh
$ ruby -v --jit code/array/insert-vs-unshift.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
       Array#unshift    18.000  i/100ms
        Array#insert     1.000  i/100ms
Calculating -------------------------------------
       Array#unshift    185.371  (± 3.8%) i/s -    936.000  in   5.056657s
        Array#insert      1.191  (± 0.0%) i/s -      6.000  in   5.036528s

Comparison:
       Array#unshift:      185.4 i/s
        Array#insert:        1.2 i/s - 155.59x  (± 0.00) slower
```

```sh
$ ruby -v code/array/insert-vs-unshift.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
       Array#unshift    15.000  i/100ms
        Array#insert     1.000  i/100ms
Calculating -------------------------------------
       Array#unshift    159.391  (± 3.1%) i/s -    810.000  in   5.086914s
        Array#insert      1.196  (± 0.0%) i/s -      6.000  in   5.015627s

Comparison:
       Array#unshift:      159.4 i/s
        Array#insert:        1.2 i/s - 133.24x  (± 0.00) slower
```

```sh
$ ruby -v code/array/insert-vs-unshift.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
       Array#unshift    17.000  i/100ms
        Array#insert     1.000  i/100ms
Calculating -------------------------------------
       Array#unshift    170.680  (± 4.1%) i/s -    867.000  in   5.087800s
        Array#insert      1.200  (± 0.0%) i/s -      6.000  in   5.000729s

Comparison:
       Array#unshift:      170.7 i/s
        Array#insert:        1.2 i/s - 142.25x  (± 0.00) slower
```

### Enumerable

##### `Enumerable#each + push` vs `Enumerable#map` [code](code/enumerable/each-push-vs-map.rb)

```sh
$ ruby -v --jit code/enumerable/each-push-vs-map.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
   Array#each + push    17.698k i/100ms
           Array#map    24.778k i/100ms
Calculating -------------------------------------
   Array#each + push    170.629k (± 3.8%) i/s -    867.202k in   5.090043s
           Array#map    282.464k (± 2.9%) i/s -      1.412M in   5.004473s

Comparison:
           Array#map:   282463.8 i/s
   Array#each + push:   170628.6 i/s - 1.66x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/each-push-vs-map.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
   Array#each + push    15.398k i/100ms
           Array#map    26.020k i/100ms
Calculating -------------------------------------
   Array#each + push    152.391k (± 2.7%) i/s -    769.900k in   5.056003s
           Array#map    259.621k (± 2.5%) i/s -      1.301M in   5.014306s

Comparison:
           Array#map:   259621.0 i/s
   Array#each + push:   152391.1 i/s - 1.70x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/each-push-vs-map.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
   Array#each + push    16.554k i/100ms
           Array#map    28.017k i/100ms
Calculating -------------------------------------
   Array#each + push    166.426k (± 3.2%) i/s -    844.254k in   5.078074s
           Array#map    276.889k (± 3.7%) i/s -      1.401M in   5.066854s

Comparison:
           Array#map:   276888.5 i/s
   Array#each + push:   166425.8 i/s - 1.66x  (± 0.00) slower
```

##### `Enumerable#each` vs `for` loop [code](code/enumerable/each-vs-for-loop.rb)

```sh
$ ruby -v --jit code/enumerable/each-vs-for-loop.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
            For loop    29.813k i/100ms
               #each    32.017k i/100ms
Calculating -------------------------------------
            For loop    304.155k (± 3.1%) i/s -      1.520M in   5.003854s
               #each    319.839k (± 2.8%) i/s -      1.601M in   5.009251s

Comparison:
               #each:   319839.4 i/s
            For loop:   304155.2 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/enumerable/each-vs-for-loop.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
            For loop    24.845k i/100ms
               #each    28.987k i/100ms
Calculating -------------------------------------
            For loop    256.260k (± 2.7%) i/s -      1.292M in   5.045390s
               #each    290.581k (± 2.2%) i/s -      1.478M in   5.090117s

Comparison:
               #each:   290580.6 i/s
            For loop:   256259.6 i/s - 1.13x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/each-vs-for-loop.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
            For loop    29.179k i/100ms
               #each    31.971k i/100ms
Calculating -------------------------------------
            For loop    287.077k (± 2.8%) i/s -      1.459M in   5.086186s
               #each    314.473k (± 2.2%) i/s -      1.599M in   5.085853s

Comparison:
               #each:   314472.8 i/s
            For loop:   287076.6 i/s - 1.10x  (± 0.00) slower
```

##### `Enumerable#each_with_index` vs `while` loop [code](code/enumerable/each_with_index-vs-while-loop.rb)

> [rails/rails#12065](https://github.com/rails/rails/pull/12065)

```sh
$ ruby -v --jit code/enumerable/each_with_index-vs-while-loop.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
code/enumerable/each_with_index-vs-while-loop.rb:8: warning: possibly useless use of + in void context
Warming up --------------------------------------
          While Loop    42.629k i/100ms
     each_with_index    16.934k i/100ms
Calculating -------------------------------------
          While Loop      1.144M (± 3.8%) i/s -      5.755M in   5.036169s
     each_with_index    180.197k (± 3.4%) i/s -    914.436k in   5.080720s

Comparison:
          While Loop:  1144412.1 i/s
     each_with_index:   180197.1 i/s - 6.35x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/each_with_index-vs-while-loop.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
code/enumerable/each_with_index-vs-while-loop.rb:8: warning: possibly useless use of + in void context
Warming up --------------------------------------
          While Loop    43.403k i/100ms
     each_with_index    17.555k i/100ms
Calculating -------------------------------------
          While Loop    423.031k (± 3.8%) i/s -      2.127M in   5.035298s
     each_with_index    174.927k (± 1.9%) i/s -    877.750k in   5.019656s

Comparison:
          While Loop:   423030.6 i/s
     each_with_index:   174926.8 i/s - 2.42x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/each_with_index-vs-while-loop.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
code/enumerable/each_with_index-vs-while-loop.rb:8: warning: possibly useless use of + in void context
Warming up --------------------------------------
          While Loop    41.759k i/100ms
     each_with_index    22.114k i/100ms
Calculating -------------------------------------
          While Loop    424.396k (± 2.8%) i/s -      2.130M in   5.022342s
     each_with_index    215.801k (± 3.2%) i/s -      1.084M in   5.027004s

Comparison:
          While Loop:   424396.1 i/s
     each_with_index:   215801.4 i/s - 1.97x  (± 0.00) slower
```

##### `Enumerable#map`...`Array#flatten` vs `Enumerable#flat_map` [code](code/enumerable/map-flatten-vs-flat_map.rb)

> -- @sferik [rails/rails@3413b88](https://github.com/rails/rails/commit/3413b88), [Replace map.flatten with flat_map](https://github.com/rails/rails/commit/817fe31196dd59ee31f71ef1740122b6759cf16d), [Replace map.flatten(1) with flat_map](https://github.com/rails/rails/commit/b11ebf1d80e4fb124f0ce0448cea30988256da59)

```sh
$ ruby -v --jit code/enumerable/map-flatten-vs-flat_map.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Array#map.flatten(1)     8.870k i/100ms
   Array#map.flatten     4.065k i/100ms
      Array#flat_map    10.354k i/100ms
Calculating -------------------------------------
Array#map.flatten(1)     90.814k (± 4.9%) i/s -    461.240k in   5.092080s
   Array#map.flatten     41.822k (± 2.9%) i/s -    211.380k in   5.058486s
      Array#flat_map    105.686k (± 2.2%) i/s -    538.408k in   5.096995s

Comparison:
      Array#flat_map:   105686.5 i/s
Array#map.flatten(1):    90813.9 i/s - 1.16x  (± 0.00) slower
   Array#map.flatten:    41822.3 i/s - 2.53x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/map-flatten-vs-flat_map.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Array#map.flatten(1)     8.827k i/100ms
   Array#map.flatten     4.018k i/100ms
      Array#flat_map     9.906k i/100ms
Calculating -------------------------------------
Array#map.flatten(1)     88.472k (± 3.3%) i/s -    450.177k in   5.094245s
   Array#map.flatten     41.856k (± 2.9%) i/s -    212.954k in   5.092115s
      Array#flat_map     99.612k (± 2.8%) i/s -    505.206k in   5.075814s

Comparison:
      Array#flat_map:    99611.6 i/s
Array#map.flatten(1):    88472.0 i/s - 1.13x  (± 0.00) slower
   Array#map.flatten:    41856.3 i/s - 2.38x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/map-flatten-vs-flat_map.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Array#map.flatten(1)     7.050k i/100ms
   Array#map.flatten     4.456k i/100ms
      Array#flat_map     9.473k i/100ms
Calculating -------------------------------------
Array#map.flatten(1)     69.278k (± 2.6%) i/s -    352.500k in   5.091771s
   Array#map.flatten     46.757k (± 2.8%) i/s -    236.168k in   5.055013s
      Array#flat_map    103.136k (± 3.4%) i/s -    521.015k in   5.057713s

Comparison:
      Array#flat_map:   103136.0 i/s
Array#map.flatten(1):    69277.8 i/s - 1.49x  (± 0.00) slower
   Array#map.flatten:    46756.5 i/s - 2.21x  (± 0.00) slower
```

##### `Enumerable#reverse.each` vs `Enumerable#reverse_each` [code](code/enumerable/reverse-each-vs-reverse_each.rb)

> `Enumerable#reverse` allocates an extra array.  <br>
> `Enumerable#reverse_each` yields each value without allocating an extra array. <br>
> This is the reason why `Enumerable#reverse_each` exists. <br>
> -- @sferik [rails/rails#17244](https://github.com/rails/rails/pull/17244)

```sh
$ ruby -v --jit code/enumerable/reverse-each-vs-reverse_each.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  Array#reverse.each    29.402k i/100ms
  Array#reverse_each    32.429k i/100ms
Calculating -------------------------------------
  Array#reverse.each    317.826k (± 2.8%) i/s -      1.617M in   5.092053s
  Array#reverse_each    331.333k (± 3.0%) i/s -      1.686M in   5.094355s

Comparison:
  Array#reverse_each:   331333.1 i/s
  Array#reverse.each:   317826.1 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/enumerable/reverse-each-vs-reverse_each.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  Array#reverse.each    29.199k i/100ms
  Array#reverse_each    29.790k i/100ms
Calculating -------------------------------------
  Array#reverse.each    286.777k (± 2.4%) i/s -      1.460M in   5.093774s
  Array#reverse_each    296.647k (± 2.4%) i/s -      1.490M in   5.024229s

Comparison:
  Array#reverse_each:   296646.6 i/s
  Array#reverse.each:   286776.6 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/enumerable/reverse-each-vs-reverse_each.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  Array#reverse.each    30.667k i/100ms
  Array#reverse_each    31.144k i/100ms
Calculating -------------------------------------
  Array#reverse.each    308.667k (± 2.4%) i/s -      1.564M in   5.069930s
  Array#reverse_each    314.703k (± 1.9%) i/s -      1.588M in   5.049025s

Comparison:
  Array#reverse_each:   314703.5 i/s
  Array#reverse.each:   308667.3 i/s - same-ish: difference falls within error
```

##### `Enumerable#sort_by.first` vs `Enumerable#min_by` [code](code/enumerable/sort_by-first-vs-min_by.rb)
`Enumerable#sort_by` performs a sort of the enumerable and allocates a
new array the size of the enumerable.  `Enumerable#min_by` doesn't
perform a sort or allocate an array the size of the enumerable.
Similar comparisons hold for `Enumerable#sort_by.last` vs
`Enumerable#max_by`, `Enumerable#sort.first` vs `Enumerable#min`, and
`Enumerable#sort.last` vs `Enumerable#max`.

```sh
$ ruby -v --jit code/enumerable/sort_by-first-vs-min_by.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
   Enumerable#min_by    17.304k i/100ms
Enumerable#sort_by...first
                        10.174k i/100ms
Calculating -------------------------------------
   Enumerable#min_by    174.638k (±11.6%) i/s -    865.200k in   5.048445s
Enumerable#sort_by...first
                         99.156k (±12.5%) i/s -    488.352k in   5.021464s

Comparison:
   Enumerable#min_by:   174638.4 i/s
Enumerable#sort_by...first:    99156.1 i/s - 1.76x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/sort_by-first-vs-min_by.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
   Enumerable#min_by    16.454k i/100ms
Enumerable#sort_by...first
                        10.393k i/100ms
Calculating -------------------------------------
   Enumerable#min_by    164.171k (± 2.4%) i/s -    822.700k in   5.014462s
Enumerable#sort_by...first
                        101.481k (± 3.6%) i/s -    509.257k in   5.025207s

Comparison:
   Enumerable#min_by:   164171.5 i/s
Enumerable#sort_by...first:   101481.0 i/s - 1.62x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/sort_by-first-vs-min_by.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
   Enumerable#min_by    20.612k i/100ms
Enumerable#sort_by...first
                        12.981k i/100ms
Calculating -------------------------------------
   Enumerable#min_by    208.578k (± 3.1%) i/s -      1.051M in   5.044864s
Enumerable#sort_by...first
                        126.901k (± 2.5%) i/s -    636.069k in   5.015488s

Comparison:
   Enumerable#min_by:   208578.2 i/s
Enumerable#sort_by...first:   126901.2 i/s - 1.64x  (± 0.00) slower
```

##### `Enumerable#detect` vs `Enumerable#select.first` [code](code/enumerable/select-first-vs-detect.rb)

```sh
$ ruby -v --jit code/enumerable/select-first-vs-detect.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#select.first
                        18.852k i/100ms
   Enumerable#detect    84.231k i/100ms
Calculating -------------------------------------
Enumerable#select.first
                        200.680k (± 3.2%) i/s -      4.015M in  20.030741s
   Enumerable#detect    847.373k (± 4.8%) i/s -     16.930M in  20.033292s

Comparison:
   Enumerable#detect:   847373.2 i/s
Enumerable#select.first:   200679.8 i/s - 4.22x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/select-first-vs-detect.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#select.first
                        17.431k i/100ms
   Enumerable#detect    74.397k i/100ms
Calculating -------------------------------------
Enumerable#select.first
                        171.708k (± 1.9%) i/s -      3.434M in  20.005602s
   Enumerable#detect    758.742k (± 2.0%) i/s -     15.177M in  20.011523s

Comparison:
   Enumerable#detect:   758742.0 i/s
Enumerable#select.first:   171708.4 i/s - 4.42x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/select-first-vs-detect.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Enumerable#select.first
                        18.189k i/100ms
   Enumerable#detect    83.050k i/100ms
Calculating -------------------------------------
Enumerable#select.first
                        178.365k (± 4.7%) i/s -      3.565M in  20.036289s
   Enumerable#detect    810.129k (± 4.7%) i/s -     16.195M in  20.040681s

Comparison:
   Enumerable#detect:   810129.4 i/s
Enumerable#select.first:   178364.5 i/s - 4.54x  (± 0.00) slower
```

##### `Enumerable#select.last` vs `Enumerable#reverse.detect` [code](code/enumerable/select-last-vs-reverse-detect.rb)

```sh
$ ruby -v --jit code/enumerable/select-last-vs-reverse-detect.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#reverse.detect
                       215.754k i/100ms
Enumerable#select.last
                        26.025k i/100ms
Calculating -------------------------------------
Enumerable#reverse.detect
                          2.766M (± 3.2%) i/s -     14.024M in   5.075139s
Enumerable#select.last
                        266.757k (± 4.3%) i/s -      1.353M in   5.083193s

Comparison:
Enumerable#reverse.detect:  2766456.2 i/s
Enumerable#select.last:   266757.3 i/s - 10.37x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/select-last-vs-reverse-detect.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#reverse.detect
                       256.964k i/100ms
Enumerable#select.last
                        17.036k i/100ms
Calculating -------------------------------------
Enumerable#reverse.detect
                          2.526M (± 2.2%) i/s -     12.848M in   5.089620s
Enumerable#select.last
                        166.951k (± 2.6%) i/s -    834.764k in   5.003609s

Comparison:
Enumerable#reverse.detect:  2525672.9 i/s
Enumerable#select.last:   166951.3 i/s - 15.13x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/select-last-vs-reverse-detect.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Enumerable#reverse.detect
                       243.387k i/100ms
Enumerable#select.last
                        16.894k i/100ms
Calculating -------------------------------------
Enumerable#reverse.detect
                          2.524M (± 5.2%) i/s -     12.656M in   5.029239s
Enumerable#select.last
                        167.626k (± 1.9%) i/s -    844.700k in   5.040999s

Comparison:
Enumerable#reverse.detect:  2523534.9 i/s
Enumerable#select.last:   167625.7 i/s - 15.05x  (± 0.00) slower
```

##### `Enumerable#sort` vs `Enumerable#sort_by` [code](code/enumerable/sort-vs-sort_by.rb)

```sh
$ ruby -v --jit code/enumerable/sort-vs-sort_by.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         4.180k i/100ms
  Enumerable#sort_by     3.996k i/100ms
     Enumerable#sort     1.493k i/100ms
Calculating -------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         41.294k (± 5.8%) i/s -    209.000k in   5.081642s
  Enumerable#sort_by     41.856k (± 2.9%) i/s -    211.788k in   5.064192s
     Enumerable#sort     18.601k (± 4.3%) i/s -     94.059k in   5.066592s

Comparison:
  Enumerable#sort_by:    41855.8 i/s
Enumerable#sort_by (Symbol#to_proc):    41294.0 i/s - same-ish: difference falls within error
     Enumerable#sort:    18600.6 i/s - 2.25x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/sort-vs-sort_by.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         4.443k i/100ms
  Enumerable#sort_by     4.150k i/100ms
     Enumerable#sort     1.603k i/100ms
Calculating -------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         44.742k (± 2.4%) i/s -    226.593k in   5.067526s
  Enumerable#sort_by     41.935k (± 3.2%) i/s -    211.650k in   5.052672s
     Enumerable#sort     15.837k (± 2.6%) i/s -     80.150k in   5.064189s

Comparison:
Enumerable#sort_by (Symbol#to_proc):    44742.4 i/s
  Enumerable#sort_by:    41934.8 i/s - 1.07x  (± 0.00) slower
     Enumerable#sort:    15837.4 i/s - 2.83x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/sort-vs-sort_by.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         4.920k i/100ms
  Enumerable#sort_by     5.254k i/100ms
     Enumerable#sort     1.531k i/100ms
Calculating -------------------------------------
Enumerable#sort_by (Symbol#to_proc)
                         50.659k (± 2.6%) i/s -    255.840k in   5.053809s
  Enumerable#sort_by     52.313k (± 2.7%) i/s -    262.700k in   5.025383s
     Enumerable#sort     14.872k (± 3.2%) i/s -     75.019k in   5.049530s

Comparison:
  Enumerable#sort_by:    52313.2 i/s
Enumerable#sort_by (Symbol#to_proc):    50659.0 i/s - same-ish: difference falls within error
     Enumerable#sort:    14872.0 i/s - 3.52x  (± 0.00) slower
```

##### `Enumerable#inject Symbol` vs `Enumerable#inject Proc` [code](code/enumerable/inject-symbol-vs-block.rb)

Of note, `to_proc` for 1.8.7 is considerable slower than the block format

```sh
$ ruby -v --jit code/enumerable/inject-symbol-vs-block.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
       inject symbol    53.051k i/100ms
      inject to_proc     1.620k i/100ms
        inject block     2.023k i/100ms
Calculating -------------------------------------
       inject symbol    550.645k (± 3.4%) i/s -      2.759M in   5.016310s
      inject to_proc     16.512k (± 3.6%) i/s -     82.620k in   5.010290s
        inject block     20.595k (± 2.6%) i/s -    103.173k in   5.013273s

Comparison:
       inject symbol:   550645.3 i/s
        inject block:    20594.5 i/s - 26.74x  (± 0.00) slower
      inject to_proc:    16512.4 i/s - 33.35x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/inject-symbol-vs-block.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
       inject symbol    53.859k i/100ms
      inject to_proc     1.606k i/100ms
        inject block     1.788k i/100ms
Calculating -------------------------------------
       inject symbol    553.381k (± 1.1%) i/s -      2.801M in   5.061602s
      inject to_proc     16.551k (± 2.1%) i/s -     83.512k in   5.048128s
        inject block     17.893k (± 1.7%) i/s -     91.188k in   5.097857s

Comparison:
       inject symbol:   553381.4 i/s
        inject block:    17892.6 i/s - 30.93x  (± 0.00) slower
      inject to_proc:    16550.5 i/s - 33.44x  (± 0.00) slower
```

```sh
$ ruby -v code/enumerable/inject-symbol-vs-block.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
       inject symbol    56.781k i/100ms
      inject to_proc     1.471k i/100ms
        inject block     2.351k i/100ms
Calculating -------------------------------------
       inject symbol    545.587k (± 4.0%) i/s -      2.725M in   5.003945s
      inject to_proc     15.235k (± 8.4%) i/s -     76.492k in   5.067105s
        inject block     22.928k (± 2.4%) i/s -    115.199k in   5.027143s

Comparison:
       inject symbol:   545587.1 i/s
        inject block:    22928.5 i/s - 23.80x  (± 0.00) slower
      inject to_proc:    15235.2 i/s - 35.81x  (± 0.00) slower
```

### Date

##### `Date.iso8601` vs `Date.parse` [code](code/date/iso8601-vs-parse.rb)

When expecting well-formatted data from e.g. an API, `iso8601` is faster and will raise an `ArgumentError` on malformed input.

```sh
$ ruby -v --jit code/date/iso8601-vs-parse.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
        Date.iso8601    47.591k i/100ms
          Date.parse    24.769k i/100ms
Calculating -------------------------------------
        Date.iso8601    507.503k (± 3.5%) i/s -      2.570M in   5.070430s
          Date.parse    262.846k (± 3.8%) i/s -      1.313M in   5.001939s

Comparison:
        Date.iso8601:   507502.5 i/s
          Date.parse:   262846.4 i/s - 1.93x  (± 0.00) slower
```

```sh
$ ruby -v code/date/iso8601-vs-parse.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
        Date.iso8601    51.556k i/100ms
          Date.parse    23.835k i/100ms
Calculating -------------------------------------
        Date.iso8601    525.039k (± 2.4%) i/s -      2.629M in   5.011104s
          Date.parse    272.696k (± 1.1%) i/s -      1.382M in   5.070160s

Comparison:
        Date.iso8601:   525039.3 i/s
          Date.parse:   272696.2 i/s - 1.93x  (± 0.00) slower
```

```sh
$ ruby -v code/date/iso8601-vs-parse.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
        Date.iso8601    50.827k i/100ms
          Date.parse    27.943k i/100ms
Calculating -------------------------------------
        Date.iso8601    526.883k (± 4.7%) i/s -      2.643M in   5.028761s
          Date.parse    278.618k (± 3.6%) i/s -      1.397M in   5.021727s

Comparison:
        Date.iso8601:   526883.0 i/s
          Date.parse:   278617.8 i/s - 1.89x  (± 0.00) slower
```

### Hash

##### `Hash#[]` vs `Hash#fetch` [code](code/hash/bracket-vs-fetch.rb)

If you use Ruby 2.2, `Symbol` could be more performant than `String` as `Hash` keys.
Read more regarding this: [Symbol GC in Ruby 2.2](http://www.sitepoint.com/symbol-gc-ruby-2-2/) and [Unraveling String Key Performance in Ruby 2.2](http://www.sitepoint.com/unraveling-string-key-performance-ruby-2-2/).

```sh
$ ruby -v --jit code/hash/bracket-vs-fetch.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
     Hash#[], symbol     1.488M i/100ms
  Hash#fetch, symbol     1.176M i/100ms
     Hash#[], string     1.233M i/100ms
  Hash#fetch, string   695.462k i/100ms
Calculating -------------------------------------
     Hash#[], symbol     18.142M (± 3.5%) i/s -     90.747M in   5.008424s
  Hash#fetch, symbol     14.440M (± 3.1%) i/s -     72.913M in   5.054387s
     Hash#[], string     14.843M (± 3.7%) i/s -     75.220M in   5.074898s
  Hash#fetch, string      8.679M (± 3.8%) i/s -     43.814M in   5.056129s

Comparison:
     Hash#[], symbol: 18142204.4 i/s
     Hash#[], string: 14843495.6 i/s - 1.22x  (± 0.00) slower
  Hash#fetch, symbol: 14440402.9 i/s - 1.26x  (± 0.00) slower
  Hash#fetch, string:  8678826.6 i/s - 2.09x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/bracket-vs-fetch.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
     Hash#[], symbol     1.605M i/100ms
  Hash#fetch, symbol     1.117M i/100ms
     Hash#[], string     1.399M i/100ms
  Hash#fetch, string   806.076k i/100ms
Calculating -------------------------------------
     Hash#[], symbol     16.065M (± 2.6%) i/s -     81.876M in   5.099991s
  Hash#fetch, symbol     11.423M (± 2.4%) i/s -     58.080M in   5.087752s
     Hash#[], string     13.937M (± 1.5%) i/s -     69.934M in   5.018805s
  Hash#fetch, string      7.840M (± 3.1%) i/s -     39.498M in   5.042873s

Comparison:
     Hash#[], symbol: 16065412.1 i/s
     Hash#[], string: 13937494.1 i/s - 1.15x  (± 0.00) slower
  Hash#fetch, symbol: 11422632.3 i/s - 1.41x  (± 0.00) slower
  Hash#fetch, string:  7840176.0 i/s - 2.05x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/bracket-vs-fetch.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
     Hash#[], symbol     1.670M i/100ms
  Hash#fetch, symbol     1.317M i/100ms
     Hash#[], string     1.322M i/100ms
  Hash#fetch, string   832.589k i/100ms
Calculating -------------------------------------
     Hash#[], symbol     16.772M (± 2.5%) i/s -     85.153M in   5.080340s
  Hash#fetch, symbol     13.234M (± 2.7%) i/s -     67.169M in   5.079409s
     Hash#[], string     14.460M (± 1.8%) i/s -     72.698M in   5.029302s
  Hash#fetch, string      8.283M (± 3.1%) i/s -     41.629M in   5.030929s

Comparison:
     Hash#[], symbol: 16772038.2 i/s
     Hash#[], string: 14459609.9 i/s - 1.16x  (± 0.00) slower
  Hash#fetch, symbol: 13233805.3 i/s - 1.27x  (± 0.00) slower
  Hash#fetch, string:  8283274.0 i/s - 2.02x  (± 0.00) slower
```

##### `Hash#dig` vs `Hash#[]` vs `Hash#fetch` [code](code/hash/dig-vs-[]-vs-fetch.rb)

[Ruby 2.3 introduced `Hash#dig`](http://ruby-doc.org/core-2.3.0/Hash.html#method-i-dig) which is a readable
and performant option for retrieval from a nested hash, returning `nil` if an extraction step fails.
See [#102 (comment)](https://github.com/JuanitoFatas/fast-ruby/pull/102#issuecomment-198827506) for more info.

```sh
$ ruby -v --jit code/hash/dig-vs-\[\]-vs-fetch.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
            Hash#dig   788.821k i/100ms
             Hash#[]     1.035M i/100ms
          Hash#[] ||     1.030M i/100ms
          Hash#[] &&   494.897k i/100ms
          Hash#fetch   619.207k i/100ms
 Hash#fetch fallback   418.777k i/100ms
Calculating -------------------------------------
            Hash#dig      8.916M (± 3.1%) i/s -     44.963M in   5.047975s
             Hash#[]     10.627M (± 6.4%) i/s -     53.807M in   5.086559s
          Hash#[] ||     10.584M (± 4.6%) i/s -     53.548M in   5.070695s
          Hash#[] &&      5.179M (± 3.7%) i/s -     26.230M in   5.072042s
          Hash#fetch      6.921M (± 2.0%) i/s -     34.676M in   5.012572s
 Hash#fetch fallback      4.651M (± 3.7%) i/s -     23.452M in   5.050419s

Comparison:
             Hash#[]: 10626747.0 i/s
          Hash#[] ||: 10583646.3 i/s - same-ish: difference falls within error
            Hash#dig:  8916493.9 i/s - 1.19x  (± 0.00) slower
          Hash#fetch:  6920569.5 i/s - 1.54x  (± 0.00) slower
          Hash#[] &&:  5178816.0 i/s - 2.05x  (± 0.00) slower
 Hash#fetch fallback:  4650644.5 i/s - 2.29x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/dig-vs-\[\]-vs-fetch.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
            Hash#dig   842.835k i/100ms
             Hash#[]   999.041k i/100ms
          Hash#[] ||   932.849k i/100ms
          Hash#[] &&   415.182k i/100ms
          Hash#fetch   553.290k i/100ms
 Hash#fetch fallback   387.675k i/100ms
Calculating -------------------------------------
            Hash#dig      8.322M (± 2.1%) i/s -     42.142M in   5.066380s
             Hash#[]     10.078M (± 2.7%) i/s -     50.951M in   5.059338s
          Hash#[] ||      9.434M (± 2.0%) i/s -     47.575M in   5.045097s
          Hash#[] &&      4.078M (± 2.5%) i/s -     20.759M in   5.094346s
          Hash#fetch      5.607M (± 1.4%) i/s -     28.218M in   5.033312s
 Hash#fetch fallback      3.825M (± 2.7%) i/s -     19.384M in   5.071185s

Comparison:
             Hash#[]: 10078051.2 i/s
          Hash#[] ||:  9433660.4 i/s - 1.07x  (± 0.00) slower
            Hash#dig:  8321562.5 i/s - 1.21x  (± 0.00) slower
          Hash#fetch:  5607337.0 i/s - 1.80x  (± 0.00) slower
          Hash#[] &&:  4077678.2 i/s - 2.47x  (± 0.00) slower
 Hash#fetch fallback:  3825333.9 i/s - 2.63x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/dig-vs-\[\]-vs-fetch.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
            Hash#dig   932.511k i/100ms
             Hash#[]     1.040M i/100ms
          Hash#[] ||   962.285k i/100ms
          Hash#[] &&   413.100k i/100ms
          Hash#fetch   575.071k i/100ms
 Hash#fetch fallback   385.447k i/100ms
Calculating -------------------------------------
            Hash#dig      9.463M (± 2.4%) i/s -     47.558M in   5.028746s
             Hash#[]     10.599M (± 2.2%) i/s -     53.016M in   5.004414s
          Hash#[] ||      9.760M (± 2.1%) i/s -     49.077M in   5.030779s
          Hash#[] &&      4.179M (± 2.7%) i/s -     21.068M in   5.044721s
          Hash#fetch      5.694M (± 3.0%) i/s -     28.754M in   5.054350s
 Hash#fetch fallback      3.770M (± 3.3%) i/s -     18.887M in   5.015947s

Comparison:
             Hash#[]: 10599328.8 i/s
          Hash#[] ||:  9759533.6 i/s - 1.09x  (± 0.00) slower
            Hash#dig:  9463155.8 i/s - 1.12x  (± 0.00) slower
          Hash#fetch:  5694294.6 i/s - 1.86x  (± 0.00) slower
          Hash#[] &&:  4179434.6 i/s - 2.54x  (± 0.00) slower
 Hash#fetch fallback:  3769774.0 i/s - 2.81x  (± 0.00) slower
```

##### `Hash[]` vs `Hash#dup` [code](code/hash/bracket-vs-dup.rb)

Source: http://tenderlovemaking.com/2015/02/11/weird-stuff-with-hashes.html

> Does this mean that you should switch to Hash[]?
> Only if your benchmarks can prove that it’s a bottleneck.
> Please please please don’t change all of your code because
> this shows it’s faster. Make sure to measure your app performance first.

```sh
$ ruby -v --jit code/hash/bracket-vs-dup.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
              Hash[]   150.185k i/100ms
            Hash#dup    99.509k i/100ms
Calculating -------------------------------------
              Hash[]      1.778M (± 4.6%) i/s -      9.011M in   5.078730s
            Hash#dup      1.102M (± 4.7%) i/s -      5.573M in   5.068983s

Comparison:
              Hash[]:  1778077.6 i/s
            Hash#dup:  1101909.7 i/s - 1.61x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/bracket-vs-dup.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
              Hash[]   168.217k i/100ms
            Hash#dup   109.442k i/100ms
Calculating -------------------------------------
              Hash[]      1.709M (± 4.1%) i/s -      8.579M in   5.029715s
            Hash#dup      1.126M (± 2.7%) i/s -      5.691M in   5.059763s

Comparison:
              Hash[]:  1708526.2 i/s
            Hash#dup:  1125573.7 i/s - 1.52x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/bracket-vs-dup.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
              Hash[]   167.669k i/100ms
            Hash#dup   104.623k i/100ms
Calculating -------------------------------------
              Hash[]      1.816M (± 4.3%) i/s -      9.222M in   5.088289s
            Hash#dup      1.102M (± 4.0%) i/s -      5.545M in   5.042178s

Comparison:
              Hash[]:  1815826.5 i/s
            Hash#dup:  1101592.7 i/s - 1.65x  (± 0.00) slower
```

##### `Hash#fetch` with argument vs `Hash#fetch` + block [code](code/hash/fetch-vs-fetch-with-block.rb)

> Note that the speedup in the block version comes from avoiding repeated <br>
> construction of the argument. If the argument is a constant, number symbol or <br>
> something of that sort the argument version is actually slightly faster <br>
> See also [#39 (comment)](https://github.com/JuanitoFatas/fast-ruby/issues/39#issuecomment-103989335)

```sh
$ ruby -v --jit code/hash/fetch-vs-fetch-with-block.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  Hash#fetch + const     1.629M i/100ms
  Hash#fetch + block     1.649M i/100ms
    Hash#fetch + arg     1.217M i/100ms
Calculating -------------------------------------
  Hash#fetch + const     16.079M (± 3.7%) i/s -     81.442M in   5.072266s
  Hash#fetch + block     15.919M (± 5.4%) i/s -     80.825M in   5.093929s
    Hash#fetch + arg     10.915M (± 8.2%) i/s -     54.784M in   5.058418s

Comparison:
  Hash#fetch + const: 16079152.0 i/s
  Hash#fetch + block: 15918566.6 i/s - same-ish: difference falls within error
    Hash#fetch + arg: 10915450.7 i/s - 1.47x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/fetch-vs-fetch-with-block.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  Hash#fetch + const     1.466M i/100ms
  Hash#fetch + block     1.326M i/100ms
    Hash#fetch + arg     1.042M i/100ms
Calculating -------------------------------------
  Hash#fetch + const     14.680M (± 2.2%) i/s -     74.778M in   5.096387s
  Hash#fetch + block     14.140M (± 2.3%) i/s -     71.583M in   5.065369s
    Hash#fetch + arg     11.042M (± 2.0%) i/s -     55.250M in   5.005491s

Comparison:
  Hash#fetch + const: 14679637.6 i/s
  Hash#fetch + block: 14140045.7 i/s - same-ish: difference falls within error
    Hash#fetch + arg: 11042449.5 i/s - 1.33x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/fetch-vs-fetch-with-block.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  Hash#fetch + const     1.505M i/100ms
  Hash#fetch + block     1.510M i/100ms
    Hash#fetch + arg     1.061M i/100ms
Calculating -------------------------------------
  Hash#fetch + const     14.978M (± 3.1%) i/s -     75.273M in   5.030668s
  Hash#fetch + block     15.260M (± 3.3%) i/s -     77.005M in   5.051910s
    Hash#fetch + arg     11.201M (± 3.4%) i/s -     56.258M in   5.028531s

Comparison:
  Hash#fetch + block: 15260227.8 i/s
  Hash#fetch + const: 14977860.8 i/s - same-ish: difference falls within error
    Hash#fetch + arg: 11201042.1 i/s - 1.36x  (± 0.00) slower
```

##### `Hash#each_key` instead of `Hash#keys.each` [code](code/hash/keys-each-vs-each_key.rb)

> `Hash#keys.each` allocates an array of keys;  <br>
> `Hash#each_key` iterates through the keys without allocating a new array.  <br>
> This is the reason why `Hash#each_key` exists.  <br>
> —— @sferik [rails/rails#17099](https://github.com/rails/rails/pull/17099)

```sh
$ ruby -v --jit code/hash/keys-each-vs-each_key.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
      Hash#keys.each   152.506k i/100ms
       Hash#each_key   161.794k i/100ms
Calculating -------------------------------------
      Hash#keys.each      1.777M (± 2.1%) i/s -      8.998M in   5.065938s
       Hash#each_key      1.780M (± 2.3%) i/s -      8.899M in   5.001893s

Comparison:
       Hash#each_key:  1779995.7 i/s
      Hash#keys.each:  1776959.9 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/hash/keys-each-vs-each_key.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
      Hash#keys.each   175.374k i/100ms
       Hash#each_key   174.579k i/100ms
Calculating -------------------------------------
      Hash#keys.each      1.657M (± 4.7%) i/s -      8.418M in   5.091454s
       Hash#each_key      1.742M (± 1.4%) i/s -      8.729M in   5.013318s

Comparison:
       Hash#each_key:  1741501.9 i/s
      Hash#keys.each:  1657193.9 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/hash/keys-each-vs-each_key.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
      Hash#keys.each   166.108k i/100ms
       Hash#each_key   169.679k i/100ms
Calculating -------------------------------------
      Hash#keys.each      1.661M (± 2.4%) i/s -      8.305M in   5.004528s
       Hash#each_key      1.717M (± 2.8%) i/s -      8.654M in   5.045197s

Comparison:
       Hash#each_key:  1716683.3 i/s
      Hash#keys.each:  1660554.2 i/s - same-ish: difference falls within error
```

#### `Hash#key?` instead of `Hash#keys.include?` [code](code/hash/keys-include-vs-key.rb)

> `Hash#keys.include?` allocates an array of keys and performs an O(n) search; <br>
> `Hash#key?` performs an O(1) hash lookup without allocating a new array.

```sh
$ ruby -v --jit code/hash/keys-include-vs-key.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  Hash#keys.include?     1.781k i/100ms
           Hash#key?     1.038M i/100ms
Calculating -------------------------------------
  Hash#keys.include?     19.118k (± 5.3%) i/s -     96.174k in   5.045480s
           Hash#key?     11.218M (± 4.2%) i/s -     56.027M in   5.004384s

Comparison:
           Hash#key?: 11218311.8 i/s
  Hash#keys.include?:    19118.2 i/s - 586.79x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/keys-include-vs-key.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  Hash#keys.include?     1.257k i/100ms
           Hash#key?   982.511k i/100ms
Calculating -------------------------------------
  Hash#keys.include?     13.116k (± 2.6%) i/s -     66.621k in   5.083040s
           Hash#key?      9.754M (± 1.6%) i/s -     49.126M in   5.037968s

Comparison:
           Hash#key?:  9753500.8 i/s
  Hash#keys.include?:    13116.1 i/s - 743.63x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/keys-include-vs-key.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  Hash#keys.include?   960.000  i/100ms
           Hash#key?     1.109M i/100ms
Calculating -------------------------------------
  Hash#keys.include?      9.973k (± 3.9%) i/s -     49.920k in   5.013426s
           Hash#key?     11.017M (± 2.9%) i/s -     55.449M in   5.037173s

Comparison:
           Hash#key?: 11017461.3 i/s
  Hash#keys.include?:     9973.1 i/s - 1104.71x  (± 0.00) slower
```

##### `Hash#value?` instead of `Hash#values.include?` [code](code/hash/values-include-vs-value.rb)

> `Hash#values.include?` allocates an array of values and performs an O(n) search; <br>
> `Hash#value?` performs an O(n) search without allocating a new array.

```sh
$ ruby -v --jit code/hash/values-include-vs-value.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Hash#values.include?     1.930k i/100ms
         Hash#value?     2.051k i/100ms
Calculating -------------------------------------
Hash#values.include?     22.631k (± 4.9%) i/s -    113.870k in   5.044388s
         Hash#value?     21.367k (± 3.2%) i/s -    108.703k in   5.092867s

Comparison:
Hash#values.include?:    22630.9 i/s
         Hash#value?:    21366.8 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/hash/values-include-vs-value.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Hash#values.include?     1.621k i/100ms
         Hash#value?     1.404k i/100ms
Calculating -------------------------------------
Hash#values.include?     16.953k (± 4.2%) i/s -     85.913k in   5.077191s
         Hash#value?     14.433k (± 3.2%) i/s -     73.008k in   5.064158s

Comparison:
Hash#values.include?:    16953.1 i/s
         Hash#value?:    14432.9 i/s - 1.17x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/values-include-vs-value.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Hash#values.include?   992.000  i/100ms
         Hash#value?   830.000  i/100ms
Calculating -------------------------------------
Hash#values.include?     10.007k (± 3.9%) i/s -     50.592k in   5.063909s
         Hash#value?      8.084k (± 3.6%) i/s -     40.670k in   5.037976s

Comparison:
Hash#values.include?:    10006.5 i/s
         Hash#value?:     8083.9 i/s - 1.24x  (± 0.00) slower
```

##### `Hash#merge!` vs `Hash#[]=` [code](code/hash/merge-bang-vs-\[\]=.rb)

```sh
$ ruby -v --jit code/hash/merge-bang-vs-\[\]=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
         Hash#merge!     3.746k i/100ms
            Hash#[]=     7.883k i/100ms
Calculating -------------------------------------
         Hash#merge!     40.487k (± 3.7%) i/s -    202.284k in   5.003450s
            Hash#[]=     95.270k (± 3.2%) i/s -    480.863k in   5.052738s

Comparison:
            Hash#[]=:    95269.5 i/s
         Hash#merge!:    40487.4 i/s - 2.35x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-bang-vs-\[\]=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
         Hash#merge!     4.035k i/100ms
            Hash#[]=     8.685k i/100ms
Calculating -------------------------------------
         Hash#merge!     39.980k (± 2.6%) i/s -    201.750k in   5.049826s
            Hash#[]=     88.413k (± 2.5%) i/s -    442.935k in   5.013067s

Comparison:
            Hash#[]=:    88412.7 i/s
         Hash#merge!:    39980.1 i/s - 2.21x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-bang-vs-\[\]=.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
         Hash#merge!     5.578k i/100ms
            Hash#[]=    10.063k i/100ms
Calculating -------------------------------------
         Hash#merge!     53.152k (± 3.7%) i/s -    267.744k in   5.044457s
            Hash#[]=    101.188k (± 2.6%) i/s -    513.213k in   5.075264s

Comparison:
            Hash#[]=:   101188.1 i/s
         Hash#merge!:    53151.8 i/s - 1.90x  (± 0.00) slower
```

##### `Hash#merge` vs `Hash#**other` [code](code/hash/merge-vs-double-splat-operator.rb)

```sh
$ ruby -v --jit code/hash/merge-vs-double-splat-operator.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
        Hash#**other   396.354k i/100ms
          Hash#merge   336.852k i/100ms
Calculating -------------------------------------
        Hash#**other      4.458M (± 2.8%) i/s -     22.592M in   5.072256s
          Hash#merge      3.630M (± 3.9%) i/s -     18.190M in   5.019512s

Comparison:
        Hash#**other:  4457521.6 i/s
          Hash#merge:  3629617.5 i/s - 1.23x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-vs-double-splat-operator.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
        Hash#**other   404.572k i/100ms
          Hash#merge   340.424k i/100ms
Calculating -------------------------------------
        Hash#**other      4.097M (± 3.3%) i/s -     20.633M in   5.042229s
          Hash#merge      3.517M (± 2.1%) i/s -     17.702M in   5.035749s

Comparison:
        Hash#**other:  4096743.7 i/s
          Hash#merge:  3516887.8 i/s - 1.16x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-vs-double-splat-operator.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
        Hash#**other   411.898k i/100ms
          Hash#merge   363.442k i/100ms
Calculating -------------------------------------
        Hash#**other      4.176M (± 3.0%) i/s -     21.007M in   5.034997s
          Hash#merge      3.569M (± 2.0%) i/s -     18.172M in   5.094281s

Comparison:
        Hash#**other:  4175941.9 i/s
          Hash#merge:  3568589.3 i/s - 1.17x  (± 0.00) slower
```

##### `Hash#merge` vs `Hash#merge!` [code](code/hash/merge-vs-merge-bang.rb)

```sh
$ ruby -v --jit code/hash/merge-vs-merge-bang.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
          Hash#merge     1.083k i/100ms
         Hash#merge!     3.706k i/100ms
Calculating -------------------------------------
          Hash#merge     12.453k (± 4.0%) i/s -     62.814k in   5.052139s
         Hash#merge!     39.927k (± 2.9%) i/s -    200.124k in   5.016487s

Comparison:
         Hash#merge!:    39927.3 i/s
          Hash#merge:    12453.1 i/s - 3.21x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-vs-merge-bang.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
          Hash#merge     1.260k i/100ms
         Hash#merge!     3.974k i/100ms
Calculating -------------------------------------
          Hash#merge     12.490k (± 3.4%) i/s -     63.000k in   5.049811s
         Hash#merge!     39.143k (± 2.8%) i/s -    198.700k in   5.080498s

Comparison:
         Hash#merge!:    39143.5 i/s
          Hash#merge:    12490.5 i/s - 3.13x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-vs-merge-bang.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
          Hash#merge     1.346k i/100ms
         Hash#merge!     5.094k i/100ms
Calculating -------------------------------------
          Hash#merge     13.423k (± 3.9%) i/s -     67.300k in   5.021423s
         Hash#merge!     53.258k (± 2.6%) i/s -    269.982k in   5.072735s

Comparison:
         Hash#merge!:    53258.2 i/s
          Hash#merge:    13423.3 i/s - 3.97x  (± 0.00) slower
```

##### `{}#merge!(Hash)` vs `Hash#merge({})` vs `Hash#dup#merge!({})` [code](code/hash/merge-bang-vs-merge-vs-dup-merge-bang.rb)

> When we don't want to modify the original hash, and we want duplicates to be created <br>
> See [#42](https://github.com/JuanitoFatas/fast-ruby/pull/42#issue-93502261) for more details.

```sh
$ ruby -v --jit code/hash/merge-bang-vs-merge-vs-dup-merge-bang.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
{}#merge!(Hash) do end
                         5.129k i/100ms
      Hash#merge({})     3.799k i/100ms
 Hash#dup#merge!({})     2.580k i/100ms
Calculating -------------------------------------
{}#merge!(Hash) do end
                         53.618k (± 3.8%) i/s -    271.837k in   5.077238s
      Hash#merge({})     40.874k (± 6.1%) i/s -    205.146k in   5.039384s
 Hash#dup#merge!({})     25.130k (± 8.4%) i/s -    126.420k in   5.075490s

Comparison:
{}#merge!(Hash) do end:    53618.5 i/s
      Hash#merge({}):    40873.7 i/s - 1.31x  (± 0.00) slower
 Hash#dup#merge!({}):    25130.2 i/s - 2.13x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-bang-vs-merge-vs-dup-merge-bang.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
{}#merge!(Hash) do end
                         5.050k i/100ms
      Hash#merge({})     3.983k i/100ms
 Hash#dup#merge!({})     2.549k i/100ms
Calculating -------------------------------------
{}#merge!(Hash) do end
                         49.361k (± 3.2%) i/s -    247.450k in   5.018607s
      Hash#merge({})     39.060k (± 4.2%) i/s -    195.167k in   5.005803s
 Hash#dup#merge!({})     25.347k (± 3.7%) i/s -    127.450k in   5.035586s

Comparison:
{}#merge!(Hash) do end:    49361.1 i/s
      Hash#merge({}):    39059.6 i/s - 1.26x  (± 0.00) slower
 Hash#dup#merge!({}):    25347.0 i/s - 1.95x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/merge-bang-vs-merge-vs-dup-merge-bang.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
{}#merge!(Hash) do end
                         5.382k i/100ms
      Hash#merge({})     4.444k i/100ms
 Hash#dup#merge!({})     2.757k i/100ms
Calculating -------------------------------------
{}#merge!(Hash) do end
                         53.596k (± 3.8%) i/s -    269.100k in   5.028576s
      Hash#merge({})     43.599k (± 2.5%) i/s -    222.200k in   5.099614s
 Hash#dup#merge!({})     29.819k (± 3.7%) i/s -    148.878k in   5.000324s

Comparison:
{}#merge!(Hash) do end:    53595.7 i/s
      Hash#merge({}):    43599.1 i/s - 1.23x  (± 0.00) slower
 Hash#dup#merge!({}):    29819.0 i/s - 1.80x  (± 0.00) slower
```

##### `Hash#sort_by` vs `Hash#sort` [code](code/hash/hash-key-sort_by-vs-sort.rb)

To sort hash by key.

```sh
$ ruby -v --jit code/hash/hash-key-sort_by-vs-sort.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
      sort_by + to_h    22.460k i/100ms
         sort + to_h     9.181k i/100ms
Calculating -------------------------------------
      sort_by + to_h    242.381k (± 4.3%) i/s -      1.213M in   5.013503s
         sort + to_h     92.723k (± 3.4%) i/s -    468.231k in   5.055848s

Comparison:
      sort_by + to_h:   242380.9 i/s
         sort + to_h:    92722.9 i/s - 2.61x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/hash-key-sort_by-vs-sort.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
      sort_by + to_h    21.591k i/100ms
         sort + to_h     7.403k i/100ms
Calculating -------------------------------------
      sort_by + to_h    221.314k (± 3.1%) i/s -      1.123M in   5.077919s
         sort + to_h     73.945k (± 2.0%) i/s -    370.150k in   5.007840s

Comparison:
      sort_by + to_h:   221314.0 i/s
         sort + to_h:    73944.6 i/s - 2.99x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/hash-key-sort_by-vs-sort.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
      sort_by + to_h    26.948k i/100ms
         sort + to_h    10.061k i/100ms
Calculating -------------------------------------
      sort_by + to_h    266.736k (± 2.8%) i/s -      1.347M in   5.055470s
         sort + to_h    102.361k (± 3.9%) i/s -    513.111k in   5.020611s

Comparison:
      sort_by + to_h:   266736.0 i/s
         sort + to_h:   102361.4 i/s - 2.61x  (± 0.00) slower
```

##### Native `Hash#slice` vs other slice implementations before native [code](code/hash/slice-native-vs-before-native.rb)

Since ruby 2.5, Hash comes with a `slice` method to select hash members by keys.

```sh
$ ruby -v --jit code/hash/slice-native-vs-before-native.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
Hash#native-slice      444.133k i/100ms
Array#each             244.753k i/100ms
Array#each_w/_object   186.698k i/100ms
Hash#select-include     70.843k i/100ms
Calculating -------------------------------------
Hash#native-slice         4.993M (± 2.7%) i/s -     25.316M in   5.073964s
Array#each                2.746M (± 2.7%) i/s -     13.951M in   5.084488s
Array#each_w/_object      2.007M (± 2.6%) i/s -     10.082M in   5.027616s
Hash#select-include     705.794k (± 3.0%) i/s -      3.542M in   5.023555s

Comparison:
Hash#native-slice   :  4993037.5 i/s
Array#each          :  2745867.0 i/s - 1.82x  (± 0.00) slower
Array#each_w/_object:  2006645.6 i/s - 2.49x  (± 0.00) slower
Hash#select-include :   705794.0 i/s - 7.07x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/slice-native-vs-before-native.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
Hash#native-slice      457.272k i/100ms
Array#each             215.397k i/100ms
Array#each_w/_object   177.775k i/100ms
Hash#select-include     61.668k i/100ms
Calculating -------------------------------------
Hash#native-slice         4.785M (± 3.0%) i/s -     24.235M in   5.069925s
Array#each                2.243M (± 1.9%) i/s -     11.416M in   5.091981s
Array#each_w/_object      1.740M (± 2.9%) i/s -      8.711M in   5.009844s
Hash#select-include     647.784k (± 2.5%) i/s -      3.268M in   5.048723s

Comparison:
Hash#native-slice   :  4784771.9 i/s
Array#each          :  2242796.4 i/s - 2.13x  (± 0.00) slower
Array#each_w/_object:  1740377.9 i/s - 2.75x  (± 0.00) slower
Hash#select-include :   647784.4 i/s - 7.39x  (± 0.00) slower
```

```sh
$ ruby -v code/hash/slice-native-vs-before-native.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
Hash#native-slice      509.094k i/100ms
Array#each             209.297k i/100ms
Array#each_w/_object   162.088k i/100ms
Hash#select-include     84.746k i/100ms
Calculating -------------------------------------
Hash#native-slice         5.121M (± 2.5%) i/s -     25.964M in   5.073042s
Array#each                2.291M (± 3.4%) i/s -     11.511M in   5.031372s
Array#each_w/_object      1.926M (± 2.1%) i/s -      9.725M in   5.052405s
Hash#select-include     889.123k (± 2.5%) i/s -      4.492M in   5.054825s

Comparison:
Hash#native-slice   :  5121430.6 i/s
Array#each          :  2290614.8 i/s - 2.24x  (± 0.00) slower
Array#each_w/_object:  1925780.8 i/s - 2.66x  (± 0.00) slower
Hash#select-include :   889123.2 i/s - 5.76x  (± 0.00) slower
```


### Proc & Block

##### Block vs `Symbol#to_proc` [code](code/proc-and-block/block-vs-to_proc.rb)

> `Symbol#to_proc` is considerably more concise than using block syntax. <br>
> ...In some cases, it reduces the number of lines of code. <br>
> —— @sferik [rails/rails#16833](https://github.com/rails/rails/pull/16833)

```sh
$ ruby -v --jit code/proc-and-block/block-vs-to_proc.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
               Block     8.559k i/100ms
      Symbol#to_proc     8.381k i/100ms
Calculating -------------------------------------
               Block     85.127k (± 3.4%) i/s -    427.950k in   5.033066s
      Symbol#to_proc     86.264k (± 2.9%) i/s -    435.812k in   5.056460s

Comparison:
      Symbol#to_proc:    86263.6 i/s
               Block:    85126.6 i/s - same-ish: difference falls within error
```

```sh
$ ruby -v code/proc-and-block/block-vs-to_proc.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
               Block     7.666k i/100ms
      Symbol#to_proc     8.617k i/100ms
Calculating -------------------------------------
               Block     78.325k (± 4.3%) i/s -    390.966k in   5.001360s
      Symbol#to_proc     87.072k (± 3.2%) i/s -    439.467k in   5.052488s

Comparison:
      Symbol#to_proc:    87072.1 i/s
               Block:    78324.9 i/s - 1.11x  (± 0.00) slower
```

```sh
$ ruby -v code/proc-and-block/block-vs-to_proc.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
               Block     8.088k i/100ms
      Symbol#to_proc     8.522k i/100ms
Calculating -------------------------------------
               Block     81.927k (± 5.4%) i/s -    412.488k in   5.052135s
      Symbol#to_proc     82.441k (± 2.9%) i/s -    417.578k in   5.069661s

Comparison:
      Symbol#to_proc:    82440.9 i/s
               Block:    81927.1 i/s - same-ish: difference falls within error
```

##### `Proc#call` and block arguments vs `yield`[code](code/proc-and-block/proc-call-vs-yield.rb)

In MRI Ruby before 2.5, block arguments [are converted to Procs](https://www.omniref.com/ruby/2.2.0/symbols/Proc/yield?#annotation=4087638&line=711), which incurs a heap allocation.

```sh
$ ruby -v --jit code/proc-and-block/proc-call-vs-yield.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
          block.call   837.937k i/100ms
       block + yield   881.330k i/100ms
        unused block     1.225M i/100ms
               yield     1.162M i/100ms
Calculating -------------------------------------
          block.call      9.474M (± 3.5%) i/s -     47.762M in   5.048002s
       block + yield     10.345M (± 3.1%) i/s -     51.998M in   5.031541s
        unused block     13.013M (± 3.1%) i/s -     66.172M in   5.090160s
               yield     16.058M (± 2.7%) i/s -     81.311M in   5.067338s

Comparison:
               yield: 16058328.1 i/s
        unused block: 13012664.2 i/s - 1.23x  (± 0.00) slower
       block + yield: 10344591.6 i/s - 1.55x  (± 0.00) slower
          block.call:  9473545.9 i/s - 1.70x  (± 0.00) slower
```

```sh
$ ruby -v code/proc-and-block/proc-call-vs-yield.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
          block.call   959.986k i/100ms
       block + yield     1.042M i/100ms
        unused block     1.371M i/100ms
               yield     1.332M i/100ms
Calculating -------------------------------------
          block.call      9.541M (± 3.2%) i/s -     47.999M in   5.036125s
       block + yield     10.199M (± 3.0%) i/s -     51.049M in   5.009943s
        unused block     13.483M (± 2.0%) i/s -     68.537M in   5.085278s
               yield     13.264M (± 1.4%) i/s -     66.613M in   5.023021s

Comparison:
        unused block: 13483107.0 i/s
               yield: 13264096.2 i/s - same-ish: difference falls within error
       block + yield: 10199420.0 i/s - 1.32x  (± 0.00) slower
          block.call:  9541063.8 i/s - 1.41x  (± 0.00) slower
```

```sh
$ ruby -v code/proc-and-block/proc-call-vs-yield.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
          block.call   992.187k i/100ms
       block + yield     1.108M i/100ms
        unused block     1.403M i/100ms
               yield     1.430M i/100ms
Calculating -------------------------------------
          block.call      9.808M (± 3.7%) i/s -     49.609M in   5.065464s
       block + yield     10.969M (± 3.1%) i/s -     55.384M in   5.054240s
        unused block     14.248M (± 2.7%) i/s -     71.562M in   5.026543s
               yield     14.616M (± 2.5%) i/s -     74.349M in   5.090331s

Comparison:
               yield: 14615579.2 i/s
        unused block: 14247708.2 i/s - same-ish: difference falls within error
       block + yield: 10969036.9 i/s - 1.33x  (± 0.00) slower
          block.call:  9807623.2 i/s - 1.49x  (± 0.00) slower
```

### String

##### `String#dup` vs `String#+` [code](code/string/dup-vs-unary-plus.rb)

Note that `String.new` is not the same as the options compared, since it is
always `ASCII-8BIT` encoded instead of the script encoding (usually `UTF-8`).

```sh
$ ruby -v --jit code/string/dup-vs-unary-plus.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
           String#+@     1.013M i/100ms
          String#dup   480.228k i/100ms
Calculating -------------------------------------
           String#+@     12.182M (± 2.9%) i/s -     61.767M in   5.074863s
          String#dup      5.184M (± 2.4%) i/s -     25.932M in   5.005456s

Comparison:
           String#+@: 12181797.0 i/s
          String#dup:  5183822.4 i/s - 2.35x  (± 0.00) slower
```

```sh
$ ruby -v code/string/dup-vs-unary-plus.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
           String#+@     1.095M i/100ms
          String#dup   487.249k i/100ms
Calculating -------------------------------------
           String#+@     10.859M (± 3.3%) i/s -     54.741M in   5.047154s
          String#dup      4.857M (± 2.9%) i/s -     24.362M in   5.020146s

Comparison:
           String#+@: 10858509.0 i/s
          String#dup:  4857027.5 i/s - 2.24x  (± 0.00) slower
```

```sh
$ ruby -v code/string/dup-vs-unary-plus.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
           String#+@     1.156M i/100ms
          String#dup   528.311k i/100ms
Calculating -------------------------------------
           String#+@     11.440M (± 2.2%) i/s -     57.820M in   5.056817s
          String#dup      5.679M (± 2.2%) i/s -     28.529M in   5.025902s

Comparison:
           String#+@: 11439888.1 i/s
          String#dup:  5679197.1 i/s - 2.01x  (± 0.00) slower
```

##### `String#casecmp` vs `String#downcase + ==` [code](code/string/casecmp-vs-downcase-==.rb)

```sh
$ ruby -v --jit code/string/casecmp-vs-downcase-\=\=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
String#downcase + ==   587.123k i/100ms
      String#casecmp   747.701k i/100ms
Calculating -------------------------------------
String#downcase + ==      7.290M (± 3.0%) i/s -     36.989M in   5.078497s
      String#casecmp      9.485M (± 2.8%) i/s -     47.853M in   5.049197s

Comparison:
      String#casecmp:  9484850.1 i/s
String#downcase + ==:  7290452.8 i/s - 1.30x  (± 0.00) slower
```

```sh
$ ruby -v code/string/casecmp-vs-downcase-\=\=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
String#downcase + ==   651.671k i/100ms
      String#casecmp   809.045k i/100ms
Calculating -------------------------------------
String#downcase + ==      6.582M (± 2.2%) i/s -     33.235M in   5.052305s
      String#casecmp      8.263M (± 2.5%) i/s -     42.070M in   5.095027s

Comparison:
      String#casecmp:  8262549.9 i/s
String#downcase + ==:  6581573.7 i/s - 1.26x  (± 0.00) slower
```

```sh
$ ruby -v code/string/casecmp-vs-downcase-\=\=.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
String#downcase + ==   660.304k i/100ms
      String#casecmp   865.754k i/100ms
Calculating -------------------------------------
String#downcase + ==      6.496M (± 2.9%) i/s -     33.015M in   5.087185s
      String#casecmp      8.698M (± 2.5%) i/s -     44.153M in   5.079708s

Comparison:
      String#casecmp:  8698031.0 i/s
String#downcase + ==:  6495624.9 i/s - 1.34x  (± 0.00) slower
```

##### String Concatenation [code](code/string/concatenation.rb)

```sh
$ ruby -v --jit code/string/concatenation.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
            String#+   658.626k i/100ms
       String#concat   580.936k i/100ms
       String#append   626.352k i/100ms
         "foo" "bar"     1.352M i/100ms
  "#{'foo'}#{'bar'}"     1.417M i/100ms
Calculating -------------------------------------
            String#+      6.886M (± 3.3%) i/s -     34.907M in   5.075113s
       String#concat      6.699M (± 4.0%) i/s -     33.694M in   5.038430s
       String#append      6.762M (± 8.9%) i/s -     33.823M in   5.048178s
         "foo" "bar"     13.876M (±18.6%) i/s -     66.257M in   5.054887s
  "#{'foo'}#{'bar'}"     15.426M (± 6.6%) i/s -     77.936M in   5.081120s

Comparison:
  "#{'foo'}#{'bar'}": 15426061.7 i/s
         "foo" "bar": 13876099.3 i/s - same-ish: difference falls within error
            String#+:  6885999.9 i/s - 2.24x  (± 0.00) slower
       String#append:  6761700.4 i/s - 2.28x  (± 0.00) slower
       String#concat:  6698589.7 i/s - 2.30x  (± 0.00) slower
```

```sh
$ ruby -v code/string/concatenation.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
            String#+   649.864k i/100ms
       String#concat   596.597k i/100ms
       String#append   699.015k i/100ms
         "foo" "bar"     1.365M i/100ms
  "#{'foo'}#{'bar'}"     1.352M i/100ms
Calculating -------------------------------------
            String#+      6.507M (± 1.6%) i/s -     33.143M in   5.094806s
       String#concat      6.254M (± 2.3%) i/s -     31.620M in   5.058921s
       String#append      6.980M (± 3.7%) i/s -     34.951M in   5.014667s
         "foo" "bar"     13.617M (± 1.6%) i/s -     68.247M in   5.013219s
  "#{'foo'}#{'bar'}"     13.502M (± 3.2%) i/s -     67.596M in   5.011488s

Comparison:
         "foo" "bar": 13616771.4 i/s
  "#{'foo'}#{'bar'}": 13502380.0 i/s - same-ish: difference falls within error
       String#append:  6980304.4 i/s - 1.95x  (± 0.00) slower
            String#+:  6507053.8 i/s - 2.09x  (± 0.00) slower
       String#concat:  6253518.7 i/s - 2.18x  (± 0.00) slower
```

```sh
$ ruby -v code/string/concatenation.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
            String#+   635.358k i/100ms
       String#concat   677.475k i/100ms
       String#append   763.428k i/100ms
         "foo" "bar"     1.425M i/100ms
  "#{'foo'}#{'bar'}"     1.431M i/100ms
Calculating -------------------------------------
            String#+      6.637M (± 2.5%) i/s -     33.674M in   5.076676s
       String#concat      6.552M (± 4.3%) i/s -     33.196M in   5.075904s
       String#append      7.511M (± 3.8%) i/s -     38.171M in   5.090087s
         "foo" "bar"     14.070M (± 3.7%) i/s -     71.244M in   5.071199s
  "#{'foo'}#{'bar'}"     14.256M (± 2.4%) i/s -     71.538M in   5.021101s

Comparison:
  "#{'foo'}#{'bar'}": 14255987.5 i/s
         "foo" "bar": 14069774.4 i/s - same-ish: difference falls within error
       String#append:  7510942.7 i/s - 1.90x  (± 0.00) slower
            String#+:  6637319.7 i/s - 2.15x  (± 0.00) slower
       String#concat:  6552480.8 i/s - 2.18x  (± 0.00) slower
```

##### `String#match` vs `String.match?` vs `String#start_with?`/`String#end_with?` [code (start)](code/string/start-string-checking-match-vs-start_with.rb) [code (end)](code/string/end-string-checking-match-vs-end_with.rb)

The regular expression approaches become slower as the tested string becomes
longer. For short strings, `String#match?` performs similarly to
`String#start_with?`/`String#end_with?`.

> :warning: <br>
> Sometimes you cant replace regexp with `start_with?`, <br>
> for example: `"a\nb" =~ /^b/ #=> 2` but `"a\nb" =~ /\Ab/ #=> nil`.<br>
> :warning: <br>
> You can combine `start_with?` and `end_with?` to replace
> `error.path =~ /^#{path}(\.rb)?$/` to this <br>
> `error.path.start_with?(path) && error.path.end_with?('.rb', '')`<br>
> —— @igas [rails/rails#17316](https://github.com/rails/rails/pull/17316)

```sh
$ ruby -v --jit code/string/start-string-checking-match-vs-start_with.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
           String#=~   124.031k i/100ms
       String#match?   624.310k i/100ms
  String#start_with?   802.805k i/100ms
Calculating -------------------------------------
           String#=~      1.426M (± 2.5%) i/s -      7.194M in   5.049303s
       String#match?      8.661M (± 2.1%) i/s -     43.702M in   5.048017s
  String#start_with?     10.216M (± 3.1%) i/s -     51.380M in   5.034336s

Comparison:
  String#start_with?: 10216201.7 i/s
       String#match?:  8660883.6 i/s - 1.18x  (± 0.00) slower
           String#=~:  1425638.0 i/s - 7.17x  (± 0.00) slower
```

```sh
$ ruby -v code/string/start-string-checking-match-vs-start_with.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
           String#=~   136.027k i/100ms
       String#match?   762.880k i/100ms
  String#start_with?   901.527k i/100ms
Calculating -------------------------------------
           String#=~      1.410M (± 3.2%) i/s -      7.073M in   5.022186s
       String#match?      7.567M (± 2.2%) i/s -     38.144M in   5.043414s
  String#start_with?      8.923M (± 6.2%) i/s -     45.076M in   5.081288s

Comparison:
  String#start_with?:  8922735.9 i/s
       String#match?:  7566919.3 i/s - 1.18x  (± 0.00) slower
           String#=~:  1409972.3 i/s - 6.33x  (± 0.00) slower
```

```sh
$ ruby -v code/string/start-string-checking-match-vs-start_with.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
           String#=~   144.655k i/100ms
       String#match?   785.809k i/100ms
  String#start_with?   950.595k i/100ms
Calculating -------------------------------------
           String#=~      1.396M (± 3.9%) i/s -      7.088M in   5.084480s
       String#match?      7.896M (± 2.7%) i/s -     40.076M in   5.079021s
  String#start_with?      9.661M (± 2.6%) i/s -     48.480M in   5.021590s

Comparison:
  String#start_with?:  9661270.4 i/s
       String#match?:  7896332.3 i/s - 1.22x  (± 0.00) slower
           String#=~:  1396253.5 i/s - 6.92x  (± 0.00) slower
```

```sh
$ ruby -v --jit code/string/end-string-checking-match-vs-end_with.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
           String#=~   109.295k i/100ms
       String#match?   357.956k i/100ms
    String#end_with?   637.190k i/100ms
Calculating -------------------------------------
           String#=~      1.137M (± 5.3%) i/s -      5.683M in   5.017003s
       String#match?      4.451M (± 3.3%) i/s -     22.551M in   5.072534s
    String#end_with?      7.294M (± 2.5%) i/s -     36.957M in   5.069932s

Comparison:
    String#end_with?:  7294140.5 i/s
       String#match?:  4450957.1 i/s - 1.64x  (± 0.00) slower
           String#=~:  1136524.8 i/s - 6.42x  (± 0.00) slower
```

```sh
$ ruby -v code/string/end-string-checking-match-vs-end_with.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
           String#=~   114.743k i/100ms
       String#match?   421.623k i/100ms
    String#end_with?   665.948k i/100ms
Calculating -------------------------------------
           String#=~      1.146M (± 2.3%) i/s -      5.737M in   5.010598s
       String#match?      4.224M (± 2.3%) i/s -     21.503M in   5.093533s
    String#end_with?      6.622M (± 1.3%) i/s -     33.297M in   5.029168s

Comparison:
    String#end_with?:  6622046.8 i/s
       String#match?:  4223944.4 i/s - 1.57x  (± 0.00) slower
           String#=~:  1145606.4 i/s - 5.78x  (± 0.00) slower
```

```sh
$ ruby -v code/string/end-string-checking-match-vs-end_with.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
           String#=~   116.900k i/100ms
       String#match?   423.776k i/100ms
    String#end_with?   689.014k i/100ms
Calculating -------------------------------------
           String#=~      1.140M (± 3.7%) i/s -      5.728M in   5.029970s
       String#match?      4.342M (± 2.5%) i/s -     22.036M in   5.078842s
    String#end_with?      6.903M (± 3.0%) i/s -     35.140M in   5.095493s

Comparison:
    String#end_with?:  6903230.7 i/s
       String#match?:  4341555.7 i/s - 1.59x  (± 0.00) slower
           String#=~:  1140426.0 i/s - 6.05x  (± 0.00) slower
```

##### `String#start_with?` vs `String#[].==` [code](code/string/start_with-vs-substring-==.rb)

```sh
$ ruby -v --jit code/string/start_with-vs-substring-==.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  String#start_with?   218.819k i/100ms
    String#[0, n] ==    83.744k i/100ms
   String#[RANGE] ==    76.529k i/100ms
   String#[0...n] ==    49.040k i/100ms
Calculating -------------------------------------
  String#start_with?      2.117M (± 4.0%) i/s -     10.722M in   5.072988s
    String#[0, n] ==    815.536k (± 2.1%) i/s -      4.103M in   5.033880s
   String#[RANGE] ==    753.375k (± 3.2%) i/s -      3.826M in   5.084446s
   String#[0...n] ==    475.219k (± 3.8%) i/s -      2.403M in   5.064113s

Comparison:
  String#start_with?:  2117168.3 i/s
    String#[0, n] ==:   815535.9 i/s - 2.60x  (± 0.00) slower
   String#[RANGE] ==:   753375.2 i/s - 2.81x  (± 0.00) slower
   String#[0...n] ==:   475219.3 i/s - 4.46x  (± 0.00) slower
```

```sh
$ ruby -v code/string/start_with-vs-substring-==.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  String#start_with?   206.099k i/100ms
    String#[0, n] ==    82.322k i/100ms
   String#[RANGE] ==    76.971k i/100ms
   String#[0...n] ==    49.503k i/100ms
Calculating -------------------------------------
  String#start_with?      2.172M (± 2.0%) i/s -     10.923M in   5.030505s
    String#[0, n] ==    823.822k (± 2.7%) i/s -      4.116M in   5.000296s
   String#[RANGE] ==    753.566k (± 3.6%) i/s -      3.772M in   5.011794s
   String#[0...n] ==    485.622k (± 3.6%) i/s -      2.426M in   5.001708s

Comparison:
  String#start_with?:  2172371.2 i/s
    String#[0, n] ==:   823821.9 i/s - 2.64x  (± 0.00) slower
   String#[RANGE] ==:   753566.0 i/s - 2.88x  (± 0.00) slower
   String#[0...n] ==:   485621.6 i/s - 4.47x  (± 0.00) slower
```

```sh
$ ruby -v code/string/start_with-vs-substring-==.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  String#start_with?   244.481k i/100ms
    String#[0, n] ==    83.543k i/100ms
   String#[RANGE] ==    77.058k i/100ms
   String#[0...n] ==    48.903k i/100ms
Calculating -------------------------------------
  String#start_with?      2.402M (± 3.1%) i/s -     12.224M in   5.095057s
    String#[0, n] ==    807.144k (± 5.8%) i/s -      4.094M in   5.092458s
   String#[RANGE] ==    744.363k (± 5.7%) i/s -      3.776M in   5.091404s
   String#[0...n] ==    497.864k (± 2.9%) i/s -      2.494M in   5.013742s

Comparison:
  String#start_with?:  2401642.7 i/s
    String#[0, n] ==:   807143.8 i/s - 2.98x  (± 0.00) slower
   String#[RANGE] ==:   744363.0 i/s - 3.23x  (± 0.00) slower
   String#[0...n] ==:   497864.0 i/s - 4.82x  (± 0.00) slower
```

##### `Regexp#===` vs `String#match` vs `String#=~` vs `String#match?` [code ](code/string/===-vs-=~-vs-match.rb)

`String#match?` and `Regexp#match?` are available on Ruby 2.4 or later.
ActiveSupport [provides](http://guides.rubyonrails.org/v5.1/active_support_core_extensions.html#match-questionmark)
a forward compatible extension of `Regexp` for older Rubies without the speed
improvement.

> :warning: <br>
> Sometimes you can't replace `match` with `match?`, <br>
> This is only useful for cases where you are checking <br>
> for a match and not using the resultant match object. <br>
> :warning: <br>
> `Regexp#===` is also faster than `String#match` but you need to switch the order of arguments.

```sh
$ ruby -v --jit code/string/===-vs-=~-vs-match.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
       String#match?     1.016M i/100ms
           String#=~   329.471k i/100ms
          Regexp#===   310.600k i/100ms
        String#match   273.896k i/100ms
Calculating -------------------------------------
       String#match?     10.835M (± 2.4%) i/s -     54.839M in   5.064213s
           String#=~      3.629M (± 3.9%) i/s -     18.121M in   5.001674s
          Regexp#===      3.398M (± 3.8%) i/s -     17.083M in   5.035291s
        String#match      2.979M (± 2.6%) i/s -     15.064M in   5.060805s

Comparison:
       String#match?: 10835269.3 i/s
           String#=~:  3628877.5 i/s - 2.99x  (± 0.00) slower
          Regexp#===:  3397728.0 i/s - 3.19x  (± 0.00) slower
        String#match:  2978664.7 i/s - 3.64x  (± 0.00) slower
```

```sh
$ ruby -v code/string/===-vs-=~-vs-match.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
       String#match?   930.555k i/100ms
           String#=~   355.918k i/100ms
          Regexp#===   335.243k i/100ms
        String#match   279.062k i/100ms
Calculating -------------------------------------
       String#match?      9.423M (± 2.9%) i/s -     47.458M in   5.040965s
           String#=~      3.551M (± 3.4%) i/s -     17.796M in   5.017729s
          Regexp#===      3.422M (± 2.0%) i/s -     17.433M in   5.096387s
        String#match      2.825M (± 3.1%) i/s -     14.232M in   5.042716s

Comparison:
       String#match?:  9422553.9 i/s
           String#=~:  3550996.5 i/s - 2.65x  (± 0.00) slower
          Regexp#===:  3421965.0 i/s - 2.75x  (± 0.00) slower
        String#match:  2825186.2 i/s - 3.34x  (± 0.00) slower
```

```sh
$ ruby -v code/string/===-vs-=~-vs-match.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
       String#match?   992.807k i/100ms
           String#=~   347.285k i/100ms
          Regexp#===   330.830k i/100ms
        String#match   296.952k i/100ms
Calculating -------------------------------------
       String#match?      9.950M (± 2.4%) i/s -     50.633M in   5.091621s
           String#=~      3.618M (± 3.5%) i/s -     18.406M in   5.093708s
          Regexp#===      3.365M (± 8.9%) i/s -     16.872M in   5.077195s
        String#match      3.053M (± 2.4%) i/s -     15.442M in   5.060572s

Comparison:
       String#match?:  9950193.1 i/s
           String#=~:  3618000.9 i/s - 2.75x  (± 0.00) slower
          Regexp#===:  3365099.3 i/s - 2.96x  (± 0.00) slower
        String#match:  3053134.2 i/s - 3.26x  (± 0.00) slower
```

See [#59](https://github.com/JuanitoFatas/fast-ruby/pull/59) and [#62](https://github.com/JuanitoFatas/fast-ruby/pull/62) for discussions.


##### `String#gsub` vs `String#sub` vs `String#[]=` [code](code/string/gsub-vs-sub.rb)

```sh
$ ruby -v --jit code/string/gsub-vs-sub.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
         String#gsub    72.193k i/100ms
          String#sub    93.172k i/100ms
String#dup["string"]=
                       144.053k i/100ms
Calculating -------------------------------------
         String#gsub    676.325k (±14.6%) i/s -      3.321M in   5.055755s
          String#sub    935.665k (± 9.4%) i/s -      4.659M in   5.038975s
String#dup["string"]=
                          1.412M (± 4.0%) i/s -      7.203M in   5.108679s

Comparison:
String#dup["string"]=:  1412243.1 i/s
          String#sub:   935665.0 i/s - 1.51x  (± 0.00) slower
         String#gsub:   676325.3 i/s - 2.09x  (± 0.00) slower
```

```sh
$ ruby -v code/string/gsub-vs-sub.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
         String#gsub    77.854k i/100ms
          String#sub   100.192k i/100ms
String#dup["string"]=
                       149.517k i/100ms
Calculating -------------------------------------
         String#gsub    769.171k (± 3.6%) i/s -      3.893M in   5.068084s
          String#sub    982.309k (± 2.1%) i/s -      4.909M in   5.000045s
String#dup["string"]=
                          1.510M (± 2.6%) i/s -      7.625M in   5.054634s

Comparison:
String#dup["string"]=:  1509626.0 i/s
          String#sub:   982309.5 i/s - 1.54x  (± 0.00) slower
         String#gsub:   769170.9 i/s - 1.96x  (± 0.00) slower
```

```sh
$ ruby -v code/string/gsub-vs-sub.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
         String#gsub    73.131k i/100ms
          String#sub    82.575k i/100ms
String#dup["string"]=
                       156.996k i/100ms
Calculating -------------------------------------
         String#gsub    725.106k (± 7.0%) i/s -      3.657M in   5.068252s
          String#sub    971.977k (± 3.2%) i/s -      4.872M in   5.017779s
String#dup["string"]=
                          1.649M (± 3.3%) i/s -      8.321M in   5.051540s

Comparison:
String#dup["string"]=:  1649002.4 i/s
          String#sub:   971977.0 i/s - 1.70x  (± 0.00) slower
         String#gsub:   725106.0 i/s - 2.27x  (± 0.00) slower
```

##### `String#gsub` vs `String#tr` [code](code/string/gsub-vs-tr.rb)

> [rails/rails#17257](https://github.com/rails/rails/pull/17257)

```sh
$ ruby -v --jit code/string/gsub-vs-tr.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
         String#gsub    73.573k i/100ms
           String#tr   366.071k i/100ms
Calculating -------------------------------------
         String#gsub    772.596k (± 4.8%) i/s -      3.899M in   5.059238s
           String#tr      4.133M (± 6.5%) i/s -     20.866M in   5.078037s

Comparison:
           String#tr:  4133419.5 i/s
         String#gsub:   772596.4 i/s - 5.35x  (± 0.00) slower
```

```sh
$ ruby -v code/string/gsub-vs-tr.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
         String#gsub    74.211k i/100ms
           String#tr   393.525k i/100ms
Calculating -------------------------------------
         String#gsub    784.322k (± 3.6%) i/s -      3.933M in   5.021265s
           String#tr      3.980M (± 2.2%) i/s -     20.070M in   5.044494s

Comparison:
           String#tr:  3980455.4 i/s
         String#gsub:   784322.2 i/s - 5.08x  (± 0.00) slower
```

```sh
$ ruby -v code/string/gsub-vs-tr.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
         String#gsub    81.461k i/100ms
           String#tr   413.577k i/100ms
Calculating -------------------------------------
         String#gsub    784.234k (± 3.3%) i/s -      3.992M in   5.095454s
           String#tr      4.107M (± 2.9%) i/s -     20.679M in   5.039386s

Comparison:
           String#tr:  4107046.9 i/s
         String#gsub:   784234.1 i/s - 5.24x  (± 0.00) slower
```

##### `Mutable` vs `Immutable` [code](code/string/mutable_vs_immutable_strings.rb)

```sh
$ ruby -v --jit code/string/mutable_vs_immutable_strings.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
      Without Freeze     1.354M i/100ms
         With Freeze     2.336M i/100ms
Calculating -------------------------------------
      Without Freeze     14.685M (± 2.8%) i/s -     74.471M in   5.075135s
         With Freeze     23.973M (± 2.0%) i/s -    121.491M in   5.069938s

Comparison:
         With Freeze: 23972864.8 i/s
      Without Freeze: 14685381.8 i/s - 1.63x  (± 0.00) slower
```

```sh
$ ruby -v code/string/mutable_vs_immutable_strings.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
      Without Freeze     1.234M i/100ms
         With Freeze     1.927M i/100ms
Calculating -------------------------------------
      Without Freeze     12.221M (± 2.9%) i/s -     61.720M in   5.054822s
         With Freeze     19.540M (± 1.8%) i/s -     98.277M in   5.031082s

Comparison:
         With Freeze: 19540449.9 i/s
      Without Freeze: 12220743.5 i/s - 1.60x  (± 0.00) slower
```

```sh
$ ruby -v code/string/mutable_vs_immutable_strings.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
      Without Freeze     1.347M i/100ms
         With Freeze     2.053M i/100ms
Calculating -------------------------------------
      Without Freeze     13.583M (± 2.1%) i/s -     68.697M in   5.059833s
         With Freeze     19.760M (±10.3%) i/s -     98.562M in   5.072471s

Comparison:
         With Freeze: 19759816.3 i/s
      Without Freeze: 13583154.3 i/s - 1.45x  (± 0.00) slower
```


##### `String#sub!` vs `String#gsub!` vs `String#[]=` [code](code/string/sub!-vs-gsub!-vs-[]=.rb)

Note that `String#[]` will throw an `IndexError` when given string or regexp not matched.

```sh
$ ruby -v --jit code/string/sub\!-vs-gsub\!-vs-\[\]\=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
  String#['string']=   155.587k i/100ms
 String#sub!'string'    88.547k i/100ms
String#gsub!'string'    55.095k i/100ms
  String#[/regexp/]=    93.515k i/100ms
 String#sub!/regexp/    74.343k i/100ms
String#gsub!/regexp/    39.564k i/100ms
Calculating -------------------------------------
  String#['string']=      1.622M (± 2.3%) i/s -      8.246M in   5.087158s
 String#sub!'string'    928.725k (± 3.4%) i/s -      4.693M in   5.059444s
String#gsub!'string'    579.396k (± 2.5%) i/s -      2.920M in   5.043078s
  String#[/regexp/]=      1.036M (± 3.5%) i/s -      5.237M in   5.061972s
 String#sub!/regexp/    810.389k (± 2.4%) i/s -      4.089M in   5.048584s
String#gsub!/regexp/    419.004k (± 3.0%) i/s -      2.097M in   5.009305s

Comparison:
  String#['string']=:  1621793.6 i/s
  String#[/regexp/]=:  1035818.8 i/s - 1.57x  (± 0.00) slower
 String#sub!'string':   928725.2 i/s - 1.75x  (± 0.00) slower
 String#sub!/regexp/:   810389.5 i/s - 2.00x  (± 0.00) slower
String#gsub!'string':   579396.2 i/s - 2.80x  (± 0.00) slower
String#gsub!/regexp/:   419003.8 i/s - 3.87x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub\!-vs-gsub\!-vs-\[\]\=.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
  String#['string']=   157.777k i/100ms
 String#sub!'string'    93.140k i/100ms
String#gsub!'string'    57.170k i/100ms
  String#[/regexp/]=    90.788k i/100ms
 String#sub!/regexp/    75.282k i/100ms
String#gsub!/regexp/    39.272k i/100ms
Calculating -------------------------------------
  String#['string']=      1.651M (± 1.8%) i/s -      8.362M in   5.067412s
 String#sub!'string'    906.303k (± 2.5%) i/s -      4.564M in   5.039044s
String#gsub!'string'    579.907k (± 2.2%) i/s -      2.916M in   5.030400s
  String#[/regexp/]=    939.903k (± 2.1%) i/s -      4.721M in   5.025033s
 String#sub!/regexp/    755.217k (± 2.0%) i/s -      3.839M in   5.085878s
String#gsub!/regexp/    394.308k (± 1.8%) i/s -      2.003M in   5.081155s

Comparison:
  String#['string']=:  1650761.3 i/s
  String#[/regexp/]=:   939903.0 i/s - 1.76x  (± 0.00) slower
 String#sub!'string':   906303.3 i/s - 1.82x  (± 0.00) slower
 String#sub!/regexp/:   755217.1 i/s - 2.19x  (± 0.00) slower
String#gsub!'string':   579907.2 i/s - 2.85x  (± 0.00) slower
String#gsub!/regexp/:   394307.7 i/s - 4.19x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub\!-vs-gsub\!-vs-\[\]\=.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
  String#['string']=   182.703k i/100ms
 String#sub!'string'   100.499k i/100ms
String#gsub!'string'    59.191k i/100ms
  String#[/regexp/]=   105.282k i/100ms
 String#sub!/regexp/    81.647k i/100ms
String#gsub!/regexp/    43.487k i/100ms
Calculating -------------------------------------
  String#['string']=      1.794M (± 3.1%) i/s -      9.135M in   5.096329s
 String#sub!'string'    989.686k (± 3.4%) i/s -      5.025M in   5.083456s
String#gsub!'string'    594.950k (± 4.5%) i/s -      3.019M in   5.085620s
  String#[/regexp/]=      1.047M (± 3.5%) i/s -      5.264M in   5.034630s
 String#sub!/regexp/    845.412k (± 2.8%) i/s -      4.246M in   5.025997s
String#gsub!/regexp/    434.939k (± 2.5%) i/s -      2.218M in   5.102404s

Comparison:
  String#['string']=:  1794287.9 i/s
  String#[/regexp/]=:  1046919.8 i/s - 1.71x  (± 0.00) slower
 String#sub!'string':   989686.2 i/s - 1.81x  (± 0.00) slower
 String#sub!/regexp/:   845411.5 i/s - 2.12x  (± 0.00) slower
String#gsub!'string':   594950.3 i/s - 3.02x  (± 0.00) slower
String#gsub!/regexp/:   434939.0 i/s - 4.13x  (± 0.00) slower
```

##### `String#sub` vs `String#delete_prefix` [code](code/string/sub-vs-delete_prefix.rb)

[Ruby 2.5 introduced](https://bugs.ruby-lang.org/issues/12694) `String#delete_prefix`.
Note that this can only be used for removing characters from the start of a string.

```sh
$ ruby -v --jit code/string/sub-vs-delete_prefix.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
String#delete_prefix   600.748k i/100ms
          String#sub   103.351k i/100ms
Calculating -------------------------------------
String#delete_prefix      6.857M (± 4.5%) i/s -     34.243M in   5.004566s
          String#sub      1.078M (± 3.1%) i/s -      5.478M in   5.085222s

Comparison:
String#delete_prefix:  6856606.6 i/s
          String#sub:  1078236.3 i/s - 6.36x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub-vs-delete_prefix.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
String#delete_prefix   649.381k i/100ms
          String#sub   104.200k i/100ms
Calculating -------------------------------------
String#delete_prefix      6.422M (± 1.9%) i/s -     32.469M in   5.057367s
          String#sub      1.030M (± 1.9%) i/s -      5.210M in   5.061797s

Comparison:
String#delete_prefix:  6422441.4 i/s
          String#sub:  1029661.2 i/s - 6.24x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub-vs-delete_prefix.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
String#delete_prefix   632.384k i/100ms
          String#sub   101.553k i/100ms
Calculating -------------------------------------
String#delete_prefix      6.533M (± 3.4%) i/s -     32.884M in   5.039351s
          String#sub      1.051M (± 1.3%) i/s -      5.281M in   5.025364s

Comparison:
String#delete_prefix:  6533452.7 i/s
          String#sub:  1050999.0 i/s - 6.22x  (± 0.00) slower
```

##### `String#sub` vs `String#chomp` vs `String#delete_suffix` [code](code/string/sub-vs-chomp-vs-delete_suffix.rb)

[Ruby 2.5 introduced](https://bugs.ruby-lang.org/issues/13665) `String#delete_suffix`
as a counterpart to `delete_prefix`. The performance gain over `chomp` is
small and during some runs the difference falls within the error margin.
Note that this can only be used for removing characters from the end of a string.

```sh
$ ruby -v --jit code/string/sub-vs-chomp-vs-delete_suffix.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
          String#sub    90.690k i/100ms
        String#chomp   559.900k i/100ms
String#delete_suffix   590.850k i/100ms
Calculating -------------------------------------
          String#sub      1.027M (± 4.3%) i/s -      5.169M in   5.044656s
        String#chomp      6.293M (± 3.4%) i/s -     31.914M in   5.077268s
String#delete_suffix      6.920M (± 3.1%) i/s -     34.860M in   5.042218s

Comparison:
String#delete_suffix:  6920473.1 i/s
        String#chomp:  6293365.7 i/s - 1.10x  (± 0.00) slower
          String#sub:  1026680.9 i/s - 6.74x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub-vs-chomp-vs-delete_suffix.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
          String#sub    98.381k i/100ms
        String#chomp   580.462k i/100ms
String#delete_suffix   639.385k i/100ms
Calculating -------------------------------------
          String#sub      1.031M (± 1.3%) i/s -      5.214M in   5.056296s
        String#chomp      5.709M (± 3.0%) i/s -     29.023M in   5.088369s
String#delete_suffix      6.357M (± 1.4%) i/s -     31.969M in   5.030161s

Comparison:
String#delete_suffix:  6356758.4 i/s
        String#chomp:  5709074.7 i/s - 1.11x  (± 0.00) slower
          String#sub:  1031396.0 i/s - 6.16x  (± 0.00) slower
```

```sh
$ ruby -v code/string/sub-vs-chomp-vs-delete_suffix.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
          String#sub   103.116k i/100ms
        String#chomp   610.075k i/100ms
String#delete_suffix   635.100k i/100ms
Calculating -------------------------------------
          String#sub      1.045M (± 2.4%) i/s -      5.259M in   5.036039s
        String#chomp      6.091M (± 3.1%) i/s -     30.504M in   5.013253s
String#delete_suffix      6.411M (± 2.5%) i/s -     32.390M in   5.055271s

Comparison:
String#delete_suffix:  6411497.7 i/s
        String#chomp:  6090657.0 i/s - same-ish: difference falls within error
          String#sub:  1044872.5 i/s - 6.14x  (± 0.00) slower
```

##### `String#unpack1` vs `String#unpack[0]` [code](code/string/unpack1-vs-unpack[0].rb)

[Ruby 2.4.0 introduced `unpack1`](https://bugs.ruby-lang.org/issues/12752) to skip creating the intermediate array object.

```sh
$ ruby -v --jit code/string/unpack1-vs-unpack\[0\].rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
      String#unpack1   673.160k i/100ms
    String#unpack[0]   526.137k i/100ms
Calculating -------------------------------------
      String#unpack1      7.918M (± 3.5%) i/s -     39.716M in   5.022316s
    String#unpack[0]      6.086M (± 1.9%) i/s -     30.516M in   5.016072s

Comparison:
      String#unpack1:  7918086.2 i/s
    String#unpack[0]:  6085817.1 i/s - 1.30x  (± 0.00) slower
```

```sh
$ ruby -v code/string/unpack1-vs-unpack\[0\].rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
      String#unpack1   707.574k i/100ms
    String#unpack[0]   549.608k i/100ms
Calculating -------------------------------------
      String#unpack1      7.055M (± 4.3%) i/s -     35.379M in   5.025016s
    String#unpack[0]      5.578M (± 1.2%) i/s -     28.030M in   5.025493s

Comparison:
      String#unpack1:  7055151.1 i/s
    String#unpack[0]:  5578334.9 i/s - 1.26x  (± 0.00) slower
```

```sh
$ ruby -v code/string/unpack1-vs-unpack\[0\].rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
      String#unpack1   702.332k i/100ms
    String#unpack[0]   553.609k i/100ms
Calculating -------------------------------------
      String#unpack1      7.191M (± 3.8%) i/s -     36.521M in   5.086719s
    String#unpack[0]      5.640M (± 3.0%) i/s -     28.234M in   5.010341s

Comparison:
      String#unpack1:  7190690.2 i/s
    String#unpack[0]:  5640461.2 i/s - 1.27x  (± 0.00) slower
```

##### Remove extra spaces (or other contiguous characters) [code](code/string/remove-extra-spaces-or-other-chars.rb)

The code is tested against contiguous spaces but should work for other chars too.

```sh
$ ruby -v --jit code/string/remove-extra-spaces-or-other-chars.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
 String#gsub/regex+/     5.228k i/100ms
      String#squeeze   127.846k i/100ms
Calculating -------------------------------------
 String#gsub/regex+/     52.237k (± 4.8%) i/s -    261.400k in   5.016504s
      String#squeeze      1.338M (± 3.9%) i/s -      6.776M in   5.073108s

Comparison:
      String#squeeze:  1337774.4 i/s
 String#gsub/regex+/:    52237.4 i/s - 25.61x  (± 0.00) slower
```

```sh
$ ruby -v code/string/remove-extra-spaces-or-other-chars.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
 String#gsub/regex+/     5.526k i/100ms
      String#squeeze   132.933k i/100ms
Calculating -------------------------------------
 String#gsub/regex+/     54.487k (± 3.4%) i/s -    276.300k in   5.077096s
      String#squeeze      1.327M (± 2.5%) i/s -      6.647M in   5.011834s

Comparison:
      String#squeeze:  1327043.9 i/s
 String#gsub/regex+/:    54487.0 i/s - 24.36x  (± 0.00) slower
```

```sh
$ ruby -v code/string/remove-extra-spaces-or-other-chars.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
 String#gsub/regex+/     6.081k i/100ms
      String#squeeze   119.376k i/100ms
Calculating -------------------------------------
 String#gsub/regex+/     58.717k (± 3.0%) i/s -    297.969k in   5.079559s
      String#squeeze      1.159M (± 2.8%) i/s -      5.849M in   5.049158s

Comparison:
      String#squeeze:  1159430.1 i/s
 String#gsub/regex+/:    58716.9 i/s - 19.75x  (± 0.00) slower
```

### Time

##### `Time.iso8601` vs `Time.parse` [code](code/time/iso8601-vs-parse.rb)

When expecting well-formatted data from e.g. an API, `iso8601` is faster and will raise an `ArgumentError` on malformed input.

```sh
$ ruby -v --jit code/time/iso8601-vs-parse.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
        Time.iso8601    23.558k i/100ms
          Time.parse     6.957k i/100ms
Calculating -------------------------------------
        Time.iso8601    242.468k (± 3.4%) i/s -      1.225M in   5.058188s
          Time.parse     74.030k (± 2.3%) i/s -    375.678k in   5.077476s

Comparison:
        Time.iso8601:   242468.4 i/s
          Time.parse:    74029.7 i/s - 3.28x  (± 0.00) slower
```

```sh
$ ruby -v code/time/iso8601-vs-parse.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
        Time.iso8601    20.646k i/100ms
          Time.parse     6.342k i/100ms
Calculating -------------------------------------
        Time.iso8601    238.509k (± 2.3%) i/s -      1.197M in   5.023439s
          Time.parse     73.663k (± 2.6%) i/s -    374.178k in   5.083005s

Comparison:
        Time.iso8601:   238508.6 i/s
          Time.parse:    73663.4 i/s - 3.24x  (± 0.00) slower
```

```sh
$ ruby -v code/time/iso8601-vs-parse.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
        Time.iso8601    23.257k i/100ms
          Time.parse     7.630k i/100ms
Calculating -------------------------------------
        Time.iso8601    230.468k (± 3.5%) i/s -      1.163M in   5.051940s
          Time.parse     75.038k (± 3.2%) i/s -    381.500k in   5.089453s

Comparison:
        Time.iso8601:   230468.3 i/s
          Time.parse:    75037.9 i/s - 3.07x  (± 0.00) slower
```

### Range

##### `cover?` vs `include?` [code](code/range/cover-vs-include.rb)

`cover?` only check if it is within the start and end, `include?` needs to traverse the whole range.

```sh
$ ruby -v --jit code/range/cover-vs-include.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) +JIT [x86_64-darwin20]
Warming up --------------------------------------
        range#cover?   301.689k i/100ms
      range#include?    10.120k i/100ms
       range#member?    10.297k i/100ms
       plain compare   558.591k i/100ms
Calculating -------------------------------------
        range#cover?      3.119M (± 3.1%) i/s -     15.688M in   5.034271s
      range#include?    103.832k (± 2.1%) i/s -    526.240k in   5.070534s
       range#member?    104.091k (± 3.9%) i/s -    525.147k in   5.053924s
       plain compare      5.911M (± 2.1%) i/s -     29.605M in   5.010538s

Comparison:
       plain compare:  5911316.2 i/s
        range#cover?:  3119206.3 i/s - 1.90x  (± 0.00) slower
       range#member?:   104091.1 i/s - 56.79x  (± 0.00) slower
      range#include?:   103831.8 i/s - 56.93x  (± 0.00) slower
```

```sh
$ ruby -v code/range/cover-vs-include.rb
ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [x86_64-darwin20]
Warming up --------------------------------------
        range#cover?   307.025k i/100ms
      range#include?    10.185k i/100ms
       range#member?    10.338k i/100ms
       plain compare   453.522k i/100ms
Calculating -------------------------------------
        range#cover?      3.023M (± 3.5%) i/s -     15.351M in   5.084943s
      range#include?    104.974k (± 1.9%) i/s -    529.620k in   5.047043s
       range#member?    104.205k (± 2.0%) i/s -    527.238k in   5.061640s
       plain compare      4.733M (± 2.3%) i/s -     24.037M in   5.081713s

Comparison:
       plain compare:  4732591.9 i/s
        range#cover?:  3022984.0 i/s - 1.57x  (± 0.00) slower
      range#include?:   104974.0 i/s - 45.08x  (± 0.00) slower
       range#member?:   104205.5 i/s - 45.42x  (± 0.00) slower
```

```sh
$ ruby -v code/range/cover-vs-include.rb
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
Warming up --------------------------------------
        range#cover?   333.470k i/100ms
      range#include?    11.548k i/100ms
       range#member?    11.235k i/100ms
       plain compare   506.372k i/100ms
Calculating -------------------------------------
        range#cover?      3.654M (± 3.4%) i/s -     18.341M in   5.025381s
      range#include?    111.882k (± 3.2%) i/s -    565.852k in   5.062837s
       range#member?    108.565k (± 4.9%) i/s -    550.515k in   5.083447s
       plain compare      5.162M (± 3.5%) i/s -     25.825M in   5.008834s

Comparison:
       plain compare:  5162425.3 i/s
        range#cover?:  3653964.9 i/s - 1.41x  (± 0.00) slower
      range#include?:   111881.7 i/s - 46.14x  (± 0.00) slower
       range#member?:   108565.2 i/s - 47.55x  (± 0.00) slower
```


## Less idiomatic but with significant performance ruby

Checkout: https://github.com/JuanitoFatas/fast-ruby/wiki/Less-idiomatic-but-with-significant-performance-difference


## Submit New Entry

Please! [Edit this README.md](https://github.com/JuanitoFatas/fast-ruby/edit/master/README.md) then [Submit a Awesome Pull Request](https://github.com/JuanitoFatas/fast-ruby/pulls)!


## Something went wrong

Code example is wrong? :cry: Got better example? :heart_eyes: Excellent!

[Please open an issue](https://github.com/JuanitoFatas/fast-ruby/issues/new) or [Open a Pull Request](https://github.com/JuanitoFatas/fast-ruby/pulls) to fix it.

Thank you in advance! :wink: :beer:


## One more thing

[Share this with your #Rubyfriends! <3](https://twitter.com/intent/tweet?url=http%3A%2F%2Fgit.io%2F4U3xdw&text=Fast%20Ruby%20--%20Common%20Ruby%20Idioms%20inspired%20by%20%40sferik&original_referer=&via=juanitofatas&hashtags=#RubyFriends)

Brought to you by [@JuanitoFatas](https://twitter.com/juanitofatas)

Feel free to talk with me on Twitter! <3


## Also Checkout

- [Derailed Benchmarks](https://github.com/schneems/derailed_benchmarks)

  Go faster, off the Rails - Benchmarks for your whole Rails app

- [Benchmarking Ruby](https://speakerdeck.com/davystevenson/benchmarking-ruby)

  Talk by Davy Stevenson @ RubyConf 2014.

- [davy/benchmark-bigo](https://github.com/davy/benchmark-bigo)

  Provides Big O notation benchmarking for Ruby.

- [The Ruby Challenge](https://therubychallenge.com/)

  Talk by Prem Sichanugrist @ Ruby Kaigi 2014.

- [Fasterer](https://github.com/DamirSvrtan/fasterer)

  Make your Rubies go faster with this command line tool.


## License

![CC-BY-SA](CC-BY-SA.png)

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).


## Code License

### CC0 1.0 Universal

To the extent possible under law, @JuanitoFatas has waived all copyright and related or neighboring rights to "fast-ruby".

This work belongs to the community.
