# Linux VM Health Check Ansible Playbook

A comprehensive Ansible playbook for monitoring and diagnosing the health of Linux virtual machines across multiple distributions including RHEL, CentOS, Ubuntu, Debian, and SUSE.

## 🎯 Purpose

This playbook was created to provide automated health monitoring for Linux VMs in enterprise environments. It performs comprehensive system checks and generates detailed health reports that can be integrated with monitoring systems or used for compliance reporting.

## ✨ Features

### System Monitoring
- **CPU Usage Analysis** - Real-time CPU utilization with configurable thresholds
- **Memory Assessment** - RAM and swap usage monitoring with alerts
- **Disk Space Monitoring** - Multi-mount point disk usage tracking
- **Network Interface Status** - Interface state and connectivity checks
- **System Load Analysis** - Load average monitoring with trend analysis

### Service Health Checks
- **Critical Service Monitoring** - Configurable service status verification
- **Port Availability Checks** - Network service accessibility testing
- **Systemd Unit Monitoring** - Failed unit detection and reporting
- **Log Analysis** - Recent error detection in system logs

### Reporting & Alerting
- **JSON Health Reports** - Structured output for integration with monitoring tools
- **Configurable Thresholds** - Environment-specific warning and critical levels
- **Real-time Status Display** - Immediate console feedback during execution
- **Multi-environment Support** - Production, staging, and development configurations

## 🚀 Quick Start

### Prerequisites
- Ansible 8.0+ installed on the control node
- Python 3.8+ on target systems
- SSH access to target Linux VMs
- Sudo privileges on target systems

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/keepithuman/linux-vm-health-check.git
   cd linux-vm-health-check
   ```

2. **Install dependencies:**
   ```bash
   ansible-galaxy install -r requirements.yml
   pip install -r requirements.txt
   ```

3. **Configure inventory:**
   Edit `inventory.yml` to add your Linux VMs:
   ```yaml
   production:
     hosts:
       web-server-01:
         ansible_host: 10.1.1.10
         ansible_user: admin
       db-server-01:
         ansible_host: 10.1.1.11
         ansible_user: admin
   ```

4. **Run the health check:**
   ```bash
   ansible-playbook -i inventory.yml linux-health-check.yml
   ```

## 📋 Configuration

### Inventory Variables

Configure health check parameters in `inventory.yml`:

```yaml
all:
  vars:
    # Performance thresholds
    disk_threshold_warning: 80      # Disk usage warning %
    disk_threshold_critical: 90     # Disk usage critical %
    memory_threshold_warning: 80    # Memory usage warning %
    memory_threshold_critical: 90   # Memory usage critical %
    cpu_threshold_warning: 80       # CPU usage warning %
    load_threshold_warning: 2.0     # Load average warning threshold
    
    # Services to monitor
    check_services:
      - sshd
      - systemd-resolved
      - nginx
      - mysql
    
    # Network ports to verify
    check_ports:
      - 22   # SSH
      - 80   # HTTP
      - 443  # HTTPS
      - 3306 # MySQL
    
    # Behavior settings
    fail_on_critical: false  # Stop playbook on critical issues
```

### Environment-Specific Settings

Different environments can have different thresholds:

```yaml
production:
  vars:
    disk_threshold_warning: 70    # Stricter for production
    memory_threshold_warning: 75
    fail_on_critical: true        # Fail fast in production

development:
  vars:
    disk_threshold_warning: 85    # Relaxed for development
    fail_on_critical: false       # Never fail in dev
```

## 🎮 Usage Examples

### Basic Health Check
```bash
# Check all configured VMs
ansible-playbook -i inventory.yml linux-health-check.yml

# Check specific environment
ansible-playbook -i inventory.yml linux-health-check.yml --limit production

# Check specific host
ansible-playbook -i inventory.yml linux-health-check.yml --limit web-server-01
```

### Tag-Based Execution
```bash
# Only check performance metrics
ansible-playbook -i inventory.yml linux-health-check.yml --tags performance

# Only check services
ansible-playbook -i inventory.yml linux-health-check.yml --tags services

# Skip network checks
ansible-playbook -i inventory.yml linux-health-check.yml --skip-tags network
```

### Verbose Output
```bash
# Detailed execution information
ansible-playbook -i inventory.yml linux-health-check.yml -v

