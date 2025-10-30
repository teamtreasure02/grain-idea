# grain - package manager for steel

**team**: teamtreasure02 (taurus ♉ - building blocks)  
**purpose**: install, manage, and distribute Steel modules  
**status**: MVP specification, prototype in progress! 🌾⚒️

---

## what is grain?

`grain` is the package manager for Steel (Rust-hosted Scheme)!

think of it like:
- `cargo` for Rust
- `npm` for Node  
- `pip` for Python
- **`grain` for Steel!** 🌾

---

## why grain needed this

**the problem:** Steel has no package manager!

currently to use a Steel module:
1. manually clone the repo
2. figure out the path
3. write `(require "/full/path/to/module.scm")`
4. hope dependencies work

**not ideal!** 😅

**with grain:**
```bash
grain install grainorder
```

done! now in any Steel script:
```scheme
(require "grainorder.scm")
```

it just works! ⚒️

---

## usage (mvp commands)

### install a module

```bash
grain install grainorder
grain install teamtreasure02/grainbuild
grain install kae3g/graintime
```

**what it does:**
1. clones repo from GitHub
2. checks out stable branch (graintime!)
3. installs to `~/.grain/modules/`
4. adds to Steel require path

### list installed modules

```bash
grain list
```

**output:**
```
📦 installed modules:
  grainorder (xzvsjl @ 1740-PDT)
  grainbuild (xzvshm @ 1740-PDT)
  graintime (coming soon!)
```

### update a module

```bash
grain update grainorder
```

**what it does:**
1. fetches latest stable branch
2. checks grainorder (newer = smaller!)
3. updates if available

### search for modules

```bash
grain search grainorder
grain search time
```

**queries grain registry** (VPS-hosted index!)

---

## architecture

### grain CLI (rust binary)

**location:** `~/.grain/bin/grain`

**structure:**
```
grain/
├── src/
│   ├── main.rs              (CLI entry point)
│   ├── install.rs           (module installation)
│   ├── registry.rs          (HTTP registry client)
│   ├── grainorder.rs        (version comparison!)
│   └── config.rs            (user settings)
├── Cargo.toml
└── readme.md
```

**size goal:** <1000 lines total (small, focused!)

### grain registry (HTTP API)

**hosted:** VPS at `registry.grain.network` (or similar)

**endpoints:**
- `GET /modules` - list all modules
- `GET /modules/{name}` - module metadata
- `GET /search?q=time` - search modules

**response format:**
```json
{
  "name": "grainorder",
  "org": "teamtreasure02",
  "repo": "https://github.com/teamtreasure02/grainorder",
  "stable_branch": "12025-10-29--1740-PDT--moon-shravana-asc-arie25-sun-08h--teamtreasure02",
  "grainorder": "xzvsjl",
  "description": "Permutation-based file chronology",
  "modules": ["grainorder.scm", "grainorder-macros.scm"]
}
```

### local installation

**structure:**
```
~/.grain/
├── bin/grain                  (the CLI itself!)
├── modules/                   (installed packages)
│   ├── grainorder/
│   │   ├── grainorder.scm
│   │   ├── grainorder-macros.scm
│   │   └── function-box-*.scm
│   └── grainbuild/
│       └── ...
├── config.toml                (user settings)
└── registry-cache.json        (cached module list)
```

**Steel require path updated:** `~/.grain/modules/` added automatically!

---

## commands (full spec)

### package management

```bash
grain install <module>         # install module
grain uninstall <module>       # remove module
grain update <module>          # update to latest stable
grain update                   # update all modules
grain list                     # show installed
grain search <query>           # search registry
grain info <module>            # show module details
```

### development

```bash
grain init <name>              # create new module scaffold
grain test                     # run module tests
grain publish                  # publish to registry
grain link                     # link local dev version
```

### configuration

```bash
grain config set registry <url>  # change registry
grain config show                # show settings
grain cache clear                # clear cached data
```

---

## grainorder integration

**key insight:** grain uses grainorder for version comparison!

**how it works:**
- each release has a grainorder (in grainbranch name!)
- newer = smaller grainorder
- grain compares: `xzvsjl` < `xzvsjm` → newer!

**benefits:**
- no semver parsing!
- chronologically precise
- works with graintime branches
- leverages existing grainorder algorithm!

