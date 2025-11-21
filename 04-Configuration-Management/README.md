# Configuration Management

## What is Configuration Management?

Configuration Management (CM) is the process of systematically handling changes to a system in a way that maintains integrity over time. It involves automating the deployment and configuration of servers, applications, and infrastructure.

## Why Configuration Management?

### Benefits
- **Consistency**: Same configuration across all environments
- **Automation**: Reduce manual errors
- **Scalability**: Manage hundreds or thousands of servers
- **Version Control**: Track configuration changes
- **Disaster Recovery**: Quickly rebuild infrastructure
- **Documentation**: Code serves as documentation
- **Compliance**: Ensure security and compliance standards

### Problems It Solves
- Configuration drift
- Snowflake servers (unique, undocumented configurations)
- Manual configuration errors
- Time-consuming deployments
- Inconsistent environments

## Configuration Management vs Infrastructure as Code

While related, they have different focuses:

- **Configuration Management**: Manages software and configuration on existing servers
- **Infrastructure as Code**: Provisions and manages the infrastructure itself

Many tools do both (e.g., Terraform + Ansible).

## Key Concepts

### Idempotency
Running the same configuration multiple times produces the same result. No matter how many times you apply the configuration, the outcome is consistent.

### Declarative vs Imperative

**Declarative** (What):
```yaml
# Desired state
package:
  name: nginx
  state: present
```

**Imperative** (How):
```bash
# Step-by-step instructions
if ! rpm -q nginx; then
  yum install -y nginx
fi
```

### Push vs Pull

**Push Model**:
- Control server pushes configurations to nodes
- Examples: Ansible, SaltStack (can do both)

**Pull Model**:
- Nodes pull configurations from central server
- Examples: Puppet, Chef

## Popular Configuration Management Tools

### 1. Ansible

**Type**: Agentless, Push-based, Declarative

**Key Features**:
- No agent required (uses SSH)
- YAML-based playbooks
- Large module library
- Easy to learn
- Python-based

**Architecture**:
```
Control Node → SSH → Managed Nodes
```

**Example Playbook**:
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy configuration file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Inventory File** (`hosts.ini`):
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Running Ansible**:
```bash
# Run playbook
ansible-playbook -i hosts.ini playbook.yml

# Ad-hoc command
ansible webservers -i hosts.ini -m ping

# Check mode (dry run)
ansible-playbook playbook.yml --check

# Limit to specific hosts
ansible-playbook playbook.yml --limit web1.example.com
```

### 2. Puppet

**Type**: Agent-based, Pull-based, Declarative

**Key Features**:
- Master-agent architecture
- Ruby-based DSL
- Strong type system
- Large module ecosystem (Puppet Forge)
- Enterprise support

**Architecture**:
```
Puppet Master ← Pull (periodic) ← Puppet Agents
```

**Example Manifest**:
```puppet
# site.pp or module manifest
node 'webserver.example.com' {
  class { 'nginx':
    ensure => present,
  }
  
  package { 'nginx':
    ensure => installed,
  }
  
  service { 'nginx':
    ensure => running,
    enable => true,
    require => Package['nginx'],
  }
  
  file { '/etc/nginx/nginx.conf':
    ensure  => file,
    source  => 'puppet:///modules/nginx/nginx.conf',
    require => Package['nginx'],
    notify  => Service['nginx'],
  }
}
```

**Module Structure**:
```
nginx/
├── manifests/
│   └── init.pp
├── files/
│   └── nginx.conf
├── templates/
│   └── site.conf.erb
├── tests/
└── metadata.json
```

### 3. Chef

**Type**: Agent-based, Pull-based, Imperative (Ruby DSL)

**Key Features**:
- Ruby-based recipes
- Test-driven infrastructure
- Powerful, but complex
- Chef Supermarket for cookbooks

**Architecture**:
```
Chef Server ← Pull (periodic) ← Chef Clients
```

