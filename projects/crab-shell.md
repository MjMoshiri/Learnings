# Crab Shell (Rust)

A small POSIX/Unix shell in Rust. Read a line, parse it, fork, exec, wait. Built mainly to get hands-on with Rust ownership in a domain where lifetimes and process I/O collide.

Rust's compiler turned a class of bugs I'd hit before into compile errors. Pipes and process plumbing also forced me to look at `std::process` and `nix` crates closely.
