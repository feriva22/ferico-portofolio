---
title: 'Running Kubernetes Locally with k3s: A Practical Lab Setup'
date: 2026-09-24T09:00:00+07:00
draft: false
type: 'blog'
tags:
  - kubernetes
  - k3s
  - docker
  - devops
  - homelab
  - cloud-native
topics:
  - Kubernetes
featured: false
weight: 21
images:
  featured_image: '/img/blog/feature-sample-3.png'
---

Every DevOps engineer eventually needs a Kubernetes cluster they can break without consequences. Not a shared staging cluster where a mistake pages someone, and not a managed cloud cluster that costs money whether you use it or not — a local cluster where you can delete a namespace, corrupt an etcd snapshot, or test a bad RBAC policy and simply rebuild it.

**k3s** is the most practical way to get there. It is a CNCF-certified Kubernetes distribution in a single binary under 100 MB, and it runs comfortably on a laptop.

This post covers why k3s over the alternatives, how to get it running, and — more usefully — how to build a lab that actually mirrors production concerns like multi-node scheduling, storage, ingress, and observability.

## Why k3s instead of the alternatives

| Option | Strength | Why it often falls short for a lab |
| --- | --- | --- |
| **minikube** | Very easy start | Heavy VM/container layer; multi-node requires extra drivers |
| **kind** | Excellent for CI, fast to create/destroy | Nodes are containers; networking mirrors CI better than production |
| **Docker Desktop K8s** | Zero setup | Single node, limited control, tied to Docker Desktop licensing |
| **k3s** | Real multi-node, small footprint, production-credible defaults | Slightly more setup than a one-liner |

k3s wins for a **learning lab** specifically because it is a genuine multi-node Kubernetes distribution. You get real nodes, real scheduling constraints, real `kubectl` behaviour, and a packaging that fits in a single binary. It also uses **containerd** by default rather than Docker, which is what you will encounter on most managed platforms today.

It is also not a toy. k3s runs in production at the edge for many organisations, so what you learn transfers directly.

## Prerequisites

- **Linux host** (native, VM, or WSL2). k3s is designed for Linux; on macOS and Windows you will run the server inside a Linux VM.
- At least **2 CPUs and 2 GB RAM** for the server. 4 GB is more comfortable if you plan to run observability workloads on it.
- **sudo** access and no existing Kubernetes components on the machine.

### A note on Windows

If your daily machine is Windows, the cleanest path is a Linux VM rather than fighting the tooling. WSL2 works but the systemd integration needed for the k3s service can be fiddly. A small Ubuntu Server VM with 4 GB and 2 vCPUs is a better investment, and it lets you add worker VMs later for a genuine multi-node lab.

## Installing the server node

The install script is genuinely all you need for a first cluster:

```bash
# Single-node: control plane + worker on the same host
curl -sfL https://get.k3s.io | sh -

# Verify
sudo kubectl get nodes
sudo kubectl get pods -A
```

By default k3s bundles **Traefik** as the ingress controller, **ServiceLB** as the load balancer, **local-path-provisioner** for dynamic storage, and **CoreDNS** and **Metrics Server**. That means you have a cluster that can serve traffic and report resource usage without installing anything else — a meaningful difference from a bare `kubeadm` cluster.

If you want to skip Traefik and run your own ingress (say NGINX), disable it at install time:

```bash
curl -sfL https://get.k3s.io | \
  INSTALL_K3S_EXEC="--disable=traefik" sh -
```

## Making kubectl usable without sudo

The kubeconfig is written to `/etc/rancher/k3s/k3s.yaml` with root-only permissions. Set up your user properly rather than prefixing everything with `sudo`:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown "$(id -u):$(id -g)" ~/.kube/config
chmod 600 ~/.kube/config

# Point the kubeconfig at the real server address, not 127.0.0.1
kubectl config set-cluster default \
  --server https://$(hostname -I | awk '{print $1}'):6443

kubectl get nodes
```

That `--server` correction matters as soon as you want to drive the cluster from another machine.

## Adding worker nodes

This is where k3s pulls ahead of single-node alternatives. Grab the node token from the server:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Then on each worker host:

```bash
curl -sfL https://get.k3s.io | \
  K3S_URL=https://<server-ip>:6443 \
  K3S_TOKEN=<node-token> \
  sh -
```

Back on the server:

```bash
kubectl get nodes -o wide
```

You should now see multiple nodes in `Ready` state. Two or three worker VMs is enough to make scheduling, affinity, and anti-affinity meaningful — and to make the `Pending` state something you encounter and have to reason about rather than a theoretical concept.

## Enabling local storage that behaves like the real thing

k3s ships with the **local-path-provisioner**, which creates `PersistentVolumeClaims` backed by directories on whichever node the pod lands on. It is fine for a lab, with one important caveat: **the data is tied to that node**. If the pod reschedules, it does not see the same volume.

To make this limitation explicit rather than surprising, test it:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: local-path
  resources:
    requests:
      storage: 1Gi
```

Create a pod using it, delete the pod, and watch where the replacement lands:

```bash
kubectl apply -f demo-pvc.yaml
kubectl get pvc demo-pvc -o wide   # note the node affinity
```

Encountering this constraint in a lab is exactly the kind of thing that prevents a production surprise later.

## Creating a dedicated kubeconfig context for the lab

If you already have a production kubeconfig, do not mix contexts. Give the lab its own name so that `kubectl config current-context` tells you unambiguously where you are:

