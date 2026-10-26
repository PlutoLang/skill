---
name: pluto
description: Write, read, debug, and review Pluto (.pluto) code and Pluto-based projects. Pluto is a superset of Lua 5.5 with classes, switch, enums, string interpolation, lambdas, pipes, safe navigation, type hints, a preprocessor, and an extended standard library. Use this skill whenever the user mentions Pluto, .pluto files, plutoc/pluto, APM, or asks for Pluto code, and whenever you need to decide whether Lua code is compatible with Pluto.
---

# Pluto Development Skill

Pluto is a superset of Lua 5.5 (current series 0.13.x). Almost all Lua code runs unchanged, and Pluto adds syntax and library functions on top. Write idiomatic Pluto when the target is Pluto, and write plain Lua (or use compatibility mode) when the target must run in stock Lua.

For standard library signatures and integrator/sandbox configuration, read `reference.md` in this skill folder.

## 1. Decide the target first

Before writing code, establish which runtime it must run on:

- **Pluto only**: use any Pluto syntax and library functions.
- **Lua and Pluto**: avoid Pluto-only syntax and libraries. Stick to Lua 5.5 features.
- **Existing project with compatibility concerns**: check for `-- @pluto_use` lines, `pluto_use` statements, or `-c` usage, and follow the project's convention.

Remember that Pluto bytecode is not the same as Lua bytecode in some cases (see section 9). Lua 5.4 bytecode does not load into Pluto 0.13+, and Pluto 0.13+ bytecode does not load into Lua 5.4.

## 2. Keywords and compatibility

Pluto adds these reserved words: `switch`, `case`, `default`, `continue`, `enum`, `new`, `class`, `parent`, `export`.

- Pluto infers when a keyword is used as an identifier and only disables the keyword when the code clearly uses the feature. Still, prefer names that avoid these words (e.g. `kind` instead of `class`).
- Reserved words can be used as keys in shorthand table syntax (`{ class = "x" }`), as field names with `t.class`, and as `goto` labels.
- In compatibility mode, each keyword has a prefixed form that is always valid: `pluto_switch`, `pluto_continue`, `pluto_enum`, `pluto_new`, `pluto_class`, `pluto_parent`, `pluto_export`.
- To make a script parse identically in any environment, put this at the top:
  - `pluto_use "0.13.0"` (enables keywords up to that version), or
  - `pluto_use switch, continue, enum` (enable specific keywords), or
  - `-- @pluto_use * = false` (disable all keywords by default, then opt in).
- `pluto_use *` alone triggers a `bad-practice` warning. Prefer a version string.

Note: `try`/`catch` was removed in 0.13.0. Do not write `try ... catch`. Use `pcall`/`xpcall`, or `new exception("msg")` (which throws) and inspect `.what`/`.where`.

## 3. Syntax cheat sheet

### Blocks
`do` is a universal block opener and can replace `then`, `begin`, or `do`-less openers in `if`, `class`, `enum`, etc.
```pluto
if x == 3 do print("three") end
class Human do ... end
```

### Operators
- `!` = `not`, `!=` = `~=` (in expressions).
- Compound assignments (statements only): `+= -= *= /= //= %= ^= &= |= ~= <<= >>= ..= ??=`. They call the same metamethods as the plain operator.
- `++x` / `x++` (prefix/postfix increment; `++x` as a statement, also usable as expression).
- `**` = integer exponentiation (`2 ** 10` is integer 1024). `^` is float.
- Chained comparisons: `1 < x <= 10`.
- `a ?? b`: nil-coalescing (no metamethods). `t.k ??= v` assigns when nil.
- `a ? b : c`: ternary (no falsy-value pitfalls).
- `if c then a else b end` also works as an expression.
- `x <=> y`: spaceship, returns -1, 0, 1.
- Walrus: `if v := f() then ... end`, `while line := next_line() do ... end`. Only the first variable is checked for truthiness. Not usable inside a function call's arguments.
- Integer/bit ops from Lua are unchanged.

### Strings
- Interpolation: `$"Hello {name}, you have {#items} items"`. Any expression is allowed; values pass through `tostring`. Nested strings are allowed.
- Char indexing: `s[1]` (first char), `s[-1]` (last char), out of range gives `nil`. Not Lua bytecode compatible.
- `"hi":upper()` works (optional parentheses around string/table literals before method calls).
- Escape `\{` inside interpolated strings for a literal brace.
- Concatenating booleans works: `"x=" .. true`.

