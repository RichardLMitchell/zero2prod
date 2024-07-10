Author: Richard L Mitchell

Credit: These notes and this project is based on Luca Palmieri, Zero to Production in Rust, 2022. All credit should go to Luca Palmieri: https://github.com/LukeMathWalker/zero-to-production

This project will focus on writing cloud-native applications in rust, particularly on the challenges facing a team of 4-5 engineers of varying levels of experience and capabilities. The project will build an email newletter.


### 1. Setting Up the Development Environment

local machine: MacBook pro, 2.3 GHz Quad-Core Intel Core i7, 16 GB RAM. IDE used:    
VSCode Version: 1.9.0 (Universal)
Local Dev Environment Setup Procedure

    Install C compiler:

`$ sudo apt install build-essential`

I could not install build-essential on my local environment due to an ongoing issue with Java JRE and apt. I have the clang compiler installed as defult on mac (Apple clang version 13.0.0 (clang-1300.0.29.30)) so I'll see how I get on with this for now.


- Check compiler version:
`$ rustc --version`
rustc 1.74.1 (a28077b28 2023-12-04)

- Install Rust system (default install):    
`$ curl --proto '=https' --tlsv1.2 -sSf <https://sh.rustup.rs> | sh`    


- Check rust version:
`$ rustup --version`  
rustup 1.26.0 (5af9b9484 2023-04-05)    

- Create a repo in GitHub called 'zero2prod'   

- Open GitHub repo project folder in VSCode

Project folder: /zero2prod

- Save project files in local folder 'zero2prod'

- Create the initial rust project files (incl Cargo.toml, etc.):   
`$ cargo new zero2prod`

- Create initial .gitignore file to exclude .vscode, cargo.lock, target files, etc.:

#Ignore .vscode folder     
.vscode/    

#Ignore compiled binaries     
target/    

#Ignore Cargo.lock file    
Cargo.lock    

- Initialise the git repo inside the sim project folder:
`$ git init`

- Add, commit and push files to the new origin main GitHub repo:

$ git status    
$ git add .     
$ git commit -m"initial commit"     
$ git remote add origin https://github.com/RichardLMitchell/zero2prod.git    
git push origin main

- Check GitHub repo is populated with inital commit and zero2prod project files, and excludes .gitignore files:

https://github.com/RichardLMitchell/zero2prod.git

- Add README.md, dev_notes.md


### Inner Development Loop:

 1. Make a change
 2. Compile the application
 3. Run tests
 4. Run the application

The spped of the inner development loop - the number of development iterations that can be completed within a unit of time e.g. in 1 minute. With Rust this can be greatly affected by compilation time, particularly on larger projects:

- Install LLVM lld linker which is faster than the standard linker - The purpose is to speed up compilation, reducing the compilation time of each code iteration:

`$ brew install llvm`  (using Homebrew to install llvm on my mac dev machine)

(to install on ubuntu linux: `$ sudo apt-get install lld clang`)

NOTE: LLVM Clang takes a long time to install/compile (about 3 hours on my 4 core, 2.3 Ghz macbook pro) and also requires memory - llvm+clang v10 on Linux builds 8 GB in bin, 13GB in lib and 5+ GB in tools, that's ~30 GB. 


- Install cargo-watch to reduce 'percieved' compilation time - cargo watch monitors the codebase for code changes then automatically kicks off compilation behind the scenes while you're reviewing your changes. Cargo-watch can also be chained to include cargo watch check, test, run - `$ cargo watch -x check -x test -x run`:

`$ cargo install cargo-watch`


### Continuos Integration

#### Tests:

- All tests can be run with: `$ cargo test`

which automatically builds the project before running the tests so there is no need to run `cargo build` before cargo run (note: `cargo build` caches dependencies before `cargo run`  or `cargo test`).


#### Code Coverage:
- We will be using 'cargo-tarpaulin' to measure code coverage, as a quick way of spotting if any parts of the codebase have been overlooked during testing:
`$ cargo install cargo-tarpaulin`

TODO: Codecover or Coveralls - We will check out these utlities which can be used to display code coverage metrics.

