# Apollo Cache Control

Apollo Cache Control is a GraphQL extension for fine-grained cache control that can inform server-side or client-side GraphQL caches. It describes a format for a GraphQL API to return information about cache expiration and scope, as well as controls that a client can use to override caching.

It’s already supported by [Apollo Engine](https://www.apollographql.com/engine/), and we’re excited to see what other kinds of integrations people can build on top of this format.

## Supported GraphQL Servers

- [Node.js](https://github.com/apollographql/apollo-cache-control-js)

If you’re interested in working on support for other GraphQL servers, please get in touch on the `#apollo-cache-control` channel on the [Apollo Slack](http://www.apollodata.com/#slack).

## Response Format

The GraphQL specification allows servers to [include additional information as part of the response under an `extensions` key](https://facebook.github.io/graphql/#sec-Response-Format):
> The response map may also contain an entry with key `extensions`. This entry, if set, must have a map as its value. This entry is reserved for implementors to extend the protocol however they see fit, and hence there are no additional restrictions on its contents.

Apollo Cache Control exposes cache control hints for an individual request under a `cacheControl` key in `extensions`:

```
{
  "data": ...,
  "errors": ...,
  "extensions": {
    "cacheControl": {
      "version": 1,
      "hints: [
        {
          "path": [...],
          "maxAge": <seconds>,
          "scope": <PUBLIC or PRIVATE>
        },
        ...
      ]
    }
  }
}
```

- The `path` is the response path in a format similar to the error result format specified in the GraphQL specification:
> This field should be a list of path segments starting at the root of the response and ending with the field associated with the error. Path segments that represent fields should be strings, and path segments that represent list indices should be 0‐indexed integers. If the error happens in an aliased field, the path to the error should use the aliased name, since it represents a path in the response, not in the query.

- `maxAge` indicates that anything under this path shouldn't be cached for more than the specified number of seconds, unless the value is overridden on a subpath.

- If `scope` is set to `PRIVATE`, that indicates anything under this path should only be cached per-user, unless the value is overridden on a subpath. `PUBLIC` is the default and means anything under this path can be stored in a shared cache.

### Example

```graphql
query {
  post(id: 1) {
    title
    votes
    readByCurrentUser
  }
}
```

```json
"cacheControl": {
  "version": 1,
  "hints": [
    {
      "path": [
        "post"
      ],
      "maxAge": 240
    },
    {
      "path": [
        "post",
        "votes"
      ],
      "maxAge": 30
    },
    {
      "path": [
        "post",
        "readByCurrentUser"
      ],
      "scope": "PRIVATE"
    }
  ]
}
```

## Request Format

Apollo Cache Control also allows clients to include cache control instructions in a request. For now, the only specified field is `noCache`, which forces the proxy never to return a cached response, but always fetch the query from the origin.

```json
"extensions": {
  "cacheControl": {
    "version": 1,
    "noCache": true
  }
}
```
