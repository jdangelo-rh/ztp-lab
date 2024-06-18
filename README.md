# ztp-lab

Crucible

docs: https://github.com/redhat-partner-solutions/crucible


dnf -y install ansible-core python3-netaddr skopeo podman openshift-clients ipmitool python3-pyghmi python3-jmespath nmstate


ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa


subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms

git clone https://github.com/redhat-partner-solutions/crucible

cd crucible

ansible-galaxy collection install -r requirements.yml

cp inventory.yml.sample <cluster_id>_inventory.yml
vi <cluster_id>_inventory.yml

cp inventory.vault.yml.sample inventory.vault.yml
vi inventory.vault.yml


ansible-playbook -i <cluster_id>_inventory.yml deploy_prerequisites.yml -e "@inventory.vault.yml" --ask-vault-pass

ansible-playbook -i <cluster_id>_inventory.yml deploy_cluster.yml -e "@inventory.vault.yml" --ask-vault-pass

-------------------------------------------------------------------------------------------------------------------------------------------------------
ZTP

Instalar ACM 

Instalar Gitops

Instalar TALM

Crear provisioning 

apiVersion: metal3.io/v1alpha1
kind: Provisioning
metadata:
  name: provisioning-configuration
spec:
  provisioningNetwork: "Disabled"
  watchAllNamespaces: true

Crear  AgentServiceConfig

apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
 name: agent
spec:
  databaseStorage:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: <db_volume_size>
  filesystemStorage:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: <fs_volume_size>
  imageStorage:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: <img_volume_size>  

$ podman pull registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.16.0-34

$ mkdir -p ./out
$ podman run --rm --log-driver=none ztp-site-generator:latest extract /home/ztp --tar | tar x -C ./out

**** out/extra-manifest: contains the source CRs files that SiteConfig uses to generate extra manifest configMap.
**** out/source-crs: contains the source CRs files that PolicyGenTemplate uses to generate the ACM policies.
**** out/argocd/deployment: contains patches and yaml file to apply on the hub cluster for use in the next step of this procedure.
**** out/argocd/example: contains example SiteConfig and PolicyGenTemplate that represent our recommended configuration.

$ oc patch argocd openshift-gitops -n openshift-gitops  --type=merge --patch-file out/argocd/deployment/argocd-openshift-gitops-patch.json

