# Instana Agent - Delete

**Tools:** helm, oc, access to Instana

**Instruction:** To delete all data from the project, the following steps are needed: delete → install → delete (while terminating)

## Steps

1. **Delete data with command**
```bash
   helm uninstall instana-agent
```

2. **From Instana CLI, get the code to install Instana agent to cluster:**
   - Main page → More → Agents → Install Agents → OpenShift
   - Choose technology: Helm Chart
   - Enter cluster name (test, prod, etc.)
   - Copy the code

3. **Login to cluster with token**
```bash
   oc login --token=......
```

4. **Paste the copied Helm chart code** 
   - Start installation
   - This will terminate all pods

5. **When all pods change status to "Terminating", delete data again**
```bash
   helm uninstall instana-agent
```