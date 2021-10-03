# CKAD-TheHardWay [under development]

CKAD exercises in this repo are of two different difficulty levels:

1. [basic](./exercises/basic/about.md)
2. advanced

CKAD Curriculum: [link](https://github.com/cncf/curriculum)

## Contents:

1. Application desgin and build - 20%
2. Application deployment - 20%
3. Application observability and maintenance - 15%
4. Application Environment, Configuration and Security - 25%
5. Services and Networking - 20%

## Lab setup

### Pre-requisitess

1. Working k8s cluster

    If you do not have running k8s cluster, setup it by using [minikube](https://minikube.sigs.k8s.io/docs/start/)

2. kubectl & helm

    Install CLI kubectl: [steps](https://kubernetes.io/docs/tasks/tools/#kubectl)

    Install helm: [steps](https://helm.sh/docs/intro/install/)

3. kubeconfig of your k8s cluster

Once you have working k8s cluster, please get the kubeconfig for the cluster which have appropriate rights.

### Setting up the lab

    kubectl apply -f lab_setup.yaml

Running above command should setup the lab in your cluster.

## Tips and tricks

1. Tune your vim editor skills

    - Create the file ~/.vimrc with the following content:

        ```
        set tabstop=2       # tab width to 2 spaces
        set expandtab       # expand tabs to spaces
        set shiftwidth=2:   # columns of whitespace for indentation
        ```

        Since this is going to save time in indentation and help create error free yamls, here is a tip to remember the setting:
        - remember set TES
            - T for tabstop
            - E for expandtab
            - S for shiftwidth

    - Multiple lines indentation:
        1. Press `V` to enter VISUAL LINE mode.
        2. Select the text you wish to indent but using either the cursor keys or the `j` and `k` keys.
        3. To indent press `>`(shift + .).

        You can then repeat the indentation by using the `>` key. This can be useful when we copy yamls from kuberntes documentations.

2. Setup your aliases

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

## PRs

Feel free to PR and edit/add questions and solutions, but please follow to the existing format and make sure you have tested your changes before submitting a PR.

If CKAD-TheHardWay repo has helped you in any way learning Kubernetes, feel free to post on discussions, share across your blogs or buy me a coffee!

<a href="https://buymeacoffee.com/dguyhasnoname" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>