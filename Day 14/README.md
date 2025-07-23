# Day 14: Kubernetes Namespaces Explained | Isolation & Resource Management | CKA Course 2025

## Video reference for Day 14 is the following:

[![Watch the video](https://img.youtube.com/vi/uDlhbtGy1AU/maxresdefault.jpg)](https://www.youtube.com/watch?v=uDlhbtGy1AU&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Table of Contents
- [What Are Namespaces in Kubernetes?](#what-are-namespaces-in-kubernetes)
- [Analogy to Understand Namespaces](#analogy-to-understand-namespaces)
- [Why Use Namespaces?](#why-use-namespaces)
- [Default Namespaces in Kubernetes](#default-namespaces-in-kubernetes)
- [Working with Namespaces](#working-with-namespaces)
  - [Imperative and Declarative Creation](#imperative-and-declarative-creation)
  - [Commands to List, Create, and Delete Namespaces](#commands-to-list-create-and-delete-namespaces)
  - [Using `-n`, `-A`, and `--all-namespaces` Flags](#using-n-a-and--all-namespaces-flags)
- [Deploying Frontend and Backend in a Namespace](#deploying-frontend-and-backend-in-a-namespace)
- [Testing Namespace Isolation](#testing-namespace-isolation)
- [Setting a Default Namespace in the Kubernetes Context](#setting-a-default-namespace-in-the-kubernetes-context)
- [Best Practices for Using Namespaces](#best-practices-for-using-namespaces)
- [Summary](#summary)
- [References](#references)

---

## **What Are Namespaces in Kubernetes?**
A **namespace** in Kubernetes is a **logical partition** within a cluster that helps organize and isolate **resources**. Namespaces enable:  

- **Isolation & Security:** Separate workloads to prevent unwanted interactions.  
- **Avoiding Naming Conflicts:** Resources with the **same name** can exist in **different namespaces**.  
- **Resource Management:** Apply **resource quotas** and **limits** at the namespace level.  
- **Application Segregation:** Separate **environments** (e.g., **dev**, **test**, **prod**) or **projects**.  
- **Organizational Clarity:** Manage resources by **teams**, **departments**, or **projects**.  

---

## **Analogy to Understand Namespaces**  
Imagine a **large house** where **four families** live together:  

- Without namespaces, all families share **common spaces**, leading to **no privacy, security, or organization**.  
- When you **create rooms** for each family, each family has its **own space**, improving **isolation, security, and organization**.  

### **Relating to Kubernetes:**  
- The **large house** is the **Kubernetes cluster**.  
- The **families** are **applications** or **workloads**.  
- **Rooms** are **namespaces**, providing **segregation and control** over resources.  

---

## **Why Use Namespaces?**
- 🛡️ **Security:** Limit access and apply **network policies**.  
- 🚦 **Resource Management:** Define **quotas** and **limits** for CPU and memory.  
- 🎯 **Environment Management:** Isolate **development**, **testing**, and **production** environments.  
- 📦 **Multi-Tenancy:** Host **multiple applications** in the **same cluster** without conflicts.  
- 🧹 **Simplification:** Manage **related resources together**.  

---

## **Default Namespaces in Kubernetes**  
When you run `kubectl get namespaces`, you’ll see these **default namespaces**:  

| **Namespace** | **Purpose** |
|---------------|--------------|
| `default`    | Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace. |
| `kube-system` | The namespace for objects created by the Kubernetes system. Holds **Kubernetes control plane components** (e.g., **kube-dns**, **kube-proxy**). |
| `kube-public` | This namespace is readable by all clients (including those not authenticated). **Publicly accessible data**, primarily used for **cluster information**. |
| `kube-node-lease` | **Heartbeats of nodes** in the cluster, used by the **control plane** for **node health**. |

**Note:** In **KIND** (**K**ubernetes **IN** **D**ocker) clusters, the local-path-storage namespace is created by default to support persistent storage using the Local Path Provisioner.

---

## **Working with Namespaces**

### **Creating Namespaces**

#### **1. Imperative Way:**
```sh
kubectl create namespace app1-ns
```

#### **2. Declarative Way:**
```yaml
# app1-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app1-ns
```

```sh
kubectl apply -f app1-ns.yaml
```

---

### **Viewing and Deleting Namespaces**

#### **View All Namespaces:**
```sh
kubectl get namespaces
```

#### **Delete a Namespace:**
```sh
kubectl delete namespace app1-ns
```
**Warning:** Deleting a namespace **removes all resources within it**, including **pods**, **services**, **configmaps**, **secrets etc**..

---

### **Using Namespace Flags**

| **Flag** | **Description** |
|----------|-----------------|
| `-n <namespace-name>` or `--namespace` | Execute commands within a **specific namespace**. |
| `-A` or `--all-namespaces` | Execute commands across **all namespaces**. |

```sh
kubectl get pods -n app1-ns
kubectl get services -A
```

---

## **Deploying Frontend and Backend in a Namespace**

- We'll use **existing YAML files** for our **frontend** and **backend deployments**.
- Deploy the **frontend** and **backend** components in the **app1-ns** namespace.

---

### **1. Applying Frontend and Backend YAMLs**

#### **1.1 Frontend YAML (`frontend-deploy.yaml`)**
```sh
kubectl apply -f frontend-deploy.yaml -n app1-ns
```

#### **1.2 Backend YAML (`backend-deploy.yaml`)**
```sh
kubectl apply -f backend-deploy.yaml -n app1-ns
```

---

### **2. Verify the Deployments in `app1-ns`**

```sh
kubectl get all -n app1-ns
```

---

## **Testing Namespace Isolation**

### **1. Test Pod in `default` Namespace**
```sh
kubectl run test-pod --image=busybox -it --rm --restart=Never -- /bin/sh
```

```sh
curl backend-svc:9090 # ❌ Will not work
```
If Busybox image giving curl not found error, in that case i used curl image itself using which I validated the service. Command as follows:

```sh
kubectl run curl-pod --image=curlimages/curl   -it --rm --restart=Never   -- sh -c "curl http://backend-service.app1-ns:9090"
```

**For cross-namespace access, use the following format:**  
```sh
curl http://backend-svc.app1-ns:9090
```

### ✅ **Format:**  
```sh
curl http://<service-name>.<namespace-name>:<service-port>
```

- **`<service-name>`**: Name of the **Kubernetes Service**, e.g., **`backend-svc`**.  
- **`<namespace-name>`**: Namespace where the **service** is deployed, e.g., **`app1-ns`**.  
- **`<service-port>`**: The **ClusterIP port** exposed by the **service**, e.g., **`9090`**.  


### 💡 **Coming Up:**  
In our **DNS resolution lecture**, we'll explore **why this format is needed** and understand **how Kubernetes handles service discovery** within and across **namespaces**.

---

### **2. Test Pod in `app1-ns` Namespace**
```sh
kubectl run test-pod -n app1-ns --image=busybox -it --rm --restart=Never -- /bin/sh
```

```sh
curl backend-svc:9090 # ✅ Should work
```

---

## **Setting a Default Namespace in Kubernetes Context**

### **Why?**  
It’s **cumbersome** to use `-n <namespace>` with **every command**. You can set a **default namespace** in the **Kubernetes context**.

```sh
kubectl config set-context --current --namespace=app1-ns
```

- Now, you **don't need** to use `-n app1-ns` with **every command**.  
- To **check the current context**, run:  

```sh
kubectl config get-contexts
```

---

## **Best Practices for Using Namespaces**

- **Segregate Workloads:** Separate **dev**, **test**, and **prod environments**.  
- **Use Namespaces for Multi-Tenancy:** Avoid **resource conflicts** by **isolating teams or projects**.  
- **Resource Quotas:** Set **limits** on **CPU**, **memory**, and **storage** per namespace.  
- **Namespace Naming Conventions:** Use **clear** and **consistent names** (e.g., `team-app-env` → `frontend-prod-ns`).  
- **Avoid Manual Deletion:** Use `kubectl delete namespace` **carefully**, as it **removes all resources within**.  
- **Apply Network Policies:** Secure namespaces using **network policies** to control **traffic flow**.  

---

## **Summary**

- **Namespaces** provide **logical isolation** in Kubernetes.  
- They help with **security**, **resource management**, and **multi-tenancy**.  
- Use **imperative** and **declarative methods** to **create and manage namespaces**.  
- Be **careful when deleting namespaces**, as it **removes all resources within**.  
- **Set default namespaces** in your **Kubernetes context** for **ease of use**.  
- Follow **best practices** to **organize resources effectively** and **ensure security**.  

---

## References  
- [Kubernetes Namespaces Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
