# Some notes regarding NVSHAS-8534 - To block the usage of specific storage classes

## Links

-- [setup nfs on ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)  
-- [instll nfs dynamic provisioner for nfs](https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/)

## Notes I took back in 2023

[Some notes I took with scree shots](./materials/How-to-Set-Up-an-NFS-Server-on-Ubuntu.pdf)

## Existing environment

Feel free to login to play with it.

- my NFS server in the labs, IP: `10.1.45.49`
- my cluster has install the nfs dynamic provisioner installed. IP: `10.1.45.40`

**StorageClass**

```
neuvector@ubuntu2204-A:~/sc_test$ kubectl get sc nfs-client -oyaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    meta.helm.sh/release-name: nfs-subdir-external-provisioner
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2023-10-26T23:45:33Z"
  labels:
    app: nfs-subdir-external-provisioner
    app.kubernetes.io/managed-by: Helm
    chart: nfs-subdir-external-provisioner-4.0.18
    heritage: Helm
    release: nfs-subdir-external-provisioner
  name: nfs-client
  resourceVersion: "36373009"
  uid: 8c917313-3073-4432-aa24-3b44e91debf5
parameters:
  archiveOnDelete: "true"
  onDelete: "true"
provisioner: cluster.local/nfs-subdir-external-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

**nfs client provisioner**

```
neuvector@ubuntu2204-A:~/sc_test$ kubectl get pod nfs-subdir-external-provisioner-78859bfd68-snlph -oyaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-11-04T05:23:28Z"
  generateName: nfs-subdir-external-provisioner-78859bfd68-
  labels:
    app: nfs-subdir-external-provisioner
    pod-template-hash: 78859bfd68
    release: nfs-subdir-external-provisioner
  name: nfs-subdir-external-provisioner-78859bfd68-snlph
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nfs-subdir-external-provisioner-78859bfd68
    uid: 64c6017b-8c41-47fb-bd42-edb82175b2c9
  resourceVersion: "64009431"
  uid: 72fbff06-d531-468d-9e89-c4c2a5aa7d74
spec:
  containers:
  - env:
    - name: PROVISIONER_NAME
      value: cluster.local/nfs-subdir-external-provisioner
    - name: NFS_SERVER
      value: 10.1.45.49
    - name: NFS_PATH
      value: /exports/backup
    image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
    imagePullPolicy: IfNotPresent
    name: nfs-subdir-external-provisioner
    resources: {}
    securityContext: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /persistentvolumes
      name: nfs-subdir-external-provisioner-root
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-88w76
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: ubuntu2204-a
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: nfs-subdir-external-provisioner
  serviceAccountName: nfs-subdir-external-provisioner
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
  - name: nfs-subdir-external-provisioner-root
    nfs:
      path: /exports/backup
      server: 10.1.45.49
  - name: kube-api-access-88w76
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
```

## Use REST API to add rule

```
curl -X POST -k -H "Content-Type: application/json" -H "X-Auth-Apikey: test1:KdrdVJZE+vNMgqF79v9xEDjH40JRFM/8Q+7vWZ1UufjNPfZZiZp6fmto4zFo3mS9" "https://$K8sNodeIP:$ControllerSvcPORT/v1/admission/rule" --data-binary @rule_sc_name.json

// file: rule_sc_name.json
{
        "config": {
                "category": "Kubernetes",
                "cfg_type": "user_created",
                "comment": "block storage class name",
                "criteria": [
                   {
                    "name": "namespace",
                    "op": "containsAny",
                    "path": "namespace",
                    "value": "ns,ns8",
                    "value_type": "string"
                   },
                   {
                    "name": "storageClassName",
                    "op": "containsAny",
                    "path": "namespace",
                    "value": "nfs-client",
                    "value_type": "string"
                   }
                ],
                "critical": false,
                "disable": false,
                "id": 0,
                "rule_type": "deny"
        }
}
```

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
