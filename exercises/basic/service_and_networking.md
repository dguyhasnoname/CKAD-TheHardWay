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
    Create the network policy in default namespace. Save this config in netpol.yaml file.
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
    Apply above network policy.
    <p>

    ```bash
    kubectl apply -f netpol.yaml
    ```
    </p>
    Launch a busybox pod and try to access the service.
    <p>

    ```bash
    kubectl run busybox --rm -ti --image=busybox -- /bin/sh
    ```
    </p>
    Command to access the servie.
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

    Create the pod with appropriate labels and then run the command to access the service.
    <p>

    ```bash
    kubectl run busybox --rm -ti --labels="access=true" --image=busybox -- /bin/sh
    ```
    </p>
    Command to access the service.
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

3. Run below command:

    ```
    kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/basic/sn-ns-netpol-3.yaml
    ```

    Above command creates 3 namespaces `manila`, `jakarta` and `seoul`. A nginx deployment gets created in `manila` namespace. Expose the deployment `nginx` in `manila` namespace on port `80`. Create a network policy so that pods running only in `jakarta` and `seoul` namespace can access the `nginx` deployment pods running in `manila` namespace. Verify the accessibility by running pods in `jakarta` and `seoul` namespaces. Also ensure that any pod running in `default` namespace is not able to access the `nginx` service in `manila` namespace.

4. Run a pod with name `apiserver` in default namespace, marked with labels `app=bookstore` and `role=api`. Expose it on port `80`. Create a netpol to restrict the access only to other pods (e.g. other microservices) running with label `app=bookstore`.

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


5. Run a pod named `web` in default ns with image `nginx` with label `app=web` and expose it on port `80`. Create two namespaces `dev` and `prod` with labels `purpose=testing` and `purpose=production`. Create a netpol to restrict the access to pod `web` for pods only running in `dev` namespace. The netpol should only allow traffic form pods running in namespace with label `purpose=production`.

    <details><summary>steps</summary>
    Create a pod with name web in default ns, image nginx, label app=web and expose it on port 80.
    <p>

    ```bash
    kubectl run web --image=nginx --labels=app=web --expose --port 80

    ```
    </p>
    Create two namespaces dev and prod with labels purpose=testing and purpose=production.
    <p>

    ```bash
    kubectl create ns dev --labels purpose=testing
    kubectl create ns prod --labels purpose=production
    ```
    </p>
    Create a netpol to restrict the access only to pods running in dev namespace.
    <p>

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: web-allow-prod
    spec:
      podSelector:
        matchLabels:
          app: web
      ingress:
      - from:
        - namespaceSelector:
            matchLabels:
              purpose: production
    ```
    </p>
    Apply the netpol yaml.
    <p>

    ```bash
    kubectl apply -f web-allow-prod.yaml
    ```

    </p>
    </details>

    <details><summary>verify</summary>
    Test the Network Policy is blocking the traffic, by running a Pod in dev namespace:

    <p>

    ```bash
    $ kubectl run test-1 --namespace=dev --rm -i -t --image=alpine -- sh
    If you don't see a command prompt, try pressing enter.
    / # wget -qO- --timeout=2 http://web.default
    wget: download timed out

    ```
    </p>

    Test the Network Policy is allowing the traffic, by running a Pod in prod namespace:

    <p>

    ```bash
    $ kubectl run test-2 --namespace=prod --rm -i -t --image=alpine -- sh
    If you don't see a command prompt, try pressing enter.
    / # wget -qO- --timeout=2 http://web.default
    <!DOCTYPE html>
    <html>
    <head>
    ...
    ```
    </p>
    </details>

6. Create a netpol which has a combined NetworkPolicy that has the list of microservices that are allowed to connect to an application which. It should allow ingress from workloads with labels `role=search`, `role=api` and `role=web` irrespective of the namespaces in which the workloads are running.

    <details><summary>steps</summary>

    Create the netpol.
    <p>

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: redis-allow-services
    spec:
      podSelector:
        matchLabels:
          app: bookstore
          role: db
      ingress:
      - from:
        - podSelector:
            matchLabels:
              app: bookstore
              role: search
        - podSelector:
            matchLabels:
              app: bookstore
              role: api
        - podSelector:
            matchLabels:
              app: inventory
              role: web
    ```

    Apply the netpol yaml.
    <p>

    ```bash
    $ kubectl apply -f redis-allow-services.yaml
    ```
    </p>

    </details>

    <details><summary>verify</summary>

    Test the Network Policy is allowing the traffic, by running a Pod in default namespace:

    <p>

    ```bash
    $ kubectl run test-3 --labels=app=inventory,role=web --rm -i -t --image=alpine -- sh

    / # nc -v -w 2 db 6379
    db (10.59.242.200:6379) open
    ```
    </p>
    Pods with labels not matching will not be able to connect
    <p>

    ```bash
    $ kubectl run test-4 --labels=app=other --rm -i -t --image=alpine -- sh

    / # nc -v -w 2 db 6379
    nc: db (10.59.252.83:6379): Operation timed out

    ```
    </p>
    </details>




