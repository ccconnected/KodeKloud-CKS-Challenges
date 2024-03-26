# Challenge 3: https://kodekloud.com/topic/lab-challenge-3/

## BEFORE:

![image](https://i.imgur.com/6Mh1ssl.png)

## TASK:
This is a two node kubernetes cluster. Using the kube-bench utility, identify and fix all the issues that were reported as failed for the controlplane and the worker node components.
Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the cluster. Once done click on the Check button to validate your work.

kube-bench Install Docs: https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md

1) Info on "kube-bench"
-    Download 'kube-bench' from AquaSec for version v0.6.2 and extract it under '/opt' filesystem. Use the appropriate steps from the kube-bench docs to complete this task.
-    Run 'kube-bench' with config directory set to '/opt/cfg' and '/opt/cfg/config.yaml' as the config file. Redirect the result to '/var/www/html/index.html' file.

2) Info on "kube-apiserver"
-    Ensure that the --profiling argument is set to false
-    Ensure PodSecurityPolicy admission controller is enabled
-    Ensure that the --insecure-port argument is set to 0
-    Ensure that the --audit-log-path argument is set to /var/log/apiserver/audit.log
-    Ensure that the --audit-log-maxage argument is set to 30
-    Ensure that the --audit-log-maxbackup argument is set to 10
-    Ensure that the --audit-log-maxsize argument is set to 100

3) Info on "kube-controller-manager"
-    Ensure that the --profiling argument is set to false

4) Info on "etcd"
-    Correct the etcd data directory ownership

5) Info on "kube-scheduler"
-    Ensure that the --profiling argument is set to false

6) Info on "controlplane" "kubelet"
-    Ensure that the --protect-kernel-defaults argument is set to true (controlplane)

7) Info on "node" "kubelet"
-    Ensure that the --protect-kernel-defaults argument is set to true (node01)

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

root@controlplane ~ ➜  cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.04.6 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.6 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic

root@controlplane ~ ✖ cd /opt
root@controlplane /opt ➜  curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.2/kube-bench_0.6.2_linux_amd64.tar.gz -o kube-bench_0.6.2_linux_amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 7821k  100 7821k    0     0  21.4M      0 --:--:-- --:--:-- --:--:-- 21.4M

root@controlplane /opt ➜  tar -xvf kube-bench_0.6.2_linux_amd64.tar.gz
cfg/ack-1.0/config.yaml
cfg/ack-1.0/controlplane.yaml
cfg/ack-1.0/etcd.yaml
cfg/ack-1.0/managedservices.yaml
cfg/ack-1.0/master.yaml
cfg/ack-1.0/node.yaml
cfg/ack-1.0/policies.yaml
cfg/aks-1.0/config.yaml
cfg/aks-1.0/controlplane.yaml
cfg/aks-1.0/managedservices.yaml
cfg/aks-1.0/master.yaml
cfg/aks-1.0/node.yaml
cfg/aks-1.0/policies.yaml
cfg/cis-1.5/config.yaml
cfg/cis-1.5/controlplane.yaml
cfg/cis-1.5/etcd.yaml
cfg/cis-1.5/master.yaml
cfg/cis-1.5/node.yaml
cfg/cis-1.5/policies.yaml
cfg/cis-1.6/config.yaml
cfg/cis-1.6/controlplane.yaml
cfg/cis-1.6/etcd.yaml
cfg/cis-1.6/master.yaml
cfg/cis-1.6/node.yaml
cfg/cis-1.6/policies.yaml
cfg/config.yaml
cfg/eks-1.0/config.yaml
cfg/eks-1.0/controlplane.yaml
cfg/eks-1.0/managedservices.yaml
cfg/eks-1.0/master.yaml
cfg/eks-1.0/node.yaml
cfg/eks-1.0/policies.yaml
cfg/gke-1.0/config.yaml
cfg/gke-1.0/controlplane.yaml
cfg/gke-1.0/etcd.yaml
cfg/gke-1.0/managedservices.yaml
cfg/gke-1.0/master.yaml
cfg/gke-1.0/node.yaml
cfg/gke-1.0/policies.yaml
cfg/rh-0.7/config.yaml
cfg/rh-0.7/master.yaml
cfg/rh-0.7/node.yaml
cfg/rh-1.0/config.yaml
cfg/rh-1.0/controlplane.yaml
cfg/rh-1.0/etcd.yaml
cfg/rh-1.0/master.yaml
cfg/rh-1.0/node.yaml
cfg/rh-1.0/policies.yaml
kube-bench

root@controlplane /opt ➜  kube-bench
-su: kube-bench: command not found

root@controlplane /opt ✖ ./kube-bench

unable to determine benchmark version: config file is missing 'version_mapping' section

root@controlplane /opt ✖ ./kube-bench --config-dir `pwd`/opt/cfg --config `pwd`/opt/cfg/config.yaml > /var/www/html/index.html
-su: /var/www/html/index.html: No such file or directory

root@controlplane /opt ✖ mkdir -p /var/www/html/

