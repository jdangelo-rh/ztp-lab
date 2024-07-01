# ztp-lab

## Crucible

docs: https://github.com/redhat-partner-solutions/crucible


`dnf -y install ansible-core python3-netaddr skopeo podman openshift-clients ipmitool python3-pyghmi python3-jmespath nmstate`


`ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa`


`subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms`

 `git clone https://github.com/redhat-partner-solutions/crucible`

` cd crucible`

`ansible-galaxy collection install -r requirements.yml`

```shell
cp inventory.yml.sample <cluster_id>_inventory.yml
vi <cluster_id>_inventory.yml
```
```yaml
all:
  vars:
    ##################################
    # Assisted Install Configuration #
    ##################################
    # These options configure Assisted Installer and the resulting cluster
    # https://generator.swagger.io/?url=https://raw.githubusercontent.com/openshift/assisted-service/58a6abd5c99d4e41d939be89cd0962433849a861/swagger.yaml
    # See section: cluster-create-params

    # Cluster name and dns domain combine to give the cluster namespace that will contain OpenShift endpoints
    # e.g. api.clustername.example.lab, worker1.clustername.example.lab
    cluster_name: ocp4
    base_dns_domain: dominio.com

    # OpenShift version (4.6.16, 4.7.52, 4.8.43, 4.9.45, 4.10.37, 4.11.24, 4.12.31, or 4.13.10)
    openshift_full_version: 4.14.15

    #########################################################################################################################################################    
    os_images:
      - openshift_version: "4.14"
        cpu_architecture: "x86_64"
        url: "https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.14/4.14.15/rhcos-4.14.15-x86_64-live.x86_64.iso"
        rootfs_url: "https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.14/4.14.15/rhcos-4.14.15-x86_64-live-rootfs.x86_64.img"
        version: "414.92.202402261929-0"

    release_images:
      - openshift_version: "4.14"
        cpu_architecture: "x86_64"
        cpu_architectures:
          - "x86_64"
        url: "quay.io/openshift-release-dev/ocp-release:4.14.15-x86_64"
        version: "4.14.15"
    ###########################################################################################################################################################

    # Virtual IP addresses used to access the resulting OpenShift cluster
    api_vip: 10.60.0.96 # the IP address to be used for api.clustername.example.lab and api-int.clustername.example.lab
    ingress_vip: 10.60.0.97 # the IP address to be used for *.apps.clustername.example.lab

    ## Allocate virtual IPs via DHCP server. Equivalent to the vip_dhcp_allocation configuration option of Assisted Installer
    vip_dhcp_allocation: false

    # The subnet on which all nodes are (or will be) accessible.
    machine_network_cidr: 10.60.0.0/24

    # The IP address pool to use for service IP addresses
    service_network_cidr: 172.30.0.0/16

    # Check network connection when vm_host and bastion are the same 
    vm_network_test_ip: 8.8.8.8 # This should be an IP external to the bastion and vm_host

    # Cluster network settings. You are unlikely to need to change these
    cluster_network_cidr: 10.128.0.0/14 # The subnet, internal to the cluster, on which pods will be assigned IPs
    cluster_network_host_prefix: 23 # The subnet prefix length to assign to each individual node.

    # # Cluster network provider. Cannot be changed after cluster is created.
    # # The default is OpenShift SDN unless otherwise specified.
    network_type: OVNKubernetes
    # network_type: OpenShiftSDN

    extra_machine_networks:
      - cidr: fd00:6:6:2051::/64
    extra_service_networks:
      - cidr: fd02::/112
    extra_cluster_networks:
      - cidr: fd01::/48
        host_prefix: 64

    # Next two variables only supported on OCP >= 4.12
    extra_api_vip: fd00:6:6:2051::96
    extra_ingress_vip: fd00:6:6:2051::97



    ######################################
    # Prerequisite Service Configuration #
    ######################################

    # Proxy settings. These settings apply to: Assisted Installer, Day1 clusters and Day2 clusters.
    # This assumes the host where the AI runs and the OpenShift cluster share the same proxy settings.
    # http_proxy: ""
    # https_proxy: ""
    # no_proxy: ""

    # Flags to enable/disable prerequisite service setup
    # You will need to ensure alternatives are available for anything that will not be automatically set up
    setup_ntp_service: false
    setup_dns_service: false
    setup_pxe_service: false
    setup_registry_service: false # Only required for a Restricted Network installation
    setup_http_store_service: true
    setup_assisted_installer: true # default is true you may wish to turn it off if multiple users are using the same instance.


    # NTP Service
    # ntp_server is the address at which the NTP service is (or will be) available
    ntp_server: 10.40.0.100
    # ntp_server_allow is the range of IPs the NTP service will respond to
    # ntp_server_allow: 10.40.0.0/24 # not required if setup_ntp_service is false


    # Mirror Registry Service parameters for a Restricted Network installation

    # use_local_mirror_registry controls if the install process uses a local container registry (mirror_registry) or not.
    # Set this to true to use the mirror registry service set up when `setup_registry_service` is true.
    use_local_mirror_registry: false

    # HTTP Store Configuration
    # ISO name must include the `discovery` directory if you have a SuperMicro machine
    discovery_iso_name: "discovery/{{ cluster_name }}/discovery-image.iso"

    # discovery_iso_server must be discoverable from all BMCs in order for them to mount the ISO hosted there.
    # It is usually necessary to specify different values for KVM nodes and/or physical BMCs if they are on different subnets.
    discovery_iso_server: "http://{{ hostvars['http_store']['ansible_host'] }}"

    ############################
    # Local File Configuration #
    ############################

    path_base_dir: /home/redhat

    repo_root_path: "{{ path_base_dir }}/crucible" # path to repository root

    # Directory in which created/updated artifacts are placed
    fetched_dest: "{{ repo_root_path }}/fetched"

    # Configure possible paths for the pull secret
    # first one found will be used
    # note: paths should be absolute
    pull_secret_lookup_paths:
      - "{{ fetched_dest }}/pull-secret.txt"
      - "{{ repo_root_path }}/pull-secret.txt"

    # Configure possible paths for the ssh public key used for debugging
    # first one found will be used
    # note: paths should be absolute
    ssh_public_key_lookup_paths:
      - "{{ fetched_dest }}/ssh_keys/{{ cluster_name }}.pub"
      - "{{ repo_root_path }}/ssh_public_key.pub"
      - ~/.ssh/id_rsa.pub

    # Set the base directory to store ssh keys
    ssh_key_dest_base_dir: "{{ path_base_dir }}"
    # The retrieved cluster kubeconfig will be placed on the bastion host at the following location
    kubeconfig_dest_dir: "{{ path_base_dir }}"
    kubeconfig_dest_filename: "{{ cluster_name }}-kubeconfig"
    kubeadmin_dest_filename: "{{ cluster_name }}-kubeadmin.vault.yml"
    # You can comment out the line below if you want the kubeadmin credentials to be stored in plain text
    kubeadmin_vault_password_file_path: "{{ repo_root_path }}/kubeadmin_vault_password_file"

    ############################
    #    LOGIC: DO NOT TOUCH   #
    # vvvvvvvvvvvvvvvvvvvvvvvv #
    ############################

    # pull secret logic, no need to change. Configure above
    local_pull_secret_path: "{{ lookup('first_found', pull_secret_lookup_paths) }}"
    pull_secret: "{{ lookup('file', local_pull_secret_path) }}"

    # ssh key logic, no need to change. Configure above
    local_ssh_public_key_path: "{{ lookup('first_found', ssh_public_key_lookup_paths) }}"
    ssh_public_key: "{{ lookup('file', local_ssh_public_key_path) }}"

    # provided mirror certificate logic, no need to change.
    local_mirror_certificate_path: "{{ (setup_registry_service == true) | ternary(
        fetched_dest + '/' + (hostvars['registry_host']['cert_file_prefix'] | default('registry')) + '.crt',
        repo_root_path + '/mirror_certificate.txt')
      }}"
    mirror_certificate: "{{ lookup('file', local_mirror_certificate_path) }}"

    openshift_version: "{{ openshift_full_version.split('.')[:2] | join('.') }}"

    is_valid_single_node_openshift_config: "{{ (groups['nodes'] | length == 1) and (groups['masters'] | length == 1) }}"

    ############################
    # ^^^^^^^^^^^^^^^^^^^^^^^^ #
    #    LOGIC: DO NOT TOUCH   #
    ############################


  children:
    bastions: # n.b. Currently only a single bastion is supported
      hosts:
        bastion:
          ansible_host:  10.40.0.100 # Must be reachable from the Ansible control node
          ansible_connection: local # if your are not running crucible from the bastion then remove this line

    # Configuration and access information for the pre-requisite services
    # TODO: document differences needed for already-deployed and auto-deployed
    services:
      hosts:
        assisted_installer:
          ansible_host: bastion.dominio.com
          host: bastion.dominio.com
          port: 8090 # Do not change

        registry_host:
          ansible_host: registry.example.lab
          registry_port: 5000
          registry_fqdn: registry.example.lab # use in case of different FQDN for the cert
          registry_namespace: ocp4 # This is the default, use only in case the registry namespace name is different
          registry_image: openshift4 # This is the default, use only in case the registry image name is different
          cert_common_name: "{{ registry_fqdn }}"
          cert_country: US
          cert_locality: Raleigh
          cert_organization: Red Hat, Inc.
          cert_organizational_unit: Lab
          cert_state: NC

          # Configure the following secret values in the inventory.vault.yml file
          REGISTRY_HTTP_SECRET: "{{ VAULT_REGISTRY_HOST_REGISTRY_HTTP_SECRET | mandatory }}"
          disconnected_registry_user: "{{ VAULT_REGISTRY_HOST_DISCONNECTED_REGISTRY_USER | mandatory }}"
          disconnected_registry_password: "{{ VAULT_REGISTRY_HOST_DISCONNECTED_REGISTRY_PASSWORD | mandatory }}"

        dns_host:
          ansible_host: 10.40.0.100
          # upstream_dns: 8.8.8.8 # an optional upstream dns server
          # The following are required for DHCP setup
          # use_dhcp: true
          # use_pxe: false
          # dhcp_range_first: 10.60.0.101
          # dhcp_range_last:  10.60.0.105
          # prefix: 24
          # gateway: 10.60.0.1

        http_store:
          ansible_host: 10.40.0.100

        tftp_host:
          ansible_host: "{{ hostvars['dns_host']['ansible_host'] }}"
          tftp_directory: /var/lib/tftpboot/

        ntp_host:
          ansible_host: 10.40.0.100

    vm_hosts:
      hosts:
        vm_host1: # Required for using "KVM" nodes, ignored if not.
          ansible_user: root
          ansible_host: vm_host.clustername.example.lab
          host_ip_keyword: ansible_host # the varname in the KVM node hostvars which contains the *IP* of the VM
          images_dir: "{{ path_base_dir }}/libvirt/images" # directory where qcow images will be placed.
          vm_bridge_ip: 10.60.0.190 # IP for the bridge between VMs and machine network
          vm_bridge_interface: ens7f1 # Interface to be connected to the bridge. DO NOT use your primary interface.
          dns: 10.40.0.100 # DNS used by the bridge
          # ssl cert configuration
          # sushy_fqdn: ... # use in case of different FQDN for the cert
          cert_vars_host_var_key: registry_host # Look up cert values from another host by name (excluding cert_common_name)
          # or
          # cert_country: US
          # cert_locality: Raleigh
          # cert_organization: Red Hat, Inc.
          # cert_organizational_unit: Lab
          # cert_state: NC

    # Describe the desired cluster members
    nodes:
      vars:
        # Set the login information for any BMCs. Note that these will be SET on the vm_host virtual BMC.
        bmc_user: "{{ VAULT_NODES_BMC_USER | mandatory }}"
        bmc_password: "{{ VAULT_NODES_BMC_PASSWORD | mandatory }}"
        interface: "bond0"
        dns: "10.40.0.100"
        dnsipv6: "fd00:6:6:11::52"
        mask: 28
        maskipv6: 64
        gateway: "10.60.0.1"
        gatewayipv6: "fd00:6:6:2051::1"
        network_config:
          interfaces:
            - name: "{{ interface }}"
              type: bond
              state: up
              #mtu: 9000
              link_aggregation:
                mode: 802.3ad
                options:
                  miimon: "100"
                slaves:
                  - eth0
                  - eth1
              addresses:
                ipv4:
                  - ip: "{{ ansible_host}}"
                    prefix: "{{ mask }}"
                ipv6:
                  - ip: "{{ ipv6_address }}"
                    prefix: "{{ maskipv6 }}"
            - name: eth0
              type: ethernet
              mac: "{{ bondmac1 }}"
              #mtu: 9000
            - name: eth1
              type: ethernet
              mac: "{{ bondmac2 }}"
              #mtu: 9000                 
          # Only one DNS server ip per protocol
          dns_server_ips:
            - {{ dnsipv6 }}
            - {{ dns }}           
          routes:
            - destination: "0:0:0:0:0:0:0:0/0"
              address: "{{ gatewayipv6 }}"
              interface: "{{ interface }}"
            - destination: 0.0.0.0/0
              address: "{{ gateway }}"
              interface: "{{ interface }}"

      children:
        masters:
          vars:
            role: master
            vendor: HPE

          hosts:
            master1:
              ansible_host: 10.60.0.101
              ipv6_address: fd00:6:6:2051::101
              bmc_address: "<ip del puerto management>"
              mac: "DE:AD:BE:EF:C0:2C"
              bondmac1: "DE:AD:BE:EF:C0:2C"
              bondmac2: "DE:AD:BE:EF:C0:2D"

            master2:
              ansible_host: 10.60.0.102
              ipv6_address: fd00:6:6:2051::102
              bmc_address: "<ip del puerto management>"
              mac: "DE:AD:BE:EF:C0:2E"
              bondmac1: "DE:AD:BE:EF:C0:2E"
              bondmac2: "DE:AD:BE:EF:C0:2F"              

            master3:
              ansible_host: 10.60.0.103
              ipv6_address: fd00:6:6:2051::103             
              bmc_address: "<ip del puerto management>"
              mac: "DE:AD:BE:EF:C0:2G"
              bondmac1: "DE:AD:BE:EF:C0:2G"
              bondmac2: "DE:AD:BE:EF:C0:2H"  
```

