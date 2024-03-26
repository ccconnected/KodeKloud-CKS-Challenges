# Challenge 1: https://kodekloud.com/topic/lab-challenge-1/

## BEFORE:

![image](https://i.imgur.com/drgmtWx.png)

## TASK:
There are 6 images listed in the diagram on the right. Using Aquasec Trivy (which is already installed on the controlplane node), identify the image that has the least number of critical vulnerabilities and use it to deploy the alpha-xyz deployment.

Secure this deployment by enforcing the AppArmor profile called custom-nginx.

Expose this deployment with a ClusterIP type service and make sure that only incomings connections from the pod called middleware is accepted and everything else is rejected.

Click on each icon to see more details. Once done, click the Check button to test your work.

1) Info on "images"
-    Permitted images are: 'nginx:alpine', 'bitnami/nginx', 'nginx:1.13', 'nginx:1.17', 'nginx:1.16'and 'nginx:1.14'. Use 'trivy' to find the image with the least number of 'CRITICAL' vulnerabilities.

2) Info on "alpha-xyz"
-    Create a deployment called 'alpha-xyz' that uses the image with the least 'CRITICAL' vulnerabilities? (Use the sample YAML file located at '/root/alpha-xyz.yaml' to create the deployment. 
-    Please make sure to use the same names and labels specified in this sample YAML file!)
-    Deployment has exactly '1' ready replica
-    'data-volume' is mounted at '/usr/share/nginx/html' on the pod

3) Info on "alpha-pv"
-    A persistentVolume called 'alpha-pv' has already been created. Do not modify it and inspect the parameters used to create it.

4) Info on "alpha-pvc"
-    'alpha-pvc' should be bound to 'alpha-pv'. Delete and Re-create it if necessary.

5) Info on "alpha-svc"
-    Expose the 'alpha-xyz' as a 'ClusterIP' type service called 'alpha-svc'
-    'alpha-svc' should be exposed on 'port: 80' and 'targetPort: 80'

6) Info on "custom-nginx"
-    Move the AppArmor profile '/root/usr.sbin.nginx' to '/etc/apparmor.d/usr.sbin.nginx' on the controlplane node.
-    Load the 'AppArmor` profile called 'custom-nginx' and ensure it is enforced

7) Info on arrow pointing from "custom-nginx" to "alpha-xyz"
'alpha-xyz' deployment uses the 'custom-nginx' apparmor profile (applied to container called 'nginx')

8) Info on "restrict-inbound"
-    Create a NetworkPolicy called 'restrict-inbound' in the 'alpha' namespace
-    Policy Type = 'Ingress'
-    Inbound access only allowed from the pod called 'middleware' with label 'app=middleware'
-    Inbound access only allowed to TCP port 80 on pods matching the policy

9) Info on arrow pointing from "restrict-inbound" to "alpha-xyz"
Policy should be only applied on pods with label 'app=alpha-xyz'

10) Info on "middleware"
-    Pod called 'middleware' is already deployed in the 'alpha' namespace. Inspect it but do not alter it in anyway!

11) Info on "external"
-    Pod called 'external' is already deployed in the 'alpha' namespace. Inspect it but do not alter it in anyway!
-    'external' pod should NOT be able to connect to 'alpha-svc' on port 80

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

root@controlplane ~ ➜  trivy image nginx:alpine | grep -i Total
Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)

root@controlplane ~ ➜  trivy image bitnami/nginx | grep -i Total
Total: 70 (UNKNOWN: 10, LOW: 4, MEDIUM: 33, HIGH: 21, CRITICAL: 2)

root@controlplane ~ ➜  trivy image nginx:1.13 | grep -i Total
Total: 533 (UNKNOWN: 15, LOW: 22, MEDIUM: 211, HIGH: 200, CRITICAL: 85)

root@controlplane ~ ➜  trivy image nginx:1.17 | grep -i Total
Total: 325 (UNKNOWN: 6, LOW: 15, MEDIUM: 141, HIGH: 120, CRITICAL: 43)

root@controlplane ~ ➜  trivy image nginx:1.16 | grep -i Total
Total: 329 (UNKNOWN: 6, LOW: 15, MEDIUM: 143, HIGH: 122, CRITICAL: 43)

root@controlplane ~ ➜  trivy image nginx:1.14 | grep -i Total
Total: 458 (UNKNOWN: 13, LOW: 18, MEDIUM: 196, HIGH: 167, CRITICAL: 64)

-> we proceed to use nginx:alpine because it contains the least amount of CRITICAL

root@controlplane ~ ➜  cat /root/alpha-xyz.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: alpha-xyz
  name: alpha-xyz
  namespace: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-xyz
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: alpha-xyz
    spec:
      containers:
      - image: ?
        name: nginx