### Numbers
- Underscores: `10_000_000`. Binary `0b101010`, octal `0o52`, hex `0x2A`.

### Tables and objects
- Methods inline: `local t = { function f(self_arg) ... end }` creates a method (call with `:`).
- `static function f() end` inside a table or class creates a non-method.
- `$object(a, b)` creates `{ a = a, b = b }` from local variables named `a` and `b`. Use it to build tables from variables without repeating names.
- Trailing commas are allowed in tables, arguments, parameters, and destructuring.
- `table` default metatable: every table has `__mindex = table`, so `t:insert(v)`, `t:concat(",")`, `t:sort()` work. Fields take precedence over `__mindex`.

### Classes
```pluto
class Vehicle
    function __construct(protected manufacturer, public year) end
    function describe() return $"{self.year} {self.manufacturer}" end
end

class Car extends Vehicle
    model = "unknown"                 -- field with default
    private secret = 42
    function __construct(manufacturer, year, model)
        parent:__construct(manufacturer, year)  -- call parent constructor
        self.model = model
    end
end

local c = new Car("Toyota", 2025, "Corolla")
print(c:describe(), c instanceof Vehicle)
```
Rules:
- `class Name [extends Parent] ... end`, `local class Name ...`, `export class Name ...`. Also `local C = class ... end` as an expression.
- Use `new C(args)` to create instances. It calls `__construct` if defined. If the table has a static `new` function, `new C(...)` calls that instead.
- `__construct` receives `self`. If you don't define `__construct` on a subclass, the parent's is called automatically.
- Constructor promotion: `function __construct(public name, private age) end` assigns fields automatically.
- Access modifiers: `public` (default), `protected` (class and subclasses), `private` (class only). Violations are compile-time/runtime nil reads, not errors; don't rely on them for security.
- `parent.field` = `self.__parent.field`; `parent:method(...)` calls the parent method with `self`.
- `instanceof` (operator) and `instanceof(obj, Class)` (function) check inheritance.
- `extends` copies metamethods from the parent (the changelog documents `__index`, `__mindex`, `__newindex`, `__pairs`).

### Control flow
- `switch`:
```pluto
switch value do
    case 1, 2, 3:            -- shorthand fallthrough
        print("small")
        break                -- REQUIRED to stop fallthrough
    case 4:
        print("four")
        break
    default:
        print("other")
end
```
- Case conditions can be any expression. Method calls in case conditions need parentheses: `case (obj:get()):`.
- Switch expression: `local s = switch n do case 1 -> "one" case 2 -> "two" default -> "many" end`. Missing default gives `nil`.
- `continue` skips to the next loop iteration. `continue N` and `break N` affect outer loops. You **cannot** use `continue` directly inside a `switch`; use `break` in the switch and `continue` in the loop body, or restructure.
- `goto continue` with `::continue::` labels is also valid.
- `for t as v do ... end` iterates values only. `for k, v in t do ... end` iterates with no `pairs`/`next` needed (generalized iteration).
- Lambdas: `|a, b| -> a + b` (expression body) or `|x| -> do ... end` (block body). Type hints and defaults are allowed.
- Pipes: `value |> f |> g`. Only the first return value passes along. Extra args go in pipes: `"10" |> tonumber|16| |> print`. Method calls: `x |> obj:method`. Lambdas: `x |> |v| -> print(v)`.

### Functions
- Default args: `function f(mode = "r") end`.
- Named args: `f(file = "a", version = 2)` (works on functions whose signature the parser knows: local functions, `$declare`d functions, and `$declare`d hints). Positional args can come first, then named.
- `<nodiscard>` attribute: `local function add(a, b) <nodiscard> ... end` warns when the result is discarded (local functions only).
- Safe navigation:
  - `a?.b?.c` returns nil instead of erroring. Expression only, not assignable.
  - `obj?:method(x)` calls only if `obj` is non-nil. `obj:m?(x)` calls only if `m` exists.
  - Truncates multiple returns unless used in a fixed-count assignment.

### Destructuring
- Arrays: `local [a, b, c] = t`.
- Tables: `local { name, age } = t`, `local { n = name } = t`.
- Modules: `local { http, json, scheduler } = require "*"` (loads several pluto libraries at once).

