# Ansible Role: DragonflyDB

Installs and configures [DragonflyDB](https://www.dragonflydb.io/) from binary with systemd on Linux systems.

DragonflyDB is a modern in-memory datastore, fully compatible with Redis and Memcached APIs, designed for high performance and efficiency.

## Requirements

- Linux-based operating system
- Linux Kernel 4.19 or higher
- Minimum 4GB RAM
- Minimum 1 CPU core
- systemd
- Ansible 2.12+

### Supported Architectures

- x86_64 / amd64
- aarch64 / arm64

## Role Variables

### Installation Settings

| Variable                    | Default                      | Description                                          |
|-----------------------------|------------------------------|------------------------------------------------------|
| `dragonfly_version`         | `"latest"`                   | Version to install (e.g., `"v1.15.1"` or `"latest"`) |
| `dragonfly_install_dir`     | `"/opt/dragonfly"`           | Installation directory                               |
| `dragonfly_bin_path`        | `"/usr/local/bin/dragonfly"` | Binary path                                          |
| `dragonfly_data_dir`        | `"/var/lib/dragonfly"`       | Data directory                                       |
| `dragonfly_config_dir`      | `"/etc/dragonfly"`           | Configuration directory                              |
| `dragonfly_log_dir`         | `"/var/log/dragonfly"`       | Log directory                                        |
| `dragonfly_force_reinstall` | `false`                      | Force reinstall even if binary exists                |

### User/Group Settings

| Variable          | Default       | Description                  |
|-------------------|---------------|------------------------------|
| `dragonfly_user`  | `"dragonfly"` | System user to run dragonfly |
| `dragonfly_group` | `"dragonfly"` | System group                 |

### Network Settings

| Variable         | Default       | Description  |
|------------------|---------------|--------------|
| `dragonfly_bind` | `"127.0.0.1"` | Bind address |
| `dragonfly_port` | `6379`        | Listen port  |

### Memcached API Settings

| Variable                   | Default | Description                                        |
|----------------------------|---------|----------------------------------------------------|
| `dragonfly_memcached_port` | `""`    | Memcached API port (e.g., `11211`), empty=disabled |

### Memory Settings

| Variable              | Default | Description                     |
|-----------------------|---------|---------------------------------|
| `dragonfly_maxmemory` | `""`    | Maximum memory (e.g., `"12gb"`) |

### Authentication

| Variable                | Default | Description                 |
|-------------------------|---------|-----------------------------|
| `dragonfly_requirepass` | `""`    | Password for authentication |

### Database Settings

| Variable               | Default | Description                         |
|------------------------|---------|-------------------------------------|
| `dragonfly_cache_mode` | `false` | Enable cache mode (allows eviction) |
| `dragonfly_dbnum`      | `16`    | Number of databases                 |

### Persistence Settings

| Variable                  | Default  | Description                     |
|---------------------------|----------|---------------------------------|
| `dragonfly_dbfilename`    | `"dump"` | Persistence file name           |
| `dragonfly_snapshot_cron` | `""`     | Snapshot schedule (cron format) |

### Logging Settings

| Variable                | Default  | Description   |
|-------------------------|----------|---------------|
| `dragonfly_log_level`   | `"info"` | Log level     |
| `dragonfly_logtostderr` | `false`  | Log to stderr |

### Logrotate Settings

| Variable                            | Default   | Description                               |
|-------------------------------------|-----------|-------------------------------------------|
| `dragonfly_logrotate_enabled`       | `true`    | Enable logrotate configuration            |
| `dragonfly_logrotate_frequency`     | `"daily"` | Rotation frequency (daily/weekly/monthly) |
| `dragonfly_logrotate_rotate`        | `7`       | Number of rotated files to keep           |
| `dragonfly_logrotate_maxsize`       | `"100M"`  | Rotate when log exceeds this size         |
| `dragonfly_logrotate_compress`      | `true`    | Compress rotated logs                     |
| `dragonfly_logrotate_delaycompress` | `true`    | Delay compression by one rotation         |
| `dragonfly_logrotate_missingok`     | `true`    | Don't error if log file is missing        |
| `dragonfly_logrotate_notifempty`    | `true`    | Don't rotate empty logs                   |
| `dragonfly_logrotate_create_mode`   | `"0640"`  | Permissions for new log files             |
| `dragonfly_logrotate_postrotate`    | `true`    | Signal Dragonfly to reopen log files      |

### Service Settings

| Variable                         | Default        | Description             |
|----------------------------------|----------------|-------------------------|
| `dragonfly_service_enabled`      | `true`         | Enable service on boot  |
| `dragonfly_service_state`        | `"started"`    | Service state           |
| `dragonfly_service_restart`      | `"on-failure"` | Restart policy          |
| `dragonfly_service_restart_sec`  | `5`            | Restart delay (seconds) |
| `dragonfly_service_limit_nofile` | `65535`        | File descriptor limit   |

### Extra Flags

| Variable                | Default | Description                   |
|-------------------------|---------|-------------------------------|
| `dragonfly_extra_flags` | `[]`    | Additional command-line flags |

## Dependencies

None.

## Example Playbook

### Basic Installation

```yaml
- hosts: cache_servers
  roles:
    - role: oxess.dragonfly
```

### Production Configuration

```yaml
- hosts: cache_servers
  roles:
    - role: oxess.dragonfly
      vars:
        dragonfly_version: "v1.15.1"
        dragonfly_bind: "0.0.0.0"
        dragonfly_port: 6379
        dragonfly_maxmemory: "8gb"
        dragonfly_requirepass: "your_secure_password"
        dragonfly_cache_mode: true
        dragonfly_snapshot_cron: "*/30 * * * *"
        dragonfly_extra_flags:
          - "--keys_output_limit=10000"
          - "--hz=100"
```

### Cache-Only Mode (No Persistence)

```yaml
- hosts: cache_servers
  roles:
    - role: oxess.dragonfly
      vars:
        dragonfly_cache_mode: true
        dragonfly_maxmemory: "4gb"
        dragonfly_dbfilename: ""
```

### High-Availability Setup with Authentication

```yaml
- hosts: cache_servers
  roles:
    - role: oxess.dragonfly
      vars:
        dragonfly_bind: "{{ ansible_default_ipv4.address }}"
        dragonfly_requirepass: "{{ vault_dragonfly_password }}"
        dragonfly_maxmemory: "16gb"
        dragonfly_service_limit_nofile: 100000
        dragonfly_snapshot_cron: "*/15 * * * *"
```

### Redis + Memcached Dual API Mode

```yaml
- hosts: cache_servers
  roles:
    - role: oxess.dragonfly
      vars:
        dragonfly_port: 6379
        dragonfly_memcached_port: 11211
        dragonfly_maxmemory: "8gb"
```

## Service Management

After installation, manage the service using systemd:

```bash
# Check status
sudo systemctl status dragonfly

# Start/stop/restart
sudo systemctl start dragonfly
sudo systemctl stop dragonfly
sudo systemctl restart dragonfly

# View logs
sudo journalctl -u dragonfly -f
```

## Connecting to DragonflyDB

DragonflyDB is fully compatible with Redis clients:

```bash
# Using redis-cli
redis-cli -h 127.0.0.1 -p 6379

# With authentication
redis-cli -h 127.0.0.1 -p 6379 -a your_password
```

## Security Considerations

- By default, DragonflyDB binds to `127.0.0.1` only
- Set `dragonfly_requirepass` for authentication in production
- The systemd service includes security hardening options
- Consider firewall rules when binding to public interfaces

## License

MIT

## Author 

This Ansible Role was created by [Mikołaj Jeziorny](https://mikolaj-jeziorny.pl/) in 2025.
Contact with me or open an issue if you have any questions or suggestions.