root@controlplane /opt ✖ ./kube-bench --config-dir /opt/cfg --config /opt/cfg/config.yaml > /var/www/html/index.html

root@controlplane /opt ➜  cat /var/www/html/index.html 
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 Master Node Configuration Files
[PASS] 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.2 Ensure that the API server pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.3 Ensure that the controller manager pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.4 Ensure that the controller manager pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.5 Ensure that the scheduler pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.6 Ensure that the scheduler pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.7 Ensure that the etcd pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.8 Ensure that the etcd pod specification file ownership is set to root:root (Automated)
[WARN] 1.1.9 Ensure that the Container Network Interface file permissions are set to 644 or more restrictive (Manual)
[WARN] 1.1.10 Ensure that the Container Network Interface file ownership is set to root:root (Manual)
[PASS] 1.1.11 Ensure that the etcd data directory permissions are set to 700 or more restrictive (Automated)
[FAIL] 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd (Automated)
[PASS] 1.1.13 Ensure that the admin.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.14 Ensure that the admin.conf file ownership is set to root:root (Automated)
[PASS] 1.1.15 Ensure that the scheduler.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.16 Ensure that the scheduler.conf file ownership is set to root:root (Automated)
[PASS] 1.1.17 Ensure that the controller-manager.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.18 Ensure that the controller-manager.conf file ownership is set to root:root (Automated)
[PASS] 1.1.19 Ensure that the Kubernetes PKI directory and file ownership is set to root:root (Automated)
[PASS] 1.1.20 Ensure that the Kubernetes PKI certificate file permissions are set to 644 or more restrictive (Manual)
[PASS] 1.1.21 Ensure that the Kubernetes PKI key file permissions are set to 600 (Manual)
[INFO] 1.2 API Server
[WARN] 1.2.1 Ensure that the --anonymous-auth argument is set to false (Manual)
[PASS] 1.2.2 Ensure that the --basic-auth-file argument is not set (Automated)
[PASS] 1.2.3 Ensure that the --token-auth-file parameter is not set (Automated)
[PASS] 1.2.4 Ensure that the --kubelet-https argument is set to true (Automated)
[PASS] 1.2.5 Ensure that the --kubelet-client-certificate and --kubelet-client-key arguments are set as appropriate (Automated)
[FAIL] 1.2.6 Ensure that the --kubelet-certificate-authority argument is set as appropriate (Automated)
[PASS] 1.2.7 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)
[PASS] 1.2.8 Ensure that the --authorization-mode argument includes Node (Automated)
[PASS] 1.2.9 Ensure that the --authorization-mode argument includes RBAC (Automated)
[WARN] 1.2.10 Ensure that the admission control plugin EventRateLimit is set (Manual)
[PASS] 1.2.11 Ensure that the admission control plugin AlwaysAdmit is not set (Automated)
[WARN] 1.2.12 Ensure that the admission control plugin AlwaysPullImages is set (Manual)
[WARN] 1.2.13 Ensure that the admission control plugin SecurityContextDeny is set if PodSecurityPolicy is not used (Manual)
[PASS] 1.2.14 Ensure that the admission control plugin ServiceAccount is set (Automated)
[PASS] 1.2.15 Ensure that the admission control plugin NamespaceLifecycle is set (Automated)
[FAIL] 1.2.16 Ensure that the admission control plugin PodSecurityPolicy is set (Automated)
[PASS] 1.2.17 Ensure that the admission control plugin NodeRestriction is set (Automated)
[PASS] 1.2.18 Ensure that the --insecure-bind-address argument is not set (Automated)
[FAIL] 1.2.19 Ensure that the --insecure-port argument is set to 0 (Automated)
[PASS] 1.2.20 Ensure that the --secure-port argument is not set to 0 (Automated)
[FAIL] 1.2.21 Ensure that the --profiling argument is set to false (Automated)
[FAIL] 1.2.22 Ensure that the --audit-log-path argument is set (Automated)
[FAIL] 1.2.23 Ensure that the --audit-log-maxage argument is set to 30 or as appropriate (Automated)
[FAIL] 1.2.24 Ensure that the --audit-log-maxbackup argument is set to 10 or as appropriate (Automated)
[FAIL] 1.2.25 Ensure that the --audit-log-maxsize argument is set to 100 or as appropriate (Automated)
[WARN] 1.2.26 Ensure that the --request-timeout argument is set as appropriate (Automated)
[PASS] 1.2.27 Ensure that the --service-account-lookup argument is set to true (Automated)
[PASS] 1.2.28 Ensure that the --service-account-key-file argument is set as appropriate (Automated)
[PASS] 1.2.29 Ensure that the --etcd-certfile and --etcd-keyfile arguments are set as appropriate (Automated)
[PASS] 1.2.30 Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Automated)
[PASS] 1.2.31 Ensure that the --client-ca-file argument is set as appropriate (Automated)
[PASS] 1.2.32 Ensure that the --etcd-cafile argument is set as appropriate (Automated)
[WARN] 1.2.33 Ensure that the --encryption-provider-config argument is set as appropriate (Manual)
[WARN] 1.2.34 Ensure that encryption providers are appropriately configured (Manual)
[WARN] 1.2.35 Ensure that the API Server only makes use of Strong Cryptographic Ciphers (Manual)
[INFO] 1.3 Controller Manager
[WARN] 1.3.1 Ensure that the --terminated-pod-gc-threshold argument is set as appropriate (Manual)
[FAIL] 1.3.2 Ensure that the --profiling argument is set to false (Automated)
[PASS] 1.3.3 Ensure that the --use-service-account-credentials argument is set to true (Automated)
[PASS] 1.3.4 Ensure that the --service-account-private-key-file argument is set as appropriate (Automated)
[PASS] 1.3.5 Ensure that the --root-ca-file argument is set as appropriate (Automated)
[PASS] 1.3.6 Ensure that the RotateKubeletServerCertificate argument is set to true (Automated)
[PASS] 1.3.7 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)
[INFO] 1.4 Scheduler
[FAIL] 1.4.1 Ensure that the --profiling argument is set to false (Automated)
[PASS] 1.4.2 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)

