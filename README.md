
## 🔐 Self-Hosting & Security Engineering Portfolio

Welcome! I'm a **hands-on security enthusiast** building production-grade infrastructure from scratch. This repository documents my journey through self-hosting, system hardening, and privacy-focused solutions—all with a focus on real-world security practices.


## About This Project

I believe the best way to learn security is to **build, break, and rebuild**. Each project in this collection represents a real implementation I've deployed, documented, and secured. these projects are battle-tested configurations from my own infrastructure.

**Why this matters for security:**

* I understand infrastructure from the ground up
* I've implemented security controls in production
* I document failures and lessons learned
* I prioritize privacy, automation, and defense-in-depth.
* **My goal is Purple Teaming: designing robust defenses while actively simulating attacks to validate their effectiveness.**



## 📚 Projects Overview

| # | Project | Tech Stack | Security Focus |
|---|---------|------------|----------------|
| 1 | [**Rybbit Analytics**](https://github.com/rootGirly/Projects/tree/main/rybbit) | Docker, Caddy, Astro | Self-hosted data ownership, GDPR compliance |
| 2 | [**Ubuntu 24 CIS Hardening**](https://github.com/rootGirly/Projects/tree/main/ansible) | Ansible, Goss | Benchmark compliance, audit automation |
| 3 | [**Apache .htaccess Security**](https://github.com/rootGirly/Projects/tree/main/htaccess) | Apache, Security Headers | Web application hardening, CSP, HSTS |
| 4 | [**Nextcloud Infrastructure**](https://github.com/rootGirly/Projects/tree/main/NextCloud) | Docker, MariaDB, Redis, Caddy | Secure cloud deployment, backup automation |
| 5 | [**Pi-hole Network Defense**](https://github.com/rootGirly/Projects/tree/main/pi-hole) | Raspberry Pi, DNS, UFW | Network-level threat blocking, Fail2Ban |
| 6 | [**Automated Email System**](https://github.com/rootGirly/Projects/tree/main/React-email) | React Email, Resend, Docker | Secure credential handling, template automation |
| 7 | [**Proxmox Virtualization**](https://github.com/rootGirly/Projects/tree/main/proxmox) | KVM, LXC, Debian | Isolated environments, sandboxing, snapshots |
| 8 | [**Restic Backup System**](https://github.com/rootGirly/Projects/tree/main/restic) | Restic, Cron, SFTP| Encrypted offsite backups, retention policies, disaster recovery |
| 9 | [**SiteOne Web Auditor**](https://github.com/rootGirly/Projects/tree/main/SiteOneCrawler) | CLI, HTML Reports | Rapid security scanning, header validation, compliance checks |
| 10 | [**Secure Offsite Backups: Architecture**](https://github.com/rootGirly/Projects/tree/main/sftp) | Proxmox LXC, WireGuard, Restic, SFTP | Network isolation, encrypted tunneling, least privilege access |
| 11 | [**Secure Offsite Backups: Hardening**](https://github.com/rootGirly/Projects/blob/main/sftp/Verification%20%26%20Hardening.md) | SSH Hardening, Fail2Ban, UFW, WireGuard | Defense-in-depth, brute-force protection, SFTP-only access |


## 🛡️ Security Skills learned

### Infrastructure Security
- **System Hardening:** CIS Benchmarks, UFW firewall, Fail2Ban, SSH key-only authentication
- **Network Security:** DNS sinkholing, port restriction, VLAN isolation concepts, **WireGuard tunneling**, **network segmentation**
- **Virtualization:** KVM/LXC isolation, secure VM configuration, snapshot management, **containerized security gateways**
- **Disaster Recovery:** Encrypted offsite backups, isolated backup architectures, retention policy automation

### Application Security
- **Web Security:** Security headers (CSP, HSTS, X-Frame-Options), HTTPS enforcement
- **Authentication:** Secret management, environment variable security, API key handling, **SSH key-based authentication**, **SFTP-only user restrictions**
- **Data Protection:** Encrypted backups, database security, TLS everywhere, **credential file hardening, deduplicated encryption (Restic).

### Automation & DevSecOps
- **Infrastructure as Code:** Ansible playbooks, Docker Compose, automated backup scripts, cron job scheduling
- **CI/CD Concepts:** Automated deployments, configuration management
- **Monitoring:** Log analysis, audit trails, alerting concepts, intrusion detection (Fail2Ban), authentication failure monitoring.

### Privacy & Compliance
- **GDPR-Friendly:** Self-hosted alternatives to third-party services
- **Data Sovereignty**: Full control over data location and retention, geographically isolated backups
- **Open Source:** Preference for auditable, community-driven solutions
- **Operational Security:** Principle of Least Privilege, defense-in-depth architecture, secure credential storage practices

### Advanced Security Concepts
- **Zero Trust Networking:** Isolated backup servers unreachable from public internet
- **Threat Mitigation:** Brute-force attack prevention, lateral movement restriction, privilege escalation containment.
- **Resilience Engineering:** Multi-layer backup verification, tested disaster recovery procedures


## 🎓 Learning Philosophy

**Build, document, and share. Then break and test your own system security.**


## 🚀 For Recruiters & Hiring Managers

If you're evaluating my candidacy for security roles, here's what this portfolio demonstrates:

| Competency | Evidence |
|------------|----------|
| **Technical Depth** | 7 production deployments with documented configurations |
| **Security Mindset** | Hardening, auditing, defense-in-depth across all projects |
| **Purple Team Interest** | Building defensive architectures specifically to validate them against adversarial tactics |
| **Communication** | Clear documentation with diagrams, code, and screenshots |
| **Problem Solving** | Real troubleshooting (port conflicts, certificate issues, compatibility) |
| **Initiative** | Self-directed learning, no formal training required |

**Career Goal:** I am actively seeking **Junior Security Engineer**, **Infrastructure Security**, or **Purple Team** roles where I can leverage my hands-on experience in building secure systems to identify vulnerabilities, simulate attacks, and strengthen overall resilience.



## 📬 Contact & Opportunities

I'm actively seeking **Security Engineer**, or **Infrastructure Security**, **Cloud Security** roles where I can contribute my hands-on experience while continuing to grow.

- 📧 Email: rootgirl@protonmail.com 
> **OPSEC Note:** I use a pseudonym to maintain operational security. This reflects my commitment to privacy-first practices—a value I bring to security roles.

- Website : coming soon.


## 🙏 Acknowledgments

Special thanks to all the open-source communities/
Without these projects, none of this would be possible.


**Security isn't a product, it's a process. I believe the best way to learn it is to build systems myself and then test whether they hold up.**


