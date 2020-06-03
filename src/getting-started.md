# Getting Started

This documentation is _not_ intended to be comprehensive; it is meant to be a
quick guide for the most useful things. For more information, [see this
chapter](./building/how-to-build-and-run.md).

## Asking Questions

The compiler team (or "t-compiler") usually hangs out in Zulip [in this
"stream"][z]; it will be easiest to get questions answered there.

[z]: https://rust-lang.zulipchat.com/#narrow/stream/131828-t-compiler

**Please ask questions!** A lot of people report feeling that they are "wasting
expert time", but nobody on t-compiler feels this way. Contributors are
important to us.

Also, if you feel comfortable, prefer public topics, as this means others can
see the questions and answers, and perhaps even integrate them back into this
guide :)

### Experts

Not all `t-compiler` members are experts on all parts of `rustc`; it's a pretty
large project.  To find out who has expertise on different parts of the
compiler, [consult this "experts map"][map].

It's not perfectly complete, though, so please also feel free to ask questions
even if you can't figure out who to ping.

[map]: https://github.com/rust-lang/compiler-team/blob/master/content/experts/map.toml

### Etiquette

We do ask that you be mindful to include as much useful information as you can
in your question, but we recognize this can be hard if you are unfamiliar with
contributing to Rust.

Just pinging someone without providing any context can be a bit annoying and
just create noise, so we ask that you be mindful of the fact that the
`t-compiler` folks get a lot of pings in a day.

## Cloning and Building

The main repository is [`rust-lang/rust`][repo]. This contains the compiler,
the standard library (including `core`, `alloc`, `test`, `proc_macro`, etc),
and a bunch of tools (e.g. `rustdoc`, the bootstrapping infrastructure, etc).

[repo]: https://github.com/rust-lang/rust

There are also a bunch of submodules for things like LLVM, `clippy`, `miri`,
etc. You don't need to clone these immediately, but the build tool will
automatically clone and sync them (more later).

[**Take a look at the "Suggested Workflows" chapter for some helpful
advice.**][suggested]

[suggested]: ./building/suggested.md

### System Requirements

[**See this chapter for detailed software requirements.**](./building/prerequisites.md)
Most notably, you will need Python 2 to run `x.py`.

There are no hard hardware requirements, but building the compiler is
computationally expensive, so a beefier machine will help, and I wouldn't
recommend trying to build on a Raspberry Pi :P

- x86 and ARM are both supported (TODO: confirm)
- Recommended >=30GB of free disk space; otherwise, you will have to keep
  clearing incremental caches. More space is better, the compiler is a bit of a
  hog; it's a problem we are aware of.
- Recommended >=8GB RAM.
- Recommended >=2 cores; more cores really helps.
- You will need an internet connection to build; the bootstrapping process
  involves updating git submodules and downloading a beta compiler. It doesn't
  need to be super fast, but that can help.

Building the compiler takes more than half an hour on my moderately powerful
laptop. The first time you build the compiler, LLVM will also be built unless
you use system LLVM (see below).

Like `cargo`, the build system will use as many cores as possible. Sometimes
this can cause you to run low on memory. You can use `-j` to adjust the number
concurrent jobs.

Also, if you don't have too much free disk space, you may want to turn off
incremental compilation (see the "Configuring" section below). This will make
compilation take longer, but will save a ton of space from the incremental
caches.

### Cloning

You can just do a normal git clone:

```shell
git clone https://github.com/rust-lang/rust.git
```

You don't need to clone the submodules at this time.

**Pro tip**: if you contribute often, you may want to look at the git worktrees
tip in [this chapter][suggested].

### Configuring the Compiler

The compiler has a configuration file which contains a ton of settings. We will
provide some recommendations here that should work for most, but [check out
this chapter for more info][config].

[config]: ./building/how-to-build-and-run.html#create-a-configtoml

In the top level of the repo:

```shell
cp config.toml.example config.toml
```

Then, edit `config.toml`. You will need to search for, uncomment, and update
the following settings:

- `debug = true`: enables debug symbols and `debug!` logging, takes a bit longer to compile.
- `incremental = true`: enables incremental compilation of the compiler itself.
  This is turned off by default because it's technically unsound. Sometimes
  this will cause weird crashes, but it can really speed things up.
- `llvm-config`: enable building with system LLVM. [See this chapter][sysllvm]
  for more info. This avoids building LLVM, which can take a while.

[sysllvm]: ./building/suggested.html#building-with-system-llvm

### `./x.py` Intro

`rustc` is a bootstrapping compiler because it is written in Rust. Where do you
get the original compiler from? We use the current beta compiler
to build the compiler. Then, we use that compiler to build itself. Thus,
`rustc` has a 2-stage build.

We have a special tool `./x.py` that drives this process. It is used for
compiling the compiler, the standard libraries, and `rustdoc`. It is also used
for driving CI and building the final release artifacts.

### Building and Testing `rustc`

For most contributions, you only need to build stage 1, which saves a lot of time.
After updating `config.toml`, as mentioned above, you can use `./x.py`:

```shell
# Build the compiler (stage 1)
./x.py build --stage 1
```

This will take a while, especially the first time. Be wary of accidentally
touching or formatting the compiler, as `./x.py` will try to recompile it.

To run the compiler's UI test suite (the bulk of the test suite):

```
# UI tests
./x.py test --stage 1 src/test/ui
```

This will build the compiler first, if needed.

This will be enough for most people. Notably, though, it mostly tests the
compiler frontend, not codegen or debug info.  You can read more about the
different test suites [in this chapter][testing].

[testing]: https://rustc-dev-guide.rust-lang.org/tests/intro.html

If you only want to check that the compiler builds (without actually building
it) you can run the following:

```shell
./x.py check
```

