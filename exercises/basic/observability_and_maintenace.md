# Application Observability and Maintenance - 15%

1. Create a Pod that runs a container based on the `k8s.gcr.io/busybox`. The pod should run command `touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600`. Set up a liveness probe with command `ls /tmp/healthy`. The liveness check should run every `5 seconds`. The first check should run only after `10 seconds`.


    <details><summary>steps</summary>
    Genrate basic yaml for a pod.
    <p>

    ```bash
     kubectl run liveness-pod --image=k8s.gcr.io/busybox --dry-run=client -o yaml --command -- /bin/sh -c 'touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600' > liveness-pod.yaml
    ```
    <p>
    Add the liveness probe to the pod:
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        test: liveness-pod
      name: liveness-pod
    spec:
      containers:
      - name: liveness-pod
        image: k8s.gcr.io/busybox
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        livenessProbe:
          exec:
            command:
            - ls
            - /tmp/healthy
          initialDelaySeconds: 10
          periodSeconds: 5
    ```
    <p>
    Apply the pod yaml.
    <p>

    ```bash
     kubectl apply -f liveness-pod.yaml
    ```
    <p>
    </details>

    <details><summary>verify</summary>
    <p>
    Verify that the pod is running.
    <p>

    ```bash
     kubectl describe pods liveness-pod

    [06:35 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
    ┗━ ॐ  kd po liveness-pod
    Name:         liveness-pod
    Namespace:    default
    Priority:     0
    Node:         minikube/192.168.49.2
    Start Time:   Sat, 16 Oct 2021 18:34:41 +0530
    Labels:       run=liveness-pod
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.3
    IPs:
      IP:  172.17.0.3
    Containers:
      liveness-pod:
        Container ID:  docker://4dba7106ddbd630bc444bc4f9cd7912db7d7f1ec4aa86c55855d17c0c0504af7
        Image:         k8s.gcr.io/busybox
        Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
        Port:          <none>
        Host Port:     <none>
        Command:
          /bin/sh
          -c
          touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        State:          Running
          Started:      Sat, 16 Oct 2021 18:34:43 +0530
        Ready:          True
        Restart Count:  0
        Liveness:       exec [ls /tmp/healthy] delay=10s timeout=1s period=5s #success=1 #failure=3
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t5f44 (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      kube-api-access-t5f44:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
      Type    Reason     Age   From               Message
      ----    ------     ----  ----               -------
      Normal  Scheduled  29s   default-scheduler  Successfully assigned default/liveness-pod to minikube
      Normal  Pulling    29s   kubelet            Pulling image "k8s.gcr.io/busybox"
      Normal  Pulled     27s   kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.7829422s
      Normal  Created    27s   kubelet            Created container liveness-pod
      Normal  Started    27s   kubelet            Started container liveness-pod
    ```
    <p>
    </details>

2. Create a Pod named `goproxy` that runs a container based on the  image `k8s.gcr.io/goproxy:0.1` on port `8080`. Set up TCP liveness and readiness probe on port `8080`. The first readiness probe should run only after `5 seconds` and it should run every `10 seconds`. The liveness check should run every `15 seconds` and it should start after `15 seconds`.

    <details><summary>steps</summary>
    Genrate basic yaml for a pod.
    <p>

    ```bash
    kubectl run goproxy --image=k8s.gcr.io/goproxy:0.1 --dry-run=client -o yaml --port=8080  > goproxy.yaml
    ```
    <p>
    Add the liveness and readiness probe to the pod:
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: goproxy
      labels:
        app: goproxy
    spec:
      containers:
      - name: goproxy
        image: k8s.gcr.io/goproxy:0.1
        ports:
        - containerPort: 8080
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
    ```
    <p>
    Apply the pod yaml.
    <p>

    ```bash
     kubectl apply -f goproxy.yaml
    ```
    <p>

    </details>

    <details><summary>verify</summary>

    Verify that the pod is running.
    <p>

    ```bash
    [06:42 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kg po
    NAME           READY   STATUS             RESTARTS      AGE
    goproxy        0/1     Running            0             10s
    ```
    <p>
    <p>

    ```test
    [06:43 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
    ┗━ ॐ  kd po goproxy
    Name:         goproxy
    Namespace:    default
    Priority:     0
    Node:         minikube/192.168.49.2
    Start Time:   Sat, 16 Oct 2021 18:42:39 +0530
    Labels:       app=goproxy
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.4
    IPs:
      IP:  172.17.0.4
    Containers:
      goproxy:
        Container ID:   docker://a37135b07b17da846ac6a7a2963edde19dec4ce6b3dee5b149b536da5b1afa54
        Image:          k8s.gcr.io/goproxy:0.1
        Image ID:       docker-pullable://k8s.gcr.io/goproxy@sha256:5334c7ad43048e3538775cb09aaf184f5e8acf4b0ea60e3bc8f1d93c209865a5
        Port:           8080/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Sat, 16 Oct 2021 18:42:43 +0530
        Ready:          True
        Restart Count:  0
        Liveness:       tcp-socket :8080 delay=15s timeout=1s period=20s #success=1 #failure=3
        Readiness:      tcp-socket :8080 delay=5s timeout=1s period=10s #success=1 #failure=3
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2524b (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      kube-api-access-2524b:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
      Type    Reason     Age   From               Message
      ----    ------     ----  ----               -------
      Normal  Scheduled  57s   default-scheduler  Successfully assigned default/goproxy to minikube
      Normal  Pulling    56s   kubelet            Pulling image "k8s.gcr.io/goproxy:0.1"
      Normal  Pulled     53s   kubelet            Successfully pulled image "k8s.gcr.io/goproxy:0.1" in 3.2190074s
      Normal  Created    53s   kubelet            Created container goproxy
      Normal  Started    53s   kubelet            Started container goproxy
    ```
    <p>
    </details>

3. Apply below yaml:

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/liveness-probe-3.yaml
    ```
    Once the pod is up and running, check after `10 seconds`, if the pod is getting restarted. If it is restarting, try to find the issue and fix it so that pod do not restart any more.

    <details><summary>steps</summary>
    Apply the give yaml.
    <p>

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/liveness-probe-3.yaml
    ```
    <p>

    </details>

    <details><summary>verify</summary>

    Verify that the pod is running.
    <p>

    ```bash
    [06:55 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
    ┗━ ॐ  kg po
    NAME               READY   STATUS    RESTARTS      AGE
    liveness-probe-3   1/1     Running   1 (41s ago)   91s
    ```
    </p>
    Check the reason for the pod's restart.
    <p>

    ```text
      [06:55 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
      ┗━ ॐ  kd po liveness-probe-3
      Name:         liveness-probe-3
      Namespace:    default
      Priority:     0
      Node:         minikube/192.168.49.2
      Start Time:   Sat, 16 Oct 2021 18:54:53 +0530
      Labels:       test=liveness-probe-3
      Annotations:  <none>
      Status:       Running
      IP:           172.17.0.5
      IPs:
        IP:  172.17.0.5
      Containers:
        liveness-probe-3:
          Container ID:  docker://839e94fd73837d5dd55dde73f992171067c66284b61fc22136215df972076610
          Image:         k8s.gcr.io/busybox
          Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
          Port:          <none>
          Host Port:     <none>
          Args:
            /bin/sh
            -c
            touch /tmp/healthy; sleep 30; echo $HOSTNAME; sleep 600
          State:          Running
            Started:      Sat, 16 Oct 2021 18:54:55 +0530
          Ready:          True
          Restart Count:  0
          Liveness:       exec [cat /tmp/heal] delay=5s timeout=1s period=5s #success=1 #failure=3
          Environment:    <none>
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mhld2 (ro)
      Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
      Volumes:
        kube-api-access-mhld2:
          Type:                    Projected (a volume that contains injected data from multiple sources)
          TokenExpirationSeconds:  3607
          ConfigMapName:           kube-root-ca.crt
          ConfigMapOptional:       <nil>
          DownwardAPI:             true
      QoS Class:                   BestEffort
      Node-Selectors:              <none>
      Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                  node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
      Events:
        Type     Reason     Age               From               Message
        ----     ------     ----              ----               -------
        Normal   Scheduled  21s               default-scheduler  Successfully assigned default/liveness-probe-3 to minikube
        Normal   Pulling    21s               kubelet            Pulling image "k8s.gcr.io/busybox"
        Normal   Pulled     19s               kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.8025539s
        Normal   Created    19s               kubelet            Created container liveness-probe-3
        Normal   Started    19s               kubelet            Started container liveness-probe-3
        Warning  Unhealthy  1s (x3 over 11s)  kubelet            Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
        Normal   Killing    1s                kubelet            Container liveness-probe-3 failed liveness probe, will be restarted
    ```
    </p>
    The pod is failing due to incorrect liveness probe. Store the yaml of pod locally in `liveness-probe-3.yaml` and update the probe. 
    <p>

    ```yaml
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
    ```
    <p>
    Delete the running and then apply the updated yaml.
    <p>

    ```bash
    kubectl delete po liveness-probe-3
    ```
    </p>
    <p>

    ```bash
    kubectl apply -f liveness-probe-3.yaml
    ```
    <p>
    Check the status of running pod. The pod should not have any restarts now.
    <p>

    ```bash
    [07:06 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
    ┗━ ॐ  kg po
    NAME               READY   STATUS             RESTARTS        AGE
    liveness-probe-3   1/1     Running            0               34s
    ```
    </p>
    </details>

4. Delete the `liveness-probe-3` pod deployed in question 3. Configure the liveness probe as a result of which, the probe at a time should retry only `once` before giving up. Update the `periodSeconds` to `3s` and probe should timeout to `1s`. Set the `terminationGracePeriodSeconds` to `6s`. Check the events only of `liveness-probe-3` pod and store only the warnings in `error.log`.

    <details><summary>steps</summary>
    Delete the pod.
    <p>

    ```bash
    kubectl delete po liveness-probe-3
    ```
    <p>

    Configure the liveness probe to use failureThreshold as `5`.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        test: liveness-probe-3
      name: liveness-probe-3
    spec:
      containers:
      - name: liveness-probe-3
        image: k8s.gcr.io/busybox
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; echo $HOSTNAME; sleep 600
        resources: {}
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/heal
          initialDelaySeconds: 5
          periodSeconds: 3
          timeoutSeconds: 1
          failureThreshold: 1
          terminationGracePeriodSeconds: 6
    ```
    </p>
    Apply the updated yaml.
    <p>

    ```bash
    kubectl apply -f liveness-probe-3.yaml
    ```
    </p>
    Store the events in error.log after checking the events.
    <p>

    ```bash
    kubectl get events --field-selector type==Warning,involvedObject.name=liveness-probe-3 > error.log
    ```
    </p>

    </details>

    <details><summary>verify</summary>
    Check the status of pod again..
    <p>

    ```text
    [07:23 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kg po
    NAME               READY   STATUS             RESTARTS         AGE
    liveness-probe-3   1/1     Running            6 (32s ago)    5m8s
    ```
    </p>
    Check events for the pod.
    <p>

    ```bash
    kubectl get events --field-selector type==Warning,involvedObject.name=liveness-probe-3 

    LAST SEEN   TYPE      REASON          OBJECT                 MESSAGE
    59m         Warning   Unhealthy       pod/liveness-probe-3   Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
    56m         Warning   BackOff         pod/liveness-probe-3   Back-off restarting failed container
    39m         Warning   FailedKillPod   pod/liveness-probe-3   error killing pod: failed to "KillContainer" for "liveness-probe-3" with KillContainerError: "rpc error: code = Unknown desc = Error response from daemon: No such container: 1169a57854a73d64c83f5383e8ef3506ea9605b11bd6e3bd9d1323b1badf6b78"
    37m         Warning   Unhealthy       pod/liveness-probe-3   Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
    27m         Warning   BackOff         pod/liveness-probe-3   Back-off restarting failed container
    21m         Warning   Unhealthy       pod/liveness-probe-3   Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
    17m         Warning   BackOff         pod/liveness-probe-3   Back-off restarting failed container
    15m         Warning   Unhealthy       pod/liveness-probe-3   Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
    9m22s       Warning   Unhealthy       pod/liveness-probe-3   Liveness probe failed: cat: can't open '/tmp/heal': No such file or directory
    81s         Warning   BackOff         pod/liveness-probe-3   Back-off restarting failed container
    ```
    </p>
    </details>

5. `Jasmine` is a new kubernetes developer and she is trying to configure a deployment in default namespace. She is facing some issue and not able to create the deployment successfully. She is using  deployment file stored at:

    ```
    https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/om-deployment-5.yaml
    ```

    Please identify the issue with the deployment yaml. Once issue is identified, updated the deployment yaml and apply the updated yaml. Verify the pods are running.

    <details><summary>steps</summary>
    Try running the deployment yaml.
    <p>

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/om-deployment-5.yaml
    error: unable to recognize "https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/om-deployment-5.yaml": no matches for kind "Deployment" in version "apps/v1beta2"
    ```
    </p>
    Deprecated api Version is being used in the deployment yaml. Find the supported api version and update it.
    <p>

    ```bash
    kubectl api-resources --api-group apps | grep deployments
    ```
    </p>
    Update the deployment yaml.
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment-5
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx-5
            image: nginx:1.14.2
            ports:
            - containerPort: 80
    ```
    </p>
    Apply the updated yaml.
    <p>

    ```bash
    kubectl apply -f om-deployment-5.yaml
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    Check the status of deployment.
    <p>


    ```bash
    [08:15 PM IST 16.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ] 
    ┗━ ॐ  kg po
    NAME                                 READY   STATUS             RESTARTS         AGE
    nginx-deployment-5-f786fd499-csprl   1/1     Running            0                34s
    nginx-deployment-5-f786fd499-nfnhb   1/1     Running            0                34s
    ```
    </p>

    </details>








6. Run a pod with name `apiserver` in default namespace, marked with labels `app=bookstore` and `role=api`. Create a netpol to restrict the access only to other pods (e.g. other microservices) running with label `app=bookstore`.

    <details><summary>steps</summary>
    Create a pod with name apiserver in default namespace, marked with labels app=bookstore and role=api.
    <p>

    ```bash
    kubectl run apiserver --image=nginx --labels app=bookstore,role=api --expose --port 80

    ```
    </p>
    Create a netpol to restrict the access only to other pods (e.g. other microservices) running with label app=bookstore.
    <p>

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: api-allow
    spec:
      podSelector:
        matchLabels:
          app: bookstore
          role: api
      ingress:
      - from:
          - podSelector:
              matchLabels:
                app: bookstore
    ```
    </p>
    Apply the netpol yaml.
    <p>


    ```bash
    kubectl apply -f api-allow.yaml
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    Test the Network Policy is blocking the traffic, by running a Pod without the app=bookstore label:
    <p>

    ```bash
    $ kubectl run test-$RANDOM --rm -i -t --image=alpine -- sh
    / # wget -qO- --timeout=2 http://apiserver
    wget: download timed out
    ```
    </p>

    Test the Network Policy is allowing the traffic, by running a Pod with the app=bookstore label:

    <p>

    ```bash
    $ kubectl run test-$RANDOM --rm -i -t --image=alpine --labels app=bookstore,role=frontend -- sh
    / # wget -qO- --timeout=2 http://apiserver
    <!DOCTYPE html>
    <html><head>
    ```
    </p>
    </details>
