# Kubernetes Service Types - Networking Overview

This document provides an overview of the different Kubernetes service types: **ClusterIP**, **NodePort**, and **LoadBalancer**, along with usage instructions.

---

## ClusterIP

- **ClusterIP** is the **default service type** in Kubernetes.
- It enables **internal communication** between pods **within the cluster**.
- The service is **not accessible from outside the cluster** by default.
- A **Kubernetes proxy** (`kubectl proxy`) can be used to access ClusterIP services for debugging or displaying internal dashboards.

### Use Case
- Internal networking between workloads
- Debugging applications
- Displaying internal dashboards

### Example
To check if the application is reachable from a node:

```bash
curl <cluster-ip>:<port>

NodePort

    NodePort exposes the service on a static port on each node's IP address.

    No complex configuration is required.

    It routes traffic from the node IP and the NodePort to the service.

Use Case

    Ideal for experimentation, demos, and internal training.

    Not recommended for production use due to limitations.

Limitations

    One service per port

    May require a reverse proxy (e.g., NGINX)

    Dynamic container IPs may cause DNS issues

    No direct localhost access from outside the pod

Example

To access the application:

http://<public-node-ip>:31404

LoadBalancer

    LoadBalancer is the most commonly used service type for production workloads.

    Automatically provisions an external IP or DNS to expose the service.

    Integrates with cloud provider's load balancer (e.g., AWS ELB, Azure Application Gateway).

    Routes traffic based on port, protocol, hostname, or application labels.

Use Case

    Production-grade workloads

    Internet-accessible services

Example

When service type is set to LoadBalancer, Kubernetes provides an external DNS:

http://<load-balancer-dns>

Use this DNS to access the application.
Summary
Service Type	Accessibility	Use Case	External Access
ClusterIP	Internal only	Internal communication, dashboards	❌
NodePort	Node IP + Port	Demos, POCs, Internal Testing	✅ (limited)
LoadBalancer	Internet	Production, External Access	✅

    💡 For production, always prefer LoadBalancer or Ingress Controllers for efficient traffic routing and scaling.