**Example Recipe**:
```ruby
# recipes/default.rb
package 'nginx' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
  supports restart: true, reload: true
end

template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  notifies :reload, 'service[nginx]', :delayed
end

directory '/var/www/html' do
  owner 'www-data'
  group 'www-data'
  mode '0755'
  action :create
end
```

**Cookbook Structure**:
```
nginx/
├── recipes/
│   └── default.rb
├── templates/
│   └── nginx.conf.erb
├── attributes/
│   └── default.rb
├── files/
├── metadata.rb
└── README.md
```

**Knife Commands**:
```bash
# Bootstrap a node
knife bootstrap node.example.com -x ubuntu -i key.pem --sudo

# Upload cookbook
knife cookbook upload nginx

# List nodes
knife node list

# Run chef-client on all nodes
knife ssh 'name:*' 'sudo chef-client' -x ubuntu
```

### 4. SaltStack (Salt)

**Type**: Agent-based, Both Push and Pull, Declarative

**Key Features**:
- Fast and scalable (ZeroMQ)
- Python-based
- Event-driven automation
- Both push and pull modes

**Architecture**:
```
Salt Master ← ZeroMQ → Salt Minions
```

**Example State File** (`nginx.sls`):
```yaml
nginx:
  pkg.installed: []
  service.running:
    - enable: True
    - require:
      - pkg: nginx

/etc/nginx/nginx.conf:
  file.managed:
    - source: salt://nginx/files/nginx.conf
    - user: root
    - group: root
    - mode: 644
    - require:
      - pkg: nginx
    - watch_in:
      - service: nginx
```

**Top File** (`top.sls`):
```yaml
base:
  'webserver*':
    - nginx
    - php
  'database*':
    - mysql
```

**Salt Commands**:
```bash
# Test connectivity
salt '*' test.ping

# Apply state
salt '*' state.apply nginx

# Run command
salt 'web*' cmd.run 'uptime'

# Gather facts
salt '*' grains.items
```

## Ansible Deep Dive

### Ansible Modules

Common modules:
- `apt/yum`: Package management
- `service`: Service management
- `copy/template`: File operations
- `user/group`: User management
- `command/shell`: Execute commands
- `git`: Git operations
- `docker_container`: Docker management

### Ansible Roles

**Role Structure**:
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── config.j2
    ├── files/
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    └── meta/
        └── main.yml
```

**Using Roles**:
```yaml
---
- name: Configure servers
  hosts: all
  roles:
    - common
    - webserver
    - monitoring
```

### Ansible Variables

**Defining Variables**:
```yaml
# In playbook
vars:
  http_port: 80
  domain_name: example.com

# In separate file
vars_files:
  - vars/external_vars.yml

# In inventory
[webservers:vars]
http_port=8080
```

**Using Variables**:
```yaml
- name: Configure port
  lineinfile:
    path: /etc/config
    line: "port={{ http_port }}"
```

### Ansible Templates (Jinja2)

**Template File** (`nginx.conf.j2`):
```jinja2
server {
    listen {{ http_port }};
    server_name {{ domain_name }};
    
    {% for location in locations %}
    location {{ location.path }} {
        proxy_pass {{ location.backend }};
    }
    {% endfor %}
}
```

### Ansible Vault (Secrets)

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt vars.yml

# Use vault in playbook
ansible-playbook playbook.yml --ask-vault-pass
```

### Ansible Galaxy

Repository for community roles:

```bash
# Install role
ansible-galaxy install geerlingguy.nginx

# Create role skeleton
ansible-galaxy init my-role

# Install from requirements file
ansible-galaxy install -r requirements.yml
```

**requirements.yml**:
```yaml
- name: geerlingguy.nginx
  version: 3.1.4

- src: https://github.com/user/custom-role
  name: custom-role
```

## Best Practices

### 1. Version Control
- Store all configuration code in Git
- Use branches for testing changes
- Code review for configuration changes
- Tag releases

### 2. Testing

