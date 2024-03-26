# Challenge 2: https://kodekloud.com/topic/lab-challenge-2/

## BEFORE:

![image](https://i.imgur.com/biKzhJn.png)

## TASK:
A number of applications have been deployed in the dev, staging and prod namespaces. There are a few security issues with these applications.

Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the applications. Once done click on the Check button to validate your work.

1) Info on "Dockerfile (webapp)"
-    Run as non root(instead, use correct application user)
-    Avoid exposing unnecessary ports
-    Avoid copying the 'Dockerfile' and other unnecessary files and directories in to the image. Move the required files and directories (app.py, requirements.txt and the templates directory) to a subdirectory called 'app' under 'webapp' and update the COPY instruction in the 'Dockerfile' accordingly.
-    Once the security issues are fixed, rebuild this image locally with the tag 'kodekloud/webapp-color:stable'

2) Info on "kubesec"
-    Before you proceed, make sure to fix all issues specified in the 'Dockerfile(webapp)' section of the architecture diagram. Using the 'kubesec' tool, which is already installed on the node, identify and fix the following issues:
-    Fix issues with the '/root/dev-webapp.yaml' file which was used to deploy the 'dev-webapp' pod in the 'dev' namespace.
-    Redeploy the 'dev-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'
-    Fix issues with the '/root/staging-webapp.yaml' file which was used to deploy the 'staging-webapp' pod in the 'staging' namespace.
-    Redeploy the 'staging-webapp' pod once issues are fixed with the image 'kodekloud/webapp-color:stable'

3) Info on "dev-webapp"
-    Ensure that the pod 'dev-webapp' is immutable:
-    This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
-    Image used: 'kodekloud/webapp-color:stable'
-    Redeploy the pod as per the above recommendations and make sure that the application is up.

4) Info on "staging-webapp"
-    Ensure that the pod 'staging-webapp' is immutable:
-    This pod can be accessed using the 'kubectl exec' command. We want to make sure that this does not happen. Use a startupProbe to remove all shells (sh and ash) before the container startup. Use 'initialDelaySeconds' and 'periodSeconds' of '5'. Hint: For this to work you would have to run the container as root!
-    Image used: 'kodekloud/webapp-color:stable'
-    Redeploy the pod as per the above recommendations and make sure that the application is up.

5) Info on "prod-web"
-    The deployment has a secret hardcoded. Instead, create a secret called 'prod-db' for all the hardcoded values and consume the secret values as environment variables within the deployment.

6) Info on "prod-netpol"
-    Use a network policy called 'prod-netpol' that will only allow traffic only within the 'prod' namespace. All the traffic from other namespaces should be denied.

