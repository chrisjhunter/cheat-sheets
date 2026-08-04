# HashiCorp Consul Cheat Sheet

## Cluster status
```bash
consul members                   # cluster member list
consul members -detailed
consul info                      # local agent stats/info
consul operator raft list-peers  # raft peers (servers only)
consul catalog services          # all registered services
consul catalog nodes             # all nodes in catalog
```

## Agent
```bash
consul agent -dev                       # quick local dev agent (single node, in-memory)
consul agent -config-dir=/etc/consul.d  # start with config files
consul reload                           # reload config without restart
consul leave                            # gracefully leave the cluster
consul validate /etc/consul.d           # validate config files
```

## Services
```bash
consul services register service.hcl
consul services deregister service.hcl
consul catalog services -tags  # show service tags
consul health checks web       # health checks for a service
consul health service web      # healthy instances of a service
```

Example service definition:
```hcl
service {
  name = "web"
  port = 80
  tags = ["primary"]

  check {
    http     = "http://localhost:80/health"
    interval = "10s"
    timeout  = "2s"
  }
}
```

## KV store
```bash
consul kv put config/app/timeout 30
consul kv get config/app/timeout
consul kv get -recurse config/          # dump a whole tree
consul kv delete config/app/timeout
consul kv export config/ > backup.json  # export subtree
consul kv import @backup.json           # import
```

## DNS interface
```bash
dig @127.0.0.1 -p 8600 web.service.consul      # resolve a service via Consul DNS
dig @127.0.0.1 -p 8600 web.service.consul SRV  # get port info too
```

## Connect (service mesh)
```bash
consul connect proxy -sidecar-for web  # run a sidecar proxy manually
consul intention create web db         # allow web -> db traffic
consul intention check web db          # check if traffic is allowed
```

## ACLs
```bash
consul acl bootstrap
consul acl token create -description "ci token" -policy-name=deploy
consul acl policy create -name deploy -rules @policy.hcl
consul acl token list
```

## Useful one-liners
```bash
consul catalog services -tags | column -t                           # readable service+tag list
curl -s http://localhost:8500/v1/health/service/web?passing | jq .  # healthy instances via HTTP API
consul monitor -log-level=DEBUG                                     # stream agent logs live
consul kv get -recurse config/ | grep -i secret                     # audit KV for secrets (shouldn't be any)
watch -n2 'consul members'                                          # live cluster membership
```