**example:**
```bash
grain update grainorder

# grain checks:
# installed: xzvsjm (1150-PDT)
# available: xzvsjl (1150-PDT)  
# xzvsjl < xzvsjm → NEWER! update!
```

---

## registry design

### simple HTTP + JSON

**philosophy:** keep it simple!

**NOT:**
- ❌ complex GraphQL
- ❌ authentication (for MVP)
- ❌ fancy CDN
- ❌ blockchain

**YES:**
- ✅ static JSON files
- ✅ simple HTTP GET
- ✅ GitHub as source of truth
- ✅ cached locally

### registry.json structure

```json
{
  "modules": [
    {
      "name": "grainorder",
      "org": "teamtreasure02",
      "stable": "12025-10-29--1740-PDT--moon-shravana-asc-arie25-sun-08h--teamtreasure02",
      "grainorder": "xzvsjl",
      "files": ["grainorder.scm", "grainorder-macros.scm", "function-box-fs.scm"]
    },
    {
      "name": "grainbuild",
      "org": "teamtreasure02",
      "stable": "12025-10-29--1740-PDT--moon-shravana-asc-arie25-sun-08h--teamtreasure02",
      "grainorder": "xzvshm",
      "files": ["grainbuild.scm", "grainbuild-macros.scm", "grainbuild-specs.scm"]
    }
  ],
  "updated": "12025-10-29--1810-PDT"
}
```

**hosted as:** `https://registry.grain.network/registry.json`

**updated:** whenever a module publishes a new stable branch!

---

## mvp implementation plan

### phase 1: local install (no registry)

**Goal:** `grain install teamtreasure02/grainorder` works

**Implementation:**
1. parse org/repo from command
2. clone from GitHub
3. checkout stable branch (latest graintime!)
4. copy to `~/.grain/modules/`
5. success!

**No VPS needed yet!** Just Git + filesystem!

### phase 2: simple registry

**Goal:** `grain install grainorder` works (no org needed!)

**Implementation:**
1. static `registry.json` file
2. host on simple VPS (nginx serving JSON!)
3. `grain` fetches registry
4. resolves `grainorder` → `teamtreasure02/grainorder`
5. installs as phase 1

**VPS cost:** ~$5/month (smallest DigitalOcean droplet!)

### phase 3: publish command

**Goal:** `grain publish` uploads your module

**Implementation:**
1. reads your repo metadata
2. generates JSON entry
3. submits to registry (HTTP POST)
4. registry updates `registry.json`
5. module now searchable!

### phase 4: dependencies

**Goal:** transitive dependency installation

**Implementation:**
1. `grain.toml` in each module lists deps
2. `grain install` reads it
3. installs deps recursively
4. dependency graph resolution

---

## implementation: rust CLI

### grain CLI structure

```rust
// src/main.rs
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "grain")]
#[command(about = "Package manager for Steel modules 🌾", long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Install { module: String },
    List,
    Update { module: Option<String> },
    Search { query: String },
}

fn main() {
    let cli = Cli::parse();
    match cli.command {
        Commands::Install { module } => install_module(&module),
        Commands::List => list_modules(),
        Commands::Update { module } => update_modules(module),
        Commands::Search { query } => search_registry(&query),
    }
}
```

**Dependencies:**
- `clap` - CLI parsing
- `reqwest` - HTTP client
- `serde_json` - JSON parsing
- `git2` - Git operations

**Size estimate:** ~500 lines Rust (clean, fast!)

---

## installation

### install grain itself

```bash
curl -sSL https://grain.network/install.sh | sh
```

**what it does:**
1. downloads grain binary
2. installs to ~/.grain/bin/grain
3. adds to PATH
4. creates ~/.grain/ structure

### or build from source

```bash
git clone https://github.com/teamtreasure02/grain
cd grain
cargo build --release
cp target/release/grain ~/.grain/bin/
```

---

## grain.toml format

**in each module:**

```toml
[package]
name = "grainorder"
org = "teamtreasure02"
grainorder = "xzvsjl"
graintime = "12025-10-29--1740-PDT--moon-shravana-asc-arie25-sun-08h--teamtreasure02"

[dependencies]
# no deps for grainorder!

[files]
main = "grainorder.scm"
macros = "grainorder-macros.scm"
tests = "grainorder-test.scm"
function-boxes = [
  "function-box-fs.scm",
  "function-box-strings.scm"
]
```

