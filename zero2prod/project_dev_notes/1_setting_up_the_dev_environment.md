Author: Richard L Mitchell

Credit: These notes and this project is based on Luca Palmieri, Zero to Production in Rust, 2022. All credit should go to Luca Palmieri: https://github.com/LukeMathWalker/zero-to-production

This project will focus on writing cloud-native applications in rust, particularly on the challenges facing a team of 4-5 engineers of varying levels of experience and capabilities. The project will build an email newletter.


# 1. Setting Up the Development Environment

local machine: MacBook pro, 2.3 GHz Quad-Core Intel Core i7, 16 GB RAM. IDE used:    
VSCode Version: 1.9.0 (Universal)
Local Dev Environment Setup Procedure

1. Install C compiler:

`$ sudo apt install build-essential`

I could not install build-essential on my local environment due to an ongoing issue with Java JRE and apt. I have the clang compiler installed as defult on mac (Apple clang version 13.0.0 (clang-1300.0.29.30)) so I'll see how I get on with this for now.


2. Check compiler version:
`$ rustc --version`
rustc 1.74.1 (a28077b28 2023-12-04)


3. Install Rust system (default install):    
`$ curl --proto '=https' --tlsv1.2 -sSf <https://sh.rustup.rs> | sh`    


4. Check rust version:
`$ rustup --version`  
rustup 1.26.0 (5af9b9484 2023-04-05)    


5. Create a repo in GitHub called 'zero2prod'   


6. Open GitHub repo project folder in VSCode

- Project folder: /zero2prod


7. Save project files in local folder 'zero2prod'


8. Create the initial rust project files (incl Cargo.toml, etc.):   

`$ cargo new zero2prod`


9. Create initial .gitignore file to exclude .vscode, cargo.lock, target files, etc.:  


#Ignore .vscode folder     
.vscode/    

#Ignore compiled binaries     
target/    

#Ignore Cargo.lock file    
Cargo.lock    



10. Initialise the git repo inside the sim project folder:

`$ git init`


11. Add, commit and push files to the new origin main GitHub repo:

```  
$ git status    
$ git add .     
$ git commit -m"initial commit"     
$ git remote add origin https://github.com/RichardLMitchell/zero2prod.git    
git push origin main   

```


12. Check GitHub repo is populated with inital commit and zero2prod project files, and excludes .gitignore files:

https://github.com/RichardLMitchell/zero2prod.git



13. Add README.md, dev_notes.md


## Inner Development Loop:

 - Make a change
 - Compile the application
 - Run tests
 - Run the application

The spped of the inner development loop - the number of development iterations that can be completed within a unit of time e.g. in 1 minute. With Rust this can be greatly affected by compilation time, particularly on larger projects:

1. Install LLVM lld linker which is faster than the standard linker - The purpose is to speed up compilation, reducing the compilation time of each code iteration:

`$ brew install llvm`  (using Homebrew to install llvm on my mac dev machine)

(to install on ubuntu linux: `$ sudo apt-get install lld clang`)

NOTE: LLVM Clang takes a long time to install/compile (about 3 hours on my 4 core, 2.3 Ghz macbook pro) and also requires memory - llvm+clang v10 on Linux builds 8 GB in bin, 13GB in lib and 5+ GB in tools, that's ~30 GB. 

PROBLEMS: llvm installed without issue but later produced issues (to be resolved) with rust-analyser.


2. Install cargo-watch to reduce 'percieved' compilation time - cargo watch monitors the codebase for code changes then automatically kicks off compilation behind the scenes while you're reviewing your changes. Cargo-watch can also be chained to include cargo watch check, test, run - `$ cargo watch -x check -x test -x run`:

`$ cargo install cargo-watch`


## Continuos Integration

### Tests:

1. All tests can be run with: `$ cargo test`

which automatically builds the project before running the tests so there is no need to run `cargo build` before cargo run (note: `cargo build` caches dependencies before `cargo run`  or `cargo test`).


### Code Coverage:

1. We will be using 'cargo-tarpaulin' to measure code coverage, as a quick way of spotting if any parts of the codebase have been overlooked during testing:
`$ cargo install cargo-tarpaulin`

TODO: Codecover or Coveralls - We will check out these utlities which can be used to display code coverage metrics.


### Linting:
The linter will try to spot unidiomatic code, overly complex constructs and common mistakes/inefficiencies. We will be using cargo clippy which is the official rust linter and included withing rustup.

1. To install Clippy (if required): `$ rustup component add clippy`
2. To run clippy: `$ cargo clippy`
3. To fail the linter check if clippy emits any warnings: `$ cargo clippy -- -D warnings`
4. To disable specific warnings, use the directive: $ #[allow(clippy::lint_name)] above the code block.

TODO: see the clippy README.md for more details.


### Formatting:
We will use the official rust formatter 'rustfmt' to automatically format our code. rustfmt is already included in the rustup package (like clippy), but if we need to install it for any reason:

1. To install rustfmt: `$ rustup component add rustfmt`
2. To format the whole project: `$ cargo fmt`
3. In the CI pipeline we will add: `$ cargo fmt -- --check`

TODO: Check the rustfmt README.md for details on tuning rustfmt.


### Security Vulnerabilities:
We will use cargo-audit to check our rust packages for exploitable vulnerabilities. cargo-audit is a convenient sub-command that checks if any vulnerabilities have been reported to the Advisory Databes maintained by the Rust Secure COde working group.

1. To install cargo-audit: `$ cargo install cargo-audit`
2. To run: `$ cargo audit`
3. We will include cargo-audit into our CI pipeline to run on every audit and to run daily to check for secureity updates.

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


**PROBLEMS:**
Both LLVM and 'tarpc-plugins' raised issues with rust-analyser.    

I used `$ cargo tree` to identify/confirm which crate was bringing in 'tarpc-plugins'. This is the part of the dependency tree related to LLVM and tarpaulin, where we can see tarpc-plugins is brought in by tarpaulin.    

Removing both LLVM and Tarpaulin temporarily from inclusion in the cargo.toml file until I can implement the necessary fixes, clears all outstanding problems in rust-analyser:    

[dependencies]
#llvm = "0.0.1"
cargo-watch = "8.5.2"
#tarpaulin = "0.1.0"
clippy = "0.0.302"
fmt = "0.1.0"
cargo-audit = "0.20.0" 