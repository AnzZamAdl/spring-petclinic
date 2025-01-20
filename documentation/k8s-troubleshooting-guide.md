# Kubernetes Deployment Troubleshooting Guide

## 1. Database Connection Issues (CrashLoopBackOff)

**Symptoms:**
```bash
pod/petclinic-app-8c6f8f875-xjn7c   0/1     CrashLoopBackOff   5 (32s ago)   7m29s
Error: Communications link failure
```

**Troubleshooting Steps:**
1. Check MySQL service and pod status:
```bash
kubectl get pods -l app=mysql -n <namespace>
kubectl get svc mysql-service -n <namespace>
```

2. Verify database connectivity:
```bash
# Test network connectivity
kubectl exec -it <pod-name> -n <namespace> -- nc -zv mysql-service 3306

# Check MySQL directly
kubectl exec -it <mysql-pod> -n <namespace> -- mysql -u root -p
```

3. Verify environment variables:
```bash
# Check configured environment variables
kubectl exec -it <pod-name> -n <namespace> -- env | grep MYSQL

# Verify ConfigMap
kubectl get configmap app-config -n <namespace> -o yaml

# Check Secrets
kubectl get secret db-secrets -n <namespace> -o yaml
```

4. Decode base64 secrets:
```bash
echo "<base64-string>" | base64 -d
```

**Common Solutions:**
- Update database credentials in secrets
- Verify MySQL service name and port in connection string
- Check MySQL user permissions
- Ensure database exists and is initialized

## 2. EBS Volume and Pod Scheduling Issues

**Symptoms:**
```bash
pod/mysql-db-5b68fff86f-95nz7   0/1     Pending   0          10m
persistentvolumeclaim "mysql-pvc" not bound
```

**Troubleshooting Steps:**
1. Check PV/PVC status:
```bash
kubectl get pv,pvc -n <namespace>
kubectl describe pv mysql-pv
kubectl describe pvc mysql-pvc -n <namespace>
```

2. Verify EBS volume and node AZ alignment:
```bash
# Get node AZ information
aws ec2 describe-instances \
    --query "Reservations[*].Instances[*].[InstanceId,Placement.AvailabilityZone]" \
    --filters "Name=private-ip-address,Values=<node-ip>"

# Check EBS volume AZ
aws ec2 describe-volumes --volume-ids <volume-id>
```

3. Verify PV configuration:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

**Common Solutions:**
- Create EBS volume in same AZ as target node
- Add nodeSelector to MySQL deployment:
```yaml
nodeSelector:
  kubernetes.io/hostname: <node-name>
```
- Verify storage class matches between PV and PVC

## 3. ConfigMap and Secret Issues

**Symptoms:**
```bash
CreateContainerConfigError: secret "db-secrets" not found
```

**Troubleshooting Steps:**
1. Verify resources exist:
```bash
kubectl get configmap,secret -n <namespace>
```

2. Check resource contents:
```bash
# ConfigMap verification
kubectl describe configmap app-config -n <namespace>

# Secret verification
kubectl get secret db-secrets -n <namespace> -o yaml
```

3. Validate deployment configuration:
```bash
kubectl get deployment <deployment-name> -n <namespace> -o yaml
```

**Common Solutions:**
- Create missing ConfigMap/Secret resources
- Ensure names match in deployment spec
- Fix base64 encoding for secrets
- Check namespace matches

## 4. Container Image Issues

**Symptoms:**
```bash
ImagePullBackOff: failed to pull image
```

**Troubleshooting Steps:**
1. Check pod events:
```bash
kubectl describe pod <pod-name> -n <namespace>
```

2. Verify ECR authentication:
```bash
# Configure authentication
aws ecr get-login-password --region <region> | \
    docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com

# Check image exists
aws ecr describe-images \
    --repository-name <repo-name> \
    --image-ids imageTag=<tag>
```

3. Check node IAM role:
```bash
# Get node IAM role
aws eks describe-nodegroup --cluster-name <cluster> --nodegroup-name <nodegroup>
```

**Common Solutions:**
- Update image tag in deployment
- Add ECR permissions to node IAM role
- Verify image exists in ECR repository

## 5. Network and DNS Issues

**Symptoms:**
- Services unreachable
- DNS resolution failures

