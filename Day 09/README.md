# Day 9: YAML Tutorial for Kubernetes | Imperative vs Declarative | CKA Certification Course 2025

## Video reference for Day 9 is the following:

[![Watch the video](https://img.youtube.com/vi/LT3U9fkOVDQ/maxresdefault.jpg)](https://www.youtube.com/watch?v=LT3U9fkOVDQ&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---
## Imperative vs. Declarative

![Alt text](./images/9b.png)

There are two fundamental approaches to system configuration:
  1. **Imperative**: Tells the system *how* to achieve a desired state. Think of it as a step-by-step instruction manual. **Examples** below:
      - **Kubernetes**: `kubectl run my-pod --image=nginx`
        - This command directly creates a Pod with the specified image. It doesn't define the desired state, but rather instructs Kubernetes to perform a specific action.
      - **General:** Copy file A to location B, then execute command C.
  2. **Declarative**: Describes the **desired state** of the system. The system figures out how to reach that state. **Examples** below: 
      - **Kubernetes**: Create a YAML file defining the Pod specification (e.g., `pod.yaml`) and then apply it using `kubectl apply -f pod.yaml`.
        - This approach describes the desired state of the Pod (its name, image, resources, etc.) in a YAML file. Kubernetes then reconciles the actual state of the system with the desired state defined in the YAML.
      - **General:** Ensure file A exists at location B with specific permissions.

**Why Declarative is Preferred in Kubernetes**:
  - **Idempotency**: Applying the same declarative configuration repeatedly yields the same result.
  - **Simplicity**: Easier to read, write, and maintain. Focus on the **desired outcome**, not the specific steps.
  - **Version Control**: Declarative configurations are easily versioned and tracked, making it simpler to roll back changes or understand the evolution of your system.

### Imperative vs. Declarative - Table

| **Aspect**              | **Imperative**                                                          | **Declarative**                                                     |
|-------------------------|------------------------------------------------------------------------|--------------------------------------------------------------------|
| **Definition**          | Instructs the system on *how* to achieve the desired state, step-by-step. | Describes *what* the desired state should be, letting the system figure out how to achieve it. |
| **Example Command**     | `kubectl run nginx-pod --image=nginx`                                   | `kubectl apply -f nginx-pod.yaml`                                  |
| **Focus**               | Procedural: Focuses on the individual commands to reach the goal.       | Declarative: Focuses on the final desired state.                   |
| **Ease of Use**         | Simpler for quick, one-off tasks.                                       | Better for managing complex configurations.                        |
| **Idempotency**         | Not inherently idempotent (running the same command may duplicate resources). | Idempotent by design (re-applying results in the same state).      |
| **Scalability**         | Difficult to scale for multiple resources.                             | Easily scalable with version-controlled YAML manifests.            |
| **Error Prone**         | Higher risk of human error due to manual commands.                     | Reduces errors with predefined configurations.                     |
| **Use Case**            | Quick testing or temporary setups.                                     | Long-term infrastructure management and repeatable deployments.    |
| **Tooling**             | Relies on CLI commands.                                                | Uses configuration files (YAML/JSON) and tools like `kubectl apply`.|

---

## Introduction to YAML

- **YAML** (YAML Ain't Markup Language) is a human-readable data serialization language.
- In YAML, **indentation** is used to denote structure and hierarchy, typically with spaces (not tabs). **Line breaks** are used to separate different elements, such as key-value pairs, lists, and nested structures, ensuring readability and clarity.

- It's widely used in DevOps tools like Kubernetes, Terraform, Helm, Ansible, and Prometheus.
- **Key Characteristics**:
  - **Simple**: Easy to read and write, making it user-friendly.
  - **Data Serialization**: In the context of YAML, **data serialization** refers to the process of **converting** complex data structures (like Dictionaries, Lists, and nested combinations of these) into a **human-readable and machine-parsable format**.

**NOTE:** The full form **"YAML Ain't Markup Language"** highlights YAML's simplicity and difference from traditional markup languages like XML, which use verbose tags (`<`, `>`). It focuses on data serialization rather than document formatting and uses a playful recursive acronym to emphasize its lightweight, human-readable design.



### YAML vs JSON vs XML

| Feature              | YAML                                  | JSON                                   | XML                                   |
|----------------------|---------------------------------------|---------------------------------------|---------------------------------------|
| **Readability**      | Highly human-readable (indentation)  | Machine-readable, less human-friendly | Less human-readable (verbose tags)   |
| **Syntax**           | Uses indentation and line breaks     | Uses curly braces `{}` and quotes     | Uses start `<tag>` and end `</tag>`  |
| **Comments Support** | Supports comments (`#`)              | No native support                     | Limited support with `<!-- comment -->` |
| **Usage**            | Configuration files (Kubernetes, Ansible) | APIs, modern web apps                | Legacy systems, document standards   |
| **File Extensions**  | `.yml` or `.yaml`                    | `.json`                               | `.xml`                               |
| **Data Representation** | Simple and concise                | Balanced between simplicity and verbosity | Flexible but verbose               |



You can clearly observe the **simplicity** and **readability** of YAML in the snippet below:

![Alt text](./images/9a.png)


### Basic YAML Data Structures

In YAML, data is primarily represented as **key-value pairs**, where each `key` is associated with a `value`. The `value` assigned to a `key` determines the data type it represents. The `value` can be a scalar, a list, or another dictionary. These data types include:

- **Scalars**: Single values such as strings, integers, floats, booleans, and null.
  - **Strings**: A sequence of characters used to represent text. In YAML, strings don't need to be enclosed in single (`'`) or double (`"`) quotes. However, using quotes can improve readability, especially for strings that contain special characters or spaces (e.g., `Hello, World!` or `"Hello, World!"`).

  - **Integers**: Whole numbers without a fractional part (e.g., 42, -7).  
  - **Floats**: Numbers with a decimal point, representing fractional values (e.g., 3.14, -0.5).  
  - **Booleans**: Logical values representing truth or falsehood, written as true or false in YAML.  
  - **Null**: A special value representing the absence of a value or "nothing," written as null in YAML.  


- **Lists**: Ordered collections of items, represented by a series of values prefixed with hyphens (`-`).
- **Dictionaries (Maps)**: Unordered collections of key-value pairs, allowing for nested structures.

**Key-Value Pair Rules**:
  - Use hyphens or underscores for word separation (e.g., `first_name`, `my_app`).
  - Use lowercase letters for keys.
  - Avoid spaces and special characters in keys.
  - **Colon (`:`)** acts as the separator between key and value.

  **NOTE:** You can use online tools like [YAML Lint](https://www.yamllint.com/) to validate your YAML files. Additionally, you can leverage VS Code extensions to ensure you adhere to YAML syntax and best practices. These tools can help you catch errors and maintain clean, well-structured YAML configurations.


**1. Scalars**
  - the term **scalar** can be thought of as an umbrella term for all the primitive data types, including strings, integers, floats, booleans, and null.
  - In YAML, a scalar is the simplest and most fundamental data type.
  - **Example:**
    ```yaml
    first_name: Prakash # String scalar
    height: 5.9 # Float scalar
    roll_number: 9 # Integer scalar
    is_engineer: true # Boolean scalar
    pending_dues: null # Null scalar
    ```

**2. Dictionaries (Maps)**:
   - Is a collection of key-value pairs.
   - Example:
      ```yaml
      first_name: Prakash # String scalar
      height: 5.9 # Float scalar
      roll_number: 9 # Integer scalar
      is_engineer: true # Boolean scalar
      pending_dues: null # Null scalar
      address:       # Dictionary
        flat_number: x-001
        street: that_street
        state: that_state
        country: that_country
      ```

**3. Lists**:
  
  - Begins with a hyphen (-) for each item.
  - Items are typically on separate lines for readability.
  - Example:
  ```yaml
  first_name: Prakash # String scalar
  height: 5.9 # Float scalar
  roll_number: 9 # Integer scalar
  is_engineer: true # Boolean scalar
  pending_dues: null # Null scalar
  address:       # Dictionary
    flat_number: x-001
    street: that_street
    state: that_state
    country: that_country
  hobbies:      # List
    - singing
    - traveling
    - dancing
  ```
  - **Indentation**: Use consistent indentation (e.g., two spaces) for all elements within a list.

**Advanced Use Cases:**
- **Case 1**: *Nested Dictionary* and *Nested List*
This example includes a nested dictionary under `address` for contact details and a nested list under `hobbies` for different sports.

```yaml
first_name: Prakash # String scalar
height: 5.9 # Float scalar
roll_number: 9 # Integer scalar
is_engineer: true # Boolean scalar
pending_dues: null # Null scalar
address:       # Dictionary
  flat_number: x-001
  street: that_street
  state: that_state
  country: that_country
  contact: # **Nested dictionary**
    phone: 000111222
    email: prakash@example.com
hobbies:      # list
  - singing
  - traveling
  - dancing
  - sports:  # **Nested List**
      - cricket
      - football
```
- **Case 2**: **Dictionary with Nested List** and **List with Nested Dictionary**

```yaml
first_name: Prakash # String scalar
height: 5.9 # Float scalar
roll_number: 9 # Integer scalar
is_engineer: true # Boolean scalar
pending_dues: null # Null scalar
address:       # Dictionary
  flat_number: x-001 # String scalar
  street: that_street # String scalar
  state: that_state # String scalar
  country: that_country # String scalar
  contact: # Nested dictionary
    phone: 000111222 # Integer scalar
    email: prakash@example.com # String scalar
hobbies:      # List
  - singing # String scalar
  - traveling # String scalar
  - dancing # String scalar
  - sports:  # Nested list
      - cricket # String scalar
      - football # String scalar
education:    # **Dictionary with Nested List**
  degrees:
    - degree: Bachelors # String scalar
      field: Computer Science # String scalar
    - degree: Masters # String scalar
      field: Data Science # String scalar
work_experience:  # **List with Nested Dictionary**
  - company: ABC Corp # String scalar
    role: Devops Engineer # String scalar
    duration: 3 years # String scalar
  - company: XYZ Inc # String scalar
    role: DevOps Consultant # String scalar
    duration: 2 years # String scalar
```
**Dictionary with Nested List** Example:

- `education` is a **key** whose value is a dictionary with a nested list.  
- The key `degrees` in the dictionary contains a list with 2 items.  
- Each item in the list is a dictionary.  
- Each dictionary has 2 key-value pairs:  
  - `degree`: value  
  - `field`: value  


**List with Nested Dictionary** Example:
- `work_experience` is a **key** whose value is a **list** with 2 items.  
- Each item in the list is a **dictionary**.  
- Each dictionary contains **3 key-value pairs**:  
  - `company: value`  
  - `role: value`  
  - `duration: value`  


---

### Kubernetes Pod Manifest Example

- Let’s create a simple Pod manifest with multiple containers:
  ```yaml
  apiVersion: v1 # String scalar
  kind: Pod # String scalar
  metadata: # Dictionary
    name: my-pod-2 # String scalar
    labels: # Nested Dictionary
      app: nginx # String scalar
      environment: development # String scalar
  spec: # Dictionary
    containers: # List
      - name: nginx-container # String scalar
        image: nginx # String scalar
        ports: # **Nested List**
          - containerPort: 80 # Integer scalar
      - name: sidecar-container # String scalar
        image: busybox # String scalar
        args: ["sleep", "3600"] # Nested List
  ```

- **Explanation**:
  - `apiVersion` and `kind` define the type of Kubernetes resource.
  - `metadata` contains information about the Pod, such as its name.
  - `spec` defines the **desired state** of the Pod.
  - `containers` section defines the containers that will run within the Pod.
  - Each container has its own `name`, `image`, and potentially `ports`.


- **YAML Explanation**:
  - `apiVersion` is a **key** with a **string scalar** value (`v1`).  
  - `kind` is a **key** with a **string scalar** value (`Pod`).  
  - `metadata` is a **key** whose value is a **dictionary** containing:  
    - `name`: A **key** with a **string scalar** value (`my-pod-2`).  
    - `labels`: A **key** with a **nested dictionary** containing:  
      - `app`: A **key** with a **string scalar** value (`nginx`).  
      - `environment`: A **key** with a **string scalar** value (`development`).  
  - `spec` is a **key** whose value is a **dictionary** containing:  
    - `containers`: A **key** with a **list** containing 2 items.  
      - The first item is a **dictionary** with:  
        - `name`: A **key** with a **string scalar** value (`nginx-container`).  
        - `image`: A **key** with a **string scalar** value (`nginx`).  
        - `ports`: A **key** with a **nested list** containing:  
          - An item that is a **dictionary** with:  
            - `containerPort`: A **key** with an **integer scalar** value (`80`).  
      - The second item is a **dictionary** with:  
        - `name`: A **key** with a **string scalar** value (`sidecar-container`).  
        - `image`: A **key** with a **string scalar** value (`busybox`).  
        - `args`: A **key** with a **nested list** containing 2 string scalar values (`"sleep"`, `"3600"`).  


---

### Different Names for Kubernetes Configuration Files
When using the declarative approach to write configuration files for Kubernetes, these files can go by many names. While **Kubernetes Manifest Files** is the most commonly used term, you'll also encounter other variations such as:

- **Configuration Files**
- **Deployment Files**
- **Resource Files**
- **YAML Files**
- **K8s Configuration Files**
- **Kubernetes YAML Files**
- **Kubernetes Spec Files**
- **Kubernetes Definition Files**

These terms all refer to the same concept of defining and managing Kubernetes resources *declaratively*.

---

### Basic Pod-Related Commands

- Create a Pod: `kubectl apply -f pod.yaml`  
- List all Pods: `kubectl get pods`
- List Pods with details: `kubectl get pods -o wide`
- Describe a Pod: `kubectl describe pod <pod-name>`
- View Pod logs: `kubectl logs <pod-name>`  
- View logs for a specific container: `kubectl logs <pod-name> -c <container-name>`  
- Stream live Pod logs: `kubectl logs -f <pod-name>`
- Execute a command in a Pod: `kubectl exec -it <pod-name> -- /bin/bash`
- Execute a command in a specific container: `kubectl exec -it <pod-name> -c <container-name> -- /bin/bash`  
- Delete a Pod: `kubectl delete pod <pod-name>`  
  