```shell
cp inventory.vault.yml.sample inventory.vault.yml
vi inventory.vault.yml
```
```yaml
---
#######################################
# Prerequisite services configuration #
#######################################

# HTTP Secret for the Container Registry.
# More information on the vars used to configure the Registry can be found here:
# https://docs.docker.com/registry/configuration/#http
VAULT_REGISTRY_HOST_REGISTRY_HTTP_SECRET: SECRET

# Credentials for the Disconnected Registry (if relevant)
VAULT_REGISTRY_HOST_DISCONNECTED_REGISTRY_USER: USER
VAULT_REGISTRY_HOST_DISCONNECTED_REGISTRY_PASSWORD: PASSWORD

#######################
# Nodes configuration #
#######################

# Default credentials for the BMCs
VAULT_NODES_BMC_USER: USER
VAULT_NODES_BMC_PASSWORD: PASSWORD

# # Set custom BMC credentials for super1 and worker1 nodes.
# # These vault variables then have to be referenced in the inventory file.
# VAULT_NODES_SUPER1_BMC_USER: USER
# VAULT_NODES_SUPER1_BMC_PASSWORD: PASSWORD
# VAULT_NODES_WORKER1_BMC_USER: USER
# VAULT_NODES_WORKER1_BMC_PASSWORD: PASSWORD

```

