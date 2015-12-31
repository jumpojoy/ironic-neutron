Ironic Neutron integration testing
==================================

1. Devstack configuration guilde is available at

   ```
   https://review.openstack.org/#/c/258596/
   ```

2. Create SSH key

   ```
   nova keypair-add ironic_key > ironic_key.pem
   ```

3. Get user image_id

   ```
   image=$(nova image-list | egrep "cirros-0.3.4-x86_64-disk"'[^-]' | awk '{ print $2 }')
   ```

4. Get private network id

   ```
   net_id=$(neutron net-list | egrep "private"'[^-]' | awk '{ print $2 }')
   ```

5. Boot the instance

   ```
   nova boot --flavor baremetal --image $image --key-name ironic_key --nic net-id=$net_id vm_name
   ```

6. Check that during provision ironic node is in provision network.
   Node will recieve different IP during provision and in ACTIVE state.
   Find provision provider:segmentation_id field.

   ```
   neutron net-show ironic-provision
   ```

7. vm port should be dynamicly plugged to segmentation_id vlan. Example:

   ```
   sudo ovs-vsctl show
   ...
   Port "ovs-vm-0"
     tag: 215
     Interface "ovs-vm-0"
     type: internal
   ...
   ```

8. when node is in active state check that tag has been changed to private network. Example:

   ```
   sudo ovs-vsctl show
   ...
   Port "ovs-vm-0"
     tag: 218
     Interface "ovs-vm-0"
     type: internal
   ...
   ```
