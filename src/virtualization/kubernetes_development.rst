Kubernetes Development
======================

.. contents:: Table of Contents

API
---

Stability
~~~~~~~~~

New Kubernetes APIs go through a life-cycle of updates before being marked as stable. Below are the different possible stages in ascending order.

-  Development or Prototype = Not found in any official releases. May not work.
-  Alpha = Partially implemented. Disabled by default. Versioning starts with ``v1alpha1``.
-  Beta = Feature complete. Includes mostly completed API test coverage. Upgrades may break. Versioning starts with ``v1beta1``.
-  Stable or General Availability (GA) = Fully supported in Kubernetes. Will remain backwards compatible with upgrades. Versioning starts with ``v1``.

[4]

Types
~~~~~

All of the available APIs are categorized into these types:

-  Cluster
-  Config and Storage
-  Metadata
-  Service
-  Workloads

[5]

Resources
~~~~~~~~~

Resource APIs are used to create objects in Kubernetes. They define the desired state of objects. Controllers are used to enforce that state. Every object manifest has the following fields. Typically these are defined declaratively via a YAML manifest file.

-  **apiVersion (string)** = The version of the API. Normally ``v1`` or ``<APIGROUP>/v1``.
-  **kind (string)** = Name of the API to create an object from.
-  **metadata (dictionary)** = Metadata for the object.

   -  **name (string)** = The unique name of this object. Only one object with this Resoure kind and name can exist in a namespace.
   -  **labels (dictionary)** = Any key-value pair to help identify this object. This is optional but recommended to help find specific or related objects.

-  **spec (dictionary)** = Provide information on how this object will be created and used. Valid inputs are different for every API. Not all APIs will have a spec.
-  status = The current state information for the object. This can be shown via ``kubectl get <RESOURCE_API> <OBJECT> -o yaml``.

.. code-block:: yaml

   ---
   apiVersion: <RESOURCE_APIGROUP>/<RESOURCE_APIVERSION>
   kind: <RESOURCE_KIND>
   metadata:
     name: <OBJECT_NAME>
     labels:
       <KEY>: <VALUE>
   spec:

[6]

List the values for each Resource such as the ``<NAME>``, ``<APIGROUP>``, ``<KIND>``, and if it supports namespaces. Further documentation on all of the available configuration fields for a Resource can also be shown.

.. code-block:: sh

   $ kubectl api-resources
   $ kubectl explain <RESOURCE_NAME>
   $ kubectl explain <RESOURCE_NAME>.spec --recursive
   $ kubectl explain <RESOURCE_NAME> --recursive

View the ``<RESOURCE_APIGROUP>/<RESOURCE_APIVERSION>`` versions available to use.

.. code-block:: sh

   $ kubectl api-versions

Show all objects from one of the Resource APIs.

.. code-block:: sh

   $ kubectl get <RESOURCE_NAME>

View details about an object.

.. code-block:: sh

   $ kubectl describe <RESOURCE_NAME> <OBJECT_NAME>

[7]

Edit or view the YAML configuration for an existing object.

.. code-block:: sh

   $ kubectl edit <RESOURCE_NAME> <OBJECT_NAME>
   $ kubectl get <RESOURCE_NAME> <OBJECT_NAME> -o yaml --export

Create a basic template for a Deployment or any object. It can be saved and used as a starting point for a new template. No object will be created.

.. code-block:: sh

   $ kubectl run <DEPLOYMENT_NAME> --image=<CONTAINER_IMAGE_NAME> --dry-run -o yaml
   $ kubectl create <RESOURCE_NAME> <OBJECT_NAME> --dry-run -o yaml

[8]

Imperative and Declarative Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Imperative

   -  `commands <https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/>`__ = Using only the CLI (no configuration file) to create and manage resources. Syntax: ``kubectl run`` for Pods and ``kubectl create <RESOURCE_API>`` for most other resources.
   -  `object configuration <https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-config/>`__ = Using the CLI and an existing configuration file/directory to create and manage resources. Syntax: ``kubectl {create,delete,get,replace} -f <FILE>.yaml``.

-  Declarative

   -  `object configuration <https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/>`__ = Directly apply a configuration and change it's state using a manifest file. Syntax: ``kubectl {apply,diff} -f <FILE>.yaml``.

A YAML file can be used to define an object that will be created using an API resource. This is commonly called a manifest, definition, declarative, or an object configuration file. Once it has been applied it becomes a live object configuration that is stored in Kubernetes back-end database. It is recommended to use declarative objects because they can be easily tracked and updated through a source code management (SCM) such as git. [9]

**Run Generators**

In Kubernetes < 1.18, the imperative command ``kubectl run`` would create a Deployment. It could optionally be used to create a Pod instead.

.. code-block:: sh

   $ kubectl run <DEPLOYMENT_NAME> --image=<IMAGE>
   kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.

.. code-block:: sh

   $ kubectl run --generator=run-pod/v1 <POD_NAME> --image=<IMAGE>

In Kubernetes >= 1.18, the command can only create a Pod. This is to align the command with the functionality of ``docker run``.

.. code-block:: sh

   $ kubectl run <POD_NAME> --image=<IMAGE>

[10]

API Resources
-------------

Each section lists the following information:

-  <API_GROUP>

   -  <API_RESOURCE> = <DESCRIPTION>

A manifest file can be created to use the resource following this format:

.. code-block:: yaml

   ---
   apiVersion: <GROUP>/<API_VERSION>
   kind: <API_RESOURCE>
   metadata:
     name: <NAME>
   spec:

Information about every API can be found be using the ``kubectl explain`` command, viewing the `API Reference Docs <https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/>`__, or viewing the `Kubernetes Documentation <https://kubernetes.io/docs/home/>`__.

Cluster
~~~~~~~

Cluster APIs are used by Kubernetes cluster operators to define how it is configured. [5] These are not to be confused with the singular `Cluster API <https://kind.sigs.k8s.io/>`__ that is used to create development Kubernetes clusters using containers.

-  apiregistration.k8s.io

   -  APIService = Add third-party Kubernetes APIs.

-  auditregistration.k8s.io

   -  AuditSink = Audit a Kubernetes cluster dynamically with webhooks.

-  authentication.k8s.io

   -  TokenRequest = Create a token.
   -  TokenReview = Verify if a token is authenticated.

-  authorization.k8s.io

   -  LocalSubjectAccessReview = Check if a specific action can be used by a user within a namespace.
   -  SelfSubjectAccessReview = Check if a specific action can be used by the current user.
   -  SelfSubjectRulesReview = View the actions the current user can do in a namespace.
   -  SubjectAccessReview = Check if a specific action can be used by a user.

-  certificates.k8s.io

   -  CertificateSigningRequest = Force certificates to be signed either automatically or manually.

-  coordination.k8s.io

   -  Lease = Provides an efficient heartbeat from the kubelet service to let the kube-controller-manager know it is still available.

-  core

   -  Binding = Bind objects together.
   -  ComponentStatus = Provides the status of Kubernetes cluster services such as etcd, kube-scheduler, and kube-controller-manager.
   -  Namespace = Create namespaces for developers to isolate their objects.
   -  Node = Manage attributes of any Node (Control Plane or Worker).
   -  PersistentVolume = Manage persistent and stateful volumes. PersistentVolumeClaims can be created from this object.
   -  ResourceQuota = Manage resource allocations and limits.
   -  ServiceAccount = Manage Kubernetes accounts that are used by automation tools (not humans).

-  flowcontrol.apiserver.k8s.io

   -  FlowSchema = Assign priorities to incoming requests.
   -  PriorityLevelConfiguration = Manage the limit of outstanding and queued requests to the kube-apiserver.

-  networking.k8s.io

   -  NetworkPolicy = Manage Pod networks. The network plugin in the Kubernetes cluster has to support this feature (not every plugin does).

-  node.k8s.io

   -  RuntimeClass = Configure containerd or CRI-O runtimes. This can then be used by a Pod.

-  rbac.authorization.k8s.io

   -  ClusterRole = Role-based access control (RBAC) for all resources regardless of namespace separation.
   -  ClusterRoleBinding = A list of users and their permissions for a given ClusterRole.
   -  Role = RBAC for all namespaced resources.
   -  RoleBinding = A list of users and their permissions for a given Role.

Namespace
^^^^^^^^^

-  API group / version (latest): v1
-  Shortname: ns
-  Namespaced: false

----

``ns.spec:``

-  finalizers (list of strings) = This list must be empty before a namespace can be deleted. It can contain any arbitrary values.

----

**Examples:**

NS example.

.. code-block:: yaml

   ---
   kind: Namespace
   apiVersion: v1
   metadata:
     name: new-namespace

NS with finalizers.

.. code-block:: yaml

   ---
   kind: Namespace
   apiVersion: v1
   metadata:
     name: my-namespace
   spec:
     finalizers:
       - foo
       - bar

[5]

NetworkPolicy
^^^^^^^^^^^^^

-  API group / version (latest): networking.k8s.io/v1
-  Shortname: netpol
-  Namespaced: true

----

``netpol.spec:``

-  egress (map)

   -  ports (list of maps)

      -  port (string)
      -  protocol (string)

   -  to (list of maps)

      -  ipBlock (map) = IP addresses that are allowed.

         -  **cidr** (string) = A CIDR of IP addresses to allow.
         -  except (list of strings) = A CIDR of IP addresses to exclude from the ``cidr`` range.

      -  namespaceSelector (`map of Selector <#selector>`_) = The Namespace to apply the NetworkPolicy for. By default, it is the Namespace that the Pod is in. If this field is empty, it will apply the NetworkPolicy to all Namespaces.
      -  podSelector (`map of Selector <#selector>`_) = The Pod to apply the NetworkPolicy to. If this field is empty, the NetworkPolicy will apply to all Pods.

-  ingress (map)

   -  ports (list of maps)

      -  port (string)
      -  protocol (string)

   -  from (list of maps)

      -  ipBlock (map)

         -  **cidr** (string)
         -  except (list of strings)

      -  namespaceSelector (`map of Selector <#selector>`_)
      -  podSelector (`map of Selector <#selector>`_)

