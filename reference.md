# Pluto Reference

Companion to `SKILL.md`. Use this file to verify function signatures and to configure sandboxed or embedded builds.

## Global additions

| Name | Signature / notes |
|---|---|
| `_PVERSION` | Pluto version string. `nil` on plain Lua. |
| `os.platform` | `"windows"`, `"emscripten"`, `"linux"`, `"macos"`, `"android"`, `"unknown"`. |
| `os.arch` | e.g. `"x86, 64-bit"`, `"arm, 64-bit"`, `"wasm, 32-bit"`. |
| `dumpvar(v)` | Returns a readable string. Strings show `string(N) "..."`. Includes metatable `__name`. |
| `exportvar(v)` | Returns valid Lua/Pluto source for `v`. Errors for unrepresentable values (C functions). |
| `range(n)` / `range(a, b)` | Table of numbers, inclusive. |
| `compareversions(a, b)` | Three-way compare of version strings. Pre-release (`-dev`) is less than final. |
| `wcall(f, ...)` | Calls `f` and returns a string of warnings raised. |
| `sdiv(a, b)`, `udiv(a, b)` | Signed/unsigned integer division. |
| `smod(a, b)`, `umod(a, b)` | Signed/unsigned integer modulo. |
| `callonce(f)` | Calls `f` once, caches its return value. |
| `instanceof(obj, Class)` | Function form of the `instanceof` operator (Pluto-only). |

Default metatables: tables use `{ __mindex = _G.table }` (so `t:insert(...)` works; fields take precedence). Threads/coroutines use `{ __index = _G.coroutine }`.

## `string` (extended)

| Function | Signature / behavior |
|---|---|
| `string.format` | Adds `%Q` (UTF-8-safe quoting). |
| `s:upper(i?)` / `s:lower(i?)` | Optional index: change only that character. |
| `s:split(sep, limit?)` | Returns table. `sep` is required. `""` splits into characters. |
| `s:rfind(pattern)` | Like `string.find` but searches right-to-left. Accepts Lua patterns. |
| `s:strip(chars?)`, `s:lstrip(chars?)`, `s:rstrip(chars?)` | Default chars: space, `\n \r \t \v \0`. |
| `s:isascii()`, `s:islower()`, `s:isupper()`, `s:isalpha()`, `s:isalnum()`, `s:iswhitespace()` | Boolean checks. Whitespace isn't alpha/lower/upper. |
| `s:uformat(...)` | Like `format` but under `en_US.UTF-8` locale. |
| `s:replace(old, new, max?)` | Plain-text replace; no capture limit. Errors if `max < 0` or `new` is empty. `max` default is unlimited. |
| `s:truncate(len, ellipsis?)` | If `ellipsis` is true, last 3 chars become `...` when truncated. |
| `s:contains(sub)` | Boolean. |
| `string.casefold(a, b)` | Case-insensitive equality. |
| `string.formatint(n, sep?, group?)` | Inserts separators every `group` digits. Accepts integer or digit string (optionally with a leading sign). |
| `s:partition(sep, fromright?)` | Returns `before, after` split once. |
| `s:startswith(prefix)`, `s:endswith(suffix)` | Boolean. |
| `s:tohex(spaced?, upper?)`, `s:fromhex()` | Hex encode/decode. |
| `s:span(start, len)` | Substring by start and length (unlike `sub`, which uses start and end). |

## `table` (extended)

