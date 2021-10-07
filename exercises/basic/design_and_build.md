# Application desgin and build - 20%

1. Create a namespace `lisbon`. Run a pod `lisbon-pod` in the namespace `lisbon` which runs multiple containers. The first container should be named `home` using image `nginx` and the second container should be named `office` using image `busybox`. The `busybox` container should run command `sleep 1000`. Set the port of `nginx` container to 8080.

    <details><summary>steps</summary>
    <p>

    ```bash
    kubectl create ns lisbon
    ```

    </p>
    Genrate basic pod yaml in the namespace `lisbon`.
    <p>

    ```bash
    kubectl run lisbon-pod --image=nginx --port=8080 --namespace=lisbon --dry-run=client -o yaml > lisbon-pod.yaml
    ```

    </p>
    Edit yaml for the pod to add the second container.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: lisbon-pod
      name: lisbon-pod
      namespace: lisbon
    spec:
      containers:
      - image: nginx 
        name: home
        ports:
        - containerPort: 8080
        resources: {}
      - image: busybox
        name: office 
        command: ["/bin/sh", "-c", "sleep 1000"]
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    ```

    </p>
    Apply the pod yaml.
    <p>

    ```bash
    kubectl apply -f lisbon-pod.yaml
    ```

    </p>
    </details>

    <details><summary>verify</summary>
    <p>

    ```json
    kubectl get po lisbon-pod -n lisbon -o jsonpath-as-json={.spec.containers[*]}
    [
        {
            "image": "nginx",
            "imagePullPolicy": "Always",
            "name": "home",
            "ports": [
                {
                    "containerPort": 8080,
                    "protocol": "TCP"
                }
            ],
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "volumeMounts": [
                {
                    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                    "name": "kube-api-access-c9hp6",
                    "readOnly": true
                }
            ]
        },
        {
            "command": [
                "/bin/sh",
                "-c",
                "sleep 1000"
            ],
            "image": "busybox",
            "imagePullPolicy": "Always",
            "name": "office",
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "volumeMounts": [
                {
                    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                    "name": "kube-api-access-c9hp6",
                    "readOnly": true
                }
            ]
        }
    ]
    ```

    </p>
    </details>

2. Create a deployment `deploy-with-init` that spins up a single Pod with image `nginx:1.17.3-alpine` and serves files from a mounted volume at `/usr/share/nginx/html`, which is empty right now in namespace `lisbon`. Add an InitContainer named `myinit-con` which also mounts the same volume and creates a file `index.html` with content `Hello from Lisbon!` in the root dir of the mounted volume. The InitContainer should be using image `busybox`. Run a `curl` pod to check if the index.html is accessible. The `curl` pod shoudl use `nginx:alpine` image and it should cease to exist after the test.

    <details><summary>steps</summary>
    Genrate basic deployment yaml in the namespace `lisbon`.
    <p>

    ```bash
    kubectl create deploy deploy-with-init --image=nginx:1.17.3-alpine --namespace=lisbon --dry-run=client -o yaml > deploy-with-init.yaml
    ```

    </p>
    Edit yaml for the pod to add the init container and the volume mapping.
    <p>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: deploy-with-init
      name: deploy-with-init
      namespace: lisbon
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: deploy-with-init
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: deploy-with-init
        spec:
          initContainers:
          - name: myinit-con
            image: busybox
            command:
            - /bin/sh
            - -c
            - echo 'Hello from Lisbon!' > /tmp/foo/index.html
            volumeMounts:
            - name: vol
              mountPath: /tmp/foo
          containers:
          - image: nginx:1.17.3-alpine
            name: nginx
            resources: {}
            volumeMounts:
            - name: vol
              mountPath: /usr/share/nginx/html
          volumes:
          - name: vol
            emptyDir: {}
    ```

    </p>
    Apply the deploy-with-init.yaml.
    <p>

    ```bash
    kubectl apply -f deploy-with-init.yaml
    ```

    </p>
    Get the pod IP of the pod run by deployment and then run curl pod to verify the index.html is accessible.
    <p>

    ```bash
    kubectl run curl -it --rm --image=nginx:alpine --restart=Never -n lisbon -- /bin/sh -c 'curl -I http://172.17.0.6'
    ```

    </details>

    <details><summary>verify</summary>
    <p>

    ```bash
    [11:38 PM IST 05.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
    ┗━ ॐ  kubectl get po -n lisbon -o wide
    NAME                               READY   STATUS    RESTARTS         AGE     IP            NODE       NOMINATED NODE   READINESS GATES
    deploy-with-init-b74c59b7d-h85dz   1/1     Running   0                2m18s   172.17.0.6    minikube   <none>           <none>
    ```

    </p>
    Verify if init container was created and the index.html is written.
    <p>

    ```json
    kubectl get po -n lisbon deploy-with-init-b74c59b7d-h85dz -o jsonpath-as-json={.spec.initContainers[*]}
    [
        {
            "command": [
                "/bin/sh",
                "-c",
                "echo 'Hello from Lisbon!' \u003e /tmp/foo/index.html"
            ],
            "image": "busybox",
            "imagePullPolicy": "Always",
            "name": "myinit-con",
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "volumeMounts": [
                {
                    "mountPath": "/tmp/foo",
                    "name": "vol"
                },
                {
                    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                    "name": "kube-api-access-vgj9k",
                    "readOnly": true
                }
            ]
        }
    ]
    ```

    </p>
    Running the curl pod.
    <p>

    ```text
    kubectl run curl -it --rm --image=nginx:alpine --restart=Never -n lisbon -- /bin/sh -c 'curl -I http://172.17.0.6'
    HTTP/1.1 200 OK
    Server: nginx/1.17.3
    Date: Tue, 05 Oct 2021 18:09:26 GMT
    Content-Type: text/html
    Content-Length: 19
    Last-Modified: Tue, 05 Oct 2021 18:06:39 GMT
    Connection: keep-alive
    ETag: "615c942f-13"
    Accept-Ranges: bytes

    pod "curl" deleted
    ```

    </p>
    <p>

    ```text
    kubectl run curl -it --rm --image=nginx:alpine --restart=Never -n lisbon -- /bin/sh -c 'curl  http://172.17.0.6'
    Hello from Lisbon!
    pod "curl" deleted
    ```

    </p>
    </details>

