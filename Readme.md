# kubeha

Simple opinionated high availability kubernetes cluster deployed in libvirt. 

APIServer connections can be  made highly available for kube-controller-manager and scheduler as well and can be toggled with `kcm_scheduler_with_ha_apiserver_connection` variable in `vars.yaml` (WARNING: setting to true can cause problems with upgrades).

# Creating the cluster

1. Install ansible requirements: `ansible-galaxy collection install -r requirements.yml`
2. Create `kubeha` network with `ansible-playbook base/network-init.yml`
3. Prepare base VM
    1. Install fedora rawhide and name it fedora-rawhide-base. Select `kubeha` network as a source of the VM's NIC. 
    2. Input the base VM name and IP address into `hosts` file
    3. Start the base VM and create ssh keys: `ansible-playbook base/base-vm-start.yml`
    4. Copy public key into the VM: `ssh-copy-id -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -i auth/id_rsa root@${BASE_VM_IP}`
    5. Prepare the base VM and turn it off: `ansible-playbook base/base-vm-prepare.yml`
4. Clone fedora-rawhide-base into as many masters and workers as desired via `virt-manager`
5. Start all the VMs to obtain generated IP addresses
6. Insert the VM names and IP addresses into `hosts` file
7. Inspect vars.yaml for any customization
8. Install DNS servers: `ansible-playbook install-dns.yml`
9. Install the cluster: `ansible-playbook install-cluster.yml`
10. Either add `dns` group IPs from `./hosts` file as your DNS server or add the following entry to your hosts file: `echo '192.168.150.2 api-kube.kubeha.knet' >> /etc/hosts` 
11. Use ./lifecycle and ./cluster scripts for management of the cluster
