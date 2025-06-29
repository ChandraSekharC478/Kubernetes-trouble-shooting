
## 📘 Main Repository Overview: Kubernetes Troubleshooting Guides

This repository is dedicated to capturing real-world Kubernetes issues and their resolutions — all tested, debugged, and documented from scratch. This serves as a central troubleshooting hub.

---

## 🔧 Covered Scenarios

- ✅ CrashLoopBackOff Debugging  
  _Why it happens:_ Happens when the application fails repeatedly during startup. Common causes include misconfigured command/entrypoint, missing dependencies, or bad image versions.

- ✅ ImagePullBackOff  
  _Why it happens:_ Kubernetes is unable to fetch the container image. Reasons include incorrect image name, private registry access issue, or network DNS failures.

- ✅ StatefulSet with Persistent Volume Claim (PVC) not working after migration  
  _Why it happens:_ After cloud migration, existing PVCs may mismatch with the StorageClass or node affinity rules.

- ✅ NetworkPolicies: Unauthorized pod access to Redis  
  _Why it happens:_ When pods are not isolated, any pod in the cluster can talk to Redis unless a proper NetworkPolicy is enforced.

Each issue includes:
- The observed problem
- The root cause
- Step-by-step resolution
- Key `kubectl` and manifest usage

---

## 📂 Subdirectories and Readme Navigation

| Directory | Description |
|----------|-------------|
| `crashloopbackoff/` | Pod crashes & init failures troubleshooting |
| `imagepullbackoff/` | Image pull failures and debugging steps |
| `statefulset-pv/`   | StatefulSet & PVC debug during cloud migration |
| `networkpolicies/` | Network security using NetworkPolicy + Redis |

Each folder contains a `README.md` with full instructions, commands, and YAML manifests.

---

## 📌 Prerequisites
- Kubernetes Cluster (tested on Minikube & EKS)
- `kubectl` installed
- Basic knowledge of YAML and Kubernetes architecture


## 🧑‍💻 Author
**Chandrasekhar**  
Senior System Engineer | Frontend & DevOps Practitioner

---

> 🔐 *Security, stability, and resilience — that's what we're building into our Kubernetes systems.*