| Function | Signature / behavior |
|---|---|
| `table.sort(t, cmp?)` | Returns the table (so `t = t:sort()` is safe). |
| `table.sorted(t, cmp?)` | Copying variant. Nested tables are not copied. |
| `table.size(t)` | Counts array and hash parts. |
| `table.clear(t)` | Removes all entries. |
| `table.contains(t, v)` | Returns the index/key where `v` is found, or `nil`. |
| `table.find(t, fn)` | Returns the first element where `fn(el)` is truthy, or `nil`. |
| `table.findkey(t, fn)` / `table.findindex(t, fn)` | Returns the key/index. |
| `table.reverse(t)` / `table.reversed(t)` | In-place / copy. Returns the table. |
| `table.reorder(t)` / `table.reordered(t)` | Makes the array part contiguous. |
| `table.foreach(t, fn, withKey?)` | Non-nil callback return is propagated. |
| `table.filter(t, fn, withKey?)` / `table.filtered` | Removes entries where `fn` is falsy. Pair with `reorder`. |
| `table.map(t, fn, withKey?)` / `table.mapped` | Replaces each value. |
| `table.reduce(t, fn, init?)` | `acc = fn(acc, v)`. Default init is `0`. |
| `table.min(t)` / `table.max(t)` | Smallest/largest value. |
| `table.checkall(t, fn)` | True if `fn` holds for every element. |
| `table.keys(t)` / `table.values(t)` | New array-like tables. |
| `table.countvalues(t)` | `{ [value] = count }`. |
| `table.dedup(t)` / `table.deduplicate(t)` / `table.deduped` / `table.deduplicated` | Sets repeated-value keys to nil. |
| `table.chunk(t, size)` | Array of chunks. Errors if `size <= 0`. |
| `table.invert(t)` | Swaps keys and values. |
| `table.back(t)` | Last element (`t[#t]`). |
| `table.slice(t, from, to?)` | New sequence slice. |
| `table.modget(t, i)` / `table.modset(t, i, v)` | Index with wrap-around. |
| `table.clone(t, depth?)` | Depth defaults to 100. |
| `table.isarray(t)` | True if keys are a hole-free sequence `1..size`. |
| `table.invert`, `table.values`, `table.keys` | Also available as methods. |
| `table.freeze(t)`, `table.isfrozen(t)` | Only if built with `PLUTO_ENABLE_TABLE_FREEZING`. Irreversible. |

## `io` (extended)

Pluto's io functions take UTF-8 paths on every platform.

| Function | Signature / behavior |
|---|---|
| `io.rename(a, b)`, `io.remove(p)` | Raise error on failure; no return values. |
| `io.copy(src, dst)` | Copies file (path or stream). Creates `dst`. Errors on failure. |
| `io.part(p, "parent"\|"name"?)` | Splits path. With no part, returns `parent, name`. |
| `io.isdir(p)`, `io.isfile(p)`, `io.exists(p)` | Booleans. |
| `io.makedir(p)` / `io.mkdir(p)` | Returns boolean. |
| `io.makedirs(p)` / `io.mkdirs(p)` | Creates all missing parents. |
| `io.listdir(p, recurse?)` / `io.scandir` | Array of paths. |
| `io.absolute(p, canonical?)` | Absolute path. `canonical` resolves symlinks. |
| `io.relative(p)` | Relative path from cwd. |
| `io.contents(p, data?)` | Getter (returns contents or `nil`) or setter (writes). |
| `io.filesize(p)` | Bytes. |
| `io.writetime(p, time?)` | Last-write Unix time (getter) or setter. |
| `io.currentdir(p?)` / `io.cwd` / `io.chdir` | Getter or setter for cwd. |
| `io.chmod(p?, mode?)` | Unix only. No args = availability check; path only = getter; both = setter. |
| `io.unique(name, ext)` | Returns a non-existing name like `index (2).pluto`. |
| `io.symlink(target, linkpath)` | Errors on failure. Needs admin on Windows. |

## `json`

Requires `require "json"`.

- `json.encode(v, "compact" | "pretty" | "msgpack")` (default `"compact"`). Errors on NaN/inf. Uses `__order` if present.
- `json.decode(s, flags?)`. Flags (bitwise OR): `json.withnull` (null -> `json.null` instead of `nil`), `json.withorder` (adds `__order`), `json.msgpack` (input is MessagePack).
- `json.null` sentinel.

## `base64`, `base32`

- `base64.encode(s, pad = true)`, `base64.decode(s)`.
- `base64.urlencode(s, pad = false)`, `base64.urldecode(s)`.
- `base32.encode(s, pad = true)`, `base32.decode(s)`.

## `crypto`

Requires `require "pluto:crypto"` or `require "crypto"`.

