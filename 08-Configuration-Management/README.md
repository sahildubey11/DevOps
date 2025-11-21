# Configuration Management

## What is Configuration Management?

Configuration Management is the practice of handling changes systematically so that a system maintains its integrity over time. It ensures consistency across infrastructure.

## Why Configuration Management?

- **Consistency**: Ensure all servers are configured identically
- **Automation**: Reduce manual configuration errors
- **Scalability**: Manage hundreds or thousands of servers
- **Compliance**: Maintain security and compliance standards
- **Documentation**: Code serves as documentation
- **Disaster Recovery**: Quickly rebuild infrastructure

## Configuration Management Tools

1. **Ansible**: Agentless, YAML-based, push model
2. **Puppet**: Agent-based, declarative, pull model
3. **Chef**: Agent-based, Ruby DSL, pull model
4. **SaltStack**: Agent-based or agentless, Python

## Ansible Deep Dive

### Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# macOS
brew install ansible

# Python pip
pip install ansible
```

### Inventory Management

**Static Inventory (inventory.ini)**
```ini
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11

[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[loadbalancers]
lb.example.com ansible_host=192.168.1.5

[production:children]
webservers
databases
loadbalancers

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

**Dynamic Inventory (AWS)**
```python
#!/usr/bin/env python3
import boto3
import json

def get_inventory():
    ec2 = boto3.client('ec2', region_name='us-east-1')
    response = ec2.describe_instances()
    
    inventory = {
        '_meta': {'hostvars': {}},
        'all': {'children': ['ungrouped']}
    }
    
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] == 'running':
                public_ip = instance.get('PublicIpAddress')
                if public_ip:
                    tags = {tag['Key']: tag['Value'] for tag in instance.get('Tags', [])}
                    env = tags.get('Environment', 'ungrouped')
                    
                    if env not in inventory:
                        inventory[env] = {'hosts': []}
                    inventory[env]['hosts'].append(public_ip)
                    
                    inventory['_meta']['hostvars'][public_ip] = {
                        'ansible_host': public_ip,
                        'instance_id': instance['InstanceId'],
                        'instance_type': instance['InstanceType']
                    }
    
    return inventory

if __name__ == '__main__':
    print(json.dumps(get_inventory(), indent=2))
```

### Complete Playbook Example

```yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    app_user: webapp
    app_directory: /var/www/myapp
    ssl_enabled: true

  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

  tasks:
    - name: Install required packages
      apt:
        name:
          - nginx
          - python3-pip
          - git
          - ufw
        state: present

    - name: Create application user
      user:
        name: "{{ app_user }}"
        shell: /bin/bash
        create_home: yes

    - name: Create application directory
      file:
        path: "{{ app_directory }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'

    - name: Copy Nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
      notify: Restart Nginx

    - name: Enable Nginx site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Restart Nginx

    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - 22
        - 80
        - 443

    - name: Enable firewall
      ufw:
        state: enabled

    - name: Install SSL certificate (if enabled)
      block:
        - name: Install certbot
          apt:
            name: certbot
            state: present

        - name: Generate SSL certificate
          command: certbot certonly --nginx -d example.com --non-interactive --agree-tos -m admin@example.com
          args:
            creates: /etc/letsencrypt/live/example.com/fullchain.pem
      when: ssl_enabled

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted

  post_tasks:
    - name: Verify Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

### Advanced Ansible Features

**Ansible Roles Structure**
```
roles/
  webserver/
    tasks/
      main.yml
      install.yml
      configure.yml
    handlers/
      main.yml
    templates/
      nginx.conf.j2
    files/
      index.html
    vars/
      main.yml
    defaults/
      main.yml
    meta/
      main.yml
```

**Using Roles**
```yaml
---
- name: Setup Infrastructure
  hosts: all
  become: yes
  
  roles:
    - role: common
      tags: ['common']
    
    - role: security
      tags: ['security']
    
    - role: webserver
      when: "'webservers' in group_names"
      tags: ['webserver']
    
    - role: database
      when: "'databases' in group_names"
      tags: ['database']
```

**Ansible Galaxy**
```bash
# Install role from Galaxy
ansible-galaxy install geerlingguy.nginx

# Create role structure
ansible-galaxy init my-role

# Install from requirements.yml
ansible-galaxy install -r requirements.yml
```

**requirements.yml**
```yaml
roles:
  - name: geerlingguy.nginx
    version: 3.1.4
  
  - name: geerlingguy.docker
    version: 4.1.3

collections:
  - name: community.general
    version: 4.0.0
```

### Ansible Vault (Secrets Management)

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/production.yml

# Decrypt file
ansible-vault decrypt vars/production.yml

# View encrypted file
ansible-vault view secrets.yml

# Change vault password
ansible-vault rekey secrets.yml
```

**Using Vault in Playbook**
```yaml
---
- name: Deploy Application
  hosts: webservers
  become: yes
  
  vars_files:
    - vars/common.yml
    - vault/secrets.yml  # Encrypted file
  
  tasks:
    - name: Configure database connection
      template:
        src: config.j2
        dest: /app/config.yml
      vars:
        db_password: "{{ vault_db_password }}"
```

**Run with Vault**
```bash
ansible-playbook playbook.yml --ask-vault-pass
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

## Puppet

### Basic Manifest

**site.pp**
```puppet
node 'webserver.example.com' {
  # Install Apache
  package { 'apache2':
    ensure => installed,
  }

  # Manage Apache service
  service { 'apache2':
    ensure  => running,
    enable  => true,
    require => Package['apache2'],
  }

  # Manage configuration file
  file { '/etc/apache2/sites-available/myapp.conf':
    ensure  => file,
    content => template('myapp/apache.conf.erb'),
    notify  => Service['apache2'],
  }

  # Create application directory
  file { '/var/www/myapp':
    ensure => directory,
    owner  => 'www-data',
    group  => 'www-data',
    mode   => '0755',
  }
}
```

### Puppet Module Structure
```
modules/
  apache/
    manifests/
      init.pp
      install.pp
      configure.pp
    templates/
      httpd.conf.erb
    files/
      index.html
    lib/
    tests/
    metadata.json
```

## Chef

### Recipe Example

**default.rb**
```ruby
# Install package
package 'nginx' do
  action :install
end

# Manage service
service 'nginx' do
  action [:enable, :start]
  supports :restart => true, :reload => true
end

# Create directory
directory '/var/www/myapp' do
  owner 'www-data'
  group 'www-data'
  mode '0755'
  action :create
end

# Manage configuration file
template '/etc/nginx/sites-available/myapp' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  notifies :restart, 'service[nginx]'
end
```

## Configuration Management Best Practices

### 1. Infrastructure as Code
- Version control all configurations
- Use meaningful commit messages
- Code review changes
- Test before deploying

### 2. Idempotency
- Ensure configurations can run multiple times
- Check current state before making changes
- Use appropriate modules/resources

### 3. Modularity
- Create reusable roles/modules
- Follow DRY (Don't Repeat Yourself)
- Separate concerns
- Use parameterization

### 4. Security
- Encrypt sensitive data
- Use secrets management
- Follow least privilege
- Regular security updates

### 5. Testing
- Test in development environment
- Use check mode/dry run
- Validate syntax
- Test edge cases

### 6. Documentation
- Document roles and playbooks
- Add comments for complex logic
- Maintain README files
- Keep inventory updated

### 7. Environment Separation
- Separate dev, staging, production
- Use different inventories
- Environment-specific variables
- Controlled promotion

## Comparison of Tools

| Feature | Ansible | Puppet | Chef | SaltStack |
|---------|---------|--------|------|-----------|
| Architecture | Agentless | Agent-based | Agent-based | Both |
| Language | YAML | Puppet DSL | Ruby DSL | YAML/Python |
| Model | Push | Pull | Pull | Both |
| Learning Curve | Easy | Moderate | Steep | Moderate |
| Scalability | Good | Excellent | Excellent | Excellent |
| Windows Support | Good | Good | Good | Good |

## Interview Questions

1. What is configuration management?
2. Explain the difference between push and pull models
3. What is idempotency and why is it important?
4. How do you manage secrets in Ansible?
5. What are Ansible roles and why use them?
6. Explain the difference between Ansible playbooks and ad-hoc commands
7. How do you handle different environments in configuration management?
8. What is the purpose of Ansible Galaxy?
9. How do you test Ansible playbooks?
10. What are handlers in Ansible?

## Practical Exercises

1. Create an Ansible playbook to configure web servers
2. Build reusable Ansible roles
3. Set up dynamic inventory for cloud resources
4. Implement secrets management with Ansible Vault
5. Create a complete infrastructure configuration
6. Configure multiple environments (dev, staging, prod)
7. Implement testing for configuration code
8. Set up continuous configuration management

## Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Puppet Documentation](https://puppet.com/docs/)
- [Chef Documentation](https://docs.chef.io/)
- [SaltStack Documentation](https://docs.saltproject.io/)
- [Ansible for DevOps (Book)](https://www.ansiblefordevops.com/)
