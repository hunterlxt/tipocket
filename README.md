# Tipocket

Tipocket is a testing toolkit designed to test TiDB, it encapsulates some testing tools which are also suitable for testing other databases.

Tipocket is inspired by [jepsen-io/jepsen](https://github.com/jepsen-io/jepsen), a famous library on the distributed system field. Tipocket focuses on stability testing on TiDB, it uses [chaos-mesh](https://github.com/pingcap/chaos-mesh) to inject all-round kinds of nemesis on a TiDB cluster.

## Requirements

* [TiDB Operator](https://github.com/pingcap/tidb-operator) >= v1.1.0-rc.2

* [Chaos Mesh](https://github.com/pingcap/chaos-mesh)

## Toolkit

* [go-sqlsmith](https://github.com/pingcap/tipocket/tree/master/pkg/go-sqlsmith): go-sqlsmith is our Go implementation of sqlsmith, it's a fuzz-testing tool which can generate random MySQL-dialect SQL queries.
* [go-elle](https://github.com/pingcap/tipocket/tree/master/pkg/elle): Our Go port version of jepsen-io/elle, a general transactional concsistency checker for black-box databases.

## Nemesis

* random_kill, all_kill, minor_kill, major_kill, kill_tikv_1node_5min, kill_pd_leader_5min: As their name implies, these nemeses inject unavailable in a specified period of time.
* short_kill_tikv_1node, short_kill_pd_leader: Kill selected container, used to inject short duration of unavailable fault.
* partition_one: Isolate single nodes
* scaling: Scale up/down TiDB/PD/TiKV nodes randomly
* shuffle-leader-scheduler/shuffle-region-scheduler/random-merge-scheduler: Just as there name implies.
* delay_tikv, delay_pd, errno_tikv, errno_pd, mixed_tikv, mixed_pd: Inject IO-related fault.
* small_skews, subcritical_skews, critical_skews, big_skews, huge_skews: Clock skew, small_skews ~100ms, subcritical_skews ~200ms, critical_skews ~250ms, big_skews ~500ms and huge_skews ~5s.

## Debug and Run

If you have a K8s cluster, you can use the below commands to deploy and run the case on a TiDB cluster.

### On a K8s cluster

```sh
make build
export KUBECONFIG=$(YOUR_KUBECONFIG_PATH)
bin/${testcase} -namespace=${ns} -hub=docker.io -image-version=nightly -purge=true -storage-class=local-storage
```

### On the local environment

Another convenient way we recommend you is using tiup to deploy a cluster on local and use it to debug cases.

* Start a TiDB cluster

```sh
tiup playground --kv 3
```

* Specify that cluster as SUT cluster on Tipocket

```Go
suit := util.Suit{
      ...
      Provisioner:   cluster.NewLocalClusterProvisioner([]string{"127.0.0.1:4000"}, []string{"127.0.0.1:2379"}, []string{"127.0.0.1:20160"}),
   }
```

## Workloads

Tipocket includes some consistency, isolation and other kinds of tests

### Consistency

* **bank** check bank accounts using a linearizability checker [porcupine](https://github.com/anishathalye/porcupine)
* **vbank** like bank but cover more TiKV features
* **ledger** yet another bank test
* **scbank** transfers between rows of a shared table
* **rawkv-linearizability** rawkv linearizability checker
* **tpcc** use [go-tpc](https://github.com/pingcap/go-tpc) testing consistency

### Isolation

* **append** checks for dependency cycles in transactions using Elle