-  **podSelector** (`map of Selector <#selector>`_)
-  policyTypes (list of strings) = Optionally explicitly define the NetworkPolicy type. If not defined, it will be determined based on if ``netpol.spec.egress`` and/or ``netpol.spec.ingress`` are defined. By defining only "Ingress" or "Egress", the opposite traffic type will be completely disallowed.

    -  Ingress
    -  Egress
    -  "Ingress,Egress"

[5]

PersistentVolume
^^^^^^^^^^^^^^^^

-  API group / version (latest): v1
-  Shortname: pv
-  Namespaced: false

----

``pv.spec:``

-  **accessModes** (list) [2]

   -  ReadOnlyMany = More than one Pod can only read the data to/from this storage
   -  ReadWriteOnce = Only one Pod can read and write to/from this storage.
   -  ReadWriteMany = More than one Pod can read and write data to/from this storage.

-  **capacity (map)**

   -  **storage (string)** = The capacity, in "Gi", that the PV pool contains.

-  claimRef (map) = A reference to bind this PVC object to a PV object.
-  mountOptions (list) = Linux mount options for the PVC on a Pod.
-  nodeAffinity (map) = NodeAffinity settings for selecting what Worker Nodes this PVC should be used on.
-  persistentVolumeReclaimPolicy (string) = What to do when the volume is no longer required by a Pod.

   -  Retain = Default for manually provisioned PV.
   -  Delete = Default for dynamically provisioned PV.

-  **storageClassName (string)** = Any unique name or the name of an existing StorageClass to inherit attributes from. It is used by PVCs to identify the PV to create storage from. Leave blank to use the default StorageClass (if one exists).
-  volumeMode (string) = The volume type required for the PVC object.

**Storage plugin types (select one and then configure the map of settings):**

-  awsElasticBlockStore
-  azureDisk
-  azureFile
-  cephfs

   -  **monitors** (list of strings) = Ceph monitors to connect to.
   -  path (string) = Default is /. The mounted root.
   -  readOnly (boolean) - If the PV will be read-only.
   -  secretFile (string) = Default is /etc/ceph/user.secret. The key ring file used for authenticating as the RADOS user.
   -  secretRef (map)

      -  name (string) = The name of the Secret object that contains the RADOS key ring file. Use "key" as the key name in the Secret.

   -  user (string) = The RADOS user.

-  csi
-  cinder = OpenStack's Block-Storage-as-a-Service.

   -  fsType (string) = Default is ext4. The file system of the volume.
   -  readOnly (boolean)
   -  secretRef (map) = Authentication details for OpenStack.
   -  **volumeID** (string) = The Cinder volume ID to use.

-  fc (Fibre Channel)
-  flexVolume
-  flocker
-  gcePersistentDisk
-  glusterfs

   -  **endpoints** (string) = The Endpoint that is tied to all of the GlusterFS server IPs.
   -  endpointsNamespace (string) = The namespace the Endpoint is in.
   -  **path** = The GlusterFS network volume/share name.
   -  readOnly (boolean)

-  hostPath = Use a local directory on a Worker Node to store data. Set a "nodeAffinity" to the Worker Node that will have the hostPath directory and data available.

   -  **path** (string) = The file system path to use.
   -  type (string) = How to manage the path.

      -  "" = No operation on the path.
      -  BlockDevice = Use a block device.
      -  CharDevice = Use a character device.
      -  Directory = Use an existing directory.
      -  DirectoryOrCreate = Create the directory if it does not exist.
      -  File = Use an existing file.
      -  FileOrCreate = Create the file if it does not exist.
      -  Socket = Use a UNIX socket.

-  iscsi

   -  chapAuthDiscovery (boolean)
   -  chapAuthSession (boolean)
   -  fsType (string)
   -  initiatorName (string) = Set a custom iSCSI Initiator name.
   -  **iqn** (string) = The iSCSI Target.
   -  iscsiInterface (string) = Default is default. The iSCSI Interface name.
   -  **lun** (integer) = The Target LUN number.
   -  portals (list of strings) = A list of ``<IP>:<PORT>`` strings for each iSCSI Portal.
   -  readOnly (boolean)
   -  secretRef (map)

      -  name (string) = The Secret object that contains the CHAP authentication details.

   -  **targetPortal** (string) = The primary iSCSI Target Portal to use.

-  local = Mount a local partition.

   -  fsType (string)
   -  **path** (string) = The full path to the partition to mount.

-  nfs

   -  **path** (string) = The NFS file share.
   -  readOnly (boolean)
   -  **server** (string) = The NFS server address.

-  photonPersistentDisk
-  portworxVolume
-  quobyte
-  rbd

   -  fsType (string)
   -  **image** (string) = The RADOS image to use.
   -  **monitors** (list of strings) = The list of Ceph monitors to connect to.
   -  pool (string) = The RADOS pool to use.
   -  readOnly (boolean)
   -  secretRef (map)

      - name (string) = The Secret name to used for authenticating as the RADOS user.

   -  user (string)

-  scaleIO
-  storageos
-  vsphereVolume

[5][21]

----

**Examples:**

PV with CephFS.

.. code-block:: yaml

   ---
   kind: Secret
   apiVersion: v1
   metadata:
     name: secret-cephfs-key
   data:
     key: lEhoWAwcyRxurSYkGwizxUtVFagtlPIJEntXmzNyfWaCmCMRRuliOr==

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-cephfs
   spec:
     accessModes:
       - ReadWriteMany
       - ReadWriteOnce
     capacity:
       storage: 100Gi
     cephfs:
       monitors:
         - 10.0.0.101
         - 10.0.0.102
         - 10.0.0.103
        secretRef:
          name: secret-cephfs-key
        user: foo

PV with OpenStack's Cinder block storage service. The Kubernetes cluster must first be `configured to work with OpenStack <https://docs.openshift.com/container-platform/3.11/install_config/configuring_openstack.html#install-config-configuring-openstack>`__.

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-cinder
   spec:
     accessModes:
       - ReadWriteMany
       - ReadWriteOnce
     capacity:
       storage: 10Gi
     cinder:
       fsType: ext4
       volumeID: d6dac7fb-e17f-44bb-9708-ee27a679273b

PV with GlusterFS. The GlusterFS client utility ``glusterfs-fuse`` needs to be installed on each Node. A Service and Endpoint are required to access the network shares. They both must share the same object name. The "ports" values are not used but are required by the APIs. [20]

.. code-block:: yaml

   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: glusterfs-network
   spec:
     ports:
       - port: 1
   ---
   kind: Endpoint
   apiVersion: v1
   metadata:
     name: glusterfs-network
   subsets:
     - addresses:
         - ip: 10.10.10.201
       ports:
         - port: 1
     - addresses:
         - ip: 10.10.10.202
       ports:
         - port: 1
     - addresses:
         - ip: 10.10.10.203
       ports:
         - port: 1

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-glusterfs
   spec:
     accessModes:
       - ReadWriteMany
       - ReadWriteOnce
     capacity:
       storage: 300Mi
     glusterfs:
       endpoints: glusterfs-network
       path: glusterVol

PV with hostPath.

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-hostpath
   spec:
     accessModes:
       - ReadWriteOnce
     capacity:
       storage: 50Mi
     hostPath:
       path: /var/lib/k8s-hospath
       type: DirectoryOrCreate

PV with iSCSI.

.. code-block:: yaml

   ---
   kind: Secret
   apiVersion: v1
   metadata:
     name: secret-iscsi-chap
   type: "kubernetes.io/iscsi-chap"
   data:
     discovery.sendtargets.auth.username:
     discovery.sendtargets.auth.password:
     discovery.sendtargets.auth.username_in:
     discovery.sendtargets.auth.password_in:
     node.session.auth.username:
     node.session.auth.password:
     node.session.auth.username_in:
     node.session.auth.password_in:

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-iscsi
   spec:
     accessModes:
       - ReadWriteOnce
     capacity:
       storage: 1Ti
     iscsi:
       chapAuthDiscovery: true
       chapAuthSession: true
       fsType: xfs
       iqn: iqn.food.bar.tld:example
       lun: 0
       readOnly: true
       secretRef:
         name: secret-iscsi-chap
       targetPortal: 192.168.1.15

PV with a local mount.

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-local
   spec:
     accessModes:
       - ReadWriteOnce
     capacity:
       storage: 500Gi
     local:
       fsType: xfs
       path: /dev/vd3

PV with Network File Share (NFS)

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-nfs
   spec:
     accessModes:
       - ReadWriteOnce
     capacity:
       storage: 1Gi
     nfs:
       path: "/"
       server: nfs.server.tld

PVC with RADOS Block Device (RBD).

.. code-block:: yaml

   ---
   kind: Secret
   apiVersion: v1
   metadata:
     name: secret-rbd-key
   data:
     key: eFuBtFpciHkPQBSrJXVpZnsfluklbDYnPRaLrfjoqGbnZfcfunlSyB==

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv-rbd
   spec:
     capacity:
       storage: 150Gi
     rbd:
       monitors:
         - 10.0.0.201
         - 10.0.0.202
         - 10.0.0.203
        secretRef:
          name: secret-rbd-key
        user: fu

[19]

ServiceAccount
^^^^^^^^^^^^^^

-  API group / version (latest): v1
-  Shortname: sa
-  Namespaced: true

----

There is no ``spec`` section for ServiceAccounts.

``sa:``

-  automountServiceAccountToken (boolean) = If the ServiceAccount token should be automatically mounted on Pods.
-  imagePullSecrets (list of maps) = A list of Secrets to use for pulling container images from a remote source.

   -  name (string) = The name of the Secret object.

-  secrets (list of maps) = A list of Secret objects that can be used for authenticating to the ServiceAccount.

   -  apiVersion (string)
   -  fieldPath (string)
   -  kind (string)
   -  **name** (string) = The name of the Secret object to use.
   -  namespace (string)
   -  resourceVersion (string)
   -  uid (string)

----

**Examples:**

ServiceAccount example. A random Secret token will automatically be generated.

