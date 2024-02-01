# kubeha

Simple opinionated high availability Kubernetes cluster deployed in libvirt. 

APIServer connections can be made highly available for kube-controller-manager and scheduler as well and can be toggled with the `kcm_scheduler_with_ha_apiserver_connection` variable in `vars.yaml` (WARNING: setting to true can cause problems with upgrades).

# Creating the Cluster

1. Install ansible requirements: `ansible-galaxy collection install -r requirements.yml`
2. Copy the `hosts.example` file into the `hosts` file.
3. Create `kubeha` network with `ansible-playbook base/network-init.yml`
4. Prepare the base VM
    1. Install fedora rawhide and name it fedora-rawhide-base. Select `kubeha` network as the source of the VM's NIC. 
    2. Input the base VM name and IP address into `hosts` file
    3. Start the base VM and create ssh keys: `ansible-playbook base/base-vm-start.yml`
    4. Copy the public key into the VM: `ssh-copy-id -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -i auth/id_rsa root@${BASE_VM_IP}`
    5. Prepare the base VM and turn it off: `ansible-playbook base/base-vm-prepare.yml`
5. Clone fedora-rawhide-base into as many masters and workers as desired via `virt-manager`.
6. Start all the VMs to obtain generated IP addresses.
7. Insert the VM names and IP addresses into the `hosts` file.
8. Inspect vars.yaml for any customization.
9. Install the DNS servers: `ansible-playbook install-dns.yml`
10. Install the cluster: `ansible-playbook install-cluster.yml`
11. Either add `dns` group IPs from the `./hosts` file as your DNS server, or add the following entry to your hosts file: `echo '192.168.150.2 api-kube.kubeha.knet' >> /etc/hosts` 
12. Use ./lifecycle and ./cluster scripts for management of the cluster.


# Upgrading the Cluster

Run `ansible-playbook cluster/upgrade-to-latest.yml` to upgrade the cluster, the system and its packages to the latest version.


### Upgrading or Downgrading the Cluster to a Specific Version

This option will only upgrade or downgrade the kubernetes packages, not the whole system.
There is no guarantee that this will work.

1. Set a `k8s_version` variable in `vars.yaml` to the desired Kubernetes version.
2. Run `ansible-playbook cluster/upgrade-downgrade-to-version.yml` to upgrade or downgrade the cluster.
3. The `force_upgrade_downgrade` variable can be set to `true` in `vars.yaml` if you encounter errors (e.g. when downgrading).

Upgrade versions and validations can be checked by sshing into a master node and running `kubeadm upgrade plan`.