- Ejecutar los siguientes comandos si se usó vault encriptado

`ansible-playbook -i <cluster_id>_inventory.yml deploy_prerequisites.yml -e "@inventory.vault.yml" --ask-vault-pass`

`ansible-playbook -i <cluster_id>_inventory.yml deploy_cluster.yml -e "@inventory.vault.yml" --ask-vault-pass`

- Ejecutar los siguientes comandos si no se usó vault  encriptado

`ansible-playbook -i <cluster_id>_inventory.yml deploy_prerequisites.yml -e "@inventory.vault.yml"`

`ansible-playbook -i <cluster_id>_inventory.yml deploy_cluster.yml -e "@inventory.vault.yml"` 

- Ejecutar los siguientes comandos si no se usó vault

`ansible-playbook -i <cluster_id>_inventory.yml deploy_prerequisites.yml `

`ansible-playbook -i <cluster_id>_inventory.yml deploy_cluster.yml`

-------------------------------------------------------------------------------------------------------------------------------------------------------
## ZTP

- Instalar ACM 

- Instalar Gitops

- Instalar TALM

- Crear provisioning 

```yaml
apiVersion: metal3.io/v1alpha1
kind: Provisioning
metadata:
  name: provisioning-configuration
spec:
  provisioningNetwork: "Disabled"
  watchAllNamespaces: true
```