.. code-block:: sh

   ---
   kind: ServiceAccount
   apiVersion: v1
   metadata:
     name: sa-bot
     namespace: ci-automation

ServiceAccount using an existing Secret token.

.. code-block:: sh

   ---
   kind: ServiceAccount
   apiVersion: v1
   metadata:
     name: sa-example
   secrets:
     - name: secret-foo-bar

[5]

Config and Storage
~~~~~~~~~~~~~~~~~~

Config and storage APIs manages key-value stores and persistent data storage. [5]

-  core

   -  ConfigMap = Manage key-value stores.
   -  Secret = Manage base64 encoded key-value stores.
   -  PersistentVolumeClaim = Manage persistent storage created from a PersistentVolume.
   -  Volume = Manage local or network volume mounts.

-  storage.k8s.io

   -  CSIDriver = Define how Kubernetes will interact with the CSI storage back-end.
   -  CSINode = Define CSI drivers.
   -  StorageClass = Manage the automatic creation of persistent storage.
   -  VolumeAttachment = Record when a CSI volume is created. This is used by other resources to then act upon the creation of the object.

ConfigMap
^^^^^^^^^

-  API group / version (latest): v1
-  Shortname: cm
-  Namespaced: true

ConfigMap does not have a ``cm.spec`` section. The ``cm.data:`` field is used the most.

``cm:``

-  binaryData (map) = Define key-value pairs where the value is a base64 encoded string.
-  data (map) = Define key-value pairs.
-  immutable (boolean) = If the key-value pairs in the object should be read-only.

[5]

----

**Examples:**

ConfigMap using all of it's available options.

.. code-block:: yaml

   ---
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: cm-env
   immutable: true
   data:
     hello: world
     foo: bar
   binaryData:
     goodbye: Y3J1ZWwgd29ybGQ=

PersistentVolumeClaim
^^^^^^^^^^^^^^^^^^^^^

-  API group / version (latest): v1
-  Shortname: pvc
-  Namespaced: true

----

Use either ``pvc.spec.selector``, ``pvc.spec.storageClassName``, or ``pvc.spec.volumeName`` to define what PersistentVolume to bind to.

``pvc.spec:``

-  **accessModes** (list of strings) = The accessModes to allow. The lists values must also be allowed in the PV.

   -  ReadOnlyMany
   -  ReadWriteOnce
   -  ReadWriteMany

-  dataSource (map) An existing object to create a new PVC object from.

   -  apiGroup (string) = The API group for the kind. Do not define this key if using PersistentVolume. Use "snapshot.storage.k8s.io" as the value for VolumeSnapshot.
   -  **kind** (string) = PersistentVolumeClaim or VolumeSnapshot.
   -  **name** (string) = The object name.

-  **resources** (map)

   -  limits (map) = The maximum storage allocation.

      -  storage (string) = Specify the requested storage size in the format ``<PVC_STORAGE>Gi``.

   -  **requests** (map) = The minimum storage allocation. This will be the default if ``limits`` is not defined.

      -  **storage** (string)

-  **selector** (`map of Selector <#selector>`_) = The key-value label pairs to find a PV to bind to.
-  **storageClassName** (string) = The StorageClass to create a PVC from.
-  volumeMode (string) = How to manage the PVC when attaching it to a Pod.

   -  Block = The block device will be formatted and then mounted.
   -  Filesystem = The filesystem will be mounted.

-  **volumeName** (string) = The PersistentVolume name to create a PVC from.

----

**Examples:**

PVC example.

.. code-block:: yaml

   ---
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: pvc-app
   spec:
     accessModes:
       - ReadWriteMany
       - ReadWriteOnce
     resources:
       requests:
         storage: 5Gi
     volumeName: <PERSISTENTVOLUME_NAME>

[5]

Secret
^^^^^^^

-  API group / version (latest): v1
-  Shortname: (None)
-  Namespaced: true

Secrets are **not** encrypted. They use base64 encoding. Secret does not have a ``secret.spec`` section. The ``secret.data:`` field is used the most.

``secret:``

-  data (map) = Define key-value pairs with base64 encoded values.
-  immutable (boolean) = If the key-value pairs in the object should be read-only.
-  stringData (map) = Define key-value pairs as strings. The values will be converted into base64 and merged into the ``secret.data`` section. The plain-text values will not be displayed by the API.
-  type (string) = The type of Secret to create. The full list can be found `here <https://github.com/kubernetes/kubernetes/blob/v1.18.0/pkg/apis/core/types.go#L4800-L4886>`__. By default, it is "Opaque" meaning that the key-value pairs are general purpose.

[5]

----

**Examples:**

Secret using all of it's available options.

.. code-block:: sh

   $ echo -n 'kenobi' | base64
   a2Vub2Jp

.. code-block:: yaml

   ---
   kind: Secret
   apiVersion: v1
   metadata:
     name: secret-http-auth
   immutable: true
   type: kubernetes.io/basic-auth
   stringData:
     username: obiwan
   data:
     password: a2Vub2Jp

.. code-block:: sh

   $ kubectl get secret secret-http-auth -o yaml | grep -A 2 ^data:
   data:
     password: a2Vub2Jp
     username: b2Jpd2Fu

[5]

Metadata
~~~~~~~~

Metadata APIs are used to change the behvaior of other objects. [5]

-  admissionregistration.k8s.io

   -  MutatingWebhookConfiguration = Validate and optionally modify API webhook requests.
   -  ValidatingWebhookConfiguration = Validate API webhook requests.

-  apiextensions.k8s.io

   -  CustomResourceDefinition = Create a new API resource.

-  apps

   -  ControllerRevision = View the full history of a Deployment.
   -  PodTemplate = Create a base template that can be used to create Pods from.

-  autoscaling

   -  HorizontalPodAutoscaler = Define metrics to collect for automatic Pod scaling.

-  core

   -  Event = Create a custom event to track and log.
   -  LimitRange = Define default resource requirements for Pods.

-  policy

   -  PodDisruptionBudget = Define the minimum and maximum amount of Pods that should be running during special situations such as eviction.
   -  PodSecurityPolicy = Define Pod users and permissions.

-  scheduling.k8s.io

   -  PriorityClass = Define a custom priority to be used by Pods.

-  settings.k8s.io

   -  PodPreset = Define default settings that a Pod can use.

Service
~~~~~~~

Service APIs are used to manage networks for Pods. [5]

-  core

   -  Endpoints = View simple information about the running Kubernetes networking objects.
   -  Service = Manage internal access to a Pod.

-  discovery.k8s.io

   -  EndpointSlice = A more advanced implementation of Endpoints.

-  networking.k8s.io

   -  Ingress = Manage external access to a Pod based on an existing Service.
   -  IngressClass = Configure the Ingress controller back-end.

Ingress
^^^^^^^

-  API group / version (latest): networking.k8s.io/v1
-  Shortname: ing
-  Namespaced: true

----

``ing.spec:``

-  backend (map) = The default backend for when no rule is matched.

   -  resource (map) = Use this OR serviceName and servicePort.

      -  apiGroup (string) = The object API group.
      -  **kind** (string) = The object API kind.
      -  **name** (string) = The object name.

   -  serviceName (string) = The Service name to use.
   -  servicePort (string) = The Service port to use.

-  ingressClassName (string) = The Ingress Controller to use.
-  rules (list of maps) = Rules to define when and where to route public traffic to.

   -  host (string) = The domain name (not an IP address) to accept requests on. This domain should resolve an IP address on one of the Control Plane Nodes in the Kubernetes cluster.
   -  http (map)

      -  paths (list of maps)

         -  **backend** (map) = Backend details specific to this path.

            -  resource (map)

               -  apiGroup (string)
               -  **kind** (string)
               -  **name** (string)

            -  service (map)

               -  name (string) = The name of the Service object to use as a backend.
               -  port (map)

                  -  number (integer)

         -  path (string) = The HTTP path to use. Pathes must begin with ``/``.
         -  **pathType** (string) = How to find a match for the path. Default is ImplementationSpecific. This field is required if the Ingress Controller does not have a default.

            -  Exact = Match the exact path.
            -  Prefix = Split the path by the ``/`` character and find a matching path from that ordered list.
            -  ImplementationSpecific = The IngressClass can determine how to interpret the path.

-  tls (list of maps) = List of all of the SSL/TLS certificates.

   -  hosts (list of strings) = A list of hosts to bind the SSL/TLS certificate to.
   -  secretName (string) = The Secret object name that contains the SSL/TLS certificate.

----

**Examples:**

Ingress with domain name.

.. code-block:: yaml

   ---
   kind: Ingress
   apiVersion: networking.k8s.io/v1
   metadata:
     name: ing-domain
   spec:
     rules:
       - host: app.example.com
         http:
           paths:
             - path: /app
               pathType: Prefix
               backend:
                 service:
                   name: svc-foo
                   port:
                     number: 80

Ingress with an existing TLS certificate.

.. code-block:: yaml

   ---
   kind: Secret
   apiVersion: v1
   metadata:
     name: secret-tls
   type: kubernetes.io/tls
   data:
     tls.crt: <CERTIFICATE_BASE64_ENCODED>
     tls.key: <KEY_BASE64_ENCODED>
   ---
   kind: Ingress
   apiVersion: networking.k8s.io/v1
   metadata:
     name: ing-tls
   spec:
     rules:
       - host: login.example.com
         http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: svc-bar
                   port:
                     number: 80
     tls:
       - hosts:
           - login.example.com
         secretName: secret-tls

[5]

Ingress with the internal ``ing.spec.rules.http.paths.path`` being routed to the root path ``/``. In this example, a HTTP request to ``http://foo.bar.com/`` will load up the contents of ``http://foo.bar.com/some/path/here/``.

.. code-block:: yaml

   ---
   kind: Ingress
   apiVersion: networking.k8s.io/v1
   metadata:
     name: ing-rewrite-target-example
   annotations:
     # NGINX
     nginx.ingress.kubernetes.io/rewrite-target: /
     # Traefik
     #traefik.ingress.kubernetes.io/rewrite-target: /
   spec:
     # NGINX
     ingressClassName: nginx
     # Traefik
     #ingressClassName: traefik
     rules:
       - host: foo.bar.com
         http:
           paths:
             - path: /some/path/here
               pathType: Prefix
               backend:
                 service:
                   name: svc-rewrite-target-example
                   port:
                     number: 80