== Remediations master ==
1.1.9 Run the below command (based on the file location on your system) on the master node.
For example,
chmod 644 <path/to/cni/files>

1.1.10 Run the below command (based on the file location on your system) on the master node.
For example,
chown root:root <path/to/cni/files>

1.1.12 On the etcd server node, get the etcd data directory, passed as an argument --data-dir,
from the below command:
ps -ef | grep etcd
Run the below command (based on the etcd data directory found above).
For example, chown etcd:etcd /var/lib/etcd

1.2.1 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--anonymous-auth=false

1.2.6 Follow the Kubernetes documentation and setup the TLS connection between
the apiserver and kubelets. Then, edit the API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml on the master node and set the
--kubelet-certificate-authority parameter to the path to the cert file for the certificate authority.
--kubelet-certificate-authority=<ca-string>

1.2.10 Follow the Kubernetes documentation and set the desired limits in a configuration file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameters.
--enable-admission-plugins=...,EventRateLimit,...
--admission-control-config-file=<path/to/configuration/file>

1.2.12 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to include
AlwaysPullImages.
--enable-admission-plugins=...,AlwaysPullImages,...

1.2.13 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to include
SecurityContextDeny, unless PodSecurityPolicy is already in place.
--enable-admission-plugins=...,SecurityContextDeny,...

1.2.16 Follow the documentation and create Pod Security Policy objects as per your environment.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to a
value that includes PodSecurityPolicy:
--enable-admission-plugins=...,PodSecurityPolicy,...
Then restart the API Server.

1.2.19 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--insecure-port=0

1.2.21 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--profiling=false

1.2.22 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-path parameter to a suitable path and
file where you would like audit logs to be written, for example:
--audit-log-path=/var/log/apiserver/audit.log

1.2.23 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxage parameter to 30 or as an appropriate number of days:
--audit-log-maxage=30

1.2.24 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxbackup parameter to 10 or to an appropriate
value.
--audit-log-maxbackup=10

1.2.25 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxsize parameter to an appropriate size in MB.
For example, to set it as 100 MB:
--audit-log-maxsize=100

1.2.26 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameter as appropriate and if needed.
For example,
--request-timeout=300s

1.2.33 Follow the Kubernetes documentation and configure a EncryptionConfig file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --encryption-provider-config parameter to the path of that file: --encryption-provider-config=</path/to/EncryptionConfig/File>

1.2.34 Follow the Kubernetes documentation and configure a EncryptionConfig file.
In this file, choose aescbc, kms or secretbox as the encryption provider.

1.2.35 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM
_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM
_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM
_SHA384

1.3.1 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example:
--terminated-pod-gc-threshold=10

1.3.2 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the below parameter.
--profiling=false

1.4.1 Edit the Scheduler pod specification file /etc/kubernetes/manifests/kube-scheduler.yaml file
on the master node and set the below parameter.
--profiling=false

== Summary master ==
43 checks PASS
11 checks FAIL
11 checks WARN
0 checks INFO

[INFO] 2 Etcd Node Configuration
[INFO] 2 Etcd Node Configuration Files
[PASS] 2.1 Ensure that the --cert-file and --key-file arguments are set as appropriate (Automated)
[PASS] 2.2 Ensure that the --client-cert-auth argument is set to true (Automated)
[PASS] 2.3 Ensure that the --auto-tls argument is not set to true (Automated)
[PASS] 2.4 Ensure that the --peer-cert-file and --peer-key-file arguments are set as appropriate (Automated)
[PASS] 2.5 Ensure that the --peer-client-cert-auth argument is set to true (Automated)
[PASS] 2.6 Ensure that the --peer-auto-tls argument is not set to true (Automated)
[PASS] 2.7 Ensure that a unique Certificate Authority is used for etcd (Manual)

== Summary etcd ==
7 checks PASS
0 checks FAIL
0 checks WARN
0 checks INFO

