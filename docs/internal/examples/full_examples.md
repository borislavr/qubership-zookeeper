
# Deployment via DP Deployer Job

```yaml
global:
  name: zookeeper
  waitForPodsReady: true
  podReadinessTimeout: 300
  secrets:
    zooKeeper:
      adminUsername: "zadmin"
      adminPassword: "zadmin"
      clientUsername: "zclient"
      clientPassword: "zclient"
      additionalUsers: "user:pass"
    backupDaemon:
      username: "admin"
      password: "admin"

operator:
  affinity: {
    "podAntiAffinity": {
      "requiredDuringSchedulingIgnoredDuringExecution": [
        {
          "labelSelector": {
            "matchExpressions": [
              {
                "key": "component",
                "operator": "In",
                "values": [
                    "zookeeper-service-operator"
                ]
              }
            ]
          },
          "topologyKey": "kubernetes.io/hostname"
        }
      ]
    }
  }
  tolerations:
    - key: "key"
      operator: "Equal"
      value: "value"
      effect: "NoExecute"
      tolerationSeconds: 3600
  priorityClassName: "high-priority"

## Values for ZooKeeper deployment
zooKeeper:
  affinity: {
    "podAntiAffinity": {
      "requiredDuringSchedulingIgnoredDuringExecution": [
      {
        "labelSelector": {
          "matchExpressions": [
          {
            "key": "component",
            "operator": "In",
            "values": [
              "zookeeper"
            ]
          }
          ]
        },
        "topologyKey": "kubernetes.io/hostname"
      }
      ]
    }
  }
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoExecute"
    tolerationSeconds: 3600
  priorityClassName: "high-priority"
  disruptionBudget:
    enabled: false
  replicas: 3
  storage:
    volumes:
      - zk-pv-1
      - zk-pv-2
      - zk-pv-3
    labels:
      - key1=value1
      - key2=value2
      - key3=value3
    className:
      - standard
    nodes:
      - node-1
      - node-2
      - node-3
    size: 2Gi
  snapshotStorage:
    persistentVolumeType: predefined
    persistentVolumeName: pv-zk-snapshots
    persistentVolumeClaimName: pvc-zookeeper-snapshots
    volumeSize: 1Gi
    storageClass: standard
  heapSize: 256
  jolokiaPort: 9087
  resources:
    requests:
      cpu: 50m
      memory: 512Mi
    limits:
      cpu: 300m
      memory: 512Mi
  quorumAuthEnabled: true
  securityContext: {
    "fsGroup": 1000
  }
  environmentVariables:
    - CONF_ZOOKEEPER_propertyName=propertyValue
    - KEY=VALUE

## Values for ZooKeeper Monitoring deployment
monitoring:
  install: true
  affinity: {
    "podAffinity": {
      "preferredDuringSchedulingIgnoredDuringExecution": [
      {
        "podAffinityTerm": {
          "labelSelector": {
            "matchExpressions": [
            {
              "key": "component",
              "operator": "In",
              "values": [
                "zookeeper"
              ]
            }
            ]
          },
          "topologyKey": "kubernetes.io/hostname"
        },
        "weight": 100
      }
      ]
    }
  }
  tolerations:
  - key: "key2"
    operator: "Equal"
    value: "value2"
    effect: "NoExecute"
    tolerationSeconds: 3600
  priorityClassName: "low-priority"
  resources:
    requests:
      cpu: 25m
      memory: 128Mi
    limits:
      cpu: 200m
      memory: 256Mi
  monitoringType: prometheus
  installGrafanaDashboard: true
  zooKeeperBackupDaemonHost: zookeeper-backup-daemon
  securityContext: {
    "runAsUser": 1000
  }

## Values for ZooKeeper Backup Daemon deployment
backupDaemon:
  install: false
  affinity: {
    "podAffinity": {
      "preferredDuringSchedulingIgnoredDuringExecution": [
      {
        "podAffinityTerm": {
          "labelSelector": {
            "matchExpressions": [
            {
              "key": "component",
              "operator": "In",
              "values": [
                "zookeeper"
              ]
            }
            ]
          },
          "topologyKey": "kubernetes.io/hostname"
        },
        "weight": 100
      }
      ]
    }
  }
  tolerations:
  - key: "key3"
    operator: "Equal"
    value: "value3"
    effect: "NoExecute"
    tolerationSeconds: 3600
  priorityClassName: "low-priority"
  backupStorage:
    persistentVolumeType: standalone
    persistentVolumeName: pv-zk-snapshots
    persistentVolumeClaimName: pvc-zookeeper-snapshots
    storageClass: standard
    persistentVolumeLabel: "key=value"
    nodeName: "node-1"
    volumeSize: 1Gi
  resources:
    requests:
      cpu: 25m
      memory: 512Mi
    limits:
      cpu: 300m
      memory: 512Mi
  backupSchedule: "0 * * * *"
  evictionPolicy: "0/1d,7d/delete"
  ipv6: false
  zooKeeperHost: zookeeper
  zooKeeperPort: 2181
  securityContext: {
    "runAsUser": 1000
  }

# Values for Vault Secret Management
vaultSecretManagement:
  enabled: false
  url: http://vault-service.vault:8200
  method: kubernetes
  role: kubernetes-operator-role
  path: secret
  writePolicies: true
  passwordGenerationMechanism: operator
  refreshCredentials: false

## ZooKeeper Integration Tests parameters
integrationTests:
  install: false
  waitForResult: true
  timeout: 300
  service:
    name: zookeeper-integration-tests-runner
  tags: "zookeeper_crud"
  url: "https://kube.com:6443"
  zookeeperIsManagedByOperator: "true"
  zookeeperHost: "zookeeper"
  zookeeperPort: 2181
  pvType: "nfs"
  resources:
    requests:
      memory: 256Mi
      cpu: 200m
    limits:
      memory: 256Mi
      cpu: 400m
```