#### Linting:
The linter will try to spot unidiomatic code, overly complex constructs and common mistakes/inefficiencies. We will be using cargo clippy which is the official rust linter and included withing rustup.

- To install Clippy (if required): `$ rustup component add clippy`
- To run clippy: `$ cargo clippy`
- To fail the linter check if clippy emits any warnings: `$ cargo clippy -- -D warnings`

- To disable specific warnings, use the directive: $ #[allow(clippy::lint_name)] above the code block.

TODO: see the clippy README.md for more details.

#### Formatting:
We will use the official rust formatter 'rustfmt' to automatically format our code. rustfmt is already included in the rustup package (like clippy), but if we need to install it for any reason:

- To install rustfmt: `$ rustup component add rustfmt`
- To format the whole project: `$ cargo fmt`
- In the CI pipeline we will add: `$ cargo fmt -- --check`

TODO: Check the rustfmt README.md for details on tuning rustfmt.


#### Security Vulnerabilities:
We will use cargo-audit to check our rust packages for exploitable vulnerabilities. cargo-audit is a convenient sub-command that checks if any vulnerabilities have been reported to the Advisory Databes maintained by the Rust Secure COde working group.

- To install cargo-audit: `$ cargo install cargo-audit`
- To run: `$ cargo audit`

We will include cargo-audit into our CI pipeline to run on every audit and to run daily to check for secureity updates.

After setup was complete, the cargo.toml file included the following packages and dependencies info:

[package]
name = "zero2prod"
version = "0.1.0"
edition = "2021"

[dependencies]
llvm = "0.0.1"
cargo-watch = "8.5.2"
tarpaulin = "0.1.0"
clippy = "0.0.302"
fmt = "0.1.0"
cargo-audit = "0.20.0"


PROBLEMS:
Both LLVM and 'tarpc-plugins' raised issues with rust-analyser.


I used `$ cargo tree` to identify/confirm which crate was bringing in 'tarpc-plugins'. This is the part of the dependency tree related to LLVM and tarpaulin, where we can see tarpc-plugins is brought in by tarpaulin:

