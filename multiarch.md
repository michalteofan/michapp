# Lab OpenShift multi-architecture cluster

### Using the labs

#### Labs Requirements
1. VPN connection to the Lab (OpenVPN) - ip address, username and password provided by e-mail.
2. Admin account for OCP cluster - provided by e-mail.
3. Lab user account (labuserX) for lab workstation and OCP cluster - provided by e-mail.
4. Web browser (Firefox or Chrome).
6. SSH client.
7. Hosts file with entries based on an example below - IP addresses and FQDNs will be provided by e-mail.

```
10.XX.XX.XX	api.yyy.yyy.yy
10.XX.XX.XX	console-openshift-console.apps.yyy.yyy.yy
10.XX.XX.XX	oauth-openshift.apps.yyy.yyy.yy
10.XX.XX.XX	default-route-openshift-image-registry.apps.yyy.yyy.yy
10.XX.XX.XX	labuserXapp1-labuserX.apps.yyy.yyy.yy
10.XX.XX.XX	labuserXapp2-labuserX.apps.yyy.yyy.yy
```
where X is your lab user number, e.g. labuser1

#### Lab Env
OCP cluster API: https://api.yyy.yyy.yy:6443

OCP console: https://console-openshift-console.apps.yyy.yyy.yy
OCP cluster registry: https://default-route-openshift-image-registry.apps.yyy.yyy.yy

Lab workstation: ssh labuserX@10.ZZ.ZZ.ZZ

## Login to OpenShift and create new project

```
oc login -u labuser0 -p https://api.yyy.yyy.yy:6443
```

## List nodes in cluster

```
[labuser0@5437wks ~]$ oc get nodes
NAME                             STATUS   ROLES                  AGE    VERSION
5437baw10m1.yyy.yyy.yy   Ready    control-plane,master   10d    v1.28.7+f1b5f6c
5437baw10m2.yyy.yyy.yy   Ready    control-plane,master   9d     v1.28.7+f1b5f6c
5437baw10m3.yyy.yyy.yy   Ready    control-plane,master   9d     v1.28.7+f1b5f6c
5437baw10w1.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c
5437baw10w2.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c
5437baw10w3.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c
5437tenobcy.yyy.yyy.yy   Ready    worker                 3d8h   v1.28.7+f1b5f6c
```
```
[labuser0@5437wks ~]$ oc get nodes -o wide
NAME                             STATUS   ROLES                  AGE    VERSION           INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                                                       KERNEL-VERSION                  CONTAINER-RUNTIME
5437baw10m1.yyy.yyy.yy   Ready    control-plane,master   10d    v1.28.7+f1b5f6c   10.110.140.64   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437baw10m2.yyy.yyy.yy   Ready    control-plane,master   9d     v1.28.7+f1b5f6c   10.110.140.65   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437baw10m3.yyy.yyy.yy   Ready    control-plane,master   9d     v1.28.7+f1b5f6c   10.110.140.66   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437baw10w1.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c   10.110.140.67   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437baw10w2.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c   10.110.140.68   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437baw10w3.yyy.yyy.yy   Ready    worker                 9d     v1.28.7+f1b5f6c   10.110.140.69   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.ppc64le   cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
5437tenobcy.yyy.yyy.yy   Ready    worker                 3d8h   v1.28.7+f1b5f6c   10.110.140.70   <none>        Red Hat Enterprise Linux CoreOS 415.92.202403191241-0 (Plow)   5.14.0-284.57.1.el9_2.x86_64    cri-o://1.28.4-5.rhaos4.15.git159b3c8.el9
```

## List nodes in cluster with ppc64le and adm64 architecture

```
[labuser0@5437wks ~]$ oc get nodes -l kubernetes.io/arch=ppc64le
NAME                             STATUS   ROLES                  AGE   VERSION
5437baw10m1.yyy.yyy.yy   Ready    control-plane,master   10d   v1.28.7+f1b5f6c
5437baw10m2.yyy.yyy.yy   Ready    control-plane,master   9d    v1.28.7+f1b5f6c
5437baw10m3.yyy.yyy.yy   Ready    control-plane,master   9d    v1.28.7+f1b5f6c
5437baw10w1.yyy.yyy.yy   Ready    worker                 9d    v1.28.7+f1b5f6c
5437baw10w2.yyy.yyy.yy   Ready    worker                 9d    v1.28.7+f1b5f6c
5437baw10w3.yyy.yyy.yy   Ready    worker                 9d    v1.28.7+f1b5f6c

[labuser0@5437wks ~]$ oc get nodes -l kubernetes.io/arch=amd64
NAME                             STATUS   ROLES    AGE    VERSION
5437tenobcy.yyy.yyy.yy   Ready    worker   3d8h   v1.28.7+f1b5f6c
```

