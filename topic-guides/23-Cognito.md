# Section 23: Cognito

## The idea

**Amazon Cognito** manages customer authentication and access to AWS resources through two separate components:

* **User Pools** manage user accounts, sign-up, sign-in, password resets, and authentication.
* **Identity Pools** exchange identity tokens for temporary AWS credentials, allowing applications to access AWS resources directly.

Key terms:

* **JWT (JSON Web Token)** — a signed token containing claims about an authenticated user.
* **STS (AWS Security Token Service)** — issues temporary AWS credentials.
* **IdP (Identity Provider)** — a service that authenticates users, such as Google, Facebook, or a corporate SAML provider.

### The core distinction (THE thing to know)

|                   | **User Pools**                                                                                                           | **Identity Pools**                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| Question answered | **AUTHENTICATION** — who are you?                                                                                        | **AWS resource access** — what AWS resources may you access?              |
| What it is        | User directory: sign-up, sign-in, password resets                                                                        | Exchanges identity tokens for AWS credentials                             |
| What it returns   | **JWT tokens**                                                                                                           | **Temporary AWS credentials** (via **STS**)                               |
| Extras            | **Hosted UI** (pre-built login pages), **MFA**, **social login** (Google/Facebook/Apple), **SAML** enterprise federation | **Guest / unauthenticated access**, fine-grained per-user IAM permissions |
| Plugs into        | **API Gateway** and **Application Load Balancer (ALB)** for authentication                                               | S3, DynamoDB, and other AWS APIs — directly from the app                  |

**Exam trap:** If users need to access S3 directly from a mobile app, **Cognito User Pools** alone are insufficient. A JWT cannot authenticate directly to S3; direct AWS API access requires AWS credentials. Choose **Identity Pools** to obtain temporary credentials.

### Key takeaway

| Requirement                                                       | Best service                                                                   |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Authenticate users (sign-up/sign-in)                              | **Cognito User Pool**                                                          |
| Authenticate REST API requests using Cognito                      | **Amazon API Gateway Cognito User Pool authorizer**                            |
| Grant application users direct AWS resource access (S3, DynamoDB) | **Cognito Identity Pool** — not needed for API-only access through API Gateway |
| Enable social login (Google, Facebook, Apple)                     | **Cognito User Pool**                                                          |
| Add MFA to customer logins                                        | **Cognito User Pool**                                                          |
| Allow unauthenticated guests limited AWS access                   | **Cognito Identity Pool**                                                      |
| Restrict each user to their own S3 prefix                         | **Cognito Identity Pool + IAM policy variables**                               |
| Federate enterprise customers through a corporate SAML provider   | **Cognito User Pool with SAML federation**                                     |
| Give employees SSO access to multiple AWS accounts                | **AWS IAM Identity Center**, not Cognito                                       |

**Key distinction:** An API Gateway authorizer validates a user's token to control API access. An Identity Pool provides temporary AWS credentials for direct access to AWS services. These solve different problems.

### The classic flow

```text
 User logs in (email / Google / SAML)
        │
        ▼
 ┌─────────────┐   JWT token   ┌───────────────┐   exchange via STS   ┌──────────────────┐
 │  User Pool   │ ────────────▶ │ Identity Pool │ ───────────────────▶ │ Temporary AWS    │
 │              │               │               │                      │ credentials       │
 └─────────────┘               └───────────────┘                      └──────────────────┘
                                                                               │
                                                                               ▼
                                                        App calls S3 / DynamoDB directly
```

Identity Pools can accept tokens from a **User Pool**, supported **social identity providers**, or **SAML** providers, depending on the configured federation setup.

### The clever bits worth exam points

* **Authorizers:** A **Cognito User Pool authorizer on API Gateway** validates JWTs before requests reach the backend. If the requirement is to authenticate API users without writing custom authentication code, choose a User Pool authorizer. An **Application Load Balancer (ALB)** can also authenticate users through Cognito before forwarding requests.
* **Guest access:** Identity Pools support **unauthenticated identities**. To let users access limited content before signing up, use Identity Pool guest access and restrict the associated IAM permissions.
* **Per-user permissions with policy variables:** An IAM policy used with an Identity Pool can include `${cognito-identity.amazonaws.com:sub}` to restrict each user to their own S3 prefix, such as `s3://bucket/${their-own-id}/*`. **"Each user accesses only their own folder"** → Identity Pool + **IAM policy variables**. One policy can enforce per-user access without creating a separate policy for every user.

### Cognito vs IAM Identity Center — don't mix up the audiences

**Exam trap:** *"Employees need single sign-on to AWS accounts"* → choose **AWS IAM Identity Center**, the successor to AWS Single Sign-On (AWS SSO), not Cognito.

* **Customers of your application** (external users) → **Cognito**
* **Employees / workforce** signing into AWS accounts and business applications → **IAM Identity Center**

For workforce access to AWS accounts, choose IAM Identity Center rather than Cognito.

## Question patterns

> *"Mobile app needs user sign-up, sign-in, and login with Google and Facebook"* → **Cognito User Pools** (directory + social login + hosted UI).

> *"Authenticated app users must upload files directly to S3 from the device"* → **Cognito Identity Pools** (JWT → STS → temporary AWS credentials; only creds can call S3).

> *"Allow users to browse limited content without creating an account"* → **Identity Pool unauthenticated (guest) access**.

> *"Authenticate users of a REST API on API Gateway without custom code"* → **Cognito User Pool authorizer** (API Gateway validates the JWT for you).

> *"Each user may only read and write objects in their own S3 prefix"* → **Identity Pool + IAM policy variables** (`${cognito-identity...:sub}` scopes one policy per user).

> *"Company employees need SSO access to multiple AWS accounts"* → **IAM Identity Center, NOT Cognito** (workforce = Identity Center; customers = Cognito).

> *"Enterprise customers must log into your SaaS app with their corporate SAML identity provider"* → **User Pools with SAML federation** (authentication through a federated identity provider).

> *"Add MFA to your application's customer logins"* → **User Pools** (MFA is configured for authentication).

## Pocket card

| Keyword                                    | Answer                                                                        |
| ------------------------------------------ | ----------------------------------------------------------------------------- |
| Sign-up / sign-in / user directory         | User Pools                                                                    |
| JWT tokens                                 | User Pools                                                                    |
| Social login (Google/Facebook/Apple), SAML | User Pools (federation)                                                       |
| Hosted UI, MFA                             | User Pools                                                                    |
| API Gateway / ALB authorizer               | User Pool authorizer                                                          |
| Temporary AWS credentials for app users    | Identity Pools (via STS)                                                      |
| Direct app access to S3/DynamoDB           | Identity Pools                                                                |
| Guest / unauthenticated access             | Identity Pools                                                                |
| Each user only their own folder            | Identity Pool + IAM policy variables                                          |
| Employees SSO to AWS accounts              | IAM Identity Center (never Cognito)                                           |
| Core distinction                           | User Pool = authentication and JWT; Identity Pool = temporary AWS credentials |

Cognito handles customer identities. When identities are managed in a corporate Microsoft Active Directory environment, consider **AWS Directory Service** and its supported directory integration options.