root@controlplane ~ ✖ kubectl get pvc -A
NAMESPACE   NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
alpha       alpha-pvc   Pending                                      local-storage   39m

root@controlplane ~ ➜  kubectl get pv -A
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
alpha-pv   1Gi        RWX            Delete           Available           local-storage            39m

root@controlplane ~ ➜  kubectl get pv -oyaml
apiVersion: v1
items:
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"name":"alpha-pv"},"spec":{"accessModes":["ReadWriteMany"],"capacity":{"storage":"1Gi"},"local":{"path":"/data/pages"},"nodeAffinity":{"required":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"kubernetes.io/hostname","operator":"In","values":["controlplane"]}]}]}},"persistentVolumeReclaimPolicy":"Delete","storageClassName":"local-storage","volumeMode":"Filesystem"}}
    creationTimestamp: "2024-03-24T14:26:48Z"
    finalizers:
    - kubernetes.io/pv-protection
    name: alpha-pv
    resourceVersion: "6077"
    uid: e8b624f4-8e09-4d7f-8792-6a9ffb03e6dc
  spec:
    accessModes:
    - ReadWriteMany
    capacity:
      storage: 1Gi
    local:
      path: /data/pages
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - controlplane
    persistentVolumeReclaimPolicy: Delete
    storageClassName: local-storage
    volumeMode: Filesystem
  status:
    phase: Available
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

root@controlplane ~ ➜  kubectl get pvc -n alpha alpha-pvc -oyaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"alpha-pvc","namespace":"alpha"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"1Gi"}},"storageClassName":"local-storage"}}
  creationTimestamp: "2024-03-24T14:26:48Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: alpha-pvc
  namespace: alpha
  resourceVersion: "6076"
  uid: ef482bd9-fc2e-4827-a33d-537779f57b37
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
  volumeMode: Filesystem
status:
  phase: Pending

root@controlplane ~ ➜  kubectl get pvc -n alpha alpha-pvc -oyaml > alpha-pvc-bak.yaml

root@controlplane ~ ➜  k edit pvc -n alpha alpha-pvc 
error: persistentvolumeclaims "alpha-pvc" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-3252216165.yaml"
error: Edit cancelled, no valid changes were saved.

root@controlplane ~ ➜  cat /tmp/kubectl-edit-3252216165.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"alpha-pvc","namespace":"alpha"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"1Gi"}},"storageClassName":"local-storage"}}
  creationTimestamp: "2024-03-24T14:26:48Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: alpha-pvc
  namespace: alpha
  resourceVersion: "6076"
  uid: ef482bd9-fc2e-4827-a33d-537779f57b37
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
  volumeMode: Filesystem
status:
  phase: Pending

root@controlplane ~ ✖ k delete pvc -n alpha alpha-pvc 
persistentvolumeclaim "alpha-pvc" deleted

root@controlplane ~ ➜  k apply -f /tmp/kubectl-edit-3252216165.yaml
persistentvolumeclaim/alpha-pvc created

root@controlplane ~ ➜  kubectl get pvc -A
NAMESPACE   NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
alpha       alpha-pvc   Bound    alpha-pv   1Gi        RWX            local-storage   20s

root@controlplane ~ ➜  kubectl get pv -A
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS    REASON   AGE
alpha-pv   1Gi        RWX            Delete           Bound    alpha/alpha-pvc   local-storage            52m

root@controlplane ~ ➜  cp /root/alpha-xyz.yaml /root/alpha-xyz-bak.yaml

root@controlplane ~ ➜  nano /root/alpha-xyz.yaml

root@controlplane ~ ➜  cat /root/alpha-xyz.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: alpha-xyz
  name: alpha-xyz
  namespace: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-xyz
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: alpha-xyz
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: data-volume
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: alpha-pvc

root@controlplane ~ ➜  k apply -f /root/alpha-xyz.yaml 
deployment.apps/alpha-xyz created

root@controlplane ~ ➜  k get deploy -n alpha -w
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
alpha-xyz   1/1     1            1           14s

root@controlplane ~ ➜  k expose -n alpha deploy alpha-xyz --port=80 --target-port=80 --name=alpha-svc
service/alpha-svc exposed

root@controlplane ~ ➜  k get -n alpha svc alpha-svc 
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
alpha-svc   ClusterIP   10.105.167.193   <none>        80/TCP    20s

root@controlplane ~ ➜  k get -n alpha svc alpha-svc -owide
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
alpha-svc   ClusterIP   10.105.167.193   <none>        80/TCP    25s   app=alpha-xyz

root@controlplane ~ ➜  cp /root/usr.sbin.nginx /root/usr.sbin.nginx-bak
root@controlplane ~ ➜  mv /root/usr.sbin.nginx /etc/apparmor.d/usr.sbin.nginx