# Debug level output
ansible-playbook -i inventory.yml linux-health-check.yml -vvv
```

## 📊 Health Report Format

The playbook generates comprehensive JSON reports in `/tmp/health_reports/`:

```json
{
  "health_check": {
    "timestamp": "2025-06-30T21:30:00Z",
    "hostname": "web-server-01",
    "status": "WARNING",
    "summary": {
      "warnings_count": 2,
      "critical_issues_count": 0
    }
  },
  "system_info": {
    "hostname": "web-server-01",
    "distribution": "Ubuntu",
    "distribution_version": "22.04",
    "kernel": "5.15.0-72-generic",
    "uptime": "2592000"
  },
  "performance_metrics": {
    "cpu": {
      "usage_percent": 15.2,
      "load_average": {
        "1min": "0.85",
        "5min": "0.92",
        "15min": "1.15"
      }
    },
    "memory": {
      "total_mb": 8192,
      "used_percent": 72.5,
      "swap_used_percent": 5.2
    }
  },
  "issues": {
    "warnings": [
      "High disk usage on /var: 85%",
      "Port 443 is not listening"
    ],
    "critical": []
  }
}
```

## 🏷️ Available Tags

| Tag | Description |
|-----|-------------|
| `setup` | Create directories and prepare environment |
| `info` | Gather system information |
| `cpu` | CPU usage and load average checks |
| `memory` | Memory and swap usage analysis |
| `disk` | Disk space utilization checks |
| `network` | Network interface and port checks |
| `services` | Service status verification |
| `logs` | System log analysis |
| `performance` | All performance-related checks |
| `report` | Generate health reports |

## 🔧 Integration with Itential Automation Gateway (IAG)

This playbook is designed to work seamlessly with Itential Automation Gateway:

### Creating the IAG Repository
```bash
iagctl create repository linux-health-repo \
  --description "Linux VM Health Check Repository" \
  --url "https://github.com/keepithuman/linux-vm-health-check.git" \
  --reference main
```

### Creating the IAG Service
```bash
iagctl create service ansible-playbook linux-vm-health-check \
  --repository linux-health-repo \
  --playbook linux-health-check.yml \
  --inventory inventory.yml \
  --description "Comprehensive Linux VM health monitoring service" \
  --tag health-check \
  --tag monitoring \
  --tag linux
```

### Running via IAG
```bash
iagctl run service ansible-playbook linux-vm-health-check
```

## 🎛️ Customization

### Adding Custom Checks

Extend the playbook by adding custom tasks:

```yaml
- name: Check custom application status
  uri:
    url: "http://{{ ansible_default_ipv4.address }}:8080/health"
    method: GET
    timeout: 10
  register: app_health
  ignore_errors: true
  tags: ['custom']

- name: Evaluate application health
  set_fact:
    warnings: "{{ warnings + ['Custom application health check failed'] }}"
  when: app_health.status != 200
  tags: ['custom']
```

### Custom Service Lists

Override service checks per environment:

```yaml
web_servers:
  vars:
    check_services:
      - sshd
      - nginx
      - php-fpm
      - redis

database_servers:
  vars:
    check_services:
      - sshd
      - mysql
      - redis
```

## 🔍 Troubleshooting

### Common Issues

1. **Permission Denied**
   ```bash
   # Ensure sudo access
   ansible-playbook -i inventory.yml linux-health-check.yml --become --ask-become-pass
   ```

2. **Python Interpreter Not Found**
   ```yaml
   # Set in inventory.yml
   ansible_python_interpreter: /usr/bin/python3
   ```

3. **SSH Connection Issues**
   ```bash
   # Test connectivity
   ansible -i inventory.yml all -m ping
   ```

### Debug Mode

Enable detailed logging:
```bash
ansible-playbook -i inventory.yml linux-health-check.yml -vvv --diff
```

## 📝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Support

- **Issues**: [GitHub Issues](https://github.com/keepithuman/linux-vm-health-check/issues)
- **Discussions**: [GitHub Discussions](https://github.com/keepithuman/linux-vm-health-check/discussions)
- **Documentation**: This README and inline code comments

## 🎉 Acknowledgments

- Built for the Ansible community
- Designed for enterprise environments
- Optimized for Itential Automation Gateway integration
- Inspired by modern DevOps practices

---

**Made with ❤️ for reliable infrastructure monitoring**