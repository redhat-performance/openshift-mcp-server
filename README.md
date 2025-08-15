# OpenShift MCP Server

A Model Context Protocol (MCP) server for monitoring and managing OpenShift/Kubernetes clusters. This server provides real-time cluster insights through AI chatbots, enabling natural language queries about your cluster health, performance, and resource usage.

## Features

### Core Monitoring Capabilities
- **Cluster Health Monitoring**: Check overall cluster health and identify stability issues
- **Performance Metrics**: Retrieve real-time performance metrics for nodes and pods
- **Resource Issue Detection**: Detect pods and nodes with resource allocation problems
- **Pod Disruption Analysis**: Analyze pod disruptions and restart patterns
- **Node Condition Monitoring**: Check node conditions and identify problematic nodes
- **Deployment Monitoring**: Monitor deployment status and rollout health
- **System Diagnostics**: Check kubelet, CRI-O status and analyze journalctl logs

### Deployment & Configuration Management
- **Application Deployment**: Create deployments with guaranteed QoS and resource management
- **Database Services**: Deploy PostgreSQL, MySQL, MongoDB, or Redis with persistent storage
- **Auto-scaling**: Set up horizontal pod autoscalers with CPU/memory targets
- **Service Management**: Create and configure services (ClusterIP, NodePort, LoadBalancer)
- **Network Security**: Create network policies for secure pod-to-pod communication

### Performance Testing & Benchmarking
- **Cluster Performance**: Execute kube-burner density testing for application deployment performance
- **Storage Benchmarks**: Run FIO workloads for sequential/random read/write performance testing
- **Network Testing**: Test network throughput, latency, and packet loss with iperf3
- **Resource Stress Testing**: Perform CPU and memory stress testing on worker nodes
- **Database Benchmarking**: Execute database performance tests with sysbench and pgbench

### Remote Execution Capabilities for Advanced Performance Testing and Benchmarking
- **Bastion Host Support**: Execute tests on private clusters through jump hosts
- **SSH Integration**: Secure remote command execution with credential management
- **Multi-Cluster Testing**: Test across different OpenShift environments
- **Automated Benchmarking**: Schedule and run performance tests through AI chatbots
- **Remote Execution**: Execute performance benchmarks and tools on remote clusters through bastion hosts
- **Multi-Cluster Support**: Manage and test across different OpenShift environments

## Prerequisites

- Node.js 18+ 
- Access to an OpenShift/Kubernetes cluster
- Valid kubeconfig file or in-cluster configuration
- SSH access to bastion hosts (for remote deployments)

## Bastion Host Setup

For remote cluster access through bastion hosts, follow these two setup phases:

### Phase 1: Authentication Setup

For remote cluster access through bastion hosts, you can use either SSH key (recommended) or password authentication:

#### Option 1: SSH Key Authentication (Recommended)

