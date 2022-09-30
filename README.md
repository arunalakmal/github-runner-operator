# GitHub Runner Operator

This is an Operator to deploy the GitHub Runners on the OpenShift/Kubernetes Clusters. This project was started to facilitate the CI operations running on the private Openshift Container platform. For example, OpenShift builds and Tekton pipelines. It was inherited from the customized Official OpenShift Runners. 

GitHub Actions Pipelines act as the middle man between the GitHub repository and the OpenShift Container Platform. Operator also supports to run the Maven builds creating the custom settings file inside the runner pod. Project intent to use the JFrog Artifactory and it act as the cache, a private artifacts and repositories, to facilitate that requirement it creates the ´settings.xml´ generated by the JFrog Artifactory Maven repository in the pod's ´/home/runner/.m2/´ path. The content of the ´settings.xml´ file should be updated in the runner deployment manifest. 

These runners are also supported with repository level and organization level runner registrations. 

Eventhough this Operator and the runners created for the OpenShift Container platform there is no OpenShift specific implementation which means this can be used with any Kubernetes cluster. 

## Authentication

Runners support two authentication options. 
- Uising PAT Tokens
- Using GitHub Apps  

If it is an Organisationl level runner registration GitHub Apps auth method is supported and usage of the runners can be controlled using the ´Runner Groups´. 

## Operator Installation

Currently deployment supports with the manifests as we don't have a Helm Chart created to deploy the Operator. In order to deploy this Operator you need to have the ´cluster-admin´ privileges.

´´´
oc create -f <>
´´´

There will be a new project/namespace created named ´github-runner-operator-system´ and Operator will be deployed. Operator Will be deployed to this Project/Namespace. Operator will be looking into the all namespaces for the runners.  

## Runner Deployments

Once the Operator is ready, runners can be deployed in any namespace using the runner manifests. 

Sample Runner Manifest with the GitHub App Authentication and Maven Settings File. GitHub App will be generated a SSH key and that key should be added to the runner manifest. This is the full list of options and only necessary parameters are needed. 

Sample Manifest 

´´´
apiVersion: operators.ikea.com/v1alpha1
kind: GitHubRunners
metadata:
  name: githubrunners-sample
spec:
  affinity: {}
  annotations: {}
  appIdSecretKey: github-app-id
  appInstallIdSecretKey: github-install-id
  appName: actions-runner
  appPemSecretKey: github-pem
  appSecretName: github-app
  clusterPKI: false
  cpuLimit: 250m
  cpuRequest: 100m
  ephemeral: false
  githubAppId: ""
  githubAppInstallId: ""
  githubAppPem: |-
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEA7cJYIxBKhasq1EJiqecdoE57uoJfDQigrtpnLwGDhC1p0Ebe
    9wwLESQV4l3O6R8Rz3eczEEweYecnBk2839M4pCYr/QVITnhQE+EAFz+Lr5ciVrW
    UlYXn<THIS IS AN INVALID KEY UPDATE THE GHAPP KEY HERE>N2fe4jAng
    PC6WL26a9l9TBurjrbHQiE8RQCgN7CqD8yf8voSPM4h4qHmGlIlpXn/rLuVeCrdG
    /FCkHtISbTfC8LumJ0Ghh2c2FWQO5VsaUSyHVRJHhHELgP9ibuq/5Q==
    -----END RSA PRIVATE KEY-----
  githubDomain: ""
  githubOwner: Inter-IKEA-Digital
  githubPat: ""
  githubRepository: ""
  imagePullSecrets: {}
  mavenconfig:
    enabled: true
    settingsFile: |-
      <?xml version="1.0" encoding="UTF-8"?>
      <settings xmlns="http://maven.apache.org/SETTINGS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">
        <servers>
            <server>
              <username>username</username>
              <password>password</password>
              <id>central</id>
            </server>
            <server>
              <username>username</username>
              <password>password</password>
              <id>snapshots</id>
            </server>
        </servers>
        <profiles>
            <profile>
              <repositories>
                  <repository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>central</id>
                    <name>libs-release</name>
                    <url>http://repo1.maven.org/maven2/</url>
                  </repository>
                  <repository>
                    <snapshots />
                    <id>snapshots</id>
                    <name>libs-snapshot</name>
                    <url>http://repo1.maven.org/maven2/</url>
                  </repository>
              </repositories>
              <pluginRepositories>
                  <pluginRepository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>central</id>
                    <name>plugins-release</name>
                    <url>http://repo1.maven.org/maven2/</url>
                  </pluginRepository>
                  <pluginRepository>
                    <snapshots />
                    <id>snapshots</id>
                    <name>plugins-snapshot</name>
                    <url>http://repo1.maven.org/maven2/</url>
                  </pluginRepository>
              </pluginRepositories>
              <id>artifactory</id>
            </profile>
        </profiles>
        <activeProfiles>
            <activeProfile>artifactory</activeProfile>
        </activeProfiles>
        <proxies>
            <proxy>
              <id>My Proxy</id>
              <active>true</active>
              <protocol>http</protocol>
              <host>proxy.company.com</host>
              <port>8080</port>
              <username>dummy</username>
              <password>password</password>
              <nonProxyHosts>www.google.com|*.example.com</nonProxyHosts>
            </proxy>
        </proxies>
      </settings>
  memoryLimit: 1Gi
  memoryRequest: 512Mi
  nodeSelector: {}
  replicas: 1
  runnerEnv: null
  runnerGroup: ""
  runnerImage: quay.io/redhat-github-actions/runner
  runnerLabels: []
  runnerTag: v1
  secretKey: github-pat
  secretName: github-pat
  serviceAccountName: default
´´´

Create an object with the above manifest (´runner.yaml´). 

´´´
oc create -f runner.yaml
´´´

Check the pod status and the runner status in the GitHub. 