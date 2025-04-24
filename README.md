# Lua WASM Bindings

WASM bindings and binaries for 5.4.6

## Building

* Install `emscripten` using your native package manager (eg `pacman`, `apt`, `brew`, etc ...)
    * https://emscripten.org/index.html
* Run `./scripts/setup.sh`
* Run `npm run build` (or whatever JS build system you use)

> [!IMPORTANT]  
> The binding script doesn't include the entire Lua API, some effort might be needed to add missing features.
> For the complete Lua API reference, visit https://www.lua.org/manual/5.4/manual.html

## Example

```ts
import { LUA_OK } from "lua-wasm-bindings/dist/lua";

import { lauxlib, lua, lualib } from "lua-wasm-bindings/dist/lua.54";

const luaCode = `return "Hello"`;
consol.log(executeLua(luaCode));

function executeLua(luaCode: string): string | Error | never {
  const L = lauxlib.luaL_newstate();
  lualib.luaL_openlibs(L);

  // Optional Load modules
  // lua.lua_getglobal(L, "package");
  // lua.lua_getfield(L, -1, "preload");
  // lauxlib.luaL_loadstring(L, jsonLib); // Load extenal package from string
  // lua.lua_setfield(L, -2, "json");

  const status = lauxlib.luaL_dostring(L, luaCode);

  if (status === LUA_OK) {
    if (lua.lua_isstring(L, -1)) {
      const result = lua.lua_tostring(L, -1);
      lua.lua_close(L);
      return result === null ? undefined : result;
    } else {
      const returnType = lua.lua_typename(L, lua.lua_type(L, -1));
      lua.lua_close(L);
      throw new Error(`Unsupported Lua return type: ${returnType}`);
    }
  } else {
    const luaStackString = lua.lua_tostring(L, -1);
    const message = luaStackString.replace(/^\[string "(--)?\.\.\."\]:\d+: /, "");
    lua.lua_close(L);
    return new Error(message);
  }
}
```