## Check architecture specific Machine Config Pools

```
oc get mcp
```

## Login to OpenShift and create new project

```
oc login -u labuser0 -p https://api.yyy.yyy.yy:6443
oc new-project labuser0
```

## Preapare deployemnt for Fredora Linux caontainer using tag latest
Note: no architecture specified in the image.

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: fedora-labuser0
  namespace: labuser0
  labels:
    app: fedora-labuser0
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fedora-labuser0
  template:
    metadata:
      labels:
        app: fedora-labuser0
    spec:
      containers:
          - name: fedora-labuser0
            command:
            - sleep
            args:
            - infinity
            imagePullPolicy: Always
            image: 'docker.io/fedora:latest'
```

## Run Fedora deployment

```
[labuser0@5437wks ~]$ oc apply -f fedora-labuser0-depl.yaml 
deployment.apps/fedora-labuser0 created
```

## Check pod status, worker node and pod architecture

```
[labuser0@5437wks ~]$ oc get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE                             NOMINATED NODE   READINESS GATES
fedora-labuser0-66d8b9595f-qfxw5   1/1     Running   0          41s   10.129.0.19   5437tenobcy.yyy.yyy.yy   <none>           <none>

[labuser0@5437wks ~]$ oc describe node 5437tenobcy.yyy.yyy.yy | grep arch
Labels:             beta.kubernetes.io/arch=amd64
                    kubernetes.io/arch=amd64

[labuser0@5437wks ~]$ oc rsh fedora-labuser0-66d8b9595f-qfxw5 uname -a
Linux fedora-labuser0-66d8b9595f-qfxw5 5.14.0-284.57.1.el9_2.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Mar 1 09:45:44 EST 2024 x86_64 GNU/Linux
```

## Create new deployment file and add node selector

```
[labuser0@5437wks ~]$ cat fedora-labuser0-depl-ppc64le.yaml 
kind: Deployment
apiVersion: apps/v1
metadata:
  name: fedora-labuser0-ppc64le
  namespace: labuser0
  labels:
    app: fedora-labuser0-ppc64le
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fedora-labuser0-ppc64le
  template:
    metadata:
      labels:
        app: fedora-labuser0-ppc64le
    spec:
      containers:
          - name: fedora-labuser0-ppc64le
            command:
            - sleep
            args:
            - infinity
            imagePullPolicy: Always
            image: 'docker.io/fedora:latest'
      nodeSelector:
        kubernetes.io/arch: ppc64le
```

## Run the new deployment

```
[labuser0@5437wks ~]$ oc apply -f fedora-labuser0-depl-ppc64le.yaml 
deployment.apps/fedora-labuser0-ppc64le created
```

## Check pod status and worker node and pod architecture for the new deployment

```
[labuser0@5437wks ~]$ oc get pods -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE                             NOMINATED NODE   READINESS GATES
fedora-labuser0-66d8b9595f-qfxw5           1/1     Running   0          22m   10.129.0.19   5437tenobcy.yyy.yyy.yy   <none>           <none>
fedora-labuser0-ppc64le-6d45fd45bf-7k9tw   1/1     Running   0          15s   10.130.2.16   5437baw10w3.yyy.yyy.yy   <none>           <none>
```
```
[labuser0@5437wks ~]$ oc describe node 5437baw10w3.yyy.yyy.yy | grep arch
Labels:             beta.kubernetes.io/arch=ppc64le
                    kubernetes.io/arch=ppc64le
```
```
[labuser0@5437wks ~]$ oc rsh fedora-labuser0-ppc64le-6d45fd45bf-7k9tw uname -a
Linux fedora-labuser0-ppc64le-6d45fd45bf-7k9tw 5.14.0-284.57.1.el9_2.ppc64le #1 SMP Fri Mar 1 08:54:39 EST 2024 ppc64le GNU/Linux
```
## Deploy Fedora Linux from OCP console

Login to Openshift console as labuserX, where X is your lab user number:
https://console-openshift-console.apps.yyy.yyy.yy/
with htpasswd provider.

1. Switch to developer view.
2. Select +Add and From image.
3. Image: https://docker.io/fedora
4. Application name: labuserXapp1
5. Name: labuserXapp1
6. Click Create

Check pod status. If pod is in error state debug why it is not running using following commands:
```
oc get pod
oc describe pod
oc logs pod
oc get node
oc describe node
```
and
https://

