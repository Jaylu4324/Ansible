# Docker Installation on Ubuntu using Ansible

## 1. Update Package Cache

```yaml
- name: Update Packages
  apt:
    update_cache: yes
```

### What?

Updates the APT package index.

### Why?

Ensures Ubuntu has the latest package information before installing Docker.

### How?

Runs the equivalent of:

```bash
sudo apt update
```

---

## 2. Install Docker

```yaml
- name: Installing Docker in Ubuntu
  apt:
    name: docker.io
    state: present
  notify:
    - Enable the docker service
    - Start the docker service
```

### What?

Installs Docker.

### Why?

Downloads and installs the `docker.io` package from Ubuntu's repository.

### How?

Uses the `apt` module. After installation, it **notifies the handlers** to enable and start the Docker service.

---

## 3. Handlers

```yaml
- name: Enable the docker service
  service:
    name: docker
    enabled: yes

- name: Start the docker service
  service:
    name: docker
    state: started
```

### What?

* **Enable** → Starts Docker automatically after every system reboot.
* **Start** → Starts the Docker service immediately.

### Why use Handlers?

Handlers run **only when notified** (for example, after Docker is installed or its configuration changes). This avoids restarting or enabling the service unnecessarily every time the playbook runs.

---

## 4. Add User to Docker Group

```yaml
- name: Append user to the docker group
  user:
    name: "{{ ansible_user }}"
    groups: docker
    append: yes
```

### What?

Adds the current user to the `docker` group.

### Why?

Allows running Docker commands without using `sudo`.

### How?

The `user` module adds the user to the `docker` group while keeping existing group memberships.