Service
^^^^^^^

-  API group / version (latest): v1
-  Shortname: svc
-  Namespaced: true

----

``svc.spec:``

-  clusterIP (string) = Define a static IP address to use for a ClusterIP, LoadBalancer, or Node type.
-  externalIPs (list of strings) = Static IP addresses of from an external unmanaged load balancer.
-  externalName (string) = The domain name to use for routing internal traffic.
-  externalTrafficPolicy (string)

   -  Cluster = Clustered sessions are slower but equally distributed.
   -  Local = Local sessions are faster and more reliable but may not be equally distributed.

-  healthCheckNodePort (integer) = The port to use for health checks. This only works when these two settings are in use: ``svc.spec.type: LoadBalancer`` and ``svc.spec.externalTrafficPolicy: Local``
-  ipFamily (string) = The IP version to use. ``IPv4`` or ``IPv6``.
-  loadBalancerIP (string) = If supported by the cloud-provider, specify an IP address for the load balancer.
-  loadBalancerSourceRanges (list of strings) = If supported by the cloud-provider, only allow incoming connects from these IP addresses.
-  ports (list of maps) = Ports to expose/open.
-  publishNotReadyAddresses (boolean) = Default is false. Publish IP address information to the internal Kubernetes DNS server before a Pod is in a ready state.
-  **selector** (`map of Selector <#selector>`_) = Bind this Service object to a Pod based on the provided labels.
-  sessionAffinity (map) = Default is None.

   -  ClientIP = Keep the same session for a client connecting to a Pod.
   -  None = Do not keep the same session. A client reconnecting may connect to a new Pod.

-  sessionAffinityConfig (map) = Additional settings for the sessionAffinity.

   -  clientIP (map)

      -  timeoutSeconds (integer) = Default is 3 hours. The sticky session timeout in seconds.

-  topologyKeys (list of strings) = A list of Endpoint labels to bind to. The first Endpoint found from the list will be used.
-  **type** (string) = Default is ClusterIP. The type of Service to create.

   -  ClusterIP = Create an internal IP address that load balances requests to a specific Pod.
   -  ExternalName = The same as ClusterIP except it relies on a domain name instead of an IP address.
   -  LoadBalancer = If the cloud provider has an external load balancer offering, this Service object will create a new load balancer.
   -  NodePort = Open a port on every Node and map it to a specific Pod.

----

**Examples:**

SVC with ClusterIP and a static IP address.

.. code-block:: yaml

   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: svc-clusterip
   spec:
     clusterIP: 10.0.0.222
     ports:
       - port: 80
         protocol: TCP
         targetPort: 80
     selector:
       <POD_LABEL_KEY>: <POD_LABEL_VALUE>

SVC with ExternalName.

.. code-block:: yaml

   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: svc-externalname
   spec:
     type: ExternalName
     externalName: foo.bar.com
     ports:
       - port: 50000
         protocol: TCP
         targetPort: 50000
     selector:
       <POD_LABEL_KEY>: <POD_LABEL_VALUE>

SVC with LoadBalancer.

.. code-block:: yaml

   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: svc-loadbalancer
   spec:
     type: LoadBalancer
     externalTrafficPolicy: Local
     loadBalancerSourceRanges:
       - 172.80.0.0/16
       - 130.100.20.0/24
     ports:
       - port: 80
         protocol: TCP
         targetPort: 8080
     selector:
       <POD_LABEL_KEY>: <POD_LABEL_VALUE>

SVC with NodePort.

.. code-block:: yaml

   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: svc-nodeport
   spec:
     type: NodePort
     ports:
       - port: 3000
         protocol: TCP
         targetPort: 3000
     selector:
       <POD_LABEL_KEY>: <POD_LABEL_VALUE>

[5]

Workloads
~~~~~~~~~

Workload APIs manage running applications. [5]

-  apps

   -  DaemonSet = Manages Kubernetes Pods that run on worker nodes. Objects created using this API are usually for logging or networking.
   -  Deployment = Uses both the Pod and ReplicaSet API along with managing the life-cycle of an application. It is designed for stateless applications.
   -  ReplicaSet = New API for manging replicas that has support for label selectors.
   -  StatefulSet = Similar to a Deployment except it can handle persistent storage along with ordered scaling and rolling updates. Each new Pod created will have a new persistent volume claim created (if applicable). [1]

-  batch

   -  CronJob = Schedule Pods to run at specific intervals of time.
   -  Job = A one-time execution of a Pod.

-  core

   -  Pod = The smallest API resource that can be used to create containers.
   -  ReplicationController = Older API for managing replicas. [11]

Most applications should use the Deployment or the StatefulSet API due to the collection of features it provides.

CronJob
^^^^^^^

-  API group / version (latest): batch/v1beta1
-  Shortname: cj
-  Namespaced: true

----

``cj.spec:``

-  concurrencyPolicy (string) = What action to take if a CronJob object is running again overlapping with itself.

   -  Allow = Default. Allow the CronJob to start even if another CronJob is running.
   -  Forbid = Skip this scheduled CronJob if the last one has not completed yet.
   -  Replace = Stop the last CronJob and then start a new one.

-  failedJobsHistoryLimit (integer) = Default is 1. The number of failed Jobs to keep logged.
-  **jobTemplate** (`map of Job <#job>`_) = The Job definition to run.
-  **schedule** (string) = The `cron <https://crontab.guru/>`__ schedule/interval.
-  startingDeadlineSeconds (integer) = The amount of time to wait before marking the Job as failed if a CronJob misses it's scheduled time.
-  successfulJobHistoryLimit (integer) = Default is 3. The number of successful Jobs to keep logged.
-  suspend (boolean) = Default is false. Only run the CronJob once. Do not run it again.

----

**Examples:**

CronJob example.

.. code-block:: yaml

   ---
   kind: CronJob
   apiVersion: batch/v1beta11
   metadata:
     name: cj-calculate
   spec:
     concurrencyPolicy: Forbid
     failedJobsHistoryLimit: 10
     jobTemplate:
       spec:
         backoffLimit: 10
         completions: 2
         parallelism: 4
         template:
           spec:
             containers:
               - name: calculus-equation
                 image: clculus-equation:1.0.0
                 args:
                   - scenario17
                   - --verbose
             restartPolicy: OnFailure
         ttlSecondsAfterFinished: 3600
     schedule: "0 * * * *"

[5]

Deployment
^^^^^^^^^^

-  API group / version (latest): apps/v1
-  Shortname: deploy
-  Namespaced: true

----

``deploy.spec:``

-  minReadySeconds (integer) = Default is 0 seconds. The amount of seconds to wait for a Pod to put into the "ready" state.
-  paused (boolean) = If the deployment is paused.
-  progressDeadlineSeconds (integer) = The amount of seconds before a non-ready Deployment is considered to be in the "failed" state.
-  replicas (integer) = Default is 1. The number of Pods to create.
-  revisionHistoryLimit (integer) = Default is 10. The amount of ReplicaSets from a previous Deployment to keep for the purpose of a rollback.
-  **selector** (`map of Selector <#selector>`_) = The ReplicaSet will match Pods with these labels.
-  strategy (map) = The Deployment strategy when updating and rolling back a Deployment.
-  **template** (`map of a Pod manifest <#pod>`_) = The Pod definition to manage as a Deployment.

   -  metadata (map) = Specify any non-``name`` value here.
   -  spec (map)

----

**Examples:**

Deployment example.

.. code-block:: yaml

   ---
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     name: deploy-website
   spec:
     replicas: 5
     selector:
       matchLabels:
         foo: bar
     template:
       metadata:
         labels:
           foo: bar
       spec:
         containers:
           - name: nginx
             image: nginx:1.7.0
             ports:
               - containerPort: 80
           - name: php-fpm
             image: php-fpm:7.0
             ports:
               - containerPort: 8080

[5]

Job
^^^

-  API group / version (latest): batch/v1
-  Shortname: (None)
-  Namespaced: true

----

``job.spec:``

-  activeDeadlineSeconds (integer) = The amount of time, in seconds, to wait for a Job to be finished before terminating the Pods.
-  backoffLimit (integer) = Default is 6. The amount of retries before marking a Job as failed.
-  completions (integer) = How many times the Job should complete before being marked as a success.
-  manualSelector (boolean) = Set to true to manually manage the ``job.spec.selector``.
-  parallelism (integer) = The number of Pods that can run at the same time.
-  selector (`map of Selector <#selector>`_) = By default, this is managed automatically. The number of Pods managed by the Job should match the labels provided.
-  **template** (`map of a Pod manifest <#pod>`_) = The Pod definition to manage as a Job. In that definition the default restartPolicy of "Always" is not allowed. Use "OnFailure" or "Never" instead.
-  ttlSecondsAfterFinished (integer) = The time to wait before deleting Pods from a Job.

----

**Examples:**

Job example.

.. code-block:: yaml

   ---
   kind: Job
   apiVersion: batch/v1
   metadata:
     name: job-calculate
   spec:
     backoffLimit: 10
     completions: 2
     parallelism: 4
     template:
       spec:
         containers:
           - name: calculus-equation
             image: clculus-equation:1.0.0
             args:
               - scenario17
               - --verbose
         restartPolicy: OnFailure
     ttlSecondsAfterFinished: 3600

[5]

Pod
^^^

-  API group / version (latest): v1
-  Shortname: po
-  Namespaced: true

----

``po.spec:``

-  activeDeadlineSeconds (integer) = The startTime, in seconds, to wait before marking a Pod as failed.
-  affinity (map) = Define scheduling constraints.

   -  nodeAffinity (map) = Specify NodeAffinity spec values here.

      -  requiredDuringSchedulingIgnoredDuringExecution (map)
      -  requiredDuringSchedulingRequiredDuringExecution (map)
      -  preferredDuringSchedulingIgnoredDuringExecution (map)

