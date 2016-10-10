===================================
Ironic with portgroups and DevStack
===================================

This guide with show how to prepare devstack environment with portgroup support.


DevStack Configuration with portgroups
--------------------------------------
The following is ``local.conf`` example that will setup Devstack with 3 VMs that
are registered in ironic and configured with portgroups.

::

    [[local|localrc]]
    Q_USE_DEBUG_COMMAND=True
    NETWORK_GATEWAY=10.1.0.1
    DEST=/opt/stack/new
    # move DATA_DIR outside of DEST to keep DEST a bit cleaner
    DATA_DIR=/opt/stack/data
    ACTIVE_TIMEOUT=90
    BOOT_TIMEOUT=90
    ASSOCIATE_TIMEOUT=60
    TERMINATE_TIMEOUT=60
    MYSQL_PASSWORD=secretmysql
    DATABASE_PASSWORD=secretdatabase
    RABBIT_PASSWORD=secretrabbit
    ADMIN_PASSWORD=secretadmin
    SERVICE_PASSWORD=secretservice
    SERVICE_TOKEN=111222333444
    SWIFT_HASH=1234123412341234
    ROOTSLEEP=0
    ENABLED_SERVICES=ceilometer-acentral,ceilometer-acompute,ceilometer-alarm-evaluator,ceilometer-alarm-notifier,ceilometer-anotification,ceilometer-api,ceilometer-collector,dstat,g-api,g-reg,horizon,ir-api,ir-cond,key,mysql,n-api,n-cond,n-cpu,n-crt,n-obj,n-sch,q-agt,q-dhcp,q-l3,q-meta,q-metering,q-svc,quantum,rabbit,s-account,s-container,s-object,s-proxy,tempest
    SKIP_EXERCISES=boot_from_volume,bundle,client-env,euca
    SERVICE_HOST=127.0.0.1
    # Screen console logs will capture service logs.
    SYSLOG=False
    SCREEN_LOGDIR=/opt/stack/new/screen-logs
    LOGFILE=/opt/stack/new/devstacklog.txt
    VERBOSE=True
    FIXED_RANGE=10.1.0.0/20
    FLOATING_RANGE=172.24.5.0/24
    PUBLIC_NETWORK_GATEWAY=172.24.5.1
    FIXED_NETWORK_SIZE=4096
    VIRT_DRIVER=ironic
    SWIFT_REPLICAS=1
    LOG_COLOR=False
    # Don't reset the requirements.txt files after g-r updates
    UNDO_REQUIREMENTS=False
    CINDER_PERIODIC_INTERVAL=10
    export OS_NO_CACHE=True
    CEILOMETER_BACKEND=mysql
    LIBS_FROM_GIT=
    DATABASE_QUERY_LOGGING=True
    # set this until all testing platforms have libvirt >= 1.2.11
    # see bug #1501558
    EBTABLES_RACE_FIX=True
    CINDER_SECURE_DELETE=False
    CINDER_VOLUME_CLEAR=none
    IRONIC_DEPLOY_DRIVER=agent_ssh
    IRONIC_BAREMETAL_BASIC_OPS=True
    IRONIC_VM_LOG_DIR=/opt/stack/new/ironic-bm-logs
    DEFAULT_INSTANCE_TYPE=baremetal
    BUILD_TIMEOUT=600
    IRONIC_CALLBACK_TIMEOUT=600
    Q_AGENT=openvswitch
    Q_ML2_TENANT_NETWORK_TYPE=vxlan
    IRONIC_BUILD_DEPLOY_RAMDISK=False
    SWIFT_ENABLE_TEMPURLS=True
    SWIFT_TEMPURL_KEY=secretkey
    IRONIC_ENABLED_DRIVERS=fake,agent_ssh,agent_ipmitool
    IRONIC_VM_EPHEMERAL_DISK=0
    IRONIC_VM_SPECS_RAM=1024
    VOLUME_BACKING_FILE_SIZE=24G
    TEMPEST_HTTP_IMAGE=http://git.openstack.org/static/openstack.png
    FORCE_CONFIG_DRIVE=False

    IRONIC_VM_COUNT=3
    TEMPEST_PLUGINS=/opt/stack/new/ironic
    enable_plugin ironic git://git.openstack.org/openstack/ironic refs/changes/76/382476/4
    IRONIC_DEPLOY_DRIVER_ISCSI_WITH_IPA=True
    IRONIC_VM_SPECS_RAM=1024
    IRONIC_RAMDISK_TYPE=tinyipa
    enable_plugin networking-generic-switch git://git.openstack.org/openstack/networking-generic-switch
    Q_PLUGIN_EXTRA_CONF_PATH=/etc/neutron/plugins/ml2
    Q_PLUGIN_EXTRA_CONF_FILES['networking-generic-switch']=ml2_conf_genericswitch.ini
    IRONIC_USE_LINK_LOCAL=True
    OVS_PHYSICAL_BRIDGE=brbm
    PHYSICAL_NETWORK=mynetwork
    IRONIC_PROVISION_NETWORK_NAME=ironic-provision
    IRONIC_PROVISION_SUBNET_PREFIX=10.0.5.0/24
    IRONIC_PROVISION_SUBNET_GATEWAY=10.0.5.1
    Q_PLUGIN=ml2
    ENABLE_TENANT_VLANS=True
    Q_ML2_TENANT_NETWORK_TYPE=vlan
    TENANT_VLAN_RANGE=100:150
    IRONIC_ENABLED_NETWORK_INTERFACES=flat,neutron
    IRONIC_NETWORK_INTERFACE=neutron
    IRONIC_USE_BOND=True

    NOVA_BRANCH=refs/changes/63/206163/14

    LIBS_FROM_GIT+=",python-ironicclient"
    IRONICCLIENT_BRANCH=refs/changes/30/362130/12

    # Use image with static bond until configdrive part is ready.
    IMAGE_URLS+="https://www.dropbox.com/s/owby2p2kln1mojm/cirros-disk-bonding-static.qcow2"
    IRONIC_IMAGE_NAME=cirros-disk-bonding-static

Verification
------------

1. Run tempest ironic-portgroup test

   ```
   cd /opt/stack/new/tempest
   tox -eall-plugin -- test_baremetal_bonding --concurrency=1
   ```
