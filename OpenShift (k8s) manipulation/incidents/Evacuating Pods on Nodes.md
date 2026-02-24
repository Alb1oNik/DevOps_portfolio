# OpenShift — Evacuating Pods on Nodes (Node Drain)

## Concept Overview

**Node draining** is the process of safely evicting all workloads from a node before performing maintenance, upgrades, or decommissioning. In OpenShift (and Kubernetes), this is done using the `oc adm drain` command.

When a node is drained:
1. It is marked as **unschedulable** (`cordon`) — no new pods will be scheduled on it
2. All existing pods are **evicted** — gracefully terminated and rescheduled on other nodes

---

**Real-world problem (this case):**
- A node update was triggered (e.g., MachineConfig change)
- Some pods were **not managed by any controller** (bare pods) or were part of a **DaemonSet**
- These pods blocked the drain process
- The node got stuck in `degraded` state

**Solution:** Force drain with `--force` and `--ignore-daemonsets` flags.

---

## How Node Drain Works

```
oc adm drain <node>
       │
       ├── Cordons the node (marks as unschedulable)
       │
       ├── Evicts all pods (respecting PodDisruptionBudgets)
       │
       └── Fails if:
               ├── Bare pods exist (no controller) → fix: --force
               ├── DaemonSet pods exist          → fix: --ignore-daemonsets
               └── PDB blocks eviction           → fix: --disable-eviction or fix PDB
```

---

## Step-by-Step: Draining a Stuck Node

### Step 1 — Check the node status

```bash
oc get node <node1>
```

Look for status:
- `Ready,SchedulingDisabled` — node is cordoned (drain in progress)
- `NotReady` — node has issues
- `degraded` in MachineConfigPool — MCO is stuck

Check MachineConfigPool status:

```bash
oc get mcp
```

### Step 2 — Identify what is blocking the drain

```bash
oc adm drain <node1> --dry-run=client
```

This simulates the drain without making any changes — use this to see which pods are blocking.

Common blocking reasons:
| Blocker | Message |
|---|---|
| Bare pod | `cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet` |
| DaemonSet pod | `cannot delete DaemonSet-managed Pods` |
| PDB violation | `cannot evict pod as it would violate the pod's disruption budget` |

### Step 3 — Force drain with required flags

**Force deletion of bare (unmanaged) pods:**

```bash
oc adm drain <node1> --force=true
```

> `--force` permanently deletes pods not managed by any controller. They will **not** be rescheduled automatically.

**Ignore DaemonSet-managed pods:**

```bash
oc adm drain <node1> --ignore-daemonsets=true
```

> DaemonSet pods are intentionally present on every node — they will be recreated automatically after the node comes back.

**Combined command (most common in production):**

```bash
oc adm drain <node1> <node2> \
  --force=true \
  --ignore-daemonsets=true \
  --delete-emptydir-data=true
```

| Flag | Purpose |
|---|---|
| `--force=true` | Delete bare pods not managed by any controller |
| `--ignore-daemonsets=true` | Skip DaemonSet pods (they auto-recreate) |
| `--delete-emptydir-data=true` | Allow deletion of pods using emptyDir volumes (data will be lost) |
| `--dry-run=client` | Preview what would be drained without making changes |
| `--grace-period=0` | Skip graceful termination (use only in emergency) |
| `--timeout=300s` | Set timeout for drain operation |

### Step 4 — Verify the node is drained

```bash
oc get pods --all-namespaces --field-selector spec.nodeName=<node1>
```

Expected result: no pods running on the node (only DaemonSet pods if `--ignore-daemonsets` was used).

### Step 5 — Allow scheduling again (uncordon)

After maintenance is complete and the node is healthy:

```bash
oc adm uncordon <node1>
```

> In OpenShift with MCO, uncordon is usually done **automatically** after a successful node update. Do not manually uncordon unless you know what you're doing.

---

## Common Mistakes and Pitfalls

| Mistake | Why it's a problem | Solution |
|---|---|---|
| Not running `--dry-run` first | Blind drain can cause unexpected pod loss | Always dry-run first |
| Using `--grace-period=0` carelessly | Apps don't get time to flush/close connections | Only use in real emergencies |
| Forgetting `--delete-emptydir-data` | Drain silently fails if pods use emptyDir | Add the flag when needed |
| Draining multiple nodes at once | Can cause service outage if replicas are low | Drain one node at a time |
| Manually uncordoning during MCO update | MCO controls the node lifecycle — interfering can corrupt the update | Let MCO manage the process |

---

## Best Practices

- **Always dry-run before draining in production**  
  `oc adm drain <node> --dry-run=client --ignore-daemonsets --force`

- **Check PodDisruptionBudgets before draining**  
  `oc get pdb --all-namespaces`

- **Drain one node at a time** unless you have confirmed capacity on remaining nodes

- **Monitor MachineConfigPool** during upgrades  
  `watch oc get mcp`

- **Use `--timeout`** to prevent drain from hanging indefinitely  
  `oc adm drain <node> --force --ignore-daemonsets --timeout=300s`

- **Check node events** if drain is stuck  
  `oc describe node <node1>`

---

## Quick Reference Card

```bash
# Check node and MCP status
oc get node <node1>
oc get mcp

# Dry-run (simulate drain)
oc adm drain <node1> --dry-run=client --force --ignore-daemonsets

# Full force drain (most common fix for degraded node)
oc adm drain <node1> \
  --force=true \
  --ignore-daemonsets=true \
  --delete-emptydir-data=true \
  --timeout=300s

# Check remaining pods on node
oc get pods --all-namespaces --field-selector spec.nodeName=<node1>

# Uncordon node after maintenance
oc adm uncordon <node1>
```

---

## References

- [OpenShift Docs — Understanding node rebooting](https://docs.openshift.com/container-platform/latest/nodes/nodes/nodes-nodes-rebooting.html)
- [Kubernetes Docs — Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
- `oc adm drain --help`