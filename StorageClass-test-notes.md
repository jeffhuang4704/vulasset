# Some notes regarding NVSHAS-8534 - To block the usage of specific storage classes

## Links

-- [setup nfs on ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)  
-- [instll nfs dynamic provisioner for nfs](https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/)

## Notes I took back in 2023

[Some notes I took with scree shots](./materials/How-to-Set-Up-an-NFS-Server-on-Ubuntu.pdf)

## Existing environment

Feel free to login to play with it.

- my NFS server in the labs, IP: 10.1.45.49
- my cluster has install the nfs dynamic provisioner installed. IP: 10.1.45.40

## Some yaml

**PVC**

```
neuvector@ubuntu2204-A:~/sc_test$ cat pvc1.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim1
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

**Pod**

```
neuvector@ubuntu2204-A:~/sc_test$ cat pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim1
  containers:
    - name: task-pv-container1
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

**Deployment**

```
neuvector@ubuntu2204-A:~/sc_test$ cat deploy-use-pvc1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-dep-nginx
  name: my-dep-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-dep-nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-dep-nginx
    spec:
      volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: task-pv-claim1
      containers:
      - image: nginx
        name: nginx
        resources: {}
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: task-pv-storage
status: {}
neuvector@ubuntu2204-A:~/sc_test$
```

**StatefulSet**

```
neuvector@ubuntu2204-A:~/sc_test$ cat statefulset-1-node-my-nfs.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: rqlite
spec:
  selector:
    matchLabels:
      app: rqlite # has to match .spec.template.metadata.labels
  serviceName: rqlite-svc-internal
  replicas: 1
  podManagementPolicy: "Parallel"
  template:
    metadata:
      labels:
        app: rqlite # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: rqlite
        image: rqlite/rqlite
        args: ["-disco-mode=dns","-disco-config={\"name\":\"rqlite-svc-internal\"}","-bootstrap-expect=1", "-join-interval=1s", "-join-attempts=120"]
        ports:
        - containerPort: 4001
          name: rqlite
        readinessProbe:
          httpGet:
            scheme: HTTP
            path: /readyz
            port: 4001
          periodSeconds: 5
          timeoutSeconds: 2
          # As rqlite manages a larger and larger dataset, it can take longer
          # to be ready. This value may need to increase, depending on your experience.
          initialDelaySeconds: 10
        livenessProbe:
          httpGet:
            scheme: HTTP
            path: /readyz?noleader
            port: rqlite
          initialDelaySeconds: 2
          timeoutSeconds: 2
          failureThreshold: 3
        volumeMounts:
        - name: rqlite-file
          mountPath: /rqlite/file
      imagePullSecrets:
        - name: docker-secret
  volumeClaimTemplates:
  - metadata:
      name: rqlite-file
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "nfs-client"
      resources:
        requests:
          storage: 1Gi
neuvector@ubuntu2204-A:~/sc_test$
```