root@controlplane ~ ✖ aa-status 
apparmor module is loaded.
56 profiles are loaded.
19 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/lxc-start
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/chromium-browser/chromium-browser//browser_java
   /usr/lib/chromium-browser/chromium-browser//browser_openjdk
   /usr/lib/chromium-browser/chromium-browser//sanitized_helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   docker-default
   lxc-container-default
   lxc-container-default-cgns
   lxc-container-default-with-mounting
   lxc-container-default-with-nesting
   man_filter
   man_groff
37 profiles are in complain mode.
   /usr/lib/chromium-browser/chromium-browser
   /usr/lib/chromium-browser/chromium-browser//chromium_browser_sandbox
   /usr/lib/chromium-browser/chromium-browser//lsb_release
   /usr/lib/chromium-browser/chromium-browser//xdgsettings
   /usr/lib/dovecot/anvil
   /usr/lib/dovecot/auth
   /usr/lib/dovecot/config
   /usr/lib/dovecot/deliver
   /usr/lib/dovecot/dict
   /usr/lib/dovecot/dovecot-auth
   /usr/lib/dovecot/dovecot-lda
   /usr/lib/dovecot/dovecot-lda///usr/sbin/sendmail
   /usr/lib/dovecot/imap
   /usr/lib/dovecot/imap-login
   /usr/lib/dovecot/lmtp
   /usr/lib/dovecot/log
   /usr/lib/dovecot/managesieve
   /usr/lib/dovecot/managesieve-login
   /usr/lib/dovecot/pop3
   /usr/lib/dovecot/pop3-login
   /usr/lib/dovecot/ssl-params
   /usr/sbin/avahi-daemon
   /usr/sbin/dnsmasq
   /usr/sbin/dnsmasq//libvirt_leaseshelper
   /usr/sbin/dovecot
   /usr/sbin/identd
   /usr/sbin/mdnsd
   /usr/sbin/nmbd
   /usr/sbin/nscd
   /usr/sbin/smbd
   /usr/sbin/smbldap-useradd
   /usr/sbin/smbldap-useradd///etc/init.d/nscd
   /usr/{sbin/traceroute,bin/traceroute.db}
   klogd
   ping
   syslog-ng
   syslogd
20 processes have profiles defined.
20 processes are in enforce mode.
   docker-default (2253) 
   docker-default (2270) 
   docker-default (2280) 
   docker-default (2292) 
   docker-default (2398) 
   docker-default (2461) 
   docker-default (2467) 
   docker-default (2495) 
   docker-default (4130) 
   docker-default (4151) 
   docker-default (4312) 
   docker-default (4432) 
   docker-default (27782) 
   docker-default (27810) 
   docker-default (27993) 
   docker-default (28048) 
   docker-default (29299) 
   docker-default (29418) 
   docker-default (29462) 
   docker-default (29463) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

root@controlplane ~ ➜  cat /etc/apparmor.d/usr.sbin.nginx
#include <tunables/global>