### Enums
```pluto
enum Color do RED, GREEN = 10, BLUE end   -- RED=1, GREEN=10, BLUE=11
enum class Status begin OK, ERROR end       -- must use Status.OK
enum begin A, B end                          -- anonymous; A=1, B=2
Color:names() / :values() / :kvmap() / :vkmap()
```
Named enums also expose their enumerators as fields of the enum table (e.g. `Color.RED`).

### Export
`export function f() end` and `export x = 1` aggregate into a table returned at the end of the enclosing block/file. A file can be a module with no explicit `return` needed.

### Type hints (compile-time warnings only, no runtime cost)
- `local x: string = "a"`, `function f(a: number): string`.
- Types: `string number int float boolean/bool function table userdata nil any void`.
- Unions: `string|int`. Optional: `?string` (nil or string).
- Multiple returns: `(bool, string)`.
- Table shape: `{ x: number, y: number }`.
- Named types: `$type Point = { x: number, y: number }`.
- Function types: `$type Cb = function(msg: string): void`.
- Globals: `$declare _VERSION: string` and `$declare function tonumber(s: string, base: ?number): number`.
- `$getproptype(x)` reports the parser's inferred type (don't depend on exact output).

Type mismatches produce `type-mismatch` warnings; they don't stop compilation unless warnings are errors.

