# Application desgin and build - 20%

1. Create multi-container pod `two-containers` in `default` namespace. The first container should run `nginx` image and voume mounted at `/usr/share/nginx/html` directory. The second container should run `busybox` image and volume mounted at `/data` directory. The `busybox` container should run command `echo Hello from the debian container > /data/index.html`.  The content written by `busybox` container should be visible in the `nginx` container. Run an extra container in the pod which can let us see the content of `/usr/share/nginx/html/index.html` from `kubectl logs command`. Verify the logs of the third container.

    <details><summary>solution</summary>
    Pod two-containers yaml:
    <p>

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: two-containers
    spec:
      restartPolicy: Never
      volumes:
      - name: shared-data
        emptyDir: {}
      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
      - name: busybox-container
        image: busybox
        volumeMounts:
        - name: shared-data
          mountPath: /data
        command: ["/bin/sh"]
        args: ["-c", "echo Hello from the busybox container > /data/index.html"]
      - name: sidecar
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "tail -f /usr/share/nginx/html/index.html"]
        volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
    ```
    </p>
    <p>

    ```bash
    [08:43 PM IST 17.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kubectl exec -it two-containers -- /bin/sh -c 'cat /usr/share/nginx/html/index.html'
    Defaulted container "nginx-container" out of: nginx-container, busybox-container
    Hello from the busybox container
    ```
    </p>
    <p>

    ```bash
    [08:55 PM IST 17.10.2021 ☸ 127.0.0.1:59140 📁 ~ 𖦥 ]
    ┗━ ॐ  kl two-containers -c sidecar
    Hello from the busybox container
    ```
    </p>
    </details>