[INFO] 3 Control Plane Configuration
[INFO] 3.1 Authentication and Authorization
[WARN] 3.1.1 Client certificate authentication should not be used for users (Manual)
[INFO] 3.2 Logging
[WARN] 3.2.1 Ensure that a minimal audit policy is created (Manual)
[WARN] 3.2.2 Ensure that the audit policy covers key security concerns (Manual)

== Remediations controlplane ==
3.1.1 Alternative mechanisms provided by Kubernetes such as the use of OIDC should be
implemented in place of client certificates.

3.2.1 Create an audit policy file for your cluster.

3.2.2 Consider modification of the audit policy in use on the cluster to include these items, at a
minimum.

== Summary controlplane ==
0 checks PASS
0 checks FAIL
3 checks WARN
0 checks INFO

[INFO] 4 Worker Node Security Configuration
[INFO] 4.1 Worker Node Configuration Files
[PASS] 4.1.1 Ensure that the kubelet service file permissions are set to 644 or more restrictive (Automated)
[PASS] 4.1.2 Ensure that the kubelet service file ownership is set to root:root (Automated)
[PASS] 4.1.3 If proxy kubeconfig file exists ensure permissions are set to 644 or more restrictive (Manual)
[PASS] 4.1.4 Ensure that the proxy kubeconfig file ownership is set to root:root (Manual)
[PASS] 4.1.5 Ensure that the --kubeconfig kubelet.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 4.1.6 Ensure that the --kubeconfig kubelet.conf file ownership is set to root:root (Manual)
[PASS] 4.1.7 Ensure that the certificate authorities file permissions are set to 644 or more restrictive (Manual)
[PASS] 4.1.8 Ensure that the client certificate authorities file ownership is set to root:root (Manual)
[PASS] 4.1.9 Ensure that the kubelet --config configuration file has permissions set to 644 or more restrictive (Automated)
[PASS] 4.1.10 Ensure that the kubelet --config configuration file ownership is set to root:root (Automated)
[INFO] 4.2 Kubelet
[PASS] 4.2.1 Ensure that the anonymous-auth argument is set to false (Automated)
[PASS] 4.2.2 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)
[PASS] 4.2.3 Ensure that the --client-ca-file argument is set as appropriate (Automated)
[PASS] 4.2.4 Ensure that the --read-only-port argument is set to 0 (Manual)
[PASS] 4.2.5 Ensure that the --streaming-connection-idle-timeout argument is not set to 0 (Manual)
[FAIL] 4.2.6 Ensure that the --protect-kernel-defaults argument is set to true (Automated)
[PASS] 4.2.7 Ensure that the --make-iptables-util-chains argument is set to true (Automated)
[PASS] 4.2.8 Ensure that the --hostname-override argument is not set (Manual)
[WARN] 4.2.9 Ensure that the --event-qps argument is set to 0 or a level which ensures appropriate event capture (Manual)
[WARN] 4.2.10 Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Manual)
[PASS] 4.2.11 Ensure that the --rotate-certificates argument is not set to false (Manual)
[PASS] 4.2.12 Verify that the RotateKubeletServerCertificate argument is set to true (Manual)
[WARN] 4.2.13 Ensure that the Kubelet only makes use of Strong Cryptographic Ciphers (Manual)

== Remediations node ==
4.2.6 If using a Kubelet config file, edit the file to set protectKernelDefaults: true.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
--protect-kernel-defaults=true
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

4.2.9 If using a Kubelet config file, edit the file to set eventRecordQPS: to an appropriate level.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

4.2.10 If using a Kubelet config file, edit the file to set tlsCertFile to the location
of the certificate file to use to identify this Kubelet, and tlsPrivateKeyFile
to the location of the corresponding private key file.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameters in KUBELET_CERTIFICATE_ARGS variable.
--tls-cert-file=<path/to/tls-certificate-file>
--tls-private-key-file=<path/to/tls-key-file>
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

4.2.13 If using a Kubelet config file, edit the file to set TLSCipherSuites: to
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256
or to a subset of these values.
If using executable arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the --tls-cipher-suites parameter as follows, or to a subset of these values.
--tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

== Summary node ==
19 checks PASS
1 checks FAIL
3 checks WARN
0 checks INFO

