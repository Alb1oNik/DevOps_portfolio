# OpenShift Node — Increasing RAM and CPU

## Step 1: Drain the node

```bash
oc adm drain <node_name> --ignore-daemonsets --delete-emptydir-data --force
```

> Node is cordoned automatically during drain — no new pods will be scheduled on it

---

## Step 2: Log into the node

```bash
oc debug node/<node_name>
```

---

## Step 3: Shut down the node

```bash
chroot /host
sudo su -
shutdown -h now
```

---

## Step 4: Request resource increase

> Submit a request to the VMware team to increase RAM and CPU for the VM

---

## Step 5: Mark node as schedulable after the increase

```bash
oc adm uncordon <node_name>
```

or in GUI.