-  automountServiceAccountToken (boolean) = If the service account token should be available via a mount. The default is true.
-  **containers** (list of `Containers map <#containers>`_) = The list of containers the Pod should create and manage.
-  dnsConfig (map) = DNS settings to add to the /etc/resolv.conf file.

   -  nameservers (list) = List of nameservers.
   -  options (list of maps) = List of options.

      -  name (string)
      -  value (string) = Optional. A value to bind to the option name.

   -  searches (list) = List of searches.

-  dnsPolicy (string) = DNS resolution settings managed by Kubernetes.

   -  ClusterFirst = Default. Quries for domain names that do not include the Kubernetes cluster hostname will use the resolvers from the worker Node.
   -  ClusterFirstWithHostNet = ``Pod.spec.dnsPolicy.ClusterFirst`` for Pods using the ``Pod.spec.hostNetwork`` option.
   -  Default = Use the worker Node's DNS resolution settings.
   -  None = Only provide DNS settings via ``Pod.spec.dnsConfig``.

-  enableServiceLinks (boolean) = Provide Service information via environment variables.
-  ephemeralContainers (list of `Containers map <#containers>`_) = Temporary containers for debugging.
-  hostAliases (map) = Additional /etc/hosts entries.

   -  hostnames (string)
   -  ip (string)

-  hostIPC (boolean) = Default is false. Use the IPC namespace.
-  hostPID (boolean) = Default is false. Use the PID namespace.
-  hostname (string) = Default is "<HOSTNAME>.<SUBDOMAIN>.<POD_NAMESPACE.svc.<CLUSTER_DOMAIN>". The cluster domain default is "cluster.local".  A custom hostname for the Pod.
-  hostNetwork (boolean) = Default is false. Expose the ``po.spec.containers.ports.containerPort`` directly on the Node it is running on. Unlike a Service, this will create a 1:1 mapping of the port used by the containers to the exact same port number on the Node.
-  imagePullSecrets (list of maps)

   -  name (string) = The name of the Secret to use.

-  initContainers (list of `Containers map <#containers>`_) = A list of containers to create in order. If any of them fail then the entire Pod is marked as failed.
-  nodeName (string) = The name of the work Node to schedule the Pod on.
-  nodeSelector (map) = Key-value pairs on a worker Node that must be matched.
-  overhead (`map of System Resources <#system-resources>`_) = The amount of resource overhead by having Kubernetes run the Pod. This is added ontop of amounts defined by ``Pod.spec.containers.resources.limits`` and ``Pod.spec.containers.resources.requests``.
-  preemptionPolicy (string) Defaults to PreemptLowerPriority. Specify a Policy for low priority Pods.
-  priority (integer) = Specify a high or low priority value for the Pod.
-  priorityClassName (string) = Specify a PriorityClass object name to use for priority settings.
-  readinessGates (list of strings) = The readiness gates that need to pass for a Pod to be marked as ready.

   -  conditionType (string) = A valid value from the Pod's condition list.

-  restartPolicy (string) = The policy for when containers stop in a Pod.

   -  Always = Default.
   -  Never
   -  OnFailure

-  runtimeClassName (string) = The container RuntimeClass settings to use.
-  schedulerName (string) = Use a different scheduler besides the default kube-scheduler.
-  securityContext (map) = Permissions to set for all containers in the Pod.

   -  fsGroup (integer) = A group to use volume mounts.
   -  fsGroupChangePolicy (string) = The policy for changing the group permission.

      -  Always (default)
      -  OnRootMismatch

   -  runAsGroup (integer)
   -  runAsNonRoot (boolean)
   -  runAsUser (integer)
   -  seLinuxOptions (map)
   -  supplementalGroups (list of integers) = Additional GID to assign to the process.
   -  sysctls (list of maps) = sysctl parameters to set.

      -  name (string)
      -  value (string)

   -  windowsOptions (map)

-  serviceAccountName (string) = Run the Pod under a different ServiceAccount.
-  shareProcessNamespace (boolean) = Default is false. Use the same namespace for all containers in the Pod.
-  subdomain (string) = The subdomain to use in the full hostname of the Pod.
-  terminationGracePeriodSeconds (integer) = Default is 30. The amount of seconds before forcefully stopping a all containers in the Pod.
-  tolerations (list of maps) = Specify tolerations to Node taints.

   -  key (string) = Taint key.
   -  value (string) = Taint value.
   -  operator (string) = Default is Equal. Alternatively use Exists.
   -  effect (string) = NoExecute, NoSchedule, or PreferNoSchedule.
   -  tolerationSeconds (integer) = The amount of seconds to tolerate a taint.

-  topologySpreadConstraints (map) = Define how to spread Pods across the Kubernetes cluster.

   -  labelSelector (map) = A key-value pair to find similar Pods. Schedule the Pod to run on that worker Node.
   -  maxSkew (integer) = The number of Pods that can be unevenly distributed.
   -  topologyKey (string) = A key label on a worker Node to look for.
   -  whenUnsatisfiable (string) = Default is DoNotSchedule. Alternatively use ScheduleAnyway.

-  volumes (list of maps) = Volumes to expose to all of the containers.

   -  name (string) = The name of the PVC
   -  <PV_STORAGE_PLUGIN_TYPE> (map) = Settings for the PVC.

[5]

----

**Examples:**

Pod with two containers.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: two-apps
   spec:
     containers:
       - name: nginx
         image: nginx
       - name: php
         image: php-fpm

Pod thate overrides the ENTRYPOINT for a container.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: phun
   spec:
     containers:
       - name: php
         image: php-fpm
         args:
           - php-fpm
           - --nodaemonize

Pod with persistent storage (without a PVC).

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: db-cb
   spec:
     containers:
       - name: couchbase
         image: couchbase-server:community-6.0.0
         volumeMounts:
           - name: local-volume
             mountPath: /opt/couchbase/var
       volumes:
         - name: local-volume
           hostPath:
             path: /var/lib/couchbase

Pod with persistent storage (with a PVC).

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: db-mysql
   spec:
     containers:
       - name: mariadb
         image: mariadb:10.5
         volumeMounts:
           - mountPath: /var/lib/mysql
             name: mariadb-volume
     volumes:
       - name: mariadb-volume
         persistentVolumeClaim:
           claimName: <PVC_NAME>

Pod with environment variables from different sources.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: all-the-sources
   spec:
     containers:
       - name: nginx
         image: nginx:1.9.0
         env:
           - name: foo
             value: bar
           - name: <KEY>
             valueFrom:
               configMapKeyRef:
                 name: <CONFIGMAP_NAME>
                 key: <CONFIGMAP_KEY>
         envFrom:
           - configMapRef:
               name: <CONFIGMAP_NAME>
           - secretRef:
               name: <SECRET_NAME>

Pod with Secret key-values provided as files on an ephemeral volume.

.. code-block:: sh

   $ kubectl create secret generic --from-literal=foo=bar 007

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: webapp
   spec:
     containers:
       - name: nginx
         image: nginx
         volumeMounts:
           - name: secret-volume
             mountPath: /opt/nginx-config
             readOnly: true
     volumes:
       - name: secret-volume
         secret:
           secretName: "007"

.. code-block:: sh

   $ kubectl exec webapp -- ls -1 /opt/nginx-config/
   foo
   $ kubectl exec webapp -- cat /opt/nginx-config/foo
   bar

Pod with common security settings.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: http-secure
   spec:
     containers:
       - name: nginx
         image: nginx:1.9.0
         securityContext:
           runAsUser: 1000
           capabilities:
             add: ["NET_ADMIN", "SYS_TIME"]
           privileged: false

Pod with quotas set (without a ResourceQuota).

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: miniapp
   spec:
     containers:
       - name: nginx
         image: nginx:1.9.0
      resources:
        requests:
          cpu: 1
          memory: "256Mi"
        limits:
          cpu: 2
          memory: "512Mi"

Pod running on a specific Node based on the Node's hostname.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: simple-app
   spec:
     containers:
       - name: nginx
         image: nginx:1.9.0
     nodeSelector:
       kubernetes.io/hostname: worker04

Pod with ports exposed on the Node it is running on.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: dns-app
   spec:
     hostNetwork: True
     containers:
       - name: coredns
         image: coredns
         ports:
           - containerPort: 53
             protocol: TCP
             name: coredns-tcp
           - containerPort: 53
             protocol: UDP
             name: coredns-udp

Third-Party
~~~~~~~~~~~

Cluster
^^^^^^^

-  API group / version (latest): kind.x-k8s.io/v1alpha4
-  Shortname: (None)
-  Namespaced: false

``Cluster`` is an API desgined by the ``kind`` special interest group. It is designed to help configure development Kubernetes clusters.

----

``Cluster:``

-  featureGates (map)

   -  ``<FEATURE>`` (boolean) = Enable or disable experimental Kubernetes features. The full list of features gates is provided `here <https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/>`__.

-  runtimeConfig (map)

   -  ``<API_GROUP>/<API_VERSION>`` (boolean) = Enable or disable API groups. Validation options can be found `here <https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/>`__ and includes: ``api/[all|ga|beta|alpha]: [true|false]``.

-  networking (map)

   -  apiServerAddress (string) = ``127.0.0.1`` by default. The IP address to listen to for internal Kubernetes Nodes to communicate with each other.
   -  apiServerPort (string) = ``6443`` by default. The port to listen on for internal Kubernetes Nodes to communicate with each other.
   -  disableDefaultCNI (boolean) = By default, the custom "kindnetd" CNI is installed. Disable this to allow installing a different CNI plugin after the new cluster is created.
   -  ipFamily (string) = ``ipv4`` (default) or ``ipv6``. Dual-stack IP addressing is not supported in the Cluset API yet.
   -  kubeProxyMode (string) = ``iptables`` (default) or ``ipvs``.
   -  podSubnet (string) = ``10.244.0.0/16`` by default. The IP range to use for Pod networking (internal access).
   -  serviceSubnet (string) = ``10.96.0.0/12`` by default. The public IP range to use for Service networking (external access).

