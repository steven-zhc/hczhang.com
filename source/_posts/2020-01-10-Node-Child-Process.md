---
title: Node Child Process
date: 2020-01-10 13:53:29
tags: [fork, node]
---

Take note from [node-child process](https://jscomplete.com/learn/node-beyond-basics/child-processes) More information, read source.

4 ways to create a child process

- spawn()
- fork()
- exec()
- execFile()

<!--more-->

## 1 Spawned Child Processes

```javascript
const { spawn } = require("child_process")
const child = spawn("pwd")

child.on('exit', (code, signal) => {
  console.log(`child process exited with code ${code} and signal ${signal}`)
})
```

### Child events

- The *`disconnect`* event is emitted when the parent process manually calls the child.disconnect method.
- The *`error`* event is emitted if the process could not be spawned or killed.
- The *`close`* event is emitted when the stdio streams of a child process get closed.
- The *`message`* event is the most important one. It’s emitted when the child process uses the `process.send()` function to send messages. This is how parent/child processes can communicate with each other.

### Child stdio

- child.stdin
- child.stdout
- child.stderr

```javascript
const { spawn } = require("child_process");

const child = spawn("wc");

process.stdin.pipe(child.stdin);

child.stdout.on("data", data => {
  console.log(`child stdout:\n${data}`)
})
```

```javascript
const { spawn } = require("child_process");

const find = spawn("find", [".", "-type", "f"]);
const wc = spawn("wc", ["-l"]);

find.stdout.pipe(wc.stdin);

wc.stdout.on("data", data => {
  console.log(`Number of files ${data}`);
});
```

### spawn options

```javascript
const child = spawn("find . -type f | wc -l", {
  stdio: "inherit",
  shell: true,
  cwd: '/usr/local',
  env: { USER: 'STEVEN' },
})

child.unref()
```

#### stdio: 'inherit'

when we execute the code, the child process inherits the main process `stdin`, `stdout`, and `stderr`. This causes the child process data events handlers to be triggered on the main `process.stdout` stream, making the script output the result right away.

#### shell: true

we were able to use the shell syntax in the executed command, just like we did with `exec`.

#### cwd

we can use the cwd option to change the working directory of the script

#### env

we can use is env to specify the environment variables that will be visible to the new child process. The default for this option is process.env which gives any command access to the current process environment. If we want to override that behavior, we can simply pass an empty object as the env option or new values there to be considered as the only environment variables

#### detached

The exact behavior of detached child processes depends on the OS. On Windows, the detached child process will have its own console window while on Linux the detached child process will be made the leader of a new process group and session.

```javascript
// parent.js
const { spawn } = require('child_process');

const child = spawn('node', ['timer.js'], {
  detached: true,
  stdio: 'ignore'
});

child.unref();

// timer.js
setTimeout(() => {  
  // keep the event loop busy
}, 20000);
```

## 2 Shell Syntax and the exec Function

```javascript
const { exec } = require("child_process");

exec("find . -type f | wc -l", (err, stdout, stderr) => {
  if (err) {
    console.error(`exec error: ${err}`);
    return;
  }

  console.log(`Number of files ${stdout}`);
});
```

## 3 execFile Function

execute a file without using a shell, the execFile. It behaves exactly like the exec function, but does not use a shell, which makes it a bit more efficient


## 4 The fork Function

```javascript
// parent.js

const { fork } = require("child_process");

const forked = fork("child.js");

forked.on("message", msg => { console.log("Message from child", msg) })

forked.send({ hello: "world" })

// child.js

process.on("message", msg => { console.log("Message from parent:", msg) })

let counter = 0
setInterval(() => {
  //send back to the parent process
  process.send({ counter: counter++ })
}, 1000)
```
