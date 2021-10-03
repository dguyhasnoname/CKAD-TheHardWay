# CKAD-TheHardWay [under development]

CKAD exercises in this repo are of two different difficulty levels:

1. [basic](./exercises/basic)
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

1. Tune your vim editor

    ```
    set tabstop=2
    set expandtab
    set shiftwidth=2
    ```

2. Setup your aliases

    ```bash
    alias 'kg=kubectl get'
    alias 'kd=kubectl describe'
    alias 'kc=kubectl create'
    alias 'kr=kubectl run'
    alias 'ka=kubectl apply'
    alias 'kdel=kubectl delete --force --grace-period=0'
    alias 'kl=kubectl logs'
    alias 'kgy=kubectl get -o yaml'
    alias 'kcd=kubectl --dry-run=client -o yaml create'
    alias 'krd=kubectl --dry-run=client -o yaml run'
    alias 'ked=kubectl --dry-run=client -o yaml expose'
    alias 'krbb=kubectl run bb --rm -it --image=busybox --restart=Never'
    alias 'krna=kubectl run na --rm -it --image=nginx:alpine --restart=Never'
    alias 'kn=kubectl config set-context --current --namespace'
    ```

    You can find more aliases in [kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## PRs

Feel free to PR and edit/add questions and solutions, but please floow to the existing format and make sure you have tested your changes before submitting a PR.

If CKAD-TheHardWay repo has helped you in any way learning Kubernetes, feel free to post on discussions, share across your blogs or buy me a coffee!

<a href="https://buymeacoffee.com/dguyhasnoname" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>