[INFO] 5 Kubernetes Policies
[INFO] 5.1 RBAC and Service Accounts
[WARN] 5.1.1 Ensure that the cluster-admin role is only used where required (Manual)
[WARN] 5.1.2 Minimize access to secrets (Manual)
[WARN] 5.1.3 Minimize wildcard use in Roles and ClusterRoles (Manual)
[WARN] 5.1.4 Minimize access to create pods (Manual)
[WARN] 5.1.5 Ensure that default service accounts are not actively used. (Manual)
[WARN] 5.1.6 Ensure that Service Account Tokens are only mounted where necessary (Manual)
[INFO] 5.2 Pod Security Policies
[WARN] 5.2.1 Minimize the admission of privileged containers (Manual)
[WARN] 5.2.2 Minimize the admission of containers wishing to share the host process ID namespace (Manual)
[WARN] 5.2.3 Minimize the admission of containers wishing to share the host IPC namespace (Manual)
[WARN] 5.2.4 Minimize the admission of containers wishing to share the host network namespace (Manual)
[WARN] 5.2.5 Minimize the admission of containers with allowPrivilegeEscalation (Manual)
[WARN] 5.2.6 Minimize the admission of root containers (Manual)
[WARN] 5.2.7 Minimize the admission of containers with the NET_RAW capability (Manual)
[WARN] 5.2.8 Minimize the admission of containers with added capabilities (Manual)
[WARN] 5.2.9 Minimize the admission of containers with capabilities assigned (Manual)
[INFO] 5.3 Network Policies and CNI
[WARN] 5.3.1 Ensure that the CNI in use supports Network Policies (Manual)
[WARN] 5.3.2 Ensure that all Namespaces have Network Policies defined (Manual)
[INFO] 5.4 Secrets Management
[WARN] 5.4.1 Prefer using secrets as files over secrets as environment variables (Manual)
[WARN] 5.4.2 Consider external secret storage (Manual)
[INFO] 5.5 Extensible Admission Control
[WARN] 5.5.1 Configure Image Provenance using ImagePolicyWebhook admission controller (Manual)
[INFO] 5.7 General Policies
[WARN] 5.7.1 Create administrative boundaries between resources using namespaces (Manual)
[WARN] 5.7.2 Ensure that the seccomp profile is set to docker/default in your pod definitions (Manual)
[WARN] 5.7.3 Apply Security Context to Your Pods and Containers (Manual)
[WARN] 5.7.4 The default namespace should not be used (Manual)

== Remediations policies ==
5.1.1 Identify all clusterrolebindings to the cluster-admin role. Check if they are used and
if they need this role or if they could use a role with fewer privileges.
Where possible, first bind users to a lower privileged role and then remove the
clusterrolebinding to the cluster-admin role :
kubectl delete clusterrolebinding [name]

5.1.2 Where possible, remove get, list and watch access to secret objects in the cluster.

5.1.3 Where possible replace any use of wildcards in clusterroles and roles with specific
objects or actions.

5.1.4 Where possible, remove create access to pod objects in the cluster.

5.1.5 Create explicit service accounts wherever a Kubernetes workload requires specific access
to the Kubernetes API server.
Modify the configuration of each default service account to include this value
automountServiceAccountToken: false

5.1.6 Modify the definition of pods and service accounts which do not need to mount service
account tokens to disable it.

5.2.1 Create a PSP as described in the Kubernetes documentation, ensuring that
the .spec.privileged field is omitted or set to false.

5.2.2 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.hostPID field is omitted or set to false.

5.2.3 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.hostIPC field is omitted or set to false.

5.2.4 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.hostNetwork field is omitted or set to false.

5.2.5 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.allowPrivilegeEscalation field is omitted or set to false.

5.2.6 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.runAsUser.rule is set to either MustRunAsNonRoot or MustRunAs with the range of
UIDs not including 0.

5.2.7 Create a PSP as described in the Kubernetes documentation, ensuring that the
.spec.requiredDropCapabilities is set to include either NET_RAW or ALL.

5.2.8 Ensure that allowedCapabilities is not present in PSPs for the cluster unless
it is set to an empty array.

5.2.9 Review the use of capabilites in applications running on your cluster. Where a namespace
contains applicaions which do not require any Linux capabities to operate consider adding
a PSP which forbids the admission of containers which do not drop all capabilities.

5.3.1 If the CNI plugin in use does not support network policies, consideration should be given to
making use of a different plugin, or finding an alternate mechanism for restricting traffic
in the Kubernetes cluster.

5.3.2 Follow the documentation and create NetworkPolicy objects as you need them.

5.4.1 if possible, rewrite application code to read secrets from mounted secret files, rather than
from environment variables.

5.4.2 Refer to the secrets management options offered by your cloud provider or a third-party
secrets management solution.

5.5.1 Follow the Kubernetes documentation and setup image provenance.

5.7.1 Follow the documentation and create namespaces for objects in your deployment as you need
them.

5.7.2 Seccomp is an alpha feature currently. By default, all alpha features are disabled. So, you
would need to enable alpha features in the apiserver by passing "--feature-
gates=AllAlpha=true" argument.
Edit the /etc/kubernetes/apiserver file on the master node and set the KUBE_API_ARGS
parameter to "--feature-gates=AllAlpha=true"
KUBE_API_ARGS="--feature-gates=AllAlpha=true"
Based on your system, restart the kube-apiserver service. For example:
systemctl restart kube-apiserver.service
Use annotations to enable the docker/default seccomp profile in your pod definitions. An
example is as below:
apiVersion: v1
kind: Pod
metadata:
  name: trustworthy-pod
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: docker/default
spec:
  containers:
    - name: trustworthy-container
      image: sotrustworthy:latest

5.7.3 Follow the Kubernetes documentation and apply security contexts to your pods. For a
suggested list of security contexts, you may refer to the CIS Security Benchmark for Docker
Containers.

