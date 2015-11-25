Ironic Neutron integration testing
==================================

1. Get devstack repo and create stack user

   ```
   git clone https://github.com/openstack-dev/devstack.git devstack
   sudo ./devstack/tools/create-stack-user.sh
   ```

2. Sudo with stack and get devstack

   ```
   sudo su - stack
   git clone https://github.com/openstack-dev/devstack.git devstack
   ```

3. Clone this repo and copy example ironic-neutron-vm-local.conf to ~/devstac/local.conf

   ```
   git clone https://github.com/jumpojoy/ironic-neutron.git
   mv ironic-neutron/devstack/examples/ironic-neutron-vm-local.conf devstack/local.conf
   ```

4. Set password for stack user. It is needed in order to connect to localhost via SSH by ML2 driver.
   In future ssh-key support will be added

   ```
   sudo su -c "echo 'stack:r00tme' | chpasswd"
   ```

5. Add ML2 generic_switch config. Update username/password accordinly

   ```
   sudo mkdir -p /etc/neutron/plugins/ml2/
   sudo chown -R stack:stack /etc/neutron/
   cat >/etc/neutron/plugins/ml2/ml2_conf_genericswitch.ini <<END
   [genericswitch:brbm]
   device_type=ovs_linux
   ip=localhost
   username=stack
   password=r00tme
   END
   ```

6. Apply patches that allow to test ironic neutron integration

   ```
   cd devstack/
   git fetch https://review.openstack.org/openstack-dev/devstack refs/changes/74/248074/6 && git checkout -b ironic-neutron FETCH_HEAD
   ./stack.sh
   ```

7. Create SSH key

   ```
   nova keypair-add ironic_key > ironic_key.pem
   ```

8. Get user image_id

   ```
   image=$(nova image-list | egrep "cirros-0.3.4-x86_64-disk"'[^-]' | awk '{ print $2 }')
   ```

9. Get private network id

   ```
   net_id=$(neutron net-list | egrep "private"'[^-]' | awk '{ print $2 }')
   ```

10. Boot the instance

   ```
   nova boot --flavor baremetal --image $image --key-name ironic_key --nic net-id=$net_id vm_name
   ```

11. Check that during provision ironic node is in provision network.
   Node will recieve different IP during provision and in ACTIVE state.
   Find provision provider:segmentation_id field.

   ```
   neutron net-show ironic-provision
   ```

12. vm port should be dynamicly plugged to segmentation_id vlan. Example:

   ```
   sudo ovs-vsctl show
   ...
   Port "ovs-vm-0"
     tag: 215
     Interface "ovs-vm-0"
     type: internal
   ...
   ```

13. when node is in active state check that tag has been changed to private network. Example:

   ```
   sudo ovs-vsctl show
   ...
   Port "ovs-vm-0"
     tag: 218
     Interface "ovs-vm-0"
     type: internal
   ...
   ```
