# Bootstrapping

The Rust compiler is written in Rust, so it compiles itself. While this is
the standard nowadays, this is tedious for hacking the compiler, since it means
that the time from making a change to seeing it in action is very long. 

The instructions on building it on your machine are here:
https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md

There are several compilation stages, going from 0 to 3. So when you make a 
change and rebuild, you rebuild stage0, which compiles stage1, which compiles
stage2, which has the decency not to go further. Yay!

So, can we make this quicker?

One way is to use the optimized
[makefile targets](https://github.com/rust-lang/rust/blob/master/Makefile.in),
such as `rustc-stage1`, that stop at a certain compilation stage. On my laptop,
`rust-stage1` takes about 10 minutes to compile everything. So it's still too
long for quick experiments.

Another approach would be to compile only the updated parts of the project 
(just the corresponding `.so` files). This could theoretically work if the 
ABI is stable and we don't modify signatures at the boundary between crates:
     * `make --dry-run` and copy just the commands before it goes into 
`--cfg stage1`, and place them in a bash script
     * set the configuration triple (e.g. `x86_64-unknown-linux-gnu`)
     * run the newly created script, which is equivalent to running the first
part of the `make rustc-stage1` command.

In my experiments this doesn't work, bailing out since names are mangled 
differently than before:
```
../x86_64-unknown-linux-gnu/stage1/bin/rustc: symbol lookup error: .../x86_64-unknown-linux-gnu/stage1/lib/librustc_driver-xxxxxx.so: undefined symbol: _ZN13resolve_crate20h1c2e6325611dd2cc6BhE
```

If we look at the symbol:
```
$ c++filt _ZN13resolve_crate20h1c2e6325611dd2cc6BhE
resolve_crate::h1c2e6325611dd2cc6Bh
```

I wonder where that symbol comes from, since I don't see any function that
should get mangled... Suggestions welcome :)

Any other ideas are welcome!