### Preprocessor (compile time)
- `$define NAME = const` (must be a compile-time constant).
- `$if COND then ... $elseif ... $else ... $end` with compile-time conditions (equality checks allowed).
- `$alias name = expr`, `$alias f(a, b) = a + b`. Multi-line with `\`.
- `$include "file.pluto"` for shared headers (e.g. `$declare` files).
- `$haltcompiler` stops parsing; everything after it is ignored.
- Compile-time function calls with `$`: `$crypto.joaat("x")`, `$math.floor(2.5)`, `$io.contents("f")`, `$tostring`, `$tonumber`, `$utostring`, `$utonumber`, `$assert`, and any preloaded Pluto library function (`base64`, `json`, ...). Arguments must be compile-time known.
- `$expr` evaluates select functions at compile time.

### Misc
- `local x <const> = 1` for constants; `local x <close>` for to-be-closed values.
- `$object(a, b)` creates `{ a = a, b = b }`.
- `string.format("%Q", s)` gives UTF-8-safe quoting.
- `range(10)` gives `{1..10}`; `range(4, 8)` gives `{4..8}`.
- `callonce(f)` caches the first result of `f`.

## 4. Warnings

Warnings appear at compile time. Common types:
`unused`, `var-shadow`, `global-shadow` (off by default), `type-mismatch`, `unreachable-code`, `excessive-arguments`, `bad-practice`, `possible-typo`, `non-portable-code`, `non-portable-name`, `non-portable-bytecode` (off by default), `unannotated-fallthrough`, `discarded-return`, `field-shadow`.

Control them with comments:
- `-- @pluto_warnings: disable-var-shadow` / `enable-...` / `error-...`
- `-- @pluto_warnings: enable-all, disable-non-portable-bytecode`
- `-- @pluto_warnings: disable-next` (applies to the next line)
- `-- @fallthrough` before a case to mark intentional fallthrough.

## 5. Standard library and modules

- Pluto-specific libraries are loaded with `require "pluto:name"` or `require "name"`. Local files take precedence over built-in libraries of the same name. `require "pluto:*"` or `require "*"` returns all of them for destructuring.
- Built-in libraries include: `assert`, `base32`, `base64`, `bigint`, `buffer`, `canvas`, `cat`, `coroutine` (extended), `crypto`, `exception`, `ffi`, `http`, `io` (extended), `json`, `math` (extended), `os` (extended), `regex`, `scheduler`, `socket`, `string` (extended), `table` (extended), `url`, `vector3`, `wasm`, `xml`.
- Global additions: `_PVERSION`, `os.platform`, `os.arch`, `dumpvar`, `exportvar`, `range`, `compareversions`, `wcall`, `sdiv`, `udiv`, `smod`, `umod`, `callonce`, `instanceof`.
- The full signature list is in `reference.md`. Check there before inventing a function name.

Key behaviors to remember:
- `table.sort` returns the table (so `t = t:sort()` works). `table.sorted` returns a copy.
- `string.split(s, sep, limit)` requires `sep`. `split("ABC", "")` yields characters.
- `io.contents(path)` is a getter, and `io.contents(path, data)` is a setter. Both are faster than `io.open`.
- `io.remove`, `io.rename`, `io.makedir` (and `mkdir`), `io.makedirs`, `io.listdir` (`scandir`), `io.copy`, `io.part`, `io.absolute`, `io.relative`, `io.currentdir` (`cwd`, `chdir`), `io.writetime`, `io.filesize`, `io.exists`, `io.isfile`, `io.isdir` are available. Pluto's io functions assume UTF-8 paths.
- `json.encode(v, "compact"|"pretty"|"msgpack")`. Use `__order = {"a","b"}` to fix key order. `json.decode(s, json.withnull | json.withorder | json.msgpack)`.
- `http.request(url_or_options)` returns `body, status, headers, status_text` on success, or `nil, reason` on failure. It yields when called inside a coroutine and blocks otherwise.
- `coroutine.sleep(ms)` yields (non-blocking). `os.sleep(ms)` blocks. Use `coroutine.sleep` inside scheduled coroutines.
- `scheduler`: `new (require "pluto:scheduler")()`, then `:add(fn)`, `:addloop(fn)` (repeat while true), `:run()`. Set `yieldfunc`/`errorfunc` as needed.
- `exception` class: `new exception("msg")` throws. Fields: `.what`, `.where`.
- `assert` library: `require "assert"` then `assert.equal`, `assert.nequal`, `assert.istrue`, `assert.isnil`, `assert.haserror`, `assert.searcherror`, `assert.contains`, etc.
- `regex` library: `new regex [[/pattern/flags]]`, then `:match`, `:search`, `:replace`, `:substitute`. Flags include `g`.
- `bigint`: `new bigint("123")`; arithmetic metamethods; `:div()` returns quotient and remainder; `:tohex()` is `:hex()`.
- `buffer`: `new buffer()`, `:append()`, `tostring(b)`. Use for building large strings.
- `socket`: TCP/UDP clients and servers. `socket.bind(sched, port, fn)` spins up a coroutine per client. Not available in WASM builds.
- `crypto`: many hashes (`md5`, `sha1`, `sha256`, `sha384`, `sha512`, `ripemd160`, `whirlpool`, non-crypto `djb2`, `fnv1a`, `crc32`, `joaat`, etc.), `hmac`, AES (`encrypt`/`decrypt` with cbc, cfb, ecb, gcm), RSA, Curve25519 (`x25519`), `random` (CSPRNG), `compress`/`decompress` (deflate, lzf).
- `ffi` (Pluto) loads native libraries. Not in WASM. Avoid unless the user asked for native interop.
- `canvas`, `wasm`, `xml`, `cat`, `url`, `vector3`, `base32`, `base64`: see `reference.md`.
- Default metatables: tables get `__mindex = table`; threads/coroutines get `__index = coroutine`; strings have their standard metatable.

## 6. Packages

- APM (Agnostic Package Manager) is the recommended way to pull in third-party code. Write a `deps.pluto` script that loads APM and declares `git` dependencies with `from` and `to` paths. Re-run it to update.
- Known packages: `iniparser`, `pluto-discord-bot`, `pluto-websocket`, `pluto-zip`, `pluto-query`, `pluto-dns`, `pluto-s3`, `pluto-pg`.
- `package.path` already searches `.pluto` files and `./?/init.pluto`. To include a `lib/` directory, append to `package.path`.

## 7. Writing Pluto code: checklist

1. Confirm the target runtime (Pluto-only vs. also Lua).
2. Use `pluto_use "0.13.0"` at the top of shared/module files for portability.
3. Prefer `class` over hand-rolled metatables, and `new` over `setmetatable`, when writing Pluto-only code.
4. Use `switch` with `break` in every case unless you want fallthrough (then add `-- @fallthrough`).
5. Use `coroutine.sleep` (not `os.sleep`) inside scheduler tasks.
6. Use `?.`, `??`, and `?:` for optional access rather than nested `if` guards.
7. Use `$"..."` interpolation rather than `..` + `tostring`.
8. Use `io.contents` for whole-file read/write.
9. Use `json`/`crypto`/`regex` from the built-in libraries rather than writing your own.
10. Don't use `try`/`catch`, `let`, `const` (statement form), `<constexpr>`, `$type(x)` (use `$type Name = ...`), `crypto.hexstream`, or `string.find_first_of`. These were removed.
11. Check `reference.md` before using any function you haven't verified.

## 8. Common pitfalls

- Forgetting `break` in a `switch` case leads to fallthrough.
- `continue` inside `switch` is invalid. Use `break` to leave the switch and restructure.
- A method call in a `case` expression must be parenthesized: `case (t:get()):`.
- `in` on tables searches **values**, not keys. `"x" in "text"` searches substrings. On strings, the left side must be a string.
- Pipes forward only the first return value. Wrap in parentheses to restrict.
- Compound operators (`+=` etc.) are statements only. They can't be used in expressions.
- `?.` cannot be used on the left side of an assignment.
- Walrus `:=` cannot appear inside a function call's argument list.
- `table.sort` returns the table; `table.sorted` returns a new one.
- `enum` and `class` names can clash with variables named `class`/`enum` unless `pluto_use` or prefixed keywords are used.
- `ffi`, `socket`, and `os.execute`-style functions are unavailable under WASM and sandboxed builds (see integrator settings in `reference.md`).
- Bytecode: string indexing, `in`, the iteration shortcut `for k,v in t`, `for-as`, and some class/`switch` output are not Lua-bytecode-compatible. Only run such bytecode in Pluto.

## 9. Bytecode and compatibility

- Pluto 0.13+ is based on Lua 5.5. Bytecode from Lua 5.4 or earlier Pluto versions will not run in 0.13+, and vice versa.
- Pluto reads and writes Lua 5.5 bytecode. Features documented as non-portable (e.g. `in` on strings/tables, string indexing, `for k,v in t` shortcut, `for t as v`) produce bytecode that plain Lua cannot run. Enable `non-portable-bytecode` warnings to catch these.
- Lua modules load fine in Pluto. Pluto modules loaded in plain Lua need to avoid Pluto syntax.
- Compatibility mode (`-c` on `pluto`/`plutoc`, or `-- @pluto_use * = false`) disables Pluto-only keywords by default.

## 10. Running and building

- `pluto script.pluto` runs a script. `plutoc` compiles to bytecode and lints.
- Editor support: VS Code (`calamity-inc.pluto-syntax-highlighting`), Sublime Text (Package Control: "Pluto"), Visual Studio (`calamity-inc.pluto-vsix`), IntelliJ (TextMate bundle from the Syntax-Highlighting repo).
- Online: "Write Pluto Online" at pluto-lang.org/web (WASM).

Use the `reference.md` file for detailed library signatures, integrator build flags, and sandboxing options.

### Installing on Debian / Ubuntu (preferred in a Linux sandbox)

Run these as root, or prefix `sudo` if you are not root and sudo is available:

```bash
wget -qO- https://calamity-inc.github.io/deb-repo/key.gpg | sudo tee /usr/share/keyrings/calamity-inc.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/calamity-inc.gpg] https://calamity-inc.github.io/deb-repo/ buster main" | sudo tee /etc/apt/sources.list.d/calamity-inc.list > /dev/null
sudo apt update
sudo apt install pluto
```

- The package provides `pluto` and `plutoc`, the shared library, and headers under `<pluto/...>`.
- The distribution name `buster` is hard-coded in the repo line. Keep it as written; the repo is not tied to the host's release. If `apt update` or `apt install` fails, report the exact error and fall back to one of the options below.
- Verify the install with a script rather than assuming flags:
  ```bash
  printf 'print(_PVERSION)\n' > /tmp/v.pluto && pluto /tmp/v.pluto
  ```

### Fallbacks when apt is unavailable

1. **Prebuilt binary from GitHub Releases** (`https://github.com/PlutoLang/Pluto/releases`). List the assets of the latest release with the GitHub API (`curl -sL https://api.github.com/repos/PlutoLang/Pluto/releases/latest`), pick the Linux asset, download it with `curl -L -o`, extract it, and `chmod +x` the binaries. Confirm the architecture matches `uname -m`.
2. **Build from source.** Source is not in the sandbox by default, so obtain it first, either with `git clone https://github.com/PlutoLang/Pluto` or by downloading `https://github.com/PlutoLang/Pluto/archive/refs/heads/main.zip` and unpacking it. Then build:
   - Install prerequisites: `sudo apt install git php-cli clang make` (PHP is needed only for the PHP scripts).
   - PHP scripts (from the repo root): `php scripts/compile.php clang`, then `php scripts/link_pluto.php clang` and `php scripts/link_plutoc.php clang`. Binaries go to `src/`.
   - Or `make -j PLAT=<platform>` from the repo root. Check `src/Makefile` for the platform names; on Linux this is usually `linux`.
3. Other package managers (not usually present in Debian or Ubuntu sandboxes): Chocolatey, AUR, Termux.
