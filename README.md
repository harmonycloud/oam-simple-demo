# oam-simple-demo
This is a simple demo for oam in HarmonyCloud

```yaml
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl get trait 
NAME                 AGE
better-auto-scaler   160m
host-policy          160m
log-pilot            111m
manual-scaler        160m
nginx-ingress        160m
resources-policy     160m
schedule-policy      160m
volume-mounter       160m
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl -n oam-oamdemo get comp,appconfig
NAME                                          AGE
componentschematic.core.oam.dev/demo-mysql    33m
componentschematic.core.oam.dev/demo-redis    33m
componentschematic.core.oam.dev/demo-tomcat   33m

NAME                                         AGE
applicationconfiguration.core.oam.dev/demo   33m
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl -n oam-oamdemo get comp -oyaml
apiVersion: v1
items:
- apiVersion: core.oam.dev/v1alpha1
  kind: ComponentSchematic
  metadata:
    annotations:
      version: "18"
    creationTimestamp: "2020-05-06T12:21:21Z"
    generation: 1
    name: demo-mysql
    namespace: oam-oamdemo
    resourceVersion: "188951"
    selfLink: /apis/core.oam.dev/v1alpha1/namespaces/oam-oamdemo/componentschematics/demo-mysql
    uid: 187010de-8f94-11ea-8257-00163e146fef
  spec:
    containers:
    - args: []
      command: []
      env:
      - fromParam: password
        name: MYSQL_ROOT_PASSWORD
      image: onlineshop/mysql:8.0.19
      imagePullPolicy: IfNotPresent
      name: mysql
      ports:
      - containerPort: 3306
        name: tcp-3306
        protocol: TCP
      resources:
        cpu:
          required: 500m
        memory:
          required: 512Mi
        volumes:
        - accessMode: RWX
          disk:
            ephemeral: true
            required: 10Gi
          mountPath: /var/lib/mysql
          name: mysql-data
    parameters:
    - default: "123456"
      description: password for mysql root
      name: password
      required: false
      type: string
    workloadType: core.oam.dev/v1alpha1.Server
- apiVersion: core.oam.dev/v1alpha1
  kind: ComponentSchematic
  metadata:
    annotations:
      version: "18"
    creationTimestamp: "2020-05-06T12:21:21Z"
    generation: 1
    name: demo-redis
    namespace: oam-oamdemo
    resourceVersion: "188952"
    selfLink: /apis/core.oam.dev/v1alpha1/namespaces/oam-oamdemo/componentschematics/demo-redis
    uid: 187d939e-8f94-11ea-8257-00163e146fef
  spec:
    containers:
    - args: []
      command: []
      image: onlineshop/redis:3.2-alpine
      imagePullPolicy: IfNotPresent
      name: redis
      ports:
      - containerPort: 6379
        name: tcp-6379
        protocol: TCP
      resources:
        cpu:
          required: 500m
        memory:
          required: 512Mi
    workloadType: core.oam.dev/v1alpha1.Server
- apiVersion: core.oam.dev/v1alpha1
  kind: ComponentSchematic
  metadata:
    annotations:
      version: "18"
    creationTimestamp: "2020-05-06T12:21:21Z"
    generation: 1
    name: demo-tomcat
    namespace: oam-oamdemo
    resourceVersion: "188950"
    selfLink: /apis/core.oam.dev/v1alpha1/namespaces/oam-oamdemo/componentschematics/demo-tomcat
    uid: 186be010-8f94-11ea-8257-00163e146fef
  spec:
    containers:
    - args: []
      command: []
      image: onlineshop/tomcat:v8.0
      imagePullPolicy: IfNotPresent
      name: tomcat
      ports:
      - containerPort: 8080
        name: tcp-8080
        protocol: TCP
      resources:
        cpu:
          required: 500m
        memory:
          required: 512Mi
    workloadType: core.oam.dev/v1alpha1.Server
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl -n oam-oamdemo get appconfig demo -oyaml
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  creationTimestamp: "2020-05-06T12:21:22Z"
  generation: 1
  name: demo
  namespace: oam-oamdemo
  resourceVersion: "189215"
  selfLink: /apis/core.oam.dev/v1alpha1/namespaces/oam-oamdemo/applicationconfigurations/demo
  uid: 18c2c59e-8f94-11ea-8257-00163e146fef
spec:
  components:
  - componentName: demo-tomcat
    instanceName: demo-tomcat
    parameterValues: []
    traits:
    - name: better-auto-scaler
      properties:
        cpu-up: 50
        maximum: 3
        memory-up: 50
        minimum: 2
    - name: log-pilot
      properties:
        container: tomcat
        name: logindex
        path: /usr/local/tomcat/logs
        pilotLogPrefix: aliyun
        tags: owner:appconfig/demo
    - name: nginx-ingress
      properties:
        hostname: tomcat.example
        path: /
        servicePort: 8080
  - componentName: demo-mysql
    instanceName: demo-mysql
    parameterValues:
    - name: password
      value: passwordtest
    traits:
    - name: manual-scaler
      properties:
        replicaCount: 1
    - name: resources-policy
      properties:
        container: mysql
        limits:
          cpu: 1000m
          memory: 1024Mi
    - name: volume-mounter
      properties:
        storageClass: default
        volumeName: mysql-data
  - componentName: demo-redis
    instanceName: demo-redis
    parameterValues: []
    traits:
    - name: host-policy
      properties:
        hostIpc: "false"
        hostNetwork: "true"
        hostPid: "false"
    - name: manual-scaler
      properties:
        replicaCount: 1
    - name: schedule-policy
      properties:
        nodeAffinity:
          type: preferred
        podAffinity:
          selector:
            app: tomcat
          type: preferred
        podAntiAffinity:
          type: preferred
status:
  modules:
  - groupVersion: core.oam.dev/v1alpha1
    kind: Server
    name: demo-tomcat
    status: Healthy
  - groupVersion: core.oam.dev/v1alpha1
    kind: Server
    name: demo-mysql
    status: Healthy
  - groupVersion: core.oam.dev/v1alpha1
    kind: Server
    name: demo-redis
    status: Healthy
  phase: Synced
  resources:
  - apiVersion: apps/v1
    component: demo-tomcat
    kind: Deployment
    name: demo-tomcat
    role: workload
    status: 'Ready: 2/2, Up-to-date: 2, Available: 2.'
  - apiVersion: v1
    component: demo-tomcat
    kind: Service
    name: demo-tomcat
    role: workload
    status: 'Type: ClusterIP, Cluster-IP: 1.254.119.183, Port(s): 8080/TCP.'
  - apiVersion: extensions/v1beta1
    component: demo-tomcat
    kind: Ingress
    name: demo-tomcat
    role: trait
    status: 'Hosts: tomcat.example '
  - apiVersion: harmonycloud.cn/v1beta1
    component: demo-tomcat
    kind: HorizontalPodAutoscaler
    name: demo-tomcat
    role: trait
    status: 'CurrentReplicas: 2, DesiredReplicas: 2.'
  - apiVersion: v1
    component: demo-mysql
    kind: PersistentVolumeClaim
    name: mysql-data
    role: trait
    status: 'Volume: pvc-18e5f923-8f94-11ea-8257-00163e146fef, Capacity: 10Gi, AccessModes:
      [ReadWriteOnce], StorageClass: default, Status: Bound.'
  - apiVersion: v1
    component: demo-mysql
    kind: Service
    name: demo-mysql
    role: workload
    status: 'Type: ClusterIP, Cluster-IP: 1.254.72.82, Port(s): 3306/TCP.'
  - apiVersion: v1
    component: demo-redis
    kind: Service
    name: demo-redis
    role: workload
    status: 'Type: ClusterIP, Cluster-IP: 1.254.86.25, Port(s): 6379/TCP.'
  - apiVersion: apps/v1
    component: demo-mysql
    kind: Deployment
    name: demo-mysql
    role: workload
    status: 'Ready: 1/1, Up-to-date: 1, Available: 1.'
  - apiVersion: apps/v1
    component: demo-redis
    kind: Deployment
    name: demo-redis
    role: workload
    status: 'Ready: 1/1, Up-to-date: 1, Available: 1.'
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl -n oam-oamdemo describe appconfig demo
Name:         demo
Namespace:    oam-oamdemo
Labels:       <none>
Annotations:  <none>
API Version:  core.oam.dev/v1alpha1
Kind:         ApplicationConfiguration
Metadata:
  Creation Timestamp:  2020-05-06T12:21:22Z
  Generation:          1
  Resource Version:    189215
  Self Link:           /apis/core.oam.dev/v1alpha1/namespaces/oam-oamdemo/applicationconfigurations/demo
  UID:                 18c2c59e-8f94-11ea-8257-00163e146fef
Spec:
  Components:
    Component Name:  demo-tomcat
    Instance Name:   demo-tomcat
    Parameter Values:
    Traits:
      Name:  better-auto-scaler
      Properties:
        Cpu - Up:     50
        Maximum:      3
        Memory - Up:  50
        Minimum:      2
      Name:           log-pilot
      Properties:
        Container:         tomcat
        Name:              logindex
        Path:              /usr/local/tomcat/logs
        Pilot Log Prefix:  aliyun
        Tags:              owner:appconfig/demo
      Name:                nginx-ingress
      Properties:
        Hostname:      tomcat.example
        Path:          /
        Service Port:  8080
    Component Name:    demo-mysql
    Instance Name:     demo-mysql
    Parameter Values:
      Name:   password
      Value:  passwordtest
    Traits:
      Name:  manual-scaler
      Properties:
        Replica Count:  1
      Name:             resources-policy
      Properties:
        Container:  mysql
        Limits:
          Cpu:     1000m
          Memory:  1024Mi
      Name:        volume-mounter
      Properties:
        Storage Class:  default
        Volume Name:    mysql-data
    Component Name:     demo-redis
    Instance Name:      demo-redis
    Parameter Values:
    Traits:
      Name:  host-policy
      Properties:
        Host Ipc:      false
        Host Network:  true
        Host Pid:      false
      Name:            manual-scaler
      Properties:
        Replica Count:  1
      Name:             schedule-policy
      Properties:
        Node Affinity:
          Type:  preferred
        Pod Affinity:
          Selector:
            App:  tomcat
          Type:   preferred
        Pod Anti Affinity:
          Type:  preferred
Status:
  Modules:
    Group Version:  core.oam.dev/v1alpha1
    Kind:           Server
    Name:           demo-tomcat
    Status:         Healthy
    Group Version:  core.oam.dev/v1alpha1
    Kind:           Server
    Name:           demo-mysql
    Status:         Healthy
    Group Version:  core.oam.dev/v1alpha1
    Kind:           Server
    Name:           demo-redis
    Status:         Healthy
  Phase:            Synced
  Resources:
    API Version:  apps/v1
    Component:    demo-tomcat
    Kind:         Deployment
    Name:         demo-tomcat
    Role:         workload
    Status:       Ready: 2/2, Up-to-date: 2, Available: 2.
    API Version:  v1
    Component:    demo-tomcat
    Kind:         Service
    Name:         demo-tomcat
    Role:         workload
    Status:       Type: ClusterIP, Cluster-IP: 1.254.119.183, Port(s): 8080/TCP.
    API Version:  extensions/v1beta1
    Component:    demo-tomcat
    Kind:         Ingress
    Name:         demo-tomcat
    Role:         trait
    Status:       Hosts: tomcat.example 
    API Version:  harmonycloud.cn/v1beta1
    Component:    demo-tomcat
    Kind:         HorizontalPodAutoscaler
    Name:         demo-tomcat
    Role:         trait
    Status:       CurrentReplicas: 2, DesiredReplicas: 2.
    API Version:  v1
    Component:    demo-mysql
    Kind:         PersistentVolumeClaim
    Name:         mysql-data
    Role:         trait
    Status:       Volume: pvc-18e5f923-8f94-11ea-8257-00163e146fef, Capacity: 10Gi, AccessModes: [ReadWriteOnce], StorageClass: default, Status: Bound.
    API Version:  v1
    Component:    demo-mysql
    Kind:         Service
    Name:         demo-mysql
    Role:         workload
    Status:       Type: ClusterIP, Cluster-IP: 1.254.72.82, Port(s): 3306/TCP.
    API Version:  v1
    Component:    demo-redis
    Kind:         Service
    Name:         demo-redis
    Role:         workload
    Status:       Type: ClusterIP, Cluster-IP: 1.254.86.25, Port(s): 6379/TCP.
    API Version:  apps/v1
    Component:    demo-mysql
    Kind:         Deployment
    Name:         demo-mysql
    Role:         workload
    Status:       Ready: 1/1, Up-to-date: 1, Available: 1.
    API Version:  apps/v1
    Component:    demo-redis
    Kind:         Deployment
    Name:         demo-redis
    Role:         workload
    Status:       Ready: 1/1, Up-to-date: 1, Available: 1.
Events:
  Type     Reason       Age                From               Message
  ----     ------       ----               ----               -------
  Normal   Created      36m                hc-oam-controller  Resource deployments/demo-tomcat created successfully
  Normal   Created      36m                hc-oam-controller  Resource services/demo-tomcat created successfully
  Normal   Created      36m                hc-oam-controller  Resource ingresses/demo-tomcat created successfully
  Normal   Created      36m                hc-oam-controller  Resource hchpas/demo-tomcat created successfully
  Normal   Created      36m                hc-oam-controller  Resource persistentvolumeclaims/mysql-data created successfully
  Normal   Created      36m                hc-oam-controller  Resource deployments/demo-mysql created successfully
  Normal   Created      36m                hc-oam-controller  Resource services/demo-mysql created successfully
  Normal   Created      36m                hc-oam-controller  Resource deployments/demo-redis created successfully
  Normal   Created      36m                hc-oam-controller  Resource services/demo-redis created successfully
  Warning  Sync Failed  36m (x5 over 36m)  hc-oam-controller  Operation cannot be fulfilled on applicationconfigurations.core.oam.dev "demo": the object has been modified; please apply your changes to the latest version and try again
  Normal   Synced       35m (x7 over 36m)  hc-oam-controller  Sync Successfully
[root@iZ2zeh0tj14w6z89nobjtrZ oamdemo]# kubectl -n oam-oamdemo get deploy,svc,pod,ingress,pvc
NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/demo-mysql    1/1     1            1           37m
deployment.extensions/demo-redis    1/1     1            1           37m
deployment.extensions/demo-tomcat   2/2     2            2           37m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/demo-mysql    ClusterIP   1.254.72.82     <none>        3306/TCP   37m
service/demo-redis    ClusterIP   1.254.86.25     <none>        6379/TCP   37m
service/demo-tomcat   ClusterIP   1.254.119.183   <none>        8080/TCP   37m

NAME                               READY   STATUS    RESTARTS   AGE
pod/demo-mysql-56fd44f7bd-jljj6    1/1     Running   0          37m
pod/demo-redis-5646dc74c4-97fj5    1/1     Running   0          37m
pod/demo-tomcat-7bdbf7cd7c-smnc9   1/1     Running   0          37m
pod/demo-tomcat-7bdbf7cd7c-tkz7s   1/1     Running   0          37m

NAME                             HOSTS            ADDRESS   PORTS   AGE
ingress.extensions/demo-tomcat   tomcat.example             80      37m

NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mysql-data   Bound    pvc-18e5f923-8f94-11ea-8257-00163e146fef   10Gi       RWO            default        37m
```