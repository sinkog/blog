# GitOps and the Tension Between Declared and Actual State

## Introduction

**GitOps** has become a widely accepted approach in modern infrastructure management. Its core principle is that the **desired state** of a system is defined declaratively, stored in Git, and enforced automatically through a continuous reconciliation process.

An ideal GitOps workflow involves five key steps:

1. ✅ **Define desired state** (e.g., in YAML, JSON, HCL)
2. 🔍 **Compute the diff** between desired and actual state
3. 🛡 **Validate** whether the change is safe and feasible
4. 🚀 **Apply** the changes to the infrastructure
5. 🔁 **Verify and monitor** the outcome continuously

This model assumes that the actual state is **reliably observable**. But what happens when that's not the case?

---

## The Problem: Where Does Truth Come From?

In GitOps, the *desired state* is clearly defined — it’s the content of your Git repository.

But what is the *actual state*?

- In Kubernetes, it’s easy: `kubectl get` can always return the current state, and controllers reconcile it automatically.
- But what if you’re managing virtual machines in Proxmox, cloud resources in AWS, or bare-metal systems via Terraform?

Here’s where problems emerge: these tools often **lack a single, reliable source of truth for the current state**.

---

## Example: Terraform and the State File

Terraform stores what it *thinks* exists in a `.tfstate` file.

However:

- This file does **not automatically refresh**
- It does **not always reflect reality**
- It can easily become **outdated** or **diverge from the actual system**

As a result, any `terraform plan` is calculated based on a cached version of the world — not the live infrastructure — and **drift becomes inevitable**.

---

## Why This Breaks GitOps Assumptions

GitOps assumes that:

- A **valid diff** can be computed between Git and the real world
- This diff is **deterministic** and **auditable**
- Changes are **only applied when necessary**
- Post-apply verification confirms **actual state == desired state**

This only works if:

✅ The real system is **observable in real time**  
✅ The tool can compute a meaningful **diff**  
✅ The change is **idempotent** and **verifiable**

Without this, GitOps becomes more about *hope* than *guarantee*.

---

## Which Tools Fit This Model?

| Tool / System             | GitOps-Ready? | Notes                                                         |
|---------------------------|---------------|---------------------------------------------------------------|
| ✅ **Kubernetes**         | Yes           | Built-in state, CRDs, reconciliation loops                    |
| ⚠️ **Ansible**           | Partially     | No stored state, relies on assumptions and conditionals       |
| ⚠️ **Terraform**         | Limited       | State is local or remote but not live, risk of divergence     |
| ✅ **Custom Observers**   | Yes           | Full control, pull-based state from APIs, modular workflows   |

---

## What Would Be Ideal?

For a GitOps system to be reliable, the following should be true:

- Actual state must be **pull-based** and live from the system
- State must be **machine-readable** and normalizable
- It must be **versionable** and comparable to the spec
- The diff should result in a **modular and verifiable execution**

This means **the tooling must follow the logic**, not the other way around. The Git repository holds the truth — the system must adapt.

---

## Conclusion

GitOps is not about the tooling — it’s about the mindset:
> *"Declare what should be, observe what is, and reconcile the two — safely, verifiably, and repeatably."*

When tools rely on isolated, cached state (like `terraform.tfstate`), they no longer fit this paradigm as primary drivers.

In such cases, it's better to:

- Use Terraform as an execution backend, not as the source of truth
- Or replace it entirely with pull-based API clients and declarative logic

🧠 The question is no longer *"Which tool do you use?"* but:

> *"Can you truly observe your infrastructure — and trust what you see?"*

---

