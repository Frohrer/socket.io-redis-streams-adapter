# Socket.IO Redis Streams adapter

The `@socket.io/redis-streams-adapter` package allows broadcasting packets between multiple Socket.IO servers.

Supported features:

- [broadcasting](https://socket.io/docs/v4/broadcasting-events/)
- [utility methods](https://socket.io/docs/v4/server-instance/#Utility-methods)
  - [`socketsJoin`](https://socket.io/docs/v4/server-instance/#socketsJoin)
  - [`socketsLeave`](https://socket.io/docs/v4/server-instance/#socketsLeave)
  - [`disconnectSockets`](https://socket.io/docs/v4/server-instance/#disconnectSockets)
  - [`fetchSockets`](https://socket.io/docs/v4/server-instance/#fetchSockets)
  - [`serverSideEmit`](https://socket.io/docs/v4/server-instance/#serverSideEmit)
- [connection state recovery](https://socket.io/docs/v4/connection-state-recovery)

Related packages:

- Redis adapter: https://github.com/socketio/socket.io-redis-adapter/
- Redis emitter: https://github.com/socketio/socket.io-redis-emitter/
- MongoDB adapter: https://github.com/socketio/socket.io-mongo-adapter/
- MongoDB emitter: https://github.com/socketio/socket.io-mongo-emitter/
- Postgres adapter: https://github.com/socketio/socket.io-postgres-adapter/
- Postgres emitter: https://github.com/socketio/socket.io-postgres-emitter/

**Table of contents**

- [Installation](#installation)
- [Usage](#usage)
  - [With the `redis` package](#with-the-redis-package)
  - [With the `redis` package and a Redis cluster](#with-the-redis-package-and-a-redis-cluster)
  - [With the `ioredis` package](#with-the-ioredis-package)
  - [With the `ioredis` package and a Redis cluster](#with-the-ioredis-package-and-a-redis-cluster)
- [Options](#options)
- [How it works](#how-it-works)
- [License](#license)

## Installation

```
npm install @socket.io/redis-streams-adapter redis
```

## Usage

### With the `redis` package

```js
import { createClient } from "redis";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-streams-adapter";

const redisClient = createClient({ url: "redis://localhost:6379" });

await redisClient.connect();

const io = new Server({
  adapter: createAdapter(redisClient)
});

io.listen(3000);
```

### With the `redis` package and a Redis cluster

```js
import { createCluster } from "redis";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-streams-adapter";

const redisClient = createCluster({
  rootNodes: [
    {
      url: "redis://localhost:7000",
    },
    {
      url: "redis://localhost:7001",
    },
    {
      url: "redis://localhost:7002",
    },
  ],
});

await redisClient.connect();

const io = new Server({
  adapter: createAdapter(redisClient)
});

io.listen(3000);
```

### With the `ioredis` package

```js
import { Redis } from "ioredis";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-streams-adapter";

const redisClient = new Redis();

const io = new Server({
  adapter: createAdapter(redisClient)
});

io.listen(3000);
```

### With the `ioredis` package and a Redis cluster

```js
import { Cluster } from "ioredis";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-streams-adapter";

const redisClient = new Cluster([
  {
    host: "localhost",
    port: 7000,
  },
  {
    host: "localhost",
    port: 7001,
  },
  {
    host: "localhost",
    port: 7002,
  },
]);

const io = new Server({
  adapter: createAdapter(redisClient)
});

io.listen(3000);
```

## Options

| Name                | Description                                                                                                       | Default value  |
|---------------------|-------------------------------------------------------------------------------------------------------------------|----------------|
| `streamName`        | The name of the Redis stream.                                                                                     | `socket.io`    |
| `maxLen`            | The maximum size of the stream. Almost exact trimming (~) is used.                                                | `10_000`       |
| `readCount`         | The number of elements to fetch per XREAD call.                                                                   | `100`          |
| `sessionKeyPrefix`  | The prefix of the key used to store the Socket.IO session, when the connection state recovery feature is enabled. | `sio:session:` |
| `heartbeatInterval` | The number of ms between two heartbeats.                                                                          | `5_000`        |
| `heartbeatTimeout`  | The number of ms without heartbeat before we consider a node down.                                                | `10_000`       |

## How it works

The adapter will use a [Redis stream](https://redis.io/docs/data-types/streams/) to forward events between the Socket.IO servers.

Notes:

- a single stream is used for all namespaces
- the `maxLen` option allows to limit the size of the stream
- unlike the adapter based on Redis PUB/SUB mechanism, this adapter will properly handle any temporary disconnection to the Redis server and resume the stream
- if [connection state recovery](https://socket.io/docs/v4/connection-state-recovery) is enabled, the sessions will be stored in Redis as a classic key/value pair

## License

[MIT](LICENSE)
