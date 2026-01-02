🛠️ Task Requirements
🔹 1. Root Account Security

Enable MFA on the root account

Do not use the root account for daily tasks

🔹 2. IAM Users Design

Create IAM users for:

admin-user

dev-user

audit-user

❗ No permissions should be directly attached to users.

🔹 3. IAM Groups Design

Create the following groups:

AdminGroup

DeveloperGroup

AuditGroup

Add:

admin-user → AdminGroup

dev-user → DeveloperGroup

audit-user → AuditGroup

🔹 4. IAM Policies (Advanced)

Create customer-managed policies:

🔸 Admin Policy

Full IAM access except:

Cannot delete IAM users

Cannot modify permission boundaries

🔸 Developer Policy

Can:

View IAM roles and policies

Assume specific roles

Cannot:

Create users

Attach policies

🔸 Audit Policy

Read-only access to IAM

Cannot modify anything

🔹 5. Permission Boundaries (Critical)

Create a permission boundary that:

Prevents:

iam:CreateUser

iam:DeleteUser

iam:AttachUserPolicy

Apply this boundary to:

All IAM users

All IAM roles

✅ Even admins must respect the boundary.

🔹 6. IAM Roles (No Services)

Create the following roles:

🔸 AdminRole

Assumable only by AdminGroup

Requires MFA to assume

🔸 DeveloperRole

Assumable only by DeveloperGroup

Limited IAM permissions

❗ No role should have permanent credentials.

🔹 7. Trust Policies

Define trust relationships so that:

Only intended groups/users can assume their roles

Cross-assumption is not allowed

🔹 8. IAM Policy Conditions (Advanced)

Add conditions such as:

MFA required for sensitive actions

Deny role assumption without MFA

Restrict actions based on IAM context keys

🔹 9. Explicit Deny Rules

Add explicit deny statements to:

Prevent privilege escalation

Prevent policy modification beyond allowed scope

🔹 10. Validation & Testing

Manually verify:

Admin cannot bypass permission boundary

Developer cannot create users

Audit user cannot modify any IAM object

Explicit deny always overrides allow