profile custom-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  network inet tcp,
  network inet udp,
  network inet icmp,

  deny network raw,

  deny network packet,

  file,
  umount,

  deny /bin/** wl,
  deny /usr/share/nginx/html/restricted/* rw,
  deny /boot/** wl,
  deny /dev/** wl,
  deny /etc/** wl,
  deny /home/** wl,
  deny /lib/** wl,
  deny /lib64/** wl,
  deny /media/** wl,
  deny /mnt/** wl,
  deny /opt/** wl,
  deny /proc/** wl,
  deny /root/** wl,
  deny /sbin/** wl,
  deny /srv/** wl,
  deny /tmp/** wl,
  deny /sys/** wl,
  deny /usr/** wl,

  audit /** w,

  /var/run/nginx.pid w,

  /usr/sbin/nginx ix,

  deny /bin/dash mrwklx,
  deny /bin/sh mrwklx,
  deny /usr/bin/top mrwklx,

  capability chown,
  capability dac_override,
  capability setuid,
  capability setgid,
  capability net_bind_service,

  deny @{PROC}/{*,**^[0-9*],sys/kernel/shm*} wkx,
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,
  deny mount,
  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/efi/efivars/** rwklx,
  deny /sys/kernel/security/** rwklx,
}

root@controlplane ~ ➜  apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx

root@controlplane ~ ➜  aa-status
apparmor module is loaded.
57 profiles are loaded.
20 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/lxc-start
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/chromium-browser/chromium-browser//browser_java
   /usr/lib/chromium-browser/chromium-browser//browser_openjdk
   /usr/lib/chromium-browser/chromium-browser//sanitized_helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   custom-nginx
   docker-default
   lxc-container-default
   lxc-container-default-cgns
   lxc-container-default-with-mounting
   lxc-container-default-with-nesting
   man_filter
   man_groff
37 profiles are in complain mode.
   /usr/lib/chromium-browser/chromium-browser
   /usr/lib/chromium-browser/chromium-browser//chromium_browser_sandbox
   /usr/lib/chromium-browser/chromium-browser//lsb_release
   /usr/lib/chromium-browser/chromium-browser//xdgsettings
   /usr/lib/dovecot/anvil
   /usr/lib/dovecot/auth
   /usr/lib/dovecot/config
   /usr/lib/dovecot/deliver
   /usr/lib/dovecot/dict
   /usr/lib/dovecot/dovecot-auth
   /usr/lib/dovecot/dovecot-lda
   /usr/lib/dovecot/dovecot-lda///usr/sbin/sendmail
   /usr/lib/dovecot/imap
   /usr/lib/dovecot/imap-login
   /usr/lib/dovecot/lmtp
   /usr/lib/dovecot/log
   /usr/lib/dovecot/managesieve
   /usr/lib/dovecot/managesieve-login
   /usr/lib/dovecot/pop3
   /usr/lib/dovecot/pop3-login
   /usr/lib/dovecot/ssl-params
   /usr/sbin/avahi-daemon
   /usr/sbin/dnsmasq
   /usr/sbin/dnsmasq//libvirt_leaseshelper
   /usr/sbin/dovecot
   /usr/sbin/identd
   /usr/sbin/mdnsd
   /usr/sbin/nmbd
   /usr/sbin/nscd
   /usr/sbin/smbd
   /usr/sbin/smbldap-useradd
   /usr/sbin/smbldap-useradd///etc/init.d/nscd
   /usr/{sbin/traceroute,bin/traceroute.db}
   klogd
   ping
   syslog-ng
   syslogd
20 processes have profiles defined.
20 processes are in enforce mode.
   docker-default (2253) 
   docker-default (2270) 
   docker-default (2280) 
   docker-default (2292) 
   docker-default (2398) 
   docker-default (2461) 
   docker-default (2467) 
   docker-default (2495) 
   docker-default (4130) 
   docker-default (4151) 
   docker-default (4312) 
   docker-default (4432) 
   docker-default (27782) 
   docker-default (27810) 
   docker-default (27993) 
   docker-default (28048) 
   docker-default (29299) 
   docker-default (29418) 
   docker-default (29462) 
   docker-default (29463) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

root@controlplane ~ ➜  cp /root/alpha-xyz.yaml /root/alpha-xyz-bak-2.yaml

root@controlplane ~ ➜  nano /root/alpha-xyz.yaml

root@controlplane ~ ➜  cat /root/alpha-xyz.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: alpha-xyz
  name: alpha-xyz
  namespace: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-xyz
  strategy: {}
  template:
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/nginx: localhost/custom-nginx
      creationTimestamp: null
      labels:
        app: alpha-xyz
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: data-volume
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: alpha-pvc

root@controlplane ~ ➜  k replace -f /root/alpha-xyz.yaml
deployment.apps/alpha-xyz replaced

root@controlplane ~ ➜  k get deploy -n alpha alpha-xyz -w
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
alpha-xyz   1/1     1            1           18m

root@controlplane ~ ✖ k get pod -n alpha --show-labels
NAME                         READY   STATUS    RESTARTS   AGE   LABELS
alpha-xyz-75ddb5d4b6-xg28d   1/1     Running   0          22m   app=alpha-xyz,pod-template-hash=75ddb5d4b6
external                     1/1     Running   0          25m   app=external
middleware                   1/1     Running   0          25m   app=middleware

root@controlplane ~ ➜  nano netpol-restrict-inbound.yaml

root@controlplane ~ ➜  cat netpol-restrict-inbound.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-inbound
  namespace: alpha
spec:
  podSelector:
    matchLabels:
      app: alpha-xyz
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: middleware
    ports:
    - protocol: TCP
      port: 80

root@controlplane ~ ➜  k apply -f netpol-restrict-inbound.yaml
networkpolicy.networking.k8s.io/restrict-inbound created

root@controlplane ~ ✖ k get -n alpha endpoints
NAME        ENDPOINTS      AGE
alpha-svc   10.50.0.6:80   43m

root@controlplane ~ ➜  k exec -it -n alpha middleware -- sh
/ # 
/ # curl alpha-xyz
sh: curl: not found
/ # nc alpha-svc -v 80
alpha-svc (10.105.167.193:80) open
^Cpunt!

root@controlplane ~ ✖ k exec -it -n alpha external -- sh
/ # 
/ # nc alpha-svc -v 80
^Cpunt!
```

## AFTER:

![image](https://i.imgur.com/nTOAc3m.png)