5.7.4 Ensure that namespaces are created to allow for appropriate segregation of Kubernetes
resources and that all new resources are created in a specific namespace.

== Summary policies ==
0 checks PASS
0 checks FAIL
24 checks WARN
0 checks INFO

== Summary total ==
69 checks PASS
12 checks FAIL
41 checks WARN
0 checks INFO

root@controlplane /opt ✖ ls -l /etc/kubernetes/manifests
total 20
-rw------- 1 root root 2221 Mar 26 09:03 etcd.yaml
-rw------- 1 root root 4238 Mar 26 11:12 kube-apiserver.yaml
-rw------- 1 root root 3365 Mar 26 09:03 kube-controller-manager.yaml
-rw------- 1 root root 1435 Mar 26 09:03 kube-scheduler.yaml

root@controlplane /opt ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.18.113.9:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.18.113.9
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.23.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.18.113.9
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.18.113.9
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.18.113.9
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

root@controlplane /opt ➜  cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/kube-apiserver-bak.yaml

root@controlplane /opt ➜  k get pods -n kube-system -owide
NAME                                   READY   STATUS    RESTARTS       AGE    IP              NODE           NOMINATED NODE   READINESS GATES
coredns-64897985d-jnrdp                1/1     Running   0              124m   10.244.0.2      controlplane   <none>           <none>
coredns-64897985d-q59zg                1/1     Running   0              124m   10.244.0.3      controlplane   <none>           <none>
etcd-controlplane                      1/1     Running   0              124m   192.18.113.9    controlplane   <none>           <none>
kube-apiserver-controlplane            1/1     Running   0              124m   192.18.113.9    controlplane   <none>           <none>
kube-controller-manager-controlplane   1/1     Running   0              124m   192.18.113.9    controlplane   <none>           <none>
kube-proxy-gflf6                       1/1     Running   0              124m   192.18.113.9    controlplane   <none>           <none>
kube-proxy-q4gnz                       1/1     Running   0              123m   192.18.113.12   node01         <none>           <none>
kube-scheduler-controlplane            1/1     Running   0              124m   192.18.113.9    controlplane   <none>           <none>
weave-net-fqvxx                        2/2     Running   1 (123m ago)   124m   192.18.113.9    controlplane   <none>           <none>
weave-net-t9t98                        2/2     Running   0              123m   192.18.113.12   node01         <none>           <none>

root@controlplane /etc/kubernetes/manifests ➜  kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
      --admission-control strings              Admission is divided into two phases. In the first phase, only mutating admission plugins run. In the second phase, only validating admission plugins run. The names in the below list may represent a validating plugin, a mutating plugin, or both. The order of plugins in which they are passed to this flag does not matter. Comma-delimited list of: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass, DefaultStorageClass, DefaultTolerationSeconds, DenyServiceExternalIPs, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodSecurity, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, RuntimeClass, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, TaintNodesByCondition, ValidatingAdmissionWebhook. (DEPRECATED: Use --enable-admission-plugins or --disable-admission-plugins instead. Will be removed in a future version.)
      --enable-admission-plugins strings       admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, LimitRanger, ServiceAccount, TaintNodesByCondition, PodSecurity, Priority, DefaultTolerationSeconds, DefaultStorageClass, StorageObjectInUseProtection, PersistentVolumeClaimResize, RuntimeClass, CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass, MutatingAdmissionWebhook, ValidatingAdmissionWebhook, ResourceQuota). Comma-delimited list of admission plugins: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass, DefaultStorageClass, DefaultTolerationSeconds, DenyServiceExternalIPs, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodSecurity, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, RuntimeClass, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, TaintNodesByCondition, ValidatingAdmissionWebhook. The order of plugins in this flag does not matter.

root@controlplane /opt ➜  nano /etc/kubernetes/manifests/kube-apiserver.yaml

root@controlplane /var/log/apiserver ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.18.113.9:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-log-path=/var/log/apiserver/audit.log
    - --profiling=false
    - --insecure-port=0
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --advertise-address=192.18.113.9
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.23.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.18.113.9
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.18.113.9
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.18.113.9
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/log/apiserver/
      name: audit-log
      readOnly: false
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - name: audit-log
    hostPath:
      path: /var/log/apiserver/
      type: DirectoryOrCreate
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

