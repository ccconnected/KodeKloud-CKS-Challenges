# Challenge 4: https://kodekloud.com/topic/lab-challenge-4/

## BEFORE:

![image](https://i.imgur.com/lWUHwAr.png)

## TASK:
There are a number of Kubernetes objects created inside the omega, citadel and eden-prime namespaces. However, several suspicious/abnormal operations have been observed in these namespaces!.

For example, in the citadel namespace, the application called webapp-color is constantly changing! You can see this for yourself by clicking on the citadel-webapp link and refreshing the page every 30 seconds. Similarly there are other issues with several other objects in other namespaces.

To understand what's causing these anomalies, you would be required to configure auditing in Kubernetes and make use of the Falco tool.

Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the cluster. Once done click on the Check button to validate your work.

1) Info on "Falco"
-    Install the 'falco' utility on the controlplane node and start it as a systemd service

2) Info on "file-output"
-    Configure falco to save the event output to the file '/opt/falco.log'

3) Info on "Auditing"
-    The audit policy file should be stored at '/etc/kubernetes/audit-policy.yaml'
-    Use a volume called 'audit' that will mount only the file '/etc/kubernetes/audit-policy.yaml' from the controlplane inside the api server pod in a read only mode.
-    Create a single rule in the audit policy that will record events for the 'two' objects depicting abnormal behaviour in the 'citadel' namespace. This rule should however be applied to all 'three' namespaces shown in the diagram at a 'metadata' level. Omit the 'RequestReceived' stage.

4) Info on "audit-log"
-    audit-log-path set to '/var/log/kubernetes/audit/audit.log'

5) Info on "kube-apiserver"
-    API server running?

6) Info on "Security Reports"
-    Inspect the API server audit logs and identify the user responsible for the abnormal behaviour seen in the 'citadel' namespace. Save the name of the 'user', 'role' and 'rolebinding' responsible for the event to the file '/opt/blacklist_users' file (comma separated and in this specific order).
-    Inspect the 'falco' logs and identify the pod that has events generated because of packages being updated on it. Save the namespace and the pod name in the file '/opt/compromised_pods' (comma separated - namespace followed by the pod name)

7) Info on "citadel" "secret"
-    Delete the role causing the constant deletion and creation of the configmaps and pods in this namespace. Do not delete any other role!

8) Info on "citadel" "deploy"
-    Delete the rolebinding causing the constant deletion and creation of the configmaps and pods in this namespace. Do not delete any other rolebinding!

9) Info on "omega" "pod"
-    Delete pods belonging to the 'omega' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

10) Info on "eden-prime" "pod"
-    Delete pods belonging to the 'eden-prime' namespace that were flagged in the 'Security Report' file '/opt/compromised_pods'. Do not delete the non-compromised pods!

