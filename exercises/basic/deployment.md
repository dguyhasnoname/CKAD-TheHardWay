# Application deployment - 20%

1. Create a namespace with name `denver`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create namespace denver
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```text
    namespace/denver created
    ```
    </p>
    </details>

2. Label namespace `denver` with label `type: ckad`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl label namespace denver type=ckad
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl get namespace denver --show-labels
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```text
    NAME     STATUS   AGE   LABELS
    denver   Active   22m   kubernetes.io/metadata.name=denver,type=ckad
    ```
    </p>
    </details>

3. Annotate namespace `denver` with annotation:  `description: for ckad lab`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl annotate namespace denver description='for ckad lab'
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl get namespace denver -o jsonpath={.metadata.annotations}
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```text
    {"description":"for ckad lab"}
    ```
    </p>
    </details>

4. Run a pod name `game` with image `dguyhasnoname/game2048:latest` in namespace `denver`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl run game -n denver --image=dguyhasnoname/game2048:latest --restart=Never
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    kubectl get po -n denver
    NAME   READY   STATUS    RESTARTS   AGE
    game   1/1     Running   0          6m32s
    ```
    </p>
    </details>

5. Get yaml for the pod `game` in namespace `denver`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl get po game -n denver -o yaml
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```yaml
    metadata:
      creationTimestamp: "2021-10-03T05:45:26Z"
    labels:
      run: game
    name: game
    namespace: denver
    resourceVersion: "240869"
    uid: 9f79d237-c090-4cdc-a422-48ba266f3497
    spec:
      containers:
      - image: dguyhasnoname/game2048:latest
        imagePullPolicy: Always
        name: game
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-l4b8x
          readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: minikube
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Never
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
    - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
    volumes:
    - name: kube-api-access-l4b8x
        projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
                path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
                path: namespace
    status:
      conditions:
      - lastProbeTime: null
        lastTransitionTime: "2021-10-03T05:45:26Z"
        status: "True"
        type: Initialized
      - lastProbeTime: null
        lastTransitionTime: "2021-10-03T05:45:59Z"
        status: "True"
        type: Ready
      - lastProbeTime: null
        lastTransitionTime: "2021-10-03T05:45:59Z"
        status: "True"
        type: ContainersReady
      - lastProbeTime: null
        lastTransitionTime: "2021-10-03T05:45:26Z"
        status: "True"
        type: PodScheduled
      containerStatuses:
      - containerID: docker://a22afea1c9fef2c8f0d7c171d71c44901858db40105c53fd6722e3a66a768467
        image: dguyhasnoname/game2048:latest
        imageID: docker-pullable://dguyhasnoname/game2048@sha256:d39bc83cd36b5179e547ce172332e687f83cbe3e4b5ee24f4714073068663708
        lastState: {}
        name: game
        ready: true
        restartCount: 0
        started: true
          state:
            running:
              startedAt: "2021-10-03T05:45:59Z"
    hostIP: 192.168.49.2
    phase: Running
    podIP: 172.17.0.23
    podIPs:
    - ip: 172.17.0.23
    qosClass: BestEffort
    startTime: "2021-10-03T05:45:26Z"
    ```
    </p>
    </details>

6. Execute a simple shell on the `game` pod in namespace `denver`

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl exec -it game -n denver -- /bin/sh
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl exec -it game -n denver -- /bin/sh
    # hostname
    game
    ```
    </p>
    </details>

7. Create a busybox pod with name `bb` in default namespace that echos all the environment variables inside the pod and then exits.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl run bb --image=busybox --restart=Never -it -- env
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOSTNAME=bb
    TERM=xterm
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    KUBERNETES_PORT_443_TCP_PORT=443
    KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
    KUBERNETES_SERVICE_HOST=10.96.0.1
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_SERVICE_PORT_HTTPS=443
    KUBERNETES_PORT=tcp://10.96.0.1:443
    KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
    HOME=/root
    ```
    </p>
    <p>

    ```bash
    kubectl get po
    NAME                     READY   STATUS             RESTARTS         AGE
    bb                       0/1     Completed          0                86s
    ```
    </p>
    </details>

8. Create pod with name `leaf` in default namespace that runs commad `echo This is leaf pod.`. The output should be visible on stdout and after that pod should get deleted by itself. The pod should use image `busybox`.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl run leaf --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo This is leaf pod.'
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl run leaf --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo This is leaf pod.'
    This is leaf pod.
    pod "leaf" deleted
    ```
    </p>
    </details>

