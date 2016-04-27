auca-judge-agent
================

*auca-judge-agent* is a remote control agent for build and test execution inside
containers for the *auca-judge* system.

*auca-judge-agent* allows to move the build and test container runtime to a
separate host from the *auca-judge* back end.

## Configuration

*auca-judge-agent* searches for configuration files in the JSON format in the
following locations

1. */etc/auca-judge-agent-configuration.json*
2. *~/.auca-judge-agent-configuration.json*
3. *./auca-judge-agent-configuration.json*
4. file specified in the `AUCA_JUDGE_AGENT_CONFIGURATION` environment variable

Multiple configuration files are merged together. Values in files with higher
priority override values from files with lower priority. For example, fields
from *./auca-judge-agent-configuration.json* will override values from
*/etc/auca-judge-agent-configuration.json*.

Every configuration option in configuration files can be overriden by a command
line flag with the same name.

### Configuration Format

```json
{
  "option": "value",
  "option": ["value", "value"],
  "option": 42
}
```

### Configuration Options

* `"port": number` (*--port=number*)

  a port to bind to (set to 8080 by default)

* `"command": string` (*--command=string*)

  a command to start

  The agent searches for an executable in all locations specified by the `PATH`
  environment variable. If the command contains a slash, it is tried directly
  and the search through `PATH` is not performed.

* `"arguments": [string, ...]` (*--arguments=string,string,...*)

  a list of arguments to the command

* `"shell": string` (*--shell=string*)

  a command with all its parameters to be run in a shell with `/bin/sh -c`

  `shell` overrides the `command` and `arguments` options.

* `"workingDirectory": string` (*--workingDirectory=string*)

  a path to a directory to switch to before starting a command or shell (set to
  the process's current working directory `"."` by default)

* `"artifacts": [string, ...]` (*--artifacts=string,string,...*)

  a list of generated files to send back

### Sample Configuration Files

```json
{
  "port": 7070,
  "command": "gcc",
  "arguments": ["-o", "submission", "submission.c"],
  "workingDirectory": ".",
  "artifacts": ["submission"]
}
```

```json
{
  "shell": "./compile.sh",
  "workingDirectory": ".",
  "artifacts": ["submission"]
}
```

```json
{
  "shell": "./submission < input > output",
  "artifacts": ["output"]
}
```

## Communication

*auca-judge-agent* responds to the following HTTP request

**POST** */process*

```json
{
  "files": {
    "file name": "file data as a Base64 string",
    "file name": "file data as a Base64 string"
  }
}
```

For the `/process` request *auca-judge-agent* saves every file to the current
working directory, starts a command or shell specified in the configuration,
waits for it to exit, and returns the exit status, captured output streams, and
all generated artifact files specified in the configuration.

```json
{
  "status": 0,
  "stdout": "standard output",
  "stderr": "standard error",
  "artifacts": {
    "file name": "file data as a Base64 string",
    "file name": "file data as a Base64 string",
    "file name": null
  }
}
```

## Licensing

*auca-judge-agent* is licensed under the MIT license. See LICENSE for the full
license text.

## Credits

*auca-judge-agent* was created by [Dmitrii Toksaitov](https://github.com/toksaitov).