- Crear  AgentServiceConfig

```yaml
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

```
- Crear Repo de ejemplo

`$ podman pull registry.redhat.io/openshift4/ztp-site-generate-rhel8:latest`

`$ mkdir -p ./out`

`$ podman run --rm --log-driver=none ztp-site-generate-rhel8:latest extract /home/ztp --tar | tar x -C ./out`

*out/extra-manifest: contains the source CRs files that SiteConfig uses to generate extra manifest configMap.
out/source-crs: contains the source CRs files that PolicyGenTemplate uses to generate the ACM policies.
out/argocd/deployment: contains patches and yaml file to apply on the hub cluster for use in the next step of this procedure.
out/argocd/example: contains example SiteConfig and PolicyGenTemplate that represent our recommended configuration.*

- Patchear Argocd

`$ oc patch argocd openshift-gitops -n openshift-gitops  --type=merge --patch-file out/argocd/deployment/argocd-openshift-gitops-patch.json`

- Create a git repository with directory structure similar to the example directory.
```rst
Configure access to the repository using the ArgoCD UI. Under Settings configure:
*** Repositories --> Add connection information (URL ending in .git, eg https://repo.example.com/repo.git, and credentials)
*** Certificates --> Add the public certificate for the repository if needed
Modify the two ArgoCD Applications (out/argocd/deployment/clusters-app.yaml and out/argocd/deployment/policies-app.yaml) based on your GIT repository:
*** Update URL to point to git repository. The URL must end with .git, eg: https://repo.example.com/repo.git
*** The targetRevision should indicate which branch to monitor
*** The path should specify the path to the SiteConfig or PolicyGenTemplate CRs respectively
```


- Crear Secret con los valores de las BMC y Pull-Secret:

```yaml
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

```

- Crear Siteconfig dentro del path configurado en out/argocd/deployment/clusters-app.yaml

```yaml
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
    holdInstallation: false
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
```
- Apply pipeline configuration to your hub cluster using the following command.

`$ oc apply -k out/argocd/deployment`

- Sincronizar Argocd

- Aplicar secret creado anteriormente

`$ oc apply -f secret.yaml`

- Monitoreo del progreso

```bash
$ export CLUSTER=<clusterName>
$ oc get agentclusterinstall -n $CLUSTER $CLUSTER -o jsonpath='{.status.conditions[?(@.type=="Completed")]}' | jq
$ curl -sk $(oc get agentclusterinstall -n $CLUSTER $CLUSTER -o jsonpath='{.status.debugInfo.eventsURL}')  | jq '.[-2,-1]'
```