**Non-cryptographic hashes** (return integers unless noted):
`lua`, `djb2`, `sdbm`, `joaat`, `times33`, `murmur1`, `murmur2`, `murmur2a`, `murmur2neutral`, `murmur64a` (signed), `murmur64b`, `fnv1` (64-bit), `fnv1a` (64-bit), `fnv1a32`, `crc32(s, init?)`, `crc32c(s, init?)`, `adler32(s, init?)`, `lookup3(s, init?)`.

**Semi/cryptographic hashes** (return lowercase hex, or raw bytes if the second arg is `true`):
`md5(s, raw?)`, `sha1(s, raw?)`, `sha256(s, raw?)`, `sha384(s, raw?)`, `sha512(s, raw?)`, `ripemd160(s, raw?)`, `whirlpool(s, raw?)`.

- `crypto.hmac(algo, key, msg, raw?)` with `algo` in `"sha1"`, `"sha256"`, `"sha384"`, `"sha512"`.

**Random**: `crypto.random()` (full int range), `crypto.random(n)` (1..n), `crypto.random(l, u)` (l..u). Cryptographically secure.

**AES** (`key` 16/24/32 bytes; `iv` 16 bytes):
- `crypto.encrypt(data, "aes-cbc-pkcs7" | "aes-cfb-pkcs7" | "aes-cbc" | "aes-cfb", key, iv)`
- `crypto.decrypt(data, same_mode, key, iv)`. Errors on bad padding.
- ECB: `crypto.encrypt(data, "aes-ecb-pkcs7" | "aes-ecb", key)`. Weakest; avoid for new designs.
- GCM: `crypto.encrypt(data, "aes-gcm", aad, key, iv)` returns `ciphertext, tag`. `crypto.decrypt(data, "aes-gcm", aad, key, iv, tag)`. Errors on auth failure.
- Reuse of IV in CBC/CFB/GCM is insecure. Generate a random IV per message.

