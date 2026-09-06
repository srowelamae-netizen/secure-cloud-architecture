# Secure Cloud Architecture Plan

The proposed architecture follows this basic flow:

```
Users
  ↓
CDN
  ↓
Load Balancer
  ↓
Application Servers
  ↓
Private Database
```

# CDN
The CDN (Content Delivery Network) stores cached copies of static content, such as images, CSS, and HTML pages, on servers located closer to users. This reduces latency and improves page loading speed while also absorbing some traffic before it reaches the core infrastructure.

# Load Balancer
The load balancer sits between the CDN and the application servers and distributes incoming requests evenly across multiple application servers. This prevents any single server from becoming overloaded and allows the system to keep working even if one server goes down.

# Application Servers
Application servers process requests from users, such as retrieving or updating student records. These servers should be placed in a private subnet so they cannot be reached directly from the Internet — all traffic must pass through the load balancer first.

# Database
The database stores student records, including personal and academic information. The database should remain private and should not be directly accessible from the Internet. Only the application servers should be allowed to query it.

# Public and Private Resources

| Resource | Public or Private? | Explanation |
|---|---|---|
| CDN | Public | The CDN must be reachable by users on the Internet so it can serve cached static content quickly. |
| Load Balancer | Public | The load balancer needs a public endpoint to receive incoming user traffic and route it to the application servers. |
| Application Server | Private | Application servers should only receive traffic from the load balancer, not directly from users, so they are placed in a private subnet. |
| Database | Private | The database holds sensitive student data and should only be reachable from the application servers, never directly from the Internet. |

# Security Controls

# IAM
Access to the cloud environment should be limited to people who actually need it to do their jobs. Administrators manage the overall infrastructure, developers manage application code and deployments, and instructors/students only interact with the application itself, not the cloud console. Each person should have their own individual account rather than shared credentials, so actions can be traced back to a specific user.

# MFA
Multi-Factor Authentication should be required for all accounts that can access the cloud console or infrastructure, especially Administrator and Developer accounts, since compromising these accounts could expose or damage the entire system. MFA adds a second verification step (such as a one-time code) so a stolen password alone is not enough to log in.

# Firewall / Security Group
Firewall/security group rules should only allow the minimum connections needed for the system to function:
- Internet → Load Balancer = Allowed
- Load Balancer → Application Server = Allowed
- Application Server → Database = Allowed
- Internet → Database = Blocked

All other inbound connections should be denied by default.

# Encryption
Student information includes personally identifiable data (names, emails, student numbers) and should be encrypted both in transit (using HTTPS/TLS between users, the CDN, load balancer, and application servers) and at rest (encrypting the database storage). This protects the data even if network traffic is intercepted or storage media is compromised.

# Logging
Logging should record who accessed the system, what actions they performed (logins, record views, edits, deletions), when the action occurred, and from what IP address. This creates an audit trail that can be used to investigate incidents or unauthorized changes.

# Monitoring
Monitoring should watch for suspicious activity such as repeated failed login attempts, logins from unusual locations or at unusual times, unexpected spikes in traffic (which could indicate an attack), and unauthorized attempts to access the database directly. Alerts should notify administrators so they can respond quickly.

# Backup
The database should have regular, automated backups so that student data can be restored in the event of accidental deletion, hardware failure, corruption, or a ransomware/security incident. Backups should also be stored securely and tested periodically to confirm they can actually be restored.

# Principle of Least Privilege

| User | Allowed Access |
|---|---|
| Administrator | Full access to cloud infrastructure, IAM settings, security configurations, and all application data. Reserved for a small number of trusted personnel. |
| Instructor | Can view and manage student records relevant to their classes (e.g., view/update grades or enrollment) but cannot access cloud infrastructure or IAM settings. |
| Student | Can view only their own personal student record (profile, grades, enrollment) through the application; no access to other students' data or any administrative functions. |
| Developer | Can access application code, deployment pipelines, and non-production/test environments to build and fix the application, but should not have standing access to production student data or full infrastructure control. |

Administrator access is not given to everyone — only those who need to manage the infrastructure itself.

# Shared Responsibility Model

| Responsibility | Cloud Provider or Customer? |
|---|---|
| Physical data center | Cloud Provider |
| Physical servers | Cloud Provider |
| User accounts | Customer |
| Student data | Customer |
| IAM permissions | Customer |
| Application security | Customer |
| Database access rules | Customer |
| Backups | Customer |

*What does Security OF the Cloud mean?*
Security "of" the cloud refers to the cloud provider's responsibility to secure the underlying infrastructure — the physical data centers, hardware, networking, and virtualization layer that everything else runs on top of.

*What does Security IN the Cloud mean?*
Security "in" the cloud refers to the customer's responsibility to securely configure and use the services they build on top of the provider's infrastructure — including user accounts, IAM permissions, application code, data encryption, firewall rules, and backups.

# Architecture Questions

*1. Which resource should be directly accessible from the Internet?*
The CDN and the load balancer should be directly accessible from the Internet, since they are the entry points designed to receive public user traffic.

*2. Why should the database remain private?*
The database stores sensitive student information. Keeping it private ensures it can only be reached through the application servers, reducing the attack surface and preventing attackers from querying or stealing data directly.

*3. Why should users not connect directly to the database?*
Direct user connections would bypass the application's business logic, validation, and access controls, making it much easier for someone to view, modify, or delete data they shouldn't have access to, or to run malicious queries against the database.

*4. What is the purpose of a load balancer?*
A load balancer distributes incoming traffic across multiple application servers so that no single server is overwhelmed, and it improves availability by routing traffic away from any server that fails.

*5. What happens if one application server fails?*
If one application server fails, the load balancer detects this and stops sending traffic to it, routing all requests to the remaining healthy server(s) so users experience little or no disruption.

*6. What is the purpose of a CDN?*
A CDN caches static content closer to users geographically, which reduces load times, decreases the load on the origin servers, and improves the overall user experience.

*7. Why should administrator accounts use MFA?*
Administrator accounts have the highest level of access to the system, so if one is compromised, an attacker could take control of the entire infrastructure. MFA makes it much harder for an attacker to gain access even if they steal a password.

*8. Why should administrator access not be given to every employee?*
Limiting administrator access follows the principle of least privilege — the more people who have full access, the greater the chance of accidental misconfiguration, insider misuse, or a compromised account leading to a major breach.

*9. Why are logging and monitoring important?*
Logging and monitoring allow the team to detect suspicious or unauthorized activity quickly, investigate incidents after they happen, and maintain accountability by tracking who did what and when.

*10. Why are backups important?*
Backups ensure that student data can be recovered if it is accidentally deleted, corrupted, or lost due to hardware failure or a security incident, preventing permanent data loss.
