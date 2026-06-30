# Docker Installation on Amazon Linux using Ansible

## 1. Install Docker

```yaml
- name: Install docker in amazon linux
  dnf:
    name: docker
    state: present
  notify:
    - Enable the docker service
    - Start the docker service
```

### What?

Installs the Docker package.

### Why?

Amazon Linux provides Docker in its default repository, so no extra repository needs to be added.

### How?

The `dnf` module downloads and installs Docker. After installation, it **notifies the handlers** to enable and start the Docker service.

---

## 2. Handlers

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

Handlers run **only when notified**. If Docker is already installed and nothing changes, the handlers are skipped, making the playbook idempotent and avoiding unnecessary service restarts.

---

## 3. Add User to Docker Group

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

Allows the user to run Docker commands without using `sudo`.

### How?

The `user` module adds the user to the `docker` group while preserving existing group memberships.

---

# Why is Amazon Linux simpler?

Amazon Linux includes Docker in its default repositories, so only these steps are needed:

1. Install Docker.
2. Enable the Docker service.
3. Start the Docker service.
4. Add the user to the `docker` group.

Unlike Red Hat, no additional Docker repository or `daemon.json` configuration is required.

