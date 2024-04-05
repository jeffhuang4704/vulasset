## NVSHAS-8534 - To block the usage of specific storage classes

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Design](#section-2-design)

## Section 1: Background

Hello Team,

Can we block the usage of specific storage classes with NeuVector? Maybe with Admission Controls?

There are some predefined storage classes that users should not use; they need to be blocked.

Please let me know if any information is needed.

Thanks.

TODO: mention Kubewarden implementation

## Section 2: Kubernetes resources

TODO: explain a bit about how a workload uses storageclass.

- illustrate the workload and PVC
- (1) use PVC
- (2) use PVC template (Statefulset)
  (show real yaml)

TODO: mentioned the resource creation, use the diagram in the powerpoint..
TODO: show the use case... create deployment first, check the status in pending and then create PVC.. PV. show the screen shot

```
neuvector@ubuntu2204-A:~/temp$ kubectl apply -f deploy.yaml
deployment.apps/my-dep created

neuvector@ubuntu2204-A:~/temp$ kubectl get deploy my-dep
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-dep   0/1     1            0           16s

neuvector@ubuntu2204-A:~/temp$ kubectl get pod
NAME                                               READY   STATUS    RESTARTS      AGE
my-dep-84b7cf5584-jp84g                            0/1     Pending   0             30s

neuvector@ubuntu2204-A:~/temp$ kubectl describe pod my-dep-84b7cf5584-jp84g
Status:         Pending
Containers:
  nginx:
    Image:        nginx
    Mounts:
      /usr/share/nginx/html from task-pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v657h (ro)

Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  40s   default-scheduler  0/3 nodes are available: 3 persistentvolumeclaim "task-pv-claim" not found.
```

**Create dependent PVC**

```
neuvector@ubuntu2204-A:~/temp$ kubectl apply -f pvc.yaml
persistentvolumeclaim/task-pv-claim created
neuvector@ubuntu2204-A:~/temp$ kubectl apply -f pv.yaml
persistentvolume/task-pv-volume created

neuvector@ubuntu2204-A:~/temp$ kubectl get pods
NAME                                               READY   STATUS    RESTARTS      AGE
my-dep-84b7cf5584-jp84g                            1/1     Running   0             3m47s
```

<p align="left">
<img src="./materials/pvc-3.png" width="80%">
</p>

## Section 3: Admission Controller behavior

TODO: mention the validation call from k8s

## Section 4: Proposal plan

Add a new criteria, "certain storage classname usage"
Describe the plan... remember the validation request which has the storageclass criteria

show the case

- (1) when workload request comes in, the PVC information is ready
- (2) when workload request comes in, the PVC information is NOT ready

including the UI change

## Section 5: References

<p align="left">
<img src="./materials/pvc-1.png" width="80%">
</p>

<p align="left">
<img src="./materials/pvc-2.png" width="80%">
</p>