9. Generate yaml for a deployment named `running` in namespace `exercise` that uses image `alpine:latest` and replicas 2. The deployment should have a label `app=running-daily`. The container should have a command `echo "Hello from running"`. The name of container should be `running-daily`. Do not run the deployment.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create ns exercise
    ```
    <p>

    ```bash
    kubectl create deploy running --image=alpine:latest --replicas=2 --namespace=exercise --dry-run=client -o yaml -- /bin/sh -c 'echo "Hello from running"'
    ```
    </p>

    Edit the container name to `running-daily` and add the label `app=running-daily`.
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: running-daily
    name: running
    namespace: exercise
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: running-daily
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: running-daily
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - echo "Hello from running"
            image: alpine:latest
            name: running-daily
            resources: {}
    status: {}
    ```
    </details>

10. Convert the pod `game`(deployed in question 4) to deployment `game` in namespace `denver`. Use same image, container name as `game` and number to replicas should be 2.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create deployment game -n denver --image=dguyhasnoname/game2048:latest --replicas=2 --dry-run=client -o yaml > game_deploy.yaml
    ```
    </p>
    <p>

    ```bash
    vi game_deploy.yaml
    ```
    </p>
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
    labels:
      app: game
    name: game
    namespace: denver
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: game
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: game
        spec:
          containers:
          - image: dguyhasnoname/game2048:latest
            name: game2048
            resources: {}
    status: {}
    ```
    </p>
    <p>

    ```bash
    kubectl apply -f game_deploy.yaml
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    kubectl get po -n denver
    NAME                    READY   STATUS    RESTARTS   AGE
    game                    1/1     Running   0          20m
    game-6dbf688b5f-jqfhn   1/1     Running   0          12s
    game-6dbf688b5f-qm7gm   1/1     Running   0          12s
    ```
    </p>
    <p>

    ```text
    kubectl get deployment -n denver
    NAME   READY   UP-TO-DATE   AVAILABLE   AGE
    game   2/2     2            2           82s
    ```
    </p>
    </details>

11. Scale the deployment `game`(deployment created in question 10) to 3 replicas.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl scale deployment game -n denver --replicas=3
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    kubectl get po -n denver
    NAME                    READY   STATUS              RESTARTS   AGE
    game                    1/1     Running             0          68m
    game-6dbf688b5f-jqfhn   1/1     Running             0          48m
    game-6dbf688b5f-qm7gm   1/1     Running             0          48m
    game-6dbf688b5f-xbmwt   0/1     ContainerCreating   0          5s
    ```
    </p>
    </details>

12. Expose the deployment `game` in namespace `denver` to the outside world. Run a `nginx:alpine` pod and test it using curl to see if the deployment is accessible.

    <details><summary>steps</summary>
    <p>
        
    ```bash
    kubectl expose deployment game -n denver --port=4444 --target-port=80 --type=ClusterIP --name=games
    ```  
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl get svc -n denver
    NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    games   ClusterIP   10.111.182.190   <none>        4444/TCP         2m40s
    ```
    </p>

    <p>

    ```bash
    kubectl run curl --rm -it --image=nginx:alpine --restart=Never -n denver -- /bin/sh -c 'curl -I http://10.111.182.190:4444'
    ```
    </p>

    <p>

    ```text
    HTTP/1.1 200 OK
    Server: nginx/1.15.12
    Date: Sun, 03 Oct 2021 07:09:06 GMT
    Content-Type: text/html
    Content-Length: 3378
    Last-Modified: Wed, 26 Sep 2018 18:37:14 GMT
    Connection: keep-alive
    ETag: "5babd1da-d32"
    Accept-Ranges: bytes

    pod "curl" deleted
    ```
    </p>
    </details>