1. **Generate SSH key pair** (if you don't have one):
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_rsa_bastion -C "mcp-server-access"
   ```

2. **Copy public key to bastion host**:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa_bastion.pub user@your-bastion-host.com
   ```

3. **Test SSH connectivity**:
   ```bash
   ssh -i ~/.ssh/id_rsa_bastion user@your-bastion-host.com
   ```

4. (optional) **Configure MCP environment variables**:
   ```bash
   export MCP_BASTION_HOST="your-bastion-host.com"
   export MCP_BASTION_USER="your-username"
   export MCP_SSH_KEY="~/.ssh/id_rsa_bastion"
   export MCP_REMOTE_PATH="/opt/cursor-kube-mcp-server"
   export MCP_REMOTE_KUBECONFIG="/path/to/remote/kubeconfig"
   export MCP_REMOTE_NODE_ENV="production"
   ```

#### Option 2: Password Authentication (Optional)

For environments where SSH keys are not feasible:

1. **Install sshpass** (required for password authentication):
   ```bash
   # Ubuntu/Debian
   sudo apt-get install sshpass
   
   # macOS
   brew install sshpass
   
   # RHEL/CentOS
   sudo yum install sshpass
   ```

2. (optional) **Configure MCP environment variables**:
   ```bash
   export MCP_BASTION_HOST="your-bastion-host.com"
   export MCP_BASTION_USER="your-username"
   export MCP_BASTION_PASSWORD="your-secure-password"
   export MCP_REMOTE_PATH="/opt/cursor-kube-mcp-server"
   export MCP_REMOTE_KUBECONFIG="/path/to/remote/kubeconfig"
   export MCP_REMOTE_NODE_ENV="production"
   ```

**Security Note**: SSH key authentication is strongly recommended over password authentication for better security. Password authentication should only be used when SSH keys are not available.

#### Enhanced Remote Launcher Features

The `simple-remote-launcher.js` provides comprehensive functionality:

- ✅ **Automatic Configuration Validation** - Validates all required environment variables
- ✅ **Pre-Flight Connectivity Testing** - Tests SSH connection before launching MCP server
- ✅ **Dual Authentication Support** - SSH key (recommended) and password authentication
- ✅ **Cross-Platform Compatibility** - Works on Windows, Linux, and macOS
- ✅ **Robust Error Handling** - Detailed error messages and troubleshooting guidance
- ✅ **Graceful Shutdown** - Proper signal handling and resource cleanup
- ✅ **SSH Keep-Alive** - Automatic connection maintenance for stability
- ✅ **Colored Output** - Clear status messages with color coding
- ✅ **Enhanced Security** - Protection against command injection attacks

### Phase 2: Remote MCP Server Setup

After configuring authentication, set up the MCP server on the bastion host:

#### Step 1: Initial Setup (Run Once)

Run the setup script to prepare the bastion host:

```bash
./scripts/setup-bastion.sh
```

You may run the script using the following options:

```bash
Usage: ./scripts/setup-bastion.sh options
    Options:
        -H host         Bastion host
        -U user         Bastion username
        -P password     Bastion password
        -p path         Path to mcp server on bastion
        -s ssh_key      SSH keyfile
        -k kubeconfig   Remote kubeconfig
        -n              Don't actually run commands
        -v              Verbose reporting
        -q              No verbose reporting (default)
```

This script will:
- ✅ Install Node.js and npm on the bastion host (if not present)
- ✅ Create the remote directory structure
- ✅ Copy project files to the bastion host
- ✅ Install all Node.js dependencies
- ✅ Verify dependency installation with import tests
- ✅ Test cluster connectivity and kubeconfig

#### Step 2: Test Connection

After successful setup, test the remote MCP server connection:

```bash
node ./scripts/simple-remote-launcher.js
```
You may run the script using the following options:

```bash
Usage: simple-remote-launcher.js [options]

Options:
  -H, --host <host>           Bastion host address
  -U, --user <user>           Bastion username  
  -P, --password <password>   Bastion password (alternative to SSH key)
  -p, --path <path>           Remote MCP server path
  -s, --ssh-key <keyfile>     SSH private key file path
  -k, --kubeconfig <config>   Remote kubeconfig file path
  -e, --env <environment>     Node environment (default: production)
  -h, --help                  Show this help message

Environment Variables (used as defaults):
  MCP_BASTION_HOST            Bastion host address
  MCP_BASTION_USER            Bastion username
  MCP_BASTION_PASSWORD        Bastion password
  MCP_REMOTE_PATH             Remote MCP server path
  MCP_SSH_KEY                 SSH private key file path
  MCP_REMOTE_KUBECONFIG       Remote kubeconfig file path
  MCP_REMOTE_NODE_ENV         Node environment

Examples:
  # Using CLI arguments only
  ./scripts/simple-remote-launcher.js -H bastion.example.com -U root -s ~/.ssh/id_rsa -k /path/to/kubeconfig -p /opt/mcp-server

  # Using environment variables (backward compatible)
  export MCP_BASTION_HOST=bastion.example.com
  ./scripts/simple-remote-launcher.js

  # Mixed: CLI args override env vars
  export MCP_BASTION_HOST=bastion.example.com
  ./scripts/simple-remote-launcher.js -U admin -k /custom/kubeconfig
```

This enhanced launcher will:
- Validate configuration and test connectivity
- Connect to the bastion host via SSH (key or password auth)
- Start the MCP server remotely with robust error handling
- Forward stdio for MCP protocol communication
- Enable AI Code Assistant integration with the remote cluster

#### Step 3: Configure AI Code Assistant

Update your AI Code Assistant MCP configuration to use the enhanced remote launcher:

```json
{
  "mcpServers": {
    "openshift-mcp-server": {
      "command": "node",
      "args": ["/path/to/openshift-mcp-server/simple-remote-launcher.js"],
      "env": {
        "MCP_BASTION_HOST": "your-bastion-host.com",
        "MCP_BASTION_USER": "your-username",
        "MCP_SSH_KEY": "/path/to/your/ssh/key",
        "MCP_REMOTE_PATH": "/opt/cursor-kube-mcp-server",
        "MCP_REMOTE_KUBECONFIG": "/path/to/remote/kubeconfig",
        "MCP_REMOTE_NODE_ENV": "production"
      },
      "cwd": "/path/to/openshift-mcp-server"
    }
  }
}
```

#### Troubleshooting Bastion Setup

**Common Issues:**

1. **SSH Connection Failures**
   - Verify SSH key permissions: `chmod 600 ~/.ssh/id_rsa_bastion`
   - Test manual SSH connection: `ssh -i ~/.ssh/id_rsa_bastion user@bastion-host`
   - Check network connectivity to bastion host

2. **Node.js Installation Issues**
   - Ensure bastion host has internet access for package downloads
   - Check if package manager (dnf/yum/apt) is available
   - Verify sudo permissions for Node.js installation

3. **Dependency Installation Failures**
   - Check npm registry accessibility from bastion host
   - Verify sufficient disk space in remote directory
   - Review npm install logs for specific errors

4. **Kubeconfig Issues**
   - Verify kubeconfig file exists at specified path
   - Test cluster connectivity from bastion host: `kubectl cluster-info`
   - Check network policies and firewall rules

**Debug Mode:**
```bash
export DEBUG=*
./scripts/setup-bastion.sh
```

**Re-run Setup:**
If you need to re-run the setup (e.g., after fixing issues):
```bash
./scripts/setup-bastion.sh  # Safe to run multiple times
```

## Installation

This MCP server can be deployed in two ways:

1. **Local Direct Access**: Run locally with direct cluster access
2. **Remote Bastion Host Access**: Run on a bastion host for private cluster access

### Local Installation

```bash
npm install
```

### Remote Installation

For remote bastion host deployment, follow the [Bastion Host Setup](#bastion-host-setup) section above.

## Configuration

The server uses the standard Kubernetes client configuration methods:

1. **KUBECONFIG environment variable**: Set the path to your kubeconfig file
   ```bash
   export KUBECONFIG=/path/to/your/kubeconfig
   ```

2. **Default kubeconfig location**: `~/.kube/config`

3. **In-cluster configuration**: When running inside a Kubernetes pod

## Usage

### Running the Server

```bash
npm start
```

Or directly:
```bash
node index.js
```

### Using with AI Code Assistant

Once the MCP server is configured in your AI Code Assistant, you can interact with your OpenShift cluster using natural language for monitoring and observability tasks:

#### Cluster Health & Monitoring
- *"Check the overall health of the OpenShift cluster"*
- *"Show me the current node conditions and identify any issues"*
- *"Are there any resource issues in the cluster right now?"*
- *"Monitor deployment status in the production namespace"*
- *"Get performance metrics for the last hour"*
- *"Analyze pod disruptions in the default namespace over the past 24 hours"*

#### System Diagnostics
- *"Check kubelet service status and recent logs"*
- *"Analyze CRI-O container runtime status and errors"*
- *"Show me journalctl logs for pod-related errors in the last 6 hours"*
- *"What system errors are occurring on the cluster?"*

#### Resource Analysis
- *"Detect pods with high CPU or memory usage"*
- *"Find nodes that are under resource pressure"*
- *"Show me pods that have restarted frequently"*
- *"What deployments are not ready or unhealthy?"*

#### Deployment & Configuration Management
- *"Create a new deployment with guaranteed QoS using 4 CPU cores in namespace test-ns"*
- *"Deploy a PostgreSQL database with persistent storage in the production namespace"*
- *"Create a service mesh configuration for microservices communication"*
- *"Set up a horizontal pod autoscaler for the web application"*
- *"Create network policies to secure pod-to-pod communication"*
- *"Set up ingress controllers and SSL termination"*
- *"Deploy a Redis cluster with high availability configuration"*

#### Performance Testing & Benchmarking
- *"Execute cluster density testing with kube-burner to measure application deployment performance"*
- *"Run storage performance benchmarks using FIO workloads"*
- *"Test network throughput between pods using iperf3"*
- *"Perform CPU and memory stress testing on worker nodes"*
- *"Execute database performance tests with sysbench or pgbench"*
- *"Run comprehensive cluster health validation before production deployment"*
- *"Benchmark container startup times and resource allocation"*
- *"Test cluster recovery scenarios and failover mechanisms"*

## 📚 Documentation

- **[User Guide](MCP-TOOLS-USER-GUIDE.md)** - Comprehensive guide with practical examples and workflows

### Available Tools

1. **check_cluster_health**
   - Check overall OpenShift cluster health
   - Parameters: `detailed` (boolean, optional)

2. **get_performance_metrics**
   - Retrieve current performance metrics
   - Parameters: `namespace` (string, optional), `timeRange` (string, default: "1h")

3. **detect_resource_issues**
   - Detect resource allocation issues
   - Parameters: `thresholds` (object with cpu, memory, restarts limits)

4. **analyze_pod_disruptions**
   - Analyze pod disruptions and restart patterns
   - Parameters: `namespace` (string, optional), `hours` (number, default: 24)

5. **check_node_conditions**
   - Check node conditions and identify issues
   - Parameters: none

6. **monitor_deployments**
   - Monitor deployment status and health
   - Parameters: `namespace` (string, optional)

7. **check_kubelet_status**
   - Check kubelet service status and recent logs for errors
   - Parameters: `hoursBack` (number, default: 24), `includeSystemErrors` (boolean, default: false)

8. **check_crio_status**
   - Check CRI-O container runtime status and recent logs
   - Parameters: `hoursBack` (number, default: 24), `includeContainerErrors` (boolean, default: true)

9. **analyze_journalctl_pod_errors**
   - Analyze journalctl logs for specific pod-related errors and system issues
   - Parameters: `pod` (string, optional), `hoursBack` (number, default: 24), `service` (string, optional), `errorTypes` (array, optional)

### Deployment Tools

10. **create_deployment**
    - Create a new deployment with specified configuration
    - Parameters: `name` (string, required), `image` (string, required), `namespace` (string, default: "default"), `replicas` (number, default: 1), `resources` (object, optional), `ports` (array, optional)

11. **deploy_database**
    - Deploy a database with persistent storage
    - Parameters: `type` (string: postgresql|mysql|mongodb|redis, required), `name` (string, required), `namespace` (string, default: "default"), `storageSize` (string, default: "10Gi"), `resources` (object, optional)

12. **create_hpa**
    - Set up a horizontal pod autoscaler for an application
    - Parameters: `targetDeployment` (string, required), `namespace` (string, default: "default"), `minReplicas` (number, default: 1), `maxReplicas` (number, default: 10), `cpuTarget` (number, default: 70), `memoryTarget` (number, default: 80)

13. **create_service**
    - Create a service to expose an application
    - Parameters: `name` (string, required), `namespace` (string, default: "default"), `selector` (object, required), `ports` (array, required), `type` (string: ClusterIP|NodePort|LoadBalancer, default: "ClusterIP")

14. **create_network_policy**
    - Create network policies to secure pod-to-pod communication
    - Parameters: `name` (string, required), `namespace` (string, default: "default"), `podSelector` (object, required), `ingress` (array, optional), `egress` (array, optional)

### Performance Testing Tools

15. **run_kube_burner**
    - Execute cluster density testing with kube-burner to measure application deployment performance
    - Parameters: `testType` (string: cluster-density-v2|node-density|pvc-density|crd-scale, default: "cluster-density-v2"), `iterations` (number, default: 5), `namespace` (string, default: "kube-burner-test"), `timeout` (string, default: "10m")

16. **run_storage_benchmark**
    - Run storage performance benchmarks using FIO workloads
    - Parameters: `testType` (string: sequential-read|sequential-write|random-read|random-write|mixed, default: "mixed"), `blockSize` (string, default: "4k"), `duration` (string, default: "60s"), `storageClass` (string, optional), `volumeSize` (string, default: "10Gi")

17. **run_network_test**
    - Test network throughput between pods using iperf3
    - Parameters: `testType` (string: throughput|latency|packet-loss, default: "throughput"), `duration` (string, default: "30s"), `parallel` (number, default: 1), `protocol` (string: tcp|udp, default: "tcp"), `bandwidth` (string, default: "1G")

18. **run_cpu_stress_test**
    - Perform CPU and memory stress testing on worker nodes
    - Parameters: `testType` (string: cpu|memory|combined, default: "combined"), `duration` (string, default: "2m"), `cpuCores` (number, default: 2), `memorySize` (string, default: "1G"), `nodeSelector` (object, optional)

19. **run_database_benchmark**
    - Execute database performance tests with sysbench or pgbench
    - Parameters: `dbType` (string: postgresql|mysql, required), `testType` (string: oltp_read_write|oltp_read_only|oltp_write_only, default: "oltp_read_write"), `threads` (number, default: 10), `duration` (string, default: "60s"), `tableSize` (number, default: 100000)

## Example Usage with MCP Client

### Monitoring Examples

```javascript
// Check cluster health
{
  "method": "tools/call",
  "params": {
    "name": "check_cluster_health",
    "arguments": {
      "detailed": true
    }
  }
}

// Get performance metrics
{
  "method": "tools/call",
  "params": {
    "name": "get_performance_metrics",
    "arguments": {
      "namespace": "production",
      "timeRange": "1h"
    }
  }
}
```

### Deployment Examples

```javascript
// Create a web application deployment
{
  "method": "tools/call",
  "params": {
    "name": "create_deployment",
    "arguments": {
      "name": "web-app",
      "image": "nginx:latest",
      "namespace": "production",
      "replicas": 3,
      "resources": {
        "cpuRequest": "200m",
        "memoryRequest": "256Mi",
        "cpuLimit": "1000m",
        "memoryLimit": "512Mi"
      },
      "ports": [{"containerPort": 80}]
    }
  }
}

// Deploy PostgreSQL database
{
  "method": "tools/call",
  "params": {
    "name": "deploy_database",
    "arguments": {
      "type": "postgresql",
      "name": "app-database",
      "namespace": "production",
      "storageSize": "20Gi"
    }
  }
}

// Create horizontal pod autoscaler
{
  "method": "tools/call",
  "params": {
    "name": "create_hpa",
    "arguments": {
      "targetDeployment": "web-app",
      "namespace": "production",
      "minReplicas": 3,
      "maxReplicas": 20,
      "cpuTarget": 70,
      "memoryTarget": 80
    }
  }
}
```

### Performance Testing Examples

```javascript
// Run storage benchmark
{
  "method": "tools/call",
  "params": {
    "name": "run_storage_benchmark",
    "arguments": {
      "testType": "mixed",
      "duration": "120s",
      "blockSize": "4k",
      "volumeSize": "10Gi"
    }
  }
}

// Test network throughput
{
  "method": "tools/call",
  "params": {
    "name": "run_network_test",
    "arguments": {
      "testType": "throughput",
      "duration": "60s",
      "protocol": "tcp",
      "parallel": 4
    }
  }
}

// CPU stress test
{
  "method": "tools/call",
  "params": {
    "name": "run_cpu_stress_test",
    "arguments": {
      "testType": "combined",
      "duration": "5m",
      "cpuCores": 4,
      "memorySize": "2G"
    }
  }
}
```
```

## Examples

### Performance Testing Integration

For detailed examples of extending the MCP server with performance testing capabilities, see:

- **[Kube-Burner Integration Example](docs/kube-burner-example.md)** - Comprehensive guide for integrating kube-burner-ocp performance testing tools

This example demonstrates how to:
- Add remote performance testing capabilities
- Execute benchmarks through AI chatbots
- Collect and analyze performance metrics
- Implement best practices for different cluster types

## Development

### Project Structure

```
openshift-mcp-server/
├── index.js                    # Main MCP server implementation
├── simple-remote-launcher.js   # Enhanced remote launcher for bastion hosts
├── package.json               # Node.js dependencies and scripts
├── package-lock.json          # Dependency lock file
├── README.md                  # This file
├── .gitignore                 # Git ignore patterns
├── backup/                    # Backup files (for reference)
│   ├── README.md              # Backup documentation
│   └── remote-mcp-wrapper.sh.backup # Original shell script (replaced)
├── scripts/                   # Utility scripts
│   └── setup-bastion.sh      # Bastion host setup script
├── docker/                   # Docker configuration
│   ├── Dockerfile            # Container image definition
│   └── docker-compose.yml    # Development setup
└── kubernetes/               # Kubernetes deployment manifests
    ├── deployment.yaml       # Server deployment
    ├── service.yaml          # Service definition
    ├── configmap.yaml        # Configuration
    └── rbac.yaml             # RBAC permissions
```

### Adding New Tools

1. Add the tool definition to the `ListToolsRequestSchema` handler
2. Add the implementation case to the `CallToolRequestSchema` handler
3. Implement the tool method in the class

### Error Handling

The server includes comprehensive error handling for:
- Kubernetes API connection issues
- Authentication/authorization problems
- Resource access failures
- Malformed requests

## Deployment

### Docker

Build and run using Docker:

```bash
docker build -f docker/Dockerfile -t openshift-mcp-server .
docker run -v ~/.kube/config:/root/.kube/config openshift-mcp-server
```

### Kubernetes

Deploy to your OpenShift/Kubernetes cluster:

```bash
kubectl apply -f kubernetes/
```

## Security Considerations

- The server requires cluster-wide read permissions
- Use appropriate RBAC configurations in production
- Secure kubeconfig files and service account tokens
- Consider network policies for pod-to-pod communication

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify kubeconfig file path and validity
   - Check service account permissions

2. **Connection Timeouts**
   - Verify cluster connectivity
   - Check firewall and network policies

3. **Missing Metrics**
   - Ensure metrics-server is deployed
   - Verify metrics API availability

### Debugging

Enable debug logging:
```bash
export DEBUG=*
node index.js
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

ISC License - see package.json for details

## Quick Reference

### Available Commands

| Command | Description | Example Usage |
|---------|-------------|---------------|
| `check_cluster_health` | Validate cluster health | `"Check cluster health"` |
| `get_performance_metrics` | Get current performance data | `"Get performance metrics for last hour"` |
| `detect_resource_issues` | Find resource problems | `"Are there any resource issues?"` |
| `analyze_pod_disruptions` | Analyze pod restart patterns | `"Show pod disruptions in namespace X"` |
| `check_node_conditions` | Check node health status | `"What are the current node conditions?"` |
| `monitor_deployments` | Monitor deployment status | `"Check deployment status in test namespace"` |

### Configuration Examples

#### AI Code Assistant MCP Configuration

**Local Direct Access:**
```json
{
  "mcpServers": {
    "openshift-mcp-server": {
      "command": "node",
      "args": ["/path/to/openshift-mcp-server/index.js"],
      "env": {
        "KUBECONFIG": "/path/to/kubeconfig"
      }
    }
  }
}
```

**Remote Bastion Host Access - SSH Key (Recommended):**
```json
{
  "mcpServers": {
    "openshift-mcp-server": {
      "command": "node",
      "args": ["/path/to/openshift-mcp-server/simple-remote-launcher.js"],
      "env": {
        "MCP_BASTION_HOST": "your-bastion-host.com",
        "MCP_BASTION_USER": "admin",
        "MCP_SSH_KEY": "/path/to/your/ssh/key",
        "MCP_REMOTE_PATH": "/opt/openshift-mcp-server",
        "MCP_REMOTE_KUBECONFIG": "/path/to/remote/kubeconfig",
        "MCP_REMOTE_NODE_ENV": "production"
      },
      "cwd": "/path/to/openshift-mcp-server"
    }
  }
}
```

**Remote Bastion Host Access - Password (Optional):**
```json
{
  "mcpServers": {
    "openshift-mcp-server": {
      "command": "node",
      "args": ["/path/to/openshift-mcp-server/simple-remote-launcher.js"],
      "env": {
        "MCP_BASTION_HOST": "your-bastion-host.com",
        "MCP_BASTION_USER": "admin",
        "MCP_BASTION_PASSWORD": "your-secure-password",
        "MCP_REMOTE_PATH": "/opt/openshift-mcp-server",
        "MCP_REMOTE_KUBECONFIG": "/path/to/remote/kubeconfig",
        "MCP_REMOTE_NODE_ENV": "production"
      },
      "cwd": "/path/to/openshift-mcp-server"
    }
  }
}
```

#### Environment Variables

**Local Direct Access:**
```bash
export KUBECONFIG=/path/to/kubeconfig
```

**Remote Bastion Host Access - SSH Key:**
```bash
export MCP_BASTION_HOST=bastion.example.com
export MCP_BASTION_USER=admin
export MCP_SSH_KEY=/path/to/your/ssh/key
export MCP_REMOTE_PATH=/opt/openshift-mcp-server
export MCP_REMOTE_KUBECONFIG=/path/to/remote/kubeconfig
export MCP_REMOTE_NODE_ENV=production
```

**Remote Bastion Host Access - Password:**
```bash
export MCP_BASTION_HOST=bastion.example.com
export MCP_BASTION_USER=admin
export MCP_BASTION_PASSWORD=your-secure-password
export MCP_REMOTE_PATH=/opt/openshift-mcp-server
export MCP_REMOTE_KUBECONFIG=/path/to/remote/kubeconfig
export MCP_REMOTE_NODE_ENV=production
```

## Support

For issues and questions:
- Check the troubleshooting section
- Review Kubernetes/OpenShift documentation
- Review [kube-burner-ocp documentation](https://kube-burner.github.io/kube-burner-ocp/)
- File an issue in the repository