**grainbuild example:**

```toml
[package]
name = "grainbuild"
org = "teamtreasure02"

[dependencies]
grainorder = "teamtreasure02/grainorder"  # needs grainorder!

[files]
main = "grainbuild.scm"
macros = "grainbuild-macros.scm"
specs = "grainbuild-specs.scm"
```

---

## vps setup (simple!)

### option 1: static hosting

**cheapest:** GitHub Pages!
- Host `registry.json` on GitHub
- Free, reliable, fast
- `https://teamtreasure02.github.io/grain-registry/registry.json`

### option 2: small VPS

**DigitalOcean $4/month droplet:**
```bash
# install nginx
apt install nginx

# serve registry
echo '{"modules": [...]}' > /var/www/html/registry.json

# done!
```

**access:** `http://YOUR-IP/registry.json`

### option 3: fancy domain

**grain.network domain:**
- Register domain (~$12/year)
- Point to VPS or GitHub Pages
- `https://registry.grain.network/registry.json`

**Start with GitHub Pages (free!), upgrade later!** 🌾

---

## next steps (rapid!)

### 1. create MVP rust CLI (tonight!)

```bash
cd ~/github/teamtreasure02/grain
cargo init --bin
```

**implement:**
- `grain install` (clone from GitHub!)
- `grain list` (show ~/.grain/modules/)
- minimal, working!

### 2. test it locally (10 min!)

```bash
grain install teamtreasure02/grainorder
steel -c '(require "grainorder.scm") (displayln (prev-grainorder "xzvsnm"))'
```

**proves the concept!**

### 3. create registry.json (5 min!)

```json
{
  "modules": [
    {"name": "grainorder", "org": "teamtreasure02"},
    {"name": "grainbuild", "org": "teamtreasure02"},
    {"name": "grain-steel-stdlib", "org": "teamtreasure02"}
  ]
}
```

Host on GitHub Pages!

### 4. publish to crates.io (when ready!)

```bash
cargo publish
```

**anyone can:** `cargo install grain-pm`

---

## why this is HUGE

### for steel community

- ✅ First package manager for Steel!
- ✅ Lowers barrier to entry
- ✅ Encourages module sharing
- ✅ Accelerates ecosystem growth

### for grain network

- ✅ Easy distribution (`grain install grainorder`!)
- ✅ Version management (grainorder comparison!)
- ✅ Dependency resolution (grain.toml!)
- ✅ Discovery (search registry!)

### for adoption

- ✅ Professional tool (like real languages!)
- ✅ Easy onboarding (one command install!)
- ✅ Community building (shared registry!)
- ✅ Steel becomes production-ready!

---

## implementation timeline

### tonight (2-3 hours)

- ⚒️ Create Rust CLI skeleton
- ⚒️ Implement `grain install teamtreasure02/grainorder`
- ⚒️ Test locally

### tomorrow

- ⚒️ Add `grain list`, `grain update`
- ⚒️ Create registry.json
- ⚒️ Host on GitHub Pages

### this week

- ⚒️ Implement search
- ⚒️ Add grain.toml parsing
- ⚒️ Dependency resolution
- ⚒️ Publish to crates.io!

### this month

- ⚒️ `grain publish` command
- ⚒️ VPS registry (if needed)
- ⚒️ Community modules added
- ⚒️ Steel team feedback!

---

## this could change EVERYTHING

imagine:
```bash
# install steel itself
curl -sSL https://steel-lang.org/install.sh | sh

# install grain
cargo install grain-pm

# start building!
grain install grainorder
grain install grainbuild
grain install graintime

# your project:
steel myproject.scm
```

**Steel becomes as easy as Python/Node/Rust!** 🚀

**This could drive Steel adoption massively!** 🌾⚒️

---

## license

dual-licensed under your choice of:
- **mit license** - see [license-mit.md](license-mit.md)
- **apache license 2.0** - see [license-apache.md](license-apache.md)

you may use this software under either license, or under any other permissive open source license of your choosing, provided you include attribution to the original authors.

**we believe in maximum freedom for users and developers!** 🌾

---

now == next + 1 🌾

**grain** - harvesting the Steel ecosystem, one module at a time! ⚒️🌾🚀

ready to build the MVP TONIGHT? let's GO! 🔥

