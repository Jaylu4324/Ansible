# Docker Installation on Red Hat using Ansible

## 1. Uninstall Conflicting Packages

```yaml
- name: Uninstall Conflicts
  dnf:
    name:
      - docker
      - docker-client
      - docker-client-latest
      - docker-common
      - docker-latest
      - docker-latest-logrotate
      - docker-logrotate
      - docker-engine
      - podman
      - runc
    state: absent
```

### What?

Removes old Docker and Podman packages.

### Why?

Prevents package conflicts before installing Docker CE.

### How?

The `dnf` module removes the listed packages.

---

## 2. Install DNF Plugins

```yaml
- name: Install Docker plugins
  dnf:
    name: dnf-plugins-core
    state: present
```

### What?

Installs `dnf-plugins-core`.

### Why?

Provides the `dnf config-manager` command.

### How?

The `dnf` module installs the package.

---

## 3. Add Docker Repository

```yaml
- name: Add Docker CE Repo
  command: dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
  args:
    creates: /etc/yum.repos.d/docker-ce.repo
```

### What?

Adds Docker's official repository.

### Why?

Docker CE is not available in the default Red Hat repository.

### How?

* `command` runs the command.
* `args` passes extra options to the command.
* `creates` means:

  * If the file exists → Skip the task.
  * If the file doesn't exist → Run the command.

This prevents adding the repository again every time the playbook runs.

---

## 4. Install Docker CE

```yaml
- name: Install Docker CE
  dnf:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
      - docker-buildx-plugin
      - docker-compose-plugin
    state: present
```

### What?

Installs Docker and its required packages.

### Why?

These packages are required to run Docker.

### How?

The `dnf` module downloads and installs them.

---

## 5. Create Docker Configuration

```yaml
- name: Docker daemon
  copy:
    content: |
      {
        "iptables": false
      }
    dest: /etc/docker/daemon.json
    mode: "0644"
```

### What?

Creates `/etc/docker/daemon.json`.

### Why?

Docker was failing while creating `iptables` rules on this Red Hat server.

`"iptables": false` tells Docker:

> "Don't create iptables rules automatically."

### How?

The `copy` module creates the configuration file.

---

## 6. Reset Failed Docker Service

```yaml
- name: Reset fail docker service stats
  command: systemctl reset-failed docker.service
  changed_when: false
  failed_when: false
```

### What?

Clears Docker's failed status in systemd.

### Why?

If Docker fails many times, systemd blocks it from starting again.

### How?

* `systemctl reset-failed` removes the failed status.
* `changed_when: false` → Show **OK**, not **Changed**, because no configuration was modified.
* `failed_when: false` → If there is nothing to reset, don't stop the playbook.

---

## 7. Reload Systemd

```yaml
- name: Reload systemd
  systemd:
    daemon_reload: yes
```

### What?

Reloads systemd configuration.

### Why?

Makes systemd read any updated service configuration.

### How?

Runs the equivalent of:

```bash
systemctl daemon-reload
```

---

## 8. Restart Docker

```yaml
- name: Restart the docker service
  systemd:
    name: docker
    state: restarted
    enabled: yes
```

### What?

Restarts Docker and enables it at boot.

### Why?

Docker must restart to read the new `daemon.json` file.

### How?

Stops Docker and starts it again.

---

## 9. Add User to Docker Group

```yaml
- name: Append user to the docker group
  user:
    name: "{{ ansible_user }}"
    groups: docker
    append: yes
```

### What?

Adds the user to the `docker` group.

### Why?

Allows running Docker commands without `sudo`.

### How?

The `user` module adds the user while keeping existing groups.

---

# Why only Red Hat?

### Ubuntu

```
Install Docker
      ↓
Docker creates iptables rules
      ↓
Works ✅
```

### Amazon Linux

```
Install Docker
      ↓
Docker creates iptables rules
      ↓
Works ✅
```

### Red Hat

```
Install Docker
      ↓
Docker tries to create iptables rules
      ↓
iptables error
      ↓
Docker fails
      ↓
Create daemon.json
      ↓
Restart Docker
      ↓
Works ✅
```

---

# Quick Revision

* **`args`** → Passes extra options to a module.
* **`creates`** → Run the command only if the file doesn't already exist.
* **`changed_when: false`** → Show **OK** instead of **Changed**.
* **`failed_when: false`** → Don't stop the playbook if the command returns an error.
* **`daemon.json`** → Docker configuration file.
* **`daemon_reload`** → Reloads **systemd** configuration.
* **`restart`** → Restarts Docker so it loads the new configuration.

