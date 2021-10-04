# Application Environment, Configuration and Security - 25%

1. Create a configMap named `ckad-config` with the values key1=value1, key2=value2, key3=value3 in deafult namespace. Append a hash of the configmap to its name.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create cm ckad-config --from-literal=key1=value1 --from-literal=key2=value2 --from-literal=key3=value3 --append-hash
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    ┗━ ॐ  kd cm ckad-config-4d84kgcb4h
    Name:         ckad-config-4d84kgcb4h
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    key1:
    ----
    value1
    key2:
    ----
    value2
    key3:
    ----
    value3

    BinaryData
    ====

    Events:  <none>
    ```
    </p>
    </details>

2. Create a new config map named `my-config` with specified keys instead of file basenames on disk. The values are the same as the ones in the `my-config` configmap with each of the key being stored in key1.txt, key2.txt and key3.txt respectively.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create cm my-config --from-file=key1=./key1.txt --from-file=key2=./key2.txt --from-file=key3=./key3.txt
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    ┗━ ॐ  kubectl describe configmap/my-config
    Name:         my-config
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    key3:
    ----
    key3=value3

    key1:
    ----
    key1=value1

    key2:
    ----
    key2=value2


    BinaryData
    ====

    Events:  <none>
    ```
    </p>
    </details>

