This is a solution to the Certified Kubernetes Administrator Practice Test for Backup and Restore of ETCD Clust within a Kubernetes set up with kubeadm

## 1. Get etcdctl utility if it's not already present.

go get github.com/coreos/etcd/etcdctl

## 2. Backup

```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /tmp/snapshot-pre-boot.db
```

# -----------------------------
# Disaster Happens
# -----------------------------

## 3. Restore ETCD Snapshot to a new folder


```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=master \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-from-backup \
     --initial-cluster=master=https://127.0.0.1:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://127.0.0.1:2380 \
     snapshot restore /tmp/snapshot-pre-boot.db
```

## 4. Modify /etc/kubernetes/manifests/etcd.yaml

Update --data-dir to use new target location

```
--data-dir=/var/lib/etcd-from-backup
```

Update new initial-cluster-token to specify new cluster

```
--initial-cluster-token=etcd-cluster-1
```

Update volumes and volume mounts to point to new path

```
    volumeMounts:
    - mountPath: /var/lib/etcd-from-backup
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
```

After the manifest files are modified ETCD cluster should automatically restart
