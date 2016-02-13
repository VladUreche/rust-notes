# Phases

To see the phases, run:
```
$ rustc -Z time-passes bug-21221.rs 
```

And here's a list (correlated with the methods in `src/librustc-driver/driver.rs`):

```
front-end (phase_1_parse_input):
 ... parsing

"early phases" (phase_2_configure_and_expand):
 ... configuration 1
 ... recursion limit
 ... gated macro checking
 ... crate injection
 ... macro loading
 ... plugin loading
 ... plugin registration
 ... expansion
 ... complete gated feature checking 1
 ... configuration 2
 ... gated configuration checking
 ... maybe building test harness
 ... prelude injection
 ... checking that all macro invocations are gone
 ... checking for inline asm in case the target doesn't support it
 ... complete gated feature checking 2
 ... assigning node ids

transformation passes in the compiler_input:
 ... lowering ast -> hir
 ... indexing hir
 ... attribute checking
 ... early lint checks

analysis (phase_3_run_analysis_passes):
 ... external crate/lib resolution
 ... language item collection
 ... resolution                      // <= name resolution
 ... lifetime resolution
 ... looking for entry point
 ... looking for plugin registrar
 ... region resolution
 ... loop checking
 ... static item recursion checking
 ... type collecting
 ... variance inference
 ... coherence checking
 ... wf checking (old)
 ... item-types checking
 ... item-bodies checking
 ... drop-impl checking
 ... wf checking (new)
 ... const checking
 ... privacy checking
 ... stability index
 ... intrinsic checking
 ... effect checking
 ... match checking
 ... MIR dump
 ... liveness checking
 ... borrow checking
 ... rvalue checking
 ... reachability checking
 ... death checking
 ... stability checking
 ... unused lib feature checking
 ... lint checking

llvm (phase_4_translate_to_llvm):
 ... resolving dependency formats
 ... erasing regions from MIR
 ... translation
   ...  llvm function passes [0]
   ...  llvm module passes [0]
   ...  codegen passes [0]
   ...  codegen passes [0]
 ... LLVM passes
   ...  running linker
 ... linking
```