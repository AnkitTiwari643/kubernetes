# Kubernetes Mastery: From DevOps Engineer to Senior / Staff
### Teach It Cold. Crack Interviews. Pass CKA. Run Production.
**Author: For Ankit - DevOps Engineer (3 yrs) → Kubernetes Teacher**
**Stack: Docker Desktop + kind + kubectl on Windows | Optional: AWS free-tier EC2 with k3s**
**How this book was built:** Structure follows proven technical ebook practice (O'Reilly / Pragmatic Engineer / Manning *Kubernetes in Action 2e*): objectives → concept → why → how-internals → hands-on → failure modes → interview/staff lens → recap + lab. Explanations synthesize official Kubernetes docs (kubernetes.io/docs), CNCF practices, and 2026 production best-practices (Wolk, Standout hiring rubrics for Senior/Staff).

> **How to use this book:** 1-2 hrs/day. 40 min read + 40 min lab + 20 min teach-back aloud. Don't skip labs. Every chapter ends with "Can you teach it?" If no, re-read. Setup once, use for all chapters.

```powershell
# One-time setup - Windows PowerShell
choco install kubernetes-cli kind -y
kind create cluster --name mastery
kubectl cluster-info
kubectl get nodes -o wide
```

**Book map:**
Part 0 Beginner Runway (Ch0) | Part I Foundations (Ch 1-2) | Part II Workloads (3-5) | Part III Networking (6-8) | Part IV Config/Scale/Schedule/Store (9-11) | Part V Secure/Observe/Deliver (12-14) | Part VI Run Like Staff (15-17) | Part VII Library + Mocks (18-40)

> **First-timer? Start here, skip Senior boxes on first pass.** Each chapter now has `First 5 mins` - analogy + 3 commands + what you should see. Do only that, then come back for depth.


---

## PROLOGUE - Meet ShopKart (Our One Story for the Whole Book)

![FilterKaapi chai shop - where ShopKart started, Coimbatore mornings](https://unsplash.com/photos/gXYwwTQ_tZw/download?w=900)
*Story: clay cups of filter kaapi at dawn - Photo by AJOY DAS on Unsplash (hospitality, chai, Indian culture). Free for commercial use.*

Ankit runs DevOps for FilterKaapi, a Coimbatore chai shop that just went online as **ShopKart**. Diwali sale hits, site crashes at 2am, Ankit SSHes 3 VMs while family sleeps. Priya the dev shouts "just restart Docker!", Arjun the Staff SRE smiles and says "you don't need more SSH, you need a manager who never sleeps."

![ShopKart team - Ankit, Priya and Arjun planning the Diwali sale](https://images.unsplash.com/photo-1522071820081-009f0129c71c?w=900)
*Story: the ShopKart crew around one table - Photo by Brooke Cagle on Unsplash. Free to use.*

That manager is Kubernetes. In this book you will grow ShopKart from 1 laptop to 10 million users, and every new problem introduces a new friend:

- **Priya** ships code fast and breaks things (Deployments, rollouts)
- **Arjun** the mentor who asks "what if this dies?" (HA, PDB, probes)
- **Meena** security, strict akka (RBAC, NetworkPolicy, no latest)
- **Mani Receptionist** (apiserver, only door), **Ellamma Ledger Sisters x3** (etcd, never forget, majority must agree), **Saravanan Room Assigner** (scheduler, picks rooms not cleans them), **Kumar Floor Staff** (kubelet, runs rooms), **Pavithra Phone Operator** (kube-proxy, connects calls, not in call), **Divya Phonebook** (CoreDNS)

Same shop, same `api`, same `web`, same Diwali traffic from Ch0 to Ch40. When you see "ShopKart needs ...", that's your cue: new character, new capability. Read it like a story, run the 3 commands, then teach it to Priya. If Priya gets it, you got it.


---

## PART 0 - BEGINNER RUNWAY (Read This First, No Jargon)

### Chapter 0: 30 Minutes to Ready - Ankit's Night Before Diwali

![Masala chai spices - same recipe, different cups like same image different env](https://unsplash.com/photos/P5wfT_V3OmE/download?w=900)
*Story: same chai, different kulhads - Photo by Nikita Khowala on Unsplash. Like same Docker image, different env.*

Ankit, picture last Diwali: 3 VMs, Priya pushes `v2` at 11pm, one VM disk full, you SSH with one eye open. This chapter makes sure that never scares you again. No jargon, just hands.

**0.1 What you must know (talk like humans):**
- Hey, `docker run nginx` is just a super-isolated process, not a full house (VM). Think apartment vs independent house - same street (kernel), own rooms. Try: `docker ps`, `docker exec -it web sh`, `cat /etc/hostname` - see? Different name from your laptop. That's isolation.
- Port is just a door number. `-p 8080:80` means "knock laptop door 8080, I'll open container door 80." If `curl localhost:8080` fails, stop - nothing ahead will work.
- Same sweet, different box: `-e CITY=Coimbatore` vs `-e CITY=Mumbai` - same image, different taste. That's why we never bake config in image.
- DNS is your phonebook. You never dial 142.250.x.x, you say `google.com`. In ShopKart you'll never dial `10.244.1.3`, you'll say `api`. Divya (CoreDNS) remembers.

**0.2 Your first K8s in 3 commands (do now, grin later):**
```powershell
kind create cluster --name mastery
kubectl run hello --image=nginx
kubectl get pods -o wide # See hello Running + IP 10.244.x, then:
kubectl delete pod hello # Gone forever - Priya shouts "where did it go?!" That's why Ch4 gives us a manager next
```
What just happened? You asked for a room, Kumar gave it an IP, you vacated, no one cleaned or refilled. ShopKart can't run like this on Diwali. Good - you felt the pain. Now you want healing.

**0.3 How to read this book (promise me):** Read ONLY green `First 5 mins` → run 3 commands → tell Priya aloud what happened → tick. Ignore Raft/iptables until second pass, Arjun says "second pass, Ankit." If `kubectl get pods` says Running, you passed today. Hug yourself.

### Chapter 0B: First 5 Mins for Ch1-11 (Do These Before Depth)

**Ch1 First 5:** Analogy: K8s = hotel manager (you say "need 3 clean rooms", manager assigns/fixes, you don't clean). Run: `kubectl create deploy web --image=nginx --replicas=3; kubectl get pods -w; kubectl delete pod -l app=web --grace-period=0` → see new pod auto-creates. Pass if you say "I declared 3, K8s fixed to 3."

**Ch2 First 5:** Analogy: apiserver = reception, etcd = register book, scheduler = room assigner, kubelet = floor staff. Run: `kubectl get pods -n kube-system; kubectl get nodes` → see reception + staff running. Pass if you name 4 parts without notes.

**Ch3 First 5:** Analogy: Pod = hotel room with 2 roommates sharing bathroom (localhost) + door (IP). Run: `kubectl run web --image=nginx; kubectl describe pod web | Select-String IP,Status; kubectl exec -it web -- hostname` → same hostname for containers in pod. Pass if you say "1 Pod = 1 IP, containers share localhost."

**Ch4 First 5:** Analogy: ReplicaSet = manager ensuring 3 rooms occupied. Run: `kubectl create deploy web --image=nginx --replicas=3; kubectl get rs; kubectl delete pod -l app=web --grace-period=0; kubectl get pods` → count stays 3. Pass if you say "bare Pod dies forever, Deployment heals."

**Ch5 First 5:** Analogy: StatefulSet = apartments with nameplates 0,1,2 + own lockers (storage). DaemonSet = one cleaner per floor. Job = one-time laundry. Run: `kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(10)'`; `kubectl get jobs --watch` → Completed. Pass if you say "Job runs once, Deploy runs forever."

**Ch6 First 5:** Analogy: Service = hotel front-desk number (pods change rooms/IPs, desk number stays). Run: `kubectl expose deploy web --port=80 --type=ClusterIP; kubectl get svc web; kubectl get endpointslices -l app=web` → see VIP + 3 pod IPs. Delete pod → endpoints update but VIP same. Pass if you say "VIP stable, pod IPs churn."

**Ch7 First 5:** Analogy: Ingress = reception routing "restaurant→floor2, bar→floor3" by name. Run: `kubectl apply -f https://k8s.io/examples/service/networking/minimal-ingress.yaml; kubectl get ingress` → see host/path → service. Pass if you say "Service L4 IP, Ingress L7 name/path."

**Ch8 First 5:** Analogy: CNI = electrician wiring rooms so they can call each other. Run: `kubectl exec -it deploy/web -- ping <other-pod-IP>` → works across nodes. Pass if you say "CNI gives every Pod IP + route." Ignore Flannel/Calico/Cilium names first pass.

**Ch9 First 5:** Analogy: ConfigMap = notice board (menu changes without rebuilding hotel). Probes = housekeeping knock ("alive? ready for guests?"). Run: `kubectl create cm menu --from-literal=lunch=biryani; kubectl run web --image=nginx --env-from=configmap/menu --dry-run=client -o yaml | grep env` → see injected. Pass if you say "config outside image, probes decide restart vs remove."

**Ch10 First 5:** Analogy: taint = "floor under repair, keep out unless allowed", affinity = "I want sea-view". Run: `kubectl taint nodes kind-control-plane dedicated=true:NoSchedule; kubectl run web --image=nginx` → Pending; `kubectl taint nodes kind-control-plane dedicated-` → Running. Pass if you felt repel/allow.

**Ch11 First 5:** Analogy: container disk = whiteboard wiped on checkout, PVC = locker kept. Run: `kubectl apply -f https://k8s.io/examples/application/mysql/mysql-pv.yaml; kubectl get pv,pvc` → Bound. Pass if you say "Pod dies data dies, PVC survives."

Tick all 11 → second pass Senior depth will make sense. Don't read Raft/iptables yet.

> Closing Ch0: That night Ankit closes the laptop at 11:30pm, first time before midnight in weeks. Priya texts "site still up?" Ankit replies "3 rooms, manager on duty." Tomorrow Arjun shows who the manager really is. Turn to Ch1.

---

## PART I - FOUNDATIONS

![Kubernetes wheel - one platform to steer all ShopKart containers](https://kubernetes.io/images/kubernetes.png)
*Kubernetes logo via kubernetes.io. The wheel Ankit learns to steer.*

> ShopKart story: Ankit meets Arjun at the hotel whiteboard. Keep this wheel in mind every chapter.

### Chapter 1: What Kubernetes Really Is - Arjun's Hotel Lesson

![ShopKart goes online - from street shop to storefront](https://images.unsplash.com/photo-1441986300917-64674bd600d8?w=900)
*Story: FilterKaapi opens its online shutter - Photo by Mike Petrucci on Unsplash (store). Free to use.*

> ShopKart story: Next morning Arjun takes Ankit to a hotel. "You don't clean rooms, Ankit. You tell manager 'keep 3 clean rooms.' He assigns, cleans, refills. You sleep. That's K8s. Priya will delete pods daily - manager will refill before customers notice."

**Objectives:** Define declarative reconciliation. Explain why K8s, not just Docker or VMs. Trace `kubectl apply` end-to-end. Understand cattle vs pets, desired vs actual state, and why every Senior must think in control loops.

#### 1.1 The problem before Kubernetes
As a DevOps engineer you know this pain: you have Docker images, you `docker run` on 3 VMs, you add nginx load balancer, you write systemd units, you SSH to restart crashed containers, you run out of disk, one VM dies at 2am. With 5 services it's manageable. With 50 microservices x 3 envs x 3 replicas = 450 containers, it collapses.

Pre-K8s solutions: Ansible + Docker Compose + Consul + custom scripts. Every company reinvented orchestration badly: no standard for healthchecking, rolling updates, service discovery, bin-packing, self-healing.

Kubernetes (from Google Borg/Omega, donated to CNCF 2014) standardizes this: a single API and control loop for placement, healing, networking, storage, rollout.

What Kubernetes is NOT:
- Not a container runtime (that's containerd). It orchestrates runtimes.
- Not a PaaS (it doesn't build images, doesn't enforce Heroku-style constraints). You build platform on top.
- Not Docker Swarm++ (Swarm is imperative: "run container here". K8s is declarative: "keep 3, I don't care where").
- Not serverless (it manages machines, you still think about nodes, capacity, zones).

#### 1.2 Declarative model: the single idea to master
Imperative: `docker run --restart always nginx` on node A. If node A dies, nobody recreates. You told system WHAT TO DO once.

Declarative: `replicas: 3` in etcd. Controllers observe forever. Node dies → actual=2, desired=3 → create 1 elsewhere. You told system WHAT YOU WANT, it figures HOW.

This is called level-triggered (reconcile continuously) vs edge-triggered (act once on event). Level-triggered survives missed events, crashes, network partitions. If controller restarts, it re-lists and fixes drift. This is why K8s is resilient.

Two states:
- Desired state: objects in etcd (Deployments, Services).
- Actual state: running pods, nodes, network rules.
- Reconciliation loop pseudocode every controller runs:
```
while true:
  desired = get from apiserver/etcd
  actual = list pods/nodes
  diff = desired - actual
  if diff: create/delete/update to close gap
  sleep + watch for changes
```

#### 1.3 Cattle vs Pets in practice
Pet: `prod-db-01` VM, hand-tuned, SSH, snowflake, you nurse back to health. Cattle: `api-7f9c4d6b8-x2kqz`, identical, immutable image `:v1.2.3`, kill any, ReplicaSet makes new. Never SSH to fix pod - replace it. Logs and data must be external (stdout → Loki, data → PVC), because pod is disposable.

Senior rule: if you need to `kubectl exec` to fix prod, your design is already wrong. Exec is for debug, not repair.

#### 1.4 `kubectl apply` end-to-end with minute detail
Example: `kubectl apply -f deploy.yaml` where deploy wants 3 nginx.

1. kubectl loads kubeconfig (cert + server + namespace), builds REST POST to `https://apiserver:6443/apis/apps/v1/namespaces/default/deployments`.
2. apiserver: authenticate (cert/bearer), authorize (RBAC: can you create deployments?), mutate (set defaults: pullPolicy, restartPolicy), validate (schema), admit (LimitRanger injects defaults, PodSecurity checks). Only then writes to etcd `PUT /registry/deployments/default/myapp rev=42`.
3. Deployment controller watches deployments: sees new desired=3, creates ReplicaSet `myapp-abc` with `replicas:3`.
4. ReplicaSet controller sees RS want 3 have 0 → creates 3 Pod objects in etcd with `nodeName: ""` (unscheduled).
5. Scheduler watches unscheduled pods: for each pod Filter+Score nodes, writes Binding `pod/foo → node/worker-2`.
6. kubelet on worker-2 watches assigned pods: CRI pull image, create netns via CNI, mount volumes via CSI, start containers, run probes, report `status.phase=Running` back to apiserver.
7. kube-proxy on all nodes watches Services+EndpointSlices: programs iptables/IPVS to load-balance Service VIP to new pod IPs.
8. CoreDNS watches Services: creates `myapp.default.svc.cluster.local → ClusterIP`.
Forever: if pod crashes, kubelet restarts per restartPolicy; if node dies, Node controller marks NotReady, ReplicaSet creates replacement elsewhere.

Teach-line to memorize: *apiserver is gateway, etcd is truth, scheduler binds, kubelet runs, controllers fix.*

#### 1.5 Why this matters for Senior/Staff
Junior thinks: K8s runs my YAML. Senior thinks: K8s is distributed system with eventual consistency, watches, caches, etcd quorum, API throttling. Incidents come from stale cache, label mismatch, resource starvation, not "K8s bug". Your job: design for failure (PDB, anti-affinity, probes, limits, GitOps), observe burn-rate, control cost.

**Lab 1.1 - See the loop:**
```powershell
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
kubectl get rs,pods -l app=nginx -o wide --watch
kubectl delete pod -l app=nginx --grace-period=0; kubectl get pods -w # watch self-heal
kubectl get events -A --sort-by=.lastTimestamp | Select-Object -Last 20
```

**Can you teach it?** Whiteboard without notes: draw user→apiserver→etcd→scheduler→kubelet→controllers, explain level vs edge, cattle vs pets.

![Kubernetes Cluster Architecture - Control Plane and Nodes](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)
*Source: kubernetes.io/docs/concepts/architecture/ - Control plane manages workers. Used under CC BY 4.0.*

### Chapter 2: Architecture Deep Dive - Every Component, No Magic (20 Pages in One)

**Objectives:** Master stateless vs stateful, etcd + quorum + Raft internals, scheduler Filter/Score with real examples, all controllers, kubelet/CSI/CNI/CoreDNS, static pods, kubeconfig.

#### 2.1 What to call them?
Call them **Kubernetes Components** (binaries/processes). Control-plane components: kube-apiserver, etcd, kube-scheduler, kube-controller-manager, cloud-controller-manager. Node components: kubelet, kube-proxy, container runtime (containerd). Addons: CNI, CSI, CoreDNS, metrics-server. Never say members - member means etcd cluster member or human team member.

#### 2.2 Stateless vs Stateful - with production reasoning
Stateless example in your world: nginx serving static files from S3. Kill pod, new pod serves same. No local disk matters. You can HPA 3→30, LB freely.

Stateful example: Postgres with `/var/lib/postgresql/data` on EBS. Pod name matters (primary vs replica), data matters. Kill without volume → data loss. Need Stable identity + per-pod PVC + ordered rollout. That's why StatefulSet exists.

Decision matrix Senior uses:
- Need scale to zero + burst? → stateless.
- Need exactly-once + ordering + sticky data? → stateful (and ask: should this even be in K8s? Managed RDS often better than Postgres in K8s unless you need operator expertise).
- Can you externalize state? Best pattern: app stateless in K8s, state in managed service (RDS, S3, ElastiCache). Only run stateful in K8s when latency/cost/operator demands it (Kafka, Cassandra with local NVMe).

Why apiserver stateless? All state in etcd (WAL + snapshots on disk). Apiserver keeps only in-memory caches (informers) rebuildable on restart. Run 3 replicas behind LB (kubeadm: haproxy, EKS: AWS NLB). Rolling restart apiserver → zero impact. If apiserver stored data locally, you'd need consensus, volume replication, split-brain handling - pointless duplication of etcd.

Contrast: etcd IS stateful: each member has `data-dir=/var/lib/etcd` with BoltDB + WAL. You must backup (`etcdctl snapshot save`), monitor disk latency (`wal_fsync_duration_seconds` >10ms = danger), keep 3/5 members, never run 2 (no quorum gain, double failure domain) or 6 (quorum 4, tolerates only 2 like 5 but more cost).

#### 2.3 etcd - The Truth, Raft, Quorum, What Breaks
Purpose: consistent, durable store for all objects + resourceVersions for watches + leases for heartbeats.

What is distributed? 3 processes on 3 hosts acting as 1 logical KV via consensus. Client (apiserver) talks to any member, linearizable reads/writes.

When to distribute anything? When single node MTTR unacceptable + you can pay complexity (consensus, clock, network). K8s needs API available during AZ failure → distribute.

Why K8s team chose etcd (Raft) over ZooKeeper/MySQL? Simple HTTP/gRPC API, watch primitives, TTL leases, proven in CoreOS, easy TLS, small binary. Tradeoff: very sensitive to disk latency and network RTT - never put etcd on network disk (EBS st1) or cross-region.

What stored? Run `ETCDCTL_API=3 etcdctl get /registry --prefix --keys-only | head`: `/registry/deployments/default/web`, `/registry/pods/...`, `/registry/services/...`, `/registry/secrets/...`, `/registry/rbac/...`, `/registry/apiextensions/...` (CRDs). Secrets base64 (optionally encrypted with KMS Envelope: DEK per secret, KEK in KMS). Size limit 1.5MB per object (request limit) - don't stuff large files in ConfigMap.

Quorum math: n=1→1 (no HA), n=3→2 (tolerate 1), n=5→3 (tolerate 2). Formula `floor(n/2)+1`. Even numbers bad: n=4 quorum 3 tolerates 1 same as 3 but costs more. So always odd.

How Raft works step-by-step (teach this):
1. All start as Follower. Election timeout random 150-300ms. First timeout → Candidate, increment term, vote for self, ask others.
2. Majority grants (including log up-to-date check) → Leader for that term. Leader sends heartbeats (AppendEntries empty) every 100ms to assert.
3. Write: apiserver `PUT /registry/pods/x` → Leader appends to WAL (fsync), replicates AppendEntries to followers, waits majority ack, commits (applies to BoltDB, advances commitIndex), responds OK, piggybacks commit to followers.
4. Read linearizable: Leader confirms still leader (heartbeat) then serves from DB.
5. Partition 2-1: side with 2 elects leader, keeps writing. Side with 1 can't get majority, stops, becomes stale. When healed, stale leader steps down (higher term seen), replays logs.
6. Snapshot + compaction: WAL grows, periodic snapshot + revision compaction (default 5min, keep 1000 revs). If apiserver watch too old (`resourceVersion too old`), it relists - normal.

Failure drill Senior must know: disk slow → leader election flapping → apiserver 500s → `kubectl` timeouts but pods keep running. Fix: move etcd to io2/gp3, defrag, remove learner, restore snapshot `etcdctl snapshot restore --data-dir`.

#### 2.4 kube-scheduler - Binding, Filtering, Scoring with examples
Watches `pods with .spec.nodeName=="" && .spec.schedulerName==default-scheduler`. Scheduling cycle: queue (priority+backoff) → filter → score → reserve → permit → preBind (volume) → bind (write Binding object via apiserver, kubelet picks up).

Binding API: `POST /api/v1/namespaces/default/pods/mypod/binding {target: {name: node2}}`. This sets nodeName atomically. Scheduler never SSHs, never starts containers.

Filter plugins (removes bad):
- NodeResourcesFit: `requests.cpu 500m > allocatable 200m?` → Unschedulable.
- TaintToleration: node `gpu=true:NoSchedule`, pod has no toleration → filtered.
- NodeAffinity: pod `requiredDuringScheduling: zone=a`, node `zone=b` → out.
- PodTopologySpread, VolumeRestrictions (EBS zone), NodePorts (port already used), Affinity conflict.

Score plugins (rank 0-100, weighted sum):
- LeastAllocated (prefer empty), MostAllocated (binpack - for cost), ImageLocality (already has image → faster), InterPodAffinity (co-locate), NodeAffinity preferred.

Example: pod requests 1 CPU, 3 nodes: n1 90% used, n2 20% used + has nginx image, n3 has taint. Filter removes n3, scores n2 highest (least-loaded + image), binds n2.

Ranks nodes per pod, not pods. Priority queue orders pods (system-critical first). Preemption: high-priority pod can evict low (with PDB respected).

#### 2.5 kube-controller-manager - All loops you must name in interview
One binary running ~35 controllers, each `informer (list+watch cache) → workqueue → reconcile`. Leader election among 3 replicas (only one active via Lease lock).

Must-know 12:
1. Deployment → creates/updates ReplicaSets for rollout.
2. ReplicaSet → ensures N pods (selector vs template).
3. StatefulSet, DaemonSet, Job, CronJob (time → Job).
4. Node lifecycle: 40s no heartbeat → Unknown, 5min → evict pods (taint NoExecute).
5. Namespace: deletes all namespaced objects on delete, stuck Terminating? Check finalizers.
6. ServiceAccount: creates token Secret, mounts.
7. Endpoints/EndpointSlice: fills pod IPs for Services (mirror of selector match).
8. PV binder + provisioner: binds PVC→PV or calls CSI CreateVolume.
9. TTL, GC (orphan pods, completed Jobs), HPA (separate controller-manager but same idea).
Custom: disable via `--controllers=-ttl` for debug; write Operator with controller-runtime (CRD + loop) for DB, certs.

#### 2.6 kubelet, runtime, kube-proxy, CNI/CSI/CoreDNS, static pods
Kubelet: `--kubeconfig` + `--config` (hairpin, evictionHard `memory.available<500Mi`), registers Node (`POST /nodes`), syncLoop (apiserver pods + file static pods + HTTP), CRI `RunPodSandbox` (pause) → CNI ADD → CSI NodeStage/Publish → start init then app → probes → status Patch. Heartbeats NodeLease every 10s.

Runtime: containerd (CRI plugin) pulls from registry, unpacks overlayfs, runs runc. `crictl ps/images/pull` debug, not `docker`.

kube-proxy modes: userspace (ancient), iptables (default kind), IPVS (prod large), nftables (new), eBPF (Cilium kube-proxy replacement). iptables chain: `KUBE-SERVICES → KUBE-SVC-XXX (LB) → KUBE-SEP-YYY (DNAT to pod)`. `iptables-save | grep KUBE`, `ipvsadm -Ln`.

Static pods deep: files `/etc/kubernetes/manifests/kube-apiserver.yaml` with `ownerReference` none. Kubelet creates mirror pods `kube-apiserver-master+node` in apiserver for visibility. Delete file → pod dies. Edit file → kubelet restarts. Self-bootstrap: kubeadm writes these files, kubelet (systemd) starts control-plane, then control-plane manages rest. Other types: regular (etcd), static (file), mirror (read-only).

kubeconfig: clusters + users (cert/token/exec) + contexts. `kubectl config use-context`, `--as=system:serviceaccount:dev:bot` impersonate for RBAC test.

**Lab 2.1:**
```powershell
kubectl get pods -n kube-system -o wide
kubectl logs -n kube-system -l component=kube-scheduler --tail=50
kubectl get leases -n kube-system # leader election
kubectl get nodes -o yaml | Select-String taint -Context 2
docker exec -it mastery-control-plane crictl ps | Select-Object -First 10
```


---

## PART II - WORKLOADS: FROM POD TO PRODUCTION

> ShopKart story, late night shift week 2: Priya pushes at 1am, deletes the wrong pod, Ankit watches it vanish. No sleep, same bug twice. Arjun says "stop babysitting pods, give them a manager." This Part is that manager.

![Late night laptop - Ankit's 1am shift before the manager arrives](https://images.unsplash.com/photo-1504384308090-c894fdcc538d?w=900)
*Story: one lamp, one laptop, same bug twice. Photo via Unsplash. Free to use.*

### Chapter 3: Pods in Minute Detail - The Atomic Unit (Senior Depth)

**Objectives:** Why Pod not container, pause infra + Linux namespaces, full lifecycle + phases + conditions, initContainers vs ephemeral, probes interaction, QoS + eviction, multi-container patterns with YAML, static vs regular vs mirror, debugging like Staff.

#### 3.1 Why Pod, not container?
Docker taught one container = one app. K8s adds Pod because real apps need co-located helpers sharing fate: app + log shipper must live/die/move together, share localhost and volumes, same scheduler decision. If they were separate pods, scheduler could place them on different nodes, breaking localhost assumption, doubling Service entries.

Pod guarantees: same node, same netns (one IP), shared IPC/UTS optionally, shared volumes, single lifecycle (Pending together, killed together). Containers in pod communicate via `localhost:port` (no Service needed) and `shared emptyDir`.

#### 3.2 Pause container + Linux namespaces minute detail
When kubelet creates Pod, CRI first creates `pause:3.9` (`/pause` sleeping). It calls `unshare(CLONE_NEWNET|CLONE_NEWIPC|CLONE_NEWUTS)` to create fresh namespaces, then pins them via `/proc/<pid>/ns/net`. All app containers then `setns()` into those namespaces (`--net=container:pause`, `--ipc=container:pause`).

Check hidden: `kubectl` hides pause, but `crictl ps` shows `POD` sandbox + app containers. `lsns -p <pause-pid>`, `ip netns` won't show (kubelet uses its own), use `nsenter -t <pause-pid> -n ip addr` to see pod IP. Pod UID decides sandbox name, you cannot/should not rename - CNI and kubelet track by UID.

If pause dies, all containers die (sandbox lost) → kubelet recreates whole pod sandbox. That's why pause is tiny, never crashes.

Namespaces shared: net (IP+ports - so two containers can't both listen 8080), IPC (shm), UTS (hostname). PID optionally shared (`shareProcessNamespace: true` for debugging sidecar seeing app processes, used by Istio).

#### 3.3 Full lifecycle, phases, conditions
`spec.containers[*]` (required ≥1) + `spec.initContainers[*]` (optional, run first sequentially, each must exit 0) + `spec.ephemeralContainers[*]` (debug attach without restart, `kubectl debug -it pod --image=busybox --target=app`).

Phases (`status.phase`): Pending (scheduled? pulling? init running?), Running (all started, not necessarily Ready), Succeeded (all completed 0, for Jobs), Failed (≥1 non-zero and restartPolicy Never), Unknown (node lost, 40s no heartbeat → controller can't know).

Conditions (`status.conditions`): PodScheduled, Initialized, ContainersReady, Ready (Ready=True only if all readinessProbes pass + minReadySeconds). `kubectl wait --for=condition=Ready pod/web`.

RestartPolicy: Always (Deployments, must), OnFailure (Jobs retry), Never (Jobs run once). With Always, kubelet restarts container in same pod (same IP), backoff 10s→20s→...→5min. CrashLoopBackOff = repeated crash + backoff, check `kubectl logs --previous`, `kubectl describe` LastState Terminated exit 137 OOM, 1 app error, 139 segfault.

Termination: `SIGTERM → gracePeriod (30s default) → SIGKILL`. App must trap SIGTERM + `preStop: {exec: {command: [sleep,5]}}` to drain. Long grace for Java (60s).

#### 3.4 Bare pods, static, mirror, ephemeral
Bare pod (`kind: Pod` directly): no controller, no self-heal, no rollout. Node dies → gone. Never prod, only debug/learning. Prod always Deployment/StatefulSet.

Static pods: file on node, no apiserver. Mirror: apiserver read-only copy `podname-nodename` with `ownerReference` + annotation `kubernetes.io/created-by`. Delete via API fails - delete file.

#### 3.5 Patterns with YAML (Staff must whiteboard)
Sidecar: `app + fluent-bit` tailing `emptyDir` log volume, or `app + cloud-sql-proxy`.
Ambassador: `app → localhost:5432 (ambassador pgbouncer) → RDS`. App config never changes when DB moves.
Adapter: `app emits /metrics custom → adapter converts to Prometheus` scraping same localhost.
Init example: `wait-db + migrate + chmod` before app. Limit: no hard API limit, practical 1-3 (each adds startup time, sequential). Cannot have only inits - API requires `containers ≥1` validation.

QoS (for eviction Ch9 preview): Guaranteed (all containers req==lim) → never evicted unless system OOM; Burstable (some req); BestEffort (none) → first killed on NodePressure. `kubectl get pod -o jsonpath='{.status.qosClass}'`.

**Lab 3 - Break and teach:**
```powershell
kubectl run web --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml; kubectl describe pod web
kubectl exec -it web -- cat /etc/hostname; kubectl exec -it web -- env | Sort-Object
# two containers share localhost:
kubectl apply -f https://k8s.io/examples/pods/two-container-pod.yaml
kubectl exec -it two-containers -c 1st -- hostname; kubectl exec -it two-containers -c 2nd -- hostname # same!
kubectl debug -it web --image=busybox --target=web # ephemeral
```
Teach: draw pause + 2 containers + netns + emptyDir, explain ports conflict, Unknown vs Pending.

### Chapter 4: ReplicaSet & Deployment - Self-Healing + Zero-Downtime Rollouts

**Objectives:** Selector vs template pitfalls, why never ReplicaSet directly, RollingUpdate math (maxSurge/maxUnavailable), revision/history/rollback, readiness gates, Staff rollout design.

#### 4.1 ReplicaSet minute detail
ReplicaSet ensures `status.replicas == spec.replicas` with matching labels. Two fields confused:
- `selector.matchLabels: {app: web}` - which EXISTING pods count? Uses lister from cache.
- `template.metadata.labels: {app: web}` - what label to put on NEW pods it creates?
If mismatch (selector app=web, template app=web-v2), RS creates pods but never counts them → creates infinitely → 100s pods. `kubectl describe rs` shows `0/3` forever. Fix: delete RS with `--cascade=orphan` or fix labels.

Why never use directly? RS can't version, can't rollout (no strategy), can't rollback (no history), can't pause. Deployment adds all that by managing RSs. In prod `kind: ReplicaSet` YAML = code smell.

#### 4.2 Deployment internals
Deployment controller creates RS with `pod-template-hash` label (e.g. `web-7f9c4d6b8`). New image → new RS (v2) with 0 pods, scales v2 up + v1 down per strategy, keeps old RSs per `revisionHistoryLimit:10` for rollback. `kubectl rollout history deploy/web`, `kubectl rollout undo deploy/web --to-revision=2`.

Fields Staff sets: `replicas:3` (never 1-2 for prod - quorum/survive), `strategy.rollingUpdate {maxSurge, maxUnavailable}`, `minReadySeconds:10` (wait before considering Available), `progressDeadlineSeconds:600` (mark Failed if stuck), `revisionHistoryLimit`.

#### 4.3 maxSurge / maxUnavailable - the math that decides downtime
- Surge = extra above desired allowed. `replicas:10, maxSurge:2` → up to 12 temp (needs capacity). Surge pods scheduled first, must pass readiness before old killed (when maxUnavailable 0).
- maxUnavailable = how many desired can be DOWN (Updating + Unavailable). `replicas:10, maxUnavailable:0` → always 10 Available, zero downtime but needs extra nodes. `maxUnavailable:1` → 9 Available min, faster, brief risk.
- Percentages: `25%` of 4 =1. Ideal prod critical (payments): `maxSurge:1, maxUnavailable:0` + `readinessProbe + preStop + PDB`. Ideal dev: `maxSurge:25%, maxUnavailable:25%` for speed.
- Judge on sight: `replicas:2 + maxUnavailable:1` = 50% down = BAD (one pod update fails → 0 Available). No probes = traffic to cold JVM → 502s even with perfect strategy. Good = `≥3 replicas, maxUnavailable 0, readiness + minReady 10s, PDB minAvailable 2`.

Rollout sequence example `replicas:3, surge1 unavail0, image v1→v2`: RS v2 0→1 (new pod Pending→Ready), RS v1 3→2, RS v2 1→2, RS v1 2→1, RS v2 2→3, RS v1 1→0. `kubectl rollout status`, `kubectl get rs -w`. Failed new pod (CrashLoop) → progressDeadline → freeze, old still serves → `undo`.

**Lab 4:**
```powershell
kubectl create deploy web --image=nginx:1.25 --replicas=3 --dry-run=client -o yaml > dep.yaml
kubectl apply -f dep.yaml; kubectl get rs,pods -l app=web -w
kubectl set image deploy/web nginx=nginx:1.26 --record; kubectl rollout status deploy/web; kubectl rollout history deploy/web
kubectl rollout undo deploy/web; kubectl describe deploy web | Select-String -Pattern Strategy,Replicas
```

### Chapter 5: StatefulSet, DaemonSet, Job/CronJob - Identity, Coverage, Completion

**Objectives:** When stable identity/storage matters, headless DNS, volumeClaimTemplates, DaemonSet tolerations, Job parallelism/completion, CronJob schedule/history.

#### 5.1 StatefulSet - for DB/Kafka/etcd/ZooKeeper
Is stable hostname+storage that important? For nginx no (any pod serves). For Postgres primary/replica, Mongo replica set, Kafka broker id, etcd member - YES. They discover peers by hostname (`mongo-0.mongo:27017`), replicate by ID, and must reattach SAME data after reschedule. Random Deployment name (`web-75f4-abc`) + shared PVC breaks consensus and loses data.

Only difference vs Deployment? No - 5 differences:
1. Ordinal stable name `web-0,1,2` (not hash), stable hostname via headless Service.
2. Stable storage: `volumeClaimTemplates` creates `data-web-0, data-web-1...` each bound to own PV. Reschedule web-1 → same PVC reattaches (if AZ allows).
3. Ordered deploy/scale: 0→1→2 start, reverse teardown. `podManagementPolicy: OrderedReady` (safe) vs Parallel (fast).
4. Ordered updates: RollingUpdate with `partition` (canary one replica), `OnDelete` manual.
5. Headless Service required (`clusterIP: None`) for direct pod DNS `web-0.db.default.svc.cluster.local → PodIP`, plus SRV records.

Headless Service = no VIP, no kube-proxy LB. DNS returns A records for all Ready pods. Client load-balances or picks leader. Regular ClusterIP returns single VIP.

volumeClaimTemplate vs PVC: PVC is one claim ticket. Template is factory: StatefulSet controller creates PVC per replica automatically. Deployment can't - you'd manually pre-create or share one (wrong for DB).

When NOT to use StatefulSet? When managed service better (RDS over Postgres in K8s unless you run operator like CloudNativePG + need low latency). Staff decision: data gravity + backup + restore tested?

#### 5.2 DaemonSet - one per node, why?
Node-level duties need coverage, not count: log agent (Fluent Bit), metrics exporter (node-exporter), CNI, CSI, security agent. Want exactly 1 per eligible node, not 3 total.

Tolerates control-plane taints: masters have `node-role.kubernetes.io/control-plane:NoSchedule` to keep user workloads off. But monitoring/CNI must run there too, so DaemonSet adds `toleration: {key: node-role..., operator: Exists, effect: NoSchedule}`. Without it, masters blind.

New node = auto gets pod: DaemonSet controller watches Node objects. Node Ready + matches nodeSelector/affinity → creates pod. Node cordoned? Still runs (unless update). Node deleted → GC. Update via RollingUpdate (maxUnavailable 1 node at a time) or OnDelete.

#### 5.3 Job/CronJob - run to completion
Where runs? On workers like any pod. Flow: CronJob controller (control-plane ticker, every 10s checks cron) → creates Job object → Job controller creates pods with `restartPolicy Never/OnFailure`.

Fields that matter:
- Job: `completions:5` (need 5 successes), `parallelism:2` (2 at once), `backoffLimit:4` (retry 4 then Failed), `activeDeadlineSeconds:300` (kill if too long), `ttlSecondsAfterFinished:3600` (auto-cleanup).
- CronJob: `schedule:"*/5 * * * *"` (cron), `jobTemplate` (Job spec), `concurrencyPolicy: Allow (overlap) / Forbid (skip if prior running) / Replace (kill prior)`, `startingDeadlineSeconds:100` (missed start tolerance), `successfulJobsHistoryLimit:3, failedJobsHistoryLimit:1` (keep last N Job objects for logs, prevent etcd bloat - cron every minute without limits → 43k Jobs/month → slow apiserver), `suspend:true` (pause).

Why restartPolicy Never/OnFailure, never Always? Job counts pod success. Always would restart container inside same pod forever, Job never completes. Never = new pod per retry (clean logs per attempt). OnFailure = restart container (faster, same pod IP).

History limits = etcd hygiene + debuggability. Keep 3/1, not 100.

**Lab 5:**
```powershell
kubectl apply -f https://k8s.io/examples/application/web/web.yaml # StatefulSet demo
kubectl get sts,pvc,pods -l app=web
kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)' --dry-run=client -o yaml
kubectl create cronjob hello --image=busybox --schedule="*/1 * * * *" -- echo hello --dry-run=client -o yaml
```


---

## PART III - NETWORKING

> ShopKart story, first interview failure: Ankit says "Service keeps pods running." Interviewer smiles, rejects. That night Ankit learns Pavithra (kube-proxy) never runs pods, she only connects calls, and Divya (DNS) is the phonebook. Next interview will be different.

![Developers pairing - Ankit replays the failed interview with Priya](https://images.unsplash.com/photo-1551434678-e076c223a692?w=900)
*Story: two engineers, one whiteboard, replaying the reject. Photo via Unsplash. Free to use.*

### Chapter 6: Services - Stable Front for Ephemeral Pods (Deep)

**Objectives:** All 4 types + headless, EndpointSlice, kube-proxy iptables/IPVS chains, debug empty endpoints.

Pod IPs (10.244.x) churn on every restart. Service gives stable VIP (10.96.x) + DNS + LB. Created as `ClusterIP` object in etcd, watched by kube-proxy (programs nodes) + CoreDNS (programs DNS) + EndpointSlice controller (fills pod IPs).

Types with when:
- ClusterIP (default, internal): `my-svc:80 → 10.244.1.3:8080, 10.244.2.5:8080`. IP from `--service-cluster-ip-range 10.96.0.0/12`. You never pick except `None` for headless. Staff: set `internalTrafficPolicy: Local` to keep traffic on node (saves cross-AZ cost, but risks imbalance).
- NodePort (30000-32767 on EVERY node): `NodeIP:30080 → ClusterIP → pods`. Opens hostPort via kube-proxy. Dev/bare-metal only, never prod (port management hell, no LB).
- LoadBalancer: provisions cloud LB (AWS ELB $20/mo + hourly). `healthCheckNodePort` + `externalTrafficPolicy: Local` preserves client IP (else SNAT hides). On EKS use AWS LB Controller for ALB (L7) vs classic.
- ExternalName: pure DNS CNAME `db → rds.amazonaws.com:5432`, no proxy, no VIP. For migrating outside K8s.
- Headless (`clusterIP: None`): no VIP, DNS A records per pod. For StatefulSet + direct connect.

Internals packet path (teach): `curl my-svc → CoreDNS 10.96.0.10 → 10.96.5.5 → iptables KUBE-SERVICES (match dst) → KUBE-SVC-XXX (statistic 50/50) → KUBE-SEP-AAA (DNAT --to 10.244.1.3) → CNI veth → pod`. Return via conntrack un-DNAT. IPVS replaces chains with `ipvsadm` virtual server `10.96.5.5:80 → rr → real servers`.

iptables vs IPVS minute: iptables linear, 10k rules = 2s sync, high CPU. IPVS hash O(1), 10k services fine, supports `rr, lc, dh, sh, sed`. Enable `--proxy-mode=ipvs --ipvs-scheduler=rr`. New: nftables (faster), eBPF (Cilium replaces kube-proxy entirely, no iptables).

Debug #1 outage: `kubectl get endpointslices -l kubernetes.io/service-name=my-svc` empty → selector mismatch (`app: web` vs `app: web-v2`), or pods not Ready (readiness failing → removed), or namespace wrong. Fix labels, check `kubectl get pods -o wide`, `kubectl describe svc`.

![Service networking overview](https://kubernetes.io/images/docs/services-overview.svg)
*Source: kubernetes.io/docs/concepts/services-networking/service/*

**Lab 6:** expose, break selector, fix, switch to IPVS, `iptables-save | grep KUBE-SVC | head`.

### Chapter 7: Ingress, Gateway API, DNS - L7 Routing Done Right

Service L4 (TCP). Ingress L7 (HTTP host/path, TLS, rewrite, rate-limit). Ingress object alone does nothing - need controller Deployment (ingress-nginx, traefik, AWS ALB, Cilium) watching Ingress + creating reverse-proxy config + LoadBalancer Service.

Key fields: `ingressClassName: nginx` (which controller?), `tls.secretName` (cert), `rules.host+http.paths[path, pathType Prefix/Exact/ImplementationSpecific, backend.service]`. Annotations per controller (`nginx.ingress.kubernetes.io/canary-weight: 10` for canary, `alb.ingress.../scheme: internet-facing`).

TLS: create `kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key`, controller terminates TLS then HTTP to ClusterIP. Auto-cert with cert-manager (ACME → Let's Encrypt, renews, stores in Secret).

Limits of Ingress (why Gateway API?): single object mixes infra (LB, IP) + app routing, annotation hell, no role split, TCP/UDP/gRPC poor. Gateway API (GA 2023): `GatewayClass → Gateway (infra, port 443 + LB) → HTTPRoute/GRPCRoute/TLSRoute (dev, host/path → Service)` + `ReferenceGrant` cross-ns. Role split platform vs dev, better canary (weight), header match, filters.

CoreDNS deep: Deployment `coredns` in kube-system (2 replicas + PDB + HPA), ConfigMap Corefile (`.:53 {errors, health, kubernetes cluster.local, forward . /etc/resolv.conf, cache 30}`). Pod `/etc/resolv.conf` has `nameserver 10.96.0.10 search default.svc.cluster.local svc.cluster.local cluster.local ndots:5`. ndots:5 means any name with <5 dots tries all search domains first: `curl my-svc` → tries `my-svc.default.svc...` (hit) but `curl google.com` (2 dots) tries 4 failing local queries first → 100ms extra + CoreDNS load + 5 NXDOMAIN. Fix: use FQDN `google.com.` with trailing dot, or `ndots:2` for high-churn apps, or NodeLocal DNSCache (DaemonSet caching on node).

### Chapter 8: CNI Compared - Flannel vs Calico vs Cilium (Choose Like Staff)

CNI spec: runtime calls `/opt/cni/bin/plugin ADD <netns> <podUID> → assign IP from IPAM, create veth pair, move one end to pod netns, set routes, return IP`. DEL on delete. Config in `/etc/cni/net.d/10-calico.conflist`. IPAM: host-local (kind), calico-ipam (pools), AWS VPC (real ENI IPs).

- Flannel (simple overlay): VXLAN tunnel (UDP 8472) encapsulates pod→pod across nodes, etcd stores subnet leases (`10.244.1.0/24 → node1`). No NetworkPolicy, no BGP, ~15% overhead, single binary. Good learning/kind/edge, terrible prod secure.
- Calico (secure prod default): BGP (no encapsulation in same L2, routes pod CIDRs as /26 per node), Felix programs iptables per pod for NetworkPolicy, IPPools (`10.244.0.0/16 blockSize 26`), eBPF mode optional. Supports `GlobalNetworkPolicy`, wireguard encryption. Pick when you need compliance + simple.
- Cilium (modern scale/observe): eBPF programs in kernel (no iptables, socket LB), identity-based policy (not IP: `app=frontend can talk to app=db on 5432` even as IPs churn), DNS-aware (`allow api.google.com`), L7 (HTTP method), Hubble observability (flow map which pod→which), ClusterMesh multi-cluster, ServiceMesh without sidecars ( Envoy eBPF), kube-proxy replacement. Needs kernel ≥5.10, more CPU for Hubble. Pick for microservices/AI/scale + need visibility.

Staff choice matrix: small secure → Calico; large 500+ svcs + need L7 + trace → Cilium; learning → Flannel/kindnet. EKS: VPC CNI (native ENI, no overlay, limited IPs) + Calico policy, or Cilium chaining.

**Lab 8:** `kubectl get pods -n kube-system -l k8s-app=calico-node`, `calicoctl get ippool`, `cilium status`, `kubectl apply -f NetworkPolicy default-deny` test ping block.


---

## PART IV - CONFIG, SCHEDULE, STORE, RUN

> ShopKart story, grind month: Ankit studies 1 hour after shift, fails a mock, fixes probes at 11pm while Priya sleeps. Boring nights compound. This Part is the unsexy middle that cracks interviews.

![Coffee and notes - the grind month that cracks interviews](https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?w=900)
*Story: filter coffee, handwritten probes, 11pm fixes. Photo via Unsplash. Free to use.*

### Chapter 9: ConfigMap, Secret, Probes, Resources - App Health + Rightsizing

**Objectives:** Config vs image, Secret encryption + ESO, probe types + tuning that prevents restart loops, requests/limits + QoS + OOM.

ConfigMap (plain) vs Secret (base64, type Opaque/kubernetes.io/tls/dockerconfigjson). Mount as `envFrom` (12-factor, but needs restart on change) or `volume` (kubelet syncs every ~1min, app can reload without restart). Never bake env in image - same image `v1.2.3` runs dev/stage/prod with different Config.

Secret truth: base64 ≠ encryption. etcd stores plaintext unless `EncryptionConfiguration` with KMS (AWS KMS, `aescbc` local). Prod Staff: enable `encrypt Secrets at rest + ESO (External Secrets Operator syncs AWS Secrets Manager → K8s Secret) + no Secrets in Git + RBAC deny get secrets + audit`. `kubectl get secret db -o jsonpath='{.data.password}' | base64 -d`.

Probes deep (most misconfigured):
- liveness `httpGet /healthz`: kubelet restarts container on failure. Wrong path/initialDelay too low → restart loop even though app would start in 40s. Java needs `startupProbe` gating.
- readiness `httpGet /ready`: removes pod from EndpointSlice, no restart. Must check DB/dependency, not just process. No readiness → Service sends to cold pod → 502.
- startup `tcpSocket 8080`: disables liveness/readiness until success, for slow start (60s). After success, liveness takes over.
Tuning: `initialDelaySeconds:5 (or 0 with startup), periodSeconds:10, timeoutSeconds:2, successThreshold:1, failureThreshold:3`. Exec probe costs fork, prefer http/tcp. gRPC probe `grpc.health.v1`.

Resources + QoS + eviction:
`requests` = scheduler filter + kube-reserved (what node guarantees). `limits` = kernel cgroup max (OOM kill or CPU throttle). Example `requests {cpu:200m mem:256Mi} limits {cpu:500m mem:512Mi}`: scheduler needs 200m free, app bursts to 500m throttled, over 512Mi → OOMKilled exit 137 `dmesg`.
QoS: Guaranteed (all containers req==lim) → last evicted; Burstable (some set) → middle; BestEffort (none) → first on `NodePressure Disk/Memory`. `kubectl top pods` (metrics-server) → right-size: requests=p95 usage, limits=p99+headroom. No limits in prod = noisy neighbor starves node. No requests = scheduler overpacks → Pending/pressure.

**Lab 9:** break liveness (wrong port → CrashLoop), fix with startup, `kubectl top`, OOM demo `stress --vm 1 --vm-bytes 600M` with 512Mi limit.

### Chapter 10: Scheduling Masterclass - Place Pods Like Staff

Beyond Filter/Score you run daily:
- `nodeSelector: {disktype: ssd}` simple. `nodeAffinity requiredDuringSchedulingIgnoredDuringExecution: {nodeSelectorTerms: [{matchExpressions: [{key: zone, operator: In, values: [a]}]}]}` hard, `preferred... weight:100` soft.
- `podAffinity: {required: co-locate app+cache same hostname}` for locality (reduces latency). `podAntiAffinity: {preferred hostname + required zone}` - never 2 replicas same node/zone. Staff default for HA: `antiAffinity hostname required + topologySpread zone`.
- `topologySpreadConstraints: {maxSkew:1, topologyKey: topology.kubernetes.io/zone, whenUnsatisfiable: DoNotSchedule}` - even spread 3 replicas 1-1-1 across 3 AZs, not 3-0-0.
- `taints/tolerations`: `kubectl taint nodes gpu1 nvidia.com/gpu=true:NoSchedule` repels all; only pods with `tolerations: [{key: nvidia.com/gpu, operator: Exists, effect: NoSchedule}]` + `nodeAffinity gpu` land. Effects: NoSchedule (new pods), PreferNoSchedule (soft), NoExecute (evict existing + `tolerationSeconds:300` for drain grace).
- `nodeName/hostNetwork/hostPID` escape hatches (never prod except CNI), `schedulerName: custom` for batch.
- `priorityClassName: high-priority (1000000)` + preemption: critical pod evicts low (respects PDB). `cordon` (no new), `drain --ignore-daemonsets --delete-emptydir-data` (evict + respect PDB + delete local).
- Quota: `ResourceQuota {pods:50, requests.cpu:20}` per ns cap, `LimitRange {default request 100m limit 500m}` injects defaults to prevent BestEffort.

Staff template: 3+ replicas + anti-affinity host+zone + spread + PDB + requests set = survives node + zone loss.

### Chapter 11: Storage - PV/PVC/StorageClass/CSI + Stateful Data

Containers ephemeral (overlay upperdir lost on delete). Need persistent: PV (admin/cluster disk, e.g. `aws-ebs-vol-123 100Gi gp3 zone-a`), PVC (dev claim `need 10Gi RWO`), StorageClass (template `provisioner: ebs.csi.aws.com parameters: {type: gp3, encrypted: true} allowVolumeExpansion:true`), CSI driver (external controller + node plugins implementing `Create/Delete/Stage/Publish`).

AccessModes: RWO (1 node rw, EBS), ROX (many read, EFS), RWX (many rw, EFS/NFS, needed for shared WordPress). EBS can't RWX - pick EFS. `volumeMode: Filesystem vs Block` (DB likes Block).

Lifecycle: PVC Pending → provisioner calls CSI Create → PV Bound → kubelet CSI NodePublish (`/var/lib/kubelet/pods/<uid>/volumes/...` bind to `/data`) → Mounted. `reclaimPolicy: Retain` (PV kept + Released, manual wipe - prod DB) vs Delete (auto delete EBS on PVC delete - dev). `allowVolumeExpansion` + `kubectl patch pvc data --patch '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'` online grow (needs fs resize).

volumeClaimTemplates (STS): `metadata.name: data` → creates `data-web-0,1,2` each 10Gi. Delete STS without `--cascade=orphan`? PVCs remain (protect data) - manual cleanup `kubectl delete pvc -l app=web`.

CSI vs in-tree: old `kubernetes.io/aws-ebs` built-in, now CSI external (faster release, no K8s upgrade for driver fix). Check `kubectl get csidrivers,csinodes`, `kubectl get sc`.

Staff rule: DB in K8s only with operator (CloudNativePG, EBS snapshot backup tested restore monthly) else RDS. Always `Retain + snapshots + cross-AZ replication + PodDisruption`.

**Lab 11:** dynamic PVC, expand, `kubectl get pv,pvc,sc`, simulate AZ mismatch (EBS zone-a vs pod zone-b Pending).


---

## PART V - SECURE, OBSERVE, DELIVER

> ShopKart story, company switch: Ankit joins a bigger startup, Meena akka audits on day 3. "No latest tags, no root, no open ingress." Old habits break. This Part is how Ankit earns trust in the new company.

![Server racks - the new company Ankit must earn trust in](https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=900)
*Infra: rows of racks Ankit now owns. Photo via Unsplash. Free to use.*

### Chapter 12: Security - RBAC, NetworkPolicy, PSS, Supply Chain (Audit-Ready)

![Meena locks the shop - strict akka checks every key](https://images.unsplash.com/photo-1556740738-b6a63e27c4df?w=900)
*Story: checkout counter locked tight - Photo by Clay Banks on Unsplash (shop counter). Free to use.*

Senior bar 2026: PSS restricted + RBAC least-privilege + NetworkPolicy default-deny BEFORE prod. Auditors check these first.

RBAC deep: `Role` (ns: allow `get/list/create deployments in dev`) vs `ClusterRole` (cluster: `get nodes`, or all-ns). `RoleBinding` binds to ns, `ClusterRoleBinding` cluster-wide (danger: grants everywhere). ServiceAccount `default:dev` is pod identity (token projected, auto-rotated, 1h expiry). Never use `cluster-admin` for apps. Debug: `kubectl auth can-i create deployments --as=system:serviceaccount:dev:bot -n dev`, `kubectl auth can-i --list`. IRSA (EKS): SA annotates `eks.amazonaws.com/role-arn` → temporary AWS creds via OIDC, no static keys. Least-privilege: start deny, add verbs incrementally, separate human (`dev-read`) vs CI (`deployer`).

NetworkPolicy: default allow-all (if no policy selects pod). Prod: `default-deny ingress+egress` per ns + `allow DNS 53 + allow app labels`. Example deny-all + allow frontend→backend:53,5432. Needs Calico/Cilium (Flannel ignores → false secure). Test with `kubectl run tester --image=busybox -- nc -zv backend 80` before/after. Egress to internet needs `to: []` + DNS or FQDN policy (Cilium).

PSS (replaces PSP): namespaces labeled `pod-security.kubernetes.io/enforce: restricted` (deny privileged, require runAsNonRoot 1000, no hostPath, drop ALL caps), `baseline` (block privileged but allow some), `privileged` (kube-system only). Set `securityContext: {runAsNonRoot:true, runAsUser:1000, allowPrivilegeEscalation:false, readOnlyRootFilesystem:true, seccompProfile:{type:RuntimeDefault}}`.

Supply chain: no `:latest` (mutates, rollback impossible, use `:v1.2.3-sha`), scan with Trivy in CI (fail on CRITICAL), sign with Cosign + verify admission (Kyverno `verifyImages`), OPA Gatekeeper `require limits + deny hostNetwork`.

**Lab 12:** create dev Role, break (Forbidden), fix Binding, apply default-deny and watch app break then allow DNS.

### Chapter 13: Observability, Scaling, Reliability - SLOs + Autoscaling That Doesn't Flap

SLOs first: `availability 99.9% (43m/mo error budget) + p95 latency <300ms`. SLI via Prometheus (kube-prometheus-stack: server + node-exporter + kube-state-metrics), recording rules, Grafana burn-rate alerts (fast 5% in 1h, slow 2% in 6h). Logs JSON + `trace_id` propagated (OTel SDK → Collector → Tempo/Jaeger + Loki). Metrics: RED (Rate/Errors/Duration) per Service, USE per Node. Runbook per alert (link Ch16).

Scaling:
- HPA (pods): `min 3 max 20 target CPU 60% + custom kafka_lag`. Needs metrics-server (CPU/mem) + Prometheus Adapter (custom). Stabilization `scaleUp 0s, scaleDown 300s` prevents flap. Behavior: `kubectl autoscale deploy web --cpu-percent=60 --min=3 --max=20`.
- VPA (rightsizing): `mode: Off (recommend only)` → apply recs to requests weekly. Never VPA+HPA on same metric (fight).
- KEDA (event): `ScaledObject {trigger: kafka lag 100, rabbitmq queue}` → 0→N including to zero (HPA can't zero). Perfect for workers/AI inference queue.
- Cluster Autoscaler / Karpenter (nodes): watches Pending (no capacity) + underutilized (consolidate). Multi-AZ groups, spot for stateless (70% cheaper) + on-demand for DB. Senior = cost controller: right-size via VPA recs + HPA + suspend dev nights (kube-ns-suspender saves $300k/yr per Wolk).

Reliability: PDB `minAvailable:2 or maxUnavailable:1` (eviction/drain respects - drain blocked if would violate, use `--skip-wait` carefully), `terminationGracePeriod:60 + preStop sleep 10 + SIGTERM handler` drains in-flight, Rolling `maxUnavailable:0`.

### Chapter 14: Helm, Kustomize, GitOps - Ship Without SSH

Helm (package manager + templating): Chart (`Chart.yaml + templates/deploy.yaml with {{ .Values.replicas }} + values.yaml`), release (installed instance), repo. `helm upgrade --install web ./chart -f values-prod.yaml --set image.tag=v1.2.3 --atomic --timeout 5m`. Rollback `helm rollback web 2`. Staff: pin versions, `values.schema.json` validate, no secrets in values (use ESO), `helm template | kube-lint`.

Kustomize (no template, patch bases, Argo native): `bases: [../../base] + patches: [replicas.yaml, ingress-host.yaml] + images: [nginx=nginx:1.26]`. Better for GitOps multi-env (dev/stage/prod overlays).

GitOps golden (ArgoCD): Git = desired truth, Argo syncs (pull, not CI push). `Application {repo, path: overlays/prod, dest: prod-cluster, syncPolicy: automated prune+selfHeal for dev, manual for prod}`. Prod gate: manual sync or Rollouts canary (`steps: setWeight 10 → pause → analysis Prometheus error<1% → full`). AppSets for 100 microservices (`generators: git directories`). Secrets: SealedSecrets/ESO, never plain.

CI/CD split: CI builds+pushes `:sha` + updates Git (`kustomize edit set image`), CD (Argo) syncs. No `kubectl apply` from laptop in prod.


---

## PART VI - RUN LIKE STAFF

> ShopKart story, new company on-call: 2am PagerDuty, checkout p99 spikes, Ankit does not panic. Describe, logs, endpoints, fix in 18 minutes. Arjun texts "now you are the Arjun." This Part is that night.

![Global network at night - 2am PagerDuty Ankit does not fear](https://images.unsplash.com/photo-1451187580459-43490279c429?w=900)
*Infra: the world awake while Ankit fixes p99. Photo via Unsplash. Free to use.*

### Chapter 15: Production Operations Checklist - HA, Upgrades, Cost, Backup

From Wolk/Standout/DevOpsSchool 2026 Staff rubric (owns clusters, incident-tested, cost-aware):

1. HA: 3 AZ subnets, 3 masters (quorum 2), etcd snapshot hourly to S3 (`etcdctl snapshot save /backup/etcd-$(date).db` + `etcdutl verify`), control-plane LB, nodes in 2+ groups (system + workload). Stay N-1 version (1.29→1.30 tested in dev first, `kubeadm upgrade plan`, drain one AZ at a time, monitor PDB).
2. Nodes: MNG + Karpenter (`NodePool {requirements: arch, spot/on-demand, consolidation}`), AMI pinned, kubelet 1GB reserved. Right-size: `requests=p95, limits=p99` via VPA Off recs weekly, chargeback per ns (`kubecost`).
3. Policy: Kyverno `deny:latest, require requests/limits, require probes, require non-root`. GitOps + signed + scanned.
4. Upgrades: dev→stage→prod, backup etcd+Velero before, `kubectl drain --ignore-daemonsets`, canary 10% + analysis, rollback plan (<15min).
5. Backup/DR: Velero `velero backup create daily --include-namespaces prod --snapshot-volumes` + EBS snapshots + cross-region copy, restore game-day quarterly (RTO 45min like Palo Alto case).
6. Cost (Senior=cost controller): HPA+Karpenter+spot stateless, suspend dev 20:00-08:00, EFS→S3 lifecycle, delete orphan PVs (`kubectl get pv | grep Released`), image GC.
7. Free-tier warning: EKS control-plane ~$73/mo not free. Practice kind (free) + single EC2 t3.micro k3s (free 750h) for cloud.

### Chapter 16: Troubleshooting Runbooks - The 5 That Cover 90% (Staff Format)

Order every time: `describe (events bottom) → logs (-c --previous) → exec/env → endpoints → pvc/node/kubelet`.

```powershell
kubectl get pods -A -o wide; kubectl describe pod X # Conditions + LastState + Events
kubectl logs X -c app --previous --tail=100; kubectl logs X -c app -f
kubectl get events --sort-by=.lastTimestamp | Select-Object -Last 20
kubectl get endpointslices -l kubernetes.io/service-name=X; kubectl get pvc,sc
kubectl exec -it X -- sh -c "env | sort; cat /etc/resolv.conf; nc -zv db 5432"
docker exec -it mastery-control-plane journalctl -u kubelet -n 100 --no-pager
```

Map + runbook one-liners:
- ImagePullBackOff: wrong tag, private ECR no `imagePullSecrets`, rate-limit. `kubectl describe` shows 401/404. Fix secret `kubectl create secret docker-registry ecr --docker-server=...`.
- CrashLoopBackOff: `logs --previous` exit 1 app bug / 137 OOM (raise mem or fix leak) / 139 segfault / liveness wrong port (fix probe). `kubectl get pod -o jsonpath='{.status.containerStatuses[0].lastState}'`.
- Pending Unschedulable: `describe` shows `0/3 nodes: Insufficient cpu / Taint / Unbound PVC`. Fix requests, add toleration, check `kubectl get pvc` (EBS zone-a vs pod zone-b → use topology).
- Running 0/1 Ready: readiness failing (app not listening, DB down) → removed from Service, no traffic but not restarted. Fix readiness endpoint, check `curl localhost:8080/ready` via exec.
- No endpoints/Service 503: selector mismatch (`kubectl get pods --show-labels` vs `svc selector`), or all NotReady. Fix labels.
- Node NotReady/Unknown: kubelet down, disk pressure (`kubectl describe node` Pressure), certs expired (`openssl x509 -in /var/lib/kubelet/pki -noout -dates`), CNI down. `cordon/drain`, fix, `uncordon`.
Write 1-page runbook per failure: symptom → 3 hypotheses → 3 tests (commands) → mitigate + prevent (probe/limit/policy). On-call solves in 20min not 2h.

### Chapter 17: CKA + Interviews + Staff Design - Prove It

**CKA speed (do in <2min each, imperative only):**
```bash
k run nginx --image=nginx --dry-run=client -o yaml > p.yaml
k create deploy d --image=nginx:1.26 --replicas=3 --dry-run=client -o yaml
k expose deploy d --port=80 --target-port=8080 --type=NodePort
k config set-context --current --namespace=prod; k get pods --sort-by=.metadata.creationTimestamp
k rollout status deploy/d; k rollout undo deploy/d --to-revision=2
k create secret generic db --from-literal=password=x; k create cm cfg --from-file=app.conf
etcdctl snapshot save /tmp/etcd.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
kubeadm upgrade plan; systemctl restart kubelet; journalctl -u kubelet -f
k drain node1 --ignore-daemonsets --delete-emptydir-data; k uncordon node1
```
Exam tips: `alias k=kubectl`, `export do="--dry-run=client -o yaml"`, `kubectl explain --recursive`, contexts fast, `etcd` + `kubeadm` + `NetworkPolicy` + `Storage` always asked.

**Interview traps you now own (one-line answers):** Why Pod not container? (sidecars sharing fate). Why apiserver stateless? (scale/kill, etcd holds). Quorum 3→2? (majority avoids split-brain via Raft). Scheduler vs kubelet? (bind vs run). iptables vs IPVS? (O(n) vs O(1) hash). ndots:5 pain? (extra DNS). maxUnavailable 0? (zero downtime). Headless why? (direct pod DNS). RWO vs RWX? (EBS 1-node vs EFS many). liveness vs readiness? (restart vs remove). HPA needs? (metrics-server + stabilization). Init only? (invalid, need ≥1 app). Bare pod prod? (never, no heal).

**Staff scenarios (whiteboard 10min each):**
- 99.99% checkout: 3 AZ, Deploy 6 + anti-affinity host+zone + spread + PDB 2, HPA 3-20 CPU60, ClusterIP + Ingress nginx + WAF + cert-manager, EBS gp3 Retain + hourly snapshots, OTel traces + Prometheus burn alerts, Argo Rollouts canary 10% analysis, Velero daily + quarterly restore test.
- 4 monoliths →10 microservices + 200 ephemeral PR envs/quarter (E-Pods): ns per PR `pr-123`, Helm + Argo AppSet git generator, TTL 48h + auto-delete, ESO secrets, cost label `pr` showback,Scaled to zero after merge.
- Cut $300k/yr (Harness case): VPA recs right-size 30%, Karpenter spot 70% stateless, suspend dev nights, EFS→S3, delete orphan PVs/EBS, HPA downscale idle.

**Final teach-back:** Without notes 5min each: apply→pod flow, Service packet DNAT path, StatefulSet identity+storage, RBAC allow/deny, Pending debug. Record. If fluent, Senior-ready. Mentor one junior - Staff multiplies via others.


---

## PART VII - PRODUCTION LIBRARY (Copy-Paste Ready)

> ShopKart story, Staff years: Ankit mentors a junior who deletes prod at 1am, just like old Ankit. Instead of shouting, Ankit opens this library, hands the YAML, says "I have been you." This Part is what Staff hands over.

![High five - Staff Ankit hands the library to a junior](https://images.unsplash.com/photo-1521737604893-d14cc237f11d?w=900)
*Story: handover, not shouting. Photo via Unsplash. Free to use.*

### Chapter 18: Production YAML Library - 12 Manifests Staff Actually Apply

Use these as base. Each includes Senior defaults: 3 replicas, probes, resources, anti-affinity, PDB, non-root, labels.

**18.1 Production Deployment (zero-downtime):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: {name: api, namespace: prod, labels: {app: api, team: checkout}}
spec:
  replicas: 6
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
  minReadySeconds: 10
  strategy: {type: RollingUpdate, rollingUpdate: {maxSurge: 1, maxUnavailable: 0}}
  selector: {matchLabels: {app: api}}
  template:
    metadata: {labels: {app: api, version: v1.2.3}}
    spec:
      serviceAccountName: api-sa
      terminationGracePeriodSeconds: 60
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - {labelSelector: {matchLabels: {app: api}}, topologyKey: kubernetes.io/hostname}
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms: [{matchExpressions: [{key: topology.kubernetes.io/zone, operator: In, values: [a,b,c]}]}]
      topologySpreadConstraints:
      - {maxSkew: 1, topologyKey: topology.kubernetes.io/zone, whenUnsatisfiable: DoNotSchedule, labelSelector: {matchLabels: {app: api}}}
      containers:
      - name: api
        image: myreg/api:v1.2.3 # never :latest
        ports: [{containerPort: 8080, name: http}]
        envFrom: [{configMapRef: {name: api-cfg}}, {secretRef: {name: api-secret}}]
        resources: {requests: {cpu: 200m, memory: 256Mi}, limits: {cpu: 500m, memory: 512Mi}}
        livenessProbe: {httpGet: {path: /healthz, port: 8080}, initialDelaySeconds: 10, periodSeconds: 10, failureThreshold: 3}
        readinessProbe: {httpGet: {path: /ready, port: 8080}, initialDelaySeconds: 5, periodSeconds: 5, failureThreshold: 3}
        startupProbe: {httpGet: {path: /healthz, port: 8080}, failureThreshold: 30, periodSeconds: 5}
        lifecycle: {preStop: {exec: {command: ["sh","-c","sleep 10"]}}}
        securityContext: {runAsNonRoot: true, runAsUser: 1000, allowPrivilegeEscalation: false, readOnlyRootFilesystem: true}
        volumeMounts: [{name: tmp, mountPath: /tmp}, {name: cfg, mountPath: /etc/config}]
      volumes:
      - {name: tmp, emptyDir: {}}
      - {name: cfg, configMap: {name: api-cfg}}
      tolerations: [] # add gpu if needed
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: {name: api-pdb, namespace: prod}
spec: {minAvailable: 4, selector: {matchLabels: {app: api}}}
---
apiVersion: v1
kind: Service
metadata: {name: api, namespace: prod}
spec: {selector: {app: api}, ports: [{port: 80, targetPort: 8080}], type: ClusterIP}
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: {name: api, namespace: prod}
spec:
  scaleTargetRef: {apiVersion: apps/v1, kind: Deployment, name: api}
  minReplicas: 6
  maxReplicas: 20
  metrics: [{type: Resource, resource: {name: cpu, target: {type: Utilization, averageUtilization: 60}}}]
  behavior: {scaleDown: {stabilizationWindowSeconds: 300}, scaleUp: {stabilizationWindowSeconds: 0}}
```

**18.2 StatefulSet Postgres (with operator note):** Prefer CloudNativePG operator in prod, but raw STS for learning:
```yaml
apiVersion: v1
kind: Service
metadata: {name: pg-headless, namespace: data}
spec: {clusterIP: None, selector: {app: pg}, ports: [{port: 5432}]}
---
apiVersion: apps/v1
kind: StatefulSet
metadata: {name: pg, namespace: data}
spec:
  serviceName: pg-headless
  replicas: 3
  podManagementPolicy: OrderedReady
  updateStrategy: {type: RollingUpdate, rollingUpdate: {partition: 2}}
  selector: {matchLabels: {app: pg}}
  template:
    metadata: {labels: {app: pg}}
    spec:
      affinity: {podAntiAffinity: {requiredDuringSchedulingIgnoredDuringExecution: [{labelSelector: {matchLabels: {app: pg}}, topologyKey: kubernetes.io/hostname}]}}
      containers:
      - name: postgres
        image: postgres:15.4
        ports: [{containerPort: 5432}]
        env: [{name: POSTGRES_PASSWORD, valueFrom: {secretKeyRef: {name: pg-secret, key: password}}}]
        resources: {requests: {cpu: 500m, memory: 1Gi}, limits: {cpu: 1000m, memory: 2Gi}}
        volumeMounts: [{name: data, mountPath: /var/lib/postgresql/data}]
        livenessProbe: {exec: {command: ["pg_isready","-U","postgres"]}, initialDelaySeconds: 30, periodSeconds: 10}
        readinessProbe: {exec: {command: ["pg_isready","-U","postgres"]}, initialDelaySeconds: 5, periodSeconds: 5}
  volumeClaimTemplates:
  - metadata: {name: data}
    spec: {accessModes: [ReadWriteOnce], storageClassName: gp3, resources: {requests: {storage: 20Gi}}}
```
Notes: `reclaimPolicy: Retain` on SC, snapshot before upgrade, `pg_basebackup` test restore monthly. Delete STS keeps PVCs - manual `kubectl delete pvc data-pg-0`.

**18.3 DaemonSet Fluent-Bit, CronJob backup, Ingress, NetworkPolicy, RBAC, PVC, ConfigMap:**
```yaml
# DaemonSet tolerates masters
apiVersion: apps/v1
kind: DaemonSet
metadata: {name: fluent-bit, namespace: logging}
spec:
  selector: {matchLabels: {app: fluent-bit}}
  template:
    metadata: {labels: {app: fluent-bit}}
    spec:
      tolerations: [{key: node-role.kubernetes.io/control-plane, operator: Exists, effect: NoSchedule}]
      containers: [{name: fluent-bit, image: fluent/fluent-bit:2.2, resources: {requests: {cpu: 100m, memory: 128Mi}, limits: {cpu: 200m, memory: 256Mi}}}]
---
apiVersion: batch/v1
kind: CronJob
metadata: {name: pg-backup, namespace: data}
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 3
      activeDeadlineSeconds: 1800
      ttlSecondsAfterFinished: 86400
      template:
        spec:
          restartPolicy: OnFailure
          containers: [{name: backup, image: postgres:15.4, command: ["sh","-c","pg_dumpall | gzip > /backup/dump-$(date +%F).sql.gz"], volumeMounts: [{name: backup, mountPath: /backup}], resources: {requests: {cpu: 200m, memory: 256Mi}}}]
          volumes: [{name: backup, persistentVolumeClaim: {claimName: backup-pvc}}]
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata: {name: web, namespace: prod, annotations: {nginx.ingress.kubernetes.io/rate-limit: "100"}}
spec:
  ingressClassName: nginx
  tls: [{hosts: [shop.example.com], secretName: shop-tls}]
  rules: [{host: shop.example.com, http: {paths: [{path: /, pathType: Prefix, backend: {service: {name: web, port: {number: 80}}}}, {path: /api, pathType: Prefix, backend: {service: {name: api, port: {number: 80}}}}]}}]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny, namespace: prod}
spec: {podSelector: {}, policyTypes: [Ingress, Egress]}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: allow-front-to-back, namespace: prod}
spec:
  podSelector: {matchLabels: {app: api}}
  policyTypes: [Ingress]
  ingress: [{from: [{podSelector: {matchLabels: {app: web}}}], ports: [{port: 8080}]}]
  egress: [{to: [{namespaceSelector: {matchLabels: {name: kube-system}}, podSelector: {matchLabels: {k8s-app: kube-dns}}}], ports: [{port: 53, protocol: UDP}]}]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: {name: deployer, namespace: prod}
rules: [{apiGroups: ["apps"], resources: ["deployments"], verbs: ["get","list","patch","update"]}, {apiGroups: [""], resources: ["pods","events"], verbs: ["get","list"]}]
---
apiVersion: v1
kind: ServiceAccount
metadata: {name: ci-bot, namespace: prod}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: {name: ci-deployer, namespace: prod}
roleRef: {apiGroup: rbac.authorization.k8s.io, kind: Role, name: deployer}
subjects: [{kind: ServiceAccount, name: ci-bot, namespace: prod}]
```

Teach: apply each, break selector, watch EndpointSlice empty, fix. `kubectl auth can-i --as=system:serviceaccount:prod:ci-bot create deployments -n prod` should be no (only patch), adjust.

### Chapter 19: 30 Break-Fix Labs - From Error to Runbook (Do All)

Format per lab: SYMPTOM → HYPOTHESES (3) → TESTS (commands) → FIX → PREVENT. Timebox 15min each. Labs 1-10 beginner, 11-20 mid, 21-30 Staff.

1. ImagePullBackOff wrong tag: `kubectl run bad --image=nginx:9.99`. Test `describe` 404. Fix tag. Prevent pin + scan.
2. Private registry no secret: ECR image without imagePullSecrets → 401. `kubectl create secret docker-registry`. Attach to SA.
3. CrashLoop bad liveness: liveness `/healthz` on nginx (404) → restart 5x. `logs --previous`. Fix path or remove. Prevent startupProbe.
4. OOMKilled: `stress --vm 1 --vm-bytes 600M` with limit 256Mi → exit 137. `describe` LastState. Fix raise limit + leak check.
5. Pending CPU: requests 8 CPU on 4 CPU kind → Unschedulable. `describe` Insufficient cpu. Fix lower + HPA.
6. Pending taint: taint node `dedicated=true:NoSchedule`, pod no toleration → Pending. Add toleration or remove taint.
7. PVC unbound: PVC 100Gi with no PV/SC → Pending. `kubectl get pvc` Pending. Create SC or smaller.
8. AZ volume mismatch: EBS zone-a, pod zone-b (affinity) → `Volume zone mismatch`. Use `allowedTopologies` or EFS.
9. Selector mismatch: Service `app: web` vs pods `app: web-v2` → `endpointslices 0`. `get pods --show-labels`. Fix selector.
10. Readiness 0/1: readiness `/ready` fails (DB down) → Running but no traffic. Exec `curl localhost:ready`. Fix DB env.
11. Liveness vs readiness swapped: liveness checks DB → DB blip restarts all pods cascade. Move DB check to readiness only.
12. ConfigMap not updating (env): envFrom needs restart. Change to volume mount or `kubectl rollout restart`.
13. Secret in Git leaked: base64 committed. Rotate, enable ESO + SealedSecrets, add pre-commit.
14. RBAC Forbidden: `ci-bot` can't create deploy → 403. `auth can-i --as`. Fix RoleBinding.
15. NetworkPolicy block: default-deny without DNS allow → `nslookup` fails. Add egress 53 + allow. Test with busybox.
16. Ingress 404: wrong `pathType` Exact vs Prefix `/api` vs `/api/`. Fix Prefix + rewrite-target annotation.
17. TLS secret mismatch: `tls.crt` for `foo.com` on `bar.com` → browser error. Recreate `kubectl create secret tls` + cert-manager.
18. HPA never scales: no metrics-server → `<unknown>/60%`. Install metrics-server, set requests (HPA needs requests).
19. HPA flapping: scaleDown 0s + CPU spiky → 3→20→3. Set stabilization 300s + custom avg.
20. PDB blocks drain: `minAvailable:3` with 3 replicas → `drain` hangs Cannot evict. Temporarily lower or add replica.
21. Node NotReady disk: `df` 95% + `describe node` DiskPressure → evicts BestEffort. Clean `/var/lib/docker`, add ephemeral-storage limit.
22. etcd slow: `wal_fsync 200ms` → apiserver timeouts. `etcdctl endpoint status`, move to gp3, defrag, snapshot.
23. CoreDNS 5s delay: `ndots:5` + external `google.com` → 4 local NXDOMAIN first. Use FQDN `google.com.` or NodeLocal DNSCache.
24. Conntrack full: 1M connections + NodePort → `nf_conntrack: table full, dropping`. Raise `nf_conntrack_max`, use IPVS/Cilium.
25. Upgrade stuck: `kubeadm upgrade` with PDB + old CSI → pods Pending. Pre-check `planner`, backup etcd, upgrade CSI first.
26. Helm rollback fail: `helm upgrade` bad image + `--wait` timeout → release Failed. `helm rollback`, `helm history`, add `--atomic`.
27. Argo out-of-sync: manual `kubectl edit` vs Git → drift. `argocd app diff`, enable selfHeal + ignoreDifferences HPA replicas.
28. CronJob flood: `*/1 * * * *` with `history 1000` → 10k Jobs etcd slow. Set `successfulJobsHistoryLimit:3`, `ttlSecondsAfterFinished`.
29. Spot eviction: Karpenter spot killed mid-write → 500. Add `toleration spot` + `on-demand` for DB + `terminationGrace + preStop` + PDB.
30. Cost blowup: orphan `Released` PVs + LoadBalancer per Service + `:latest` pulls. `kubectl get pv | grep Released`, `kubecost`, switch ClusterIP + Ingress, pin tags.

For each write 1-page runbook (symptom/hypothesis/test/mitigate/prevent) - Staff promotion evidence.

### Chapter 20: Senior/Staff Interview Bank (100 in 20) + 3 Case Studies

**Bank (answer in 60s each, teach-line):** 1 Pod why not container? 2 pause what holds? 3 init vs sidecar? 4 Unknown vs Pending? 5 bare pod why never? 6 RS selector vs template bug? 7 Deployment how rollback? 8 maxSurge math 10/2? 9 maxUnavailable 0 when? 10 STS vs Deploy 5 diffs? 11 headless when? 12 volumeClaimTemplate? 13 DaemonSet why tolerates masters? 14 Job completions/parallelism? 15 CronJob history why? 16 ClusterIP vs NodePort vs LB vs ExternalName? 17 iptables vs IPVS? 18 EndpointSlice empty means? 19 Ingress needs what? 20 Gateway API why better? 21 CoreDNS ndots pain fix? 22 Flannel vs Calico vs Cilium pick? 23 ConfigMap env vs volume update? 24 Secret base64 secure? 25 liveness vs readiness vs startup? 26 requests vs limits OOM vs throttle? 27 QoS order? 28 nodeAffinity vs podAntiAffinity? 29 taint effects 3? 30 topologySpread maxSkew? 31 RWO vs RWX which EBS? 32 Retain vs Delete? 33 CSI what? 34 RBAC Role vs ClusterRole? 35 SA + IRSA? 36 NetworkPolicy default? 37 PSS restricted 3 rules? 38 HPA needs what 2? 39 VPA vs HPA conflict? 40 KEDA to zero? 41 Karpenter vs CAS? 42 PDB what blocks? 43 Helm values vs Kustomize overlay? 44 GitOps Argo sync manual vs auto prod? 45 etcd quorum 5→? 46 Raft leader write steps? 47 scheduler Filter then Score example? 48 Binding what? 49 static vs mirror? 50 apiserver stateless why?

**Case 1 - 99.99% checkout (fintech, 10k RPS):** 3 AZ EKS 1.30, 6 api replicas anti-affinity host+zone + spread, HPA 6-30 CPU60 + KEDA queue, PDB minAvailable 4, ClusterIP + ingress-nginx (3 replicas anti-affinity) + WAF + cert-manager, RDS Multi-AZ (not STS) + ElastiCache, OTel → Prometheus burn (5% 1h page), Argo Rollouts canary 10% 5min analysis error<0.5% else auto-rollback, Velero daily + RDS snapshots + quarterly game-day (kill AZ, verify 1-1-1). Result 99.99%, MTTR 18min.

**Case 2 - 4 monoliths →10 microservices + 200 PR envs/quarter (SaaS):** Namespaces `pr-123` TTL 48h (`janitor ttl`), Helm charts + Argo AppSet git generator (`path: envs/*`), ESO per-PR DB (Neon branch), KEDA to zero after 30m idle, cost label `pr` → Kubecost showback $12/env vs $80 before, `kube-ns-suspender` nights. Deploy time 2h→12min, main protected by manual sync + policy (deny latest).

**Case 3 - Cut $300k/yr + AI inference on K8s (Harness-like):** Audit: 40% over-provisioned (requests 2x usage via VPA recs), 20 LoadBalancers ($400/mo each) → 2 Ingress ALBs, dev 24/7 → suspend 20-8 + weekends, spot 70% stateless via Karpenter (on-demand DB), EFS infrequent → S3 lifecycle, orphan 200Gi PVs deleted. AI: GPU nodes taint `nvidia.com/gpu:NoSchedule` + toleration + `nodeAffinity gpu`, Cilium (eBPF, no kube-proxy overhead for 10k RPS inference), KEDA `prometheus qps>50` scale 0→20 T4, model weights on EFS RWX shared + init download. Bill $520k→$210k, p95 inference 800ms→320ms.

**Final:** Record 5min whiteboard per case without notes. If you can teach all 20 chapters + do 30 labs + answer 50/50, you are Senior/Staff ready. Mentor one junior next.

### Chapter 21: Observability Stack + Policy Guardrails + Progressive Delivery YAML

**21.1 kube-prometheus-stack values (prod minimal):**
```yaml
# values-prom.yaml
prometheus:
  prometheusSpec:
    retention: 15d
    resources: {requests: {cpu: 500m, memory: 2Gi}, limits: {cpu: 1000m, memory: 4Gi}}
    storageSpec: {volumeClaimTemplate: {spec: {storageClassName: gp3, accessModes: [ReadWriteOnce], resources: {requests: {storage: 50Gi}}}}}
grafana:
  adminPassword: <via-ESO>
  persistence: {enabled: true, storageClassName: gp3, size: 10Gi}
alertmanager:
  config:
    route: {receiver: pagerduty, group_by: [alertname, namespace], routes: [{match: {severity: critical}, receiver: pagerduty}]}
    receivers: [{name: pagerduty, pagerduty_configs: [{service_key: <key>}]}]
kubeStateMetrics: {enabled: true}
nodeExporter: {enabled: true}
# Burn-rate recording example
# recording: - record: job:slo_errors_ratio:rate5m, expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
```
Install: `helm install kube-prom kube-prometheus-stack/kube-prometheus-stack -n monitoring --create-namespace -f values-prom.yaml`. Add ServiceMonitor per app:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata: {name: api, namespace: monitoring}
spec: {selector: {matchLabels: {app: api}}, endpoints: [{port: http, interval: 15s, path: /metrics}]}
```

**21.2 OTel Collector + Loki + Tempo (traces + logs):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata: {name: otel-collector, namespace: observability}
data:
  config.yaml: |
    receivers: {otlp: {protocols: {grpc: {}, http: {}}}}
    processors: {batch: {}, memory_limiter: {limit_mib: 512}}
    exporters: {prometheus: {endpoint: 0.0.0.0:8889}, loki: {endpoint: http://loki:3100/loki/api/v1/push}, tempo: {endpoint: tempo:4317}}
    service: {pipelines: {traces: {receivers: [otlp], processors: [batch], exporters: [tempo]}, metrics: {receivers: [otlp], exporters: [prometheus]}, logs: {receivers: [otlp], exporters: [loki]}}}
```
App SDK sets `OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317`, adds `trace_id` to JSON logs. Grafana datasources: Prometheus + Loki (`{app="api"} | json | trace_id="..."`) + Tempo (traceID jump). Alert: `sum(rate(http_server_errors[5m]))/sum(rate(http_server_total[5m])) > 0.01` page.

**21.3 Kyverno policies (deny latest, require limits/probes/non-root):**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: {name: require-basics}
spec:
  validationFailureAction: Enforce
  rules:
  - name: deny-latest
    match: {any: [{resources: {kinds: [Pod]}}]}
    validate: {message: "No :latest, pin tag", pattern: {spec: {containers: [{image: "!*:latest"}]}}}
  - name: require-limits
    match: {any: [{resources: {kinds: [Pod]}}]}
    validate: {message: "Set requests+limits", pattern: {spec: {containers: [{resources: {requests: {cpu: "?*", memory: "?*"}, limits: {cpu: "?*", memory: "?*"}}}]}}}
  - name: require-probes
    match: {any: [{resources: {kinds: [Pod]}}]}
    validate: {message: "Set liveness+readiness", pattern: {spec: {containers: [{livenessProbe: {httpGet: {path: "?*"}}, readinessProbe: {httpGet: {path: "?*"}}}]}}}
  - name: require-nonroot
    match: {any: [{resources: {kinds: [Pod]}}]}
    validate: {message: "runAsNonRoot", pattern: {spec: {securityContext: {runAsNonRoot: true}}}}
```
Test: `kubectl apply --dry-run=server -f bad-pod.yaml` should be denied. Audit mode first (`Audit` not `Enforce`) to measure blast.

**21.4 ArgoCD App + Rollout canary + Velero:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: {name: api-prod, namespace: argocd}
spec:
  project: default
  source: {repoURL: https://github.com/org/shop.git, targetRevision: main, path: overlays/prod}
  destination: {server: https://kubernetes.default.svc, namespace: prod}
  syncPolicy: {automated: {prune: true, selfHeal: false}, syncOptions: [CreateNamespace=true]}
  ignoreDifferences: [{group: autoscaling, kind: HorizontalPodAutoscaler, jsonPointers: [/spec/replicas]}]
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata: {name: api, namespace: prod}
spec:
  replicas: 6
  strategy:
    canary:
      steps: [{setWeight: 10}, {pause: {duration: 5m}}, {analysis: {templates: [{templateName: success-rate}], args: [{name: service, value: api}] }}, {setWeight: 50}, {pause: {duration: 10m}}]
  selector: {matchLabels: {app: api}}
  template: {metadata: {labels: {app: api}}, spec: {containers: [{name: api, image: myreg/api:v1.2.4}]}}
---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata: {name: success-rate, namespace: prod}
spec:
  metrics: [{name: success-rate, interval: 1m, successCondition: result[0] > 0.99, provider: {prometheus: {address: http://kube-prom-prometheus.monitoring:9090, query: sum(rate(http_requests_total{app="api",status!~"5.."}[5m])) / sum(rate(http_requests_total{app="api"}[5m]))}}}]
# Velero daily
# velero install --provider aws --bucket shop-velero --backup-location-config region=us-east-1 --snapshot-location-config region=us-east-1
# velero schedule create daily-prod --schedule="0 3 * * *" --include-namespaces prod,data --snapshot-volumes --ttl 720h
# velero backup create manual-$(date +%F) --include-namespaces prod --wait; velero restore create --from-backup manual-2026-10-10 --namespace-mappings prod:prod-restore --wait
```

### Chapter 22: CKA Mock Exam (2 Hours, 17 Tasks, Solutions)

Rules: `alias k=kubectl`, `export do="--dry-run=client -o yaml"`, new cluster `kind create cluster --name mock`. Score 66% to pass.

1. RBAC (7%): Create ns `team-a`, SA `deployer`, Role `deployer` can `get/list/create deployments + get pods`, bind. Test `k auth can-i --as=system:serviceaccount:team-a:deployer create deployments -n team-a` → yes.
Solution: `k create ns team-a; k create sa deployer -n team-a; k create role deployer --verb=get,list,create --resource=deployments --verb=get --resource=pods -n team-a --dry-run=client -o yaml | k apply -f -; k create rolebinding b --role=deployer --serviceaccount=team-a:deployer -n team-a`.

2. Pod with probes+limits (5%): Pod `web` nginx:1.26, requests 100m/128Mi limits 200m/256Mi, liveness `/` port 80 delay 5, readiness `/` delay 3. `k run web --image=nginx:1.26 $do > p.yaml` edit + apply + `k describe`.

3. Deployment rollout (7%): Deploy `api` 6x nginx:1.25, surge1 unavail0, set image 1.26, history, undo. Commands Ch17.

4. StatefulSet + headless (6%): `k apply -f https://k8s.io/examples/application/web/web.yaml`, `k get sts,pvc`, explain ordinal.

5. Service debug (6%): Create deploy `bad` + svc selector wrong → 0 endpoints. `k get endpointslices`, fix selector. Must show before/after.

6. Ingress (6%): ingress-nginx `shop.example.com / → web:80` with TLS secret. `k create secret tls`.

7. NetworkPolicy (7%): default-deny ns `prod` + allow `web→api:8080` + DNS. Apply, test `busybox nslookup + wget`.

8. Storage (7%): PVC 5Gi gp3 RWO + pod mounts `/data` writes file survives restart (`k exec echo hi > /data/hi; k delete pod; k exec cat`).

9. Nodes (6%): cordon/drain node, daemonset stays, uncordon. `k cordon/drain/uncordon`.

10. Logs/events (4%): `k logs deploy/api -c api --previous --tail=50`, `k get events --sort-by`.

11. HPA (5%): `k autoscale deploy/api --cpu-percent=60 --min=3 --max=10`, `k top pods` (install metrics-server if needed).

12. etcd backup/restore (7%): `ETCDCTL_API=3 etcdctl snapshot save /tmp/s.db --cacert... --cert... --key...; etcdctl snapshot status; snapshot restore --data-dir /tmp/restore`.

13. Upgrade (6%): `kubeadm version`, `kubeadm upgrade plan`, `apt-mark unhold kubeadm && apt install kubeadm=1.30.x && kubeadm upgrade apply v1.30.x`, drain, `kubelet --version`.

14. Fix kubelet (5%): `systemctl status kubelet`, `journalctl -u kubelet`, fix `/etc/kubernetes/kubelet.conf`, restart.

15. Helm (5%): `helm repo add bitnami; helm install redis bitnami/redis -n data --set auth.password=x`, `helm list`, `rollback`.

16. CronJob (5%): `*/2 * * * *` busybox echo, history 3/1, `k get jobs --watch`.

17. Gateway/CRD (6%): `k get crds | grep gateway`, create HTTPRoute `host→svc weight 90/10` (if controller installed) else explain.

Check: `k get all -A`, time left buffer 15min for stuck (skip + return). Practice 3x timed.

### Chapter 23: Platform Setup Guide - Windows + kind + k9s + EKS vs k3s (No More Setup Hell)

**23.1 Windows Docker Desktop + kind (your path):**
```powershell
# Requires WSL2 + Docker Desktop running (Switch to Linux containers)
choco install kubernetes-cli kind k9s lens -y
docker info # must show Server Running
kind create cluster --name mastery --config - # default 1 control-plane
# HA test (3 masters, needs 8GB RAM):
# kind create cluster --name ha --config ha-kind.yaml
kubectl cluster-info; kubectl get nodes -o wide
kubectl create ns prod,dev,monitoring
# k9s for TUI: k9s -n prod (press :pods, d describe, l logs, s shell)
# Lens: Add kubeconfig C:\Users\<you>\.kube\config
```
Quirks: kind nodes are Docker containers (`docker ps` shows mastery-control-plane). `LoadBalancer` stays Pending (no cloud) - use `NodePort` or `kubectl port-forward svc/api 8080:80`. PVC uses `rancher.io/local-path` (hostPath, data lost on cluster delete - fine for labs, never prod). Memory: limit Docker Desktop 6GB+ or kind OOMs. Reset: `kind delete cluster --name mastery; kind create cluster --name mastery`.

`ha-kind.yaml`:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
networking: {podSubnet: 10.244.0.0/16, serviceSubnet: 10.96.0.0/12}
```

**23.2 Free-tier cloud (avoid $73 EKS fee):** EKS control-plane $73/mo + NAT + LB - NOT free. For cloud practice use single EC2 `t3.micro` free 750h/mo:
```bash
# EC2 Amazon Linux 2023 t3.micro, 30GB gp3, SG 22+6443
curl -sfL https://get.k3s.io | sh - # lightweight K8s, Traefik Ingress + Flannel + local-path built-in
sudo k3s kubectl get nodes; sudo cat /etc/rancher/k3s/k3s.yaml > kubeconfig (replace 127.0.0.1 with EC2 IP)
k3s kubectl apply -f deploy.yaml; k3s kubectl get svc -n kube-system traefik
```
k3s vs kind: kind = Docker nodes for CKA parity, k3s = real VM + systemd + Ingress ready for cloud demo. Both free. EKS only when company pays.

**23.3 Tooling Staff uses daily:** `kubectl` + `k9s` (TUI logs/shell/port-forward), `stern -n prod -l app=api --tail 50` (multi-pod logs), `kubectx/kubens` (switch fast), `helm + kustomize`, `velero`, `kubecost`. Never edit prod via Lens click - GitOps only.

### Chapter 24: GPU/AI on K8s + Multi-Cluster + Service Mesh (Staff Edge 2026)

![Woman engineer at boards - ShopKart AI team plans GPU pools](https://images.unsplash.com/photo-1573164713988-8665fc963095?w=900)
*Story: AI planning wall, GPU taints next. Photo via Unsplash. Free to use.*

**24.1 GPU scheduling (66% of AI inference runs on K8s per CNCF 2025):**
Nodes: `p3.2xlarge` or `g5.xlarge` with `nvidia.com/gpu: 1`, taint `nvidia.com/gpu=true:NoSchedule` (only AI tolerates, web never lands and wastes $3/hr). Install NVIDIA device plugin DaemonSet (`nvidia/k8s-device-plugin`), driver via EKS AMI GPU.
```yaml
tolerations: [{key: nvidia.com/gpu, operator: Exists, effect: NoSchedule}]
nodeSelector: {accelerator: nvidia-t4}
resources: {limits: {nvidia.com/gpu: 1}, requests: {cpu: 2, memory: 8Gi}}
affinity: {nodeAffinity: {requiredDuringSchedulingIgnoredDuringExecution: {nodeSelectorTerms: [{matchExpressions: [{key: accelerator, operator: In, values: [nvidia-t4]}]}]}}}}
```
Patterns: model weights on EFS RWX shared (`models-pvc 100Gi RWX`) + initContainer `aws s3 cp s3://models/llama /models` once per node (emptyDir cache), inference Deployment HPA custom `qps>50` + KEDA `kafka lag`, spot for batch (training checkpoint to S3), on-demand for serving. Cilium (no kube-proxy iptables overhead for 10k RPS) + TopologySpread zone. Cost: Karpenter `requirements: [{key: karpenter.sh/capacity-type, operator: In, values: [spot]}]` for batch 70% off.

**24.2 Multi-cluster (when 1 cluster isn't enough):** Reasons: blast radius (prod vs dev), data residency (eu vs us), scale (>5000 nodes). Options: Karmada/Argo AppSets (Git to N clusters), Cilium ClusterMesh (pod IP routable across clusters `10.244.0.0 ↔ 10.245.0.0` + shared Service `api.global`), Istio multi-primary (shared root CA). Start single, split only on compliance/scale. Practice: `kind create cluster --name c1/c2`, `cilium install --cluster-id 1/2`, `kubectl --context kind-c1 get pods`.

**24.3 Service mesh: need or not?** Sidecar Envoy (Istio/Linkerd) gives mTLS + retries + canary + trace without code. Cost: 100m CPU + 128Mi per pod + 5ms latency + operate control-plane. Rule: <20 services → Ingress + OTel SDK enough. 50+ with strict mTLS/audit (bank) → Linkerd (light, Rust) or Istio ambient (no sidecar, eBPF). Cilium ServiceMesh (eBPF, no sidecar) is Staff pick 2026 for perf. Demo: `linkerd install | kubectl apply -f -; linkerd viz install | kubectl apply; kubectl annotate ns prod linkerd.io/inject=enabled; linkerd viz stat deploy -n prod`.

**When to stop:** If you can whiteboard GPU toleration + multi-cluster tradeoffs + mesh yes/no with cost/latency numbers, you beat 90% Staff loops.

### Chapter 25: Windows Troubleshooting + cert-manager + external-dns End-to-End

**25.1 Docker Desktop + WSL2 fixes (90% Windows pain):**
- `kind create` hangs: Docker Desktop not running or WSL2 not default. Fix: `wsl --set-default-version 2; wsl --install`, restart Docker, `docker ps` must work. RAM: Settings → Resources → 6GB+ CPUs 4, disk 60GB. kind 1 control-plane needs 2GB, HA 3+2 needs 8GB or OOM.
- Disk full `no space left`: WSL vhdx grows forever. `wsl --shutdown`, `Optimize-VHD -Path C:\Users\<you>\AppData\Local\Docker\wsl\data\ext4.vhdx -Mode Full`, or `docker system prune -a --volumes`, `kind delete cluster` old.
- `kubectl` context wrong: `kubectl config get-contexts; kubectl config use-context kind-mastery`. Kubeconfig `C:\Users\<you>\.kube\config` overwritten by k3s copy - merge with `KUBECONFIG=~/.kube/config:./k3s.yaml kubectl config view --flatten > merged`.
- Port-forward dies: `kubectl port-forward svc/api 8080:80 --address 0.0.0.0` + Windows firewall allow. LoadBalancer Pending on kind is normal - use NodePort `30080`.
- Line endings: YAML with CRLF breaks Helm (`\r`). VSCode → LF, `.gitattributes: *.yaml text eol=lf`.

**25.2 cert-manager (auto TLS, no manual secrets):**
```powershell
helm install cert-manager jetstack/cert-manager -n cert-manager --create-namespace --set installCRDs=true
```
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata: {name: letsencrypt-prod}
spec: {acme: {server: https://acme-v02.api.letsencrypt.org/directory, email: you@example.com, privateKeySecretRef: {name: le-prod}, solvers: [{http01: {ingress: {class: nginx}}}]}}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata: {name: shop-tls, namespace: prod}
spec: {secretName: shop-tls, issuerRef: {name: letsencrypt-prod, kind: ClusterIssuer}, dnsNames: [shop.example.com]}
```
Ingress just references `secretName: shop-tls` - cert-manager creates/renews (60d → renew 30d). Debug: `kubectl get challenges,orders`, `describe challenge` DNS propagation. Staging first (`letsencrypt-staging` to avoid rate limit 50/week).

**25.3 external-dns (Route53 auto records):**
```yaml
# IRSA role allows route53:ChangeResourceRecordSets on hosted zone
apiVersion: v1
kind: ServiceAccount
metadata: {name: external-dns, namespace: kube-system, annotations: {eks.amazonaws.com/role-arn: arn:aws:iam::123:role/external-dns}}
---
helm install external-dns external-dns/external-dns -n kube-system --set provider=aws --set txtOwnerId=prod-cluster --set domainFilters[0]=example.com --set serviceAccount.create=false --set serviceAccount.name=external-dns
# Ingress annotation: external-dns.alpha.kubernetes.io/hostname: shop.example.com
# Service annotation same for LB. Verify: aws route53 list-resource-record-sets --hosted-zone-id Z123 | grep shop
```
Flow: Ingress host → external-dns watches → UPSERT A → ALB DNS → traffic. No manual Route53 click. TXT `heritage=external-dns` prevents hijack.

Lab: kind (no AWS) simulate with `nip.io`: `shop.192-168-1-10.nip.io → 192.168.1.10`, then same YAML works on EKS with real zone.

### Chapter 26: Deep Appendix - etcd Math + Scheduler Scoring Worked Examples

**26.1 etcd numbers Staff quotes:** Quorum table: 1→1 (0 fail), 3→2 (1 fail), 5→3 (2 fail), 7→4 (3 fail, rarely used - 4 must ack, slower). Latency budget: WAL fsync p99 <10ms (gp3 3000 IOPS ok, st1 50ms = elections flap). Size: max DB 8GB (default quota 2GB, raise `--quota-backend-bytes=8G`), compaction every 5min keeps 1000 revs (`--auto-compaction-retention=1000`). Backup: `etcdctl snapshot save` hourly to S3 + `etcdutl verify + defrag` weekly (frees 30% after churn). Restore test: `snapshot restore --data-dir /tmp/restore --initial-cluster m1=http://1:2380` then `etcd --data-dir` single, verify `get /registry/deployments`.

Split-brain drill: partition 3 into 2-1. Side 2 (leader) keeps `PUT rev 100→101`. Side 1 (candidate term+1) can't get majority, `election timeout → candidate → 1 vote <2 → retry`. Heal: side1 sees higher term 5>4 with committed log 101, steps down, AppendEntries replays 101, commitIndex advances. No data loss (majority had it). If you ran 2 members, partition 1-1 → both halt (no majority) → full outage. That's why odd.

**26.2 Scheduler worked scoring:** Nodes: n1 8CPU/32GB used 7CPU (87%), has image; n2 8CPU used 2CPU (25%), no image (pull 30s); n3 tainted `gpu=true:NoSchedule`. Pod: requests 1CPU, tolerates nothing, image nginx:1.26.
Filter: n3 out (taint). n1, n2 pass (CPU fit, zone ok, ports free).
Score (weights LeastAllocated 1, ImageLocality 1): n1 Least 13 (100-87), Image 100 → avg 56. n2 Least 75, Image 0 → avg 37. Winner n1 despite fuller (image wins) - unless `MostAllocated` for binpack (cost) flips to n1 even stronger. Change to `LeastAllocated` weight 5 → n1 (13*5+100)/6=27, n2 (75*5+0)/6=62 → n2 wins (spread). Staff tunes weights per goal: cost → MostAllocated, perf → Least + spreading.

Preemption example: critical `priority 1000000` pod Pending, low `priority 0` pods on n1. Scheduler checks PDB (can evict 1 of 3?), evicts lowest priority victim, binds critical, ReplicaSet recreates victim elsewhere. `kubectl describe pod critical | grep Preempted`.

Practice: `kubectl describe node | grep -A5 Allocated`, `kubectl top nodes`, `kube-scheduler --config` weights, `kubectl get events --field-selector reason=FailedScheduling`.

### Chapter 27: Hardening Checklist - CIS + Image Pipeline + Audit (Prod Gate)

**27.1 CIS Benchmarks (kube-bench fail → pass):** Run `aquasec/kube-bench` as Job. Common fails + fixes:
- 1.2.1 `--anonymous-auth=false` on apiserver (deny unauth). Fix kubeadm manifest.
- 1.2.7 `--authorization-mode=RBAC,Node` (not AlwaysAllow).
- 2.1 `--profiling=false` (disable debug pprof), `--audit-log-path=/var/log/apiserver/audit.log --audit-policy-file` (log get/list secrets).
- 4.2 kubelet `--read-only-port=0`, `--protect-kernel-defaults=true`, `--tls-cert-file` rotation (`--rotate-certificates=true`).
- 5.1 RBAC `cluster-admin` bound to `system:authenticated` → remove, least-privilege.
Gate: kube-bench score >90% before prod + `kyverno` Enforce (Ch21) + `falco` runtime (detect `sh in prod`, `sensitive file read /etc/shadow`).

**27.2 Image pipeline (build → scan → sign → verify):**
```yaml
# .github/workflows/build.yaml
# build: docker build -t reg/api:${{ github.sha }} . ; trivy image --severity CRITICAL --exit-code 1 reg/api:${{ github.sha }} (fail on CVE); cosign sign --yes reg/api:${{ github.sha }}
# verify admission (Kyverno):
#  - name: verify-signature, match Pods, verifyImages: [{imageRefs: ["reg/api:*"], attestors: [{entries: [{keys: {publicKeys: <cosign.pub>}}]}]}]
```
Rules: distroless/chainguard base (no shell, 5MB), non-root USER 1000, no secrets in layers (`docker history`), SBOM `syft` attached. Admission denies unsigned/latest.

**27.3 Audit + Falco + ESO rotation:** Audit policy log `pods/exec` + `secrets get` to Loki/S3 (1yr for SOC2). Falco rules: `Terminal shell in container`, `Read sensitive file`, `Outbound to C2`. ESO `ExternalSecret {refreshInterval: 1h, secretStore: AWS, data: [{remoteRef: {key: prod/db}, secretKey: password}]}` + rotation Lambda updates AWS → ESO syncs in 1h (no restart if volume mount). `kubectl get externalsecrets -A` Ready?

Lab: run kube-bench, fix 3 fails, break image with `latest` → Kyverno deny, exec test → Falco alert.

### Chapter 28: Progressive Delivery + Karpenter - Ship 10x/Day Without Fear

**28.1 Argo Rollouts blue/green vs canary (full YAML):**
```yaml
# Canary 10% → analysis → 50% → full (for API, checkout)
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata: {name: api, namespace: prod}
spec:
  replicas: 6
  strategy:
    canary:
      canaryService: api-canary
      stableService: api-stable
      steps:
      - {setWeight: 10}
      - {pause: {duration: 5m}}
      - {analysis: {templates: [{templateName: success-rate}], args: [{name: service, value: api-canary}]}}
      - {setWeight: 50}
      - {pause: {duration: 10m}}
      - {analysis: {templates: [{templateName: latency-p95}], args: [{name: service, value: api-canary}]}}
  selector: {matchLabels: {app: api}}
  template: {metadata: {labels: {app: api}}, spec: {containers: [{name: api, image: myreg/api:v1.2.5}]}}
---
# Blue/green (for frontend, instant switch + instant rollback, needs 2x capacity 2min)
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata: {name: web, namespace: prod}
spec:
  replicas: 4
  strategy:
    blueGreen: {activeService: web-active, previewService: web-preview, autoPromotionEnabled: false, scaleDownDelaySeconds: 600}
  selector: {matchLabels: {app: web}}
  template: {metadata: {labels: {app: web}}, spec: {containers: [{name: web, image: myreg/web:v2.0.0}]}}
# Promote: kubectl argo rollouts promote web -n prod; Abort: kubectl argo rollouts abort web (instant back to blue)
```
When: canary for high-RPS API (catch 1% errors), blue/green for UI (preview QA on preview svc, then 1-click switch). AnalysisTemplate queries Prometheus (Ch21): success>99%, p95<300ms else auto-abort + rollback to stable RS. `kubectl argo rollouts get rollout api --watch` shows weights.

**28.2 Karpenter NodePool (binpack + spot + consolidation):**
```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata: {name: general}
spec:
  template:
    metadata: {labels: {team: platform}}
    spec:
      requirements:
      - {key: karpenter.sh/capacity-type, operator: In, values: [spot, on-demand]}
      - {key: kubernetes.io/arch, operator: In, values: [amd64]}
      - {key: karpenter.k8s.aws/instance-family, operator: In, values: [m6i, m7i]}
      - {key: topology.kubernetes.io/zone, operator: In, values: [a, b, c]}
      nodeClassRef: {group: karpenter.k8s.aws, kind: EC2NodeClass, name: default}
      taints: [] # spot pool separate with spot=true:NoSchedule + app toleration for batch only
  limits: {cpu: 200, memory: 400Gi}
  disruption: {consolidationPolicy: WhenEmptyOrUnderutilized, consolidateAfter: 1m, budgets: [{nodes: 10%}]}
---
apiVersion: karpenter.k8s.aws/v1
kind: EC2NodeClass
metadata: {name: default}
spec:
  amiFamily: AL2023
  role: KarpenterNodeRole-prod
  subnetSelectorTerms: [{tags: {karpenter.sh/discovery: prod}}]
  securityGroupSelectorTerms: [{tags: {karpenter.sh/discovery: prod}}]
  blockDeviceMappings: [{deviceName: /dev/xvda, ebs: {volumeSize: 50Gi, volumeType: gp3}}]
```
Behavior: Pending pod (needs 2CPU, no space) → Karpenter launches cheapest fitting (spot m6i.large $0.03 vs on-demand $0.10) in 30s (vs CAS 3min ASG). Underutilized (2 nodes 20% each) → consolidates to 1 + drains respecting PDB + `consolidateAfter`. Spot interruption (2min notice via `aws-node-termination-handler`): cordon, evict (PDB), reschedule on-demand. Separate pools: `critical (on-demand only, DB)` vs `batch (spot only, toleration spot=true)`. Watch: `kubectl get nodepools,nodeclaims`, `kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter`.

Lab: stress `replicas 20` → watch new NodeClaim, delete deploy → watch consolidation to 1 node in 2min, simulate spot `kubectl annotate nodeclaim spot-interruption`.

### Chapter 29: Autoscaling Custom Metrics + VPA + Secrets Rotation (Prod Gaps)

**29.1 HPA custom (Kafka lag, QPS) + Prometheus Adapter:**
```yaml
# adapter values: rules: [{seriesQuery: kafka_consumergroup_lag, resources: {overrides: {namespace: {resource: namespace}}}, name: {matches: kafka_lag}, metricsQuery: avg(kafka_consumergroup_lag) by (namespace)}]
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: {name: worker, namespace: prod}
spec:
  scaleTargetRef: {apiVersion: apps/v1, kind: Deployment, name: worker}
  minReplicas: 2
  maxReplicas: 30
  metrics:
  - {type: Resource, resource: {name: cpu, target: {type: Utilization, averageUtilization: 60}}}
  - {type: Pods, pods: {metric: {name: kafka_lag}, target: {type: AverageValue, averageValue: 100}}}
  behavior: {scaleDown: {stabilizationWindowSeconds: 300, policies: [{type: Percent, value: 50, periodSeconds: 60}]}, scaleUp: {stabilizationWindowSeconds: 0}}
```
Test: `kubectl get hpa worker --watch` TARGETS `150/100` → scales. Need `requests.cpu` set or CPU shows `<unknown>`. Never HPA CPU + VPA CPU together (fight: HPA adds pods, VPA raises requests → loop). Split: HPA on CPU/custom, VPA `mode: Off` for recs only.

**29.2 VPA recommender (right-size without restart):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata: {name: api-vpa, namespace: prod}
spec:
  targetRef: {apiVersion: apps/v1, kind: Deployment, name: api}
  updatePolicy: {updateMode: "Off"} # Recreate to apply: On, Initial only for new pods
  resourcePolicy: {containerPolicies: [{containerName: api, minAllowed: {cpu: 100m, memory: 128Mi}, maxAllowed: {cpu: 1000m, memory: 1Gi}}]}
# kubectl get vpa api-vpa -o jsonpath='{.status.recommendation}' → lower: {cpu: 180m} upper: {cpu: 250m} → patch requests to 200m weekly
```
Weekly job: VPA recs → PR `requests 500m→200m` saves 60% (Wolk $300k case). KEDA to zero for queue:
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata: {name: worker-keda, namespace: prod}
spec: {scaleTargetRef: {name: worker}, minReplicaCount: 0, maxReplicaCount: 30, triggers: [{type: kafka, metadata: {bootstrapServers: kafka:9092, consumerGroup: workers, topic: jobs, lagThreshold: "100"}}]}
```

**29.3 ESO + SealedSecrets (never plain Git):**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata: {name: aws, namespace: prod}
spec: {provider: {aws: {service: SecretsManager, region: us-east-1, auth: {jwt: {serviceAccountRef: {name: eso-sa}}}}}}
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata: {name: db-pass, namespace: prod}
spec: {refreshInterval: 1h, secretStoreRef: {name: aws, kind: SecretStore}, target: {name: db-secret, creationPolicy: Owner}, data: [{secretKey: password, remoteRef: {key: prod/db, property: password}}]}
# Rotation: AWS Lambda rotates prod/db every 30d → ESO syncs in 1h → app with volume mount reloads without restart (envFrom needs rollout restart)
# SealedSecrets alt (no AWS): kubeseal --format yaml < secret.yaml > sealed.yaml (only cluster controller can decrypt with private key). Git stores sealed.yaml safe.
```
Check: `kubectl get externalsecrets db-pass -n prod` Ready True? `kubectl get secret db-secret -o jsonpath`. IRSA role `eso-sa → SecretsManagerRead` only (least-privilege). Audit `get secrets` in apiserver audit log → Loki.

Lab: break HPA (remove requests → unknown), fix; VPA Off recs → patch; rotate secret → watch sync.

### Chapter 30: Topology, Descheduler, Ephemeral Storage, Image GC - The 20% That Causes 80% Prod Pain

**30.1 topologySpreadConstraints deep (fix 3-0-0 skew):** Without spread, scheduler binpacks 3 replicas all in zone-a (cheapest scoring) → zone-a outage kills 100%. With:
```yaml
topologySpreadConstraints:
- {maxSkew: 1, topologyKey: topology.kubernetes.io/zone, whenUnsatisfiable: DoNotSchedule, labelSelector: {matchLabels: {app: api}}}
- {maxSkew: 1, topologyKey: kubernetes.io/hostname, whenUnsatisfiable: ScheduleAnyway, labelSelector: {matchLabels: {app: api}}}
```
maxSkew 1 = zones differ by max 1 (1-1-1 ok, 2-1-0 Pending until new zone capacity). `DoNotSchedule` strict (Pending if can't spread - safe, needs capacity), `ScheduleAnyway` soft (best-effort). Two constraints ANDed: first zone-strict, second host-best-effort. Check: `kubectl get pods -o wide --sort-by=.spec.nodeName`, `kubectl describe pod | grep Topology`. Common trap: single-replica Deploy with DoNotSchedule zone spread → always Pending after first (can't spread 1 across 3 with skew 1? Actually 1-0-0 skew 1 ok, but scale 2 → 1-1-0 ok, 4th → 2-1-1 ok). For daemon-like singletons use `ScheduleAnyway`.

**30.2 Descheduler (fix drift after nodes added):** Scheduler places once, never rebalances. Add node week later → old pods stay skewed 10-0. Descheduler (Job/CronJob every 15min) evicts violating pods (respects PDB, priority) → scheduler replaces balanced.
```yaml
# descheduler-policy.yaml
apiVersion: descheduler/v1alpha2
kind: DeschedulerPolicy
profiles: [{name: default, pluginConfig: [{name: RemoveDuplicates, args: {}}, {name: LowNodeUtilization, args: {thresholds: {cpu: 20, memory: 20}, targetThresholds: {cpu: 50, memory: 50}}}], plugins: {enabled: [RemoveDuplicates, LowNodeUtilization, RemovePodsViolatingTopologySpreadConstraint]}}]
# helm install descheduler descheduler/descheduler -n kube-system --set schedule="*/15 * * * *"
```
Policies: RemoveDuplicates (2 same app same node → evict one), LowNodeUtilization (node <20% while another >50% → consolidate), TopologySpreadViolation, FailedPods, NodeAffinity. Dry-run first (`--dry-run`), watch `kubectl get events --field-selector reason=Descheduled`. Staff: enable in prod with PDBs or drain storms.

**30.3 Ephemeral storage (the silent killer):** Containers + emptyDir + logs share node `/var/lib/kubelet` (default 10-30GB). No `ephemeral-storage` limit → one pod writes 20GB logs → DiskPressure → kubelet evicts BestEffort then all (even Guaranteed if over). Set:
```yaml
resources: {requests: {ephemeral-storage: 500Mi}, limits: {ephemeral-storage: 2Gi}}
volumeMounts: [{name: logs, mountPath: /var/log/app}]
# + logrotate sidecar or stdout only (never file), emptyDir sizeLimit: {name: tmp, emptyDir: {sizeLimit: 1Gi}}
```
Monitor: `kubectl describe node | grep -A3 ephemeral`, `kubelet_volume_stats_available_bytes`. Evicted message `The node was low on resource: ephemeral-storage`. Fix: `du -sh /var/lib/docker`, `crictl rmi --prune`, add `imageGCHighThresholdPercent: 80` in kubelet config.

**30.4 Image GC + kubelet config tuning (prod kubelet):**
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
imageGCHighThresholdPercent: 80 # start GC at 80% disk
imageGCLowThresholdPercent: 70 # stop at 70%
imageMinimumGCAge: 2m
evictionHard: {memory.available: 500Mi, nodefs.available: 10%, imagefs.available: 15%}
evictionSoft: {memory.available: 1Gi} # + grace 2m
maxPods: 110 # ENI limit on AWS (t3.medium 17 pods max!) - check aws-vpc-cni limits
serializeImagePulls: false # parallel pulls faster (default true slow)
registryPullQPS: 10
```
AWS ENI trap: t3.micro 4 ENIs × IPs = 17 pods max, 18th Pending `Too many pods` even with CPU free → use bigger nodes or prefix delegation (`ENABLE_PREFIX_DELEGATION=true` → 110 pods). `kubectl get node -o jsonpath='{.status.allocatable.pods}'`.

Lab: fill disk `dd if=/dev/zero of=/tmp/big bs=1G count=15` → watch Pressure → eviction → fix limits + GC.

### Chapter 31: 5 Production Postmortems - How Staff Debugs at 2AM (Full Timelines)

**PM1 - DNS 5s outage (ndots + CoreDNS overload, fintech):** Symptom 14:02 checkout p99 5.2s (was 200ms), CoreDNS CPU 95%, 40% SERVFAIL. Timeline: 14:03 `kubectl top pods -n kube-system` coredns throttled (limit 100m!). 14:05 `kubectl logs -n kube-system -l k8s-app=kube-dns --tail 100` shows `i/o timeout google.com`. Root: Java app `http://payments` (2 dots) → ndots:5 tries 4 local search first (4 queries × 2s timeout) + external-dns `forward . /etc/resolv.conf` slow upstream + CPU throttle drops. Fix 14:10 raise CoreDNS `replicas 2→6 + HPA + limits 500m`, app uses FQDN `payments.prod.svc.cluster.local` + `ndots:2`, add NodeLocal DNSCache DaemonSet (`169.254.20.10`). Prevent: load test DNS 10k QPS, monitor `coredns_dns_request_duration_seconds`, deny `ndots>2` via Kyverno. MTTR 18min → 0 after.

**PM2 - Rolling update takes down 50% (maxUnavailable + no readiness, SaaS):** Deploy `replicas:2 maxUnavailable:1` image v2 bad (CrashLoop). 10:00 v2 pod 1/2 CrashLoop, v1 killed per surge (only 1 left), v2 never Ready → 1/2 Available, then liveness kills last → 0. `kubectl rollout status` hangs progressDeadline 600s. Fix: `kubectl rollout undo`, scale `replicas 2→6`, `maxUnavailable:0`, add `readinessProbe /ready + minReadySeconds:10 + PDB minAvailable:4`. Prevent: CI `kube-linter` denies `replicas<3 + maxUnavailable>0` for prod, canary 10% analysis (Ch28) would auto-abort. Lesson: replica 2 is never HA (quorum needs 3).

**PM3 - EBS AZ mismatch Pending 45min (e-commerce):** New zone-c nodes added, StatefulSet pg-1 rescheduled zone-c but PVC `data-pg-1` zone-a (EBS zonal). `describe` `Volume zone mismatch: volume zone-a, node zone-c`. Fix: delete pod (still Pending), snapshot `aws ec2 create-snapshot`, create volume zone-c, new PV, rebind (30min). Better: use `allowedTopologies: [a,b]` + `topologySpread zone` + EBS CSI `volumeBindingMode: WaitForFirstConsumer` (delays provisioning until scheduled zone known) + Velero snapshots hourly. Long-term: Aurora/RDS Global (cross-AZ) not EBS for DB. Monitor `kube_persistentvolumeclaim_status_phase{phase="Pending"}>5m` alert.

**PM4 - OOM cascade + HPA fight (AI inference):** GPU api `limits memory 512Mi` but model needs 2Gi on load → OOM 137 under QPS spike, HPA sees CPU 90% adds pods (each OOMs), 6→20 all CrashLoop, GPU waste $60/hr. `kubectl get pods -w` CrashLoop, `describe` LastState OOMKilled, `dmesg` on node. Fix: raise `limits 4Gi + requests 2Gi` via VPA recs, add `startupProbe 60s` (model load 40s, liveness was killing early), HPA on `qps/custom` not CPU (CPU low during IO wait), KEDA `lag>50` + `min 2`. Prevent: load test with `k6 100 VUs`, set `memory limit = p99 load*1.5`.

**PM5 - Spot mass eviction + PDB block (Black Friday):** Karpenter spot 80% to save cost, AWS reclaim 10 nodes at once (spot price spike), PDB `minAvailable:5` with 6 replicas blocks drain → `drain` hangs, new on-demand nodes take 3min, 2min gap 50% 503. Fix: separate pools `critical on-demand (checkout, PDB strict)` vs `batch spot (workers, PDB loose)`, `consolidation` pause during peak (`karpenter.sh/do-not-disrupt: true` annotation Nov), `terminationGrace 120s + preStop sleep 90s` to drain, overprovision `pause pods` (2 idle 1CPU holds capacity, evicted first). Monitor `karpenter_interruption_events_total` + spot price. Lesson: spot 70% is for stateless batch, never for checkout path on peak.

Template for your runbook: Title, Impact (RPS/errors/minutes), Timeline (5 lines), Root (5 whys), Fix now, Prevent (policy/monitor/test), Action owners + dates. Staff writes this in 30min post-incident, shares org-wide.

### Chapter 32: CKA Part 2 + CKAD/CKS + 50 More Answers (Pass Trifecta)

**CKA Mock 2 (hard, 90min):** 1. Create `PriorityClass high (1000000)` + pod preempts low (show `kubectl describe` NominatedNode). 2. `ResourceQuota prod: pods 20 cpu 10` + `LimitRange default 100m/500m` → create pod no requests (gets defaults) + 21st pod Forbidden. 3. `StorageClass fast (ebs gp3, allowExpansion, Retain)` + PVC expand 5→10Gi online (`kubectl patch pvc`). 4. `PodSecurity enforce restricted` on ns `secure` → run privileged pod denied, fix non-root. 5. `Ingress canary weight 10` (nginx annotation `canary-weight: 10`) + header `X-Canary: always`. 6. `Service headless + StatefulSet 3` → `nslookup web-0.db` returns pod IP. 7. `topologySpread zone maxSkew 1` → `kubectl get pods -o wide` 1-1-1. 8. `descheduler` dry-run evicts duplicate. 9. `etcd defrag + snapshot + restore to /tmp` (commands Ch22). 10. Debug `kubelet NotReady cert expired` (`openssl x509 -dates`, `kubeadm kubeconfig user --client-name system:nodes`).

**CKAD (dev focus, 2h):** `ConfigMap volume subPath` (single file mount without hiding dir), `Secret env + volume`, `Job completions 5 parallelism 2 + CronJob every 2min`, `multi-container sidecar share emptyDir`, `Probes + resource`, `NetworkPolicy allow`, `ServiceAccount + RBAC`, `Helm values override`, `Dockerfile multistage distroless`. Tip: `k run --image --env --port --dry-run -o yaml` fastest.

**CKS (security, 2h + CKA required):** `Falco rule detect shell`, `AppArmor seccomp RuntimeDefault`, `ImagePolicyWebhook verify signature`, `NetworkPolicy default-deny + egress DNS`, `CIS kube-bench fix 3`, `Secrets encryption KMS Envelope`, `RuntimeClass gVisor`, `Audit policy log exec`. Practice: `falco -o json`, `kubectl --as=system:serviceaccount:evil get secrets -n prod` denied.

**50 more 30s answers (51-100):** 51 PDB minAvailable vs maxUnavailable? 52 topology maxSkew 1 means? 53 descheduler when? 54 ephemeral-storage evict? 55 ENI pod limit t3.micro? 56 imageGC thresholds? 57 preemption needs? 58 LimitRange injects? 59 Quota vs Limit? 60 WLC headless DNS? 61 partition rolling? 62 OnDelete when? 63 spot toleration? 64 consolidation? 65 canary vs blue/green when? 66 AnalysisTemplate query? 67 Argo selfHeal vs manual prod? 68 Sealed vs ESO? 69 IRSA how? 70 Kyverno Enforce vs Audit? 71 Cosign verify? 72 Trivy fail? 73 Falco detects? 74 audit log exec? 75 seccomp default? 76 runAsNonRoot 1000? 77 readOnly FS + tmp emptyDir? 78 distroless why? 79 SBOM syft? 80 OTel trace_id logs? 81 Prometheus ServiceMonitor? 82 Loki query `{app} | json`? 83 Tempo traceID? 84 burn-rate 5% 1h? 85 HPA stabilization 300s? 86 KEDA lag 100? 87 VPA Off recs? 88 Karpenter spot reclaim 2min? 89 NodeLocal DNS 169.254.20.10? 90 FQDN dot? 91 EBS WaitForFirstConsumer? 92 Retain Released manual? 93 RWX needs EFS? 94 snapshot restore test monthly? 95 Velero TTL 720h? 96 cert-manager renew 30d? 97 external-dns TXT heritage? 98 nip.io kind? 99 k9s :pods? 100 teach junior (Staff multiplies)?

If 90/100 in <60s each + 2 mocks timed pass, book CKA/CKAD/CKS back-to-back. You now own Senior + exam + prod.

### Chapter 33: Service Mesh Deep - Istio Ambient vs Sidecar vs Cilium (Full YAML)

**When mesh?** <20 svcs → Ingress + OTel enough. 50+ with mTLS/audit/retries/canary per-route → mesh. Cost: sidecar 100m/128Mi + 5ms p99 + operate istiod. Ambient (Istio 1.22+, no sidecar, ztunnel eBPF per node + waypoint Envoy per ns) cuts 70% overhead - Staff pick 2026 with Cilium.

**Ambient install + mTLS STRICT + retry:**
```bash
istioctl install --set profile=ambient --skip-confirmation
kubectl label ns prod istio.io/dataplane-mode=ambient
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata: {name: default, namespace: prod}
spec: {mtls: {mode: STRICT}}
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata: {name: api, namespace: prod}
spec:
  hosts: [api.prod.svc.cluster.local]
  http:
  - match: [{headers: {x-canary: {exact: "true"}}}]
    route: [{destination: {host: api, subset: v2}, weight: 100}]
  - route: [{destination: {host: api, subset: v1}, weight: 90}, {destination: {host: api, subset: v2}, weight: 10}]
    retries: {attempts: 3, perTryTimeout: 2s, retryOn: 5xx}
    timeout: 10s
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata: {name: api, namespace: prod}
spec:
  host: api
  subsets: [{name: v1, labels: {version: v1}}, {name: v2, labels: {version: v2}}]
  trafficPolicy: {connectionPool: {tcp: {maxConnections: 100}, http: {h2UpgradePolicy: UPGRADE}}, outlierDetection: {consecutive5xxErrors: 5, interval: 30s, baseEjectionTime: 30s}}
EOF
kubectl get pods -n prod -o jsonpath='{.items[*].metadata.annotations}' | grep ambient
istioctl analyze -n prod; istioctl proxy-status
```
Debug: `istioctl pc cluster deploy/api -n prod`, `istioctl authn tls-check`. Sidecar mode (old): `kubectl label ns prod istio-injection=enabled` injects Envoy per pod (`istio-proxy` 100m). Cilium alt (no istiod): `cilium install --set encryption.enabled=true` + `CiliumNetworkPolicy L7 http method GET path /api` + Hubble `hubble observe --pod api`. Choose: Linkerd (light Rust, 10m CPU) for small strict mTLS, Istio ambient for large L7, Cilium for perf + observe.

Lab: break mTLS (PERMISSIVE vs STRICT 503), fix; canary header → v2; retry 3x with `fortio` fault 500.

### Chapter 34: Multi-Tenancy - vCluster + Capsule + Tenant Isolation (Platform Team)

**Models:** Namespace per team (cheap, shared control-plane, noisy) vs Cluster per team ($$$, blast radius) vs vCluster (virtual clusters per team on shared host, own API + etcd sqlite, 10x cheaper). Staff: start ns + RBAC + Quota, move to vCluster when teams need CRDs/admin (AI team installs own operators without breaking others).

**Capsule (ns governance) + vCluster:**
```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata: {name: checkout}
spec:
  owners: [{name: alice, kind: User}]
  namespaces: [checkout-dev, checkout-prod]
  namespaceOptions: {quota: 20, additionalMetadata: {labels: {team: checkout}}}
  resourceQuotas: [{hard: {pods: "50", requests.cpu: "20", requests.memory: 40Gi}}]
  limitRanges: {items: [{limits: [{default: {cpu: 500m, memory: 512Mi}, defaultRequest: {cpu: 100m}}]}]}
  networkPolicies: {items: [{spec: {podSelector: {}, policyTypes: [Ingress], ingress: [{from: [{namespaceSelector: {matchLabels: {team: checkout}}}]}]}}]}
  roles: [admin]
---
# vCluster per tenant (host: kind-mastery, tenant gets own kubeconfig)
# helm install team-a vcluster/vcluster -n team-a-vc --create-namespace --set sync.nodes.enabled=true
# vcluster connect team-a -n team-a-vc -- kubeconfig team-a.yaml; KUBECONFIG=team-a.yaml kubectl create ns app; kubectl apply -f deploy.yaml (isolated API, syncs pods to host as vcluster pods)
```
Isolation: RBAC Tenant owner admin only its ns, NetworkPolicy deny cross-team (except gateway), ResourceQuota prevents hog, PodSecurity restricted, ESO per-tenant SecretStore (no cross-read), `kube-ops-view` read-only. Chargeback label `tenant` → Kubecost. Upgrade: host once, vClusters rolling.

Lab: create Tenant, try cross-ns `kubectl auth can-i get pods -n other` denied, over-quota 51st pod Forbidden, vCluster deploy isolated.

### Chapter 35: FinOps - Kubecost + Chargeback + 10 Levers to Cut 50% (With Numbers)

Install: `helm install kubecost kubecost/cost-analyzer -n kubecost --create-namespace --set kubecostToken=<free> --set prometheus.server.persistentVolume.size=32Gi`. UI `kubectl port-forward svc/kubecost-cost-analyzer 9090`. Allocation: by ns/team/app (`kubecostAllocation{namespace="prod"}`), idle vs used, spot vs on-demand. Alert: `weekly cost > $5000` Slack.

10 levers (shop $10k/mo → $5k):
1. Right-size via VPA recs (500m→200m, 40% pods) saves $2k. `kubectl get vpa -o jsonpath`.
2. HPA + KEDA to zero dev nights (suspend 20-8, 60h/week off) saves $1.5k. `kube-ns-suspender`.
3. Spot 70% batch (Karpenter spot pool, on-demand checkout) saves $1.2k. Monitor interruptions.
4. 20 LBs ($20 each) → 2 ALBs + Ingress saves $360. `kubectl get svc -A | grep LoadBalancer`.
5. EBS gp2→gp3 + delete Released (200Gi orphan $20) + snapshots lifecycle 30d saves $200.
6. EFS Infrequent + S3 lifecycle logs 90d → Glacier saves $150.
7. Image pull 10GB/mo cross-AZ ($0.01/GB) → ECR same AZ + image cache DaemonSet saves $100.
8. Logs 500GB Loki → 30d retention + S3 saves $120.
9. Overprovision pause pods (hold 2CPU idle, evicted first) vs 2 extra nodes $200 - pause cheaper.
10. Commitment (Savings Plan 1yr 30% for baseline DB/on-demand) saves $800.

Showback: `kubecost -l team=checkout $1200 (idle $200)`, `namespace prod $3000`, weekly email to owners. Gate: PR label `cost-impact` if requests +2CPU, Kyverno deny `requests.cpu>4` without FinOps approve.

Lab: `kubecost` port-forward, find top ns, patch requests per VPA, delete orphan PVs, switch 1 LB→ClusterIP+Ingress, measure delta.

### Chapter 36: Hardening Part 2 - OPA Gatekeeper + Falco Library + Audit Pipeline

**Gatekeeper (deny without code):**
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata: {name: k8srequiredlabels}
spec:
  crd: {spec: {names: {kind: K8sRequiredLabels}, validation: {openAPIV3Schema: {properties: {labels: {type: array, items: {type: string}}}}}}}
  targets: [{target: admission.k8s.gatekeeper.sh, rego: |
    package k8srequiredlabels
    violation[{"msg": msg}] { not input.review.object.metadata.labels.team; msg := "Need label team" }|}]}
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata: {name: need-team}
spec: {match: {kinds: [{apiGroups: [""], kinds: [Pod]}]}, parameters: {labels: [team]}}
# Test: kubectl apply --dry-run=server -f no-label-pod.yaml → denied "Need label team". Audit mode: enforcementAction: dryrun to measure before enforce.
```

**Falco 10 rules (runtime threat):**
```yaml
# falco-values.yaml: falco.rules: |
# - rule: Shell in prod, desc: sh spawned, condition: evt.type=execve and proc.name in (sh,bash) and container.image contains "prod", output: "Shell in %container.name", priority: WARNING
# - rule: Sensitive file read, condition: open_read and fd.name in (/etc/shadow,/etc/secrets), priority: CRITICAL
# - rule: K8s secret get, condition: k8s_audit verb=get and resource=secrets, priority: NOTICE
# - rule: Outbound C2, condition: fd.sip != 10.0.0.0/8 and proc.name=api, priority: CRITICAL
# Deploy: helm install falco falcosecurity/falco -n falco --create-namespace -f falco-values.yaml; logs: kubectl logs -n falco -l app=falco | grep CRITICAL; alert to Slack via falcosidekick.
```

**Audit pipeline (SOC2 1yr):** apiserver `--audit-policy-file=/etc/kubernetes/audit.yaml --audit-log-path=/var/log/apiserver/audit.log --audit-log-maxage=30` with policy log `pods/exec + secrets get + bindings` at Metadata level, rest Omit. Fluent-Bit DaemonSet tails `/var/log/apiserver/audit.log` → Loki/S3 (`s3://audit-1yr`). Alert `rate(apiserver_audit_events{verb="get",resource="secrets"}[5m])>10` page. Verify: `kubectl exec -n prod api -- sh` → audit shows `user:alice verb:create resource:pods/exec`.

### Chapter 37: Data on Kubernetes - Postgres + Kafka + Redis Full YAML + Backup Drills

**CloudNativePG (prod Postgres, not raw STS):**
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata: {name: pg, namespace: data}
spec:
  instances: 3
  imageName: ghcr.io/cloudnative-pg/postgresql:15.4
  storageConfiguration: {storageClass: gp3, size: 20Gi}
  affinity: {podAntiAffinity: {requiredDuringSchedulingIgnoredDuringExecution: [{labelSelector: {matchLabels: {cnpg.io/cluster: pg}}, topologyKey: kubernetes.io/hostname}]}}
  backup:
    barmanObjectStore: {destinationPath: s3://shop-pg-backup, s3Credentials: {inheritFromIAMRole: true}, wal: {compression: gzip}}
    retentionPolicy: "30d"
  monitoring: {enablePodMonitor: true}
# Backup: kubectl cnpg backup pg -n data --method barmanObjectStore; Restore drill monthly: kubectl cnpg restore pg-restore --source pg --recovery-target-time "2026-10-01 02:00" -n data-restore; psql verify row count.
```

**Strimzi Kafka (3 brokers + topics):**
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata: {name: kafka, namespace: data}
spec:
  kafka: {replicas: 3, version: 3.6.0, resources: {requests: {cpu: 1, memory: 2Gi}}, storage: {type: persistent-claim, size: 50Gi, class: gp3}, config: {offsets.topic.replication.factor: 3, min.insync.replicas: 2}}
  zookeeper: {replicas: 3, storage: {type: persistent-claim, size: 10Gi, class: gp3}}
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata: {name: jobs, namespace: data, labels: {strimzi.io/cluster: kafka}}
spec: {partitions: 12, replicas: 3, config: {retention.ms: 604800000}}
# Produce/consume test: kubectl run kafka-test --image=quay.io/strimzi/kafka -- bin/kafka-console-producer.sh --bootstrap-server kafka-kafka-bootstrap:9092 --topic jobs
```

**Redis cluster (cache, evict allkeys-lru):**
```yaml
# helm install redis bitnami/redis-cluster -n data --set cluster.nodes=6 --set persistence.storageClass=gp3 --set persistence.size=10Gi --set redis.config.maxmemory-policy=allkeys-lru
# Test: kubectl exec -it redis-cluster-0 -- redis-cli -c set foo bar; kill pod 0 → still get foo (failover). Backup RDB hourly to S3 via CronJob Velero.
```
Rules: DB in K8s only with operator + `Retain + cross-AZ + PodMonitor + backup tested`. Else RDS/Managed. `kubectl get pods -n data -o wide` must show 1-1-1 zone spread.

### Chapter 38: Day-2 Ops - Upgrade Live + etcd Disaster + Chaos Game-Days

**38.1 Upgrade 1.29→1.30 live (EKS/kubeadm, zero downtime):** Pre: `kubeadm upgrade plan` (etcd backup Ch22 + Velero + `kubectl get pdb -A` all healthy + CSI/CNI support 1.30). Steps: drain AZ-a one node `kubectl drain n1 --ignore-daemonsets`, upgrade kubeadm `apt install kubeadm=1.30.x`, `kubeadm upgrade node`, `kubelet 1.30 + kubectl`, `uncordon`, watch `kubectl get nodes` version + `kubectl get pods -A` Running. Repeat AZ-b/c. Control-plane first (kubeadm) then nodes. EKS: `eksctl upgrade cluster --version 1.30` + MNG rolling (`maxUnavailable 1`) + Karpenter drift. Rollback: if `CrashLoop` new CSI → `kubeadm upgrade` undo via snapshot restore `/tmp/s.db` + old AMI. Never skip 2 minors (1.28→1.30 unsupported → do 1.29 first).

**38.2 etcd disaster restore drill (quarterly, RTO 45min like Palo Alto 48h→45m):** Kill: `rm -rf /var/lib/etcd/member` on leader (simulate AZ loss). Detect: `etcdctl endpoint health` 1 unhealthy, apiserver 500s but pods Running. Restore: stop kubelet `systemctl stop kubelet`, `etcdctl snapshot restore /backup/etcd-latest.db --data-dir /var/lib/etcd-restored --initial-cluster m1=http://10.0.1.10:2380,m2=http://10.0.2.10:2380 --initial-advertise-peer-urls`, `mv restored member`, start, `etcdctl endpoint health` 3 healthy, `kubectl get deployments -A` rev intact. Practice on kind: `docker exec mastery-control-plane rm` + restore. Measure RTO, doc runbook, assign owner.

**38.3 Chaos with Litmus (prove HA, not hope):** `helm install litmus litmuschaos/litmus -n litmus --create-namespace`. Experiments: `pod-delete` (kill 1/6 api → HPA + PDB keeps 5, SLO holds), `node-drain` (kill zone-a node → spread 1-1-1 rebalances), `network-loss 50% 5m` (Cilium retry 3x hides), `disk-fill 80%` (ephemeral limit evicts BestEffort first). `kubectl apply -f chaos-pod-delete.yaml --watch`; steady-state check Prometheus `success>99%` else abort. Game-day monthly: kill AZ, verify checkout 99.9% holds, write PM (Ch31 template). No chaos without PDB + HPA + rollback plan.

### Chapter 39: Dev Platform - Backstage + AppSets for 100 Services + Preview Envs

**Backstage (developer portal, no YAML):** `npx @backstage/create-app`, catalog `catalog-info.yaml {kind: Component, name: api, owner: checkout, system: shop}` + `scaffolder template` (new svc: pick Java/Go → generates Helm + Argo App + CI + SLO). Scorecards: `requires: {probes, limits, PDB, NetworkPolicy, no latest}` (TechInsights fail → block promote). Search `api` shows owner, docs, Grafana, Argo health.

**AppSets for 100 svcs (one YAML, not 100):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata: {name: shop, namespace: argocd}
spec:
  generators: [{git: {repoURL: https://github.com/org/shop.git, revision: main, directories: [{path: services/*}]}}]
  template:
    metadata: {name: '{{path.basename}}-prod'}
    spec: {project: default, source: {repoURL: https://github.com/org/shop.git, targetRevision: main, path: '{{path}}/overlays/prod'}, destination: {server: https://kubernetes.default.svc, namespace: prod}, syncPolicy: {automated: {prune: true}}}
# Add service: git mkdir services/new-api/{base,overlays/prod} + push → Argo auto-creates App. Update: bump image tag in Git → sync canary (Ch28).
```

**Preview envs controller (E-Pods 200/quarter):** PR `123` → GitHub Action `kustomize build overlays/pr | kubectl apply -n pr-123` + `ttl.sh/48h` annotation (janitor deletes) + ESO branch DB + KEDA to zero after idle + `nip.io` host `pr-123.1-2-3-4.nip.io`. Merge → delete ns + PVC. Cost $12 vs $80 (Ch20). `kubectl get ns -l preview=true`, `kubecost -l pr=123`.

### Chapter 40: AI Batch - Training + Ray + Volcano + GPU Sharing (K8s for ML)

**PyTorchJob distributed (4 GPUs, checkpoint S3):**
```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata: {name: llama-train, namespace: ml}
spec:
  pytorchReplicaSpecs:
    Master: {replicas: 1, template: {spec: {containers: [{name: torch, image: pytorch:2.2-cuda12, command: ["torchrun","--nproc_per_node=4","train.py","--ckpt s3://models/ckpt"], resources: {limits: {nvidia.com/gpu: 1, cpu: 8, memory: 32Gi}}, volumeMounts: [{name: data, mountPath: /data}]}]}}}
    Worker: {replicas: 3, template: {spec: {containers: [{name: torch, image: pytorch:2.2-cuda12, resources: {limits: {nvidia.com/gpu: 1}}}], tolerations: [{key: nvidia.com/gpu, operator: Exists, effect: NoSchedule}]}}}
# Volcano queue fair-share: schedulerName: volcano, queue: ml-team (cap 8 GPUs), preempt low-priority batch if serving needs.
```

**Ray Serve inference (autoscale 0→20) + GPU sharing:**
```yaml
# helm install kuberay kuberay/kuberay-operator -n ray
# RayService llama: replicas 0-20, KEDA prometheus qps>10, model EFS RWX shared, init download once per node
# Sharing: MIG (A100 split 7x10GB: --mig-profile 1g.10gb) or time-slicing (4 pods share 1 T4: nvidia.com/gpu: 1, replicas 4, scheduler interleaves 10ms) for dev. Prod serving gets whole GPU (latency), batch shares.
```
Monitor: `dcgm-exporter` GPU util + `kubectl top pods -n ml`, cost `kubecost -l team=ml` ($3/hr A100 → spot $1/hr batch). Checkpoint every 30min to S3 (spot reclaim safe).

You now own full platform: secure + data + day-2 + dev + AI. Teach one junior end-to-end - that's Staff.


---

## Appendix A: Glossary - 40 Terms in 30 Seconds (Teach-Ready)
Cluster (1 control-plane + N nodes), Node (VM registered by kubelet), Pod (1 IP, shared netns via pause, ≥1 container), pause (infra holder), initContainer (run-first, must succeed), ephemeralContainer (debug attach), ReplicaSet (keep N, selector vs template), Deployment (RS manager + rollout maxSurge/maxUnavailable + revision), StatefulSet (ordinal web-0 + per-pod PVC + headless + ordered), DaemonSet (1/node + tolerates masters), Job (completions/parallelism/backoff) / CronJob (schedule/concurrency/history), Namespace (logical isolation), Label (glue) / Selector (match) / Annotation (tool meta), Service (stable VIP) / EndpointSlice (pod IPs) / headless (None), Ingress (L7 + controller) / Gateway API (Gateway+HTTPRoute), CNI (Flannel overlay / Calico BGP+policy / Cilium eBPF+L7) / CSI (EBS/EFS driver) / CoreDNS (cluster.local + ndots:5), PV (disk) / PVC (claim) / SC (provisioner gp3) / volumeClaimTemplate (per-replica factory), Taint (repel) / Toleration (allow) / Affinity (attract: node/pod) / TopologySpread (even AZ), QoS (Guaranteed>Burstable>BestEffort) / OOMKilled 137, Probe (liveness restart vs readiness remove vs startup gate), RBAC (Role/ClusterRole+Binding, least-privilege) / SA (pod identity, IRSA) / NetworkPolicy (default-deny + allow DNS/app) / PSS (restricted), HPA (pods CPU/custom) / VPA (resize) / KEDA (event to zero) / Karpenter (nodes spot) / PDB (minAvailable survives drain), Helm (chart+release+values) / Kustomize (base+overlay) / GitOps ArgoCD (Git truth + sync + canary).

## Appendix B: 6-Week Lab Plan (1-2h/day, kind on Windows)
W1: kind install, arch, Pod YAML, pause/crictl, Unknown drill. W2: Deploy rollout/rollback, maxSurge 0 test, STS web + PVC, DaemonSet node-exporter, Job pi + CronJob. W3: ClusterIP/NodePort, break selector → empty Slice → fix, Ingress nginx + TLS, CoreDNS ndots test, Calico deny-all. W4: ConfigMap volume reload, Secret ESO, probes break/fix, requests/limits OOM, affinity/taint schedule, PVC expand. W5: RBAC dev Role, NetworkPolicy, Helm chart install/rollback, HPA stress + VPA recs, Prometheus + burn alert. W6: ArgoCD sync + canary, Velero backup/restore, 5 runbooks timed, CKA mocks (etcd snapshot, upgrade, drain), interview whiteboard + teach junior.

## Appendix C: Sources & Further (researched Oct 2026)
- kubernetes.io/docs/concepts/architecture/, services-networking/service/, security/, storage/, workloads/ (primary truth, CC BY 4.0 images as credited)
- Red Hat: Introduction to Kubernetes architecture (CNI/CSI/DNS explainer)
- Manning: Kubernetes in Action 2e (Lukša/Conner) - deployment bible
- O'Reilly: The Kubernetes Book 2e, Mastering Kubernetes 4e, Cracking Kubernetes Interview
- Wolk 2026 Production Best Practices (5-area audit), Standout 2026 hiring filters (Senior cost-controller, incident-tested), DevOpsSchool Senior→Staff blueprint (6-10yrs, 3-5yrs K8s prod)
- Search performed: 3 queries x 8 results (ebook structure, senior expectations, best K8s books). Full books not scraped - synthesis + docs + production rubrics.
Images: kubernetes.io CC BY 4.0 as credited. No AI-generated diagrams per request. For print: `pandoc md -o pdf --pdf-engine=xelatex`.

## Appendix D: Cheat-Sheet - kubectl + Helm + Argo (One Page, Memorize for CKA)

```powershell
# kubectl core (alias k=kubectl, do="--dry-run=client -o yaml")
k get pods -A -o wide --sort-by=.metadata.creationTimestamp; k describe pod X; k logs X -c app --previous --tail=100 -f
k exec -it X -- sh -c "env | sort; cat /etc/resolv.conf; nc -zv db 5432"
k run web --image=nginx:1.26 $do > p.yaml; k create deploy d --image=nginx --replicas=3 $do
k expose deploy d --port=80 --target-port=8080 --type=NodePort; k expose deploy d --port=80 --type=LoadBalancer
k set image deploy/d nginx=nginx:1.26 --record; k rollout status/history/undo deploy/d
k create secret generic s --from-literal=k=v --from-file=f.txt; k create cm c --from-file=app.conf; k get secret s -o jsonpath='{.data.k}' | base64 -d
k auth can-i create deployments --as=system:serviceaccount:dev:bot -n dev; k config use-context kind-mastery; k config set-context --current --namespace=prod
k top nodes/pods; k get events --sort-by=.lastTimestamp; k get endpointslices -l kubernetes.io/service-name=X
k cordon/drain node1 --ignore-daemonsets --delete-emptydir-data; k uncordon node1; k delete pod X --grace-period=0 --force
k apply -f .; k diff -f .; k patch deploy d -p '{"spec":{"replicas":5}}'; k wait --for=condition=Ready pod/web --timeout=60s
k get pv,pvc,sc,ingress,netpol,psp,pdb,hpa,vpa,crd,csidrivers; k explain pod.spec.containers --recursive | head -40
# Helm
helm repo add bitnami https://charts.bitnami.com/bitnami; helm install redis bitnami/redis -n data --create-namespace --set auth.password=x --atomic --timeout 5m
helm list -A; helm history web -n prod; helm rollback web 2 -n prod; helm template ./chart -f values-prod.yaml | kube-lint
# Argo
argocd app list; argocd app diff api-prod; argocd app sync api-prod --prune; kubectl argo rollouts get rollout api -n prod --watch; kubectl argo rollouts promote/abort api -n prod
# etcd/kubeadm (SSH to control-plane)
ETCDCTL_API=3 etcdctl snapshot save /tmp/s.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key; etcdctl endpoint health; kubeadm upgrade plan; journalctl -u kubelet -f
```

## EPILOGUE - Ankit Gets the Offer (Your Last Page)

![Chai cheers - offer letter day at FilterKaapi](https://unsplash.com/photos/R85QslGRrrs/download?w=900)
*Story: pouring chai for the whole team. Photo by Karthick Gislen on Unsplash. Free for commercial use.*

Three years after that first Diwali outage, Ankit sits in a Staff interview. They ask "design checkout for 10k RPS." Ankit smiles, draws ShopKart on the board: Mani at the door, Ellamma sisters with the ledger, Saravanan assigning rooms, Kumar running floors, Pavithra connecting calls. The panel laughs, then hires. That night Priya calls from FilterKaapi, "teach me?" Ankit opens Chapter 2. Story continues.

*End - Now teach Chapter 2 to someone. That's mastery. Next: pick any chapter and run its lab without notes.*