root@controlplane /var/log/apiserver ➜  watch crictl ps
Every 2.0s: crictl ps                                                                                                                                                                                                    controlplane: Tue Mar 26 11:14:49 2024
time="2024-03-26T11:14:49Z" level=warning msg="runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now d
eprecated, you should set the endpoint instead."
time="2024-03-26T11:14:49Z" level=warning msg="image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now dep
recated, you should set the endpoint instead."
CONTAINER           IMAGE                                                                                          CREATED              STATE               NAME                      ATTEMPT             POD ID              POD
f60902acdace0       e6bf5ddd40982                                                                                  About a minute ago   Running             kube-apiserver            0                   a129998a19f2d       kube-apiserver-controlplane
f89952de22425       37c6aeb3663ba                                                                                  About a minute ago   Running             kube-controller-manager   2                   b39a0809ebe95       kube-controller-manager-controlpl
ane
b7599db0806f3       56c5af1d00b5c                                                                                  About a minute ago   Running             kube-scheduler            2                   12a484187f980       kube-scheduler-controlplane
3c31c6d39a3e8       a4ca41631cc7a                                                                                  2 hours ago          Running             coredns                   0                   61a593f4b47e1       coredns-64897985d-jnrdp
597273fd80170       a4ca41631cc7a                                                                                  2 hours ago          Running             coredns                   0                   78b4ffb3b4acf       coredns-64897985d-q59zg
6bf346b90b1a9       df29c0a4002c0                                                                                  2 hours ago          Running             weave                     1                   259ea220f06d2       weave-net-fqvxx
b4dd5163a8e4d       weaveworks/weave-npc@sha256:38d3e30a97a2260558f8deb0fc4c079442f7347f27c86660dbfc8ca91674f14c   2 hours ago          Running             weave-npc                 0                   259ea220f06d2       weave-net-fqvxx
14b003bcf2b49       e03484a90585e                                                                                  2 hours ago          Running             kube-proxy                0                   9eab727568f37       kube-proxy-gflf6
a99c2f105702a       25f8c7f3da61c                                                                                  2 hours ago          Running             etcd                      0                   1c4b2fbcda5fd       etcd-controlplane

root@controlplane /opt ➜  cd /var/log/apiserver
root@controlplane /var/log/apiserver ➜  ls -l
total 0
-rw------- 1 root root 0 Mar 26 11:13 audit.log

root@controlplane /opt ✖ cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep -i profiling

root@controlplane /opt ✖ nano /etc/kubernetes/manifests/kube-controller-manager.yaml

root@controlplane /opt ➜  cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep -i profiling
    - --profiling=false

root@controlplane /opt ➜  cat /etc/kubernetes/manifests/kube-scheduler.yaml | grep -i profiling

root@controlplane /opt ✖ nano /etc/kubernetes/manifests/kube-scheduler.yaml

root@controlplane /opt ➜  cat /etc/kubernetes/manifests/kube-scheduler.yaml | grep -i profiling
    - --profiling=false

root@controlplane /opt ➜  ps -ef | grep etcd | grep -i data
root        2910    2821  0 09:03 ?        00:02:05 etcd --advertise-client-urls=https://192.18.113.9:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --initial-advertise-peer-urls=https://192.18.113.9:2380 --initial-cluster=controlplane=https://192.18.113.9:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.18.113.9:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.18.113.9:2380 --name=controlplane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

root@controlplane /opt ➜  ls -l /var/lib | grep etcd
drwx------  3 etcd root 4096 Mar 25 17:42 etcd

root@controlplane /opt ➜  chown etcd:etcd /var/lib/etcd

root@controlplane /opt ➜  ls -l /var/lib | grep etcd
drwx------  3 etcd etcd 4096 Mar 25 17:42 etcd

root@controlplane /opt ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2024-03-26 11:29:07 UTC; 3min 55s ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 62665 (kubelet)
    Tasks: 45 (limit: 251379)
   CGroup: /system.slice/kubelet.service
           └─62665 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.6

Mar 26 11:29:19 controlplane kubelet[62665]: E0326 11:29:19.017245   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:29:20 controlplane kubelet[62665]: E0326 11:29:20.026540   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:30:34 controlplane kubelet[62665]: E0326 11:30:34.668540   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:30:40 controlplane kubelet[62665]: E0326 11:30:40.658975   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:30:41 controlplane kubelet[62665]: E0326 11:30:41.657912   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque
Mar 26 11:31:37 controlplane kubelet[62665]: E0326 11:31:37.667439   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:31:56 controlplane kubelet[62665]: E0326 11:31:56.666409   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque
Mar 26 11:32:06 controlplane kubelet[62665]: E0326 11:32:06.665726   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:32:39 controlplane kubelet[62665]: E0326 11:32:39.657471   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:32:57 controlplane kubelet[62665]: E0326 11:32:57.666296   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque

root@controlplane /opt ✖ systemctl status kubelet | grep -i config
           └─62665 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.6

root@controlplane /opt ✖ cat /var/lib/kubelet/config.yaml | grep -i protectKernelDefaults

root@controlplane /opt ✖ nano /var/lib/kubelet/config.yaml

root@controlplane /opt ➜  cat /var/lib/kubelet/config.yaml | grep -i protectKernelDefaults
protectKernelDefaults: true

root@controlplane /opt ➜  systemctl daemon-reload

root@controlplane /opt ➜  systemctl restart kubelet.service

root@controlplane /opt ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2024-03-26 11:29:07 UTC; 6min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 62665 (kubelet)
    Tasks: 45 (limit: 251379)
   CGroup: /system.slice/kubelet.service
           └─62665 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.6