To format the code:

```shell
# Actually format
./x.py fmt

# Just check formatting, exit with error
./x.py fmt --check
```

You can use `RUSTC_LOG=XXX` to get debug logging. [Read more here][logging].

[logging]: ./compiler-debugging.html#getting-logging-output

### Building and Testing `std`/`core`/`alloc`/`test`/`proc_macro`/etc.

TODO

### Building and Testing `rustdoc`

TODO

### Contributing code to other Rust projects

TODO: talk about things like miri, clippy, chalk, etc

## Contributor Procedures

There are some official procedures to know about. This is a tour of the
highlights, but there are a lot more details, which we will link to below.

### Code Review

When you open a PR on the `rust-lang/rust` repo, a bot called `@highfive` will
automatically assign a reviewer to the PR. The reviewer is the person that will
approve the PR to be tested and merged. If you want a specific reviewer (e.g. a
team member you've been working with), you can specifically request them by
writing `r? @user` (e.g. `r? @eddyb`) in either the original post or a followup
comment.

The reviewer may request some changes using the GitHub code review interface.
They may also request special procedures (such as a crater run; see below) for
some PRs.

When the PR is ready to be merged, the reviewer will issue a command to
`@bors`, the CI bot. Usually, this is `@bors r+` or `@bors r=@user` to approve
a PR (there are few other commands, but they are less relevant here). This puts
the PR in [bors's queue][bors] to be tested and merged. Be patient; this can take a
while and the queue can sometimes be long. PRs are never merged by hand.

[bors]: https://buildbot2.rust-lang.org/homu/queue/rust

### Bug Fixes or "Normal" code changes

For most PRs, no special procedures are needed. You can just open a PR, and it
will be reviewed, approved, and merged. This includes most bug fixes,
refactorings, and other user-invisible changes. The next few sections talk
about exceptions to this rule.

Also, note that is perfectly acceptable to open WIP PRs or GitHub [Draft
PRs][draft]. Some people prefer to do this so they can get feedback along the
way or share their code with a collaborator. Others do this so they can utilize
the CI to build and test their PR (e.g. if you are developing on a laptop).

[draft]: https://github.blog/2019-02-14-introducing-draft-pull-requests/

### New Features

Rust has strong backwards-compatibility guarantees. Thus, new features can't
just be implemented directly in stable Rust. Instead, we have 3 release
channels: stable, beta, and nightly.

- **Stable**: this is the latest stable release for general usage.
- **Beta**: this is the next release (will be stable within 6 weeks).
- **Nightly**: follows the `master` branch of the repo. This is the only
  channel where unstable, incomplete, or experimental features are usable with
  feature gates.

In order to implement a new feature, usually you will need to go through [the
RFC process][rfc] to propose a design, have discussions, etc. In some cases,
small features can be added with only an FCP (see below). If in doubt, ask the
compiler, language, or libs team (whichever is most relevant).

[rfc]: https://github.com/rust-lang/rfcs/blob/master/README.md

After a feature is approved to be added, a tracking issue is created on the
`rust-lang/rust` repo, which tracks the progress towards the implementation of
the feature, any bugs reported, and eventually stabilization.

The feature then needs to be implemented behind a feature gate, which prevents
it from being accidentally used.

Finally, somebody may propose stabilizing the feature in an upcoming version of
Rust. This requires an FCP (see below) to get the approval of the relevant teams.

After that, the feature gate can be removed and the feature turned on for all users.

[For more details on this process, see this chapter.](./implementing_new_features.md)

### Breaking Changes

As mentioned above, Rust has strong backwards-compatibility guarantees. To this
end, we are reluctant to make breaking changes. However, sometimes they are
needed to correct compiler bugs (e.g. code that compiled but should not) or
make progress on some features.

Depending on the scale of the breakage, there are a few different actions that
can be taken.  If the reviewer believes the breakage is very minimal (i.e. very
unlikely to be actually encountered by users), they may just merge the change.
More often, they will request a Final Comment Period (FCP), which calls for
rough consensus among the members of a relevant team. The team members can
discuss the issue and either accept, reject, or request changes on the PR.

If the scale of breakage is large, a deprecation warning may be needed. This is
a warning that the compiler will display to users whose code will break in the
future.  After some time, an FCP can be used to move forward with the actual
breakage.

If the scale of breakage is unknown, a team member or contributor may request a
[crater] run. This is a bot that will compile all crates.io crates and many
public github repos with the compiler with your changes. A report will then be
generated with crates that ceased to compile with or began to compile with your
changes. Crater runs can take a few days to complete.

[crater]: https://github.com/rust-lang/crater

### Major Changes

The compiler team has a special process for large changes, whether or not they
cause breakage. This process is call Major Change Proposal (MCP). MCP is a
relatively lightweight mechanism for getting feedback on large changes to the
compiler (as opposed to a full RFC or a design meeting with the team).

Example of things that might require MCPs include major refactorings, changes
to important types, or important changes to how the compiler does something.

**When in doubt, ask on [zulip][z]. We would hate for you to put a lot of work
into a PR that ends up not getting merged!**

### Performance

Compiler performance is important. We have put a lot of effort over the last
few years into [gradually improving it][perfdash].

[perfdash]: https://perf.rust-lang.org/dashboard.html

If you suspect that your change may cause a performance regression (or
improvement), you can request a "perf run" (your reviewer may also request one
before approving). This is yet another bot that will compile a collection of
benchmarks on a compiler with your changes. The numbers are reported
[here][perf], and you can see a comparison of your changes against the latest
master.

[perf]: https://perf.rust-lang.org

## Other Resources

- This guide: talks about how `rustc` works
- [The t-compiler zulip][z]
- [The compiler's documentation (rustdocs)](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/)

TODO: am I missing any?