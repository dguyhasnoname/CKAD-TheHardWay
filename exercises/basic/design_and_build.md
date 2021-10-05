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

2. Team Tokyo needs a Job in namespace `tokyo` . This Job `job-in-tokyo` should run image `busybox` and execute `sleep 5 && echo Hello Tokyo`. It should run a total of `5` times and should execute `3` runs in `parallel`. Pods in the job should have the label `id: tokyo-job`.

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

3. Team Paris needs a CronJob which will run every minute. Team paris do not own any namespace neither they want one. Please create a CrojJob for them which can be accessed by everyone who has read access to the cluster. The CronJob should run command `echo -n "Hello Paris : " && date && sleep 15`. Pods scheduled by the CronJob should have the label `id: paris-job` and image `busybox`. Configure the CronJob in such a way that if it runs late by more than 16 secs, it should be counted as failed. Limit the history of successfull runs to 2 and failed runs to 3.

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