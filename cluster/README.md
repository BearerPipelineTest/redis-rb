# Redis::Cluster

## Getting started

Install with:

```
$ gem install redis-clustering
```

You can connect to Redis by instantiating the `Redis::Cluster` class:

```ruby
require "redis-clustering"

redis = Redis::Cluster.new(nodes: (7000..7005).map { |port| "redis://127.0.0.1:#{port}" })
```

NB: Both `redis_cluster` and `redis-cluster` are unrelated and abandoned gems.

```ruby
# Nodes can be passed to the client as an array of connection URLs.
nodes = (7000..7005).map { |port| "redis://127.0.0.1:#{port}" }
redis = Redis::Cluster.new(nodes: nodes)

# You can also specify the options as a Hash. The options are the same as for a single server connection.
(7000..7005).map { |port| { host: '127.0.0.1', port: port } }
```

You can also specify only a subset of the nodes, and the client will discover the missing ones using the [CLUSTER NODES](https://redis.io/commands/cluster-nodes) command.

```ruby
Redis::Cluster.new(nodes: %w[redis://127.0.0.1:7000])
```

If you want [the connection to be able to read from any replica](https://redis.io/commands/readonly), you must pass the `replica: true`. Note that this connection won't be usable to write keys.

```ruby
Redis::Cluster.new(nodes: nodes, replica: true)
```

The calling code is responsible for [avoiding cross slot commands](https://redis.io/topics/cluster-spec#keys-distribution-model).

```ruby
redis = Redis::Cluster.new(nodes: %w[redis://127.0.0.1:7000])

redis.mget('key1', 'key2')
#=> Redis::CommandError (CROSSSLOT Keys in request don't hash to the same slot)

redis.mget('{key}1', '{key}2')
#=> [nil, nil]
```

* The client automatically reconnects after a failover occurred, but the caller is responsible for handling errors while it is happening.
* The client support permanent node failures, and will reroute requests to promoted slaves.
* The client supports `MOVED` and `ASK` redirections transparently.

## Cluster mode with SSL/TLS
Since Redis can return FQDN of nodes in reply to client since `7.*` with CLUSTER commands, we can use cluster feature with SSL/TLS connection like this:

```ruby
Redis.new(cluster: %w[rediss://foo.example.com:6379])
```

On the other hand, in Redis versions prior to `6.*`, you can specify options like the following if cluster mode is enabled and client has to connect to nodes via single endpoint with SSL/TLS.

```ruby
Redis.new(cluster: %w[rediss://foo-endpoint.example.com:6379], fixed_hostname: 'foo-endpoint.example.com')
```

In case of the above architecture, if you don't pass the `fixed_hostname` option to the client and servers return IP addresses of nodes, the client may fail to verify certificates.
