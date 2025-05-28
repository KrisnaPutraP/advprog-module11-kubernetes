>Compare the application logs before and after you exposed it as a Service. Try to open the app several times while the proxy into the Service is running. What do you see in the logs? Does the number of logs increase each time you open the app?

Before I exposed it as a service, when I only ran the pod and checked its logs I saw just the startup messages.

[!kubebefore](/img/kubebefore.png)

After I exposed it as a service, every HTTP request I made shows up as a `GET /` in the logs.

[!kubeafter](/img/kubeafter.png)


Each time I refresh or open the app, I get one more `GET /` line. This shows that my Service is correctly routing external requests into the pod.  

>Notice that there are two versions of `kubectl get` invocation during this tutorial section. The first does not have any option, while the latter has `-n` option with value set to `kube-system`. What is the purpose of the `-n` option and why did the output not list the pods/services that you
explicitly created?

The -n (or --namespace) flag tells kubectl which namespace to target when listing or manipulating resources. By default, if you don’t pass -n, kubectl uses the default namespace:

[!getbefore](/img/getbefore.png)

This shows all Services in default namespace. That’s why we see hello-node Service there.

When we use -n flag and specify which namespace to target:

[!getafter](/img/getafter.png)