**Test Kitchen** (Chef/Ansible):
```yaml
# .kitchen.yml
driver:
  name: vagrant

provisioner:
  name: ansible_playbook
  playbook: test.yml

platforms:
  - name: ubuntu-20.04
  - name: centos-8

suites:
  - name: default
    verifier:
      name: inspec
```

**Molecule** (Ansible):
```bash
# Initialize molecule
molecule init role my-role

# Test sequence
molecule create
molecule converge
molecule verify
molecule destroy

# Full test
molecule test
```

### 3. Idempotency
Always ensure configurations are idempotent:

```yaml
# Good - Idempotent
- name: Ensure line in file
  lineinfile:
    path: /etc/config
    line: "setting=value"

# Bad - Not idempotent
- name: Append to file
  shell: echo "setting=value" >> /etc/config
```

### 4. Documentation
- Comment complex logic
- Maintain README files
- Document variables and their purposes
- Keep playbooks/manifests readable

### 5. Security
- Use Ansible Vault for secrets
- Rotate credentials regularly
- Principle of least privilege
- Audit trail for changes

### 6. Organization

**Directory Structure**:
```
ansible/
├── inventories/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── group_vars/
│   └── all.yml
├── host_vars/
└── ansible.cfg
```

### 7. Use Variables and Templates
- Avoid hardcoding values
- Use templates for configuration files
- Separate environment-specific values

### 8. Error Handling

```yaml
- name: Attempt risky operation
  command: /usr/bin/risky-command
  register: result
  ignore_errors: yes

- name: Handle failure
  debug:
    msg: "Command failed: {{ result.stderr }}"
  when: result.failed
```

## Configuration Management Workflow

1. **Develop**: Write configuration code
2. **Test**: Test in development environment
3. **Review**: Code review process
4. **Stage**: Apply to staging environment
5. **Validate**: Verify staging works correctly
6. **Deploy**: Apply to production
7. **Monitor**: Watch for issues

## Comparison Matrix

| Feature | Ansible | Puppet | Chef | Salt |
|---------|---------|--------|------|------|
| Agent | No | Yes | Yes | Yes |
| Language | YAML | Ruby DSL | Ruby | YAML |
| Model | Push | Pull | Pull | Both |
| Learning Curve | Easy | Medium | Hard | Medium |
| Speed | Medium | Medium | Medium | Fast |
| Scalability | Good | Excellent | Excellent | Excellent |
| Community | Large | Large | Large | Medium |

## Use Cases

### Ansible
- Simple automation tasks
- Multi-tier deployments
- Application deployment
- Ad-hoc tasks

### Puppet
- Large-scale infrastructure
- Compliance enforcement
- Long-term configuration management
- Enterprise environments

### Chef
- Complex infrastructure
- Test-driven infrastructure
- Cloud deployments
- DevOps workflows

### Salt
- High-performance requirements
- Real-time remote execution
- Event-driven automation
- Cloud orchestration

## Common Patterns

### Rolling Updates
```yaml
- name: Rolling update
  hosts: webservers
  serial: 1  # One at a time
  
  tasks:
    - name: Remove from load balancer
      # Remove from LB
    
    - name: Update application
      # Deploy new version
    
    - name: Add back to load balancer
      # Add back to LB
```

### Blue-Green Deployment
```yaml
- name: Deploy to green environment
  hosts: green
  tasks:
    - name: Deploy application
      # Deploy code

- name: Switch traffic to green
  hosts: loadbalancer
  tasks:
    - name: Update LB config
      # Point to green
```

## Resources

- Ansible Documentation: https://docs.ansible.com/
- Puppet Documentation: https://puppet.com/docs/
- Chef Documentation: https://docs.chef.io/
- SaltStack Documentation: https://docs.saltproject.io/
- Ansible Galaxy: https://galaxy.ansible.com/
- Puppet Forge: https://forge.puppet.com/

## Next Steps

Continue to [Containerization (Docker)](../05-Containerization/) to learn about container technologies.
