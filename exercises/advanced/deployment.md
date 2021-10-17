# Application deployment - 20%

1. Run below command:

  ```bash
  kubectl apply -f https://raw.githubusercontent.com/dguyhasnoname/CKAD-TheHardWay/master/lab-setup/manifests/advanced/deployment-question-1.yaml
  ```

  Above command creates a deployment named `nginx` in namespace `bingo`. The deployment is expected to run `5` replicas of the nginx pod, but only `two` are running. Identify the reason and fix it.


