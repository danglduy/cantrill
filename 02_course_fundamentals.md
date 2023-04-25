# AWS Account

- Users & Resources
- Root user
  - Email (unique)
  - Credit card
  - Full control over the aws account
  - Can't be restricted
- IAM users
  - Can be restricted
  - IAM Groups & Roles

# MFA

- Extra step to secure users
- Methods
  - Virtual MFA (Mobile App)
  - U2F Security Key
  - Other hardware keys

# IAM Basics

- Root users: Full permission, unrestricted
- IAM:
  - Globally resilient service => available all regions
  - Create
    - Identities (users)
    - Groups - collections of users
    - Roles
      - Can be used by AWS services & external access
      - Allow uncertain things to do something
  - Policy: Rules to allow or deny something
- IAM Purposes
  - Manage identities - An Identity Provider (IDP)
  - Authentication
  - Authorization

# IAM Access keys

- To user with command line
- Long term: No rotation
- Can have 2 access keys per IAM user at the same time
- Cannot view secret key after created