Mar 26 11:31:37 controlplane kubelet[62665]: E0326 11:31:37.667439   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:31:56 controlplane kubelet[62665]: E0326 11:31:56.666409   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque
Mar 26 11:32:06 controlplane kubelet[62665]: E0326 11:32:06.665726   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:32:39 controlplane kubelet[62665]: E0326 11:32:39.657471   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:32:57 controlplane kubelet[62665]: E0326 11:32:57.666296   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque
Mar 26 11:33:33 controlplane kubelet[62665]: E0326 11:33:33.663519   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:33:51 controlplane kubelet[62665]: E0326 11:33:51.659908   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:34:26 controlplane kubelet[62665]: E0326 11:34:26.667554   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-controller-manager-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod reque
Mar 26 11:34:48 controlplane kubelet[62665]: E0326 11:34:48.667463   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-apiserver-controlplane\" is forbidden: PodSecurityPolicy: no providers available to validate pod request" pod="
Mar 26 11:35:05 controlplane kubelet[62665]: E0326 11:35:05.663019   62665 kubelet.go:1711] "Failed creating a mirror pod for" err="pods \"kube-scheduler-controlplane\" is forbidden: PodSecurityPolicy: no prov

root@controlplane /opt ➜  ssh node01

root@node01 ~ ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systeroot@node01 ~ ➜  systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2024-03-26 09:04:05 UTC; 2h 32min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 1529 (kubelet)
    Tasks: 47 (limit: 251379)
   CGroup: /system.slice/kubelet.service
           └─1529 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.6

Mar 26 11:13:09 node01 kubelet[1529]: E0326 11:13:09.272999    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?timeout=10s\": dial tcp 192.18
Mar 26 11:13:09 node01 kubelet[1529]: E0326 11:13:09.273023    1529 kubelet_node_status.go:447] "Unable to update node status" err="update node status exceeds retry count"
Mar 26 11:13:16 node01 kubelet[1529]: E0326 11:13:16.070157    1529 controller.go:144] failed to ensure lease exists, will retry in 7s, error: Get "https://controlplane:6443/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/node01?timeout=10s"
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.311453    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?resourceVersion=0&timeout=10s\
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.312443    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?timeout=10s\": dial tcp 192.18
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.313397    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?timeout=10s\": dial tcp 192.18
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.314484    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?timeout=10s\": dial tcp 192.18
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.315572    1529 kubelet_node_status.go:460] "Error updating node status, will retry" err="error getting node \"node01\": Get \"https://controlplane:6443/api/v1/nodes/node01?timeout=10s\": dial tcp 192.18
Mar 26 11:13:19 node01 kubelet[1529]: E0326 11:13:19.315589    1529 kubelet_node_status.go:447] "Unable to update node status" err="update node status exceeds retry count"
Mar 26 11:13:23 node01 kubelet[1529]: E0326 11:13:23.071983    1529 controller.go:144] failed to ensure lease exists, will retry in 7s, error: Get "https://controlplane:6443/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/node01?timeout=10s"

root@node01 ~ ➜ cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS

root@node01 ~ ➜ cat /var/lib/kubelet/config.yaml | grep -i protectKernelDefaults

root@node01 ~ ✖ cat /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

root@node01 ~ ➜ nano /var/lib/kubelet/config.yaml

root@node01 ~ ➜ cat /var/lib/kubelet/config.yaml | grep -i protectKernelDefaults
protectKernelDefaults: true

root@node01 ~ ➜ systemctl daemon-reload
root@node01 ~ ➜ systemctl restart kubelet.service

root@node01 ~ ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2024-03-26 11:41:08 UTC; 24s ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 47873 (kubelet)
    Tasks: 30 (limit: 251379)
   CGroup: /system.slice/kubelet.service
           └─47873 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.6

Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189371   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-bin2\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-cni-bin2\
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189387   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-conf\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-cni-conf\
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189402   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"lib-modules\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-lib-mo
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189500   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-proxy\" (UniqueName: \"kubernetes.io/configmap/bedfa60b-929f-4055-bd8f-c138db7ea6f9-kube-pr
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189542   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"xtables-lock\" (UniqueName: \"kubernetes.io/host-path/bedfa60b-929f-4055-bd8f-c138db7ea6f9-xtabl
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189559   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"weavedb\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-weavedb\")
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189574   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-bin\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-cni-bin\")
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.189588   47873 reconciler.go:216] "operationExecutor.VerifyControllerAttachedVolume started for volume \"dbus\" (UniqueName: \"kubernetes.io/host-path/1675ea3e-2148-4816-9336-c4bd26b57eab-dbus\") pod \
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.190392   47873 reconciler.go:157] "Reconciler: start to sync state"
Mar 26 11:41:10 node01 kubelet[47873]: I0326 11:41:10.709531   47873 prober_manager.go:255] "Failed to trigger a manual run" probe="Readiness"

root@node01 ~ ✖ exit
logout
Connection to node01 closed.

root@controlplane /opt ✖ 
```

## AFTER:

![image](https://i.imgur.com/5d57wuI.png)

