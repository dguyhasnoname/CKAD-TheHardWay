# CKAD-TheHardWay [under development]

CKAD exercises in this repo are of two different difficulty levels:

1. [basic](./exercises/basic/about.md)
2. advanced

CKAD Curriculum: [link](https://github.com/cncf/curriculum)

## Contents:

1. Application desgin and build - 20%
    <details><summary>details</summary>
    <p>

    - Define, build and modify container images 
    - Understand Jobs and CronJobs 
    - Understand multi-container Pod design patterns (e.g. sidecar, init and others) 
    - Utilize persistent and ephemeral volumes 

    </p>
    </details>
2. Application deployment - 20%
    <details><summary>details</summary>
    <p>

    - Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary) 
    - Understand Deployments and how to perform rolling updates 
    - Use the Helm package manager to deploy existing packages 
    </p>
    </details>
3. Application observability and maintenance - 15%
    <details><summary>details</summary>
    <p>

    - Understand API deprecations 
    - Implement probes and health checks 
    - Use provided tools to monitor Kubernetes applications 
    - Utilize container logs 
    - Debugging in Kubernetes 
    </p>
4. Application Environment, Configuration and Security - 25%
    <details><summary>details</summary>
    <p>

    - Discover and use resources that extend Kubernetes (CRD)
    - Understand authentication, authorization and admission control
    - Understanding and defining resource requirements, limits and quotas
    - Understand ConfigMaps
    - Create & consume Secrets
    - Understand ServiceAccounts
    - Understand SecurityContexts

    </p>
5. Services and Networking - 20%

    <details><summary>details</summary>
    <p>

    - Demonstrate basic understanding of NetworkPolicies
    - Provide and troubleshoot access to applications via services
    - Use Ingress rules to expose applications

    </p>


## Lab setup

### Pre-requisitess

1. Working k8s cluster

    If you do not have running k8s cluster, setup it by using [minikube](https://minikube.sigs.k8s.io/docs/start/)

2. kubectl & helm

    Install CLI kubectl: [steps](https://kubernetes.io/docs/tasks/tools/#kubectl)

    Install helm: [steps](https://helm.sh/docs/intro/install/)

3. kubeconfig of your k8s cluster

Once you have working k8s cluster, please get the kubeconfig for the cluster which have appropriate rights. If you are using minikube, the default kubeconfig will be picked automatically.

### Setting up the lab (under development)

    kubectl apply -f lab_setup.yaml

Running above command should setup the lab in your cluster.

## Tips and tricks

1. Tune your vim editor skills

    - Create/update the file ~/.vimrc with the following content:

        ```
        set tabstop=2       # tab width to 2 spaces
        set expandtab       # expand tabs to spaces
        set shiftwidth=2:   # columns of whitespace for indentation
        ```

        Since this is going to save time in indentation and help create error free yamls, here is a tip to remember the setting:
        - remember TES(reverse of set)
            - T for tabstop
            - E for expandtab
            - S for shiftwidth

    - Multiple lines indentation:
        1. Press `V` to enter VISUAL LINE mode.
        2. Select the text you wish to indent but using either the cursor keys or the `j` and `k` keys.
        3. To indent press `>`(shift + .).

        You can then repeat the indentation by using the `>` key. This can be useful when we copy yamls from kuberntes documentations.

    - Other vim shortcuts:

        ```text
        w : jump to the next word
        W : jump to the previous word
        e : jump to the end of the current word
        E : jump to the beginning of the next word
        $ : jump to the end of the current line
        0 : jump to the beginning of the current line
        ^ : jump to the beginning of the previous line

        G : end of the file
        g : beginning of the file

        dG : delete till the end of the file
        d0 : delete till the beginning of the file
        4dd : delete the next 4 lines
        d: delete the current line
        D: delete till end of line
        dw: delete the current word

        f + <char> : jump to the next occurrence of <char>
        F + <char> : jump to the previous occurrence of <char>

        yy: copy the current line
        3yy: copy the next 3 lines
        ```

2. Setup your aliases

    Use below aliases save time in typing:

    ```bash
    alias 'kg=kubectl get'
    alias 'kd=kubectl describe'
    alias 'kc=kubectl create'                                               # useful for creating deployments/services
    alias 'kr=kubectl run'                                                  # useful for creating pods
    alias 'ka=kubectl apply'                                                # used for creating resources by files
    alias 'kdel=kubectl delete --force --grace-period=0'                    # used for deleting resources quickly
    alias 'kl=kubectl logs'
    alias 'kgy=kubectl get -o yaml'
    alias 'kcd=kubectl --dry-run=client -o yaml create'                     # used for creating resources yaml
    alias 'krd=kubectl --dry-run=client -o yaml run'                        # used for creating pods yaml
    alias 'ked=kubectl --dry-run=client -o yaml expose'                     # used for creating services yaml
    alias 'krb=kubectl run bb --rm -it --image=busybox --restart=Never'     # used for creating pod with busybox
    alias 'krc=kubectl run na --rm -it --image=nginx:alpine --restart=Never'# used for creating pod for curl
    alias 'kn=kubectl config set-context --current --namespace'             # used for setting namespace
    ```

    These aliases can help to save lot of time, especially the `-dry-run=client -o yaml`. You can find more aliases in [kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

    You can also use `kubectl completion` to get the completion for kubectl.

3. Try to use imperative kubectl commands to create resources yaml. You can use `kubectl get` to get the list of resources.

4. Use the bookmarks for k8s documentation to jump to the desired section. CKA, CKAD and CKS exams let you use the official documentation. Only tab is allowed to be opened at a time.

    - [Bookmark I used to pass CKAD and CKA](./cka_ckad_bookmarks.html)

    You can create your own bookmarks to be more comfortable. Try creating a bookmark for each type of example so that it can be accessed during exam quickly.

## PRs

Feel free to PR and edit/add questions and solutions, but please follow to the existing format and make sure you have tested your changes before submitting a PR.

If CKAD-TheHardWay repo has helped you in any way learning Kubernetes, feel free to post on discussions, star it, fork it and share across your blogs or buy me a coffee!

<a href="https://buymeacoffee.com/dguyhasnoname" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>