## SOLUTION:
```
root@controlplane ~ ➜  echo $SHELL
/bin/sh

root@controlplane ~ ➜  source <(kubectl completion bash) 
root@controlplane ~ ➜  echo "source <(kubectl completion bash)" >> ~/.bashrc 
root@controlplane ~ ➜  alias k=kubectl
root@controlplane ~ ➜  complete -o default -F __start_kubectl k

root@controlplane ~ ➜  export dr="--dry-run=client -oyaml"
root@controlplane ~ ➜  export now="--force --grace-period=0"

root@controlplane ~ ➜  cd /tmp

root@controlplane /tmp ✖ cat /etc/os-release 
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

-> Instruction to install Falco https://falco.org/docs/install-operate/installation/#installation-details

root@controlplane /tmp ➜  FALCO_FRONTEND=noninteractive apt-get install -y falco
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  falco
0 upgraded, 1 newly installed, 0 to remove and 6 not upgraded.
Need to get 0 B/14.9 MB of archives.
After this operation, 38.6 MB of additional disk space will be used.
Selecting previously unselected package falco.
(Reading database ... 76419 files and directories currently installed.)
Preparing to unpack .../falco_0.31.1_amd64.deb ...
Unpacking falco (0.31.1) ...
Setting up falco (0.31.1) ...
Loading new falco-b7eb0dd65226a8dc254d228c8d950d07bf3521d2 DKMS files...
Building for 4.15.0-171-generic
Building initial module for 4.15.0-171-generic
Done.
falco:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-171-generic/updates/dkms/
depmod...
DKMS: install completed.

root@controlplane /tmp ➜  systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: https://falco.org/docs/

root@controlplane /tmp ✖ systemctl start falco

root@controlplane /tmp ➜  systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-03-26 15:01:44 UTC; 8s ago
     Docs: https://falco.org/docs/
  Process: 22969 ExecStartPre=/sbin/modprobe falco (code=exited, status=0/SUCCESS)
 Main PID: 22984 (falco)
    Tasks: 10 (limit: 2314)
   CGroup: /system.slice/falco.service
           └─22984 /usr/bin/falco --pidfile=/var/run/falco.pid

Mar 26 15:01:44 controlplane falco[22984]: Tue Mar 26 15:01:44 2024: Loading rules from file /etc/falco/falco_rules.local.yaml:
Mar 26 15:01:44 controlplane falco[22984]: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mar 26 15:01:44 controlplane falco[22984]: Tue Mar 26 15:01:44 2024: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mar 26 15:01:45 controlplane falco[22984]: Starting internal webserver, listening on port 8765
Mar 26 15:01:45 controlplane falco[22984]: Tue Mar 26 15:01:45 2024: Starting internal webserver, listening on port 8765
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.286520000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:69320d71dc71 k8s_POD_kube-proxy-xkchm_kube-system_ac46e118-795f-450a-94b1-5b977ef47bf1_0 (id=69320d71dc71)
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.289474000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:a3467ac90ea2 k8s_POD_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=a3467ac90ea2) 
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.305557000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:833bb6215ff0 k8s_weave-npc_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=833bb621
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.314551000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:d67df1e8f833 k8s_weave_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_1 (id=d67df1e8f833
Mar 26 15:01:46 controlplane falco[22984]: 15:01:46.343226320: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime
lines 1-20/20 (END)

root@controlplane /tmp ✖ falco
Tue Mar 26 15:02:47 2024: Falco version 0.31.1 (driver version b7eb0dd65226a8dc254d228c8d950d07bf3521d2)
Tue Mar 26 15:02:47 2024: Falco initialized with configuration file /etc/falco/falco.yaml
Tue Mar 26 15:02:47 2024: Loading rules from file /etc/falco/falco_rules.yaml:
Tue Mar 26 15:02:48 2024: Loading rules from file /etc/falco/falco_rules.local.yaml:
Tue Mar 26 15:02:48 2024: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Tue Mar 26 15:02:49 2024: Starting internal webserver, listening on port 8765
Tue Mar 26 15:02:49 2024: Runtime error: Could not create embedded webserver: null context when constructing CivetServer. Possible problem binding to port.. Exiting.

root@controlplane /tmp ✖ systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-03-26 15:01:44 UTC; 1min 24s ago
     Docs: https://falco.org/docs/
  Process: 22969 ExecStartPre=/sbin/modprobe falco (code=exited, status=0/SUCCESS)
 Main PID: 22984 (falco)
    Tasks: 10 (limit: 2314)
   CGroup: /system.slice/falco.service
           └─22984 /usr/bin/falco --pidfile=/var/run/falco.pid

Mar 26 15:01:45 controlplane falco[22984]: Tue Mar 26 15:01:45 2024: Starting internal webserver, listening on port 8765
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.286520000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:69320d71dc71 k8s_POD_kube-proxy-xkchm_kube-system_ac46e118-795f-450a-94b1-5b977ef47bf1_0 (id=69320d71dc71)
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.289474000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:a3467ac90ea2 k8s_POD_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=a3467ac90ea2) 
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.305557000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:833bb6215ff0 k8s_weave-npc_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=833bb621
Mar 26 15:01:45 controlplane falco[22984]: 15:01:45.314551000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:d67df1e8f833 k8s_weave_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_1 (id=d67df1e8f833
Mar 26 15:01:46 controlplane falco[22984]: 15:01:46.343226320: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime
Mar 26 15:02:01 controlplane falco[22984]: 15:02:01.797804995: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime
Mar 26 15:02:26 controlplane falco[22984]: 15:02:26.787319767: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime
Mar 26 15:02:46 controlplane falco[22984]: 15:02:46.768428984: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime
Mar 26 15:03:02 controlplane falco[22984]: 15:03:02.092802598: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime

root@controlplane /tmp ➜  cat /etc/falco/falco.yaml | grep -i output
# If true, the times displayed in log messages and output messages
# Whether to output events in json or text
json_output: false
# When using json output, whether or not to include the "output" property
# (user=root ....") in the json output.
json_include_output_property: true
# When using json output, whether or not to include the "tags" property
# itself in the json output. If set to true, outputs caused by rules
# false, the "tags" field will not be included in the json output at all.
# Whether or not output to any of the output channels below is
buffered_outputs: false
# Falco continuously monitors outputs performance. When an output channel does not allow
# which output is blocking notifications.
# Note that the notification will not be discarded from the output queue; thus,
# output channels may indefinitely remain blocked.
# An output timeout error indeed indicate a misconfiguration issue or I/O problems
# The "output_timeout" value specifies the duration in milliseconds to wait before
# With a 2000ms default, the notification consumer can block the Falco output
output_timeout: 2000
outputs:
# Multiple outputs can be enabled.
syslog_output:
# continuously written to, with each output message on its own
# for each output message.
file_output:
stdout_output:
# Possible additional things you might want to do with program output:
#         program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/XXX"
# continuously written to, with each output message on its own
# for each output message.
program_output:
  program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/XXX"
http_output:
# By default, the gRPC server is disabled, with no enabled services (see grpc_output)
# gRPC output service.
# By enabling this all the output events will be kept in memory until you read them with a gRPC client.
grpc_output:

root@controlplane /tmp ➜  cd /opt/

root@controlplane /opt ➜  ls -l
total 8
drwxr-xr-x 3 root root 4096 Mar 17  2022 cni
drwx--x--x 4 root root 4096 Mar 17  2022 containerd

root@controlplane /opt ➜  cat /etc/falco/falco.yaml | grep -i file_output -A10
file_output:
  enabled: false
  keep_alive: false
  filename: ./events.txt

stdout_output:
  enabled: true

# Falco contains an embedded webserver that can be used to accept K8s
# Audit Events. These config options control the behavior of that
# webserver. (By default, the webserver is enabled).

root@controlplane /opt ➜  nano /etc/falco/falco.yaml

root@controlplane /opt ➜  cat /etc/falco/falco.yaml | grep -i file_output -A10
file_output:
  enabled: true
  keep_alive: false
  filename: /opt/falco.log

stdout_output:
  enabled: true

# Falco contains an embedded webserver that can be used to accept K8s
# Audit Events. These config options control the behavior of that
# webserver. (By default, the webserver is enabled).

root@controlplane /opt ➜  systemctl restart falco

root@controlplane /opt ➜  systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-03-26 15:08:58 UTC; 9s ago
     Docs: https://falco.org/docs/
  Process: 1340 ExecStopPost=/sbin/rmmod falco (code=exited, status=0/SUCCESS)
  Process: 1346 ExecStartPre=/sbin/modprobe falco (code=exited, status=0/SUCCESS)
 Main PID: 1368 (falco)
    Tasks: 10 (limit: 2314)
   CGroup: /system.slice/falco.service
           └─1368 /usr/bin/falco --pidfile=/var/run/falco.pid

Mar 26 15:08:58 controlplane falco[1368]: Tue Mar 26 15:08:58 2024: Loading rules from file /etc/falco/falco_rules.local.yaml:
Mar 26 15:08:58 controlplane falco[1368]: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mar 26 15:08:58 controlplane falco[1368]: Tue Mar 26 15:08:58 2024: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mar 26 15:08:59 controlplane falco[1368]: Starting internal webserver, listening on port 8765
Mar 26 15:08:59 controlplane falco[1368]: Tue Mar 26 15:08:59 2024: Starting internal webserver, listening on port 8765
Mar 26 15:08:59 controlplane falco[1368]: 15:08:59.256731000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:69320d71dc71 k8s_POD_kube-proxy-xkchm_kube-system_ac46e118-795f-450a-94b1-5b977ef47bf1_0 (id=69320d71dc71) 
Mar 26 15:08:59 controlplane falco[1368]: 15:08:59.258700000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:a3467ac90ea2 k8s_POD_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=a3467ac90ea2) i
Mar 26 15:08:59 controlplane falco[1368]: 15:08:59.266690000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:833bb6215ff0 k8s_weave-npc_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=833bb6215
Mar 26 15:08:59 controlplane falco[1368]: 15:08:59.270060000: Informational Privileged container started (user=root user_loginuid=0 command=container:d67df1e8f833 k8s_weave_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_1 (id=d67df1e8f833)
Mar 26 15:09:01 controlplane falco[1368]: 15:09:01.818026882: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_

root@controlplane /opt ✖ ls -l
total 12
drwxr-xr-x 3 root root 4096 Mar 17  2022 cni
drwx--x--x 4 root root 4096 Mar 17  2022 containerd
-rw------- 1 root root 1265 Mar 26 13:56 falco.log

root@controlplane /opt ➜  cat falco.log
15:08:59.256731000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:69320d71dc71 k8s_POD_kube-proxy-xkchm_kube-system_ac46e118-795f-450a-94b1-5b977ef47bf1_0 (id=69320d71dc71) image=k8s.gcr.io/pause:3.6)
15:08:59.258700000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:a3467ac90ea2 k8s_POD_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=a3467ac90ea2) image=k8s.gcr.io/pause:3.6)
15:08:59.266690000: Informational Privileged container started (user=<NA> user_loginuid=0 command=container:833bb6215ff0 k8s_weave-npc_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_0 (id=833bb6215ff0) image=weaveworks/weave-npc:2.8.1)
15:08:59.270060000: Informational Privileged container started (user=root user_loginuid=0 command=container:d67df1e8f833 k8s_weave_weave-net-7vt8w_kube-system_2a0cc17e-9eed-4f6f-8441-cf2c8afd59ec_1 (id=d67df1e8f833) image=weaveworks/weave-kube:2.8.1)
15:09:01.818026882: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:09:26.792862254: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:09:46.809789195: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)

root@controlplane /opt ➜  cd /etc/kubernetes

root@controlplane /etc/kubernetes ➜  ls -l
total 36
-rw------- 1 root root 5636 Mar 26 12:55 admin.conf
-rw------- 1 root root 5675 Mar 26 12:55 controller-manager.conf
-rw------- 1 root root 1984 Mar 26 12:55 kubelet.conf
drwxr-xr-x 2 root root 4096 Mar 26 12:55 manifests
drwxr-xr-x 4 root root 4096 Mar 26 13:37 pki
-rw------- 1 root root 5623 Mar 26 12:55 scheduler.conf

root@controlplane /etc/kubernetes ✖ cd manifests

root@controlplane /etc/kubernetes/manifests ➜  ls -l
total 16
-rw------- 1 root root 2239 Mar 26 14:17 etcd.yaml
-rw------- 1 root root 3869 Mar 26 14:17 kube-apiserver.yaml
-rw------- 1 root root 3365 Mar 26 14:17 kube-controller-manager.yaml
-rw------- 1 root root 1435 Mar 26 14:17 kube-scheduler.yaml

root@controlplane /etc/kubernetes/manifests ➜  nano /etc/kubernetes/audit-policy.yaml

root@controlplane /etc/kubernetes/manifests ➜  cat /etc/kubernetes/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods", "configmaps"]
    namespaces: ["omega","citadel","eden-prime"]

root@controlplane /etc/kubernetes/manifests ➜  cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/kube-apiserver-bak.yaml

root@controlplane /etc/kubernetes/manifests ➜  watch crictl ps
Every 2.0s: crictl ps                                                                                                                                                                                                      controlplane: Tue Mar 26 15:13:15 2024
CONTAINER           IMAGE                                                                                            CREATED             STATE               NAME                      ATTEMPT             POD ID
9aeece5c70722       kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423   12 seconds ago      Running             webapp-color              0                   8a346056a7794
a822d48762be9       kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423   32 seconds ago      Running             webapp-color              0                   3ab5847a3c9f2
6777ee426ea20       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  13 minutes ago      Running             omega-software5           0                   67fe62dc8aea0
25bb19f070b51       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  13 minutes ago      Running             omega-software4           0                   54e34327e3698
3380897ebe94d       ubuntu@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e                   13 minutes ago      Running             eden-software2            0                   bf1e7b4684c9a
c3c4c5b8f284e       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  13 minutes ago      Running             omega-software6           0                   be2bf5786c00c
0d59b2d692e26       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  13 minutes ago      Running             eden-software3            0                   e0099b7a046db
0c75cc9b01453       nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e                    13 minutes ago      Running             nginx                     0                   736c125b01430
d7ea82087b576       nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e                    13 minutes ago      Running             nginx                     0                   ca9ff025387f7
c0e30267f834d       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  13 minutes ago      Running             eden-software1            0                   89676e29ad014
c47bacab55a71       a4ca41631cc7a                                                                                    54 minutes ago      Running             coredns                   0                   bc0e7edd7fe8c
27782e46b8661       a4ca41631cc7a                                                                                    55 minutes ago      Running             coredns                   0                   ed3a39a4f2458
d67df1e8f8335       df29c0a4002c0                                                                                    55 minutes ago      Running             weave                     1                   a3467ac90ea25
833bb6215ff0b       7f92d556d4ffe                                                                                    55 minutes ago      Running             weave-npc                 0                   a3467ac90ea25
72861840475e1       e03484a90585e                                                                                    55 minutes ago      Running             kube-proxy                0                   69320d71dc719
2cd384ea3212b       56c5af1d00b5c                                                                                    55 minutes ago      Running             kube-scheduler            0                   8c318e058c0c7
eb3b99fc8d42f       e6bf5ddd40982                                                                                    55 minutes ago      Running             kube-apiserver            0                   7b5146eb241c6
984c60161aba7       37c6aeb3663ba                                                                                    55 minutes ago      Running             kube-controller-manager   0                   ec35f3ac8b71c
e5b8a07c09fea       25f8c7f3da61c                                                                                    55 minutes ago      Running             etcd                      0                   3c0cd246f1e99

root@controlplane /etc/kubernetes/manifests ➜  nano /etc/kubernetes/manifests/kube-apiserver.yaml

root@controlplane /etc/kubernetes/manifests ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.121.231:6443
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
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --advertise-address=192.168.121.231
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
        host: 192.168.121.231
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
        host: 192.168.121.231
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
        host: 192.168.121.231
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/
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
  - name: audit
    hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/kubernetes/audit/
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

root@controlplane /etc/kubernetes/manifests ➜  watch crictl ps
Every 2.0s: crictl ps                                                                                                                                                                                                      controlplane: Tue Mar 26 15:15:49 2024
CONTAINER           IMAGE                                                                                            CREATED              STATE               NAME                      ATTEMPT             POD ID
02de55042bfcc       e6bf5ddd40982                                                                                    21 seconds ago       Running             kube-apiserver            0                   2891188c54724
3750d727d5813       37c6aeb3663ba                                                                                    51 seconds ago       Running             kube-controller-manager   1                   ec35f3ac8b71c
4e04ad46ccc02       56c5af1d00b5c                                                                                    51 seconds ago       Running             kube-scheduler            1                   8c318e058c0c7
3f2616b574e28       kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423   About a minute ago   Running             webapp-color              0                   2a396426590c2
6777ee426ea20       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  16 minutes ago       Running             omega-software5           0                   67fe62dc8aea0
25bb19f070b51       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  16 minutes ago       Running             omega-software4           0                   54e34327e3698
3380897ebe94d       ubuntu@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e                   16 minutes ago       Running             eden-software2            0                   bf1e7b4684c9a
c3c4c5b8f284e       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  16 minutes ago       Running             omega-software6           0                   be2bf5786c00c
0d59b2d692e26       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  16 minutes ago       Running             eden-software3            0                   e0099b7a046db
0c75cc9b01453       nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e                    16 minutes ago       Running             nginx                     0                   736c125b01430
d7ea82087b576       nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e                    16 minutes ago       Running             nginx                     0                   ca9ff025387f7
c0e30267f834d       busybox@sha256:650fd573e056b679a5110a70aabeb01e26b76e545ec4b9c70a9523f2dfaf18c6                  16 minutes ago       Running             eden-software1            0                   89676e29ad014
c47bacab55a71       a4ca41631cc7a                                                                                    57 minutes ago       Running             coredns                   0                   bc0e7edd7fe8c
27782e46b8661       a4ca41631cc7a                                                                                    57 minutes ago       Running             coredns                   0                   ed3a39a4f2458
d67df1e8f8335       df29c0a4002c0                                                                                    57 minutes ago       Running             weave                     1                   a3467ac90ea25
833bb6215ff0b       7f92d556d4ffe                                                                                    57 minutes ago       Running             weave-npc                 0                   a3467ac90ea25
72861840475e1       e03484a90585e                                                                                    57 minutes ago       Running             kube-proxy                0                   69320d71dc719
e5b8a07c09fea       25f8c7f3da61c                                                                                    58 minutes ago       Running             etcd                      0                   3c0cd246f1e99

root@controlplane /etc/kubernetes/manifests ➜  cat /var/log/kubernetes/audit/audit.log | grep -i username
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"cb5c6bfd-12d3-4640-9ed2-849f91a002f6","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/omega/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556\u0026timeout=9m52s\u0026timeoutSeconds=592\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"Forbidden","code":403},"requestReceivedTimestamp":"2024-03-26T15:15:31.609380Z","stageTimestamp":"2024-03-26T15:15:31.609562Z","annotations":{"authorization.k8s.io/decision":"forbid","authorization.k8s.io/reason":"no relationship found between node 'controlplane' and this object"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"cb5c6bfd-12d3-4640-9ed2-849f91a002f6","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556\u0026timeout=9m52s\u0026timeoutSeconds=592\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"Forbidden","code":403},"requestReceivedTimestamp":"2024-03-26T15:15:31.609380Z","stageTimestamp":"2024-03-26T15:15:31.611758Z","annotations":{"authorization.k8s.io/decision":"forbid","authorization.k8s.io/reason":"no relationship found between node 'controlplane' and this object"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"24fc3c9c-a75a-4de2-85d6-894607af37bc","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5600\u0026timeout=8m27s\u0026timeoutSeconds=507\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"Forbidden","code":403},"requestReceivedTimestamp":"2024-03-26T15:15:31.611818Z","stageTimestamp":"2024-03-26T15:15:31.612000Z","annotations":{"authorization.k8s.io/decision":"forbid","authorization.k8s.io/reason":"no relationship found between node 'controlplane' and this object"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"24fc3c9c-a75a-4de2-85d6-894607af37bc","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5600\u0026timeout=8m27s\u0026timeoutSeconds=507\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"Forbidden","code":403},"requestReceivedTimestamp":"2024-03-26T15:15:31.611818Z","stageTimestamp":"2024-03-26T15:15:31.612063Z","annotations":{"authorization.k8s.io/decision":"forbid","authorization.k8s.io/reason":"no relationship found between node 'controlplane' and this object"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"c4c967bb-bba3-4127-99e9-ec916c4eb1fe","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5600","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:32.469546Z","stageTimestamp":"2024-03-26T15:15:32.471193Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"99e7d15b-7481-4f95-ab21-7e491230e7d4","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5625\u0026timeout=9m23s\u0026timeoutSeconds=563\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:32.473695Z","stageTimestamp":"2024-03-26T15:15:32.474172Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"841fcb72-868a-49d2-89b1-8843fa6e30c9","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:32.642059Z","stageTimestamp":"2024-03-26T15:15:32.642751Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"e1d5837a-859c-4662-b6df-80d8f97d853a","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/omega/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5625\u0026timeout=8m0s\u0026timeoutSeconds=480\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"omega","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:32.643496Z","stageTimestamp":"2024-03-26T15:15:32.643876Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"b3e1de1c-c54c-4e4e-86a9-52f794379ddd","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2024-03-26T15:15:41.284302Z","stageTimestamp":"2024-03-26T15:15:41.310876Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"ba763a8e-043c-48ef-8b5e-429ff7c965cd","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:41.313996Z","stageTimestamp":"2024-03-26T15:15:41.318594Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"75380586-f62d-4bfc-9fa7-5c62f59233b1","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:15:41.388232Z","stageTimestamp":"2024-03-26T15:15:41.394135Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"7876496f-3a3b-4e67-b02a-722a8af259f1","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:41.464361Z","stageTimestamp":"2024-03-26T15:15:41.477069Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6300a7f1-2de6-477c-84e9-17f6f94e2143","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:41.478858Z","stageTimestamp":"2024-03-26T15:15:41.496276Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"67d1a272-e8ef-47fe-9c81-b3206fe5d2d4","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"NotFound","code":404},"requestReceivedTimestamp":"2024-03-26T15:15:41.480825Z","stageTimestamp":"2024-03-26T15:15:41.511379Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"99e7d15b-7481-4f95-ab21-7e491230e7d4","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5625\u0026timeout=9m23s\u0026timeoutSeconds=563\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:32.473695Z","stageTimestamp":"2024-03-26T15:15:41.535216Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"8da16423-be22-4089-9dbb-058f12fe4d53","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:15:42.328339Z","stageTimestamp":"2024-03-26T15:15:42.341945Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"","pod-security.kubernetes.io/enforce-policy":"privileged:latest"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"9ecfd411-5b26-4a7c-925b-0b6b4b6ed386","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods?limit=500","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"eden-prime","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:44.528874Z","stageTimestamp":"2024-03-26T15:15:44.533535Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"aaaa7e86-bcaf-4960-90c0-ba03719ea136","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software1","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software1","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:44.595055Z","stageTimestamp":"2024-03-26T15:15:44.597003Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"5dea55f5-587c-4309-98d0-2b13309edb54","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:44.658536Z","stageTimestamp":"2024-03-26T15:15:44.660764Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"7669ba5c-2e8d-4b71-a173-e5c23cf727fc","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods?limit=500","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:45.271718Z","stageTimestamp":"2024-03-26T15:15:45.275589Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"efbbb803-edf0-4d2e-a23b-1202219b4f3e","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software4","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software4","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:45.338752Z","stageTimestamp":"2024-03-26T15:15:45.340587Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"9df57ad2-da2a-4cda-a613-703a28c9a607","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software5","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software5","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:45.400710Z","stageTimestamp":"2024-03-26T15:15:45.402614Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"ead84d33-f35e-4830-96d1-caecce988ff6","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/omega/pods/omega-software6","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"kubectl/v1.23.0 (linux/amd64) kubernetes/ab69524","objectRef":{"resource":"pods","namespace":"omega","name":"omega-software6","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:45.528307Z","stageTimestamp":"2024-03-26T15:15:45.533396Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"bd0974e6-f79a-4f66-af28-ed39b1ea755b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:15:46.280886Z","stageTimestamp":"2024-03-26T15:15:46.283477Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"9ab3a2e5-d91b-4315-81a7-926996e481b4","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:01.557784Z","stageTimestamp":"2024-03-26T15:16:01.560077Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"cfee3bf9-2a83-458d-8624-8ce64475aab3","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2024-03-26T15:16:01.556009Z","stageTimestamp":"2024-03-26T15:16:01.606927Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"50bf5331-d892-45cd-ab94-e456d385e7f4","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:01.608537Z","stageTimestamp":"2024-03-26T15:16:01.610762Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"e3f3a1e6-eaee-4922-b7d6-accfe78a1ed5","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:01.676272Z","stageTimestamp":"2024-03-26T15:16:01.679314Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"9eb282a1-6a9a-4f49-84a8-ad593d23b060","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:01.747275Z","stageTimestamp":"2024-03-26T15:16:01.757169Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"339ee2ac-d069-4f1d-adbe-8447ac467aef","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:01.759930Z","stageTimestamp":"2024-03-26T15:16:01.763080Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"7d176542-2fbd-4716-bef5-3d1d64e9ce12","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:01.983357Z","stageTimestamp":"2024-03-26T15:16:01.987482Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"","pod-security.kubernetes.io/enforce-policy":"privileged:latest"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"821804e5-1519-41f5-81bf-bdcddc210ae7","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.002711Z","stageTimestamp":"2024-03-26T15:16:02.003512Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"473ead08-b4e8-4690-b1e9-d37215036671","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.003809Z","stageTimestamp":"2024-03-26T15:16:02.004633Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"8e38cb6f-54b9-4f3e-bb96-d824a48dbfaf","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.004808Z","stageTimestamp":"2024-03-26T15:16:02.010561Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"a200dbd4-3df1-4824-83ae-6d401893b10f","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5666\u0026timeout=8m15s\u0026timeoutSeconds=495\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.010147Z","stageTimestamp":"2024-03-26T15:16:02.010683Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6942d002-de4f-4b22-9c05-13fc3c44ace3","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5666\u0026timeout=7m52s\u0026timeoutSeconds=472\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.010755Z","stageTimestamp":"2024-03-26T15:16:02.011282Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6a27b888-24ac-4f94-bc06-ececc5964769","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:04.439581Z","stageTimestamp":"2024-03-26T15:16:04.441663Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dd8061bc-926f-4adb-ae02-2ab617ed6314","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/eden-prime/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556\u0026timeout=7m18s\u0026timeoutSeconds=438\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"eden-prime","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:11.487052Z","stageTimestamp":"2024-03-26T15:16:11.501423Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dd8061bc-926f-4adb-ae02-2ab617ed6314","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556\u0026timeout=7m18s\u0026timeoutSeconds=438\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"eden-prime","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:11.487052Z","stageTimestamp":"2024-03-26T15:16:11.501927Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"fb0520e1-14e5-4a9a-9881-e0c30403709e","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5556","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"eden-prime","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:12.366439Z","stageTimestamp":"2024-03-26T15:16:12.368561Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"86441214-1299-4869-9a64-ad253a2f526e","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/eden-prime/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5666\u0026timeout=7m9s\u0026timeoutSeconds=429\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"eden-prime","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:12.369407Z","stageTimestamp":"2024-03-26T15:16:12.370066Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"51ff4af0-24cc-4641-8b78-4ddebd3868b3","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:12.595942Z","stageTimestamp":"2024-03-26T15:16:12.598357Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dd9f28be-337e-460d-a06e-fca15c35d13a","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:12.600301Z","stageTimestamp":"2024-03-26T15:16:12.601989Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"2943463b-6c62-4830-a5e1-0d927fac0ec8","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2024-03-26T15:16:21.555338Z","stageTimestamp":"2024-03-26T15:16:21.574584Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"9edd4b4a-913c-4263-b7e4-7d5999723204","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:21.576141Z","stageTimestamp":"2024-03-26T15:16:21.577799Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"404557ad-5b7e-44a8-9630-4738d62c198c","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:21.644121Z","stageTimestamp":"2024-03-26T15:16:21.648959Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6942d002-de4f-4b22-9c05-13fc3c44ace3","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5666\u0026timeout=7m52s\u0026timeoutSeconds=472\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.010755Z","stageTimestamp":"2024-03-26T15:16:21.767369Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"a200dbd4-3df1-4824-83ae-6d401893b10f","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5666\u0026timeout=8m15s\u0026timeoutSeconds=495\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:02.010147Z","stageTimestamp":"2024-03-26T15:16:21.767485Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f2d20589-587d-4587-94e1-c55414733fb5","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:21.718862Z","stageTimestamp":"2024-03-26T15:16:21.767365Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"35927a89-6dec-48ec-9dbe-b837c671ae89","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"NotFound","code":404},"requestReceivedTimestamp":"2024-03-26T15:16:21.738338Z","stageTimestamp":"2024-03-26T15:16:21.781231Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"0e9da4b5-da4a-4591-ac23-9e9153910077","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:21.780115Z","stageTimestamp":"2024-03-26T15:16:21.790952Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"899b6664-364c-4873-a337-ea10026fe7f6","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:21.992968Z","stageTimestamp":"2024-03-26T15:16:21.999681Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"","pod-security.kubernetes.io/enforce-policy":"privileged:latest"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f3f0849f-775e-47cb-baeb-fb33b70b274b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.020383Z","stageTimestamp":"2024-03-26T15:16:22.028539Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"32bd8401-d0f7-4217-9e49-f860aeca3481","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.028638Z","stageTimestamp":"2024-03-26T15:16:22.036434Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"2a4f0759-a3af-49c2-ac43-a4c50c734ddd","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.027251Z","stageTimestamp":"2024-03-26T15:16:22.040497Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"44d62cf1-c409-4135-988e-1ce821012ab0","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5711\u0026timeout=8m1s\u0026timeoutSeconds=481\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.041668Z","stageTimestamp":"2024-03-26T15:16:22.042219Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dd968dc4-cfd6-49d6-9fb7-f6c2b3f285d2","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5711\u0026timeout=5m39s\u0026timeoutSeconds=339\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.040380Z","stageTimestamp":"2024-03-26T15:16:22.045212Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"c823dabb-de17-43cf-b13d-b5b2052593dc","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:24.110607Z","stageTimestamp":"2024-03-26T15:16:24.113759Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"686c5d80-3752-46e7-adba-8bdc38880633","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:26.555442Z","stageTimestamp":"2024-03-26T15:16:26.557278Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"87761630-8d0b-40af-abc1-2bd70ef7735b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.558264Z","stageTimestamp":"2024-03-26T15:16:41.576854Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"4f00b113-d979-4fd1-b541-04c75beaa160","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.578562Z","stageTimestamp":"2024-03-26T15:16:41.579946Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"446491f7-102d-4ce0-86ac-26d5b469b4cc","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:41.645950Z","stageTimestamp":"2024-03-26T15:16:41.648806Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"e7e5e533-a6d0-477a-8ee4-afab5c508dfd","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.716059Z","stageTimestamp":"2024-03-26T15:16:41.747434Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dd968dc4-cfd6-49d6-9fb7-f6c2b3f285d2","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5711\u0026timeout=5m39s\u0026timeoutSeconds=339\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.040380Z","stageTimestamp":"2024-03-26T15:16:41.758518Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"1a0a2ac9-f8d6-4449-a2a1-58ae1fdf0ba9","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"NotFound","code":404},"requestReceivedTimestamp":"2024-03-26T15:16:41.736581Z","stageTimestamp":"2024-03-26T15:16:41.758760Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"a585e475-cf22-4f0f-8328-ec37ead858bf","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.753301Z","stageTimestamp":"2024-03-26T15:16:41.759151Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"44d62cf1-c409-4135-988e-1ce821012ab0","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5711\u0026timeout=8m1s\u0026timeoutSeconds=481\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:22.041668Z","stageTimestamp":"2024-03-26T15:16:41.760393Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"ca48427a-2477-4ca7-bb16-7d9694f5cbd1","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:16:41.968106Z","stageTimestamp":"2024-03-26T15:16:41.971409Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"","pod-security.kubernetes.io/enforce-policy":"privileged:latest"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"4dfb30f7-9cfe-4af4-a115-f22440845807","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.985456Z","stageTimestamp":"2024-03-26T15:16:41.986233Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"c39a466b-c48e-4e57-bd3c-bde474c181e6","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.984043Z","stageTimestamp":"2024-03-26T15:16:41.986374Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"b271b7d0-6d31-4b85-83e4-bc518028535e","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.986428Z","stageTimestamp":"2024-03-26T15:16:41.990567Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"89374f31-958e-42d6-ba69-9d344ecfb6ff","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5755\u0026timeout=5m38s\u0026timeoutSeconds=338\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.994581Z","stageTimestamp":"2024-03-26T15:16:41.994987Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"91c1a816-c46d-411a-b8b1-00c0fe0a4e88","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5755\u0026timeout=8m17s\u0026timeoutSeconds=497\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.995440Z","stageTimestamp":"2024-03-26T15:16:41.995819Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"04f84cf3-75f7-4133-8763-52bb549b526a","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:44.132442Z","stageTimestamp":"2024-03-26T15:16:44.135232Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"2aca7351-0eae-49eb-822b-e5d1c1c3b266","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:46.551842Z","stageTimestamp":"2024-03-26T15:16:46.553668Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"830ba321-7eb9-457a-8fbf-2d1aa6090958","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:53.375949Z","stageTimestamp":"2024-03-26T15:16:53.383788Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"dea81363-7843-469d-8963-02c9adbe0f09","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:53.385467Z","stageTimestamp":"2024-03-26T15:16:53.387871Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"4fb33c4a-cae0-4063-ac42-00292558d24c","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/eden-prime/pods/eden-software2","verb":"get","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"eden-prime","name":"eden-software2","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:01.866555Z","stageTimestamp":"2024-03-26T15:17:01.868806Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"7df67428-5bd0-4277-b352-e7984c807999","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps/webapp-config-map","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","code":200},"requestReceivedTimestamp":"2024-03-26T15:17:01.869155Z","stageTimestamp":"2024-03-26T15:17:01.895109Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"454e01cf-7932-4397-adee-69a93ac68902","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:01.897051Z","stageTimestamp":"2024-03-26T15:17:01.902153Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"e91303a6-3065-4d7b-a71e-278fd3646147","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:17:01.979056Z","stageTimestamp":"2024-03-26T15:17:01.984936Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"91c1a816-c46d-411a-b8b1-00c0fe0a4e88","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5755\u0026timeout=8m17s\u0026timeoutSeconds=497\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.995440Z","stageTimestamp":"2024-03-26T15:17:02.090269Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"89374f31-958e-42d6-ba69-9d344ecfb6ff","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5755\u0026timeout=5m38s\u0026timeoutSeconds=338\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:16:41.994581Z","stageTimestamp":"2024-03-26T15:17:02.090404Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"41afdafb-bfc5-4149-8bf6-79525464bce5","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"delete","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.063480Z","stageTimestamp":"2024-03-26T15:17:02.111751Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"90d37e44-f6ae-4ba4-ba0c-d160c57d8ec5","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Failure","reason":"NotFound","code":404},"requestReceivedTimestamp":"2024-03-26T15:17:02.072636Z","stageTimestamp":"2024-03-26T15:17:02.136833Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"4b1a5534-2c3b-4587-9975-5710dbadd0a1","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldSelector=metadata.name%3Dwebapp-color","verb":"list","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.131628Z","stageTimestamp":"2024-03-26T15:17:02.138106Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"560eafc3-4c08-4ed7-8b65-5d32e427d901","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"impersonatedUser":{"username":"agent-smith","groups":["system:authenticated"]},"sourceIPs":["192.168.121.1"],"userAgent":"kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2024-03-26T15:17:02.332027Z","stageTimestamp":"2024-03-26T15:17:02.338685Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\"","pod-security.kubernetes.io/enforce-policy":"privileged:latest"}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"27b19d52-34ad-4a60-8928-33091a38db7f","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.360194Z","stageTimestamp":"2024-03-26T15:17:02.362970Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"4fa143f0-7490-4243-a77b-044592f7b8cb","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026limit=500\u0026resourceVersion=0","verb":"list","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.363197Z","stageTimestamp":"2024-03-26T15:17:02.370534Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f7875d73-7406-46a2-8dc2-e8bff99e0351","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dwebapp-config-map\u0026resourceVersion=5801\u0026timeout=8m1s\u0026timeoutSeconds=481\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"webapp-config-map","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.373725Z","stageTimestamp":"2024-03-26T15:17:02.374496Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"1b653a25-25a7-4d1f-af4e-771ac06f0170","stage":"ResponseStarted","requestURI":"/api/v1/namespaces/citadel/configmaps?allowWatchBookmarks=true\u0026fieldSelector=metadata.name%3Dkube-root-ca.crt\u0026resourceVersion=5801\u0026timeout=5m33s\u0026timeoutSeconds=333\u0026watch=true","verb":"watch","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"configmaps","namespace":"citadel","name":"kube-root-ca.crt","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.374735Z","stageTimestamp":"2024-03-26T15:17:02.375067Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f6c2be32-ceed-4dc6-85b1-f1446ce51432","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:02.368182Z","stageTimestamp":"2024-03-26T15:17:02.381936Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"6b113bdd-31a4-4d37-81c2-2bf3cd95ac0d","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:04.608464Z","stageTimestamp":"2024-03-26T15:17:04.611396Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"730cb460-5452-4a83-a5c6-5cbdfb9a352b","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:12.768882Z","stageTimestamp":"2024-03-26T15:17:12.772078Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"db3a1a54-4514-45c6-8a81-b2988410e6df","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/citadel/pods/webapp-color","verb":"get","user":{"username":"system:node:controlplane","groups":["system:nodes","system:authenticated"]},"sourceIPs":["127.0.0.1"],"userAgent":"Go-http-client/2.0","objectRef":{"resource":"pods","namespace":"citadel","name":"webapp-color","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2024-03-26T15:17:12.778285Z","stageTimestamp":"2024-03-26T15:17:12.779863Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}

root@controlplane /etc/kubernetes/manifests ➜  cat /var/log/kubernetes/audit/audit.log | grep -i agent-smith | jq
 [....] {
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "20a12146-aa99-4410-b37e-2fcd9b34d25a",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/citadel/configmaps/webapp-config-map",
  "verb": "delete",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "impersonatedUser": {
    "username": "agent-smith",
    "groups": [
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.121.1"
  ],
  "userAgent": "kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47",
  "objectRef": {
    "resource": "configmaps",
    "namespace": "citadel",
    "name": "webapp-config-map",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "status": "Success",
    "code": 200
  },
  "requestReceivedTimestamp": "2024-03-26T15:17:21.862770Z",
  "stageTimestamp": "2024-03-26T15:17:21.884950Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""
  }
}
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "59efce65-b2af-41c7-a113-d7eae5332142",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/citadel/configmaps?fieldSelector=metadata.name%3Dwebapp-config-map",
  "verb": "list",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "impersonatedUser": {
    "username": "agent-smith",
    "groups": [
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.121.1"
  ],
  "userAgent": "kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47",
  "objectRef": {
    "resource": "configmaps",
    "namespace": "citadel",
    "name": "webapp-config-map",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 200
  },
  "requestReceivedTimestamp": "2024-03-26T15:17:21.886238Z",
  "stageTimestamp": "2024-03-26T15:17:21.888045Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""
  }
}
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "049528d5-c2c7-443a-9c0b-c01169663c5c",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/citadel/configmaps?fieldManager=kubectl-create",
  "verb": "create",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "impersonatedUser": {
    "username": "agent-smith",
    "groups": [
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.121.1"
  ],
  "userAgent": "kubectl/v1.20.0 (linux/amd64) kubernetes/af46c47",
  "objectRef": {
    "resource": "configmaps",
    "namespace": "citadel",
    "name": "webapp-config-map",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 201
  },
  "requestReceivedTimestamp": "2024-03-26T15:17:21.952664Z",
  "stageTimestamp": "2024-03-26T15:17:21.956138Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by RoleBinding \"important_binding_do_not_delete/citadel\" of Role \"important_role_do_not_delete\" to User \"agent-smith\""
  }
}
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "3bc19711-36a5-4075-840d-6bcf8fb822e0",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/citadel/pods/webapp-color",
  "verb": "delete",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "impersonatedUser": {
    "username": "agent-smith",
    "groups": [
      "system:authenticated"
    ]
  },  [....]

root@controlplane ~ ➜  nano /opt/blacklist_users
root@controlplane ~ ➜  cat /opt/blacklist_users
agent-smith,important_role_do_not_delete,important_binding_do_not_delete

root@controlplane ~ ✖ tail -n 30 /opt/falco.log
15:16:01.713000329: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:16:26.674088622: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:16:46.691139951: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:17:02.042788876: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:17:26.971669633: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:17:47.007663956: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:18:01.284682186: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:18:26.284574761: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:18:46.309372611: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:19:01.599809215: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:19:26.583039858: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:19:46.633522697: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:20:01.943801800: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:20:26.923887897: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:20:46.916358026: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:21:01.224010050: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:21:01.769721707: Warning Shell history had been deleted or renamed (user=root user_loginuid=-1 type=openat command=bash fd.name=/root/.bash_history name=/root/.bash_history path=<NA> oldpath=<NA> host (id=host))
15:21:26.220319043: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:21:46.216233185: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:22:01.535828663: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:22:26.511718502: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:22:46.494540128: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:23:01.789935124: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:23:26.773785520: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:23:46.782715034: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:24:02.080782638: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:24:27.052407857: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:24:47.055324607: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:25:01.348183227: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)
15:25:26.338514671: Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=3380897ebe94 container_name=k8s_eden-software2_eden-software2_eden-prime_110c765b-d30b-422e-813f-c91e925216ba_0 image=ubuntu:latest)

root@controlplane ~ ✖ k get pods -n eden-prime 
NAME                       READY   STATUS    RESTARTS   AGE
eden-fe-77574c68cd-mxlxx   1/1     Running   0          27m
eden-software1             1/1     Running   0          27m
eden-software2             1/1     Running   0          27m
eden-software3             1/1     Running   0          27m

root@controlplane ~ ➜  k get pods -n eden-prime eden-software2 -oyaml | grep -i container -A10
  containers:
  - args:
    - sleep
    - "200000"
    image: ubuntu
    imagePullPolicy: Always
    name: eden-software2
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
--
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-03-26T14:59:33Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://3380897ebe94d34db1bc69e96d0adea71aa1ca351855d627628b544daa8070b7
    image: ubuntu:latest
    imageID: docker-pullable://ubuntu@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e
    lastState: {}
    name: eden-software2
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-03-26T14:59:49Z"

root@controlplane ~ ➜  nano /opt/compromised_pods
root@controlplane ~ ➜  cat /opt/compromised_pods
eden-prime,eden-software2

root@controlplane ~ ➜  k delete pod -n eden-prime eden-software2 $now
pod "eden-software2" deleted

root@controlplane ~ ➜  k get role -n citadel 
NAME                           CREATED AT
dev1                           2024-03-26T14:59:32Z
important_citadel_user_role    2024-03-26T14:59:33Z
important_role_do_not_delete   2024-03-26T14:59:32Z

root@controlplane ~ ➜  k delete role -n citadel important_role_do_not_delete $now
role.rbac.authorization.k8s.io "important_role_do_not_delete" deleted

root@controlplane ~ ➜  k delete rolebindings.rbac.authorization.k8s.io -n citadel 
error: resource(s) were provided, but no name was specified

root@controlplane ~ ✖ k get rolebindings.rbac.authorization.k8s.io -n citadel
NAME                              ROLE                                AGE
dev1                              Role/dev1                           34m
important_binding_do_not_delete   Role/important_role_do_not_delete   34m
important_citadel_user_binding    Role/important_citadel_user_role    34m

root@controlplane ~ ➜  k delete rolebindings.rbac.authorization.k8s.io -n citadel important_binding_do_not_delete $now
rolebinding.rbac.authorization.k8s.io "important_binding_do_not_delete" deleted
```

## AFTER:

![image](https://i.imgur.com/BsSQjXY.png)


