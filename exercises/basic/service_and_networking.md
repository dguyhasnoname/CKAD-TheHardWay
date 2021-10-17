# Services and Networking - 20%

1. Create a `nginx` deployment in `deafult` namespace and expose it on port `80`. Create a network policy in `default` namespace so that we can limit the access to the nginx service so that only Pods with the label `access: true` can query it. Test access to the service from a `busybox` pod without access label defined.

    <details><summary>steps</summary>
    Create the nginx deployment in default namespace and expose it on port 80.
    <p>

    ```bash
    kubectl create deployment nginx --image=nginx
    ```
    </p>
    <p>

    ```bash
    kubectl expose deployment nginx --port=80
    ```
    </p>
    Create the network policy in default namespace.
    <p>

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: access-nginx
    spec:
      podSelector:
        matchLabels:
          app: nginx
      ingress:
      - from:
        - podSelector:
            matchLabels:
              access: "true"
    ```
    </p>
    Launch a busybox pod and try to access the service.
    <p>

    ```bash
    kubectl run busybox --rm -ti --image=busybox -- /bin/sh
    ```
    </p>
    <p>

    ```bash
    # wget --spider --timeout=1 nginx
    ```
    </p>

    </details>

    <details><summary>verify</summary>
    Check the svc, pods and deployment.
    <p>

    ```text
    [07:19 PM IST 17.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kubectl get svc,pod,deploy
    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/nginx        ClusterIP   10.105.89.112   <none>        80/TCP    13m

    NAME                                     READY   STATUS    RESTARTS          AGE
    pod/nginx-6799fc88d8-lvmw4               1/1     Running   0                 14m

    NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx                1/1     1            1           14m
    ```
    </p>
    Verify that the service is accessible from the busybox pod.
    <p>

    ```bash
    [07:18 PM IST 17.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kubectl run busybox --rm -ti --image=busybox -- /bin/sh
    # wget --spider --timeout=1 nginx
    Connecting to nginx (10.100.0.16:80)
    wget: download timed out

    ```
    </p>
    <p>

    ```bash
    wget --spider --timeout=1 nginx
    ```
    </p>

    </details>

2. Create a pod with appropriate labels so that we can access the `nginx` service created in question 1. The pod should get deleted after successfully access the service.

    <details><summary>steps</summary>

    Create the pod with appropriate labels.
    <p>

    ```bash
    kubectl run busybox --rm -ti --labels="access=true" --image=busybox -- /bin/sh
    ```
    </p>
    <p>

    ```bash
    # wget --spider --timeout=1 nginx
    ```
    </p>

    </details>

    <details><summary>verify</summary>
    Check if the service is accessible.
    <p>

    ```bash
    [07:33 PM IST 17.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kubectl run busybox --rm -ti --labels="access=true" --image=busybox -- /bin/sh
    If you don't see a command prompt, try pressing enter.
    / # wget --spider --timeout=1 nginx
    Connecting to nginx (10.105.89.112:80)
    remote file exists
    ```
    </p>
    </details>







