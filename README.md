# AWS efs-ustils pre-built packages

This repo contains pre-built RPM and DEB packages for both ARM and x86 architectures.

First download proper version that fits your Linux distribution and CPU architecture.

```shell
PKG=$(if command -v rpm >/dev/null 2>&1; then echo rpm; elif command -v dpkg >/dev/null 2>&1; then echo deb; fi)
ARCH=$(arch=$(uname -m); case "$arch" in x86_64|i386|i686) echo x86_64 ;; arm*|aarch64) echo arm64 ;; esac)
curl -fsL "https://github.com/paul-key/aws-efs-utils/releases/latest/download/aws-efs-utils-${ARCH}.${PKG}"
```

## DEB-based Distributions

### Debian/Ubuntu

```bash
sudo apt-get -y install aws-efs-utils-${ARCH}.${PKG}
```

## RPM-based Distributions

### RHEL/CentOS/Amazon Linux/Fedora

```bash
sudo yum -y install aws-efs-utils-${ARCH}.${PKG}
```

### OpenSUSE/SLES

```bash
sudo zypper --no-gpg-checks install -y aws-efs-utils-${ARCH}.${PKG}
```


## Ansible task

To install by running ansible task:

```yaml
    - name: Install EFS utils
      block:
        - name: Get architecture
          set_fact:
            is_arm: "{{ ansible_architecture is regex('arm.*|aarch64', ignorecase=True) }}"
            is_x86: "{{ ansible_architecture is regex('x86_64|i386|i686', ignorecase=True) }}"
            pkg: "{{ (ansible_os_family == 'Debian') | ternary('deb', 'rpm') }}"
        - name: Download EFS package
          get_url:
            url: "https://github.com/paul-key/aws-efs-utils/releases/latest/download/aws-efs-utils-{% if is_arm %}arm64{% elif is_x86 %}amd64{% endif %}.{{ pkg }}"
            dest: /tmp/aws-efs-utils.{{ pkg }}
          register: efs_package
        - name: Install EFS
          command:
            argv:
              - "{{ ansible_pkg_mgr }}"
              - -y
              - install
              - "{{ efs_package.dest }}"
```