# fastify-basic-auth

[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](http://standardjs.com/)
 [![Build Status](https://travis-ci.org/fastify/fastify-basic-auth.svg?branch=master)](https://travis-ci.org/fastify/fastify-basic-auth)

 A simple basic auth plugin for Fastify.

 ## Install
```
npm i fastify-basic-auth
```
## Usage
This plugin decorates the fastify instance with a `basicAuth` function, which you can use inside a `preHandler` hook, in a `beforeHander` or with [`fastify-auth`](https://github.com/fastify/fastify-auth).

```js
const fastify = require('fastify')()
const authenticate = {realm: 'Westeros'}
fastify.register(require('fastify-basic-auth'), { validate, authenticate })
// `this` inside validate is `fastify`
function validate (username, password, req, reply, done) {
  if (username === 'Tyrion' && password === 'wine') {
    done()
  } else {
    done(new Error('Winter is coming'))
  }
}

fastify.after(() => {
  fastify.addHook('preHandler', fastify.basicAuth)

  fastify.get('/', (req, reply) => {
    reply.send({ hello: 'world' })
  })
})
```

Promises and *async/await* are supported as well!
```js
const fastify = require('fastify')()
const authenticate = {realm: 'Westeros'}
fastify.register(require('fastify-basic-auth'), { validate, authenticate })
async function validate (username, password, req, reply) {
  if (username !== 'Tyrion' || password !== 'wine') {
    return new Error('Winter is coming')
  }
}
```

Use with `beforeHandler`:
```js
const fastify = require('fastify')()
const authenticate = {realm: 'Westeros'}
fastify.register(require('fastify-basic-auth'), { validate, authenticate, disableHook: true })
async function validate (username, password, req, reply) {
  if (username !== 'Tyrion' || password !== 'wine') {
    return new Error('Winter is coming')
  }
}

fastify.after(() => {
  fastify.route({
    method: 'GET',
    url: '/',
    beforeHandler: fastify.basicAuth,
    handler: async (req, reply) => {
      return { hello: 'world' }
    }
  })
})
```

Use with [`fastify-auth`](https://github.com/fastify/fastify-auth):
```js
const fastify = require('fastify')()
const authenticate = {realm: 'Westeros'}
fastify.register(require('fastify-auth'))
fastify.register(require('fastify-basic-auth'), { validate, authenticate, disableHook: true })
async function validate (username, password, req, reply) {
  if (username !== 'Tyrion' || password !== 'wine') {
    return new Error('Winter is coming')
  }
}

fastify.after(() => {
  // use preHandler to authenticate all the routes
  fastify.addHook('preHandler', fastify.auth([fastify.basicAuth]))

  fastify.route({
    method: 'GET',
    url: '/',
    // use beforeHanderto authenticatejust this one
    beforeHandler: fastify.auth([fastify.basicAuth]),
    handler: async (req, reply) => {
      return { hello: 'world' }
    }
  })
})
```

## Options

### `validate` <Function> (required)

The `validate` function is called on each request made, 
and is passed the `username`, `password`, `req` and `reply` 
parameters in that order. An optional fifth parameter, `done` may be
used to signify a valid request when called with no arguments, 
or an invalid request when called with an `Error` object. Alternatively, 
the `validate` function may return a promise, resolving for valid 
requests and rejecting for invalid. This can also be achieved using
an `async/await` function, and throwing for invalid requests.

See code above for examples.

### `authenticate` <Boolean|Object> (optional, default: false)

When supplied, the `authenticate` option will cause the 
[`WWW-Authenticate` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate) to be added. It may also be used to set the `realm` value.

This can be useful in situations where we want to trigger client-side authentication interfaces - for instance the browser authentication dialog.

As a boolean setting `authenticate` to `true` will set a header like so: `WWW-Authenticate: Basic`. When `false`, no header is added. This is the default.

```js
fastify.register(require('fastify-basic-auth'), { 
  validate, 
  authenticate: true // WWW-Authenticate: Basic
})

fastify.register(require('fastify-basic-auth'), { 
  validate, 
  authenticate: false // no authenticate header, same as omitting authenticate option
})
```

When supplied as an object the `authenticate` option may have a `realm` key.

If the `realm` key is supplied, it will be appended to the header value:

```js
fastify.register(require('fastify-basic-auth'), { 
  validate, 
  authenticate: {realm: 'example'} // WWW-Authenticate: Basic realm="example"
})
```


## License

Licensed under [MIT](./LICENSE).
