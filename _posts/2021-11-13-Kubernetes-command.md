---
title:  Kubernetes command
categories:
  - Cloud computing
tags:
  - Kubernetes

---
Content

{% include toc %}

# deployment

* edit the deployment config file

`kubectl edit deployment '<resource>' -n '<namespace>'`

`kubectl edit deployment ping-test -n prj-ipdd-test`

`kubectl --kubeconfig ~/.kube/config-c2 edit deployment ping-test -n prj-ipdd-test`

# config

* view the current config

`kubectl config view`

* Reach for pods with specific config files

`kubectl --kubeconfig ~/.kube/config-c2 get pods`



# pod

* view CPU and memory usage

`kubectl top pod '<pod name>' -n '<namespace>'`

`kubectl top pod ping-test-5db46bbf74-fz2kq -n prj-ipdd-test`

* pod bash

`kubectl exec -it gray-gateway-gw-productops-public-backend-deployment-86c46vmbrz -n prj-apigateway-b bash`

* search pods in specific namespace

`kubectl get pods -n prj-ipdd-test -o wide`



# log

* view the pod log

`kubectl log gray-gateway-gw-productops-public-backend-deployment-86c469vnvc -n prj-apigateway-b -c gray-gateway-gw-productops-public-backend -f`
