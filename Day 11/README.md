# Day 11: Microservices & 3-Tier Architecture | Software Design for Kubernetes | CKA Course 2025

## Video reference for Day 11 is the following:

[![Watch the video](https://img.youtube.com/vi/waJ8brXMtYg/maxresdefault.jpg)](https://www.youtube.com/watch?v=waJ8brXMtYg&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

# **Understanding Frontend, Backend, Middleware, and the Three-Tier Architecture**  

A **three-tier architecture** is a common software design pattern that separates applications into **three logical layers** to improve scalability, maintainability, and separation of concerns.  

### Monolithic Architecture  

![Alt text](./images/11c.png)

Before moving to **3-tier architecture**, let's understand **monolithic design**, where everything existed as a **single codebase**.

A **monolithic application** is a **self-contained, tightly integrated** system where the **UI, business logic, and database** are part of a **single deployment unit**. While simple to develop initially, it becomes **harder to scale, update, and maintain** as the application grows.  

Modern applications **typically avoid** monolithic or single codebase designs, but you may still encounter them in **legacy systems** that have not been refactored into modular architectures.

**Challenges with Monolithic Design**
1. **Tightly Coupled Systems:** Components (UI, business logic, database) were tightly coupled, making changes risky and prone to breaking other parts.  
2. **Difficult Maintenance and Upgrades:** Fixes or upgrades required changes to the entire application, causing downtime and complex testing.  
3. **Limited Scalability:** Scaling required scaling the entire system, leading to inefficient resource use and bottlenecks.  
4. **Lack of Flexibility in Development:** Development was slower due to dependencies across components, making changes prone to errors.

---

## **1. Three-Tier Architecture Overview**  

![Alt text](./images/11a.png)

| **Tier**       | **Description**  | **Example Technologies** |
|---------------|----------------|--------------------------|
| **Frontend (Web Tier)** | User interface that interacts with the application | HTML, CSS, JavaScript, React, Angular, Vue.js |
| **Middleware** | Handles authentication, logging, caching, API communication, and service coordination | Express.js, Nginx, API Gateway, Kafka, RabbitMQ |
| **Backend (Application Tier)** | Handles business logic, processing, and communication with databases | Python, Node.js, Java, Go, .NET |
| **Database (Data Tier)** | Stores, manages, and retrieves data | MySQL, PostgreSQL, MongoDB, Redis |


---

## **2. Understanding Each Tier in Detail**  

### **Frontend (Web Tier)**  
The **frontend** is the interface through which users interact with an application. It typically includes **web pages, mobile applications, or graphical user interfaces (GUIs)**.  
 

📌 **Example in an E-Commerce Website:**  
- The **homepage, product listing, and shopping cart UI** are part of the frontend.  
- When a user clicks "Add to Cart," the request is sent to the backend.  

### **Middleware**  
Middleware is software that sits **between the frontend and backend**, handling tasks such as:  

- **Authentication & Authorization** (e.g., OAuth, JWT, API keys)  
- **Logging & Monitoring** (e.g., ELK stack, Prometheus, Grafana)  
- **Caching** (e.g., Redis, Memcached)  
- **Service Communication** (e.g., API Gateways, Message Brokers)  

📌 **Example in an E-Commerce Website:**  
- A **payment gateway middleware** validates credit card payments before passing them to the backend.  
- **Rate-limiting middleware** protects APIs from excessive requests.  

### **Backend (Application Tier)**  
The **backend** is responsible for **processing user requests, applying business logic, and communicating with the database**. It acts as the **intermediary** between the frontend and the database.  

📌 **Example in an E-Commerce Website:**  
- The **order management system** processes orders, updates inventory, and sends confirmation emails.  
- A **wishlist service** allows users to save items.  

### **Database (Data Tier)**  
The **database** stores, manages, and retrieves structured or unstructured data.  

📌 **Example in an E-Commerce Website:**  
- **User Database** stores account details.  
- **Product Database** stores items, prices, and availability.  
- **Order History Database** keeps track of past purchases. 

---
### 3-Tier Deployment Models
Here are some deployment models of the 3-tier architecture. Please note that this list is indicative and not exhaustive, as it does not include models like Event-Driven, Serverless, and others.

### A. Traditional Single-Server Deployment 

In the early days of software development, applications were **deployed on a single physical server**, where:  

- The **frontend (UI), backend (business logic), and database** all ran on the **same machine**.  
- This setup was **simple** but had **major drawbacks**:  
  - **Scalability issues** – The entire system had to be scaled together (vertical scaling).  
  - **Performance bottlenecks** – A high load on one component affected the entire application.  
  - **High risk of failure** – If the server crashed, everything went down.  

📌 **Example:** A small retail business running an inventory management system on a single server. If the system experienced heavy traffic, the entire application would slow down or crash.  


### B. Moving to a Three-Server Model (Pre-Virtualization Era)

Companies that could afford it **separated the three tiers** onto **dedicated physical servers**:  

1. **Frontend Server** – Handled user requests and served web pages.  
2. **Backend Server** – Processed business logic and communicated with the database.  
3. **Database Server** – Stored and managed data.  

This **three-server model** provided **better performance, fault isolation, and scalability** but was **expensive** because each physical server required **dedicated hardware, power, and maintenance**.  

📌 **Example:** A financial institution investing in three separate servers to handle user interactions, business logic, and sensitive transaction data securely.  


### C. How Virtualization Enabled Widespread 3-Tier Adoption

With the rise of **virtualization**, even smaller businesses could adopt **three-tier architectures** without needing three physical machines. Virtualization allowed:  

- **Running multiple virtual machines (VMs) on a single physical server**.  
- **Isolating different tiers in separate VMs**, providing fault tolerance and security.  
- **Easier scaling** – Businesses could allocate resources dynamically to different tiers.  

📌 **Example:** A growing e-commerce startup using **VMware or KVM-based virtualization** to run frontend, backend, and database **on separate VMs instead of physical servers**.  


### D. The Shift to Cloud and Kubernetes

While **virtual machines improved efficiency**, they still had limitations in terms of **scalability and automation**. This led to:  

- **Cloud Computing (AWS, Azure, GCP)** – Instead of managing VMs, companies now use **cloud-based services** to run their applications.  
- **Kubernetes & Containers** – Instead of running each tier on a VM, **applications are containerized** and deployed using **Kubernetes** for better scalability, automation, and resource efficiency.  

📌 **Example:** A modern SaaS company running its application in Kubernetes with:  
- **Frontend pods** handling UI requests.  
- **Backend microservices** running in separate deployments.  
- **Database hosted on a cloud-managed service (e.g., AWS RDS, Google Cloud SQL).**  

---

# **3. Challenges with the Traditional 3-Tier Architecture**  

While the **three-tier architecture** provides better organization and scalability compared to monolithic applications, it also has **several challenges**:  

1. **Scalability Issues** – Scaling is inefficient; even if one function needs more resources, the entire tier must be scaled.  
2. **Single Point of Failure (SPOF)** – A failure in one tier can bring down the entire application.  
3. **Tight Coupling** – Codebases within each tier are monolithic, making updates and deployments risky.  
4. **Slow Development & Deployment** – Teams working on different features face delays due to large, interdependent codebases.  
5. **Limited Technology Flexibility** – All components in a tier typically use the same technology stack, limiting optimization.  

Microservices address these issues by **breaking down large, tightly coupled tiers into smaller, independent services** that **scale, deploy, and evolve separately**. Let's now explore **what microservices are and how they solve these challenges.**  

---

# **4. Introduction to Microservices**  

![Alt text](./images/11b.png)

## **Why Do We Need Microservices?**  

Traditional applications followed a **monolithic architecture**, where all components (frontend, backend, database) were tightly coupled into a **single codebase**. This approach led to **scalability issues, deployment difficulties, and maintenance challenges**.  

### **Example: Monolithic E-Commerce Application**  
In a **monolithic** e-commerce platform, all features (login, payments, product catalog, orders, wishlist) exist in **one large application**. If the **wishlist feature crashes, the entire application might go down**.  

### **Microservices: A Better Alternative**  
A **microservices architecture** breaks down an application into **small, independent services** that communicate over APIs. 

Unlike monolithic architectures, microservices allow independent scaling, deployment, and updates for each service. A key benefit is fault isolation—if one service fails, the rest of the application continues to function. For example, in an e-commerce app, if the **Wishlist Service** goes down, users can still browse products and place orders, but they won’t be able to view or add items to their wishlist until the service is restored.


📌 **Example in an E-Commerce Website:**  
- **User Service:** Handles authentication and profiles.  
- **Product Service:** Manages product listings and inventory.  
- **Order Service:** Processes orders and payments.  
- **Wishlist Service:** Allows users to save favorite items.  

**If the Wishlist Service goes down, the rest of the application continues to work.**  

Why Wishlist **Service**, not Wishlist **Microservice**?
In a microservices design, components are called **Wishlist Service** instead of **Wishlist Microservice** for clarity and convention. "Service" is a widely accepted term that avoids redundancy, as microservices are inherently services. This naming keeps it simple, aligns with industry standards, and applies across different architectures.

### **Benefits of Microservices**  

| **Benefit**        | **Explanation** |
|-------------------|---------------|
| **Scalability** | Each microservice can scale independently based on demand. |
| **Fault Isolation** | Failure in one service does not crash the entire application. |
| **Faster Deployment** | Teams can deploy and update services independently. |
| **Technology Flexibility** | Different services can use different programming languages. |
| **Easier Maintenance** | Smaller codebases are easier to manage and debug. |

---

# **5. How Microservices Fit Into the Three-Tier Architecture in Kubernetes**  

Each **tier** in a microservices-based application can have **multiple microservices**, and each microservice will have its **own Deployment in Kubernetes**.

 

### **Mapping Microservices to a Three-Tier Architecture**  

| **Tier**       | **Microservices Example**  |
|---------------|----------------|
| **Frontend (Web Tier)** | Web UI, Mobile UI |
| **Middleware** | API Gateway, Authentication, Logging, Caching |
| **Backend (Application Tier)** | Order Service, Payment Service, Wishlist Service |
| **Database (Data Tier)** | Outside Kubernetes |

### **Database (Data Tier) – Where Should Databases Be Hosted?**  

The **database tier** stores, manages, and retrieves structured or unstructured data. While **Kubernetes supports databases using StatefulSets**, running databases inside Kubernetes is **not a common practice in production environments** due to operational complexity. 

**Stateful applications** maintain information about previous interactions, while stateless applications do not. Stateful applications are better for session management and database management, while stateless applications are better for scaling

### **Why Are Databases Usually Not Hosted in Kubernetes?**  
- **Stateful Workloads Are Hard to Manage** – Databases need stable networking, persistent storage, and HA setups, which Kubernetes doesn’t handle natively.  
- **Performance Concerns** – Cloud-managed database services or dedicated VMs provide **better storage I/O, backup, and replication capabilities**.  
- **High Availability & Backups** – Setting up HA, replication, and automatic failover is easier with **cloud-managed services like AWS RDS, Google Cloud SQL, or Azure Database**.  

### **Where Are Databases Hosted in Production?**  
- **Cloud-Managed Database Services** (e.g., AWS RDS, Google Cloud SQL) – Fully managed with automated scaling and backups.  
- **Dedicated Virtual Machines (VMs) with HA Configurations** – Gives full control over performance tuning.  
- **Hybrid Approach** – Applications run in **Kubernetes**, while databases are hosted externally.  
- **StatefulSets** can be used to deploy **databases** onto Kubernetes.  
**StatefulSets** in Kubernetes manage **stateful applications** that need stable network identities, persistent storage, and ordered deployment. Unlike **Deployments**, StatefulSets ensure each pod gets a **unique, persistent identity** across restarts.  
  - **Common Use Cases:**  
    - Databases like **MySQL** and **PostgreSQL**  
    - Distributed systems like **Kafka** and **Zookeeper**  

### **When Would You Run a Database in Kubernetes?**  
- For **development or testing** where persistence isn't critical.  
- If using **Kubernetes-native database operators** (e.g., MySQL Operator, PostgreSQL Operator). We will discuss Operators later in the course.
  - A **Kubernetes Operator** is a custom controller that automates the deployment, scaling, and management of complex applications on Kubernetes. It extends Kubernetes capabilities by encoding human operational knowledge into software, enabling self-healing and lifecycle management of applications.
- For small-scale, **self-managed** deployments with lower operational complexity. 




---

### **Example: How an E-Commerce Application Works in Kubernetes**  

1. A user **visits** `https://example.com`.  
2. The request is routed to **Ingress Controller**, which sends it to the **Frontend Service**.  
3. The **Frontend (React, Vue, Angular)** makes an API call to the **API Gateway (Middleware)**.  
4. The **API Gateway** forwards the request to the appropriate backend microservice, e.g., the **Wishlist Service**.  
5. The **Wishlist Service** retrieves data from the **Database** and sends it back to the frontend.  
6. The user sees the updated wishlist on their screen.  

### **Scaling Microservices in Kubernetes**  
- Each microservice runs as its own **Kubernetes Deployment.**  
- **HPA (Horizontal Pod Autoscaler)** can scale each microservice separately. HPA can create and delete pods based on load, we will look into this later in the course.

---

# **6. Reference Links**  
- Microservices Explained: [https://aws.amazon.com/microservices/](https://aws.amazon.com/microservices/)  

- What are Kubernetes Operators?: [https://kubernetes.io/docs/concepts/extend-kubernetes/operator/](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)  

- What is an API Gateway?: [https://www.nginx.com/resources/glossary/api-gateway/](https://www.nginx.com/resources/glossary/api-gateway/)  
- API Gateway in Microservices: [https://microservices.io/patterns/apigateway.html](https://microservices.io/patterns/apigateway.html)  
  