```bash
kubectl config rename-context default k3s-lab

# If you rename the cluster too, update it in ~/.kube/config
kubectl config get-contexts
```

This sounds trivial until the day you run a destructive command in the wrong terminal. Set a distinct prompt or use `kubectx` if you switch often.

## Deploying a real workload

A lab is only useful once something runs on it. A minimal deployment with a Service and Ingress exercises the whole path: scheduling, storage-free statelessness, service discovery, ingress routing.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels: { app: whoami }
  template:
    metadata:
      labels: { app: whoami }
    spec:
      # spread replicas across nodes so the lab is actually multi-node
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels: { app: whoami }
      containers:
        - name: whoami
          image: traefik/whoami:v1.10
          ports:
            - containerPort: 80
          resources:
            requests: { cpu: 50m, memory: 32Mi }
            limits: { cpu: 200m, memory: 128Mi }
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  selector: { app: whoami }
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
spec:
  rules:
    - host: whoami.lab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: whoami
                port:
                  number: 80
```

```bash
kubectl apply -f whoami.yaml
kubectl get pods -o wide          # replicas should land on different nodes
kubectl get ingress
```

Add the host to your local DNS or `/etc/hosts` and the request path is closed:

```bash
echo "$(hostname -I | awk '{print $1}') whoami.lab.local" | sudo tee -a /etc/hosts
curl http://whoami.lab.local
```

The `topologySpreadConstraints` block is doing real work here — without it you may get all three replicas on one node and never notice.

## Backing up etcd before you break things

The value of a throwaway lab is that you can destroy it. The value of a *good* lab is that you can restore it too, because backup and restore is a skill that only exists if you practise it.

With k3s's embedded etcd, snapshots are built in:

```bash
# Enable scheduled snapshots (edit /etc/rancher/k3s/config.yaml)
# etcd-snapshot-schedule-cron: "0 */6 * * *"
# etcd-snapshot-retention: 5

# Take one manually
sudo k3s etcd-snapshot save --name pre-upgrade

# List them
sudo k3s etcd-snapshot list
```

Practice the restore path deliberately. Break something real — delete a namespace with workloads in it — and bring it back.

```bash
sudo systemctl stop k3s
sudo k3s server \
  --cluster-reset \
  --cluster-reset-restore-path=/var/lib/rancher/k3s/server/db/snapshots/pre-upgrade
sudo systemctl start k3s
```

Most engineers have never restored a cluster snapshot. It is far better to learn the procedure in a lab than during an actual outage.

## Adding observability to the lab

Once the cluster is healthy, the lab becomes the perfect place to rehearse an observability rollout. This is what I described in my [LGTM stack post](/blog/lgtm-stack-observability/) — and the two posts are designed to be used together.

A sensible order for the lab:

1. **Metrics** — `kube-prometheus-stack` via Helm, giving Prometheus, Alertmanager, and Grafana with Kubernetes dashboards already wired up.
2. **Logs** — Loki in single-binary mode with filesystem storage. Fine for a lab.
3. **Traces** — Tempo plus an OpenTelemetry Collector deployed as a DaemonSet.
4. **Correlation** — datasource links in Grafana between the three.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install kube-prom prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  --set prometheus.prometheusSpec.retention=7d \
  --set grafana.adminPassword=labadmin
```

The lab size is the constraint to respect: run Grafana and Prometheus with modest resource requests and accept that some dashboards will be slow. The goal is to learn the wiring, not to benchmark at scale.

## What this lab teaches that a managed cluster does not

**Scheduling failures.** With real nodes and real resource limits, you will hit `Pending` pods and have to read `kubectl describe pod` events. On a managed cluster with autoscaling you rarely see this.

**Node failure.** Stop one worker VM and watch pods reschedule. Drain it properly and see how PodDisruptionBudgets behave.

**Storage constraints.** Discover firsthand which workloads can survive a volume moving nodes and which cannot.

**etcd operations.** Snapshots, restores, and the fact that the control plane is a thing you are responsible for.

**Networking.** ServiceLB and Traefik in a lab behave like an actual load balancer and ingress, so you learn why an Ingress needs a matching Service and why a NetworkPolicy blocks traffic you expected to pass.

## Pitfalls and gotchas

**Port 6443 and 8472 must be reachable.** Multi-node k3s needs the API port plus the VXLAN port (8472/UDP) for pod networking. Silent firewall blocks here produce nodes that join but never become `Ready`.

**Disable the firewall rules you did not write.** If you are using `firewalld` or `ufw`, add explicit rules for k3s rather than disabling the firewall wholesale — but be aware that `ufw` on Ubuntu can interfere with pod-to-pod traffic if not configured for the CNI.

**local-path volumes do not move.** As covered above, the default storage class pins data to a node. Do not build a lab assumption on top of it that you expect to hold in production.

**Resource requests are not optional in a lab.** Without them, one memory-hungry workload destabilises the whole node and you learn nothing except that the node has no OOM protection.

**Version skew.** k3s tracks upstream Kubernetes closely. Check `k3s --version` and pin your version explicitly in a lab, or an unattended upgrade will change your learning environment underneath you.

## Closing thoughts

A local k3s cluster is the cheapest way to convert Kubernetes knowledge from *reading* into *doing*. Two or three VMs, the install script, a snapshot you have actually restored, and an observability stack you wired up yourself will teach more than any certification lab.

The setup above takes an afternoon. The value is that afterwards you have somewhere to test the risky ideas — the RBAC change, the upgrade, the tail-sampling policy, the restore procedure — before they touch anything that matters.