# Deployment via Groovy Deployer Job

```text
DEPLOY_W_HELM=true;
CUSTOM_RESOURCE_NAME=zookeeper;

global.name=zookeeper;
global.waitForPodsReady=true;
global.podReadinessTimeout=300;
global.secrets.zooKeeper.adminUsername=zadmin;
global.secrets.zooKeeper.adminPassword=zadmin;
global.secrets.zooKeeper.clientUsername=zclient;
global.secrets.zooKeeper.clientPassword=zclient;
global.secrets.zooKeeper.additionalUsers=user:pass;
global.secrets.backupDaemon.username=admin;
global.secrets.backupDaemon.password=admin;

operator.affinity='{"podAntiAffinity": {"requiredDuringSchedulingIgnoredDuringExecution": [{"labelSelector": {"matchExpressions": [{"key": "component", "operator": "In", "values": ["zookeeper-service-operator"]}]}, "topologyKey": "kubernetes.io/hostname"}]}}';
operator.tolerations='[{"key":"key1", "operator":"Equal", "value":"value1", "effect":"NoExecute", "tolerationSeconds":3600}]';
operator.priorityClassName=high-priority;

zooKeeper.affinity='{"podAntiAffinity": {"requiredDuringSchedulingIgnoredDuringExecution": [{"labelSelector": {"matchExpressions": [{"key": "component", "operator": "In", "values": ["zookeeper"]}]}, "topologyKey": "kubernetes.io/hostname"}]}}';
zooKeeper.tolerations='[{"key":"key1", "operator":"Equal", "value":"value1", "effect":"NoExecute", "tolerationSeconds":3600}]';
zooKeeper.priorityClassName=high-priority;
zooKeeper.disruptionBudget.enabled=false;
zooKeeper.replicas=3;
zooKeeper.storage.volumes='["zk-pv-1","zk-pv-2","zk-pv-3"]';
zooKeeper.storage.labels='["key1=value1","key2=value2","key3=value3"]';
zooKeeper.storage.className='["standard"]';
zooKeeper.storage.nodes='["node-1","node-2","node-3"]';
zooKeeper.storage.size=2Gi;
zooKeeper.snapshotStorage.persistentVolumeType=predefined;
zooKeeper.snapshotStorage.persistentVolumeName=pv-zk-snapshots;
zooKeeper.snapshotStorage.persistentVolumeClaimName=pvc-zookeeper-snapshots;
zooKeeper.snapshotStorage.volumeSize=1Gi;
zooKeeper.snapshotStorage.storageClass=standard;
zooKeeper.heapSize=256;
zooKeeper.jolokiaPort=9087;
zooKeeper.resources.requests.cpu=50m;
zooKeeper.resources.requests.memory=512Mi;
zooKeeper.resources.limits.cpu=300m;
zooKeeper.resources.limits.memory=512Mi;
zooKeeper.quorumAuthEnabled=true;
zooKeeper.securityContext='{"fsGroup": 1000}';
zooKeeper.environmentVariables='["CONF_ZOOKEEPER_propertyName=propertyValue","KEY=VALUE"]';

monitoring.install=true;
monitoring.affinity='{"podAffinity": {"preferredDuringSchedulingIgnoredDuringExecution": [{"podAffinityTerm": {"labelSelector": {"matchExpressions": [{"key": "component", "operator": "In", "values": ["zookeeper"]}]}, "topologyKey": "kubernetes.io/hostname"}, "weight": 100}]}}';
monitoring.tolerations='[{"key":"key2", "operator":"Equal", "value":"value2", "effect":"NoExecute", "tolerationSeconds":3600}]';
monitoring.priorityClassName=low-priority;
monitoring.resources.requests.cpu=25m;
monitoring.resources.requests.memory=128Mi;
monitoring.resources.limits.cpu=200m;
monitoring.resources.limits.memory=256Mi;
monitoring.monitoringType=prometheus;
monitoring.installGrafanaDashboard=true;
monitoring.zooKeeperBackupDaemonHost=zookeeper-backup-daemon;
monitoring.securityContext='{"runAsUser": 1000}';

backupDaemon.install=false;
backupDaemon.affinity='{"podAffinity": {"preferredDuringSchedulingIgnoredDuringExecution": [{"podAffinityTerm": {"labelSelector": {"matchExpressions": [{"key": "component", "operator": "In", "values": ["zookeeper"]}]}, "topologyKey": "kubernetes.io/hostname"}, "weight": 100}]}}';
backupDaemon.tolerations='[{"key":"key3", "operator":"Equal", "value":"value3", "effect":"NoExecute", "tolerationSeconds":3600}]';
backupDaemon.priorityClassName=low-priority;
backupDaemon.backupStorage.persistentVolumeType=standalone;
backupDaemon.backupStorage.persistentVolumeName=pv-zk-snapshots;
backupDaemon.backupStorage.persistentVolumeClaimName=pvc-zookeeper-snapshots;
backupDaemon.backupStorage.storageClass=standard;
backupDaemon.backupStorage.persistentVolumeLabel=key=value;
backupDaemon.backupStorage.nodeName=node-1;
backupDaemon.backupStorage.volumeSize=1Gi;
backupDaemon.resources.requests.cpu=25m;
backupDaemon.resources.requests.memory=512Mi;
backupDaemon.resources.limits.cpu=300m;
backupDaemon.resources.limits.memory=512Mi;
backupDaemon.backupSchedule=0 * * * *;
backupDaemon.evictionPolicy=0/1d,7d/delete;
backupDaemon.ipv6=false;
backupDaemon.zooKeeperHost=zookeeper;
backupDaemon.zooKeeperPort=2181;
backupDaemon.securityContext='{"runAsUser": 1000}';

vaultSecretManagement.enabled=false;
vaultSecretManagement.url=http://vault-service.vault:8200;
vaultSecretManagement.method=kubernetes;
vaultSecretManagement.role=kubernetes-operator-role;
vaultSecretManagement.path=secret;
vaultSecretManagement.writePolicies=true;
vaultSecretManagement.passwordGenerationMechanism=operator;
vaultSecretManagement.refreshCredentials=false;

integrationTests.install=true;
integrationTests.waitForResult=true;
integrationTests.timeout=300;
integrationTests.service.name=zookeeper-integration-tests-runner;
integrationTests.tags=zookeeper_crud;
integrationTests.url=https://kube.com:6443;
integrationTests.zookeeperIsManagedByOperator=true;
integrationTests.zookeeperHost=zookeeper;
integrationTests.zookeeperPort=2181;
integrationTests.pvType=nfs;
integrationTests.resources.requests.memory=256Mi;
integrationTests.resources.requests.cpu=200m;
integrationTests.resources.limits.memory=256Mi;
integrationTests.resources.limits.cpu=400m;

ESCAPE_SEQUENCE=true;
```
