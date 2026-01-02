
## 1. What is AWS IAM?

AWS Identity and Access Management (IAM) is a **global AWS service** that allows you to **securely manage access to AWS resources**.
It enables you to define **who (identity)** can access **which AWS resources** and **what actions** they are allowed to perform.
IAM works on the concepts of **authentication** (verifying identity) and **authorization** (granting permissions).
Using IAM, organizations can avoid sharing root credentials and instead assign **fine-grained permissions**.

---

## 2. Why is IAM important in AWS?

IAM is critical because AWS resources are **publicly accessible by default** if not secured properly.
IAM helps in:

* Preventing unauthorized access
* Enforcing least-privilege access
* Centralized access management
* Improving security and compliance
  Without IAM, managing multiple users and services securely would be impossible.

---

## 3. What are the main components of IAM?

IAM consists of:

* **Users**: Individual identities for people or applications
* **Groups**: Logical collections of users with shared permissions
* **Roles**: Temporary identities assumed by trusted entities
* **Policies**: JSON documents that define permissions
  These components work together to control access securely.

---

## 4. Difference between authentication and authorization in IAM?

* **Authentication** verifies *who you are* using credentials like passwords, access keys, or MFA.
* **Authorization** determines *what actions you can perform* after authentication, based on IAM policies.
  IAM first authenticates the identity and then evaluates authorization rules.

---

## 5. What is an IAM user?

An IAM user represents a **person or application** that needs long-term access to AWS.
Each user has:

* Unique credentials (password or access keys)
* Permissions assigned via policies or groups
  IAM users are best suited for **human access**, not AWS services.

---

## 6. What is an IAM group?

An IAM group is a **collection of IAM users**.
Permissions are assigned to the group, and all users in the group inherit those permissions.
Groups simplify permission management and reduce administrative overhead.

---

## 7. What is an IAM role?

An IAM role is an identity with permissions that **does not have long-term credentials**.
Roles are assumed by:

* AWS services (EC2, Lambda, etc.)
* Users
* External accounts
  Roles provide **temporary security credentials**, making them more secure than users.

---

## 8. What is an IAM policy?

An IAM policy is a **JSON document** that defines permissions.
It specifies:

* Allowed or denied actions
* Resources affected
* Conditions under which permissions apply
  Policies are the core of authorization in IAM.

---

## 9. Types of IAM policies?

* **Managed policies**: Reusable and centrally managed
* **Inline policies**: Embedded directly into a single identity
  Managed policies are preferred for scalability and management.

---

## 10. What is the default IAM policy behavior?

By default, **all access is denied**.
Permissions must be **explicitly allowed**.
This ensures a secure default posture and prevents accidental access.

---

## 11. Difference between IAM users and IAM roles?

IAM users have **permanent credentials**, while IAM roles provide **temporary credentials**.
Users are used for humans, whereas roles are ideal for **applications and AWS services**.

---

## 12. When should you use IAM roles instead of users?

Use roles when:

* An AWS service needs access to another service
* Cross-account access is required
* You want to avoid storing access keys
  Roles improve security and scalability.

---

## 13. What is a trust relationship in IAM?

A trust relationship defines **who can assume a role**.
It is written in the role’s trust policy and specifies trusted entities.

---

## 14. What is an inline policy?

An inline policy is directly attached to a single user, group, or role.
It cannot be reused and is deleted when the identity is deleted.

---

## 15. What is a managed policy?

A managed policy is a standalone policy that can be attached to multiple identities.
It can be AWS-managed or customer-managed.

---

## 16. Difference between AWS-managed and customer-managed policies?

* AWS-managed: Maintained by AWS
* Customer-managed: Fully controlled by the account owner
  Customer-managed policies offer more customization.

---

## 17. What is the principle of least privilege?

It means granting **only the minimum permissions required** to perform a task.
This reduces security risks and limits damage in case of compromise.

---

## 18. What happens if no IAM policy is attached to a user?

The user cannot access **any AWS resources** because all actions are implicitly denied.

---

## 19. Can one IAM user belong to multiple groups?

Yes. A user can be part of multiple groups and inherits all group permissions.

---

## 20. Can an IAM role be attached to a group?

No. Roles are **assumed**, not attached to users or groups.

---

## 21. What is a permission boundary?

A permission boundary sets the **maximum permissions** an identity can have, even if policies allow more.

---

## 22. What is the use of IAM policy conditions?

Conditions restrict access based on context such as:

* IP address
* Time
* MFA usage
  They provide fine-grained control.

---

## 23. How does IAM evaluate policies?

IAM follows:

1. Explicit Deny
2. Explicit Allow
3. Implicit Deny

---

## 24. What is explicit deny?

An explicit deny **always overrides allow permissions**, ensuring strict control.

---

## 25. Identity-based vs resource-based policies?

* Identity-based: Attached to users, groups, or roles
* Resource-based: Attached directly to AWS resources

---

## 26. Can IAM roles be used across AWS accounts?

Yes. IAM supports **cross-account role assumption**.

---

## 27. How does IAM integrate with EC2?

IAM roles can be attached to EC2 instances to grant permissions securely without access keys.

---

## 28. What is the maximum number of IAM users per account?

An AWS account supports **up to 5,000 IAM users**.

---

## 29. How do you secure the root account?

* Enable MFA
* Avoid daily usage
* Use strong credentials

---

## 30. What is MFA and why is it important?

MFA adds an extra authentication factor, greatly reducing unauthorized access risks.

---

## 31. EC2 access to S3 without access keys?

Attach an IAM role with S3 permissions to the EC2 instance.

---

## 32. Restrict a user to one S3 bucket?

Use a policy that allows access only to that specific bucket.

---

## 33. Allow read-only access?

Attach a read-only IAM policy.

---

## 34. What if both allow and deny permissions exist?

Explicit deny always takes precedence.

---

## 35. How do you audit IAM activity?

Using **AWS CloudTrail**, which logs all IAM API calls.

