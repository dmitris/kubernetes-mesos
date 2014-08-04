kubernetes-mesos
================

When [Google Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) meets [Apache Mesos](http://mesos.apache.org/)


[![GoDoc] (https://godoc.org/github.com/mesosphere/kubernetes-mesos?status.png)](https://godoc.org/github.com/mesosphere/kubernetes-mesos)

Kubernetes and Mesos are a match made in heaven. Kubernetes enables the Pod (group of containers) abstraction, along with Pod labels for service discovery, load-balancing, and replication control. Mesos provides the fine-grained resource allocations for pods across nodes in a cluster, and can make Kubernetes play nicely with other frameworks running on the same cluster resources. With the Kubernetes framework for Mesos, the framework scheduler will register Kubernetes with Mesos, and then Mesos will begin offering Kubernetes sets of available resources from the cluster nodes (slaves/minions). The framework scheduler will match Mesos' resource offers to Kubernetes pods to run, and then send a launchTasks message to the Mesos master, which will forward the request onto the appropriate slaves. The slave will then fetch the kubelet/executor and the pod/task to run, start running the pod, and send a TASK_RUNNING update back to the Kubernetes framework scheduler. Once the pods are running on the slaves/minions, they can make use of Kubernetes' pod labels to register themselves in etcd for service discovery, load-balancing, and replication control. 

### Roadmap
This is still very much a work-in-progress, but stay tuned for updates as we continue development. If you have ideas or patches, feel free to contribute!

- [x] Launching pods
  1. Implement Kube-scheduler API
  1. Pick a Pod (FCFS), match it to an offer.
  1. Launch it!
  1. Kubelet as Executor+Containerizer
- [x] Pod Labels: for Service Discovery + Load Balancing
- [ ] Replication Control
- [ ] Smarter scheduling
  1. Pluggable to reuse existing Kubernetes schedulers
  1. Implement our own!

### Build

NOTE: kubernetes-mesos requires [godep](https://github.com/tools/godep).

```shell
$ go get github.com/GoogleCloudPlatform/kubernetes
$ go get github.com/mesosphere/kubernetes-mesos
$ pushd $GOPATH/src/github.com/mesosphere/kubernetes-mesos
$ GOPATH=$GOPATH:$GOPATH/src/github.com/GoogleCloudPlatform/kubernetes/third_party godep restore
$ popd
$ go install github.com/mesosphere/kubernetes-mesos/kubernetes-mesos
$ go install github.com/mesosphere/kubernetes-mesos/kubernetes-executor
$ go install github.com/GoogleCloudPlatform/kubernetes/cmd/proxy
$ $GOPATH/bin/kubernetes-mesos -h
```

### Start the framework

Assuming your mesos cluster is started, and the master is running on `127.0.1.1:5050`, then:

```shell
$ ./bin/kubernetes-mesos \
  -machines=$(hostname) \
  -mesos_master=127.0.1.1:5050 \
  -executor_uri=$(pwd)/bin/kubernetes-executor
```

###Launch a Pod

Assuming your framework is running on `localhost:8080`, then:

```shell
$ curl -L http://localhost:8080/api/v1beta1/pods -XPOST -d @examples/pod.json
```

After the pod get launched, you can check it's status via `curl` or your web browser:
```shell
$ curl -L http://localhost:8080/api/v1beta1/pods
```

```json
{
	"kind": "PodList",
	"items": [
		{
			"id": "php",
			"labels": {
				"name": "foo"
			},
			"desiredState": {
				"manifest": {
					"version": "v1beta1",
					"id": "php",
					"volumes": null,
					"containers": [
						{
							"name": "nginx",
							"image": "dockerfile/nginx",
							"ports": [
								{
									"hostPort": 8080,
									"containerPort": 80
								}
							],
							"livenessProbe": {
								"enabled": true,
								"type": "http",
								"httpGet": {
									"path": "/index.html",
									"port": "8080"
								},
								"initialDelaySeconds": 30
							}
						}
					]
				}
			},
			"currentState": {
				"manifest": {
					"version": "",
					"id": "",
					"volumes": null,
					"containers": null
				}
			}
		}
	]
}
```

Or, you can run `docker ps -a` to verify that the example container is running:

```shell
CONTAINER ID        IMAGE                       COMMAND                CREATED             STATUS              PORTS               NAMES
3fba73ff274a        busybox:buildroot-2014.02   sh -c 'rm -f nap &&    57 minutes ago                                              k8s--net--php--9acb0442   
```

### Test

Run test suite with:

```shell
$ go test github.com/mesosphere/kubernetes-mesos/kubernetes-mesos -v
```
