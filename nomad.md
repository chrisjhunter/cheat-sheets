# HashiCorp Nomad Cheat Sheet

## Cluster status
```bash
nomad server members                 # server cluster members
nomad node status                      # list client nodes
nomad node status -verbose               # more detail
nomad node status <node-id>                # single node detail
nomad agent-info                             # info about local agent
nomad status                                   # list all jobs
```

## Jobs
```bash
nomad job init                       # scaffold an example job file (example.nomad.hcl)
nomad job validate job.nomad.hcl       # validate syntax without running
nomad job plan job.nomad.hcl             # dry-run, show what would change
nomad job run job.nomad.hcl                # submit/update a job
nomad job stop myjob                         # stop (with graceful deregistration)
nomad job stop -purge myjob                    # stop and remove all job history
nomad job status myjob                           # job overview + allocations
nomad job status -verbose myjob
nomad job history myjob                            # version history
nomad job revert myjob 3                             # roll back to version 3
nomad job restart myjob                                # restart all allocations
nomad job scale myjob taskgroup 5                        # scale a task group
```

## Allocations (running instances of tasks)
```bash
nomad alloc status <alloc-id>              # allocation detail
nomad alloc status -verbose <alloc-id>
nomad alloc logs <alloc-id>                  # stdout logs
nomad alloc logs -stderr <alloc-id>
nomad alloc logs -f <alloc-id>                 # follow logs
nomad alloc exec -i -t <alloc-id> /bin/sh        # shell into a running allocation
nomad alloc fs <alloc-id> /                        # browse allocation filesystem
nomad alloc restart <alloc-id>                       # restart allocation's tasks
nomad alloc stop <alloc-id>                            # stop/reschedule an allocation
```

## Deployments
```bash
nomad deployment list
nomad deployment status <deployment-id>
nomad deployment promote <deployment-id>      # promote canaries
nomad deployment fail <deployment-id>           # force-fail a deployment (triggers rollback)
nomad deployment pause <deployment-id>
```

## Example job file
```hcl
job "web" {
  datacenters = ["dc1"]
  type        = "service"

  group "web" {
    count = 3

    network {
      port "http" { to = 80 }
    }

    task "server" {
      driver = "docker"

      config {
        image = "nginx:latest"
        ports = ["http"]
      }

      resources {
        cpu    = 200
        memory = 128
      }

      service {
        name = "web"
        port = "http"
        provider = "consul"

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

## Namespaces & ACLs
```bash
nomad namespace list
nomad namespace apply -description "staging env" staging
nomad job run -namespace=staging job.nomad.hcl
nomad acl bootstrap                       # bootstrap ACL system (first time)
nomad acl token create -name="ci" -policy=deploy
nomad acl policy apply deploy policy.hcl
```

## Volumes & CSI
```bash
nomad volume status
nomad volume register volume.hcl
nomad volume deregister <vol-id>
nomad plugin status                    # CSI plugin status
```

## Useful one-liners
```bash
nomad job run -check-index 0 job.nomad.hcl              # force run even if job changed elsewhere
nomad status | grep -v dead                                # only active jobs
nomad alloc status -json <alloc-id> | jq .ClientStatus        # scriptable status check
nomad node status -self                                          # info about node you're running on
nomad job dispatch parameterized-job                                # dispatch a parameterized/batch job
nomad monitor -log-level=DEBUG                                        # stream agent logs live
nomad operator raft list-peers                                          # raft peer status (servers)
nomad operator autopilot get-config                                       # autopilot health settings
watch -n2 'nomad status'                                                     # live job list refresh
```
