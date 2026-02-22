Markdown
# Instructions for Validating RBAC and Application Status

This guide provides step-by-step instructions to verify that the ServiceAccount, RBAC roles, and application connectivity are correctly configured in the `todoapp` namespace.

---

## 1. Make the nex commans:
```bash
kind delete cluster
kind create cluster --config cluster.yml
./bootstrap.sh
```



## 2. Verify Resource Status
Ensure that all pods, services, and service accounts are correctly deployed before proceeding.

```bash
# Check pods in the todoapp namespace
kubectl get pods -n todoapp

# Verify the existence of the ServiceAccount
kubectl get sa secrets-listener -n todoapp
```

## 3. API Validation (Inside the Pod)
To confirm that the application can successfully use its assigned token to communicate 
with the Kubernetes API, execute a manual curl command.
# Replace <pod-name> with the actual pod name from 'kubectl get pods -n todoapp'
```bash
kubectl exec -it <pod-name> -n todoapp -- sh
```
Execute the API request:
Copy and paste the following block inside the pod terminal:

```bash
# Set environment variables for authentication
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Perform the curl request to the internal Kubernetes API
curl --cacert $CACERT \
     --header "Authorization: Bearer $TOKEN" \
     -X GET [https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets](https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets)
```
Expected Result: A JSON response starting with "kind": "SecretList"

## 4. Troubleshooting Tips
Use the auth can-i command to check if the RBAC rule is active without entering the pod.

```bash
kubectl auth can-i list secrets \
  --as=system:serviceaccount:todoapp:secrets-listener \
  -n todoapp
```
Expected Result: yes

## 5. Troubleshooting Tips

Status: Pending: Run kubectl describe pod <pod-name> -n todoapp to check for unschedulable nodes.
Status: CrashLoopBackOff: Check logs for "Table already exists" (requires database wipe) or "Unknown MySQL host" (check DB_HOST variable).
403 Forbidden: Ensure the RoleBinding name correctly references the secrets-listener ServiceAccount and the secrets-listener-role.