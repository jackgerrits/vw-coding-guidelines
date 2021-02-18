_This page is primarily targeted at maintainers and contributors_ 

**Note:** These guidelines are currently being updated. (Feb 11, 2021)

The idea of these guidelines is to form a general consensus about the method used and help progress towards the general goal of the VW project to create a very fast, efficient, and capable learning algorithm

# Guidelines

1. All variables should be initialized. If explicitly allocating memory (malloc), ensure it has been zeroed.
2. Memory allocation is avoided by reusing allocated memory where possible. 
3. Floats are preferred over doubles.   Doubles are only used for accumulators.
4. Templates are used to eliminate duplicate code and in some places to remove branches from inner loops.
5. Pass by reference is the default, except for objects of pointer size. Use a const reference whenever possible.
6. All learning reductions are confined to a single file with a single entry point. 
7. Learning reductions transform an example from one problem type to another.  
    * A problem type is defined by (label, prediction, features)
8. Don't manually manage memory. Don't use new/delete, malloc/free. Use RAII and smart pointers whenever possible.
9. Function pointer interfaces are explicit.
10. All examples are handled by the same stack of reductions.  
11. Examples are passed by function call.  
12. It’s not working until:
    - There are no warnings
    - Valgrind says it’s clean
    - It is running in CI
13. Prefer to use fixed size types where possible. Example: `uint32_t`
14. Unions are not allowed
15. C style casts are not allowed. Use `reinterpret_cast`
16. No direct access to `std::cout`, `std::cerr`. Use the VW logging interface.
17. Use [future compat](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/explore/future_compat.h) whenever possible
18. Use `constexpr` whenever possible. Use [future compat](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/explore/future_compat.h) if it requires C++14 or above.
19. Code is not fast unless a benchmark proves it
20. Reductions should not keep a reference to the all object. They should keep a reference to only what they need.
21. Rule of zero/Rule of 5
22. Use scoped enums over C enums
23. Avoid global state
24. Undefined behavior is not permitted. Not even if it is faster. Correctness is more important than speed.
25. Use `nodiscard` whenever possible

## Exception Policy
1. If an error state can be handled locally, that should always be what we do.  
2. All memory allocation should go through `memory.h` with errors handled by exception (the default) or crash (conditional compile).  In the future, we may add the ability to specify a memory allocator.
3. No new exceptions in the VW slim codepath when that is incorporated.  
4. We avoid new exceptions in the example handling path and have a goal of removing any others that exist. 
5. Where error states are unavoidable (i.e. situations like arguments-don’t-make-sense) we’ll accept off-critical-path exceptions.  A proposal for refactoring around return codes is welcome, but that will need to be driven by Rajan and is subject to priorities.  At the library interface level, I believe this can be handled by having an explicit catch in the library interface for the setup calls.

# Style
1. There is a [`.clang-format file`](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/.clang-format), which outlines the general style
2. Class member variables should be prefixed with `_`. Example: `uint32_t _number_of_actions;`
3. All types should be nested in the `VW` namespace
    - This is a work in progress
    - Ultimately this namespace will become `vw`. `VW` -> `vw`
4. All blocks should be surrounded with braces. Single statement blocks are permitted to be on the same line. This can be easily fixed with tooling like so:
    - `clang-tidy -p build -fix -format-style=file -config="{Checks: 'readability-braces-around-statements'}" vowpalwabbit/<your_file>`
        - Passing `config` like this overrides the default checks specified in the [.clang-tidy](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/.clang-tidy) file
        - `-format-style=file` means it will also format the fixed code according to the [.clang-format](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/.clang-format) file

# Improvements

This is a list of improvements that we want to make to the code.  Any help implementing them is of course welcome.  

### Speed/IO optimizations
- Change the io_buf structure to run in it's own thread.  Currently, reading bits into program space operates synchronously with parsing which implies that delays in the return of read() delay parsing.  This should speedup all input forms (daemon, stdin, file)
- Change the text parser to work in a read-once fashion.  Currently, input strings are read multiple times.
- There should be a way to use the algorithms as a library.

### Algorithmic improvements
- Alternate learning algorithms.  We have basic matrix factorization, which needs to be developed further.  We also want to push into more complex nonconvex algorithms.
- Learning reductions.  Previously, we've used VW as a library to implement learning reductions against, but adding a layer of abstraction in the system allowing reductions to directly operate should be doable, and desirable.  Especially in a cluster parallel environment, directly supporting learning reductions appears superior to a library implementation.

