# ansible-role-ghidra

An Ansible Role that installs and configures [Ghidra](https://github.com/NationalSecurityAgency/ghidra) on Debian and Windows systems.

## Requirements

### Debian Systems

None.

### Windows Systems

- **Ansible Collections**: `chocolatey.chocolatey`, `ansible.windows`, and `community.windows` (installed via `requirements.yml`)
- **Chocolatey**: Must be installed on the target Windows host prior to running this role

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
ghidra_version: "12.0.4"
ghidra_release_date: "20260303"
ghidra_checksum: "sha256:c3b458661d69e26e203d739c0c82d143cc8a4a29d9e571f099c2cf4bda62a120"
ghidra_install_dir: "/opt"
ghidra_windows_install_dir: "C:\\Tools"
ghidra_install_java: true
ghidra_state: present
ghidra_create_symlink: true
ghidra_create_desktop_file: true
ghidra_java_home_override: ""
ghidra_max_memory: ""
ghidra_extensions: []
```

### Install variables

- `ghidra_version`: Version of Ghidra to install.
- `ghidra_release_date`: Release date string used in the archive filename (`YYYYMMDD` format). Must match the chosen version.
- `ghidra_checksum`: SHA-256 checksum of the Ghidra archive for integrity verification.
- `ghidra_install_dir`: Parent directory for the Ghidra installation on Linux (default: `/opt`).
- `ghidra_windows_install_dir`: Parent directory for the Ghidra installation on Windows (default: `C:\Tools`).
- `ghidra_install_java`: Install JDK 21 as a dependency (default: `true`). Set to `false` if Java is managed separately.
- `ghidra_state`: Set to `present` to run install and configure steps (default: `present`). Any other value skips all tasks.
- `ghidra_create_symlink`: Create a stable `/opt/ghidra` symlink pointing to the versioned directory on Linux (default: `true`).
- `ghidra_create_desktop_file`: Create an XDG `.desktop` entry at `/usr/local/share/applications/ghidra.desktop` on Linux (default: `true`).

### Configure variables

- `ghidra_java_home_override`: Pin a specific JDK path via `JAVA_HOME_OVERRIDE` in `support/launch.properties`. Leave empty for auto-detection (default: `""`).
- `ghidra_max_memory`: Set the JVM max heap size (e.g., `"4G"`) via `MAXMEM` in `support/launch.properties`. Leave empty for Ghidra's default (default: `""`).
- `ghidra_extensions`: List of Ghidra extensions to install into `Extensions/Ghidra/` (default: `[]`). Each entry supports two forms:

```yaml
ghidra_extensions:
  # URL-based: downloaded and extracted automatically
  - name: "GhidraBSim"
    url: "https://example.com/GhidraBSim.zip"
    checksum: "sha256:abc123..." # optional

  # Local path: extracted from a file already on the control node
  - name: "MyLocalExt"
    src: "/path/to/MyLocalExt.zip"
```

`name` must match the directory created inside `Extensions/Ghidra/` after extraction (used for idempotency on Linux).

## Dependencies

None.

## Installation Paths

| Platform | Versioned Directory                 | Stable Reference             | System-wide Launcher      |
| -------- | ----------------------------------- | ---------------------------- | ------------------------- |
| Debian   | `/opt/ghidra_<version>_PUBLIC/`     | `/opt/ghidra` (symlink)      | `/usr/local/bin/ghidra`   |
| Windows  | `C:\Tools\ghidra_<version>_PUBLIC\` | `GHIDRA_INSTALL_DIR` env var | `ghidraRun.bat` in `PATH` |

## Example Playbook

### Install on Debian (defaults)

```yaml
- hosts: debian_hosts
  roles:
    - role: s3mme.ghidra
```

### Install on Windows (defaults)

```yaml
- hosts: windows_hosts
  roles:
    - role: s3mme.ghidra
```

### Custom JVM configuration

```yaml
- hosts: all
  roles:
    - role: s3mme.ghidra
      vars:
        ghidra_max_memory: "4G"
        ghidra_java_home_override: "/usr/lib/jvm/java-21-openjdk-amd64"
        ghidra_create_desktop_file: false
```

### Install extensions

```yaml
- hosts: all
  roles:
    - role: s3mme.ghidra
      vars:
        ghidra_extensions:
          - name: "GhidraBSim"
            url: "https://example.com/GhidraBSim.zip"
            checksum: "sha256:abc123..."
          - name: "MyLocalExt"
            src: "/path/to/MyLocalExt.zip"
```

### Upgrade to a new version

Update these three variables together: the `release_date` and `checksum` must match the new release:

```yaml
- hosts: all
  roles:
    - role: s3mme.ghidra
      vars:
        ghidra_version: "12.1.0"
        ghidra_release_date: "20261001"
        ghidra_checksum: "sha256:<new-checksum>"
```

## License

MIT