**Troubleshooting Steps:**
1. Test DNS resolution:
```bash
# From a test pod
kubectl run dns-test --rm -i --tty --image=busybox -- nslookup mysql-service

# From application pod
kubectl exec -it <pod-name> -n <namespace> -- nslookup mysql-service
```

2. Check service configuration:
```bash
kubectl get svc -n <namespace>
kubectl describe svc mysql-service -n <namespace>
kubectl get endpoints mysql-service -n <namespace>
```

3. Verify network policies:
```bash
kubectl get networkpolicies -n <namespace>
```

**Common Solutions:**
- Check CoreDNS pods are running
- Verify service selectors match pod labels
- Update security groups for pod communication
- Check network policy rules

## Best Practices

### 1. Deployment Configuration
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-db
spec:
  template:
    spec:
      containers:
      - name: mysql
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 30
        readinessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 10
```

### 2. Resource Management
```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1000m"
    memory: "1Gi"
```

### 3. Logging Best Practices:
- Set log levels appropriately
- Implement log rotation
- Use structured logging
- Collect application and system logs

### 4. Volume Management:
- Regular volume backups
- Monitor volume usage
- Use dynamic provisioning when possible
- Implement proper cleanup procedures

## Quick Reference Commands

### Pod Debugging:
```bash
# Get pod details
kubectl get pods -n <namespace> -o wide

# Get pod logs
kubectl logs -f <pod-name> -n <namespace>

# Execute commands in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
```

### Configuration Debugging:
```bash
# Get all resources in namespace
kubectl get all -n <namespace>

# Describe specific resource
kubectl describe <resource-type> <resource-name> -n <namespace>

# Get resource YAML
kubectl get <resource-type> <resource-name> -n <namespace> -o yaml
```

### Common Status Codes:
- `Pending`: Pod scheduling or volume attachment pending
- `ContainerCreating`: Container being created
- `Running`: Pod running normally
- `CrashLoopBackOff`: Container repeatedly crashing
- `Error`: General error state
- `Completed`: Pod completed its execution
- `ImagePullBackOff`: Unable to pull container image
- `CreateContainerConfigError`: Configuration error

### Important Log Locations:
- Application Logs: `kubectl logs`
- System Logs: `/var/log/syslog`
- Container Runtime: `/var/log/containers/`
- Kubelet Logs: `journalctl -u kubelet`

---
## General Debugging Commands

### Pod Debugging
```bash
# Get pod details
kubectl get pods -n <namespace> -o wide

# Get pod logs
kubectl logs <pod-name> -n <namespace>

# Execute command in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# Get pod events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Node Debugging
```bash
# Check node status
kubectl get nodes -o wide

# Debug node
kubectl debug node/<node-name> -it --image=ubuntu

# Cordon/Uncordon node
kubectl cordon <node-name>
kubectl uncordon <node-name>
```

### Network Debugging
```bash
# Test service DNS
kubectl run -it --rm --restart=Never busybox --image=busybox -- nslookup mysql-service

# Test connectivity
kubectl exec -it <pod-name> -n <namespace> -- nc -zv <service-name> <port>

# Check service endpoints
kubectl get endpoints <service-name> -n <namespace>
```

## Best Practices

1. **Resource Management:**
   - Set appropriate resource requests and limits
   - Monitor node capacity
   - Use horizontal pod autoscaling

2. **Storage:**
   - Ensure EBS volumes are in correct AZ
   - Use storage classes for dynamic provisioning
   - Implement proper backup strategies

3. **Security:**
   - Use secrets for sensitive data
   - Implement network policies
   - Regular security group audits

4. **Monitoring:**
   - Deploy metrics-server
   - Set up logging aggregation
   - Configure alerts for critical issues

5. **Deployment Strategy:**
   - Use rolling updates
   - Implement readiness/liveness probes
   - Document deployment procedures

## Quick References

### Common Status Codes:
- `Running`: Pod is running normally
- `Pending`: Pod awaiting scheduling or volume attachment
- `CrashLoopBackOff`: Container repeatedly crashing
- `ImagePullBackOff`: Unable to pull container image
- `CreateContainerConfigError`: Issues with pod configuration

### Important Paths:
- Kubelet config: `/etc/kubernetes/kubelet/kubelet-config.json`
- PKI certificates: `/etc/kubernetes/pki/`
- Container runtime: `/var/lib/containerd/`