-  nodes (list of maps)

   -  role (string) = The Nodes that should be deployed. Use ``control-plane`` and ``worker``. List the same type of Node more than once to deploy more Nodes.
   -  extraMounts (list of maps)

      -  containerPath (string) = The mount point for ``Cluster.nodes.role.extraMounts.hostPath``.
      -  hostPath (string) = A directory on the host to share with the container.

   -  extraPortMappings (list of maps)

      -  containerPort (integer) = The port inside the containers to expose.
      -  hostPort (integer) = The port on the host to use to connect to the ``Cluster.nodes.extraPortMappings.containerPort``.
      -  listenAddress (string) = Default is ``0.0.0.0``.
      -  protocol (string) = ``SCTP``, ``TCP`` (default), or ``UDP``.

   -  kubeadmConfigPatches (list of strings) = Provide additional `configurations for kubeadm <https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file>`__.

----

**Examples:**

x3 Control Plane Nodes and x2 Worker Nodes.

.. code-block:: yaml

   ---
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
     - role: control-plane
     - role: control-plane
     - role: control-plane
     - role: worker
     - role: worker

[30]

OpenShift
~~~~~~~~~

These APIs are only available on OpenShift. [28]

-  Alertmanager monitoring.coreos.com/v1
-  APIServer config.openshift.io/v1
-  AppliedClusterResourceQuota quota.openshift.io/v1
-  Authentication config.openshift.io/v1
-  Authentication operator.openshift.io/v1
-  BrokerTemplateInstance template.openshift.io/v1
-  Build build.openshift.io/v1
-  Build config.openshift.io/v1
-  BuildConfig build.openshift.io/v1
-  CatalogSource operators.coreos.com/v1alpha1
-  ClusterAutoscaler autoscaling.openshift.io/v1
-  ClusterOperator config.openshift.io/v1
-  ClusterResourceQuota quota.openshift.io/v1
-  ClusterRole authorization.openshift.io/v1
-  ClusterRoleBinding authorization.openshift.io/v1
-  ClusterServiceVersion operators.coreos.com/v1alpha1
-  ClusterVersion config.openshift.io/v1
-  Config imageregistry.operator.openshift.io/v1
-  Config operator.openshift.io/v1
-  Config samples.operator.openshift.io/v1
-  Console config.openshift.io/v1
-  Console operator.openshift.io/v1
-  ConsoleCLIDownload console.openshift.io/v1
-  ConsoleExternalLogLink console.openshift.io/v1
-  ConsoleLink console.openshift.io/v1
-  ConsoleNotification console.openshift.io/v1
-  ConsoleYAMLSample console.openshift.io/v1
-  ContainerRuntimeConfig machineconfiguration.openshift.io/v1
-  ControllerConfig machineconfiguration.openshift.io/v1
-  CredentialsRequest cloudcredential.openshift.io/v1
-  CSISnapshotController operator.openshift.io/v1
-  DeploymentConfig apps.openshift.io/v1
-  DNS config.openshift.io/v1
-  DNS operator.openshift.io/v1
-  DNSRecord ingress.operator.openshift.io/v1
-  EgressNetworkPolicy network.openshift.io/v1
-  Etcd operator.openshift.io/v1
-  FeatureGate config.openshift.io/v1
-  Group user.openshift.io/v1
-  HostSubnet network.openshift.io/v1
-  Identity user.openshift.io/v1
-  Image config.openshift.io/v1
-  Image image.openshift.io/v1
-  ImageContentSourcePolicy operator.openshift.io/v1alpha1
-  ImagePruner imageregistry.operator.openshift.io/v1
-  ImageSignature image.openshift.io/v1
-  ImageStream image.openshift.io/v1
-  ImageStreamImage image.openshift.io/v1
-  ImageStreamImport image.openshift.io/v1
-  ImageStreamMapping image.openshift.io/v1
-  ImageStreamTag image.openshift.io/v1
-  ImageTag image.openshift.io/v1
-  Infrastructure config.openshift.io/v1
-  Ingress config.openshift.io/v1
-  IngressController operator.openshift.io/v1
-  InstallPlan operators.coreos.com/v1alpha1
-  KubeAPIServer operator.openshift.io/v1
-  KubeControllerManager operator.openshift.io/v1
-  KubeletConfig machineconfiguration.openshift.io/v1
-  KubeScheduler operator.openshift.io/v1
-  KubeStorageVersionMigrator operator.openshift.io/v1
-  LocalResourceAccessReview authorization.openshift.io/v1
-  LocalSubjectAccessReview authorization.openshift.io/v1
-  Machine machine.openshift.io/v1beta1
-  MachineAutoscaler autoscaling.openshift.io/v1beta1
-  MachineConfig machineconfiguration.openshift.io/v1
-  MachineConfigPool machineconfiguration.openshift.io/v1
-  MachineHealthCheck machine.openshift.io/v1beta1
-  MachineSet machine.openshift.io/v1beta1
-  NetNamespace network.openshift.io/v1
-  Network config.openshift.io/v1
-  Network operator.openshift.io/v1
-  OAuth config.openshift.io/v1
-  OAuthAccessToken oauth.openshift.io/v1
-  OAuthAuthorizeToken oauth.openshift.io/v1
-  OAuthClient oauth.openshift.io/v1
-  OAuthClientAuthorization oauth.openshift.io/v1
-  OpenShiftAPIServer operator.openshift.io/v1
-  OpenShiftControllerManager operator.openshift.io/v1
-  OperatorGroup operators.coreos.com/v1
-  OperatorHub config.openshift.io/v1
-  OperatorSource operators.coreos.com/v1
-  PackageManifest packages.operators.coreos.com/v1
-  PodMonitor monitoring.coreos.com/v1
-  PodSecurityPolicyReview security.openshift.io/v1
-  PodSecurityPolicySelfSubjectReview security.openshift.io/v1
-  PodSecurityPolicySubjectReview security.openshift.io/v1
-  Profile tuned.openshift.io/v1
-  Project config.openshift.io/v1
-  Project project.openshift.io/v1
-  ProjectRequest project.openshift.io/v1
-  Prometheus monitoring.coreos.com/v1
-  PrometheusRule monitoring.coreos.com/v1
-  Proxy config.openshift.io/v1
-  RangeAllocation security.openshift.io/v1
-  ResourceAccessReview authorization.openshift.io/v1
-  Role authorization.openshift.io/v1
-  RoleBinding authorization.openshift.io/v1
-  RoleBindingRestriction authorization.openshift.io/v1
-  Route route.openshift.io/v1
-  Scheduler config.openshift.io/v1
-  SecurityContextConstraints security.openshift.io/v1
-  SelfSubjectRulesReview authorization.openshift.io/v1
-  ServiceCA operator.openshift.io/v1
-  ServiceMonitor monitoring.coreos.com/v1
-  SubjectAccessReview authorization.openshift.io/v1
-  SubjectRulesReview authorization.openshift.io/v1
-  Subscription operators.coreos.com/v1alpha1
-  Template template.openshift.io/v1
-  TemplateInstance template.openshift.io/v1
-  ThanosRuler monitoring.coreos.com/v1
-  Tuned tuned.openshift.io/v1
-  User user.openshift.io/v1
-  UserIdentityMapping user.openshift.io/v1

Template
^^^^^^^^

-  API group / version (latest): v1
-  Shortname: (None)
-  Namespaced: true

A Template provides a way to create more than one object using a single manifest. It also supports being passed parameters to customize the Template. This API is similar in scope to Helm in the sense that it is a package manager for OpenShift.

----

``template:``

-  metadata

   -  annotations (map of strings)

      -  openshift.io/display-name (string) = The human friendly name of the Template to display.
      -  description (string)  = A short description of the Template.
      -  openshift.io/long-description = A long description of the Template.
      -  tags (string) = A comma-separated list of descriptive tags for what the Template provides.
      -  iconClass (string) = The name of the icon to use for the Template.
      -  openshift.io/provider-display-name (string) = The name of the developer or company that created the Template.
      -  openshift.io/documentation-url (string) = The documentation URL.
      -  openshift.io/support-url (string) = The support URL.
      -  message (string) = The message to display after the Template has been created.

-  labels (map of strings) = Key-value pair labels to apply to all objects created from this Template.
-  objects (list of maps) = A list of manifests to create. Variables can be set in here.
-  parameters (list of maps) = A list of variables that can be set by end-users and replaced in the ``template.objects`` section.

[29]

(Common Reoccuring Fields)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Containers
^^^^^^^^^^

``Pod.spec.{containers,ephemeralContainers,initContainers}`` (list of maps)

-  args (list of strings) = CMD.
-  command (list of strings) = ENTRYPOINT.
-  env (list of maps) = Environment variables to load in the container.
-  envFrom (list of maps) = Environment variables (from another object) to load in the container.

   -  configMapRef (map)

      -  name (string) = Name of the ConfigMap object to load.

   -  prefix (string) = A prefix to append to each key from the ConfigMap.

-  **image** (string)
-  imagePullPolicy (string)

   -  Always = Default for "latest" tag.
   -  IfNotPresent = Default for all other tags.
   -  Never

-  lifecycle (map)

   -  postStart (map) = Action to take after a container starts.

      -  exec (map)

         -  command (list of strings) = A command to run.

      -  httpGet (map) = A HTTP URL to GET.

         -  httpHeaders (map)
         -  path (string)
         -  port (string)
         -  scheme (string) = Defaults to HTTP. Optionally set to HTTPS.

      -  tcpSocket (map) = A TCP socket to connect to.

         -  port (string)

   -  preStop (map) = Action to take before a container stops.

      -  exec (map)
      -  httpGet (map)
      -  tcpSocket (map)

-  livenessProbe (`map of Probe <#probe>`_) = Probe to see if the application in the container is running properly.
-  **name** (string) = Name of the container.
-  ports (map) = Manage ports for the container.

   -  containerPort (integer) = The port in the container to open.
   -  hostIP (string) = The IP address to bind the ``Pod.spec.containers.hostPort`` to.
   -  hostPort (integer) = The port on the Work Node to open.
   -  name (string) = Optionally provide a name. This can be used by a Service object.
   -  protocol (string) = Default is TCP. Set to TCP, UDP, or SCTP.

