# Day 31: Manage TLS Certificates in Kubernetes | Create Certificate Signing Request | CKA Course 2025

## Video reference for Day 31 is the following:

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

Here’s the revised version of your pre-requisites section without the book emoji:

---

### Pre-Requisites for Day 31

Before you dive into Day 31, make sure you have gone through the following days to get a better understanding:

1. **Day 7**: Kubernetes Architecture
   Understanding Kubernetes components and their roles will help you grasp when each component acts as a client or server.

   * **GitHub**: [Day 7 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2007)
   * **YouTube**: [Day 7 Video](https://www.youtube.com/watch?v=-9Cslu8PTjU&ab_channel=CloudWithVarJosh)

2. **Day 30**: How HTTPS & SSH Work, Encryption, and Its Types
   The concepts of encryption, as well as HTTPS and SSH mechanisms, will be essential in understanding security within Kubernetes.

   * **GitHub**: [Day 30 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2030)
   * **YouTube**: [Day 30 Video](https://www.youtube.com/watch?v=MkGPyJqCkB4&ab_channel=CloudWithVarJosh)

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Client and Server – A Refresher](#client-and-server--a-refresher)  
3. [Public Key Cryptography](#public-key-cryptography)  
    3.1. [SSH Authentication (`ssh-keygen`)](#1-secure-remote-access-ssh-keygen-ssh-authentication)  
    3.2. [TLS Certificates (`openssl`)](#2-secure-web-communication-openssl-tls-certificates--identity-validation)  
    3.3. [Key Management Best Practices](#key-management-best-practices)  
    3.4. [Common Key File Formats](#common-key-file-formats)  
    3.5. [What Public Key Cryptography (PKC) Provides](#what-public-key-cryptography-pkc-provides)  
4. [Types of TLS Certificate Authorities (CA)](#types-of-tls-certificate-authorities-ca-public-private-and-self-signed)  
    4.1. [Public CA](#public-ca)  
    4.2. [Private CA](#private-ca)  
    4.3. [Self-Signed Certificates](#self-signed-certificate)  
5. [Kubeconfig and Kubernetes Context](#kubeconfig-and-kubernetes-context)  
    5.1. [What is a Kubeconfig File?](#what-is-a-kubeconfig-file-and-why-do-we-need-it)  
    5.2. [Kubernetes Contexts](#kubernetes-context)  
    5.3. [Example Kubeconfig File](#example-kubeconfig-file)  
    5.4. [Common `kubectl config` Commands](#common-kubectl-config-commands-compact-view)  
    5.5. [Managing Multiple Kubeconfigs](#multiple-kubeconfig-files)  
6. [Mutual TLS (mTLS)](#mutual-tls-mtls)  
    6.1. [Why mTLS in Kubernetes?](#why-mtls-in-kubernetes)  
    6.2. [Private CAs in Kubernetes Clusters](#private-cas-in-kubernetes-clusters)  
7. [Kubernetes Components as Clients and Servers](#kubernetes-components-as-clients-and-servers)  
8. [Private Keys and Certificates in Kubernetes](#what-are-private-keys-and-certificates)  
    8.1. [Key-Pairs for Kubernetes Components](#key-pairs-for-kubernetes-components)  
    8.2. [API Server Key-Pairs](#understanding-api-server-key-pairs-in-kubernetes)  
9. [Granting Cluster Access to a New User (Seema)](#granting-cluster-access-to-a-new-user-seema-using-certificates-and-rbac)  
10. [References](#references)


---

## Introduction

In this session, we explore how **TLS (Transport Layer Security)** is implemented within a Kubernetes cluster. While TLS is commonly associated with secure communication over the internet, Kubernetes uses it extensively for **securing internal communication** between its various components.

By the end of this session, you will have a clear understanding of how TLS and **mutual TLS (mTLS)** work across Kubernetes components such as `kubectl`, `kube-apiserver`, `etcd`, `controller-manager`, `scheduler`, and others.

---

### Client and Server – A Refresher

A **client** is the one that initiates a request; the **server** is the one that responds.

**General Examples:**
* When **Seema** accesses `pinkbank.com`, **Seema** is the **client**, and **pinkbank.com** is the **server**.
* When **Varun** downloads something from his **S3 bucket**, **Varun** is the **client**, and the **S3 bucket** is the **server**.

**Kubernetes Examples:**
* When **Seema** use `kubectl get pods`, `kubectl` is the **client** and the **API server** is the **server**.
* When the API server talks to `etcd`, the API server is now the **client**, and `etcd` is the **server**.

This direction of communication is critical when we later talk about **client certificates** and **mTLS**.

---
## **Public Key Cryptography**

### **Public Key Cryptography in DevOps: Focus on SSH & HTTPS**  

Public Key Cryptography (PKC) underpins authentication and secure communication across multiple **application-layer protocols**. While it's used in **email security (PGP, S/MIME), VoIP, database connections, and secure messaging**, a **DevOps engineer primarily interacts with SSH and HTTPS** for managing infrastructure.

Two essential tools for handling public-private key pairs in these domains are **ssh-keygen** (for SSH authentication) and **openssl** (for TLS certificates).

---

#### **1. Secure Remote Access: `ssh-keygen` (SSH Authentication)**  

`ssh-keygen` is the primary tool for generating SSH **public/private key pairs**, which enable **secure remote login** without passwords.  

- Used for **server administration, Git authentication, CI/CD pipelines, and automation**.  
- The **private key** is kept on the **client**, while the **public key** is stored on the **server** (`~/.ssh/authorized_keys`).  
- Authentication works via **public key cryptography**, where the server **verifies the client’s signed request** using its **stored public key**.  

**Alternative SSH Key Tools:**  
- **PuTTYgen** → Windows-based tool for generating SSH keys (used with PuTTY).  
- **OpenSSH** → Built-in on most Unix-based systems, provides SSH utilities including key management.  
- **Mosh (Mobile Shell)** → Used for remote connections and can leverage SSH key authentication.  

---


#### **2. Secure Web Communication: `openssl` (TLS Certificates & Identity Validation)**  

`openssl` is widely used for **private key generation** and **certificate management**, conforming to the **X.509 standard** for TLS encryption.  

- Generates a **private key**, which is securely stored on the server.  
- Creates a **certificate**, which contains a **public key**, metadata (issuer, validity), and a **digital signature from a Certificate Authority (CA)**.  
- The **private key** is used to establish secure communication, while the certificate allows clients to verify the server’s identity.  

> **Note:** TLS is not exclusive to HTTPS. Other application-layer protocols, such as **SMTP** (for email), **FTPS** (for secure file transfer), and **IMAPS** (for secure email retrieval), also use TLS for authentication and secure communication.

**Alternative TLS Certificate Tools:**

* **CFSSL (Cloudflare’s PKI Toolkit)** → Go-based toolkit by Cloudflare for generating and signing certificates.
* **HashiCorp Vault** → 	Can issue, sign, and manage certificates securely.
* **Certbot** → Automates key + cert generation from Let's Encrypt.

**Why TLS Still Uses a Tool Named "OpenSSL"**

At first glance, it may seem ironic that **TLS certificates are still generated using a tool called *OpenSSL***, especially since **SSL is obsolete** and has been replaced by TLS. But there's a historical and practical reason behind it.

**OpenSSL** originated as a toolkit to implement **SSL (Secure Sockets Layer)**, the predecessor to TLS. As SSL was deprecated and TLS became the standard, OpenSSL evolved to support all modern versions of **TLS** (including TLS 1.3), while keeping its original name for compatibility and familiarity.

But **OpenSSL is much more than just a certificate generator**. It’s a comprehensive **cryptographic toolkit** that can:

* Generate public/private key pairs
* Create and sign X.509 certificates
* Perform encryption, decryption, and hashing
* Handle PKI-related formats like PEM, DER, PKCS#12
* Test and debug secure sockets using TLS

So while the name "OpenSSL" may sound outdated, the tool itself is **modern, versatile, and still central** to secure communications today.

---

#### **Key Management Best Practices**  

To protect private keys from unauthorized access, consider these secure storage options:  

- **Hardware Security Modules (HSMs)** → Dedicated devices for key protection.  
- **Secure Enclaves (TPM, Apple Secure Enclave)** → Isolated hardware environments restricting key access.  
- **Cloud-based KMS (AWS KMS, Azure Key Vault)** → Encrypted storage with controlled access.  
- **Encrypted Key Files (`.pem`, `.pfx`)** → Secured with strong passwords.  
- **Smart Cards & USB Tokens (YubiKey, Nitrokey)** → Portable hardware-based security.  
- **Air-Gapped Systems** → Completely offline key storage to prevent network attacks.  

Regular **key rotation** and **audits** are crucial to maintaining security and replacing compromised keys efficiently. 

#### **Common Key File Formats**

#### **For SSH Authentication**

| Format                                | Description                                         |
| ------------------------------------- | --------------------------------------------------- |
| `.pub`                                | **Public key** (shared with the remote server)      |
| `.key`, `*-key.pem`, **no extension** | **Private key** (must be kept secure on the client) |




#### **For TLS Certificates**

| Format              | Description                                                     |
| ------------------- | --------------------------------------------------------------- |
| `.crt`, `.pem`      | **Certificate** (contains a public key and metadata)            |
| `.key`, `*-key.pem`, **no extension**  | **Private key** (must be securely stored)                       |
| `.csr`              | **Certificate Signing Request** (used to request a signed cert) |

By properly managing key storage and implementing best practices, organizations can significantly reduce security risks and prevent unauthorized access.

---

#### **What Public Key Cryptography (PKC) Provides**

Public Key Cryptography (PKC) underpins secure communication by enabling:
- Authentication (Identity Verification)
- Secure Key Exchange (Foundation for Encryption)

We will now discuss both of these in details.

### 1. **Authentication (Identity Verification):**

Public Key Cryptography enables entities—**users, servers, or systems**—to prove their identity using **key pairs** and **digital signatures**. This is fundamental to ensuring that communication is happening with a legitimate party.

#### **In SSH:**

SSH supports **mutual authentication**, where:

* **Server Authentication (Host Key):**

  * When an SSH server (e.g., a Linux machine) is installed, it automatically generates a **host key pair** for each supported algorithm (like RSA, ECDSA, ED25519), stored at:

    * **Private Key:** `/etc/ssh/ssh_host_<algo>_key`
    * **Public Key:** `/etc/ssh/ssh_host_<algo>_key.pub`

  * These keys are used to **prove the identity of the server** to any connecting SSH client.

  * When a client connects:

    * The server sends its **host public key**.

    * The SSH client checks whether this key is already present in its **`~/.ssh/known_hosts`** file.

  * If it's the **first connection**, the key won't exist in `known_hosts`, and the client will show a prompt:

    ```
    The authenticity of host 'server.com (192.168.1.10)' can't be established.
    ED25519 key fingerprint is SHA256:abc123...
    Are you sure you want to continue connecting (yes/no)?
    ```

    You can manually verify the fingerprint by running the following command **on the server** to print the fingerprint of its public host key:

    ```
    ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
    ```

    * `-l` (lowercase L): shows the fingerprint of the key.
    * `-f`: specifies the path to the public key file.

    Replace `ssh_host_ed25519_key.pub` with the appropriate public key file (e.g., for RSA: `ssh_host_rsa_key.pub`) if using a different algorithm.


    * If the user accepts (`yes`), the server's host key is **saved in `~/.ssh/known_hosts`** and used for verification in all future connections.

    * This approach is called **TOFU (Trust On First Use)**—you trust the server the first time, and ensure its identity doesn't change later.

  * On subsequent connections:

    * If the server's host key has changed (possibly due to a reinstallation or a **man-in-the-middle attack**), SSH will warn the user and may block the connection unless the mismatch is explicitly resolved.

    * This warning may also appear in **cloud environments** where the server’s **ephemeral public IP** changes — even if the host key is the same — because SSH uses the IP/hostname to identify the server.

* **Client Authentication:**

  * The **server sends a challenge**, and the **client must sign it using its private key** (e.g., `~/.ssh/mykey`) to prove it possesses the corresponding private key.
  * The **server verifies the signature** using the **client’s public key**, which must be present in the server’s `~/.ssh/authorized_keys` file.

#### **In TLS (e.g., HTTPS):**

* The server presents a **digital certificate** (issued and signed by a **trusted Certificate Authority**) to the client.
* This certificate includes the server’s **public key** and a **CA signature**.
* The client verifies:
  * That the certificate is **issued by a trusted CA** (whose root cert is pre-installed).
  * That the certificate matches the **domain name** (e.g., `example.com`).
* Optionally, in **mutual TLS**, the client also presents a certificate for authentication.

---

### Order of Authentication in Mutual Authentication (SSH or mTLS)

Whether using **SSH** or **mutual TLS (mTLS)**, the **server always authenticates first**, followed by the client.

**Reasons the server authenticates first:**

* The client must verify the server's identity before sending any sensitive data.
* It prevents man-in-the-middle (MITM) attacks by ensuring the connection is to the intended endpoint.
* A secure, encrypted channel is established only after the server is trusted.
* This pattern ensures that secrets (like private keys or credentials) are never exposed to an untrusted server.

This sequence holds true for:

* SSH (server presents host key first, then client authentication occurs)
* mTLS (server presents its certificate first, then requests the client's certificate if required)

---

### 2. **Secure Key Exchange (Foundation for Encryption):**

Rather than encrypting all data directly using asymmetric cryptography, **Public Key Cryptography (PKC)** is used to **securely exchange or derive symmetric session keys**. These **session keys** are then used for efficient **symmetric encryption** of actual data in transit.

* In **SSH**:

  * Both the **client and server participate equally** in a **key exchange algorithm** (e.g., **Diffie-Hellman** or **ECDH**).
  * Each side contributes a random component and uses the other’s public part to **jointly compute the same session key**.
  * **No single party creates the session key outright**; it is derived **collaboratively**, ensuring that **neither side sends the session key directly** — making it secure even over untrusted networks.
  * >🔐 **Importantly, this session key is established and encryption begins *before* any client authentication takes place.**
    This ensures that even authentication credentials (like passwords or signed challenges) are never sent in plaintext.

* In **TLS**:

  * **Older TLS (e.g., TLS 1.2 with RSA key exchange)**

    * The client generates a session key.
    * It encrypts the session key using the server’s public key (from its certificate).
    * The server decrypts it using its private key.
    * Both sides use this session key for symmetric encryption.
      **Drawback**: If the server’s private key is compromised, past sessions can be decrypted (no forward secrecy).

  * **Modern TLS (e.g., TLS 1.3 or TLS 1.2 with ECDHE)**

    * The client and server perform an ephemeral key exchange (e.g., ECDHE – Elliptic Curve Diffie-Hellman Ephemeral).
    * Both sides derive the session key collaboratively — it is never transmitted directly.
      **Advantage**: Even if the server’s private key is compromised later, past communications remain secure (forward secrecy).

> * **In SSH, encryption begins *before* authentication**, ensuring that even credentials are exchanged over a secure channel.
>* **In HTTPS (TLS), authentication happens *before* encryption**, as the server must first prove its identity via certificate.

---

### Types of TLS Certificate Authorities (CA): Public, Private, and Self-Signed

When enabling HTTPS or TLS for applications, certificates must be signed to be trusted by clients. There are three common ways to achieve this:
1. **Public CA** – Used for production websites accessible over the internet (e.g., Let's Encrypt, DigiCert).
2. **Private CA** – Used within organizations for internal services (e.g., `*.internal` domains).
3. **Self-Signed Certificates** – Quick to create, mainly used for testing, but not trusted by browsers.

**Public CA vs Private CA vs Self-Signed Certificates**

| **Certificate Type**         | **Use Case**                                      | **Trust Level**                 | **Common Examples**                       | **Typical Use**                                                                        |
| ---------------------------- | ------------------------------------------------- | ------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------- |
| **Public CA**                | Production websites, accessible over the internet | Trusted by all major browsers   | Let's Encrypt, DigiCert, GlobalSign       | Used for production environments and public-facing sites                               |
| **Private CA**               | Internal services within an organization          | Trusted within the organization | Custom CA (e.g., internal enterprise CAs) | Used for internal applications, such as `*.internal` domains                           |
| **Self-Signed Certificates** | Testing and development                           | Not trusted by browsers         | N/A                                       | Quick certificates for testing or development purposes, not recommended for production |

---

### Public CA

When you visit a website like `pinkbank.com`, your browser needs a way to verify that the server it’s talking to is indeed `pinkbank.com` and not someone pretending to be it. That’s where a **Certificate Authority (CA)** comes into play.

* `pinkbank.com` generates its own digital certificate (often through a **Certificate Signing Request** using tools like OpenSSL) and then gets it signed by a trusted Certificate Authority (CA) such as Let’s Encrypt to prove its authenticity.
* Seema’s browser, like most browsers, already contains the **public keys of well-known CAs**. So, it can verify that the certificate presented by `pinkbank.com` is indeed signed by Let’s Encrypt.
* This trust chain ensures authenticity. Without a CA, there would be no trusted way to confirm the server’s identity.

If a certificate were self-signed or signed by an unknown entity, Seema’s browser would show a warning because it cannot validate the certificate's authenticity.

**Important Note**
> In this example, I used **Let’s Encrypt** because it is a popular choice for DevOps engineers, developers, cloud engineers, and others, as they can get their certificates signed for free.
However, for **production and enterprise use cases**, organizations typically use certificates from well-known **public Certificate Authorities (CAs)** like **Verisign**, **DigiCert**, **Google CA**, **Symantec**, etc.

---

**Public Key Infrastructure (PKI)**

PKI is a framework that manages digital certificates, keys, and Certificate Signing Requests (CSRs) to enable secure communication over networks. It involves the use of **public and private keys** to encrypt and decrypt data, ensuring confidentiality and authentication.

---

### Private CA

Just like browsers come with a list of trusted CAs, you can manually add a CA’s public key to your trust store (e.g., in a browser or an operating system).

**Summary for Internal HTTPS Access Without Warnings**

To securely expose an internal app as `https://app1.internal` without browser warnings:

* Set up a **private Certificate Authority (CA)** and issue a TLS certificate for `app1.internal`.

* Install the **private CA’s root certificate** on all internal user machines so their browsers trust the certificate:

  * **Windows**: Use **Group Policy (GPO)** to add the CA cert to the **Trusted Root Certification Authorities** store.
  * **macOS**: Use **MDM** or manually import the root cert using **Keychain Access** → System → Certificates → Trust.
  * **Linux**: Place the CA cert in `/usr/local/share/ca-certificates/` and run:

    ```bash
    sudo update-ca-certificates
    ```

* Ensure internal DNS resolves `app1.internal` to the correct internal IP.

---

### Self-Signed Certificate

A **self-signed certificate** is a certificate that is **signed with its own private key**, rather than being issued by a trusted Certificate Authority (CA).

Let’s take an example:
Our developer **Shwetangi** is building an internal application named **app2**, accessible locally at **app2.test**. She wants to enable **HTTPS** to test how her application behaves over a secure connection. Since it's only for development, she generates a self-signed certificate using tools like `openssl` and uses it to enable HTTPS on **app2.test**.

#### **Typical Use Cases of Self-Signed Certificates**

* Local development and testing environments
* Internal tools not exposed publicly
* Quick prototyping or sandbox setups
* Lab or non-production Kubernetes clusters

> ⚠️ Self-signed certificates are **not trusted by browsers or clients** by default and will trigger warnings like:
> Chrome: “Your connection is not private” (NET::ERR\_CERT\_AUTHORITY\_INVALID)
> Firefox: “Warning: Potential Security Risk Ahead”

#### **Common Internal Domain Suffixes for Testing**

* `.test` — Reserved for testing and documentation (RFC 6761)
* `.local` — Often used by mDNS/Bonjour or local network devices
* `.internal` — Used in private networks or cloud-native environments (e.g., GCP)
* `.dev`, `.example` — Reserved for documentation and sometimes local use

Using these reserved domains helps avoid accidental DNS resolution on the public internet and is a best practice for local/dev setups.

---

## Kubeconfig and Kubernetes Context

### What is a Kubeconfig File and Why Do We Need It?

* The **kubeconfig file** is a configuration file that stores:

  * **Cluster connection information** (e.g., API server URL)
  * **User credentials** (e.g., authentication tokens, certificates)
  * **Context information** (e.g., which cluster and user to use)
* It allows Kubernetes tools (like `kubectl`) to interact with the cluster securely and seamlessly.
* The kubeconfig file simplifies authentication and access control, eliminating the need to manually provide connection details each time we run a command.

### What If the Kubeconfig File Were Not There?

Without the kubeconfig file, you would need to manually specify the cluster connection information for every command. For example:

* With the kubeconfig file:

  ```bash
  kubectl get pods
  ```

* Without the kubeconfig file, you would need to include details like this:

  ```bash
  kubectl get pods \
    --server=https://<API_SERVER_URL> \
    --certificate-authority=<CA_CERT_FILE> \
    --client-key=<CLIENT_KEY_FILE> \
    --client-certificate=<CLIENT_CERT_FILE>
  ```

As you can see, without the kubeconfig file, the command becomes longer and more error-prone. Each command requires explicit details, which can be tedious to manage.

**We will look into how a kubeconfile looks like in a while.**

---

### Kubernetes Context
**Kubernetes contexts** allow users to easily manage multiple clusters and namespaces by storing cluster, user, and namespace information in the `kubeconfig` file. Each context defines a combination of a cluster, a user, and a namespace, making it simple for users like `Seema` to switch between clusters and namespaces seamlessly without manually changing the configuration each time.

## Scenario:

Seema, the user, wants to access three different Kubernetes clusters from her laptop. Let's assume these clusters are:

1. **dev-cluster** (for development)
2. **staging-cluster** (for staging)
3. **prod-cluster** (for production)

Seema will need to interact with these clusters frequently. Using Kubernetes contexts, she can set up and switch between these clusters easily.

## How Kubernetes Contexts Work:

### 1. Configuring Contexts:

In the `kubeconfig` file, each cluster, user, and namespace combination is stored as a context. So, Seema can have three different contexts, one for each cluster. The contexts will contain:

* **Cluster details**: API server URL, certificate authority, etc.
* **User details**: Authentication method (e.g., username/password, token, certificate).
* **Namespace details**: The default namespace to work in for the context (though the namespace can be overridden on a per-command basis).

### 2. Switching Between Contexts:

With Kubernetes contexts configured, Seema can easily switch between them. If she needs to work on **dev-cluster**, she can switch to the `dev-context`. Similarly, for **staging-cluster**, she switches to `staging-context`, and for **prod-cluster**, the `prod-context`.

A **Kubernetes namespace** is a virtual cluster within a physical cluster that provides logical segregation for resources. This allows multiple environments like dev, staging, and prod to coexist on the same cluster without interference.

We now use naming like `app1-dev-ns`, `app1-staging-ns`, and `app1-prod-ns` to logically group and isolate resources of `app1` across environments.

For example, `app1-dev-ns` in dev-cluster keeps all resources related to `app1`'s development isolated from production resources in `app1-prod-ns` on prod-cluster.

## Example `kubeconfig` File:

> 🗂️ **Note:** The `kubeconfig` file is usually located at `~/.kube/config` in the home directory of the user who installed `kubectl`. This file allows users to connect to one or more clusters by managing credentials, clusters, and contexts. While the file is commonly named `config`, it's often referred to as the *kubeconfig* file, and its location can vary depending on the cluster setup.


Here's a simplified example of what the `kubeconfig` file might look like:

```yaml
apiVersion: v1
# Define the clusters
clusters:
  - name: dev-cluster  # Logical name for the development cluster
    cluster:
      server: https://dev-cluster-api-server:6443  # API server endpoint (usually port 6443)
      certificate-authority-data: <certificate-data>  # Base64-encoded CA certificate
  - name: staging-cluster
    cluster:
      server: https://staging-cluster-api-server:6443
      certificate-authority-data: <certificate-data>
  - name: prod-cluster
    cluster:
      server: https://prod-cluster-api-server:6443
      certificate-authority-data: <certificate-data>

# Define users (credentials for authentication)
users:
  - name: seema  # Logical user name
    user:
      client-certificate-data: <client-cert-data>  # Base64-encoded client certificate
      client-key-data: <client-key-data>  # Base64-encoded private key

# Define contexts (combination of cluster + user + optional namespace)
contexts:
  - name: seema@dev-cluster-context
    context:
      cluster: dev-cluster
      user: seema
      namespace: app1-dev-ns  # Default namespace for this context; kubectl commands will run in this namespace when this context is active
  - name: seema@staging-cluster-context
    context:
      cluster: staging-cluster
      user: seema
      namespace: app1-staging-ns  # When this context is active, kubectl will run commands in this namespace
  - name: seema@prod-cluster-context
    context:
      cluster: prod-cluster
      user: seema
      namespace: app1-prod-ns

# Set the default context to use
current-context: seema@dev-cluster-context

# -------------------------------------------------------------------
# Explanation of Certificate Fields:

# certificate-authority-data:
#   - This is the base64-encoded public certificate of the cluster’s Certificate Authority (CA).
#   - Used by the client (kubectl) to verify the identity of the API server (ensures it is trusted).

# client-certificate-data:
#   - This is the base64-encoded public certificate issued to the user (client).
#   - Sent to the API server to authenticate the user's identity.

# client-key-data:
#   - This is the base64-encoded private key that pairs with the client certificate.
#   - Used to prove the user's identity securely to the API server.
#   - Must be kept safe, as it can be used to impersonate the user.

# Together, these enable secure mutual TLS authentication between kubectl and the Kubernetes API server.

```
---

### Viewing & Managing Your `kubeconfig`

Your `kubeconfig` file (typically at `~/.kube/config`) holds info about **clusters, users, contexts, and namespaces**. Avoid editing it manually—use `kubectl config` for safe, consistent changes.

> **Pro Tip:** Use `kubectl config -h` to explore powerful subcommands like `use-context`, `set-context`, `rename-context`, etc.

---

### Common `kubectl config` Commands (Compact View)

| Task                                      | Command & Example                                                                                                                                                                        |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| View current context                      | `kubectl config current-context`<br>Example: `seema@dev-cluster-context`                                                                                                                 |
| List all contexts                         | `kubectl config get-contexts`<br>Shows: `seema@dev-cluster-context`, etc.                                                                                                                |
| Switch to a different context             | `kubectl config use-context seema@prod-cluster-context`<br>Switches to prod                                                                                                              |
| View entire config (pretty format)        | `kubectl config view`<br>Readable view of clusters, users, contexts                                                                                                                      |
| View raw config (YAML for scripting)      | `kubectl config view --raw`<br>Useful for parsing/exporting                                                                                                                              |
| Show active config file path              | `echo $KUBECONFIG`<br>Defaults to `~/.kube/config` if not explicitly set                                                                                                                 |
| Set default namespace for current context | `kubectl config set-context --current --namespace=app1-staging-ns`<br>Updates Seema's current context                                                                                    |
| Override namespace just for one command   | `kubectl get pods --namespace=app1-prod-ns`<br>Runs the command in prod namespace                                                                                                        |
| Inspect kubeconfig at custom path         | `kubectl config view --kubeconfig=~/kubeconfigs/custom-kubeconfig.yaml`                                                                                                                  |
| Add a new user                            | `kubectl config set-credentials varun --client-certificate=varun-cert.pem --client-key=varun-key.pem --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml`                                   |
| Add a new cluster                         | `kubectl config set-cluster dev-cluster --server=https://dev-cluster-api-server:6443 --certificate-authority=ca.crt --embed-certs=true --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml` |
| Add a new context                         | `kubectl config set-context seema@dev-cluster-context --cluster=dev-cluster --user=seema --namespace=app1-dev-ns --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml`                       |
| Rename a context                          | `kubectl config rename-context seema@dev-cluster-context seema@dev-env`                                                                                                                  |
| Delete a context                          | `kubectl config delete-context seema@staging-cluster-context`                                                                                                                            |
| Delete a user                             | `kubectl config unset users.seema`                                                                                                                                                       |
| Delete a cluster                          | `kubectl config unset clusters.staging-cluster`                                                                                                                                          |
---

### Multiple Kubeconfig Files

Kubernetes supports multiple config files. Use `--kubeconfig` with any `kubectl` command to specify which one to use:

```bash
kubectl get pods --kubeconfig=~/.kube/dev-kubeconfig
kubectl config use-context dev-cluster --kubeconfig=~/.kube/dev-kubeconfig
```

Ideal for managing multiple clusters (e.g., dev/staging/prod) cleanly.

---

### Using a Custom Kubeconfig as Default via `KUBECONFIG`

By default, `kubectl` uses the kubeconfig file at `~/.kube/config`. You won't see anything with `echo $KUBECONFIG` unless you've set it yourself.

To avoid specifying `--kubeconfig` with every command, you can set the `KUBECONFIG` environment variable:

**Step-by-step:**

1. Open your shell profile:

   ```bash
   vi ~/.bashrc   # Or ~/.zshrc, depending on your shell
   ```

2. Add the line:

   ```bash
   export KUBECONFIG=$HOME/.kube/my-2nd-kubeconfig-file
   ```

3. Apply the change:

   ```bash
   source ~/.bashrc
   ```

Now, `kubectl` will automatically use that file for all commands.

---

### Adding Entries to a Kubeconfig File

Avoid manually editing the kubeconfig. Instead, use `kubectl config` to manage users, clusters, and contexts.

**Add a new user:**

```bash
kubectl config set-credentials varun \
  --client-certificate=~/kubeconfigs/varun-cert.pem \
  --client-key=~/kubeconfigs/varun-key.pem \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Add a new cluster:**

```bash
kubectl config set-cluster aws-cluster \
  --server=https://aws-api-server:6443 \
  --certificate-authority=~/kubeconfigs/aws-ca.crt \
  --embed-certs=true \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Add a new context:**

```bash
kubectl config set-context varun@aws-cluster-context \
  --cluster=aws-cluster \
  --user=varun \
  --namespace=default \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Switch to the new context:**

```bash
kubectl config use-context varun@aws-cluster-context \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```
**Verify:**



```bash
kubectl config view --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file --minify
```
 * `--kubeconfig=...`: Points to your custom kubeconfig file.
 * `--minify`: Shows only the active context and related cluster/user info.
 
```bash
kubectl config view --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```



---

## Mutual TLS (mTLS)

**Mutual TLS (mTLS)** is an extension of standard TLS where **both the client and server authenticate each other** using digital certificates. This ensures that **each party is who they claim to be**, providing strong identity verification and preventing unauthorized access. mTLS is especially important in environments where **machines, services, or workloads** need to communicate securely without human involvement — such as in microservices, Kubernetes clusters, and service meshes.

We often observe that **public-facing websites**, typically accessed by **humans via browsers**, use **one-way TLS**, where **only the server is authenticated**. For example, when Seema visits `pinkbank.com`, the server proves its identity by presenting a **signed certificate** issued by a **trusted Certificate Authority (CA)**.

However, in **machine-to-machine communication** — such as in **Kubernetes** or other distributed systems — it's common to see **mutual TLS (mTLS)**. In this model, **both parties present certificates**, enabling **bi-directional trust** and stronger security.

All major **managed Kubernetes services**, including **Amazon EKS**, **Google GKE**, and **Azure AKS**, **enforce mTLS by default** for internal control plane communication (e.g., between the API server, kubelet, controller manager, and etcd), as part of their secure-by-default approach.

> Whether it's **SSH** or **TLS/mTLS**, the **server always proves its identity first**. This is by design — the client must be sure it is talking to the correct, trusted server **before** sending any sensitive data or credentials.

---

**Why mTLS in Kubernetes?**

* Prevents unauthorized components from communicating within the cluster.
* Ensures that only trusted services (e.g., a valid API server or kubelet) can connect to each other.
* Strengthens the overall security posture of the cluster, especially in production environments.

Although some components can work with just server-side TLS, enabling mTLS is **strongly recommended** wherever possible — particularly in communication with sensitive components like `etcd`, `kubelet`, and `kube-apiserver`.

| Feature               | TLS (1-Way)             | mTLS (2-Way)                               |
| --------------------- | ----------------------- | ------------------------------------------ |
| Authentication        | Server only             | **Both client and server**                 |
| Use Case              | Public websites, APIs   | Microservices, Kubernetes, APIs w/ clients |
| Identity Verification | Server proves identity  | **Mutual identity verification**           |
| Certificate Required  | Only server certificate | **Both server and client certificates**    |
| Security Level        | Strong                  | **Stronger (mutual trust)**                |


---

### **Private CAs in Kubernetes Clusters**
In Kubernetes, **TLS certificates secure communication** between core components. These certificates are signed by a **private Certificate Authority (CA)**, ensuring authentication and encryption. Depending on security requirements, a cluster can be configured to use:

* A **single CA** for the entire cluster (simpler, easier to manage), or
* **Multiple CAs** (e.g., a separate CA for etcd) for added security and isolation.

**Why Use Multiple CAs?**

Using **multiple private CAs** strengthens security by **isolating trust boundaries**. This approach is particularly useful for sensitive components like **etcd**, which stores the **entire cluster state**.

Security Considerations:
- If **etcd is compromised**, an attacker could gain full control over the cluster.
- To **limit the blast radius** in case of a CA or key compromise, it's common to **assign a separate CA exclusively for etcd**.
- Other control plane components (e.g., API server, scheduler, controller-manager) can share a **different CA**, ensuring **compartmentalized trust**.

By segmenting certificate authorities, Kubernetes operators can **reduce risk** and **enhance security posture**.

You can configure Kubernetes components to **trust a private CA** by distributing the CA’s public certificate to each component’s trust store. For example:

* `kubectl` trusts the API server because it has the **private CA** certificate that signed the API server's certificate.
* Similarly, components like the `controller-manager`, `scheduler`, and `kubelet` trust the API server and each other using certificates signed by this **shared private CA**.

Most Kubernetes clusters—whether provisioned via tools like `kubeadm`, `k3s`, or through managed services like **EKS**, **GKE**, or **AKS**—automatically generate and manage a **private CA** during cluster initialization. This CA is used to issue certificates for key components, enabling **TLS encryption and mutual authentication** out of the box.

When using **managed Kubernetes services**, this private CA is maintained by the cloud provider. It remains **hidden from users**, but all internal components are configured to trust it, ensuring secure communication without manual intervention.

However, when setting up a cluster **“the hard way”** (e.g., via [Kelsey Hightower’s guide](https://github.com/kelseyhightower/kubernetes-the-hard-way)), **you are responsible for creating and managing the entire certificate chain**. This means:

* Generating a root CA certificate and key,
* Signing individual component certificates,
* And distributing them appropriately.

While this approach offers **maximum transparency and control**, it also demands a solid understanding of **PKI, TLS, and Kubernetes internals**.


**Do Enterprises Use Public CAs for Kubernetes?**

Enterprises do **not** use public Certificate Authorities (CAs) for **core Kubernetes internals**. Instead, they rely on **private CAs**—either auto-generated (using tools like `kubeadm`) or centrally managed—to sign certificates used by Kubernetes components like the API server, kubelet, controller-manager, and scheduler. These certificates facilitate secure **TLS encryption and mutual authentication** within the cluster.

For **public-facing services**, however, it's common to use **public CAs**. Components such as:

* Ingress controllers
* Load balancers
* Gateway API implementations

...require certificates trusted by browsers. In these cases, enterprises use public CAs (e.g., Let’s Encrypt, DigiCert, GlobalSign) to issue TLS certificates, ensuring a secure **HTTPS** experience for users and avoiding browser trust warnings.

In short:

* **Private CAs** → Used for internal Kubernetes communication
* **Public CAs** → Used for securing external-facing applications
* ⚠️ Public CAs are **not** used for control plane or internal Kubernetes components
---

### Kubernetes Components as Clients and Servers

In the diagram below, arrows represent the direction of client-server communication:

* The **arrow tail** indicates the **client**, and the **arrowhead** points to the **server**.
* Some arrows have **only arrowheads** (e.g., between **kubelet** and **API server**) to indicate that **the server initiates the connection** in specific cases:

  * When a user runs commands like `kubectl logs` or `kubectl exec`, the **API server acts as the client**, reaching out to the **kubelet**.
  * Conversely, when the **kubelet pushes node or pod health data**, it becomes the **client**, and the **API server** is the **server**.
* The **etcd arrow is colored yellow** to indicate that **etcd always acts as a server**, receiving requests from the API server.

---

### **Client (Initiates a Request)**

1. **kubectl**: **Interacts with the API server**, used by admins and DevOps engineers for cluster management, deployment, viewing logs.
2. **Scheduler**: **Requests the API server** for pod scheduling, checks for unscheduled pods, manages resource placement.
3. **API Server**: **Communicates with etcd**, **interacts with kubelet** for logs, exec commands, etc.
4. **Controller Manager**: **Requests the API server** to verify desired vs. current state, manages controllers, ensures cluster state matches desired configuration.
5. **Kube-Proxy**: **Communicates with the API server** for service discovery and endpoints, acts as a network router for traffic.
6. **Kubelet**: **Reports to the API server** about node health and pod status, fetches ConfigMaps & Secrets, ensures desired containers are running.


### **Server (Responds to the Request)**

1. **API Server**: **Responds to kubectl**, admins, DevOps, and third-party clients, manages resources and cluster state.
2. **etcd**: **Responds to API server**, stores cluster configuration, state, and secrets (only the API server interacts with etcd).
3. **Kubelet**: **Responds to API server** for pod status, logs, and exec commands, provides health metrics.

> The roles mentioned above for clients and servers are **indicative**, not exhaustive. Kubernetes components may interact in multiple ways, and their responsibilities can evolve as the system grows and new features are added.

**Client or Server? It Depends on the Context**

In Kubernetes, whether a component acts as a **client** or **server** depends entirely on the direction of the request.

A single component can play both roles depending on the scenario. For example, when a user or a tool like `kubectl` accesses the **API server**, the API server acts as the **server**. However, when the **API server** communicates with another component like the **kubelet**—for fetching logs (`kubectl logs`), executing (`kubectl exec`) into containers, or retrieving node and pod status—the API server becomes the **client**, and the kubelet acts as the **server**.

Components such as the **scheduler**, **controller manager**, and **kube-proxy** are always **clients** because they initiate communication with the **API server** to get the desired cluster state, pod placements, or service endpoints.

On the other hand, **etcd** is **always a server** in the Kubernetes architecture. It **only** communicates with the **API server**, which acts as its **client**—no other component talks to etcd directly. This design keeps etcd isolated and secure, as it holds the cluster’s source of truth.

---

### What Are Private Keys and Certificates?

Think of the **private key and certificate** as a **username and password**:

* The **certificate** is the **"username"** that proves identity.
* The **private key** is the **"password"** that validates ownership of that identity.

Each Kubernetes component uses a unique key-pair:

* The **certificate** is presented to prove the component's identity.
* The **private key** remains secure and is used to prove that the component owns the certificate.

Some components, like the **API server**, act as both a **client and a server**. In such cases, you can either:

* Use the **same private key and certificate** for both roles, or
* Generate **two separate key-pairs**, one for the client role and one for the server role.

Also note:

* The **CA** itself has its own private key and certificate.
* The CA's **private key is used to sign** all other certificates (client and server), establishing **trust across the cluster**.

---

### Key-Pairs for Kubernetes Components

| **Component**             | **Private Key**                                         | **Certificate**                                  | **Default Location**                      | **Purpose / Usage**                                                                 |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------- |
| **API Server**            | `apiserver.key`                                         | `apiserver.crt`                                  | `/etc/kubernetes/pki/`                    | Serves HTTPS; proves server identity to clients.                                    |
| **API Server → etcd**     | `apiserver-etcd-client.key`                             | `apiserver-etcd-client.crt`                      | `/etc/kubernetes/pki/`                    | Authenticates API server to etcd via mTLS.                                          |
| **API Server → Kubelet**  | `apiserver-kubelet-client.key`                          | `apiserver-kubelet-client.crt`                   | `/etc/kubernetes/pki/`                    | Used by API server to authenticate to kubelets.                                     |
| **Controller Manager**    | Referenced via `controller-manager.conf` → `client-key` | `controller-manager.conf` → `client-certificate` | `/etc/kubernetes/controller-manager.conf` | Authenticates to the API server via mTLS using a kubeconfig with embedded cert/key. |
| **Scheduler**             | Referenced via `scheduler.conf` → `client-key`          | `scheduler.conf` → `client-certificate`          | `/etc/kubernetes/scheduler.conf`          | Authenticates to the API server via mTLS using a kubeconfig with embedded cert/key. |
| **Kubelet (per node)**    | `kubelet.key`                                           | `kubelet.crt`                                    | `/var/lib/kubelet/pki/`                   | Authenticates kubelet to API server.                                                |
| **Kube Proxy (per node)** | `kube-proxy.key`                                        | `kube-proxy.crt`                                 | `/var/lib/kube-proxy/`                    | Authenticates kube-proxy to API server.                                             |
 | **etcd Server**      | `etcd/etcd.key`        | `etcd/etcd.crt`        | `/etc/kubernetes/pki/etcd/` | Presented by the etcd server to authenticate to clients (e.g., kube-apiserver).           |
 | **etcd Peer**        | `etcd/etcd-peer.key`   | `etcd/etcd-peer.crt`   | `/etc/kubernetes/pki/etcd/` | Used for mTLS between etcd nodes in a multi-node etcd (HA) setup.                         |
 | **etcd Client**      | `etcd/etcd-client.key` | `etcd/etcd-client.crt` | `/etc/kubernetes/pki/etcd/` | 	Only needed if etcd itself acts as a client, e.g., to another etcd node in very specific setups.              |
 | **Main CA**          | `ca.key`               | `ca.crt`               | `/etc/kubernetes/pki/`      | Root CA for signing control plane component certs (API server, controller-manager, etc.). |
| **Service Account Token** | `sa.key`                                                | `sa.pub`                                         | `/etc/kubernetes/pki/`                    | Used to sign service account tokens (for pod-to-API authentication).                |


---

**Understanding API Server Key-Pairs in Kubernetes**

The **Kubernetes API server** communicates with multiple components, often acting as a **client**. To establish secure communication using **mutual TLS (mTLS)**, it must present a valid **TLS certificate** to each of these components. This requires **separate key-pairs** depending on the target service and the signing Certificate Authority (CA).

In tools like **KIND (Kubernetes IN Docker)**, these certificates are **automatically generated**, with KIND opting to use **three distinct key-pairs** for improved isolation and security. A minimal setup could technically function with just **two**, but this would compromise modularity and risk compartmentalization.

---

**Why Multiple Key-Pairs?**

>Before we dive into the details, it's important to remember that in **mutual TLS (mTLS)**, **both the client and the server must authenticate themselves**. This means that **each side must possess its own certificate and private key** to prove its identity securely during the handshake—regardless of whether it is acting as a client or a server in a given interaction.

The **Kubernetes API server** plays a **dual role**:
- It acts as a **server** when components like the **scheduler, controller manager, and kubelet** initiate communication with it.
- It acts as a **client** when it needs to communicate with **etcd**.

Since different components may trust **different Certificate Authorities (CAs)**, the API server must present a **certificate signed by the correct CA** for each interaction.

---

### 📌 Why Multiple Key-Pairs?

The API server interacts with **three distinct entities**, each requiring authentication via mTLS:

1. **External clients** (e.g., `kubectl`, CI/CD tools, admin users)
2. **Kubelet and Controller Manager** (which initiate communication with the API server)
3. **etcd** (the key-value store backing the cluster)

Each of these components expects the API server to **prove its identity** using a certificate signed by the appropriate CA:

- **etcd** requires mTLS using a certificate signed by the **etcd CA** (`etcd-ca`).
- **Kubelet and Controller Manager** expect a certificate signed by the **Kubernetes CA** (`kubernetes`).

Since a single certificate **cannot be signed by two different CAs**, at least **two key-pairs** are necessary in a secure setup. **KIND uses three** to further isolate responsibilities.

---

## Breakdown of API Server Certificates

| **Certificate Name**                    | **Purpose**                                                                                   | **Signed By** |
| --------------------------------------- | --------------------------------------------------------------------------------------------- | ------------- |
| `apiserver.key` & `apiserver.crt`       | Used by the API server to serve HTTPS to external clients, controller-manager, and scheduler. | Kubernetes CA |
| `apiserver-kubelet-client.key` & `.crt` | Authenticates the API server to kubelets for executing privileged operations on nodes.        | Kubernetes CA |
| `apiserver-etcd-client.key` & `.crt`    | Authenticates the API server to etcd for reading/writing cluster state via secure mTLS.       | etcd CA       |

---

**Why Kubernetes Uses Three Key-Pairs Instead of Two**

Even though a minimal setup could technically work with just two key-pairs, most **production-grade Kubernetes clusters—including KIND—use three distinct key-pairs** for the API server. This design choice enhances security and aligns with industry best practices.

* **Improved Security Isolation**
  If one certificate or key is compromised, it does not affect the integrity or trust of other communication channels (e.g., etcd access remains secure even if kubelet certs are compromised).

* **Granular Trust Boundaries**
  Each service the API server communicates with—**etcd**, **kubelets**, and **external clients**—can independently verify only the certificate relevant to it, reducing the blast radius of trust.

* **Alignment with Production Standards**
  This mirrors best practices adopted in real-world, production-grade Kubernetes environments where **separation of concerns and minimal trust** are critical.

**How Kubernetes API Server Trusts Other Components via CAs**

In Kubernetes, for mutual TLS (mTLS) to work securely, both client and server certificates must be signed by a Certificate Authority (CA) trusted by the other party. The API server explicitly declares which CAs it trusts using flags in its manifest file (**`/etc/kubernetes/manifests/kube-apiserver.yaml`**). For example:

* `--client-ca-file=/etc/kubernetes/pki/ca.crt` tells the API server to trust certificates signed by the main Kubernetes CA (used by kubelets, controller manager, etc.).
* `--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt` tells it to trust certificates signed by the etcd CA when connecting to the etcd server.

This setup ensures secure and verified communication between Kubernetes components.

---

**How Kubernetes Components Authenticate to the API Server**

Whenever a component **initiates communication toward the API server**, the API server acts as a **server**, and the component acts as a **client**. In Kubernetes, **clients authenticate using a `kubeconfig` file**, which contains:

* The API server endpoint
* A client certificate and private key
* The CA certificate to verify the server
* Optionally, a token or other authentication method

---

**Who Uses a Kubeconfig?**

All components that **initiate a connection to the API server**—such as `kubectl`, the **controller manager**, **scheduler**, **kubelet**, and **kube-proxy**—require a `kubeconfig` file.

Here's where these files are typically located:

| **Component**          | **Kubeconfig Path**                         | **Notes**                                                     |
| ---------------------- | ------------------------------------------- | ------------------------------------------------------------- |
| **Admin (kubectl)**    | `~/.kube/config` (user default)             | For interactive CLI access to the cluster                     |
|                        | `/etc/kubernetes/admin.conf` (on master)    | Used by `kubectl` running locally on the control plane        |
| **Controller Manager** | `/etc/kubernetes/controller-manager.conf`   | Used by `kube-controller-manager` to talk to the API server   |
| **Scheduler**          | `/etc/kubernetes/scheduler.conf`            | Used by `kube-scheduler` to talk to the API server            |
| **Kubelet**            | `/var/lib/kubelet/kubeconfig` (per node)    | Used by each node’s kubelet to authenticate to the API server |
| **Kube-proxy**         | `/var/lib/kube-proxy/kubeconfig` (per node) | Mounted into the **kube-proxy DaemonSet** pod                 |

> 🔍 For `kube-proxy`, since it runs as a **DaemonSet**, its kubeconfig file is mounted into each pod. You can `kubectl exec` into a `kube-proxy` pod and find the kubeconfig typically at:
> `/var/lib/kube-proxy/kubeconfig`

---


**Key Pointers**

* Every Kubernetes component (client and server) has its **own key-pair**, signed by a **Certificate Authority (CA)**.
* The **CA** uses its **private key** to sign and validate component certificates.
* The **private key acts like a password**, while the **certificate acts like a username** to identify and authenticate components.
* Trust is established across the cluster through this signed certificate infrastructure.
* Kubeconfig files bundle the certificate, private key, and CA info to authenticate users and processes.

---

### **Granting Cluster Access to a New User (Seema) using Certificates and RBAC**

---

**Granting Cluster Access to a New User (Seema) using Certificates and RBAC**
To securely grant a new user like **Seema** access to a Kubernetes cluster, we follow a series of steps involving certificate-based authentication and Role-Based Access Control (RBAC). This ensures Seema can connect and interact with the cluster within a defined scope.

---

**Step 1: Seema Generates a Private Key**

```bash
openssl genrsa -out seema.key 2048
```

This generates a 2048-bit RSA **private key**, saved to `seema.key`. This key will be used to generate a certificate signing request (CSR) and later to authenticate to the Kubernetes cluster. It must remain **private and secure**.

---

**Step 2: Seema Generates a Certificate Signing Request (CSR)**

```bash
openssl req -new -key seema.key -out seema.csr -subj "/CN=seema"
```

Seema uses her private key to create a **CSR**. The `-subj "/CN=seema"` sets the **Common Name (CN)** to `seema`, which becomes her Kubernetes username. The generated CSR contains her public key and identity, and will be signed by a Kubernetes cluster admin.

---

**Step 3: Seema Shares the CSR with the Kubernetes Admin**

```bash
cat seema.csr | base64 | tr -d "\n"
```

The CSR must be **base64-encoded** to embed it into a Kubernetes object. This command converts the CSR into a single-line base64 string, stripping newlines with `tr -d "\n"`—a necessary step for YAML formatting.

---

**Step 4: Kubernetes Admin Creates the CSR Object in Kubernetes**

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: seema
spec:
  request: <BASE64_ENCODED_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 7776000
  usages:
  - client auth
```

The admin creates a Kubernetes `CertificateSigningRequest` object.

* `request` is the base64-encoded CSR.
* `signerName: kubernetes.io/kube-apiserver-client` instructs Kubernetes to treat this as a **client authentication** request.
* `usages` defines that this certificate will be used for **client authentication**, not server TLS or other use cases.
* `expirationSeconds` sets the certificate’s validity to **90 days** (7776000 seconds).

---

**Step 5: Kubernetes Admin Approves the CSR**

```bash
kubectl certificate approve seema
```

This command **approves and signs** the certificate request. Kubernetes issues a certificate for Seema, valid per the defined usage and expiration settings.

---

**Step 6: Admin Retrieves and Shares the Signed Certificate**

```bash
kubectl get csr seema -o jsonpath='{.status.certificate}' | base64 -d > seema.crt
```

The admin retrieves the **signed certificate** from the CSR’s status, decodes it from base64, and saves it as `seema.crt`. This certificate, along with `seema.key`, is sent back to Seema for kubeconfig configuration.

---

**Step 7: Seema Configures Her `kubeconfig` with Credentials and Cluster Info**

```bash
kubectl config set-credentials seema \
  --client-certificate=seema.crt \
  --client-key=seema.key \
  --certificate-authority=ca.crt \
  --embed-certs=true \
  --kubeconfig=~/.kube/config

kubectl config set-cluster kind-my-second-cluster \
  --server=https://127.0.0.1:59599 \
  --certificate-authority=ca.crt \
  --embed-certs=true \
  --kubeconfig=~/.kube/config

kubectl config set-context seema@kind-my-second-cluster-context \
  --cluster=kind-my-second-cluster \
  --user=seema \
  --namespace=default
```

These commands configure the **user credentials**, **cluster endpoint**, and **context** in Seema’s `kubeconfig`:

* The first command tells kubectl how to authenticate Seema using her certificate/key.
* The second registers the cluster endpoint using the correct CA.
* The third defines a context associating the user, cluster, and default namespace.

---

**Step 8: Admin Creates a Role and RoleBinding for Seema**

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: seema-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "delete"]
```

```bash
kubectl create rolebinding seema-binding \
  --role=seema-role \
  --user=seema \
  --namespace=default
```

The **Role** allows Seema to **get**, **list**, and **delete** pods in the `default` namespace.
The **RoleBinding** assigns this Role to Seema's username (`CN=seema`), authorizing her actions.

---

**Step 9: Admin Verifies Authorization with `can-i`**

```bash
kubectl auth can-i delete pods --namespace=default --as=seema
```

This command is run by the **admin** to simulate whether Seema is allowed to **delete pods** in the `default` namespace.

* `--as=seema` impersonates Seema’s user identity.
* This confirms that the **RBAC permissions** are set correctly before Seema starts using the cluster.

---

**Step 10: Seema Switches to Her Configured Context**

```bash
kubectl config use-context seema@kind-my-second-cluster-context
```

This sets Seema's **active context** to the one defined earlier, allowing `kubectl` to use her certificate and connect to the right cluster/namespace.

---

**Optional: Use REST API or Alternate `kubeconfig` Files**

```bash
curl https://<API-SERVER-IP>:<PORT>/api/v1/namespaces/default/pods \
  --cacert ca.crt --cert seema.crt --key seema.key
```

*Seema can authenticate with the cluster directly via API using her certificate.*

```bash
kubectl get pods --kubeconfig=myconfig.yaml
```

*She can manage multiple clusters by specifying alternate kubeconfig files.*

---

**Step 11: Check Certificate Expiry**

```bash
openssl x509 -noout -dates -in seema.crt
```

This displays the `notBefore` and `notAfter` dates for the certificate, helping Seema monitor its expiration.

---

### References

1. **Managing TLS in a Cluster**
   [https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

2. **Certificate Management with Kubeadm**
   [https://kubernetes.io/docs/setup/best-practices/certificates/](https://kubernetes.io/docs/setup/best-practices/certificates/)

3. **Authenticating to the Kubernetes API Server**
   [https://kubernetes.io/docs/reference/access-authn-authz/authentication/](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

4. **Kubeconfig Files Explained**
   [https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

5. **Access Clusters Using the Kubernetes API**
   [https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)

6. **TLS Bootstrapping for Kubelet**
   [https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)

7. **Authorize Access to Kubernetes Resources**
   [https://kubernetes.io/docs/reference/access-authn-authz/authorization/](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

8. **Add User Access to a Cluster**
   [https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)

---