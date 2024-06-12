# Ansible Scenario-Based Interview Questions for Professionals

This document provides a collection of scenario-based interview questions for seasoned Ansible professionals. Each scenario is designed to test practical knowledge, strategic thinking, and problem-solving skills in real-world situations.

## 1. Multi-Tier Application Deployment
### Scenario:
You need to deploy a multi-tier application consisting of a web server, application server, and database server.
### Question:
How would you design your Ansible playbooks and roles to handle this deployment efficiently?

### Answer:
To handle a multi-tier application deployment efficiently, I would use a modular approach with Ansible roles. Each tier (web server, application server, database server) would have its own role. The directory structure might look like this:

```sh
site.yml
roles/
  web/
    tasks/
      main.yml
    templates/
      web.conf.j2
  app/
    tasks/
      main.yml
    templates/
      app.conf.j2
  db/
    tasks/
      main.yml
    templates/
      db.conf.j2
inventory/
  production/
    hosts
    group_vars/
      all.yml
      web.yml
      app.yml
      db.yml

```

In site.yml, I would include all the roles:

```
- hosts: web
  roles:
    - role: web

- hosts: app
  roles:
    - role: app

- hosts: db
  roles:
    - role: db

```

This structure promotes reusability and ease of management.


## 2. Rolling Updates with Zero Downtime
### Scenario:
You are required to implement a rolling update for an application to ensure zero downtime.
### Question:
Explain how you would achieve this using Ansible.
### Answer:
To implement a rolling update with zero downtime, I would use a combination of Ansible playbooks and a load balancer. The process involves updating a subset of servers at a time while ensuring the load balancer only directs traffic to healthy nodes. Here’s a high-level approach:

Drain Traffic: Use Ansible to interact with the load balancer API to drain traffic from the first subset of servers.

Update Servers: Apply the updates to the drained servers.

Health Check: Ensure the updated servers pass health checks.

Restore Traffic: Add the updated servers back to the load balancer.

Repeat: Move on to the next subset of servers and repeat the process.

Example playbook:

```
- hosts: web_servers
  serial: 2
  tasks:
    - name: Drain traffic from web server
      uri:
        url: "http://loadbalancer/api/drain/{{ inventory_hostname }}"
        method: POST

    - name: Update web server
      include_role:
        name: web

    - name: Perform health check
      uri:
        url: "http://{{ inventory_hostname }}/health"
        status_code: 200
        retries: 5
        delay: 10

    - name: Add web server back to load balancer
      uri:
        url: "http://loadbalancer/api/add/{{ inventory_hostname }}"
        method: POST

```

## 3. Handling Secret Management
### Scenario:
You need to manage sensitive information such as passwords or API keys in your Ansible playbooks.
### Question: 
How do you securely handle this data?
### Answer:
For managing sensitive information in Ansible, I use Ansible Vault. Ansible Vault allows you to encrypt sensitive data within YAML files.

To encrypt a file:

```
ansible-vault encrypt secrets.yml
```

To use the encrypted secrets in a playbook:

```
- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Use secret API key
      uri:
        url: "https://api.example.com/data"
        headers:
          Authorization: "Bearer {{ api_key }}"

```

To run the playbook with the vault password:

```
ansible-playbook playbook.yml --ask-vault-pass
```


## 4. Idempotency Issues
### Scenario:
During a deployment, you notice that one of your tasks is not idempotent.
### Question:
How do you identify and fix this issue?
### Answer:
Idempotency means that running a task multiple times should have the same effect as running it once. To identify and fix idempotency issues:

Identify the Task: Determine which task is causing the issue by reviewing the output of ansible-playbook with the -v (verbose) flag.

Analyze the Task: Check if the task is correctly checking for the desired state before making changes. For example, ensure that file changes, service restarts, or package installations are conditional.

Fix the Task: Modify the task to check for the current state before making changes.

Example of a non-idempotent task:

