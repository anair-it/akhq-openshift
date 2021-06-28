# AKHQ Helm Chart
- https://github.com/tchiotludo/akhq
- This is a fork of [AKHQ helm|https://github.com/tchiotludo/akhq/tree/dev/helm/akhq] that will work in an Openshift service mesh platform


## Chart Details
- Openshift compatible AHKQ helm chart
- View brokers, topics, consumer groups, messages 
- Single installation to connect to multiple kafka brokers
- Integrate with Kafka broker, Kafka connect, schema registry
- UI authentication using Basic auth/OIDC/Ldap
- View Avro schemas
- Filter messages by key and values
- Openshift RBAC controls

## AKHQ up and running
- Identify the namespace to install AKHQ
- Determine if AKHQ should run in a service mesh or not. By default, service mesh deployment is ON.
  - To turn OFF service mesh and instead pick route based access
    - set `route.enabled: true` and set
    ```yaml
    serviceMesh:
      route: false
      sidecar: false
    ```
- Create a secret called `akhq-secrets`. Refer the below template
    - Enter broker and schema registry connection info and credentials
    - If using basic auth, identify the password, convert password to 256 hash using `echo -n "password" | sha256sum`
    - Add more roles and basic auth users as required. Refer [AKHQ security|https://github.com/tchiotludo/akhq#security] to configure security and roles
    ```yaml
    kind: Secret
    apiVersion: v1
    type: Opaque
    metadata:
      name: akhq-secrets
    labels:
        app.kubernetes.io/instance: akhq
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: akhq
        helm.sh/chart: akhq-0.1.3
    data:
      application-secrets.yml: >-
        micronaut:
            security:
                enabled: true
        akhq:
            connections:
                ccloud-dev:
                    properties:
                        bootstrap.servers: 
                        security.protocol: SASL_SSL
                        sasl.mechanism: PLAIN
                        sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="" password="";
                    schema-registry:
                        url: 
                        type: "confluent"
                        basic-auth-username: 
                        basic-auth-password: 
                ccloud-qa:
                    properties:
                        bootstrap.servers: 
                        security.protocol: SASL_SSL
                        sasl.mechanism: PLAIN
                        sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="" password="";
                    schema-registry:
                        url: 
                        type: "confluent"
                        basic-auth-username: 
                        basic-auth-password: 
            security:
                groups:
                    topic-reader:
                        name: topic-reader
                        roles:
                        - topic/read
                        - node/read
                        - topic/data/read
                        - group/read
                        - registry/read

                basic-auth:
                    - username: 
                      password: [SHA256 password]
                      groups:
                       - topic-reader
    ```
- Identify the gateway hostname. This is the externally exposed url like _akhq-anoop-dev.host.com_. Follow the format _[CHART_NAME].[NAMESPACE].[FQDN]_
- Install AKHQ chart using helm install. Sample helm install: 
```bash
helm install akhq . --set hostname=akhq-anoop-dev.host.com -n anoop-dev
```
- After AKHQ pod is up, fire up a browser and navigate to `https://{{ template "akhq.fullname" . }}-{{ .Release.Namespace }}.{FQDN}/`
- Navigate to each topic and view topic, consumer metadata, view messages