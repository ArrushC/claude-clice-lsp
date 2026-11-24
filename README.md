# clice-lsp

C/C++ language server ([clice](https://github.com/clice-io/clice)) for Claude Code, providing code intelligence and diagnostics through Claude Code's LSP tool.

This plugin only tells Claude Code how to launch clice (`clice serve`, over stdio). It does not ship the server: install clice separately and put it on your `PATH`.

## Supported Extensions

`.c`, `.h`, `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hxx`, `.C`, `.H`, and the C++20 module extensions `.cppm`, `.ixx`, `.mpp`

## Why clice instead of clangd?

clice is built on LLVM/Clang like clangd, but with a different architecture. The reasons you might pick it:

- **C++20 modules.** Named modules work from a `compile_commands.json` that lists the module files. clice scans the dependencies and builds the modules it needs on demand, whatever the build system or compiler.
- **Header contexts.** A header that is not self-contained compiles in the context of a source file that includes it, so it needs no entry of its own in the compilation database.
- **Template awareness.** clice resolves dependent names inside templates without concrete arguments, and go-to-implementation, references and call hierarchy on a dependent call list what each instantiation calls.
- **Crash isolation and memory management.** Compiles run in a pool of worker processes instead of threads. A compile that crashes or leaks costs one worker, not the server, and clice manages memory use and compile priorities across the pool. PCHs and PCMs are cached on disk and reused across sessions.

See [What is clice?](https://docs.clice.io/clice/guide/what-is-clice) for the full comparison.

## Installation

Requires clice v1.0 or later. Older alpha builds were launched with different flags and do not have the `serve` subcommand.

### Prebuilt binaries (Linux, macOS, Windows)

Download the archive for your platform from the [releases page](https://github.com/clice-io/clice/releases) and extract it. The archive holds `bin/clice` and `lib/clang`, the resource directory clice reads Clang's headers from. Keep the two together: add the extracted `bin` directory to your `PATH`, or symlink the binary, rather than copying `clice` out on its own.

```bash
# Linux x86_64, v1.0.0. The Linux packages need glibc 2.17 or newer.
curl -LO https://github.com/clice-io/clice/releases/download/v1.0.0/clice-1.0.0.x86_64-unknown-linux-gnu.tar.gz
mkdir -p ~/.local/opt ~/.local/bin
tar -xzf clice-1.0.0.x86_64-unknown-linux-gnu.tar.gz -C ~/.local/opt
ln -s ~/.local/opt/clice/bin/clice ~/.local/bin/clice
```

macOS archives are `clice-<version>.<arch>-apple-darwin.tar.gz` and install the same way. On Windows, extract `clice-<version>.<arch>-w64-mingw32.zip` and add its `clice\bin` folder to `PATH`.

### Arch Linux (AUR)

```bash
yay -S clice-bin
```

### Verify

```bash
clice --help
```

## Project Setup

clice needs a `compile_commands.json` ([compilation database](https://clang.llvm.org/docs/JSONCompilationDatabase.html)) to know how each file is compiled. By default it loads every `compile_commands.json` in the project root and in its immediate subdirectories (such as `build/`). Generate one with your build system:

```bash
# CMake (Makefile and Ninja generators)
cmake -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

# Make and other build systems, with Bear
bear -- make

# xmake
xmake project -k compile_commands --lsp=clangd build
```

Meson writes one into its build directory on `meson setup build`. The clice [quick start](https://docs.clice.io/clice/guide/quick-start) covers Bazel, Visual Studio and others.

Project configuration is optional and goes in `clice.toml` in the project root (or `.clice/config.toml`). Use it when the database lives somewhere clice does not look by default:

```toml
[[rules]]
compile_commands = ["out/release"]
```

See the [configuration guide](https://docs.clice.io/clice/guide/config-guide). clice keeps its cache, index and logs in `.clice/` in the project, which you will probably want in `.gitignore`.

## Do not enable alongside clangd-lsp

`clangd-lsp` and `clice-lsp` both claim the same file extensions. Enable only one of them for a project, otherwise two servers compete for the same files. Disable the other with `/plugin`, or:

```bash
claude plugin disable clangd-lsp@claude-plugins-official
```

## Install the plugin

```
/plugin marketplace add ArrushC/clice-lsp
/plugin install clice-lsp@clice-lsp
```

## Test locally

From a checkout of this repository:

```bash
claude plugin validate .
claude --plugin-dir ./
```

Then open a C or C++ project that has a `compile_commands.json` and ask Claude to go to a definition or list a file's symbols. If the server does not start, check that `clice` resolves on the `PATH` Claude Code runs with, and run `claude --debug` to see the LSP startup log. clice's own logs are in `.clice/logs/` in the project.

## More Information

- [clice documentation](https://docs.clice.io/clice/)
- [clice on GitHub](https://github.com/clice-io/clice)
- [Editor setup](https://docs.clice.io/clice/guide/editors)

## License

MIT, see [LICENSE](LICENSE). clice itself is licensed separately under Apache-2.0.
