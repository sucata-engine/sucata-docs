# sucata.dynlib

Load native shared libraries (`.dll`/`.so`/`.dylib`) at runtime and call functions exported by them.

This is a low-level API meant for native plugins, not regular game code. Exported functions must
be written against the Lua 5.4 C API (`lua.h`), with the exact signature `int (*)(lua_State *L)` —
the same signature every `sucata.*` function uses internally. When called, the plugin function
receives the real, live Lua state: read arguments and push return values using the Lua C API
(`lua_tonumber`, `lua_tostring`, `lua_pushnumber`, `luaL_error`, ...), exactly like a standard Lua C
module (this mirrors what `package.loadlib` does in stock Lua).

**Important:** the plugin must **not** statically link its own copy of the Lua library. It should
only include the Lua headers and leave the `lua_*`/`luaL_*` symbols unresolved at link time — the
Sucata Player process exports them at runtime, so `dlopen`/`LoadLibrary` resolves the plugin's calls
against the *same* Lua instance the engine is already running. Linking a second, separate copy of
Lua into the plugin will desync the interpreter state and can crash the engine, especially across
`luaL_error`/`lua_error`.

Minimal example plugin (`plugin.c`, compiled with `gcc -shared -fPIC -o plugin.so plugin.c`, no
`-llua`):

```c
#include "lua.h"
#include "lauxlib.h"

int add(lua_State *L) {
    lua_Number a = lua_tonumber(L, 1);
    lua_Number b = lua_tonumber(L, 2);
    lua_pushnumber(L, a + b);
    return 1;
}
```

```lua
local handle, err = sucata.dynlib.load("plugin.so")
if not handle then
    error(err)
end

local sum = sucata.dynlib.call(handle, "add", 3, 4)
print(sum) -- 7

sucata.dynlib.unload(handle)
```

---

## sucata.dynlib.load

Loads a native shared library from disk and returns a handle to it. Accepts `src://`, `data://`,
`build://` virtual paths (resolved the same way as `sucata.filesystem`) as well as plain relative
or absolute paths. If the path has no extension, the platform's native extension
(`.dll`/`.so`/`.dylib`) is tried as a fallback.

**parameters**

- path `string` - Path to the shared library

**return**

- handle `number|nil` - A handle to the loaded library, or `nil` on failure
- error `string?` - An error message, present only when `handle` is `nil`

---

## sucata.dynlib.call

Looks up an exported symbol in a loaded library and calls it directly as a Lua C function, forwarding
every extra argument to it on the Lua stack. Returns whatever the native function pushes.

Raises a Lua error if the handle is invalid or the symbol isn't found. If the native function itself
raises a Lua error (`luaL_error`/`lua_error`), it propagates like any other Lua error and can be
caught with `pcall`.

**parameters**

- handle `number` - A handle returned by `sucata.dynlib.load`
- function_name `string` - The name of the exported function to call
- ... `any` - Arguments forwarded to the native function

**return**

- ... `any` - Whatever the native function returns

---

## sucata.dynlib.unload

Unloads a previously loaded library and invalidates its handle.

**parameters**

- handle `number` - A handle returned by `sucata.dynlib.load`

**return**

- success `boolean` - Whether the library was unloaded
