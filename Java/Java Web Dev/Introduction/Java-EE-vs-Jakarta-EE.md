# Java EE vs Jakarta EE

Jakarta EE is the modern iteration of what was formerly known as Java EE (Java Platform, Enterprise Edition). The two are fundamentally connected, but key differences in governance, naming, and features distinguish them.

### The Transition: From Oracle to Eclipse Foundation

Originally developed by Sun Microsystems and later managed by Oracle, Java EE was a set of specifications for building enterprise-scale applications. In 2017, Oracle transferred the development and stewardship of Java EE to the Eclipse Foundation, an open-source organization.

This move was significant for several reasons:

*   **Trademark Issues**: Oracle owns the "Java" trademark. Consequently, the Eclipse Foundation could not continue using the name Java EE. After a community vote, the project was rebranded to **Jakarta EE**.
*   **Namespace Restrictions**: Oracle also restricted the use of the `javax` package namespace for any new development.

### Key Differences Between Java EE and Jakarta EE

| Feature             | Java EE                                                        | Jakarta EE                                                                     |
| ------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Governance**      | Managed by Oracle under the Java Community Process (JCP).      | Managed by the Eclipse Foundation under the open and vendor-neutral Jakarta EE Specification Process (JESP). |
| **Package Namespace** | Uses the `javax.*` namespace for all its APIs.                 | Jakarta EE 8 still used `javax.*`, but versions 9 and later use the `jakarta.*` namespace. |
| **Development Focus** | Traditionally focused on monolithic application architectures. | Stronger focus on modern architectures like microservices and cloud-native applications. |
| **Release Cadence** | Releases were less frequent.                                   | Releases are more frequent, aiming for a more agile and community-driven roadmap. |

### The "Big Bang": `javax` to `jakarta`

The most significant technical change and breaking change for developers is the package namespace migration from `javax.*` to `jakarta.*`.

*   **Jakarta EE 8**: This version is essentially a rebranding of Java EE 8 and is fully compatible. It was released to prove that the transition of code and processes to the Eclipse Foundation was successful.
*   **Jakarta EE 9**: This release marked the "big bang" — the comprehensive switch of all API package names from `javax` to `jakarta`. This change breaks backward compatibility, meaning applications and their dependencies must be updated to use the new `jakarta` packages.
*   **Jakarta EE 10 and beyond**: These versions continue to build on the new `jakarta` namespace, introducing new features and enhancements for cloud-native and microservices development.

### Why Migrate?

Migration to Jakarta EE is crucial for applications to receive new features, security updates, and long-term support, as Java EE is no longer being updated. Most modern application servers, frameworks (like Spring 6), and libraries have already transitioned to support Jakarta EE.

To assist with this transition, tools like the Eclipse Transformer, OpenRewrite, and features within IDEs like IntelliJ IDEA can automate the significant task of refactoring code from the `javax` to the `jakarta` namespace.