-  readinessProbe (`map of Probe <#probe>`_) = Probe to see if the application is ready to be exposed by a network Service..
-  resources (map)

   -  limits (`map of System Resources <#system-resources>`_) = Hard resource limits.
   -  requests (`map of System Resources <#system-resources>`_) = Estimated resource usage. Used by kube-scheduler to help find a suitable worker Node.

-  securityContext (map)

   -  allowPrivilegeEscalation (boolean) = If a user can access higher privileges than it currently has.
   -  capabilities (map) = The capabilities the container has access to.

      -  add (string)
      -  remove (string)

   -  privileged (boolean) = Default is false. If the container should run with root privileges.
   -  procMount (string) = The proc mount type.
   -  readOnlyRootFilesystem (boolean) = Default is false. If the container should be read-only.
   -  runAsGroup (integer) = GID.
   -  runAsNonRoot (boolean) = If the container should not run as the root user.
   -  runAsUser (integer) = UID.
   -  seLinuxOptions (map) = SELinux contexts to set for the container.

      -  level (string)
      -  role (string)
      -  type (string)
      -  user (string)

   -  windowsOptions (map) = Windows specific settings.

-  startupProbe (`map of Probe <#probe>`_) = Probe to see if the application in the container has fully started.
-  stdin (boolean) = Default is false. If stdin should be allowed.
-  stdinOnce (boolean) = Default is false. If stdin should be sent to the container once.
-  terminationMessagePath (string) = File path to write the termination message to.
-  terminationMessagePolicy (string) = Default is File. Alternatively use FallbackToLogsOnError.
-  tty (boolean) = Default is false. Requires ``Pod.spec.containers.stdin`` to be true. If a TTY should be created for the container.
-  volumeDevices (map) = Mount a PersistentVolumeClaim.

   -  devicePath (string) = The path in the container to mount to.
   -  name (string) = The name of the Pod's PVC to mount.

-  volumeMounts (map) = Mount a volume.

   -  mountPath (string) = The path in the container to mount to.
   -  mountPropagation (string) = Default is MountPropagationNone. How the moutns are propagated to or from the host and container.
   -  name (string)
   -  readOnly (boolean) = If the volume should be read-only.
   -  subPath (string) = Defaults to the root directory (""). The path in the volume to mount.
   -  subPathExpr (string) = The same as ``Pod.spec.volumeMounts.subPath`` except environment variables can be used.

-  workingDir (string) = The working directory for the ``Pod.spec.containers.command`` (ENTRYPOINT) or ``Pod.spec.containers.args`` (CMD).

[5]

Probe
^^^^^

``Pod.spec.containers.{liveness,readiness,startup}Probe`` (map)

-  exec (map) = Execute a command.

   -  command (list of strings) = The command and arguments to execute.

-  failureThreshold (integer) = Default is 3. Minimimum number of probe failures allowed.
-  httpGet (map)
-  initialDelaySeconds (integer) = Seconds to delay before starting a probe.
-  periodSeconds (integer) = Default is 10. The interval, in seconds, to run a probe.
-  successThreshold (integer) = Default is 1. The amount of times a probe needs to succeed before marking the a previously failed probe check as now passing.
-  tcpSocket (map)
-  timeoutSeconds (integer) = Default is 1. The amount of seconds before the probe times out.

[5]

Selector
^^^^^^^^

``deploy.spec.selector``, ``netpol.spec.podSelector``, ``netpol.spec.{egress,ingress}.{to,from}.{namespaceSelector,podSelector}``, ``pvc.spec.selector``, ``svc.spec.selector`` (map)

-  matchExpressions (list of maps) = Do a logical lookup for labels.

   -  **key** (string) = The label key.
   -  **operator** = DoesNotExist, Exists, In, or NotIn. The operator will analyze the key-value pair.
   -  values (list of strings) = A list of possible values.

-  matchLabels (map) = Specify any exact key-value label pair to match.

System Resources
^^^^^^^^^^^^^^^^

``Pod.spec.containers.resources.{limit,requests}``, ``Pod.spec.overhead`` (map)

-  cpu (string) = Specify the CPU load number.
-  memory (string) = Specify "Mi" or "Gi" of RAM.

[5]

Concepts
--------

Popular APIs
~~~~~~~~~~~~

These are common Kubernetes APIs used by developers [12]:

-  ConfigMap
-  CronJob
-  DaemonSet
-  Deployment
-  HorizontalPodAutoscaler
-  Ingress
-  Job
-  PersistentVolumeClaim
-  Pod
-  ReplicaSet
-  Secret
-  Service
-  StatefulSet
-  VerticalPodAutoscaler

Labels and Annotations
~~~~~~~~~~~~~~~~~~~~~~

Labels and annotations both provide a way to assign a key-value pair to an object. This can later be looked up by other objects and by administrators. Labels help to organize related objects and perform actions on them. Many APIs support using a selector to lookup and bind to objects with labels that are found. Helm has a variety of labels that it recommends. [27] Annotations are similar except they are meant for non-human processing.

Define labels and annotations in the metadata section of a manifest.

.. code-block:: yaml

   ---
   metadata:
     annotations:
       <KEY>: <VALUE>
     labels:
       <KEY>: <VALUE>

View all labels in use.

.. code-block:: sh

   $ kubectl get all --show-labels

View all objects with a specific label.

.. code-block:: sh

   $ kubectl get all -l "<KEY>=<VALUE>"

Namespaces
~~~~~~~~~~

Namespaces help to isolate objects. Common use cases include having one application per Namespace or one team per Namespace.

View what APIs do and do not support being created inside a Namespace. Any resource that does not support a Namespace is globally accessible [26], such as a PersistentVolume.

.. code-block:: sh

   $ kubectl api-resource --namespace=true
   $ kubectl api-resource --namespace=false

An object can declaratively bind itself to a Namespace by specifying it in the metadata.

.. code-block:: yaml

   ---
   metadata:
     namespace: <NAMESPACE_NAME>

Persistent Storage
~~~~~~~~~~~~~~~~~~

By default, all storage is emphemeral. The PersistentVolume (PV) and PersistentVolumeClaim (PVC) APIs provide a way to persistently store information for use-cases such as databases. A PV defines the available storage and connection details for the Kubernetes cluster to use. A PVC defines the storage allocation for use by a Pod.

The example below shows how to configure static storage for a Pod using a directory on a Worker Node.

-  Create a PV. Set a unique ``<PV_NAME>``, use any name for storageClassName, configure the ``<PV_STORAGE_MAX>`` gigabytes that the PV can allocate, and define the ``<LOCAL_FILE_SYSTEM_PATH>`` where the data from Pods should be stored on the Worker Nodes. In this scenario, it is also recommended to configure a ``nodeAffinity`` that restricts the PV from only being used by the Worker Node that has the local storage.

.. code-block:: yaml

   ---
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: <PV_NAME>
   spec:
     storageClassName: <STORAGE_CLASS_NAME>
     capacity:
       storage: <PV_STORAGE_MAX>Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "<LOCAL_FILE_SYSTEM_PATH>"
     nodeAffinity:
       required:
         nodeSelectorTerms:
           - matchExpressions:
             - key: kubernetes.io/hostname
               operator: In
               values:
                 - <WORKER_NODE_WITH_LOCAL_FILE_SYSTEM_PATH>

-  Create a PVC from the PV pool. Set a unique ``<PVC_NAME>`` and the ``<PVC_STORAGE>`` size. The size should not exceed the maximum available storage from the PV. To bind to the previously created PV, use the same ``<STORAGE_CLASS_NAME>``

.. code-block:: yaml

   ---
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: <PVC_NAME>
   spec:
     storageClassName: <STORAGE_CLASS_NAME>
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: <PVC_STORAGE>Gi

-  Create a Pod using the PVC. Set ``<POD_VOLUME_NAME>`` to a nickname of the PVC volume that will be used by the actual Pod and indicate the ``mountPath`` for where it should be mounted inside of the container.

.. code-block:: yaml

   ---
   kind: Pod
   apiVersion: v1
   metadata:
     name: <POD_NAME>
   spec:
     volumes:
       - name: <POD_VOLUME_NAME>
         persistentVolumeClaim:
           claimName: <PVC_NAME>
     containers:
       - name: mysql
         image: mysql:8.0
         volumeMounts:
           - mountPath: "/var/lib/mysql"
             name: <POD_VOLUME_NAME>

[3]

Networking
~~~~~~~~~~~

Pod Internal (host* and Service)

1. Host

   1a. hostNetwork (Pod API) = Expose the exact Pod ports on the Node it is running on. The Kubernetes CNI plugin does not manage the networking. This is configured via a Pod manifest by setting ``po.spec.containers.hostNetwork`` to ``True``.
   1b. hostPort

2. Service API

   2a. ClusterIP
   2b. NodePort = A port between 30000 to 32767 is exposed on every Node (even if the Pod is not running on it) and mapped to a specific Pod.
   2c. LoadBalancer
   2d. ExternalName

3. Ingress API


Pod External (Ingress)

Ingress only works for exposing public HTTP traffic. A new Gateway API is in development to provide exposing any service (even non-HTTP ones) publicly. https://github.com/kubernetes-sigs/gateway-api

Service and Ingress (Public Networking)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are two APIs for managing networking in Kubernetes: Service (internal) and Ingress (external). A Service by itself is used to expose access to a Pod and ports in it for development and testing purposes. There are various different types of services. Most can be managed by ``kubectl expose``.

ServiceTypes [22]:

-  ClusterIP = Opens a port and exposes it on an internal IP that can only be accessed on Nodes (no external connectivity). Internally in Kubernetes, requests to ``<SERVICE>.default.svc.cluster.local`` will be redirected to this IP address. The port is only open on the Nodes which have the related Pod running.
-  NodePort = Opens a port on every Node (even if a Pod it is tied to is not on it). Connectivity can be made through the IP address of the Nodes that have the Pod running.
-  LoadBalancer = Use a third-party cloud provider's load balancing service.
-  ExternalName = Similar to a ClusterIP except a domain name can be given. ``kubectl expose --type=ExternalName`` currently `does not work <https://github.com/kubernetes/kubernetes/issues/87398>`__ because there is no argument for the external name.

