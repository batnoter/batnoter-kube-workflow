The deploy workflow is used to deploy the application to kubernetes cluster.

### Setup kubectl with github action
The workflow uses [https://github.com/Azure/k8s-set-context](https://github.com/Azure/k8s-set-context) for installing `kubectl` and also configure the context.

The action requires the `kubeconfig` property set with the string value of k8s cluster config. So simply create the secret with name `KUBECONFIG` and copy the value of the kube config from `~/.kube/config` to the secret value. 

**Note:** Since the `~/.kube/config` may have multiple k8s cluster configurations. So its always better to download the k8s cluster specific config from cloud provider and use it's value as the value of `KUBECONFIG` secret variable. 

### Setting secrets for deploy job
Since the deploy job makes use of environments. We can create the secrets inside the environment settings. Following steps will help you setup the secrets required for deployment.
The workflow makes use of [https://github.com/Azure/k8s-create-secret](https://github.com/Azure/k8s-create-secret) action.
* Go to `Repo Settings > Environments > Production`
* Creating secrets of type `generic` or `kubernetes.io/dockerconfigjson` is easy. If you already have the secret in k8s then just run this command - `kubectl get secret <SECRET_NAME> -o jsonpath='{.data}' -n <NAMESPACE>` then copy the output to the secret value inside that specific environment secret on github. Then provide this secret to `k8s-create-secret` action as a value of `data` attribute.
* To get the kubernetes config of the current context, run below command
  ```
  kubectl config view --minify --flatten
  ```
  Then copy the output to `KUBECONFIG` secret.

Make sure you have setup below secrets inside `Production` environment configurations
- AUTH_SECRET
- KUBECONFIG
- POSTGRES_SECRET
- REGCRED_SECRET

**NOTE:** For `kubernetes.io/dockerconfigjson` type secret make sure you specify the type attribute as `kubernetes.io/dockerconfigjson` and not `generic`

### Using a new k8s cluster for deployment
* Make sure that you have updated the value of `KUBECONFIG` environment secret with the kubeconfig of newly created cluster.
* If you own a digital ocean cluster then you can simply download the kube config file cluster overview. The contents of this file can be copied directly to `KUBECONFIG` github secret
* After the first deployment on new cluster, update the domain's `A` record to point to the newly created load balancer.
