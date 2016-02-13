# Locating Code Paths

A very useful thing when playing with a new code base is to see where some
piece of code is called from.

An easy way to do so is to simply `panic!("code executed")` and run `rustc`
with a program that triggers the code path you're interested in. If you run
the compiler with `RUST_STACKTRACE=1 path/to/rustc example.rs`, you get a
nice stacktrace:

```
note: the compiler unexpectedly panicked. this is a bug.
note: we would appreciate a bug report: https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md#bug-reports
thread 'rustc' panicked at 'code executed', src/librustc_resolve/lib.rs:254
stack backtrace:
   1:     0x7f9ed05e59a2 - sys::backtrace::tracing::imp::write::hd7cea40cc5964d04niu
   2:     0x7f9ed060c02d - panicking::default_handler::_<closure>::closure.43172
   3:     0x7f9ed060b5d1 - panicking::default_handler::hb5aafb5327649c140zy
   4:     0x7f9ed05dafbf - panicking::on_panic::he54ad5c56692f50amEy
   5:     0x7f9ed054c7bc - sys_common::unwind::begin_unwind_inner::h6a8b6c9d5868faf0Vat
   6:     0x7f9ece77a4c3 - sys_common::unwind::begin_unwind::begin_unwind::h10167807447269063379
   7:     0x7f9ece771e89 - resolve_struct_error::hf7ea3b9cca3f6cb3MOc
   8:     0x7f9ece7717b5 - resolve_error::h820d53e8909d2b75jOc
   9:     0x7f9ece787404 - Resolver<'a, 'tcx>::resolve_trait_reference::h1f9dc99c80bd1750mJf
  10:     0x7f9ece8067cc - Resolver<'a, 'tcx>::with_optional_trait_ref::h11045455466330716747
  11:     0x7f9ece8066f3 - Resolver<'a, 'tcx>::resolve_implementation::_<closure>::closure.21601
  12:     0x7f9ece8065e6 - Resolver<'a, 'tcx>::with_type_parameter_rib::h9035794364135212755
  13:     0x7f9ece7feed3 - Resolver<'a, 'tcx>::resolve_implementation::h0c9ddc118282b410pRf
  14:     0x7f9ece7812d1 - Resolver<'a, 'tcx>::resolve_item::h81cba000fe1e61c0tsf
  15:     0x7f9ece780d42 - Resolver<'a, 'tcx>.Visitor<'v>::visit_item::h7dd0c7389b82da91Iwd
  16:     0x7f9ece780c97 - Resolver<'a, 'tcx>.Visitor<'v>::visit_nested_item::hd68c8b71ff0e7a3eswd
  17:     0x7f9ece7d63d0 - intravisit::walk_mod::walk_mod::h2112072560249040666
  18:     0x7f9ece7d6345 - intravisit::Visitor::visit_mod::visit_mod::h6603468080221708737
  19:     0x7f9ece7d61d5 - intravisit::walk_crate::walk_crate::h6031851754720692804
  20:     0x7f9ece7d616d - Resolver<'a, 'tcx>::resolve_crate::hb192f92c33fa5f2ffqf
  21:     0x7f9ece84d044 - resolve_crate::he09fd6416fbe6ccbqCh
  22:     0x7f9ed0c614ae - driver::phase_3_run_analysis_passes::_<closure>::closure.26657
  23:     0x7f9ed0c60ccc - util::common::time::time::h6922971993312121033
  24:     0x7f9ed0c5c40f - driver::phase_3_run_analysis_passes::h1863490505123096256
  25:     0x7f9ed0bf6297 - driver::compile_input::h117471c27c253feeAca
  26:     0x7f9ed0bd2115 - run_compiler::h06c12b4c190c4112pHc
  27:     0x7f9ed0bced66 - run::_<closure>::closure.20795
  28:     0x7f9ed0bcd79d - monitor::_<closure>::closure.20687
  29:     0x7f9ed0bcd24b - std::thread::Builder::spawn::_<closure>::_<closure>::closure.20677
  30:     0x7f9ed0bcd1ed - sys_common::unwind::try::try_fn::try_fn::h6096144168197913149
  31:     0x7f9ed05da3f2 - __rust_try
  32:     0x7f9ed05da0fa - sys_common::unwind::inner_try::_<closure>::closure.42215
  33:     0x7f9ed05da007 - thread::local::LocalKey<T>::with::h6789703657737126610
  34:     0x7f9ed05b423a - sys_common::unwind::inner_try::h6a1eb0b25ab2886db8s
  35:     0x7f9ed0bcd14b - sys_common::unwind::try::try::h6823107831151711579
  36:     0x7f9ed0bccfd8 - std::thread::Builder::spawn::_<closure>::closure.20652
  37:     0x7f9ed0bcd9ad - boxed::F.FnBox<A>::call_box::call_box::h12527473249904799043
  38:     0x7f9ed05c9040 - boxed::Box<FnBox<A, Output $u3d$$u20$R$GT$$u2b$$u20$$u27$a$GT$.FnOnce$LT$A$GT$::call_once::call_once::h16832674533356144004
  39:     0x7f9ed05d7352 - sys_common::thread::start_thread::h5d91b5e82512fbe4rUs
  40:     0x7f9ed06079c7 - sys::thread::Thread::new::thread_start::__rust_abi
  41:     0x7f9ed060767d - sys::thread::Thread::new::thread_start::hbee3690f0593ce46Kwx
  42:     0x7f9ec7b4a181 - start_thread
  43:     0x7f9ed01cb47c - __clone
  44:                0x0 - <unknown>
```

Another way to achieve the same result is the equivalent of the JVM's
```
  new Exception().printStackTrace();
```
which, in rust, is:
```
    let mut err = std::sys::stdio::Stderr::new().ok().as_mut().expect("no stderr?");
    let _ = std::sys_common::backtrace::write(&mut *err);
```

This is much cleaner, as we need not `panic!` and crash if the code is executed,
but currently it relies on private access:

```
src/librustc_resolve/lib.rs:255:27: 255:55 error: method `new` is private
src/librustc_resolve/lib.rs:255             let mut err = std::sys::stdio::Stderr::new().ok().as_mut().expect("no stderr?");
                                                          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/librustc_resolve/lib.rs:256:21: 256:54 error: function `write` is private
src/librustc_resolve/lib.rs:256             let _ = std::sys_common::backtrace::write(&mut *err);
                                                    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

So, until there's a way to include this code in the Rust compiler, you can
`panic!` to get the stacktrace.