├── fmt v0.1.0
├── llvm v0.0.1
│   ├── libc v0.2.155
│   └── llvm-sys v181.1.0
│       └── libc v0.2.155
│       [build-dependencies]
│       ├── anyhow v1.0.86
│       ├── cc v1.1.0
│       ├── lazy_static v1.5.0
│       ├── regex-lite v0.1.6
│       └── semver v1.0.23
└── tarpaulin v0.1.0
    ├── futures v0.1.31
    ├── tarpc v0.9.0
    │   ├── bincode v0.8.0
    │   │   ├── byteorder v1.5.0
    │   │   ├── num-traits v0.1.43
    │   │   │   └── num-traits v0.2.19 (*)
    │   │   └── serde v1.0.204 (*)
    │   ├── byteorder v1.5.0
    │   ├── bytes v0.4.12
    │   │   ├── byteorder v1.5.0
    │   │   └── iovec v0.1.4
    │   │       └── libc v0.2.155
    │   ├── cfg-if v0.1.10
    │   ├── futures v0.1.31
    │   ├── lazy_static v0.2.11
    │   ├── log v0.3.9
    │   │   └── log v0.4.22
    │   ├── net2 v0.2.39
    │   │   ├── cfg-if v0.1.10
    │   │   └── libc v0.2.155
    │   ├── num_cpus v1.16.0 (*)
    │   ├── serde v1.0.204 (*)
    │   ├── serde_derive v1.0.204 (proc-macro) (*)
    │   ├── tarpc-plugins v0.2.0
    │   │   └── itertools v0.6.5
    │   │       └── either v1.13.0
    │   ├── thread-pool v0.1.1
    │   │   ├── num_cpus v1.16.0 (*)
    │   │   └── two-lock-queue v0.1.1
    │   ├── tokio-core v0.1.18
    │   │   ├── bytes v0.4.12 (*)
    │   │   ├── futures v0.1.31
    │   │   ├── iovec v0.1.4 (*)
    │   │   ├── log v0.4.22
    │   │   ├── mio v0.6.23
    │   │   │   ├── cfg-if v0.1.10
    │   │   │   ├── iovec v0.1.4 (*)
    │   │   │   ├── libc v0.2.155
    │   │   │   ├── log v0.4.22
    │   │   │   ├── net2 v0.2.39 (*)
    │   │   │   └── slab v0.4.9 (*)
    │   │   ├── scoped-tls v0.1.2
    │   │   ├── tokio v0.1.22
    │   │   │   ├── bytes v0.4.12 (*)
    │   │   │   ├── futures v0.1.31
    │   │   │   ├── mio v0.6.23 (*)
    │   │   │   ├── num_cpus v1.16.0 (*)
    │   │   │   ├── tokio-codec v0.1.2
    │   │   │   │   ├── bytes v0.4.12 (*)
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   └── tokio-io v0.1.13
    │   │   │   │       ├── bytes v0.4.12 (*)
    │   │   │   │       ├── futures v0.1.31
    │   │   │   │       └── log v0.4.22
    │   │   │   ├── tokio-current-thread v0.1.7
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   └── tokio-executor v0.1.10
    │   │   │   │       ├── crossbeam-utils v0.7.2
    │   │   │   │       │   ├── cfg-if v0.1.10
    │   │   │   │       │   └── lazy_static v1.5.0
    │   │   │   │       │   [build-dependencies]
    │   │   │   │       │   └── autocfg v1.3.0
    │   │   │   │       └── futures v0.1.31
    │   │   │   ├── tokio-executor v0.1.10 (*)
    │   │   │   ├── tokio-fs v0.1.7
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   ├── tokio-io v0.1.13 (*)
    │   │   │   │   └── tokio-threadpool v0.1.18
    │   │   │   │       ├── crossbeam-deque v0.7.4
    │   │   │   │       │   ├── crossbeam-epoch v0.8.2
    │   │   │   │       │   │   ├── cfg-if v0.1.10
    │   │   │   │       │   │   ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │       │   │   ├── lazy_static v1.5.0
    │   │   │   │       │   │   ├── maybe-uninit v2.0.0
    │   │   │   │       │   │   ├── memoffset v0.5.6
    │   │   │   │       │   │   │   [build-dependencies]
    │   │   │   │       │   │   │   └── autocfg v1.3.0
    │   │   │   │       │   │   └── scopeguard v1.2.0
    │   │   │   │       │   │   [build-dependencies]
    │   │   │   │       │   │   └── autocfg v1.3.0
    │   │   │   │       │   ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │       │   └── maybe-uninit v2.0.0
    │   │   │   │       ├── crossbeam-queue v0.2.3
    │   │   │   │       │   ├── cfg-if v0.1.10
    │   │   │   │       │   ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │       │   └── maybe-uninit v2.0.0
    │   │   │   │       ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │       ├── futures v0.1.31
    │   │   │   │       ├── lazy_static v1.5.0
    │   │   │   │       ├── log v0.4.22
    │   │   │   │       ├── num_cpus v1.16.0 (*)
    │   │   │   │       ├── slab v0.4.9 (*)
    │   │   │   │       └── tokio-executor v0.1.10 (*)
    │   │   │   ├── tokio-io v0.1.13 (*)
    │   │   │   ├── tokio-reactor v0.1.12
    │   │   │   │   ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   ├── lazy_static v1.5.0
    │   │   │   │   ├── log v0.4.22
    │   │   │   │   ├── mio v0.6.23 (*)
    │   │   │   │   ├── num_cpus v1.16.0 (*)
    │   │   │   │   ├── parking_lot v0.9.0
    │   │   │   │   │   ├── lock_api v0.3.4
    │   │   │   │   │   │   └── scopeguard v1.2.0
    │   │   │   │   │   └── parking_lot_core v0.6.3
    │   │   │   │   │       ├── cfg-if v0.1.10
    │   │   │   │   │       ├── libc v0.2.155
    │   │   │   │   │       └── smallvec v0.6.14
    │   │   │   │   │           └── maybe-uninit v2.0.0
    │   │   │   │   │       [build-dependencies]
    │   │   │   │   │       └── rustc_version v0.2.3
    │   │   │   │   │           └── semver v0.9.0
    │   │   │   │   │               └── semver-parser v0.7.0
    │   │   │   │   │   [build-dependencies]
    │   │   │   │   │   └── rustc_version v0.2.3 (*)
    │   │   │   │   ├── slab v0.4.9 (*)
    │   │   │   │   ├── tokio-executor v0.1.10 (*)
    │   │   │   │   ├── tokio-io v0.1.13 (*)
    │   │   │   │   └── tokio-sync v0.1.8
    │   │   │   │       ├── fnv v1.0.7
    │   │   │   │       └── futures v0.1.31
    │   │   │   ├── tokio-sync v0.1.8 (*)
    │   │   │   ├── tokio-tcp v0.1.4
    │   │   │   │   ├── bytes v0.4.12 (*)
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   ├── iovec v0.1.4 (*)
    │   │   │   │   ├── mio v0.6.23 (*)
    │   │   │   │   ├── tokio-io v0.1.13 (*)
    │   │   │   │   └── tokio-reactor v0.1.12 (*)
    │   │   │   ├── tokio-threadpool v0.1.18 (*)
    │   │   │   ├── tokio-timer v0.2.13
    │   │   │   │   ├── crossbeam-utils v0.7.2 (*)
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   ├── slab v0.4.9 (*)
    │   │   │   │   └── tokio-executor v0.1.10 (*)
    │   │   │   ├── tokio-udp v0.1.6
    │   │   │   │   ├── bytes v0.4.12 (*)
    │   │   │   │   ├── futures v0.1.31
    │   │   │   │   ├── log v0.4.22
    │   │   │   │   ├── mio v0.6.23 (*)
    │   │   │   │   ├── tokio-codec v0.1.2 (*)
    │   │   │   │   ├── tokio-io v0.1.13 (*)
    │   │   │   │   └── tokio-reactor v0.1.12 (*)
    │   │   │   └── tokio-uds v0.2.7
    │   │   │       ├── bytes v0.4.12 (*)
    │   │   │       ├── futures v0.1.31
    │   │   │       ├── iovec v0.1.4 (*)
    │   │   │       ├── libc v0.2.155
    │   │   │       ├── log v0.4.22
    │   │   │       ├── mio v0.6.23 (*)
    │   │   │       ├── mio-uds v0.6.8
    │   │   │       │   ├── iovec v0.1.4 (*)
    │   │   │       │   ├── libc v0.2.155
    │   │   │       │   └── mio v0.6.23 (*)
    │   │   │       ├── tokio-codec v0.1.2 (*)
    │   │   │       ├── tokio-io v0.1.13 (*)
    │   │   │       └── tokio-reactor v0.1.12 (*)
    │   │   ├── tokio-executor v0.1.10 (*)
    │   │   ├── tokio-io v0.1.13 (*)
    │   │   ├── tokio-reactor v0.1.12 (*)
    │   │   └── tokio-timer v0.2.13 (*)
    │   ├── tokio-io v0.1.13 (*)
    │   ├── tokio-proto v0.1.1
    │   │   ├── futures v0.1.31
    │   │   ├── log v0.3.9 (*)
    │   │   ├── net2 v0.2.39 (*)
    │   │   ├── rand v0.3.23
    │   │   │   ├── libc v0.2.155
    │   │   │   └── rand v0.4.6
    │   │   │       └── libc v0.2.155
    │   │   ├── slab v0.3.0
    │   │   ├── smallvec v0.2.1
    │   │   ├── take v0.1.0
    │   │   ├── tokio-core v0.1.18 (*)
    │   │   ├── tokio-io v0.1.13 (*)
    │   │   └── tokio-service v0.1.0
    │   │       └── futures v0.1.31
    │   └── tokio-service v0.1.0 (*)
    └── tarpc-plugins v0.2.0 (*)


    Removing both LLVM and Tarpaulin temporarily from inclusion in the cargo.toml file until I can implement the necessary fixes, clears all outstanding problems in rust-analyser:

    [dependencies]
# llvm = "0.0.1"
cargo-watch = "8.5.2"
# tarpaulin = "0.1.0"
clippy = "0.0.302"
fmt = "0.1.0"
cargo-audit = "0.20.0" 