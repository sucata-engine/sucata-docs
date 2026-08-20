# sucata.system

Run OS shell commands from Lua.

**Security note:** the command string runs through the OS shell (`sh -c` on Linux/macOS, `cmd /C` on Windows), with the same privileges as the Sucata Player process itself — pipes, redirection, and chaining (`&&`, `|`, ...) all work, but so does anything else the shell allows. Never build a command string by concatenating untrusted input (player-typed text, downloaded data, etc.) into it.

**Blocking:** `execute` is synchronous — it blocks the calling frame until the process exits. Long-running commands will stall the game; keep them short or run them from `init`/a background trigger rather than every `tick`.

---

## sucata.system.execute

Runs a shell command and waits for it to finish, capturing its output.

**parameters**

- command `string` - The shell command line to run

**return**

- exit_code `number` - The process exit code. `127` is the shell's own "command not found" code; `-1` means the process couldn't be started at all (e.g. no shell available on the system)
- stdout `string` - Captured standard output
- stderr `string` - Captured standard error (or the OS error message, when `exit_code` is `-1`)

**example**

```lua
local exit_code, stdout, stderr = sucata.system.execute("echo hello")
if exit_code == 0 then
  print(stdout) -- "hello\n"
else
  print("command failed:", stderr)
end
```