**RSA**:
- `crypto.generatekeypair("rsa", bits)` returns `pub {n, e}`, `priv {p, q}` (bigints). Positive bits = strict, negative = lax.
- `crypto.derive("rsa", priv)` returns the public key.
- `crypto.exportkey(priv, "pem")`, `crypto.importkey(pem, "pem")`.
- `crypto.encrypt(data, mode, key, label?)` / `crypto.decrypt(data, mode, key, label?)`. Modes: `"rsa-sha1"`, `"rsa-sha256"`, `"rsa-sha384"`, `"rsa-sha512"` (OAEP), `"rsa-pkcs1"` (v1.5), `"rsa"` (raw, expert-only).
- `crypto.sign(data, "rsa-sha256"|"rsa-sha1", priv)` and `crypto.verify(data, mode, pub, sig)` (PKCS#1 v1.5).

**Curve25519**:
- `crypto.generatekeypair("curve25519")` returns `pub, priv` (32-byte strings).
- `crypto.derive("curve25519", priv)`, `crypto.x25519(priv, peer_pub)` (shared secret).

**Compression**:
- `crypto.compress(data, "lzf")`.
- `crypto.decompress(data, [algorithm], [offset], [max_size])`. Algorithm is `"deflate"` (default) or `"lzf"`. Returns `string, info` where `info = { compressed_size, checksum_present, checksum_mismatch }`.

## `regex`

Requires `require "pluto:regex"`. Soup's regex flavor. Patterns must include delimiters: `/pattern/flags`.

- `new regex "/pat/"` (or `regex.new`). Named groups: `(?'name'...)`.
- `re:match(s)`: table with `[0]` for the whole match, `[1..n]` or named keys for groups; `nil` on no match.
- `re:search(s, init?)`: returns start and end offsets, or `nil`.
- `re:replace(s, repl)`: first match, or all with `g` flag.
- `re:substitute(s, repl)`: like `replace` but `$1`, `$2`, ... reference groups. `$$` is a literal `$`.

## `bigint`

Requires `require "pluto:bigint"`.

- `new bigint("123")`, or `bigint.new(123)`. Accepts plain Lua ints.
- Arithmetic metamethods: `+ - * / % ^ unary -`. `/` and `%` are quotient and remainder.
- `:div(b)` returns `quotient, remainder`.
- `:gcd(b)`, `:abs()`, `:bitlength()`, `:isprobableprime(iters)`.
- `:hex()`, `:binary()`, `tostring()` (decimal).
- `:export(minbytes?)` / `bigint.import(bytes)` for big-endian binary.
- Comparison: `eq`, `lt`, `le`, plus `==`, `<`, `<=`.

## `buffer`

Requires `require "pluto:buffer"`.

- `new buffer()`, `buf:append(s)`, `tostring(buf)` / `buf:tostring()`. Efficient for incremental string building.

## `scheduler`

Requires `require "pluto:scheduler"` (or `require "*"`).

- `local s = new scheduler()`.
- `s:add(fn)`: wraps `fn` in a coroutine and runs it once. Returns the coroutine.
- `s:addloop(fn)`: repeatedly calls `fn` until it returns `false`.
- `s:run()`: runs until all coroutines finish. Rethrows errors unless `errorfunc` is set.
- `s.yieldfunc`: how the scheduler waits between ticks. Default is `os.sleep(1)`. Use `coroutine.sleep` in non-blocking contexts.
- `s.errorfunc`: `|err| -> ...` receives errors from coroutines.

## `coroutine` (extended)

- `coroutine.xresume(co, ...)`: like `resume`, but rethrows errors.
- `coroutine.sleep(ms)`: yields for ms milliseconds.

## `os` (extended)

- `os.sleep(ms)`: blocks.
- `os.nanos()`, `os.micros()`, `os.millis()`, `os.seconds()`: implementation-specific epoch.
- `os.unixseconds()`: Unix seconds.
- `os.dnsresolve(type, name)`: Windows and Linux only. Returns `{ {type=..., data=...}, ... }`.
- `os.rename`, `os.remove`: Lua versions retained, UTF-8 paths.
- `os.execute`, `os.getenv`: UTF-8 on Windows.

## `math` (extended)

- `math.isnan(x)`, `math.inf` (alias of `math.huge`).
- `math.round(x)` (`round(2.4) == 2`, `round(2.5) == 3`).
- `math.trunc(x)`, `math.cbrt(x)`, `math.hypot(a, b)`.
- `math.clz(n)`, `math.ctz(n)`, `math.popcnt(n)`.
- `math.atan2` is an alias of `math.atan`.

## `exception`

- `new exception(msg)` throws immediately. Fields `.what` (message), `.where` (location). Print the exception to get `where: what`.

## `assert`

Requires `require "assert"`. Each failing assertion raises an error with a labeled message.

- `assert.isnil(v)`, `assert.notnil(v)`, `assert.istrue(v)`, `assert.isfalse(v)`, `assert.truthy(v)`, `assert.falsy(v)`.
- `assert.equal(a, b)`, `assert.nequal(a, b)`: deep comparison for tables.
- `assert.less(a, b)`, `assert.lesseq`, `assert.greater`, `assert.greatereq`.
- `assert.noerror(f, ...)`, `assert.haserror(f, ...)`, `assert.searcherror(substr, f, ...)`.
- `assert.contains(elem, container)`: uses `in` semantics.

## `http`

Requires `require "pluto:http"`. Not in plain WASM without a coroutine-aware environment.

- `http.request(url)` or `http.request({ url = ..., method = ..., headers = ..., body = ..., ... })`.
  - Options: `method` (default GET; POST if `body` is set), `headers`, `body`, `prefer_ipv6`, `dont_reuse`, `dont_make_reusable`, `require_ecdhe`.
  - Success: `body, status_code, headers, status_text`. Failure: `nil, reason`.
  - Yields inside a coroutine. Blocks otherwise.
- `http.hasconnection(partial_url)`, `http.closeconnections()` (must be in a coroutine).

## `socket`

Requires `require "pluto:socket"` (or `"*"`). Not available in WASM.

- `socket.connect(host, port, "tcp"|"udp")`: returns socket or `nil`.
- `socket.listen(port_or_"ip:port")`: returns listener or `nil`.
- `socket.bind(scheduler, port, handler)`: accept loop; `handler(sock)` runs in its own coroutine. Errors raised to caller if bind fails.
- `socket.udpserver(port)`: pseudo-socket with `recv`/`send`.
- Socket methods: `send(data)`, `recv()` (returns chunk or `nil` if closed), `peek()`, `unrecv(data)`, `starttls(...)`, `close()`, `getpeer()` (`ip, port`), `setpeer(ip, port)` (UDP), `getside()` (`"client"`/`"server"`), `istls()`, `isudp()`, `isopen()`.
- `starttls(host_or_certs, opts?)`: client uses `{ alpn = {...}, early_data = "...", require_ecdhe = true }`. Server uses `{ { chain = pem, private_key = pem }, ... }` (RSA key).
- Listener methods: `accept()`, `hasconnection()`.

## `ffi`

Requires `require "pluto:ffi"`. Not in WASM. Use only when native interop is explicitly needed.

- `ffi.open(name)`: loads DLL/SO. Methods: `wrap(ret, name, args...)`, `value(type, name)`, `cdef(c_source)`. Types: `void i8 i16 i32 i64 u8 u16 u32 u64 f32 f64 ptr str`.
- `ffi.struct(c_source)` / `ffi.cdef(c_source)` / `ffi.new(name_or_type)` / `ffi.sizeof` / `ffi.offsetof`.
- `ffi.alloc(n)`, `ffi.write(str, ud)`, `ffi.read(ud)`, `ffi.nullptr`.
- `ffi.callback(ret, args, fn)`: x86, x86_64, aarch64 only. Incompatible with W^X.

## `canvas`

Requires `require "pluto:canvas"` (or `"canvas"`).

- `canvas.new(w, h)`, `canvas.bmp(s)`, `canvas.qrcode(s, { ecl, border, bg, fg })`.
- Methods: `get(x, y)`, `set(x, y, color)`, `fill(color)`, `size()`, `mulsize(n)`, `tobmp()`, `topng()`, `tobwstring(white_color)`.

## `cat`

Requires `require "pluto:cat"`. Colons-and-Tabs format.

- `cat.encode(tbl)`: uses `__value` for values, child tables as children.
- `cat.decode(s, "full" | "flat" | "flatwithorder")`. `"full"` (default) returns a table with `__index` for nested access.

## `xml`

Requires `require "pluto:xml"`.

- `xml.encode(tbl, pretty?)`: tables with `tag`, `attributes`, `children`.
- `xml.decode(s, "xml" | "html" | "lax")`: returns tables with `tag`, `attributes`, `children`, and an `__index` for child lookup. Multiple roots become a `body` element.

## `url`

Requires `require "pluto:url"`.

- `url.encode(s)`, `url.decode(s)`, `url.parse(u)` returns `{ scheme, user, pass, host, port, path, query, fragment }`.

## `vector3`

Requires `require "pluto:vector3"`.

- `vector3()` is `(0,0,0)`, `vector3(a)` is `(a,a,a)`, `vector3(a,b)` is `(a,b,0)`, `vector3(a,b,c)` is `(a,b,c)`.
- `+ - * /` with vector or number; `==`; `#v` and `v:magnitude()`.
- `v:distance(o)`, `v:sum()`, `v:min()`, `v:max()`, `v:dot(o)`, `v:abs()`, `v:normalized()`.
- `v:torot(up)`, `v:lookat(target, up)`, `v:todir(up)`. `up` is `"y"`, `"yl"`, `"yr"` (Y-up left/right-handed), or `"z"`, `"zr"`.

## `wasm`

Requires `require "pluto:wasm"`. Not in all builds.

- `wasm.instantiate(bin, imports?)`: returns a module with `:call(name, ...)`, `:read(offset, n)`, `:write(offset, bytes)`.
- `wasm.describe(bin)`: lists imports and exports.
- `wasm.global.new(type, init)`, `wasm.table.new(type, size, max?, is64?)`, `wasm.memory.new(pages, max?, is64?)`. Types for globals: `i32 i64 f32 f64 externref`, with optional `mut` prefix.

## Integrator and sandbox configuration (`luaconf.h` or build flags)

These are the documented macros. Define them in `luaconf.h` or the build configuration.

| Macro | Effect |
|---|---|
| `PLUTO_DISABLE_COMPILED` | Disables loading compiled bytecode. |
| `PLUTO_LOADFILE_HOOK` (e.g. `ContmodOnLoadfile`) | Calls a function on each file load, e.g. to filter content. Requires static linking. |
| `PLUTO_NO_BINARIES` | Removes `package.loadlib` and DLL/SO loading from `require`. Cannot be bypassed. Recommended for sandboxes. |
| `PLUTO_NO_FILESYSTEM` | Disables filesystem access; also disables `$include`. |
| `PLUTO_ETL_ENABLE` | Enables Execution Time Limit. Default 1 ms (1,000,000 ns); override with `PLUTO_ETL_NANOS`. Blunt tool. |
| `PLUTO_ILP_ENABLE` | Enables Infinite Loop Prevention (loops must yield). Good for game threads. Not a security boundary. |
| `PLUTO_VMDUMP` | Dumps every VM instruction. Tune with `PLUTO_VMDUMP_COND`, `lua_writestring`, `lua_writeline`. |
| `PLUTO_ENABLE_TABLE_FREEZING` | Adds `table.freeze`, `table.isfrozen`, and C APIs. Call `table.freeze` before user code runs. |
| `PLUTO_MEMORY_LIMIT` | Caps memory use. |
| `PLUTO_NO_DEBUGLIB`, `PLUTO_NO_COROLIB` | Remove the debug or coroutine library. |
| `PLUTO_FORCE_JUMPTABLE` | Enables Pluto's portable jump table (may add compile time; up to ~5% VM speed). |
| `PLUTO_USE_COLORED_OUTPUT` | ANSI color in errors and warnings. |
| `PLUTO_HTTP_REQUEST_HOOK`, `PLUTO_READ_FILE_HOOK`, `PLUTO_WRITE_FILE_HOOK` | Hooks for network and file I/O. |
| `PLUTO_WARN_GLOBAL_SHADOW`, `PLUTO_WARN_NON_PORTABLE_CODE`, `PLUTO_WARN_NON_PORTABLE_NAME`, `PLUTO_WARN_NON_PORTABLE_BYTECODE` | Enable off-by-default warnings. |
| `PLUTO_COMMON_GLOBAL_NAMES` | Overrides the list of globals checked by `global-shadow`. |
| `PLUTO_PARANOID_KEYWORD_DETECTION` | More aggressive keyword-disable inference. |
| `PLUTO_USE_LET` | Re-enables the `let` alias. |
| `PLUTO_LUA_LINKABLE` | Build variant for linking with Lua. |

Sandbox checklist for embedding Pluto with untrusted code:
1. `PLUTO_NO_BINARIES` and `PLUTO_NO_FILESYSTEM`.
2. `PLUTO_DISABLE_COMPILED` if untrusted bytecode must not run.
3. `PLUTO_ETL_ENABLE` for hard time limits, and/or `PLUTO_ILP_ENABLE` for cooperative yielding.
4. `PLUTO_LOADFILE_HOOK` for content moderation of `require`/`dofile`.
5. Freeze shared tables with `table.freeze` (requires `PLUTO_ENABLE_TABLE_FREEZING`) before exposing the environment.
6. Remove `debug` and `coroutine` libraries if not needed.
7. Don't rely on ILP as a security boundary; it is documented as bypassable.

## Bindings

| Target | Project |
|---|---|
| JavaScript (libpluto-js) | `libpluto-js`: `cwrap`s and Emscripten module access |
| HTML & JS | `PlutoScript`: `<script type="pluto">` and high-level APIs |
| JavaScript (wasm builds) | Various Lua and Pluto Emscripten modules |
| Apache2 (PHP) | `pluto-backend`: run Pluto scripts from Apache2 |

## Editor integration

- VS Code: `calamity-inc.pluto-syntax-highlighting`
- Sublime Text: Package Control, "Pluto"
- Visual Studio: `calamity-inc.pluto-vsix`
- IntelliJ: TextMate bundle from `PlutoLang/Syntax-Highlighting`

Build systems in editors: Ctrl+B (Sublime) or Ctrl+Shift+P (VS Code) to run and lint.