3. Team Tokyo needs a Job in namespace `tokyo` . This Job `job-in-tokyo` should run image `busybox` and execute `sleep 5 && echo Hello Tokyo`. It should run a total of `5` times and should execute `3` runs in `parallel`. Pods in the job should have the label `id: tokyo-job`.

    <details><summary>steps</summary>
    Create basic job yaml.
    <p>

    ```bash
    kubectl create job job-in-tokyo --image=busybox --namespace=tokyo --dry-run=client -o yaml -- /bin/sh -c "sleep 5 && echo Hello Tokyo" > job-in-tokyo.yaml
    ```

    </p>
    Modify job settings.
    <p>

    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: job-in-tokyo
      namespace: tokyo
    spec:
      completions: 5
      parallelism: 3
      template:
        metadata:
          labels:
            id: tokyo-job
        spec:
          containers:
          - image: busybox
            name: job-in-tokyo
            command:
            - sh
            - -c
            - sleep 5 && echo Hello Tokyo
            resources: {}
          restartPolicy: Never
    ```

    </p>
    Apply the job yaml.
    <p>

    ```bash
    kubectl apply -f job-in-tokyo.yaml
    ```

    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```text
    kg po -n tokyo
    NAME                                READY   STATUS    RESTARTS   AGE
    job-in-tokyo--1-c6gc6               1/1     Running   0          12s
    ```

    </p>
    <p>

    ```text
    ┗━ ॐ  kd job job-in-tokyo -n tokyo
    Name:             job-in-tokyo
    Namespace:        tokyo
    Selector:         controller-uid=dee9e531-a901-4eb4-92c3-347dbc26031d
    Labels:           controller-uid=dee9e531-a901-4eb4-92c3-347dbc26031d
                    job-name=job-in-tokyo
    Annotations:      <none>
    Parallelism:      3
    Completions:      5
    Completion Mode:  NonIndexed
    Start Time:       Mon, 04 Oct 2021 22:32:22 +0530
    Pods Statuses:    1 Running / 3 Succeeded / 0 Failed
    Pod Template:
    Labels:  controller-uid=dee9e531-a901-4eb4-92c3-347dbc26031d
            job-name=job-in-tokyo
    Containers:
    job-in-tokyo:
        Image:      busybox
        Port:       <none>
        Host Port:  <none>
        Command:
        /bin/sh
        -c
        sleep 5 && echo Hello Tokyo
        Environment:  <none>
        Mounts:       <none>
    Volumes:        <none>
    ```

    </p>
    </details>

4. Create a CronJob `vibrant-tokyo` in namespace `tokyo`. It should run `every minute` with a restartPolicy of `OnFailure`. It should run commands `date; Welcome to vibrant Tokyo!` with image `busybox`. The pod image should be pulled only when it is not present on the kubernetes node where it is scheduled. The pods should have the label `id: vibrant-tokyo`.

    <details><summary>steps</summary>
    Create basic job yaml in tokyo namespace.
    <p>

    ```bash
    kubectl create cj vibrant-tokyo --image=busybox --namespace=tokyo --dry-run=client -o yaml --schedule="*/1 * * * *" --  /bin/sh -c 'date; echo Welcome to vibrant Tokyo!'> vibrant-tokyo.yaml
    ```
    </p>
    Modify job settings.
    <p>

    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: vibrant-tokyo
      namespace: tokyo
    spec:
      jobTemplate:
        metadata:
          name: vibrant-tokyo
        spec:
          template:
            metadata:
              labels:
                id: vibrant-tokyo
            spec:
              containers:
              - command:
                - /bin/sh
                - -c
                - date; echo Welcome to vibrant Tokyo!
                image: busybox
                imagePullPolicy: IfNotPresent
                name: vibrant-tokyo
                resources: {}
              restartPolicy: OnFailure
      schedule: '*/1 * * * *'
    ```
    </p>
    Apply the job vibrant-tokyo.yaml.
    <p>

    ```bash
    kubectl apply -f vibrant-tokyo.yaml
    ```
    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```bash
    [09:26 AM IST 07.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
    ┗━ ॐ  kubectl get cj -n tokyo
    NAME            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
    vibrant-tokyo   */1 * * * *   False     1        41s             4m49s
    ```
    </p>
    Check the pods.
    <p>

    ```bash
    kubectl get po -n tokyo
    NAME                              READY   STATUS      RESTARTS   AGE
    vibrant-tokyo-27226314--1-mfnc8   0/1     Completed   0          11s
    vibrant-tokyo-27226315--1-bx7cj   0/1     Completed   0          30s
    vibrant-tokyo-27226316--1-bj8hw   0/1     Completed   0          40s
    ```
    </p>
    Check the logs.
    <p>

    ```bash
    [09:26 AM IST 07.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
    ┗━ ॐ  kubectl logs vibrant-tokyo-27226314--1-mfnc8 -n tokyo
    Thu Oct  7 03:56:30 UTC 2021
    Welcome to vibrant Tokyo!
    ```
    </p>
    </details>

