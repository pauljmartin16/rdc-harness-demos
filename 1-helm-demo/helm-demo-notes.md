# Helm Demo
- install as a single application using templated Helm Umbrella Charts
- yaml content taken from the excellent Pluralsight course at https://app.pluralsight.com/library/courses/kubernetes-packaging-applications-helm/table-of-contents
- Ingress yamls updated from /v1beta to /v1 format

# Pre-Requisites
```
Install kubectl CLI - https://kubernetes.io/docs/tasks/tools/
Install Helm CLI - https://helm.sh/docs/intro/install/
k8s Cheatsheet for reference - https://kubernetes.io/docs/reference/kubectl/cheatsheet/
```

# Install k8s Environment
- use cloud k8s only for Ingress support

# Initialize k8s Namespace
```
kubectl create namespace helm-demo-ns
kubectl config set-context --current --namespace=helm-demo-ns
```

# Install Ingress Operator
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update [ingress-nginx]        // Or omit the ingress-nginx to update all of them

helm install --set controller.watchIngressWithoutClass=true --namespace ingress-nginx --create-namespace ingress-nginx ingress-nginx/ingress-nginx

// using --watch to monitor the install - requires CTRL-C to terminate
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller

// Verify Ingress
helm list -n ingress-nginx
```

# Install Guestbook App
```
helm template guestbook
helm install demo guestbook --debug --dry-run 2>&1 | less
helm install demo guestbook
```

# Get the Ingress Host/IP
- if Address is blank then wait until ready
```
kubectl get ingress
```

# Register DNS Name Hosts file
- C:\Windows\System32\drivers\etc
- replace with the assigned Ingress IP
```
45.79.245.202	frontend.minikube.local
```

# Run the Helm app
```
https:\\frontend.minikube.local
```

# Uninstall the Helm app
```
helm uninstall demo
```