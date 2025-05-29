>Compare the application logs before and after you exposed it as a Service. Try to open the app several times while the proxy into the Service is running. What do you see in the logs? Does the number of logs increase each time you open the app?

Before I exposed it as a service, when I only ran the pod and checked its logs I saw just the startup messages.

![kubebefore](/img/kubebefore.png)

After I exposed it as a service, every HTTP request I made shows up as a `GET /` in the logs.

![kubeafter](/img/kubeafter.png)


Each time I refresh or open the app, I get one more `GET /` line. This shows that my Service is correctly routing external requests into the pod.  

>Notice that there are two versions of `kubectl get` invocation during this tutorial section. The first does not have any option, while the latter has `-n` option with value set to `kube-system`. What is the purpose of the `-n` option and why did the output not list the pods/services that you
explicitly created?

The -n (or --namespace) flag tells kubectl which namespace to target when listing or manipulating resources. By default, if you don’t pass -n, kubectl uses the default namespace:

![getbefore](/img/getbefore.png)

This shows all Services in default namespace. That’s why we see hello-node Service there.

When we use -n flag and specify which namespace to target:

![getafter](/img/getafter.png)

>What is the difference between Rolling Update and Recreate deployment strategy?

In a RollingUpdate, Kubernetes incrementally replaces old Pods with new ones, keeping our application live by maintaining a portion of the old replicas up while the new replicas spin up (and automatically rolling back if the new ones don’t become healthy). This strategy lets us tune how many Pods can be unavailable or exceeded at once, so we get near–zero downtime and a safety buffer in case something goes wrong. By contrast, the Recreate strategy shuts down every old Pod first and only then creates the new Pods. There is no overlap—so we incur a brief but total downtime during the switch. Recreate can be simpler for workloads that can’t safely run two versions side‐by‐side (for example, when performing irreversible schema migrations), whereas RollingUpdate is favored for stateless services that must remain highly available during upgrades.

>Try deploying the Spring Petclinic REST using Recreate deployment strategy and document
your attempt.

First, I created `recreate.yaml` and applied it. It set strategy.type: Recreate. Then, I ran `kubectl get deployment spring-petclinic-rest -o yaml > recreate.yaml` to pull down the current Deployment object, complete with its new strategy.type: Recreate, updated revision and generation numbers, and all the server-populated metadata and status fields

>Prepare different manifest files for executing Recreate deployment strategy.

This is my `recreate.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-05-29T02:47:29Z"
  generation: 1
  labels:
    app: spring-petclinic-rest
  name: spring-petclinic-rest
  namespace: default
  resourceVersion: "1863"
  uid: 330c000b-6357-45a6-a1e7-0fdce399a0b8
spec:
  progressDeadlineSeconds: 600
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: spring-petclinic-rest
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: spring-petclinic-rest
    spec:
      containers:
      - image: docker.io/springcommunity/spring-petclinic-rest:3.0.2
        imagePullPolicy: IfNotPresent
        name: spring-petclinic-rest
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 4
  conditions:
  - lastTransitionTime: "2025-05-29T02:47:32Z"
    lastUpdateTime: "2025-05-29T02:47:32Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2025-05-29T02:47:29Z"
    lastUpdateTime: "2025-05-29T02:47:32Z"
    message: ReplicaSet "spring-petclinic-rest-b984fd555" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 4
  replicas: 4
  updatedReplicas: 4
```

Pay attention to:

```yaml
    ...
    strategy:
        type: Recreate
    ...
```

>What do you think are the benefits of using Kubernetes manifest files? Recall your experience
in deploying the app manually and compare it to your experience when deploying the same app
by applying the manifest files (i.e., invoking `kubectl apply -f` command) to the cluster.

When we deploy by hand, we have to type lots of commands one by one, remember the right flags and image tags, and it’s easy to make mistakes or forget exactly what we did. With manifest files, we put all of that information into simple YAML files, like replica counts, image names, ports, update strategy, and so on. Then we just run `kubectl apply -f` and Kubernetes makes our cluster match those files every time. This makes deployments consistent and repeatable: we can use the same files on any machine or after a crash without having to remember commands. Keeping the manifests in Git also lets us track changes, share with teammates, and roll back if something goes wrong.
