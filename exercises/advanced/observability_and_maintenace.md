# Application Observability and Maintenance - 15%

1. Run below command:

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/advanced/om-question-1.yaml
    ```

    Above command creates a namespace `om-question-1` and runs the `counter` pod. The container in pod writes to two different log files using two different formats. Please add side containers to the pod so that both type of logs can be viewed separately. View the logs separately and store the logs in a file `count-log-1.log` and `count-log-2.log` respectively.

    <details><summary>solution</summary>
    yaml for the pod with side car containers.
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
      namespace: om-question-1
    spec:
      containers:
      - name: count
        image: busybox
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-1
        image: busybox
        args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-2
        image: busybox
        args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}
    ```
    </p>
    Storing logs in different files.
    <p>

    ```bash
    kubectl logs counter -n om-question-1 -c count-log-1 > count-log-1.log
    ```
    </p>
    <p>

    ```bash
    kubectl logs counter -n om-question-1 -c count-log-2 > count-log-2.log
    ```
    </p>
    </details>