## SOLUTION:
```
root@controlplane ~ ➜  echo $SHELL
/bin/bash

root@controlplane ~ ➜  source <(kubectl completion bash) 
root@controlplane ~ ➜  echo "source <(kubectl completion bash)" >> ~/.bashrc 
root@controlplane ~ ➜  alias k=kubectl
root@controlplane ~ ➜  complete -o default -F __start_kubectl k

root@controlplane ~ ➜  export dr="--dry-run=client -oyaml"
root@controlplane ~ ➜  export now="--force --grace-period=0"

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  pwd
/root
root@controlplane ~ ➜  ls -l
total 12
-rw-rw-rw- 1 root root 1646 Nov 14 13:49 dev-webapp.yaml
-rw-rw-rw- 1 root root 1654 Nov 14 13:49 staging-webapp.yaml
drwxrwxrwx 3 root root 4096 Nov 14 13:49 webapp

root@controlplane ~ ➜  cd webapp/
root@controlplane ~/webapp via 🐍 v3.6.9 ➜  ls -l
total 16
-rw-rw-rw- 1 root root 2259 Nov 14 13:49 app.py
-rw-rw-rw- 1 root root  301 Nov 14 13:49 Dockerfile
-rw-rw-rw- 1 root root    5 Nov 14 13:49 requirements.txt
drwxrwxrwx 2 root root 4096 Nov 14 13:49 templates

root@controlplane ~/webapp via 🐍 v3.6.9 ✖ podman
-su: podman: command not found

root@controlplane ~/webapp via 🐍 v3.6.9 ✖ docker
Usage:  docker [OPTIONS] COMMAND
A self-sufficient runtime for containers  [....]

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
busybox                              latest              66ba00ad3de8        14 months ago       4.87MB
nginx                                latest              1403e55ab369        15 months ago       142MB
nginx                                alpine              1e415454686a        15 months ago       40.7MB
k8s.gcr.io/kube-apiserver            v1.23.0             e6bf5ddd4098        2 years ago         135MB
k8s.gcr.io/kube-controller-manager   v1.23.0             37c6aeb3663b        2 years ago         125MB
k8s.gcr.io/kube-scheduler            v1.23.0             56c5af1d00b5        2 years ago         53.5MB
k8s.gcr.io/kube-proxy                v1.23.0             e03484a90585        2 years ago         112MB
k8s.gcr.io/etcd                      3.5.1-0             25f8c7f3da61        2 years ago         293MB
k8s.gcr.io/coredns/coredns           v1.8.6              a4ca41631cc7        2 years ago         46.8MB
k8s.gcr.io/pause                     3.6                 6270bb605e12        2 years ago         683kB
weaveworks/weave-npc                 2.8.1               7f92d556d4ff        3 years ago         39.3MB
weaveworks/weave-kube                2.8.1               df29c0a4002c        3 years ago         89MB
quay.io/coreos/flannel               v0.13.1-rc1         f03a23d55e57        3 years ago         64.6MB
quay.io/coreos/flannel               v0.12.0-amd64       4e9f801d2217        4 years ago         52.8MB
kodekloud/fluent-ui-running          latest              bd30270a8b9a        5 years ago         969MB
kodekloud/webapp-color               latest              32a1ce4c22f2        5 years ago         84.8MB

root@controlplane ~/webapp via 🐍 v3.6.9 ➜  mkdir app
root@controlplane ~/webapp ✖ mv templates/ app/.
root@controlplane ~/webapp via 🐍 v3.6.9 ➜  mv requirements.txt app/.
root@controlplane ~/webapp via 🐍 v3.6.9 ➜  mv app.py app/.

root@controlplane ~/webapp ➜  ls -l
total 8
drwxr-xr-x 3 root root 4096 Mar 24 19:18 app
-rw-rw-rw- 1 root root  301 Nov 14 13:49 Dockerfile

root@controlplane ~/webapp ➜  cd app/
root@controlplane ~/webapp/app via 🐍 v3.6.9 ➜  ls -l
total 12
-rw-rw-rw- 1 root root 2259 Nov 14 13:49 app.py
-rw-rw-rw- 1 root root    5 Nov 14 13:49 requirements.txt
drwxrwxrwx 2 root root 4096 Nov 14 13:49 templates
root@controlplane ~/webapp/app via 🐍 v3.6.9 ➜  cd ..

root@controlplane ~/webapp ➜  cat Dockerfile 
FROM python:3.6-alpine

## Install Flask
RUN pip install flask

## Copy All files to /opt
COPY . /opt/

## Flask app to be exposed on port 8080
EXPOSE 8080

## Flask app to be run as 'worker'
RUN adduser -D worker

## Expose port 22
EXPOSE 22

WORKDIR /opt

USER root

ENTRYPOINT ["python", "app.py"]

root@controlplane ~/webapp ➜  cp Dockerfile ../Dockerfile-bak
root@controlplane ~/webapp ➜  nano Dockerfile 
root@controlplane ~/webapp ➜  cat Dockerfile 
FROM python:3.6-alpine
## Install Flask
RUN pip install flask
## Copy required files to /opt
COPY app/ /opt/
## Flask app to be exposed on port 8080
EXPOSE 8080
## Flask app to be run as 'worker'
RUN adduser -D worker
WORKDIR /opt
USER worker
ENTRYPOINT ["python", "app.py"]

root@controlplane ~/webapp ➜  docker build . --tag kodekloud/webapp-color:stable
Sending build context to Docker daemon  8.192kB
Step 1/8 : FROM python:3.6-alpine
3.6-alpine: Pulling from library/python
59bf1c3509f3: Pull complete 
8786870f2876: Pull complete 
acb0e804800e: Pull complete 
52bedcb3e853: Pull complete 
b064415ed3d7: Pull complete 
Digest: sha256:579978dec4602646fe1262f02b96371779bfb0294e92c91392707fa999c0c989
Status: Downloaded newer image for python:3.6-alpine
 ---> 3a9e80fa4606
Step 2/8 : RUN pip install flask
 ---> Running in acdbe3a1003d
Collecting flask
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting click>=7.1.2
  Downloading click-8.0.4-py3-none-any.whl (97 kB)
Collecting importlib-metadata
  Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp36-cp36m-musllinux_1_1_x86_64.whl (29 kB)
Collecting dataclasses
  Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Collecting zipp>=0.5
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, dataclasses, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 importlib-metadata-4.8.3 itsdangerous-2.0.1 typing-extensions-4.1.1 zipp-3.6.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container acdbe3a1003d
 ---> 42ac2ea4f5c4
Step 3/8 : COPY app/ /opt/
 ---> 88df782a4ece
Step 4/8 : EXPOSE 8080
 ---> Running in 10c4976c393b
Removing intermediate container 10c4976c393b
 ---> 8d6f0bf987e5
Step 5/8 : RUN adduser -D worker
 ---> Running in 4db5b806288f
Removing intermediate container 4db5b806288f
 ---> 75f00ecf061c
Step 6/8 : WORKDIR /opt
 ---> Running in 80048a52d4dd
Removing intermediate container 80048a52d4dd
 ---> dfbaaad2d5da
Step 7/8 : USER worker
 ---> Running in c011a5f819b4
Removing intermediate container c011a5f819b4
 ---> c8b075e24758
Step 8/8 : ENTRYPOINT ["python", "app.py"]
 ---> Running in c1d63986f47b
Removing intermediate container c1d63986f47b
 ---> 7a7de00c6e8f
Successfully built 7a7de00c6e8f
Successfully tagged kodekloud/webapp-color:stable

root@controlplane ~/webapp ➜  docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
kodekloud/webapp-color               stable              7a7de00c6e8f        5 seconds ago       51.9MB
busybox                              latest              66ba00ad3de8        14 months ago       4.87MB
nginx                                latest              1403e55ab369        15 months ago       142MB
nginx                                alpine              1e415454686a        15 months ago       40.7MB
k8s.gcr.io/kube-apiserver            v1.23.0             e6bf5ddd4098        2 years ago         135MB
k8s.gcr.io/kube-scheduler            v1.23.0             56c5af1d00b5        2 years ago         53.5MB
k8s.gcr.io/kube-controller-manager   v1.23.0             37c6aeb3663b        2 years ago         125MB
k8s.gcr.io/kube-proxy                v1.23.0             e03484a90585        2 years ago         112MB
python                               3.6-alpine          3a9e80fa4606        2 years ago         40.7MB
k8s.gcr.io/etcd                      3.5.1-0             25f8c7f3da61        2 years ago         293MB
k8s.gcr.io/coredns/coredns           v1.8.6              a4ca41631cc7        2 years ago         46.8MB
k8s.gcr.io/pause                     3.6                 6270bb605e12        2 years ago         683kB
weaveworks/weave-npc                 2.8.1               7f92d556d4ff        3 years ago         39.3MB
weaveworks/weave-kube                2.8.1               df29c0a4002c        3 years ago         89MB
quay.io/coreos/flannel               v0.13.1-rc1         f03a23d55e57        3 years ago         64.6MB
quay.io/coreos/flannel               v0.12.0-amd64       4e9f801d2217        4 years ago         52.8MB
kodekloud/fluent-ui-running          latest              bd30270a8b9a        5 years ago         969MB
kodekloud/webapp-color               latest              32a1ce4c22f2        5 years ago         84.8MB
 
root@controlplane ~/webapp ➜  kubectl get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS        AGE
dev           dev-webapp                             1/1     Running   0               51m
kube-system   coredns-64897985d-892fw                1/1     Running   0               5h11m
kube-system   coredns-64897985d-bzqlm                1/1     Running   0               5h11m
kube-system   etcd-controlplane                      1/1     Running   0               5h12m
kube-system   kube-apiserver-controlplane            1/1     Running   0               5h12m
kube-system   kube-controller-manager-controlplane   1/1     Running   0               5h12m
kube-system   kube-proxy-w25dm                       1/1     Running   0               5h11m
kube-system   kube-proxy-zz9g8                       1/1     Running   0               5h11m
kube-system   kube-scheduler-controlplane            1/1     Running   0               5h12m
kube-system   weave-net-h5bbw                        2/2     Running   1 (5h11m ago)   5h11m
kube-system   weave-net-pnvwg                        2/2     Running   0               5h11m
prod          prod-db                                1/1     Running   0               51m
prod          prod-web-8747d8c84-lk6bh               1/1     Running   0               51m
staging       staging-webapp                         1/1     Running   0               51m

root@controlplane ~/webapp ➜  cat /root/dev-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: dev-webapp
  name: dev-webapp
  namespace: dev
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      allowPrivilegeEscalation: true
      capabilities:
        add:
        - NET_ADMIN
        - SYS_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-z4lvb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-z4lvb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~/webapp ➜  kubesec scan /root/dev-webapp.yaml
[
  {
    "object": "Pod/dev-webapp.dev",
    "valid": true,
    "fileName": "/root/dev-webapp.yaml",
    "message": "Failed with a score of -34 points",
    "score": -34,
    "scoring": {
      "critical": [
        {
          "id": "CapSysAdmin",
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided",
          "points": -30
        },
        {
          "id": "AllowPrivilegeEscalation",
          "selector": "containers[] .securityContext .allowPrivilegeEscalation == true",
          "reason": "",
          "points": -7
        }
      ],
      "passed": [
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]

root@controlplane ~/webapp ✦ ✖ cp /root/dev-webapp.yaml /root/dev-webapp-bak.yaml 

root@controlplane ~/webapp ✦ ➜  nano /root/dev-webapp.yaml

root@controlplane ~/webapp ➜  cat /root/dev-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: dev-webapp
  name: dev-webapp
  namespace: dev
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-z4lvb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-z4lvb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~/webapp ✦ ✖ k delete pod -n dev dev-webapp $now
pod "dev-webapp" deleted

root@controlplane ~/webapp ✦ ➜  k apply -f /root/dev-webapp.yaml 
pod/dev-webapp created

root@controlplane ~/webapp ✦ ➜  k get pod -n dev -w
NAME         READY   STATUS    RESTARTS   AGE
dev-webapp   1/1     Running   0          10s
^C

root@controlplane ~/webapp ✖ cat /root/staging-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: staging-webapp
  name: staging-webapp
  namespace: staging
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
        - SYS_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-v78f2
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-v78f2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~/webapp ✦ ✖ kubesec scan /root/staging-webapp.yaml
[
  {
    "object": "Pod/staging-webapp.staging",
    "valid": true,
    "fileName": "/root/staging-webapp.yaml",
    "message": "Failed with a score of -34 points",
    "score": -34,
    "scoring": {
      "critical": [
        {
          "id": "CapSysAdmin",
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided",
          "points": -30
        },
        {
          "id": "AllowPrivilegeEscalation",
          "selector": "containers[] .securityContext .allowPrivilegeEscalation == true",
          "reason": "",
          "points": -7
        }
      ],
      "passed": [
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]

root@controlplane ~/webapp ✦ ✖ cp /root/staging-webapp.yaml /root/staging-webapp-bak.yaml 

root@controlplane ~/webapp ✦ ➜  nano /root/staging-webapp.yaml

root@controlplane ~/webapp ✦ ➜  cat /root/staging-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: staging-webapp
  name: staging-webapp
  namespace: staging
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-v78f2
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-v78f2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~/webapp ✦ ➜  k delete pod -n staging staging-webapp $now
pod "staging-webapp" deleted

root@controlplane ~/webapp ✦ ➜  k apply -f /root/staging-webapp.yaml 
pod/staging-webapp created

root@controlplane ~/webapp ✦ ➜  k get pods -n staging -w
NAME             READY   STATUS    RESTARTS   AGE
staging-webapp   1/1     Running   0          11s
^C

root@controlplane ~/webapp ✦ ➜  cd

root@controlplane ~ ➜  ls -l
total 20
-rw-r--r-- 1 root root 1646 Mar 25 20:08 dev-webapp-bak.yaml
-rw-r--r-- 1 root root 1596 Mar 25 20:20 dev-webapp.yaml
-rw-r--r-- 1 root root 1654 Mar 25 20:15 staging-webapp-bak.yaml
-rw-r--r-- 1 root root 1604 Mar 25 20:22 staging-webapp.yaml
-rw-r--r-- 1 root root  301 Mar 24 19:23 Dockerfile-bak
drwxrwxrwx 3 root root 4096 Mar 25 20:04 webapp

root@controlplane ~ ➜  cp dev-webapp.yaml dev-webapp-bak-2.yaml 

root@controlplane ~ ➜  kubectl exec -it -n dev dev-webapp -- sh
/opt # echo $SHELL
/opt # echo $0
sh
/opt # whereis sh
sh: whereis: not found
/opt # ls -l /bin/sh
lrwxrwxrwx    1 root     root            12 Sep 11  2018 /bin/sh -> /bin/busybox
/opt # cat /etc/shells
# valid login shells
/bin/sh
/bin/ash
/bin # exit
root@controlplane ~ ➜  whereis sh
sh: /bin/sh /bin/sh.distrib /usr/share/man/man1/sh.1.gz /usr/share/man/man1/sh.1posix.gz

root@controlplane ~ ➜  nano dev-webapp.yaml 
root@controlplane ~ ➜  cat dev-webapp.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: dev-webapp
  name: dev-webapp
  namespace: dev
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    securityContext: 
      readOnlyRootFilesystem: true
    image: kodekloud/webapp-color:stable
    startupProbe:
      exec:
        command: ['sh', '-c', 'rm -f /bin/ash && rm -f /bin/sh']
      initialDelaySeconds: 5
      periodSeconds: 5
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-z4lvb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-z4lvb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~ ✦ ➜  kubectl delete pod -n dev dev-webapp $now
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "dev-webapp" force deleted

root@controlplane ~ ✦ ➜  k apply -f dev-webapp.yaml
pod/dev-webapp created

root@controlplane ~ ✦ ➜  k get pods -n dev -w
NAME         READY   STATUS    RESTARTS   AGE
dev-webapp   0/1     Running   0          8s
dev-webapp   0/1     Running   0          11s
dev-webapp   1/1     Running   0          11s
^C

root@controlplane ~ ➜  cp staging-webapp.yaml staging-webapp-bak-2.yaml

root@controlplane ~ ➜  kubectl exec -it -n staging staging-webapp -- sh
/opt # echo $0
sh
/opt # cat /etc/shells
# valid login shells
/bin/sh
/bin/ash
/bin # exit

root@controlplane ~ ➜  nano staging-webapp.yaml 
root@controlplane ~ ➜  cat staging-webapp.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: staging-webapp
  name: staging-webapp
  namespace: staging
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    securityContext: 
      readOnlyRootFilesystem: true
    image: kodekloud/webapp-color:stable
    startupProbe:
      exec:
        command: ['sh', '-c', 'rm -f /bin/ash && rm -f /bin/sh']
      initialDelaySeconds: 5
      periodSeconds: 5
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      capabilities:
        add:
        - NET_ADMIN
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-v78f2
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-v78f2
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace

root@controlplane ~ ✦ ➜  kubectl delete pod -n staging staging-webapp $now
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "dev-webapp" force deleted

root@controlplane ~ ✦ ➜  k apply -f staging-webapp.yaml
pod/dev-webapp created

root@controlplane ~ ✦ ➜  k get pods -n staging -w
NAME             READY   STATUS    RESTARTS   AGE
staging-webapp   0/1     Running   0          8s
staging-webapp   0/1     Running   0          10s
staging-webapp   1/1     Running   0          10s
^C

root@controlplane ~ ➜  k get deploy -A
NAMESPACE     NAME       READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   coredns    2/2     2            2           153m
prod          prod-web   1/1     1            1           43m

root@controlplane ~ ➜  k get deploy -n prod prod-web -o yaml > deploy-prod-web-bak.yaml

root@controlplane ~ ➜  cat deploy-prod-web-bak.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-03-25T11:11:03Z"
  generation: 1
  labels:
    name: prod-web
  name: prod-web
  namespace: prod
  resourceVersion: "12351"
  uid: 7ae9d078-07e6-4d9e-9546-3683674548c0
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: prod-web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: prod-web
      name: dev-web
    spec:
      containers:
      - env:
        - name: DB_Host
          value: prod-db
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - all
          readOnlyRootFilesystem: true
          runAsUser: 10001
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2024-03-25T11:11:12Z"
    lastUpdateTime: "2024-03-25T11:11:12Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2024-03-25T11:11:03Z"
    lastUpdateTime: "2024-03-25T11:11:12Z"
    message: ReplicaSet "prod-web-8747d8c84" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1

root@controlplane ~ ➜  k get sa -A
NAMESPACE         NAME                                 SECRETS   AGE
default           default                              1         109m
dev               default                              1         68s
kube-node-lease   default                              1         109m
kube-public       default                              1         109m
kube-system       attachdetach-controller              1         109m
kube-system       bootstrap-signer                     1         109m
kube-system       certificate-controller               1         109m
kube-system       clusterrole-aggregation-controller   1         109m
kube-system       coredns                              1         109m
kube-system       cronjob-controller                   1         109m
kube-system       daemon-set-controller                1         109m
kube-system       default                              1         109m
kube-system       deployment-controller                1         109m
kube-system       disruption-controller                1         109m
kube-system       endpoint-controller                  1         109m
kube-system       endpointslice-controller             1         109m
kube-system       endpointslicemirroring-controller    1         109m
kube-system       ephemeral-volume-controller          1         109m
kube-system       expand-controller                    1         109m
kube-system       generic-garbage-collector            1         109m
kube-system       horizontal-pod-autoscaler            1         109m
kube-system       job-controller                       1         109m
kube-system       kube-proxy                           1         109m
kube-system       namespace-controller                 1         109m
kube-system       node-controller                      1         109m
kube-system       persistent-volume-binder             1         109m
kube-system       pod-garbage-collector                1         109m
kube-system       pv-protection-controller             1         109m
kube-system       pvc-protection-controller            1         109m
kube-system       replicaset-controller                1         109m
kube-system       replication-controller               1         109m
kube-system       resourcequota-controller             1         109m
kube-system       root-ca-cert-publisher               1         109m
kube-system       service-account-controller           1         109m
kube-system       service-controller                   1         109m
kube-system       statefulset-controller               1         109m
kube-system       token-cleaner                        1         109m
kube-system       ttl-after-finished-controller        1         109m
kube-system       ttl-controller                       1         109m
kube-system       weave-net                            1         109m
prod              default                              1         68s
staging           default                              1         68s

root@controlplane ~ ➜  cat deploy-prod-web-bak.yaml | grep -i secret

root@controlplane ~ ✖ cat deploy-prod-web-bak.yaml | grep -i pass
        - name: DB_Password

root@controlplane ~ ➜  cat deploy-prod-web-bak.yaml | grep -i pass -B5 -A5
      - env:
        - name: DB_Host
          value: prod-db
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:

-> to consume the secret values as environment variables, relevant link: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-a-container-environment-variable-with-data-from-a-single-secret

root@controlplane ~ ➜  kubectl create secret generic -n prod prod-db --from-literal DB_Host=prod-db --from-literal DB_User=root --from-literal DB_Password=paswrd
secret/prod-db created

root@controlplane ~ ➜  kubectl edit deploy -n prod prod-web
deployment.apps/prod-web edited
root@controlplane ~ ➜  k get deploy -n prod prod-web -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  creationTimestamp: "2024-03-25T14:49:20Z"
  generation: 2
  labels:
    name: prod-web
  name: prod-web
  namespace: prod
  resourceVersion: "8346"
  uid: 3573d2e0-370c-4931-8a40-0bc24548bf4e
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: prod-web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: prod-web
      name: dev-web
    spec:
      containers:
      - env:
        - name: DB_User
          valueFrom:
            secretKeyRef:
              key: DB_User
              name: prod-db
        - name: DB_Host
          valueFrom:
            secretKeyRef:
              key: DB_Host
              name: prod-db
        - name: DB_Password
          valueFrom:
            secretKeyRef:
              key: DB_Password
              name: prod-db
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - all
          readOnlyRootFilesystem: true
          runAsUser: 10001
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2024-03-25T14:49:35Z"
    lastUpdateTime: "2024-03-25T14:49:35Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2024-03-25T14:49:20Z"
    lastUpdateTime: "2024-03-25T14:59:32Z"
    message: ReplicaSet "prod-web-69f67c66cc" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 2
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1

root@controlplane ~ ✦ ➜  k get deploy -n prod
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
prod-web   1/1     1            1           9m45s

root@controlplane ~ ✦ ➜  k get deploy -n prod -w
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
prod-web   1/1     1            1           11s
^C
root@controlplane ~ ✦ ✖ k get pod -n prod -w
NAME                        READY   STATUS    RESTARTS   AGE
prod-db                     1/1     Running   0          8m19s
prod-web-69f67c66cc-dwk7m   1/1     Running   0          104s
^C

root@controlplane ~ ✖ kubectl exec -n prod -it prod-web-69f67c66cc-dwk7m -- printenv
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=prod-web-69f67c66cc-dwk7m
TERM=xterm
DB_User=root
DB_Host=prod-db
DB_Password=paswrd
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PROD_DB_PORT=tcp://10.105.20.47:3306
PROD_DB_PORT_3306_TCP=tcp://10.105.20.47:3306
PROD_DB_PORT_3306_TCP_PROTO=tcp
PROD_WEB_PORT=tcp://10.103.195.102:8080
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
PROD_WEB_SERVICE_PORT=8080
PROD_DB_SERVICE_PORT=3306
PROD_DB_PORT_3306_TCP_ADDR=10.105.20.47
PROD_WEB_SERVICE_HOST=10.103.195.102
PROD_WEB_PORT_8080_TCP=tcp://10.103.195.102:8080
PROD_WEB_PORT_8080_TCP_PROTO=tcp
PROD_WEB_PORT_8080_TCP_ADDR=10.103.195.102
KUBERNETES_SERVICE_PORT_HTTPS=443
PROD_DB_SERVICE_HOST=10.105.20.47
PROD_DB_PORT_3306_TCP_PORT=3306
PROD_WEB_PORT_8080_TCP_PORT=8080
LANG=C.UTF-8
GPG_KEY=0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
PYTHON_VERSION=3.7.0
PYTHON_PIP_VERSION=18.0
HOME=/

root@controlplane ~ ✦ ➜  k get pods -A --show-labels | grep -i dns
kube-system   coredns-64897985d-89z5h                1/1     Running   0              130m   k8s-app=kube-dns,pod-template-hash=64897985d
kube-system   coredns-64897985d-k47jk                1/1     Running   0              130m   k8s-app=kube-dns,pod-template-hash=64897985d

root@controlplane ~ ✖ k get ns --show-labels
NAME              STATUS   AGE    LABELS
default           Active   164m   kubernetes.io/metadata.name=default
dev               Active   15m    kubernetes.io/metadata.name=dev
kube-node-lease   Active   164m   kubernetes.io/metadata.name=kube-node-lease
kube-public       Active   164m   kubernetes.io/metadata.name=kube-public
kube-system       Active   164m   kubernetes.io/metadata.name=kube-system
prod              Active   16m    kubernetes.io/metadata.name=prod
staging           Active   15m    kubernetes.io/metadata.name=staging

-> it is expected to add DNS for name resolution, see message https://discord.com/channels/1197109182172770304/1199643628574887986/1221794518635118682 , check with https://editor.networkpolicy.io/

root@controlplane ~ ➜  nano netpol-prod-netpol.yaml

root@controlplane ~ ➜  cat netpol-prod-netpol.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: prod-netpol
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - podSelector: {}
  egress:
    - to:
      - podSelector: {}
    - to:
      - namespaceSelector: 
          matchLabels:
            kubernetes.io/metadata.name: kube-system
        podSelector:
          matchLabels:
            k8s-app: kube-dns
      ports:
        - port: 53
          protocol: UDP

root@controlplane ~ ➜  k apply -f netpol-prod-netpol.yaml 
networkpolicy.networking.k8s.io/prod-netpol created
```

## AFTER:

![image](https://i.imgur.com/DxwbooS.png)