13. Create a service `games` in namespace `denver` that exposes the deployment `game` to the outside world. The type of service should be NodePort. Login into the node to verify if the service is accessible using node port.

    <details><summary>steps</summary>
    <p>
        
    ```bash
    kubectl expose deployment game -n denver --port=3333 --target-port=80 --type=NodePort --name=game
    ```  
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    kubectl get  svc -n denver
    NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    game    NodePort    10.98.205.242    <none>        3333:32105/TCP   9m16s
    ```
    </p>

    <p>

    ```bash
    kubectl get po -n denver -o wide
    NAME                    READY   STATUS      RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
    game-6dbf688b5f-jqfhn   1/1     Running     0          72m   172.17.0.24   minikube   <none>           <none>
    game-6dbf688b5f-qm7gm   1/1     Running     0          72m   172.17.0.25   minikube   <none>           <none>
    game-6dbf688b5f-xbmwt   1/1     Running     0          23m   172.17.0.26   minikube   <none>           <none>
    ```
    <p>

    ```bash
    minikube ssh
    Last login: Sun Oct  3 07:03:31 2021 from 192.168.49.1
    docker@minikube:~$
    ```
    </p>

    In this example, minikube is being used. If you are using some other cluster, you can login on the corresponding node where the game pod is running and then run the following command(replace minikube with your node name) to verify if the service is accessible.
    <p>

    ```bash
    docker@minikube:~$ curl -I http://minikube:32105
    HTTP/1.1 200 OK
    Server: nginx/1.15.12
    Date: Sun, 03 Oct 2021 07:15:15 GMT
    Content-Type: text/html
    Content-Length: 3378
    Last-Modified: Wed, 26 Sep 2018 18:37:14 GMT
    Connection: keep-alive
    ETag: "5babd1da-d32"
    Accept-Ranges: bytes
    ```
    </p>
    </details>

14. Deploy `tri-color` helm release in the `delhi` namespace using helm. Use bitnami/nginx helm chart to deploy the release. Deploy the helm chart with default values. Check the rollout history of deployment created by the helm release.

    <details><summary>steps</summary>
    Dowload the helm chart from bitnami/nginx repository.
    <p>

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    ```
    </p>

    Update the helm repo
    <p>

    ```bash
    helm repo update
    ```
    </p>

    Install the release
    <p>

    ```bash
    helm install tri-color bitnami/nginx --create-namespace --namespace delhi
    ```
    </p>

    Check deployment created and rollout status.
    <p>

    ```bash
    kubectl get deploy -n delhi
    kubectl rollout status deploy tri-color-nginx -n delhi
    ```
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    helm install tri-color bitnami/nginx --create-namespace --namespace delhi
    NAME: tri-color
    LAST DEPLOYED: Sun Oct  3 13:14:45 2021
    NAMESPACE: delhi
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    ** Please be patient while the chart is being deployed **

    NGINX can be accessed through the following DNS name from within your cluster:

        tri-color-nginx.delhi.svc.cluster.local (port 80)

    To access NGINX from outside the cluster, follow the steps below:

    1. Get the NGINX URL by running these commands:

    NOTE: It may take a few minutes for the LoadBalancer IP to be available.
            Watch the status with: 'kubectl get svc --namespace delhi -w tri-color-nginx'

        export SERVICE_PORT=$(kubectl get --namespace delhi -o jsonpath="{.spec.ports[0].port}" services tri-color-nginx)
        export SERVICE_IP=$(kubectl get svc --namespace delhi tri-color-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
        echo "http://${SERVICE_IP}:${SERVICE_PORT}"

    ```
    </p>
    <p>

    ```bash
    helm list -n delhi
    NAME     	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART      	APP VERSION
    tri-color	delhi    	1       	2021-10-03 13:14:45.184267 +0530 IST	deployed	nginx-9.5.5	1.21.3
    ```
    </p>
    <p>

    ```bash
    kubectl get po -n delhi
    NAME                              READY   STATUS    RESTARTS   AGE
    tri-color-nginx-c987dd577-f4lg6   1/1     Running   0          110s
    ```
    </p>

    Check rollout status of deployment created.

    <p>

    ```bash
    kubectl get deploy -n delhi
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    tri-color-nginx   1/1     1            1           3m45s
    ```

    <p>

    ```bash
    kubectl rollout status deploy tri-color-nginx -n delhi
    deployment "tri-color-nginx" successfully rolled out
    ```
    </details>

15. Change the tri-color helm release to use some other version of the chart. Check the rollout status of deployment created by the helm release.

    <details><summary>steps</summary>
    Search version for bitnami/nginx chart.
    <p>

    ```bash
    helm search repo bitnami/nginx --versions
    ```
    </p>

    Update the helm release `tri-color`
    <p>

    ```bash
    helm upgrade --install tri-color --version=9.5.0 bitnami/nginx --create-namespace --namespace delhi
    ```
    </p>

    Check the rollout history for deployment.
    <p>

    ```bash
    kubectl rollout history deploy tri-color-nginx -n delhi
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    Check the new release.
    <p>

    ```bash
    helm upgrade --install tri-color --version=9.5.0 bitnami/nginx --create-namespace --namespace delhi
    Release "tri-color" has been upgraded. Happy Helming!
    NAME: tri-color
    LAST DEPLOYED: Sun Oct  3 13:26:24 2021
    NAMESPACE: delhi
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    NOTES:
    ** Please be patient while the chart is being deployed **

    NGINX can be accessed through the following DNS name from within your cluster:

        tri-color-nginx.delhi.svc.cluster.local (port 80)

    To access NGINX from outside the cluster, follow the steps below:

    1. Get the NGINX URL by running these commands:

    NOTE: It may take a few minutes for the LoadBalancer IP to be available.
            Watch the status with: 'kubectl get svc --namespace delhi -w tri-color-nginx'

        export SERVICE_PORT=$(kubectl get --namespace delhi -o jsonpath="{.spec.ports[0].port}" services tri-color-nginx)
        export SERVICE_IP=$(kubectl get svc --namespace delhi tri-color-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
        echo "http://${SERVICE_IP}:${SERVICE_PORT}"
    ```
    </p>
    List the helm release.
    <p>

    ```bash
    helm list -n delhi
    NAME     	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART      	APP VERSION
    tri-color	delhi    	2       	2021-10-03 13:26:24.203328 +0530 IST	deployed	nginx-9.5.0	1.21.1
    ```
    </p>

    Check rollout status of deployment created.
    <p>

    ```bash
    kubectl rollout history deploy tri-color-nginx -n delhi
    deployment.apps/tri-color-nginx
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```
    </p>
    </details>

16. Delete the tri-color helm release.

    <details><summary>steps</summary>
    Delete the helm release `tri-color`.
    <p>

    ```bash
    helm uninstall tri-color -n delhi
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    helm uninstall tri-color -n delhi
    release "tri-color" uninstalled
    ```
    </p>
    </details>

17. Create yaml for a statefulset with name `nginx` in deafult namespace. Use image `k8s.gcr.io/nginx-slim:0.8`. Label the statefulset with `app=nginx-sts`. It should run 3 replicas. Configure the container name as `nginx-sts` and it should not take more than 5s to shutdown the pod if the statefulset pod is terminated. Use PVCs for sts in `RWO` accessMode and mount them at `/usr/share/nginx/html` in statefulset pods. Use the `default` storage class to create the pvcs for the statefulset. Check the statefulset yaml and deploy it. Verify the sts, pod and PVCs created and their mappings.

    <details><summary>steps</summary>
    Prepare yaml for statefulset.
    <p>

    ```text
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: nginx
    spec:
      selector:
        matchLabels:
          app: nginx-sts
      serviceName: "nginx"
      replicas: 3 # by default is 1
      template:
        metadata:
          labels:
            app: nginx-sts
        spec:
          terminationGracePeriodSeconds: 5
          containers:
          - name: nginx-sts
            image: k8s.gcr.io/nginx-slim:0.8
            volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes: [ "ReadWriteOnce" ]
          resources:
            requests:
              storage: 200Mi
    ```
    </p>

    deploy statefulset.

    <p>

    ```bash
    kubectl apply -f nginx-sts.yaml
    ```
    </p>

    Check the statefulset, pods and PVCs.
    <p>

    ```bash
    kubectl get statefulset,po,pvc
    ```
    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```text
    kubectl get statefulset,po,pvc -o wide
    NAME                     READY   AGE     CONTAINERS   IMAGES
    statefulset.apps/nginx   3/3     5m11s   nginx-sts    k8s.gcr.io/nginx-slim:0.8

    NAME                         READY   STATUS             RESTARTS          AGE     IP            NODE       NOMINATED NODE   READINESS GATES
    pod/nginx-0                  1/1     Running            0                 5m11s   172.17.0.27   minikube   <none>           <none>
    pod/nginx-1                  1/1     Running            0                 5m9s    172.17.0.28   minikube   <none>           <none>
    pod/nginx-2                  1/1     Running            0                 5m8s    172.17.0.31   minikube   <none>           <none>

    NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/www-nginx-0   Bound    pvc-2b120e12-53e6-40b4-bb5b-6bb5c1aa0698   200Mi      RWO            standard       2m59s
    persistentvolumeclaim/www-nginx-1   Bound    pvc-a34cf366-061c-4ef3-932f-ea2274d9154e   200Mi      RWO            standard       2m56s
    persistentvolumeclaim/www-nginx-2   Bound    pvc-29417f58-6048-457a-a30f-ae3194be4e39   200Mi      RWO            standard       2m53s

    ```
    </p>
    </details>

18. Create a deployment with name `nginx-deployment` in `denver` namespace. Use image `nginx:1.14.2` with container name as `nginx-deployment`. Label the deployment with `app=nginx-deployment`. It should run 3 replicas. The deployment should also run another container with image busybox and command `sleep 3600`. Once the pods for the deployment are up, update the image to `nginx:1.16.1` from`nginx:1.14.2`. and verify the pods are updated and see the rollout status. Once the pods are updated, rollback the image to previous version and ensure pod are runnig fine after rollback and no more accidental rollout happens.

    <details><summary>steps</summary>
    Create initial deployment yaml and store in nginx-deployment.yaml.
    <p>

    ```bash
    kubectl create deploy nginx-deployment -n denver --image=nginx:1.14.2 --replicas=3 --dry-run=client -o yaml
    ```
    </p>
    Edit the deployment yaml to add another container.

    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
      name: nginx-deployment
      namespace: denver
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx-deployment
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: nginx-deployment
        spec:
          containers:
          - image: busybox
            name: busybox
            command: ["/bin/sh", "-c", "sleep 3600"]
          - image: nginx:1.14.2
            name: nginx-deployment
            resources: {}
    status: {}
    ```
    </p>
    Apply the deployment yaml.

    <p>

    ```bash
    kubectl apply -f nginx-deployment.yaml
    ```
    </p>
    Update the image to nginx:1.16.1.
    <p>

    ```bash
    kubectl set image deploy nginx-deployment nginx-deployment=nginx:1.16.2 -n denver
    ```
    </p>
    Verify the pods are updated and see the rollout status.
    <p>

    ```bash
    kubectl rollout status deploy nginx-deployment -n denver
    ```
    </p>
    Rollback the image to nginx:1.14.2.
    <p>

    ```bash
    kubectl rollout undo nginx-deployment -n denver --to-revision=1
    ```
    </p>
    Pause the rollout to avoid any further release.
    <p>

    ```bash
    kubectl rollout pause nginx-deployment -n denver
    ```
    </p>
    </details>

    <details><summary>result</summary>
    Verify that deployment pods are running fine.
    <p>

    ```text
    ┗━ ॐ  kubectl get po -n denver -l app=nginx-deployment
    NAME                                READY   STATUS      RESTARTS   AGE
    nginx-deployment-6774947b7d-cprzd   2/2     Running     0          36s
    nginx-deployment-6774947b7d-g9wfm   2/2     Running     0          59s
    nginx-deployment-6774947b7d-v4zzm   2/2     Running     0          80s
    ```
    </p>

    Verify that the image is updated.
    <p>

    ```bash
    kubectl get po -n denver -l app=nginx-deployment -o jsonpath='{.items[*].spec.containers[?(@.name=="nginx-deployment")].image}'
    ```
    </p>
    Verify if the rollout is paused. Try updating a new image and see if changes gets rolled out.
    <p>

    ```bash
    ┗━ ॐ  kubectl set image deploy nginx-deployment nginx-deployment=nginx:1.16.4 -n denver
    deployment.apps/nginx-deployment image updated
    ┗━ ॐ  kubectl rollout status deploy nginx-deployment -n denver
    Waiting for deployment "nginx-deployment" rollout to finish: 0 out of 3 new replicas have been updated...
    ```
    </p>
    If you check the rollout will not happen and old pods will keep running. We may have to resume the rollout to see the changes.
    </details>

19. Update the image to nginx:1.21.3 in deployment `nginx-deployment`(created in question 18). Resume the rollout so that new image changes can be propagated.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl set image deploy nginx-deployment nginx-deployment=nginx:1.21.3 -n denver
    ```
    </p>
    Resume the rollout.
    <p>

    ```bash
    kubectl rollout resume nginx-deployment -n denver
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```bash
    ┗━ ॐ  kubectl rollout resume deploy nginx-deployment -n denver
    deployment.apps/nginx-deployment resumed
    ```
    </p>

    Verify that the pods are updated.
    <p>

    ```bash
    ┗━ ॐ      kubectl get po -n denver -l app=nginx-deployment -o jsonpath='{.items[*].spec.containers[?(@.name=="nginx-deployment")].image}'
    nginx:1.21.3 nginx:1.21.3 nginx:1.21.3
    ```
    </details>

20. Team Mumbai wants to setup stateful MySQL db in namespace `mumbai`. Create a statefulset `mysql` in `mumbai` namespace with image `mysql:5.6`. Store the password of mysql in secret `mysql-secret` and pass it to container with name `MYSQL_ROOT_PASSWORD` as an environment variable. Run the MySQL instance on port `3306`. Mount a PVC named `mysql-pv-claim` in the `mysql` container at path `/var/lib/mysql`. The PVC should be binded to a PV named `mysql-pv-volume` with storageClass as `manual`. The PV & PVC should have `ReadWriteOnce` access mode. The PVC should request for `1Gi` storage. The PV capacity should be `3Gi`. Expose the `mysql` deployment on port `3306` using a headless service.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create ns mumbai
    ```
    </p>
    Create the storageClass, PV and PVC using storage.yaml.
    <p>

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: manual
    provisioner: k8s.io/minikube-hostpath # this is for minikube clusters. Use appropriate provisioner if working on other clusters.
    reclaimPolicy: Retain
    allowVolumeExpansion: true
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mysql-pv-volume
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 3Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/mnt/data"
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
      namespace: mumbai
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    ```
    </p>
    Generate the basic mysql deployment  yaml.
    <p>

    ```bash
    kubectl create deploy mysql -n mumbai --image=mysql:5.6 --port=3306 --dry-run=client -o yaml > mysql-deploy.yaml
    ```
    </p>
    Edit the deployment yaml.
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql
      namespace: mumbai
    spec:
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - image: mysql:5.6
            name: mysql
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-pv-claim
    ```
    </p>
    Create the secret mysql-secret.
    <p>

    ```bash
    kubectl create secret generic mysql-secret --from-literal=MYSQL_ROOT_PASSWORD=password -n mumbai
    ```
    </p>
    Create the service `mysql-service` by below data in mysql-service.yaml.
    <p>

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
      namespace: mumbai
    spec:
      ports:
      - port: 3306
      selector:
        app: mysql
      clusterIP: None
    ```
    </p>
    Apply the deployment, storage and service yaml.

    <p>

    ```bash
    kubectl apply -f storage.yaml
    kubectl apply -f mysql-deploy.yaml
    kubectl apply -f mysql-service.yaml
    ```
    </p>
    </details>

    <details><summary>result</summary>
    Check if storage is created.
    <p>

    ```bash
    [11:09 PM IST 04.10.2021 ☸ 127.0.0.1:57199 📁 CKAD-TheHardWay ❱ master ▲] 
    ┗━ ॐ  kg sc,pv,pvc
    NAME                                             PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    storageclass.storage.k8s.io/manual               k8s.io/minikube-hostpath   Retain          Immediate           true                   6s
    storageclass.storage.k8s.io/standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  28h

    NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON   AGE
    persistentvolume/mysql-pv-volume                            3Gi        RWO            Retain           Bound       mumbai/mysql-pv-claim   manual                  6s
    persistentvolume/pvc-ebf0372d-f5c4-4446-b023-175d22ee1ab1   1Gi        RWO            Retain           Available   mumbai/mysql-pv-claim   manual                  6s
    ```
    </p>
    Check if mysql pod in up and mysql service is up inside the pod.
    <p>

    ```text
    ┗━ ॐ  kg po,svc,secret -n mumbai
    NAME                         READY   STATUS    RESTARTS   AGE
    pod/mysql-756f767845-tsrqq   1/1     Running   0          8m10s
    pod/mysql-client             0/1     Error     0          50s

    NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
    service/mysql   ClusterIP   None         <none>        3306/TCP   6m51s

    NAME                         TYPE                                  DATA   AGE
    secret/default-token-55rs4   kubernetes.io/service-account-token   3      11m
    secret/mysql-secret          Opaque                                1      7m43s
    ```
    </p>
    Check if mysql is accessible using the headless service.
    <p>

    ```bash
    [11:15 PM IST 04.10.2021 ☸ 127.0.0.1:57199 📁 CKAD-TheHardWay ❱ master ▲]
    ┗━ ॐ  kubectl run -n mumbai -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
    If you don't see a command prompt, try pressing enter.

    mysql>
    mysql>
    mysql> exit
    Bye
    ```
    </p>

    </details>