3. Create a multi-container pod `mpod` to mount the configmap `my-config` as volume. The first container should mount `my-config` cm at `/etc/foo`, run `busybox` image with command `sleep 3600`. The second container should run `nginx` image with env named `KEY3` fetching the value from configMap created in question 1(`ckad-config-4d84kgcb4`). Pod should run in default namespace. Verify the result.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl run mpod --image=nginx --restart=Never > pod.yaml
    ```
    </p>
    Update the configMap with the following:
    <p>

    ```text
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: mpod
    name: mpod
    spec:
      containers:
      - image: busybox                                    # add the image
        name: busybox                                     # add the name
        command: ["/bin/sh", "-c", "cat /etc/foo/key1"]   # add the command
        volumeMounts:                                     # mount the volume corresponding to cm `my-config`
        - name: vol1
          mountPath: /etc/foo
      - image: nginx
        name: mpod
        resources: {}
        env:
        - name: KEY3                                      # add the env variable name KEY3
          valueFrom:
            configMapKeyRef:                              # add the configMapKeyRef for ckad-config cm
              name: ckad-config-4d84kgcb4h
              key: key3
    volumes:
    - name: vol1
      configMap:
        name: my-config
    dnsPolicy: ClusterFirst
    restartPolicy: Never
    status: {}
    ```
    </p>
    Apply the pod yaml.
    <p>

    ```bash
    kubectl apply -f pod.yaml
    ```
    </p>
    <p>

    ```bash
    kubectl exec -it mpod -c busybox -- /bin/sh -c 'ls /etc/foo'
    ```
    </p>
    <p>

    ```bash
    kubectl exec -it mpod -c mpod -- /bin/sh -c 'env | grep -i key3'
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    ┗━ ॐ  kubectl describe po mpod
    Name:         mpod
    Namespace:    default
    Priority:     0
    Node:         minikube/192.168.49.2
    Start Time:   Mon, 04 Oct 2021 18:17:59 +0530
    Labels:       run=mpod
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.3
    IPs:
    IP:  172.17.0.3
    Containers:
    busybox:
        Container ID:  docker://9031c9a4ddf46d5c1dbf4edab69c455d9a1a6f73ce2e911ca31d762a851b0f6c
        Image:         busybox
        Image ID:      docker-pullable://busybox@sha256:f7ca5a32c10d51aeda3b4d01c61c6061f497893d7f6628b92f822f7117182a57
        Port:          <none>
        Host Port:     <none>
        Command:
        /bin/sh
        -c
        sleep 3600
        State:          Running
        Started:      Mon, 04 Oct 2021 18:18:03 +0530
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /etc/foo from vol1 (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xxqpz (ro)
    mpod:
        Container ID:   docker://1d4ffb36a45048c519d8ac2fb5bebd7565bcc2459d228672ddcb4ba98b4535be
        Image:          nginx
        Image ID:       docker-pullable://nginx@sha256:765e51caa9e739220d59c7f7a75508e77361b441dccf128483b7f5cce8306652
        Port:           <none>
        Host Port:      <none>
        State:          Running
        Started:      Mon, 04 Oct 2021 18:18:09 +0530
        Ready:          True
        Restart Count:  0
        Environment:
        KEY3:  <set to the key 'key3' of config map 'ckad-config-4d84kgcb4h'>  Optional: false
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xxqpz (ro)
    Conditions:
    Type              Status
    Initialized       True 
    Ready             True 
    ContainersReady   True 
    PodScheduled      True 
    Volumes:
    vol1:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      my-config
        Optional:  false
    kube-api-access-xxqpz:
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
    Normal  Scheduled  33s   default-scheduler  Successfully assigned default/mpod to minikube
    Normal  Pulling    34s   kubelet            Pulling image "busybox"
    Normal  Pulled     30s   kubelet            Successfully pulled image "busybox" in 3.435712452s
    Normal  Created    30s   kubelet            Created container busybox
    Normal  Started    30s   kubelet            Started container busybox
    Normal  Pulling    30s   kubelet            Pulling image "nginx"
    Normal  Pulled     24s   kubelet            Successfully pulled image "nginx" in 6.19584143s
    Normal  Created    24s   kubelet            Created container mpod
    Normal  Started    24s   kubelet            Started container mpod
    ```
    </p>
    Verify the configMap mounts and env.
    <p>

    ```bash
    ┗━ ॐ  kubectl exec -it mpod -c busybox -- /bin/sh -c 'ls /etc/foo'
    key1  key2  key3
    ```
    </p>
    <p>

    ```bash
    ┗━ ॐ  kubectl exec -it mpod -c mpod -- /bin/sh -c 'env | grep -i key3'
    KEY3=value3
    ```
    </p>
    </details>

4. Create a new config map named `envcm` from an env file in namespace `denver`. Create env file with below command `echo -e "SOMEVAR=somevalue\n\nANOTHERVAR=anothervalue" > config.env`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create cm envcm --from-env-file=config.env -n denver
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```yaml
    ┗━ ॐ  kubectl get cm envcm -o yaml -n denver
    apiVersion: v1
    data:
      ANOTHERVAR: anothervalue
      SOMEVAR: somevalue
    kind: ConfigMap
    metadata:
      creationTimestamp: "2021-10-04T13:00:24Z"
      name: envcm
      namespace: denver
      resourceVersion: "30102"
      uid: d2b3a00d-10fe-4e42-8396-f5e1a5df30bc
    ```
    </p>
    </details>

5. Create a namespace `redis`. Create a config map named `redis-config` with the value `redis-config=""`. Create a `redis` pod in namespace `redis`, image as `redis:5.0.4` which will run command `redis-server "/redis-master/redis.conf"` and have env var named `MASTER` with value `true`. The containter should run on port `6379`. Mount the config map `redis-config` to the path `/redis-master` and a emptyDir for redis mount named `data` in `redis` container at `/redis-master-data`.
    <details><summary>steps</summary>
    Create `redis-config` config map.
    <p>

    ```bash
    kubectl create cm redis-config -n redis --from-literal=redis-config=""
    ```
    </p>
    Create the pod yaml file for `redis` pod.
    <p>

    ```bash
    kubectl run redis --dry-run=client -o yaml -n redis --image=redis:5.0.4 --port=6379 --env MASTER=true --command -- redis-server /redis-master/redis.conf  > redis.yaml
    ```
    </p>
    Edit the yaml to add volumes.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: redis
      namespace: redis
    spec:
      containers:
      - name: redis
        image: redis:5.0.4
        command:
          - redis-server
          - "/redis-master/redis.conf"
        env:
        - name: MASTER
          value: "true"
        ports:
        - containerPort: 6379
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
      volumes:
        - name: data
          emptyDir: {}
        - name: config
          configMap:
            name: redis-config
            items:
            - key: redis-config
              path: redis.conf
    ```
    </p>
    <p>

    ```bash
    kubectl apply -f redis.yaml
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    ┗━ ॐ  kubectl describe po -n redis
    Name:         redis
    Namespace:    redis
    Priority:     0
    Node:         minikube/192.168.49.2
    Start Time:   Mon, 04 Oct 2021 18:51:01 +0530
    Labels:       <none>
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.6
    IPs:
      IP:  172.17.0.6
    Containers:
      redis:
        Container ID:  docker://40082115c7428e4fef693c0dfdac40baeaa0de524d1a88a5cdb03dc9910049d4
        Image:         redis:5.0.4
        Image ID:      docker-pullable://redis@sha256:2dfa6432744659268d001d16c39f7be52ee73ef7e1001ff80643f0f7bdee117e
        Port:          6379/TCP
        Host Port:     0/TCP
        Command:
          redis-server
          /redis-master/redis.conf
        State:          Running
          Started:      Mon, 04 Oct 2021 18:51:16 +0530
        Ready:          True
        Restart Count:  0
        Environment:
          MASTER:  true
        Mounts:
          /redis-master from config (rw)
          /redis-master-data from data (rw)
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pqgtx (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      data:
        Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
        Medium:     
        SizeLimit:  <unset>
      config:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      redis-config
        Optional:  false
      kube-api-access-pqgtx:
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
      Type    Reason     Age    From               Message
      ----    ------     ----   ----               -------
      Normal  Scheduled  2m50s  default-scheduler  Successfully assigned redis/redis to minikube
      Normal  Pulling    2m51s  kubelet            Pulling image "redis:5.0.4"
      Normal  Pulled     2m36s  kubelet            Successfully pulled image "redis:5.0.4" in 14.851089269s
      Normal  Created    2m36s  kubelet            Created container redis
      Normal  Started    2m36s  kubelet            Started container redis
    ```
    </p>
    </details>

6. Allocate resource limits and requests for the pod `redis` which was created in question 5. The resource limits and requests are `cpu: limit=1, request=200m, memory: limit=1Gi, request=300Mi`.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl get pod redis -n redis -o yaml > redis.yaml
    ```
    </p>
    Updated resources in the yaml.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: redis
      namespace: redis
      resourceVersion: "0"
    spec:
      containers:
      - name: redis
        image: redis:5.0.4
        command:
          - redis-server
          - "/redis-master/redis.conf"
        env:
        - name: MASTER
          value: "true"
        ports:
        - containerPort: 6379
        resources:
          limits:
            cpu: 1
            memory: 1Gi
          requests:
            cpu: 200m
            memory: 300Mi
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
      volumes:
        - name: data
          emptyDir: {}
        - name: config
          configMap:
            name: redis-config
            items:
            - key: redis-config
              path: redis.conf
    ```
    </p>
    <p>

    ```bash
    kubectl apply -f redis.yaml
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl get po redis -n redis -o jsonpath={.spec.containers[0].resources}
    {"limits":{"cpu":"1","memory":"1Gi"},"requests":{"cpu":"200m","memory":"300Mi"}}
    ```
    </p>
    </details>

7. Run pod named `security-context-pod` in namespace `denver` which should run with user id `1000` and group `3000`. Mount a `emptyDir` volume at location `/etc/sc` whose owner should be from group `2000`. Use `busybox` image to run command `sleep 1800`. The name of container should be `ckad` and it should not run in privilleged mode.

    <details><summary>steps</summary>
    Generate basic yaml for the pod.
    <p>

    ```bash
    kubectl run security-context-pod -n denver --image=busybox --restart=Never --dry-run=client -o yaml > security-context-pod.yaml
    ```
    </p>
    Edit the yaml to make changes.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: security-context-demo
      namespace: denver
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
      volumes:
      - name: sec
        emptyDir: {}
      containers:
      - name: ckad 
        image: busybox
        command: [ "sh", "-c", "sleep 1800" ]
        volumeMounts:
        - name: sec
          mountPath: /etc/sc 
        securityContext:
          allowPrivilegeEscalation: false
      ``` 
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```json
    ┗━ ॐ  kubectl get po -n denver -o json security-context-demo | jq '.spec.containers[]'
    {
      "command": [
        "sh",
        "-c",
        "sleep 1800"
      ],
      "image": "busybox",
      "imagePullPolicy": "Always",
      "name": "ckad",
      "resources": {},
      "securityContext": {
        "allowPrivilegeEscalation": false
      },
      "terminationMessagePath": "/dev/termination-log",
      "terminationMessagePolicy": "File",
      "volumeMounts": [
        {
          "mountPath": "/etc/sc",
          "name": "sec"
        },
        {
          "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
          "name": "kube-api-access-dpzg2",
          "readOnly": true
        }
      ]
    }
    ```
    </p>
    </details>

8. Create a namepsace `tokyo` which allows to run max 10 pods. Run a deployment named `tokyo-deployment` which has a pod spec with `replicas` set to `12`. The pod spec should have `image` set to `nginx:1.7.9`. The container `tody` in pod should have capabilities `CAP_SYS_CHROOT` and `CAP_NET_ADMIN`. The container should not have the capability `SYS_TIME` and it should run in privileged mode. Check the events in namespace to see why all 12 pods are not running. Store the event logs in `tokyo-events.log`.

    <details><summary>steps</summary>
    Create the namespace.
    <p>

    ```bash
    kubectl create ns tokyo
    ```
    </p>
    Create the pod quota yaml for the namespace.
    <p>

    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: pod-demo
      namespace: tokyo
    spec:
      hard:
        pods: "10"
    ```
    </p>
    <p>

    ```bash
    kubectl apply -f pod-quota.yaml
    ```
    </p>
    <p>
    Generate basic yaml for the deployment.
    <p>

    ```bash
    kubectl create deploy tokyo-deployment -n tokyo --image=nginx:1.7.9 --replicas=12 --dry-run=client -o yaml > tokyo-deployment.yaml
    ```
    </p>
    Edit the yaml to make changes.
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: tokyo-deployment
      name: tokyo-deployment
      namespace: tokyo
    spec:
      replicas: 12
      selector:
        matchLabels:
          app: tokyo-deployment
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: tokyo-deployment
        spec:
          containers:
          - image: nginx:1.7.9
            name: tody
            resources: {}
            securityContext:
              capabilities:
                add: ["CAP_SYS_CHROOT", "CAP_NET_ADMIN"]
                drop: ["SYS_TIME"]
              allowPrivilegeEscalation: true
    ```
    </p>
    Apply the yaml to create the deployment.
    <p>

    ```bash
    kubectl apply -f tokyo-deployment.yaml
    ```
    </p>
    Store event logs in `tokyo-events.log`.
    <p>

    ```bash
    kubectl get events -n tokyo > ./tokyo-events.log
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    Verify container configuration.
    <p>

    ```json
    ┗━ ॐ  kubectl get po tokyo-deployment-69c5cd57b6-4dw8h -n tokyo -o json | jq '.spec.containers[]'
    {
      "image": "nginx:1.7.9",
      "imagePullPolicy": "IfNotPresent",
      "name": "tody",
      "resources": {},
      "securityContext": {
        "allowPrivilegeEscalation": true,
        "capabilities": {
          "add": [
            "CAP_SYS_CHROOT",
            "CAP_NET_ADMIN"
          ],
          "drop": [
            "SYS_TIME"
          ]
        }
      },
      "terminationMessagePath": "/dev/termination-log",
      "terminationMessagePolicy": "File",
      "volumeMounts": [
        {
          "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
          "name": "kube-api-access-x5ccj",
          "readOnly": true
        }
      ]
    }
    ```
    </p>
    Verify the number of pods running.
    <p>

    ```bash
    [08:50 PM IST 04.10.2021 ☸ 127.0.0.1:57199 📁 ~ 𖦥 ]
    ┗━ ॐ  kuebctl get po -n tokyo
    NAME                                READY   STATUS    RESTARTS   AGE
    tokyo-deployment-69c5cd57b6-4dw8h   1/1     Running   0          8s
    tokyo-deployment-69c5cd57b6-bh4rn   1/1     Running   0          8s
    tokyo-deployment-69c5cd57b6-gkqfm   1/1     Running   0          8s
    tokyo-deployment-85d564b4b-49ckb    1/1     Running   0          8s
    tokyo-deployment-85d564b4b-7h6rx    1/1     Running   0          8s
    tokyo-deployment-85d564b4b-8db9l    1/1     Running   0          70s
    tokyo-deployment-85d564b4b-lzl8t    1/1     Running   0          8s
    tokyo-deployment-85d564b4b-mxm84    1/1     Running   0          8s
    tokyo-deployment-85d564b4b-xvd69    1/1     Running   0          70s
    tokyo-deployment-85d564b4b-asd12    1/1     Running   0          40s
    ```
    </p>
    Verify event logs
    <p>

    ```bash
    [08:50 PM IST 04.10.2021 ☸ 127.0.0.1:57199 📁 ~ 𖦥 ]
    ┗━ ॐ  kubectl get po -n tokyo
    I1004 20:50:28.398799   58575 cert_rotation.go:137] Starting client certificate rotation controller
    LAST SEEN   TYPE      REASON         OBJECT                                   MESSAGE
    75s         Warning   FailedCreate   replicaset/tokyo-deployment-85d564b4b    Error creating: pods "tokyo-deployment-85d564b4b-5rhqd" is forbidden: exceeded quota: pod-demo, requested: pods=1, used: pods=2, limited: pods=2
    ```
    </p>
    </details>