Create a git repository with directory structure similar to the example directory.
Configure access to the repository using the ArgoCD UI. Under Settings configure:
*** Repositories --> Add connection information (URL ending in .git, eg https://repo.example.com/repo.git, and credentials)
*** Certificates --> Add the public certificate for the repository if needed
Modify the two ArgoCD Applications (out/argocd/deployment/clusters-app.yaml and out/argocd/deployment/policies-app.yaml) based on your GIT repository:
*** Update URL to point to git repository. The URL must end with .git, eg: https://repo.example.com/repo.git
*** The targetRevision should indicate which branch to monitor
*** The path should specify the path to the SiteConfig or PolicyGenTemplate CRs respectively
Apply pipeline configuration to your hub cluster using the following command.
    oc apply -k out/argocd/deployment

Crear Secret con los valores de las BMC:

apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: bmh-secret-1
  namespace: cwl-site1
data:
  username: cm9vdAo=
  password: cG9vbDJkcmFjCg==
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: bmh-secret-2
  namespace: cwl-site1
data:
  username: cm9vdAo=
  password: cG9vbDJkcmFjCg==
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: bmh-secret-3
  namespace: cwl-site1
data:
  username: cm9vdAo=
  password: cG9vbDJkcmFjCg==
---
apiVersion: v1
data:
  .dockerconfigjson: <pull-secret>
kind: Secret
metadata:
  name: assisted-deployment-pull-secret
  namespace: cwl-site1
type: kubernetes.io/dockerconfigjson

Crear Siteconfig dentro del path configurado en out/argocd/deployment/clusters-app.yaml

apiVersion: ran.openshift.io/v1
kind: SiteConfig
metadata:
  name: "cwl-site1"
  namespace: "cwl-site1"
spec:
  baseDomain: "jpatete.info"
  pullSecretRef:
    name: "assisted-deployment-pull-secret"
  clusterImageSetNameRef: "openshift-4.14"
  sshPublicKey: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDikFJUEHrienq0oIzOIFPgBARssBwVgNTKU7Vhe37lMTYnCcu3GNNdryE/ZXBzD4Q1lEAWf1nAUM8zbndsgtrx7uups/eKAgv0FiaSplFS3nNI9Wtdc0Zg0ynO2NUhx505vlV/4nGYAolj3hqF3QtbVFLR83S9A8DdlUEufA/PLLe0FmITu4qHMDAmuTwABPkdi8MdAkyBQxop+9mnsC2BMabRtnxIZa3tADYYUF1RGjTgCFmntEdVff2w3eakNsYo0WRxEFgBoz5Oco0mXOEA/imur5TE+PXmZGtyvyFLEVQoHJin+sHfAmv70pUfBXh/aEgEtES3Y2zaMtflt6kBghCJC1plxZWBhv32SX/Y2rj/rXuBZFlYp6pRKtCxzraFTbl2z5ORSgJpIvXQvb8tPdR9se/KEixiGjV/duLetwvNXDRenXpJCqhwR1sIMpcPdqkgXI1KvQKcx4S1SsTzHuT0l33gyDe+HPeqMGmoFaJIsybYG2zSznELnuYlFYEbeHB5tWyZKb82habRbwAO9j4jEyaEDvfM6d9f+p23L6DhATOMdARZHNALMaMPNNzrKOZg4lL2ygH+hysdPAjhJg76iWjqqJVrluF03RZwTcuXsdbFX9pVYXoZwYkB4bCYWv4aVkskVjujgzxQXgZ2yRDSiII2Jg2gnfmHr9mHXw=="
  clusters:
  - clusterName: "cwl-site1"
    networkType: "OVNKubernetes"
    clusterLabels:
      cluster: cwl-site1
    clusterNetwork:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
    machineNetwork:
      - cidr: 10.1.198.32/28
    # For 3-node and standard clusters with static IPs, the API and Ingress IPs must be configured here
    apiVIP: 10.1.198.43
    apiVIPs:
      - 10.1.198.44
    ingressVIP:  10.1.198.43
    ingressVIPs:
      - 10.1.198.44
    serviceNetwork:
      - 172.30.0.0/16
    additionalNTPSources:
      - 10.11.160.238
      # - 1111:2222:3333:4444::2
    # Optionally; This can be used to override the KlusterletAddonConfig that is created for this cluster:
    #crTemplates:
    #  KlusterletAddonConfig: "KlusterletAddonConfigOverride.yaml"
    # Optionally; This can be used to to configure desired BIOS setting on all nodes in this cluster:
    #biosConfigRef:
    #  filePath: "example-hw.profile"
################################################################
    nodes:
      - hostName: "master1.cwl-site1.jpatete.info"
        role: "master"
        bmcAddress: "idrac-virtualmedia+https://10.1.177.21/redfish/v1/Systems/System.Embedded.1"
        bmcCredentialsName:
          name: "bmh-secret-1"
        bootMACAddress: "b4:96:91:25:93:20"
        bootMode: "UEFISecureBoot"  
        # Use UEFISecureBoot to enable secure boot for this node
        # bootMode: "UEFI"
        nodeLabels:
          cluster.ocs.openshift.io/openshift-storage: "true"
        rootDeviceHints:
          deviceName: "/dev/sda"
# As per: https://docs.openshift.com/container-platform/4.11/scalability_and_performance/cnf-talm-for-cluster-upgrades.html#cnf-about-topology-aware-lifecycle-manager-config_cnf-topology-aware-lifecycle-manager
        #diskPartition:
        #  - device: /dev/sda
        #    partitions:
        #    - mount_point: /var/recovery
        #      size: 51200
        #      start: 800000
        nodeNetwork:
          interfaces:
            - name: "eno12399"
              macAddress: "b4:96:91:25:93:20"
            - name: "eno12409"
              macAddress: "b4:96:91:25:93:22"
          config:
            interfaces:
              - name: eno12409
                type: ethernet
                state: up
                ipv4:
                  address:
                  - ip: 10.1.198.34
                    prefix-length: 24
                  enabled: true
                ipv6:
                  enabled: false
            dns-resolver:
              config:
                server:
                  - 10.1.198.36
            routes:
              config:
                - destination: 0.0.0.0/0
                  next-hop-address: 10.1.198.46
############################################################
      - hostName: "master2.cwl-site1.jpatete.info"
        role: "master"
        bmcAddress: "idrac-virtualmedia+https://10.1.177.22/redfish/v1/Systems/System.Embedded.1"
        bmcCredentialsName:
          name: "bmh-secret-2"
        bootMACAddress: "b4:96:91:1d:7a:54"
        bootMode: "UEFISecureBoot"  
        # Use UEFISecureBoot to enable secure boot for this node
        # bootMode: "UEFI"
        nodeLabels:
          cluster.ocs.openshift.io/openshift-storage: "true"
        rootDeviceHints:
          deviceName: "/dev/sda"
# As per: https://docs.openshift.com/container-platform/4.11/scalability_and_performance/cnf-talm-for-cluster-upgrades.html#cnf-about-topology-aware-lifecycle-manager-config_cnf-topology-aware-lifecycle-manager
        #diskPartition:
        #  - device: /dev/sda
        #    partitions:
        #    - mount_point: /var/recovery
        #      size: 51200
        #      start: 800000
        nodeNetwork:
          interfaces:
            - name: "eno12399"
              macAddress: "b4:96:91:1d:7a:54"
            - name: "eno12409"
              macAddress: "b4:96:91:1d:7a:56"
          config:
            interfaces:
              - name: eno12409
                type: ethernet
                state: up
                ipv4:
                  address:
                  - ip: 10.1.198.37
                    prefix-length: 24
                  enabled: true
                ipv6:
                  enabled: false
            dns-resolver:
              config:
                server:
                  - 10.1.198.36
            routes:
              config:
                - destination: 0.0.0.0/0
                  next-hop-address: 10.1.198.46
############################################################
      - hostName: "master3.cwl-site1.jpatete.info"
        role: "master"
        bmcAddress: "idrac-virtualmedia+https://10.1.177.23/redfish/v1/Systems/System.Embedded.1"
        bmcCredentialsName:
          name: "bmh-secret-3"
        bootMACAddress: "b4:96:91:1d:72:6c"
        bootMode: "UEFISecureBoot"  
        # Use UEFISecureBoot to enable secure boot for this node
        # bootMode: "UEFI"
        nodeLabels:
          cluster.ocs.openshift.io/openshift-storage: "true"
        rootDeviceHints:
          deviceName: "/dev/sda"
# As per: https://docs.openshift.com/container-platform/4.11/scalability_and_performance/cnf-talm-for-cluster-upgrades.html#cnf-about-topology-aware-lifecycle-manager-config_cnf-topology-aware-lifecycle-manager
        #diskPartition:
        #  - device: /dev/sda
        #    partitions:
        #    - mount_point: /var/recovery
        #      size: 51200
        #      start: 800000
        nodeNetwork:
          interfaces:
            - name: "eno12399"
              macAddress: "b4:96:91:1d:72:6c"
            - name: "eno12409"
              macAddress: "b4:96:91:1d:72:6e"
          config:
            interfaces:
              - name: eno12409
                type: ethernet
                state: up
                ipv4:
                  address:
                  - ip: 10.1.198.38
                    prefix-length: 24
                  enabled: true
                ipv6:
                  enabled: false
            dns-resolver:
              config:
                server:
                  - 10.1.198.36
            routes:
              config:
                - destination: 0.0.0.0/0
                  next-hop-address: 10.1.198.46
