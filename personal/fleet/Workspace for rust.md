## Options
GDB with rust-gdb and lldb with rust-lldb as wrappers for rust
the wrapper has pretty print functionality

RR is also another option: https://github.com/rr-debugger/rr/wiki/Usage

GDB support printing object using commpiled function: https://www.reddit.com/r/rust/comments/6q2j6q/can_you_call_debug_print_using_lldb/
### LLDB in vscode
Create task to build artifact
```
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "shell",
			"command": "$(cd datafusion-cli && cargo build)",
			"problemMatcher": [
				"$rustc"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"label": "Build datafusion-cli"
		}
	]
}
```
And launch.json
```
        {
            "name": "Launch",
            "type": "lldb",
            "request": "launch",
            "program": "${workspaceRoot}/datafusion-cli/target/debug/datafusion-cli",
            "args": [],
            "cwd": "${workspaceRoot}",
        }

```
Can do the same in replay mode (cargo rr), but requires more work
TODO

GDBGui also help pretty print some variable such as enum

TODO items in rust:
Debug trait object: https://github.com/rust-lang/rust/issues/1563

tags: #rust #debug