5. Team Paris needs a CronJob which will run every minute. Team paris do not own any namespace neither they want one. Please create a CrojJob for them which can be accessed by everyone who has read access to the cluster. The CronJob should run command `echo -n "Hello Paris : " && date && sleep 15`. Pods scheduled by the CronJob should have the label `id: paris-job` and image `busybox`. Configure the CronJob in such a way that if it runs late by more than 16 secs, it should be counted as failed. Limit the history of successfull runs to 2 and failed runs to 3.

    <details><summary>steps</summary>
    Create basic cronjob yaml.
    <p>

    ```bash
    kubectl create cronjob job-in-paris --image=busybox --schedule="0 * * * *" --dry-run=client -o yaml -- /bin/sh -c 'echo -n "Hello Paris : " && date && sleep 15' > cronjob-in-paris.yaml
    ```

    </p>
    Modify job settings.
    <p>

    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: job-in-paris
    spec:
      startingDeadlineSeconds: 15
      successfulJobsHistoryLimit: 2
      failedJobsHistoryLimit: 3
      jobTemplate:
        metadata:
          name: job-in-paris
        spec:
          template:
            metadata:
              labels:
                id: paris-job
            spec:
              containers:
              - command:
                - /bin/sh
                - -c
                - 'echo -n "Hello Paris : " && date && sleep 15'
                image: busybox
                name: job-in-paris
                resources: {}
              restartPolicy: OnFailure
      schedule: "*/1 * * * *"
    ```

    </p>
    Apply the job yaml.
    <p>

    ```bash
    kubectl apply -f cronjob-in-paris.yaml
    ```

    </p>
    </details>

    <details><summary>result</summary>
    <p>

    ```bash
    kubectl get cj
    NAME           SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
    job-in-paris   */1 * * * *   False     0        36s             10m
    ```

    </p>
    <p>

    ```text
    kubectl get po -l id=paris-job
    NAME                             READY   STATUS      RESTARTS      AGE
    job-in-paris-27224153--1-8dtvc   0/1     Completed   0             2m10s
    job-in-paris-27224154--1-szkmp   0/1     Completed   0             70s
    job-in-paris-27224155--1-6fm8r   1/1     Running     0             10s
    ```

    </p>
    <details>

6. Create three different jobs names `apple`, `banana` and `cherry`. Create a single template file `job-template.yaml` to create all three jobs. Each of the jobs should run command `echo Processing {{ job-name }} && sleep 5`. Pass the job name as parameter to the command. The jobs should run in `default` namespace. Do not use helm in this case. Each job should have label `id: {{ job-name }}`.

    <details><summary>steps</summary>
    Create the template file using helm
    <p>

      ```yaml
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: $ITEM
        labels:
          id: $ITEM
      spec:
        template:
          metadata:
            name: jobexample
            labels:
              jobgroup: jobexample
          spec:
            containers:
            - name: c
              image: busybox
              command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
            restartPolicy: Never
      ```

      </p>
      Generate the job yaml files.
      <p>

      ```bash
      mkdir ./jobs
      for i in apple banana cherry
      do
        cat job-tmpl.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml
      done
      ```

      </p>
      Check the job yaml files generated.
      <p>

      ```bash
      ls ./jobs
      job-apple.yaml
      job-banana.yaml
      job-cherry.yaml
      ```

      </p>
      Apply the job yaml files.
      <p>

      ```bash
      kubectl apply -f ./jobs
      ```

      </p>
      </details>

      <details><summary>verify</summary>
      <p>

      ```bash
      [11:15 PM IST 06.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
      ┗━ ॐ  for i in apple banana cherry; do cat jobs.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml; done
      [11:15 PM IST 06.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
      ┗━ ॐ  ls ./jobs
      job-apple.yaml job-banana.yaml job-cherry.yaml
      ```

      </p>

      <p>

      ```bash
      [11:15 PM IST 06.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
      ┗━ ॐ  kubectl apply -f ./jobs
      job.batch/apple created
      job.batch/banana created
      job.batch/cherry created
      ```

      </p>
      <p>

      ```text
      [11:16 PM IST 06.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
      ┗━ ॐ  kubectl get po
      NAME                             READY   STATUS      RESTARTS      AGE
      apple--1-rn87q                   0/1     Completed   0             27s
      banana--1-smhmb                  0/1     Completed   0             27s
      cherry--1-4vf4k                  0/1     Completed   0             27s
      ```

      </p>
      Check logs for each job.
      <p>

      ```bash
      [11:16 PM IST 06.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲]
      ┗━ ॐ  kubectl logs banana--1-smhmb
      Processing item banana
      ```

      </p>
      </details>

7. Create a `PersistentVolume` named `persuit-of-persistence` with storage of `3Gi`. The volumeMode should not be `Block` type. Reclaim policy should be `Recycle` with access mode set in a way so that the volume can be mounted as `read-only by many nodes`. The storage class for the PV is `persuit` with `Retain` reclaim policy. PV should mount `/mnt/data` on the k8s host.

    <details><summary>steps</summary>
    Create the storage class persuit-sc.yaml
    <p>

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: persuit
    provisioner: k8s.io/minikube-hostpath # this is for minikube clusters. Use appropriate provisioner if working on other clusters.
    reclaimPolicy: Retain
    allowVolumeExpansion: true
    ```
    </p>
    Create the persistent volume persuit-of-persistence.yaml file.
    <p>

    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: persuit-of-persistence
    spec:
      storageClassName: persuit
      capacity:
        storage: 3Gi
      accessModes:
        - ReadOnlyMany
      persistentVolumeReclaimPolicy: Recycle
      volumeMode: Filesystem
      hostPath:
        path: /mnt/data
      ```

      </p>
      Create the storage class.
      <p>

      ```bash
      kubectl apply -f persuit-sc.yaml
      ```
      </p>
      Apply the PV yaml file.
      <p>

      ```bash
      kubectl apply -f  persuit-of-persistence.yaml
      ```
      </p>

      </details>

      <details><summary>verify</summary>
      <p>

      ```bash
      [10:33 AM IST 07.10.2021 ☸ 127.0.0.1:51368 📁 CKAD-TheHardWay ❱ master ▲] 
      ┗━ ॐ  kubectl get sc,pv
      NAME                                             PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
      storageclass.storage.k8s.io/persuit              k8s.io/minikube-hostpath   Retain          Immediate           true                   6m50s

      NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON   AGE
      persistentvolume/persuit-of-persistence                     3Gi        ROX            Recycle          Available                           persuit                 43s
      ```
      </p>
      </details>