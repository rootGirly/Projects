## Getting Started with UBUNTU24-CIS Hardenning

If you’re new to Ansible like I am, all the information out there can feel overwhelming. Here’s how I hardened a fresh VPS and brought it into compliance with the [**CIS Benchmark**](https://www.cisecurity.org/) for Server Level 1.


### Key Repositories

There are two main repositories I needed to work with:

* [UBUNTU24-CIS-Audit](https://github.com/ansible-lockdown/UBUNTU24-CIS-Audit) – This repository contains the Goss audit files. It includes branches like benchmark_v1.0.0 that define the checks you’ll run.

* [UBUNTU24-CIS](https://github.com/ansible-lockdown/UBUNTU24-CIS) – This is the main Ansible role repository. It uses the audit files from UBUNTU24-CIS-Audit to perform the actual system checks.

💡 Key takeaway: I cloned both repositories and make sure the UBUNTU24-CIS role points to the correct audit repository.

[Official Documentation](https://ansible-lockdown.readthedocs.io/en/latest/audit/getting-started-audit.html)

### Key Points: 

One thing that tripped me up at first was realizing that you’re actually working with two separate repositories:

**UBUNTU24-CIS (the role)**

* This is the Ansible role you run. It contains the playbooks, tasks, and logic to apply CIS hardening and audits.

**UBUNTU24-CIS-Audit (the audit content)**

* This repository provides the actual audit files, like goss.yml and run_audit.sh. Without it, the role cannot perform the audit checks.


### What is Goss?
Goss is a YAML based serverspec alternative tool for validating a server's configuration. It eases the process of writing tests by allowing the user to generate tests from the current system state. Once the test suite is written they can be executed, waited-on, or served as a health endpoint.

[Official Documentation](https://goss.readthedocs.io/en/stable/)


On my [LXC PROXMOX](https://pve.proxmox.com/wiki/Linux_Container), I cloned the UBUNTU24-CIS repository into the same folder where I wanted to run the benchmark.

```
mkdir ansible
cd ansible
git clone https://github.com/ansible-lockdown/UBUNTU24-CIS.git
```

```
git clone https://github.com/ansible-lockdown/UBUNTU24-CIS-Audit
git checkout benchmark_v1.0.0
```

My project folder ended up looking like this:

```
ansible/
 ├── UBUNTU24-CIS/           # Main Ansible role repository
 ├── UBUNTU24-CIS-Audit/     # Goss audit files repository
 ├── inventory.ini            # Defines my VPS and connection details
 └── CIS.yaml                 # CIS benchmark configuration
 
``` 


`inventory.ini`

```
[vps]
my_vps_ip ansible_user=root ansible_ssh_private_key_file=~/.ssh/ansible
```
This setup lets me point the Ansible playbooks directly at my VPS for testing and applying the CIS benchmark.

###Creating my CIS Playbook

I created a simple playbook called CIS.yaml to run the CIS audit on my VPS:

```
- name: CIS Audit
  hosts: vps
  become: yes

  roles:
    - role: UBUNTU24-CIS
```

**Configure the Audit**

To make the CIS audit work, there’s a key configuration file that needs to be updated:

`nano UBUNTU24-CIS/defaults/main.yml`

Inside, I focused on the mandatory variables for audit mode. Here’s what my configuration looks like:


```
########################################
# Audit Mode — Mandatory vars for CIS
########################################
run_audit: true

# Where audit content will go
audit_conf_dir: /opt/cis-audit

# Git repo with goss checks
audit_file_git: https://github.com/ansible-lockdown/UBUNTU24-CIS-Audit.git
audit_file_git_branch: benchmark_v1.0.0
audit_git_version: benchmark_v1.0.0

audit_content_path: "/root/ansible/UBUNTU24-CIS-Audit"


# Path to goss binary on VPS
audit_bin: /usr/local/bin/goss

# Download goss if missing
goss_install_url: "https://github.com/goss-org/goss/releases/download/v0.4.9/goss-linux-amd64"
goss_install_path: "{{ audit_bin }}"

# Path to store a copy of default vars for audit
audit_vars_path: /opt/cis-audit/vars.yml

# Max time for a goss command
audit_cmd_timeout: 300

# Output format for audit
audit_format: json

# Pre/post remediation audit files
pre_audit_outfile: /opt/cis-audit/pre_audit.json
post_audit_outfile: /opt/cis-audit/post_audit.json

```

By configuring these variables, I ensured that the Ansible role knows where to find the audit content, how to use Goss, and where to store the results. This step was crucial for running the CIS audit successfully.



### Prepare the VPS for the Audit

Before running the audit, I made sure the required folders and permissions were set on my VPS.

The audit content needs a dedicated directory on my VPS and be sure to add to my be sure to add to my `UBUNTU24-CIS/defaults/main.yml`: 

```
sudo mkdir -p /opt/cis-audit
sudo chown root:root /opt/cis-audit
```
This ensures that Ansible and Goss can write audit results without permission issues.

**Install Goss on the VPS**

Goss is a binary that runs on the managed host, not on the Ansible control machine. I added the installation info in my configuration. (see UBUNTU24-CIS/defaults/main.yml) 

```
git clone https://github.com/ansible-lockdown/UBUNTU24-CIS-Audit.git /opt/cis-audit
cd /opt/cis-audit
git branch -r          # List remote branches
git checkout -b benchmark_v1.0.0

```

Make `/usr/local/bin` is writable

To allow Goss to install correctly, I made sure the directory exists and has the right permissions.

```
sudo mkdir -p /usr/local/bin
sudo chown root:root /usr/local/bin
sudo chmod 755 /usr/local/bin
```
With these steps, the VPS was ready for the CIS audit to run smoothly.


### Execute the audit across an inventory

`ansible-playbook -i inventory.ini CIS.yaml`
Once the audit finishes, I checked the report to see what wasn’t compliant:

###How to Decide What to Apply

`cat /opt/cis-audit/pre_audit.json | jq '.summary'`

Example output:

```
{
  "failed-count": 74,
  "skipped-count": 21,
  "summary-line": "Count: 770, Failed: 74, Skipped: 21, Duration: 0.831s",
  "test-count": 770,
  "total-duration": 831004913
}
```

Here’s what these numbers mean:

* failed-count → Number of controls that are not compliant
* skipped-count → Controls that didn’t run (usually because they don’t apply to your environment)


I run the remediation as I wanted to test on my fresh VPS

```
ansible-playbook -i inventory.ini CIS.yaml \
  --limit myvps \
  --tags level1-server
  
```


**Final Thoughts**

Hardening can really break things if applied blindly. I was lucky to be working on a fresh VPS for testing purposes, which made it safe to experiment. In a production environment, I’d need to be **extremely careful** before applying these changes.

This was really just an opportunity to test the Ansible role in a controlled environment. I could also have used a VM, but since I wanted a system that was compliant from the start, using a fresh VPS felt like the perfect chance to try it out.

After experimenting, I ran into a few real-world effects:

I got logged out of my SSH session after a few minutes of inactivity.
A small website I was running stopped working. This forced me to enable TLS and secure protocols on my Caddy server, which I hadn’t needed before hardening.

Through all of this, I learned a lot about Ansible. Honestly, it can be overwhelming at first, and a beginner-friendly guide would be super helpful. But once you get the hang of it, Ansible is an incredibly powerful tool. I can already feel that mastering it will be a real “superpower” for managing and securing servers.

**Be your own guru.**

