# Spring PetClinic Post-Deployment Verification Document

## Introduction
This document provides standard procedures for verifying the successful deployment of the Spring PetClinic application in a Kubernetes environment. Follow these steps in order to ensure all components are functioning correctly.

## Prerequisites
- kubectl CLI configured with cluster access
- Access to the namespace where PetClinic is deployed
- Basic understanding of Kubernetes commands

## 1. Resource Verification

### 1.1 Check All Resources
```bash
kubectl get all -n petclinic-dev
```

Expected Output:
```plaintext
NAME                                 READY   STATUS    RESTARTS   AGE
pod/mysql-db-XXXXX                   1/1     Running   0          Xh
pod/petclinic-app-XXXXX              1/1     Running   0          Xh

NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/mysql-service       ClusterIP      10.XXX.XXX.XXX   <none>          3306/TCP         Xh
service/petclinic-service   LoadBalancer   10.XXX.XXX.XXX   XX.XX.XX.XX     8081:31004/TCP   Xh

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-db        1/1     1            1           Xh
deployment.apps/petclinic-app   2/2     2            2           Xh
```

Verification Checklist:
- [ ] MySQL pod status is 'Running'
- [ ] PetClinic application pods status is 'Running'
- [ ] All services have valid IP addresses
- [ ] LoadBalancer has external IP assigned

## 2. Database Connectivity Verification

### 2.1 MySQL Pod Verification
```bash
# Check MySQL logs
kubectl logs mysql-db-XXXXX -n petclinic-dev

# Access MySQL pod
kubectl exec -it mysql-db-XXXXX -n petclinic-dev -- /bin/bash

# Test MySQL login
mysql -u root -p
```

Expected Results:
- No errors in MySQL logs
- Successful pod access
- Successful database login

### 2.2 Application-Database Connectivity
```bash
# Check environment variables
kubectl exec -it petclinic-app-XXXXX -n petclinic-dev -- printenv | grep MYSQL

# Test network connectivity
kubectl exec -it petclinic-app-XXXXX -n petclinic-dev -- nc -zv mysql-service 3306
```

Expected Results:
- Environment variables properly set
- Connection to MySQL service successful

## 3. Application Health Checks

### 3.1 Application Logs
```bash
# View application logs
kubectl logs -f petclinic-app-XXXXX -n petclinic-dev
```

Look for:
- Successful Spring Boot startup
- No error messages
- Database connection successful

### 3.2 Application Endpoint Testing
```bash
# Access pod shell
kubectl exec -it petclinic-app-XXXXX -n petclinic-dev -- /bin/sh

# Install curl
apk add --no-cache curl

# Test local endpoint
curl http://localhost:8081/
```

Expected Result:
- HTTP 200 response
- PetClinic homepage HTML content

## 4. DNS Resolution Verification

```bash
# Access pod
kubectl exec -it petclinic-app-XXXXX -n petclinic-dev -- /bin/sh

# Test DNS resolution
nslookup mysql-service.petclinic-dev.svc.cluster.local

# Check DNS configuration
cat /etc/resolv.conf
```

Expected Results:
- Successful DNS resolution
- Proper DNS configuration present

## 5. Troubleshooting Guide

### 5.1 Common Issues and Solutions

#### Pod Issues
```bash
# Get detailed pod information
kubectl describe pod <pod-name> -n petclinic-dev

# Check pod logs
kubectl logs <pod-name> -n petclinic-dev
```

#### Database Connection Issues
1. Verify service name matches configuration
2. Check database credentials
3. Verify network connectivity
4. Review database logs

#### Application Access Issues
1. Verify LoadBalancer external IP
2. Check service port mappings
3. Verify security group rules
4. Test internal cluster DNS

### 5.2 Quick Recovery Steps

1. Pod Recovery
```bash
kubectl rollout restart deployment <deployment-name> -n petclinic-dev
```

2. Service Recovery
```bash
kubectl delete service <service-name> -n petclinic-dev
kubectl apply -f service.yaml
```

## 6. Monitoring Recommendations

### 6.1 Key Metrics to Monitor
1. Pod Status
2. Database Connections
3. Application Response Time
4. Resource Usage (CPU/Memory)
5. Request Rate
6. Error Rate

### 6.2 Recommended Alerts
1. Pod Restarts > 3 in 15 minutes
2. Database Connection Failures
3. High CPU/Memory Usage (>80%)
4. Response Time > 2s
5. Error Rate > 1%

## 7. Verification Checklist Summary

- [ ] All pods running
- [ ] Services properly configured
- [ ] Database connection verified
- [ ] Application endpoints accessible
- [ ] DNS resolution working
- [ ] Logs show no errors
- [ ] Monitoring setup complete

## Important Notes

1. Replace 'XXXXX' with actual pod IDs from your deployment
2. Adjust namespace if not using 'petclinic-dev'
3. Keep LoadBalancer URL for application access
4. Document any custom configurations made
5. Update this guide as needed for environment-specific requirements

## Support

For additional support or questions:
1. Check application and database logs first
2. Review Kubernetes events
3. Consult team's internal documentation
4. Contact platform support team if needed

---
Document Version: 1.0
Last Updated: January 2024