```
- name: Add line to file
  lineinfile:
    path: /etc/example.conf
    line: "new configuration"

```

Fixed idempotent task:

```
- name: Add line to file if not present
  lineinfile:
    path: /etc/example.conf
    line: "new configuration"
    state: present
```


## 5. Troubleshooting Playbook Failures
### Scenario:
Your playbook worked previously, but now it’s failing.
### Question:
Describe your approach to troubleshooting a failing playbook that worked previously.
### Answer:
To troubleshoot a failing playbook:

Check Recent Changes: Review recent changes in the playbook, roles, or inventory files.

Verbose Output: Run the playbook with increased verbosity (-vvv) to get detailed output and identify where it fails.

Environment Consistency: Ensure the environment where the playbook is run hasn’t changed (e.g., different Ansible version, OS updates, or network configurations).

Isolate the Issue: Isolate the failing task by running it independently or within a minimal playbook.

Logs and Debugging: Use Ansible’s debug module to print variables and outputs at various stages.

External Dependencies: Check for external dependencies (e.g., repositories, external services) that might be causing the failure.

Example of using the debug module:

```
- name: Print variable value
  debug:
    var: my_variable

```

## 6. Ansible Dynamic Inventory
### Scenario:
You need to manage a dynamic infrastructure where servers are frequently added or removed.
### Question: 
How would you implement dynamic inventory in Ansible?
### Answer:
To manage a dynamic infrastructure, I would use Ansible’s dynamic inventory feature. This can be achieved by using inventory scripts or plugins that query external data sources such as cloud provider APIs (e.g., AWS, Azure).

Example using AWS EC2 dynamic inventory:

Install the AWS Inventory Plugin:

```
pip install boto boto3
```

Configure the AWS Inventory Plugin:

```
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production

```

Use the Dynamic Inventory in Playbooks:

```
ansible-playbook -i aws_ec2.yml playbook.yml

```


## 7. Handling Configuration Drift
### Scenario:
Over time, the configuration of servers might drift from the desired state defined in Ansible playbooks.
### Question: 
How do you ensure servers remain in the desired state?
### Answer:
To ensure servers remain in the desired state, I would implement regular configuration enforcement using Ansible Tower/AWX or a cron job that runs playbooks periodically. Additionally, I would use Ansible’s check mode to detect drift without making changes.

Example cron job:

```
0 2 * * * ansible-playbook -i inventory playbook.yml
```


## 8. Optimizing Playbook Performance
### Scenario:
Your playbook takes too long to execute.
### Question:
How do you optimize the performance of your Ansible playbooks?
### Answer:
To optimize playbook performance:

Parallelism: Increase the number of forks to run tasks in parallel.

```
ansible-playbook -i inventory playbook.yml -f 10
```

Delegate Tasks: Delegate tasks to appropriate hosts to distribute the load.

```
- name: Fetch something
  delegate_to: localhost
```

Limit Scope: Use --limit to target specific hosts.

```
ansible-playbook -i inventory playbook.yml --limit web_servers
```

Asynchronous Tasks: Use asynchronous tasks for long-running operations.

```
- name: Long running task
  command: /path/to/long_running_command
  async: 3600
  poll: 0

- name: Wait for long running task to finish
  async_status:
    jid: "{{ async_task_result.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 30
  delay: 60

```


## 9. Handling Dependencies
Scenario: 
Your playbook depends on multiple external roles and collections.
### Question:
How do you manage these dependencies efficiently?
### Answer:
I would use requirements.yml to manage external roles and collections. This file lists all dependencies, which can be installed using ansible-galaxy.

Example requirements.yml:
```
roles:
  - src: geerlingguy.nginx
  - src: geerlingguy.mysql
collections:
  - name
```

To install dependencies:

```
ansible-galaxy install -r requirements.yml
```

