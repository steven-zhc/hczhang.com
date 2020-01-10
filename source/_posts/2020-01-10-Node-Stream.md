---
title: Node Stream
date: 2020-01-10 13:22:03
tags: [node, stream]
---

Take note from [node-stream](https://jscomplete.com/learn/node-beyond-basics/node-streams) More information, read source.

<!--more-->

# Overall

- A readable stream is an abstraction for a source from which data can be consumed. An example of that is the fs.createReadStream method.
- A writable stream is an abstraction for a destination to which data can be written. An example of that is the fs.createWriteStream method.
- A duplex stream is both Readable and Writable. An example of that is a TCP socket.
- A transform stream is basically a duplex stream that can be used to modify or transform the data as it is written and read. An example of that is the zlib.createGzip stream to compress the data using gzip. You can think of a transform stream as a function where the input is the writable stream part and the output is readable stream part. You might also hear transform streams referred to as “through streams.”

All streams are instances of EventEmitter. They emit events that can be used to read and write data. 

# Pipe

> readableSrc.pipe(writableDest)

# Stream Event

| | Readable Streams | Writable Streams  |
| ------------- |:-------------|:-----|
| Events      | data, end, error, close, readable | drain, finish, error, close, pipe, unpipe |
| Methods     | pipe(), unpipe(), wrap(), destroy() | write(), destroy(), end()|
| | read(), unshift(), resume(), pause(), isPaused(), setEncoding() | cork(), uncork(), setDefaultEncoding() |

## Readable Stream Event

- The data event, which is emitted whenever the stream passes a chunk of data to the consumer
- The end event, which is emitted when there is no more data to be consumed from the stream.

## Writable Stream Event

- The drain event, which is a signal that the writable stream can receive more data.
- The finish event, which is emitted when all data has been flushed to the underlying system.

# Transform Stream

```javascript
const { Transform } = require("stream");

const upperCaseTr = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

process.stdin.pipe(upperCaseTr).pipe(process.stdout);
```