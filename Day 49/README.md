# Day 49: MASTER Kubernetes Ingress | PART 1 | What, Why & Real-World Flow of Ingress | CKA 2025

## Video reference for Day 49 is the following:

[![Watch the video](https://img.youtube.com/vi/yj-ZlKTYDUI/maxresdefault.jpg)](https://www.youtube.com/watch?v=yj-ZlKTYDUI&ab_channel=CloudWithVarJosh)


---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Table of Contents

* [Introduction](#introduction)
* [Why Ingress?](#why-ingress)
* [What Are We Looking For?](#what-are-we-looking-for)
* [What Is Ingress?](#what-is-ingress)
* [What Is an Ingress Controller?](#what-is-an-ingress-controller)
* [Real-World Flow: How Ingress Works](#real-world-flow-how-ingress-works)
* [Ingress Setup: Cloud-Native vs 3rd-Party Controllers](#ingress-setup-cloud-native-vs-3rd-party-controllers)
* [Popular Ingress Controllers](#popular-ingress-controllers)
* [What’s Ahead](#whats-ahead)
* [Conclusion](#conclusion)
* [References](#references)

---


## Introduction

Ingress is one of the most production-critical Kubernetes resources — enabling clean, cost-effective routing of HTTP and HTTPS traffic to your internal services. While Service types like NodePort and LoadBalancer get the job done, they’re limited in flexibility, cost efficiency, and routing capabilities at scale.

In this first part of our 3-part Ingress series, we’ll build a strong foundation by answering:

* Why do we need Ingress when we already have Services?
* What is an Ingress resource and what role does it play?
* What is an Ingress Controller and how does it translate Ingress rules into real routing logic?
* How does a real-world request (like `/iphone`) get routed through cloud-managed load balancers and Kubernetes Services?

By the end of this lecture, you’ll fully understand the **"What", "Why", and "How"** of Ingress — with clear architecture diagrams and request flows, preparing you for upcoming hands-on demos on Amazon EKS.

---

## Why Ingress?

Exposing applications using a `NodePort` service (as covered in [Day 12](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)) is suitable for dev/test but falls short in production. In real-world Kubernetes clusters, nodes frequently scale in/out, making it impractical to expose dynamic node IPs externally.

Using a `LoadBalancer` service type improves on this by provisioning a cloud-managed load balancer (like AWS ELB). However, this brings its own challenges:

* Provisions a **Layer 4 (TCP/UDP) Load Balancer**—no native support for HTTP-level (Layer 7) routing such as path- or host-based rules.
* Each service creates a **dedicated cloud load balancer**, leading to cost and manageability concerns in microservice environments.
* **TLS termination must happen in the app**, as L4 load balancers can't decrypt HTTPS traffic.

---

## What Are We Looking For?

To run production workloads effectively, we need a Kubernetes-native solution that:

* Supports **HTTP(S)-aware routing** like `/android`, `/iphone`, or `iphone.myapp.com`.
* Uses a **single cloud Load Balancer** to serve multiple applications, reducing cost and simplifying operations.
* Supports **declarative, version-controlled routing rules** that are part of your Kubernetes manifests.

This is where **Ingress** becomes essential.

---

## What Is Ingress?

Ingress is a **Kubernetes API object** that defines how external HTTP(S) traffic is routed to internal Services. It acts as a **Layer 7 router** that uses URL paths, hostnames, or TLS settings to decide how traffic should be forwarded.

Ingress enables:

* **Path-based routing** (`/android` → `android-svc`)
* **Host-based routing** (`iphone.myapp.com` → `ios-svc`)
* **TLS termination** at the edge (not in the app)
* **A single centralized entry point** for all HTTP(S) traffic

> 🧠 Note: The Ingress resource is **declarative only**. It doesn't actually route traffic—it's just a YAML spec.

An Ingress requires an **Ingress Controller** to interpret and implement the routing logic.

---

## What Is an Ingress Controller?

An **Ingress Controller** is a component running in your cluster that **monitors Ingress resources** and **translates rules into actual load balancer configurations**.

Put simply:

> Ingress defines **what should happen**, the Ingress Controller decides **how to make it happen**.

It provisions a Layer 7 **HTTP(S) Load Balancer** and configures:

* Listeners (80/443)
* Routing rules (based on Ingress paths/hosts)
* Health checks and target group management

Though the term "L7 Load Balancer" is common, technically Kubernetes only supports **HTTP and HTTPS**, so it's more precise to say **HTTP(S) Load Balancer**.

---

## Real-World Flow: How Ingress Works

![Real-World Ingress Flow](./images/49a.png)

Let’s say **Shwetangi** visits `myapp.com/iphone`. How does this external request reach the correct Kubernetes Pod inside the cluster?

Here's what happens step by step:

1. The request from the browser travels over the internet and reaches an **external Load Balancer**.
2. This Load Balancer was **provisioned by the Ingress Controller** running inside your Kubernetes cluster.

   * If you're using a **cloud-native Ingress Controller** (like AWS LBC or GKE Ingress), the controller provisions an **L7 HTTP(S) Load Balancer** on the cloud provider automatically.
   * If you're using a **3rd-party Ingress Controller** like NGINX or HAProxy (common in on-prem or hybrid setups), you must manually create a **Layer 4 Load Balancer** using a `Service: LoadBalancer`, which simply forwards traffic to the Ingress Controller pod.
3. The **Ingress Controller reads the Ingress resource**, which defines routing rules — for example, all requests to `/iphone` should go to `iphone-svc`.
4. Based on these rules, the controller:

   * Updates the external Load Balancer config (in cloud-native setups), or
   * Handles routing internally after receiving traffic from the L4 LB (in 3rd-party setups).
5. The request is forwarded to the matching **Kubernetes Service (`iphone-svc`)**, which then routes it to one of the associated Pods.

> 📌 Note: The **Ingress resource is not a traffic hop**. It's a **declarative object** that the controller watches to configure external or internal routing mechanisms.



---


## Ingress Setup: Cloud-Native vs 3rd-Party Controllers

**Cloud-Native (AWS Load Balancer Controller – Demo 1)**
In managed cloud setups like EKS, the controller provisions an external **L7 ALB** from your Ingress manifest. It handles **TLS termination**, **routing**, and **load balancing**—all outside the cluster—offloading the Kubernetes control plane.

**3rd-Party Controllers (NGINX, HAProxy – On-Prem or Cloud)**
These controllers run **inside the cluster** and must be exposed via a **`Service` of type `LoadBalancer`**, which provisions a **Layer 4 Load Balancer**. This forwards all traffic to the controller pod, which then:

* Handles **TLS termination**
* Applies **Ingress routing rules**
* Forwards requests to appropriate **Kubernetes Services**

All logic remains **inside the cluster**, unlike ALB-based setups.

---

## Popular Ingress Controllers

| Controller                       | Description                                                                                 |
| -------------------------------- | ------------------------------------------------------------------------------------------- |
| **AWS Load Balancer Controller** | Manages ALBs and NLBs dynamically on EKS. Deep AWS integration with high scalability        |
| **NGINX Ingress Controller**     | Most widely used, highly customizable with strong community support                         |
| **HAProxy Ingress**              | Optimized for high performance and low latency routing                                      |
| **Traefik**                      | Cloud-native, supports automatic TLS via Let’s Encrypt and real-time dynamic config updates |
| **Contour**                      | Built on Envoy, offers advanced traffic control and multi-team ingress management           |
| **Istio Ingress Gateway**        | Part of the Istio service mesh. Enables policies, mTLS, and fine-grained traffic management |

While they all support the standard Kubernetes Ingress API, their **operational models and advanced features** differ. Choose based on your **cloud, scale, and security needs**.

---

## What’s Ahead

Ingress is a cornerstone of Kubernetes networking in production. In this **three-part series**, we’ll go from concepts to real-world demos:

* **Part 1 (Day 48)**:
  What, Why & How of Ingress
  Understanding Ingress Controllers
  Real-World Ingress Flow Walkthrough

* **Part 2 (Day 49)**:
  Deploy a Multi-AZ Amazon EKS cluster
  **Demo 1: Path-Based Routing** using AWS ALB and the AWS Load Balancer Controller

* **Part 3 (Day 50)**:
  **Demo 2**: Enable HTTPS with **TLS Termination using ACM** and **Custom Domain via Route 53**
  **Demo 3**: **Name-Based Routing** for `iphone.cwvj.click`, `android.cwvj.click` using the **same ALB**

Let’s now begin **Demo 1 on Amazon EKS in Day 50** and see Ingress in action.

---

## Conclusion

In this session, we unpacked the **role of Ingress in Kubernetes**, explored how it works in tandem with **Ingress Controllers**, and analyzed how real-world HTTP(S) requests are routed to your workloads.

We saw how cloud-native controllers like the **AWS Load Balancer Controller** offload TLS termination and routing outside the cluster using an **Application Load Balancer (ALB)**, whereas 3rd-party controllers like **NGINX** or **HAProxy** rely on an **internal setup with manual Layer 4 Load Balancers**.

This foundational knowledge sets the stage for our next two parts:

* **Day 50 (Part 2)**: We’ll provision a Multi-AZ Amazon EKS cluster and walk through **Demo 1: Path-Based Routing using ALB and AWS LBC**.
* **Day 51 (Part 3)**: We’ll take it further with **TLS termination via ACM**, **Route 53-based custom domains**, and **Name-Based Routing** using a single ALB.

---

## References

* [AWS Load Balancer Controller Documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
* [Kubernetes Official Docs – Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
---
