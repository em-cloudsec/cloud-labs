### Acronyms Used
- **IAM**: Identity and Access Management (users/roles/permissions in AWS)
- **MFA**: Multi-Factor Authentication (extra login factor)
- **ARN**: Amazon Resource Name (unique resource ID in AWS)

### Config Evidence (Policies on `cl0udadmin`)
```text
arn:aws:iam::aws:policy/AdministratorAccess
arn:aws:iam::aws:policy/IAMUserChangePassword

MFA enabled for user: cl0udadmin
MFA device serial (ARN): arn:aws:iam::******728494:user/cl0ud-admin
