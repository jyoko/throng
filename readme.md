# Throng

Dead-simple one-liner for clustered apps.

```js
throng(start, { workers: 3 });

function start() {
  console.log('Started worker');

  process.on('SIGTERM', function() {
    console.log('Worker exiting');
    process.exit();
  });
}
```

## Installation

```
npm install --save throng
```

## Use

```js
throng(startFunction, options);
```

## Options

#### workers

Number of Cluster workers to create.
Defaults to number of CPUs available.

#### lifetime

Minimum time to keep the Cluster alive
(by forking new workers if any die).

In milliseconds; defaults to zero.

(Infinity = stay up forever)

#### grace

Grace period for worker shutdown.
Once each worker is sent SIGTERM, the grace period starts.
Any workers still alive when it ends are killed.
Parent exits after killing workers.

In milliseconds; defaults to 5000.

### messenger

Handler for message passing to master.
Listener created for every worker when forked.
Cluster property `isMaster` (Boolean) and `workers` (object) exposed as properties on `throng`.

## Example

This is how you might use throng in a web server:

```js
var throng = require('throng');

throng(start, {
  workers: 4,
  lifetime: Infinity,
  grace: 4000
});
```

Or using message passing:

```js
var throng = require('throng');

if ( throng.isMaster ) {
  
  function handleMsg(msg) {
    console.log('Received from worker: '+msg.txt);
    throng.workers[msg.from].send({txt: 'Get back to work'});
  }

}

throng(start, { workers: 3, messenger: handleMsg });

function start(workerID) {
  console.log('Started worker');

  process.on('message', function(msg) {
    console.log('Received from master: '+msg.txt);
  });

  process.on('SIGTERM', function() {
    console.log('Worker exiting');
    process.exit();
  });

  process.send({txt: 'Hi parent',from: workerID});
}
```

## Tests

```
npm test
```