## 10. Ansible and CI/CD Integration
### Scenario: 
You want to integrate Ansible with your CI/CD pipeline.
### Question: 
How would you achieve this?
### Answer:
To integrate Ansible with a CI/CD pipeline, I would use tools like Jenkins, GitLab CI, or GitHub Actions. The CI/CD pipeline would trigger Ansible playbooks to deploy or update infrastructure.

Example using GitLab CI:

```
stages:
  - deploy

deploy:
  stage: deploy
  script:
    - ansible-playbook -i inventory playbook.yml
```


## 11. Error Handling in Playbooks
### Scenario: 
Some tasks in your playbook are prone to failure.
### Question: 
How do you handle errors in Ansible playbooks?
### Answer:
To handle errors:

Ignore Errors: Use ignore_errors: yes for non-critical tasks.

```
- name: Task that might fail
  command: /bin/false
  ignore_errors: yes
```

Retries: Use retries and delay for tasks that might fail intermittently.

```
- name: Retry task
  command: /path/to/command
  retries: 5
  delay: 10
  until: result.rc == 0
  register: result
```

Rescue and Always: Use block, rescue, and always for structured error handling.

```
- name: Structured error handling
  block:
    - name: Task that might fail
      command: /bin/false
  rescue:
    - name: Handle failure
      debug:
        msg: "Task failed"
  always:
    - name: Always run
      debug:
        msg: "This always runs"
```


## 12. Configuring Ansible for Network Automation
### Scenario: 
You need to automate network device configurations.
### Question: 
How do you configure Ansible for network automation?
### Answer:
For network automation, I would use Ansible network modules and collections like ansible.netcommon and vendor-specific collections.

Example playbook for Cisco devices:

```
- hosts: cisco_routers
  gather_facts: no
  tasks:
    - name: Configure interface
      cisco.ios.ios_interface:
        name: GigabitEthernet1
        description: "Configured by Ansible"
        enabled: yes
```


## 13. Ansible in a Multi-Cloud Environment
### Scenario: 
You manage resources across multiple cloud providers.
### Question: 
How do you manage multi-cloud environments with Ansible?
### Answer:
To manage multi-cloud environments, I would use dynamic inventories and cloud-specific modules for each provider (AWS, Azure, GCP).

Example using multiple dynamic inventories:

```
plugin: aws_ec2
regions:
  - us-east-1

plugin: azure_rm
```

```
ansible-playbook -i aws_ec2.yml -i azure_rm.yml playbook.yml
```


## 14. Migrating Legacy Scripts to Ansible
### Scenario: 
You have legacy shell scripts for server management.
### Question: 
How do you migrate these scripts to Ansible playbooks?
### Answer:
To migrate legacy scripts to Ansible:

Identify Tasks: Break down the shell script into discrete tasks.

Use Ansible Modules: Replace shell commands with equivalent Ansible modules.

Structure Playbooks: Organize tasks into roles and playbooks for better management.

Example migration:

Original shell script:

```
#!/bin/bash
apt-get update
apt-get install -y nginx
echo "Hello, World!" > /var/www/html/index.html
```

Migrated Ansible playbook:


```
- hosts: web
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Create index.html
      copy:
        content: "Hello, World!"
        dest: /var/www/html/index.html
```


## 15. Managing Multiple Environments
### Scenario: 
You need to manage different environments (development, staging, production) with Ansible.
### Question: 
How do you organize your Ansible configuration to handle multiple environments?
### Answer:
To manage multiple environments, I would use separate inventory files and group variables for each environment.

Example directory structure:

```
inventories/
  development/
    hosts
    group_vars/
      all.yml
  staging/
    hosts
    group_vars/
      all.yml
  production/
    hosts
    group_vars/
      all.yml
site.yml
```

In site.yml, specify the inventory file based on the environment:

```
ansible-playbook -i inventories/development/hosts site.yml
ansible-playbook -i inventories/staging/hosts site.yml
ansible-playbook -i inventories/production/hosts site.yml
```

This approach keeps environment-specific configurations separate and manageable.