Ingress is used to publicily expose a Pod and it's ports. It can redirect traffic based on domain names and HTTP paths. It also supports creating load balancers and handling SSL/TLS termination. It requires a Service to bind to. [23]

Ingress Controllers are different back-ends that handle the Ingress API. They use different technologies and generally have their own use-cases. The only ones that are officially supported are NGINX and Google's Compute Engine (GCE).

Top 5 Ingress Controllers and their top use-cases [24]:

-  Ambassador = API gateway.
-  HAProxy = Load balancing.
-  Istio Ingress Gateway = Fast performance.
-  NGINX = Automatic public cloud integration.
-  Traefik = Let's Encrypt SSL/TLS generation.

A Kubernetes cluster can have more than one Ingress Controller installed. In an object's manifest, the one to use can be specified. [25]

Kubernetes < 1.18 ``ingress.metadata.annotations.kubernetes.io/ingress.class``:

.. code-block:: yaml

   metadata:
     annotations:
       kubernetes.io/ingress.class: <INGRESS_CONTROLLER>

Kubernetes >= 1.18 ``ingress.spec.ingressClassName``:

.. code-block:: yaml

   metadata:
     annotations:
       # Some Ingress Controllers still require the legacy
       # annotation to process special rules.
       kubernetes.io/ingress.class: <INGRESS_CONTROLLER>
   spec:
     ingressClassName: <INGRESS_CONTROLLER>

Helm (Package Manager)
~~~~~~~~~~~~~~~~~~~~~~

Helm is a package manager for Kubernetes applications. Helm 2 and below required a Tiller server component to be installed on the Kubernetes cluster. This is no longer required as of Helm 3. Helm is now a standalone client-side-only command. [15]

Vocabulary:

-  Chart = A Helm package with all of the related resource manifests to run an application.
-  Repository = A collection of Charts that can be installed.
-  Release = A unique name given each time a Chart is installed. This is used to help track different installations and the history of a Helm Chart.

`Helm Hub <https://hub.helm.sh/>`__ is the official repository for Helm Charts. There are currently over one thousand Charts available. Third-party repositories are also supported. Helm can even install Charts from a directory (such as a local git repository). [16]

Each Chart contains a "values.yaml" for manifest settings that can be overridden. It is expected that it contains sane defaults and can be deployed without any modifications. The manifest files are `Go templates <https://golang.org/pkg/text/template/>`__ that get rendered out based on the values provided to Helm. `The Chart Template Developer's Guide <https://helm.sh/docs/chart_template_guide/>`__ explains in more detail how to fully customize templates. It is possible to override values that are not templated, or to add new ones, by using `Kustomize <https://kustomize.io/>`__. The biggest downside to using Kustomize is that Helm no longer has visibility into the release/life-cycle of a Chart. [17]

Best Practices
~~~~~~~~~~~~~~

Probes
^^^^^^

There are three types of health probes for a Pod. At a minimum, a liveness probe should be configured for every Pod.

-  liveness = Determine if the application is still working as expected. If this probe fails, the Pod is restarted.
-  readiness = Determine if the application is available and should accept connections. If this probe fails, network connections to the Pod will be temporarily paused.
-  startup = Determine if the application has fully started (once at startup). If this probe fails, the Pod is marked as failed.

For a full list of configuration options, see the `Probe <#probe>`_ section.

Installation
------------

kubectl (CLI)
~~~~~~~~~~~~~

The ``kubectl`` command is used to manage Kubernetes objects. The binary version can manage a Kubernetes cluster of the same version and the previous minor release. [13]

Installation:

.. code-block:: sh

   $ cd ~/.local/bin/
   $ export KUBE_VER="1.18.3"
   $ curl -LO https://storage.googleapis.com/kubernetes-release/release/v${KUBE_VER}/bin/linux/amd64/kubectl
   $ chmod +x ./kubectl
   $ kubectl version --client

::

   Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:52:00Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}

By default, the configuration file (provided by the Kubernetes cluster administrator) will be loaded from the file ``~/.kube/config``. This can be set to a different file. [14]

.. code-block:: sh

   $ export KUBECONFIG="<PATH_TO_KUBE_CONFIG>.yml"
   $ kubectl config view
   $ kubectl cluster-info
   $ kubectl version

helm (CLI)
~~~~~~~~~~

Find the latest version from `Helm's GitHub releases page <https://github.com/helm/helm/releases>`__. [18]

Installation:

.. code-block:: sh

   $ export HELM_VER="3.2.2"
   $ curl -LO https://get.helm.sh/helm-v${HELM_VER}-linux-amd64.tar.gz
   $ tar -x -f helm-v${HELM_VER}-linux-amd64.tar.gz
   $ cp linux-amd64/helm ~/.local/bin/

History
-------

-  `Latest <https://github.com/ekultails/rootpages/commits/master/src/virtualization/kubernetes_development.rst>`__
-  `< 2019.10.01 <https://github.com/ekultails/rootpages/commits/master/src/virtualization/kubernetes.rst>`__

Bibliography
------------

1. "Kubernetes Persistent Volumes with Deployment and StatefulSet." Alen Komljen. January 17, 2019. Accessed May 29, 2020. https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/
2. "Persistent Volumes." Kubernetes Concepts. January 16, 2019. Accessed January 29, 2019. https://kubernetes.io/docs/concepts/storage/persistent-volumes/
3. "Configure a Pod to Use a PersistentVolume for Storage." Kubernetes Tasks. December 20, 2019. Accessed June 3, 2020. https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
4. "So you want to change the API?" GitHub kubernetes/community. June 25, 2019. Accessed April 15, 2020. https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md
5. "[Kubernetes 1.18] API OVERVIEW." Kubernetes API Reference Docs. April 13, 2020. Accessed July 13, 2020. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/
6. "Kubernetes Resources and Controllers Overview." The Kubectl Book. Accessed April 29, 2020. https://kubectl.docs.kubernetes.io/pages/kubectl_book/resources_and_controllers.html
7. "Overview of kubectl." Kubernetes Reference. March 28, 2020. Accessed April 29, 2020. https://kubernetes.io/docs/reference/kubectl/overview/
8. "Using kubectl to jumpstart a YAML file — #HeptioProTip." heptio Blog. September 21, 2017. Accessed April 29, 2020. https://blog.heptio.com/using-kubectl-to-jumpstart-a-yaml-file-heptioprotip-6f5b8a63a3ea
9. "Declarative Management of Kubernetes Objects Using Configuration Files." Kubernetes Tasks. May 2, 2020. Accessed May 28, 2020. https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/
10. "Kubernetes Tips: Create Pods With Imperative Commands in 1.18." Better Programming - Medium. April 7, 2020. Accessed May 28, 2020. https://medium.com/better-programming/kubernetes-tips-create-pods-with-imperative-commands-in-1-18-62ea6e1ceb32
11. "ReplicationController." Kuberntes Concepts. March 28, 2020. May 29, 2020. https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
12. "What are the most useful Kubernetes Resources for developers?" www.Dev4Devs.com. October 20, 2019. Accessed June 8, 2020. https://dev4devs.com/2019/10/20/what-are-the-kubernetes-resources-which-are-most-useful-for-developers/
13. "Install and Set Up kubectl." Kubernetes Tasks. May 30, 2020. Accessed June 11, 2020.https://kubernetes.io/docs/tasks/tools/install-kubectl/
14. "Configure Access to Multiple Clusters." Kubernetes Tasks. May 30, 2020. Accessed June 11, 2020. https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
15. "Helm 3.0.0 has been released!" Helm Blog. November 13, 2019. Accessed June 16, 2020. https://helm.sh/blog/helm-3-released/
16. "Using Helm." Helm Docs. Accessed June 16, 2020. https://helm.sh/docs/intro/using_helm/
17. "Customizing Upstream Helm Charts with Kustomize." Testing Clouds at 128bpm. July 20, 2018. Accessed June 16, 2020. https://testingclouds.wordpress.com/2018/07/20/844/
18. "Installing Helm. Helm Docs. Accessed June 16, 2020. https://helm.sh/docs/intro/install/
19. "examples." GitHub kubernetes/examples. May 21, 2020. Accessed June 25, 2020.  https://github.com/kubernetes/examples
20. "Complete Example Using GlusterFS." OpenShift Container Platform 3.11 Documentation. June 21, 2020. Accessed June 25, 2020. https://docs.openshift.com/container-platform/3.11/install_config/storage_examples/gluster_example.html
21. "Volumes." Kubernetes Concepts. May 15, 2020. Accessed June 25, 2020. https://kubernetes.io/docs/concepts/storage/volumes/
22. "Service." Kubernetes Concepts. May 30, 2020. Accessed June 28, 2020. https://kubernetes.io/docs/concepts/services-networking/service/
23. "Ingress." Kubernetes Concepts. May 30, 2020. Accessed June 28, 2020. https://kubernetes.io/docs/concepts/services-networking/ingress/
24. "Comparison of Kubernetes Top Ingress Controllers." caylent. May 9, 2019. Accessed June 28, 2020. https://caylent.com/kubernetes-top-ingress-controllers
25. "Ingress Controllers." Kubernetes Concepts. May 30, 2020. Accessed June 28, 2020. https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
26. "Namespaces." Kubernetes Concepts. June 22, 2020. Accessed June 30, 2020. https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
27. "Labels and Annotations." Helm Docs. Accessed June 30, 2020. https://helm.sh/docs/chart_best_practices/labels/
28. "API List." OpenShift Container Platform 4.5 Documentation. Accessed August 12, 2020. https://docs.openshift.com/container-platform/4.5/rest_api/index.html
29. "Templates." OpenShift Container Platform 3.11 Documentation. Accessed August 14, 2020.  https://docs.openshift.com/container-platform/3.11/dev_guide/templates.html
30. "Configuration." kind. January 3, 2021. Accessed January 20, 2021. https://kind.sigs.k8s.io/docs/user/configuration/
