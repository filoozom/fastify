<h1 align="center">Fastify</h1>

## Logging

Logging is disabled by default, and you can enable it by passing
`{ logger: true }` or `{ logger: { level: 'info' } }` when you create
the fastify instance. Note that if the logger is disabled, it is impossible to
enable it at runtime. We use
[abstract-logging](https://www.npmjs.com/package/abstract-logging) for
this purpose.

Since Fastify is really focused on performances, it uses [pino](https://github.com/pinojs/pino) as its logger, with the default log level, when enabled, set to `'info'`.

Enabling the logger is extremely easy:

```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

If you want to pass some options to the logger, just pass the logger option to Fastify.
You can find all the options in the [Pino documentation](https://github.com/pinojs/pino/blob/master/docs/api.md#pinooptions-stream). If you want to specify a file destination, use:

```js
const fastify = require('fastify')({
  logger: {
    level: 'info',
    file: '/path/to/file' // will use pino.destination()
  }
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

If you want to pass a custom stream to the Pino instance, just add the stream field to the logger object.

```js
const split = require('split2')
const stream = split(JSON.parse)

const fastify = require('fastify')({
  logger: {
    level: 'info',
    stream: stream
  }
})
```

<a name="logging-request-id" />

By default fastify adds an id to every request for easier tracking. If the "request-id" header is present its value is used, otherwise a new incremental id is generated. See Fastify Factory [`requestIdHeader`](https://github.com/fastify/fastify/blob/master/docs/Server.md#factory-request-id-header) and Fastify Factory [`genReqId`](https://github.com/fastify/fastify/blob/master/docs/Server.md#gen-request-id) for customization options.

The default logger is configured with a set of standard serializers that serialize objects with `req`, `res`, and `err` properties. This behavior can be customized by specifying custom serializers.
```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      req: function (req) {
        return { url: req.url }
      }
    }
  }
})
```

*This option will be ignored by any logger other than Pino.*

You can also supply your own logger instance. Instead of passing configuration options, simply pass the instance.
The logger you supply must conform to the Pino interface; that is, it must have the following methods:
`info`, `error`, `debug`, `fatal`, `warn`, `trace`, `child`.

Example:

```js
const log = require('pino')({ level: 'info' })
const fastify = require('fastify')({ logger: log })

log.info('does not have request information')

fastify.get('/', function (request, reply) {
  request.log.info('includes request information, but is the same logger instance as `log`')
  reply.send({ hello: 'world' })
})
```

*The logger instance for the current request is available in every part of the [lifecycle](https://github.com/fastify/fastify/blob/master/docs/Lifecycle.md).*

## Log Redaction

[Pino](https://getpino.io) supports low-overhead log redaction for
obscuring values of specific properties in recorded logs.
As an example, we might want to log all the HTTP headers minus the
`Authorization` header for security concerns:

```js
const fastify = Fastify({
  logger: {
    stream: stream,
    redact: ['req.headers.authorization'],
    level: 'info',
    serializers: {
      req (req) {
        return {
          method: req.method,
          url: req.url,
          headers: req.headers,
          hostname: req.hostname,
          remoteAddress: req.ip,
          remotePort: req.connection.remotePort
        }
      }
    }
  }
})
```

See https://getpino.io/#/docs/redaction for more details.
