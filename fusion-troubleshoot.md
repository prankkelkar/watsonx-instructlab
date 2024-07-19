Cleaning up localhost node from the IBM Storage Fusion HCI System cluster
=========================================================================

Follow the instructions to clean up the localhost node from the IBM Storage Fusion HCI System cluster.

About this task
---------------

The localhost node must not be added to the OpenShift® cluster, as it creates an issue at a later stage. A couple of OpenShift objects are created when the localhost node is added to the IBM Storage Fusion HCI System cluster.

Procedure
---------

Follow the steps to clean up the localhost node so that the node addition can retry after you resolve the DHCP or DNS misconfiguration issues.

1.  Run the following command to edit the `ComputeProvisionWorker CR` of the compute node that indicated an error message This node might contain an invalid hostname (localhost).
    
        # oc edit cpw provisionworker-compute-1-ru5
    
    Edit the respective CR to update the location information to an empty string to ensure that the `ComputeProvisionWorker` controller is not involved when you clean different objects of the compute node.
    
        # oc edit cpw provisionworker-compute-1-ru5
        ...
        spec:
          location: ""
    
2.  Delete the respective machine object of the compute node.
    
    You need to identify the machine object first and then mark it with an annotation. Then, scale down the machineset object to delete the machine object.
    
    1.  Run the following command to get the machine object name from the BMH object.
        
            # oc -n openshift-machine-api get bmh,machine
        
        Example output:
        
            # oc -n openshift-machine-api get bmh,machine
            
            NAME                                    STATE                    CONSUMER                           ONLINE   ERROR   AGE
            baremetalhost.metal3.io/compute-1-ru5   provisioned              isf-rackae6-42ps4-worker-0-r6fmw   true             22h
            baremetalhost.metal3.io/compute-1-ru6   provisioned              isf-rackae6-42ps4-worker-0-t922n   true             22h
            baremetalhost.metal3.io/compute-1-ru7   provisioned              isf-rackae6-42ps4-worker-0-g842m   true             22h
            baremetalhost.metal3.io/control-1-ru2   externally provisioned   isf-rackae6-42ps4-master-0         true             44h
            baremetalhost.metal3.io/control-1-ru3   externally provisioned   isf-rackae6-42ps4-master-1         true             44h
            baremetalhost.metal3.io/control-1-ru4   externally provisioned   isf-rackae6-42ps4-master-2         true             44h
            
            NAME                                                            PHASE     TYPE   REGION   ZONE   AGE
            machine.machine.openshift.io/isf-rackae6-42ps4-master-0         Running                          44h
            machine.machine.openshift.io/isf-rackae6-42ps4-master-1         Running                          44h
            machine.machine.openshift.io/isf-rackae6-42ps4-master-2         Running                          44h
            machine.machine.openshift.io/isf-rackae6-42ps4-worker-0-g842m   Running                          22h
            machine.machine.openshift.io/isf-rackae6-42ps4-worker-0-r6fmw   Running                          22h
            machine.machine.openshift.io/isf-rackae6-42ps4-worker-0-t922n   Running                          22h
            
        
        The BMH object of `compute-1-ru5` maps to the `isf-rackae6-42ps4-worker-0-r6fmw` of the machine object.
        
    2.  Mark the machine object to delete by a special annotation `machine.openshift.io/cluster-api-delete-machine: delete-me`. The special marking helps to override the machine deletion policy rule.
        
            # oc -n openshift-machine-api edit machine isf-rackae6-42ps4-worker-0-r6fmw
            
            apiVersion: machine.openshift.io/v1beta1
            kind: Machine
            metadata:
              annotations:
                machine.openshift.io/cluster-api-delete-machine: delete-me
            ...
            
        
    3.  Now you need to scale down the machineset object so that the deletion of the machine object gets initiated.
        
        Note: After the machineset scale down is performed, the machine objects corresponding to `compute-1-ru5` are cleaned up, and the status of the BMH object corresponding to `compute-1-ru5` changes to deprovisioning.
        
            # oc -n openshift-machine-api get machineset
            NAME                         DESIRED   CURRENT   READY   AVAILABLE   AGE
            isf-rackae6-cltp4-worker-0   3         3         3       3           8h
            
            # oc -n openshift-machine-api get machineset -oyaml | grep replicas
             replicas: 3
            
            # oc -n openshift-machine-api scale --replicas=<old replica value - 1> machineset <machine set name>
            machineset.machine.openshift.io/isf-rackae6-42ps4-worker-0 scaled
            
        
    4.  After the machineset scale down is performed successfully, the status of the BMH object corresponding to `compute-1-ru5` changes to ready.
        
        Important: This activity might take few minutes, and wait for it to be reflected against the BMH object before proceed with the further steps.
        
3.  Delete the compute nodes in the node object from the OpenShift Container Platform cluster if anything present.
    
        # oc get nodes
        
        NAME                                            STATUS   ROLES           AGE   VERSION
        compute-1-ru6.isf-rackae6.rtp.raleigh.ibm.com   Ready    worker          21h   v1.23.17+16bcd69
        compute-1-ru7.isf-rackae6.rtp.raleigh.ibm.com   Ready    worker          21h   v1.23.17+16bcd69
        control-1-ru2.isf-rackae6.rtp.raleigh.ibm.com   Ready    master,worker   43h   v1.23.17+16bcd69
        control-1-ru3.isf-rackae6.rtp.raleigh.ibm.com   Ready    master,worker   43h   v1.23.17+16bcd69
        control-1-ru4.isf-rackae6.rtp.raleigh.ibm.com   Ready    master,worker   43h   v1.23.17+16bcd69
        localhost                                       Ready    worker          20h   v1.23.17+16bcd69
        
        # oc delete node localhost 
        
    
4.  Delete the BMH object of the compute node.
    
         # oc -n openshift-machine-api get bmh 
        
        NAME            STATE                    CONSUMER                           ONLINE   ERROR   AGE
        compute-1-ru5   available                                                   false            24h
        compute-1-ru6   provisioned              isf-rackae6-42ps4-worker-0-t922n   true             24h
        compute-1-ru7   provisioned              isf-rackae6-42ps4-worker-0-g842m   true             24h
        control-1-ru2   externally provisioned   isf-rackae6-42ps4-master-0         true             46h
        control-1-ru3   externally provisioned   isf-rackae6-42ps4-master-1         true             46h
        control-1-ru4   externally provisioned   isf-rackae6-42ps4-master-2         true             46h
        
        # oc -n openshift-machine-api delete bmh compute-1-ru5
        baremetalhost.metal3.io "compute-1-ru5" deleted
        
    
5.  Delete all pending `CertificateSigningRequests` corresponding to the localhost node.
    
        # for i in `oc get csr --no-headers | grep -i system:node:localhost | grep -i pending | awk '{ print $1 }'`;do oc delete csr $i; done
        
    
6.  Fix the DNS or DHCP issue to get the correct hostname for the corresponding compute node instead of the local host.
7.  Delete the `ComputeProvisionWorker` object of the compute node.
    
        # oc -n ibm-spectrum-fusion-ns get cpw 
        NAME                            AGE
        provisionworker-compute-1-ru5   24h
        provisionworker-compute-1-ru6   24h
        provisionworker-compute-1-ru7   24h
        
        # oc -n ibm-spectrum-fusion-ns delete cpw provisionworker-compute-1-ru5
        computeprovisionworker.install.isf.ibm.com "provisionworker-compute-1-ru5" deleted
        
    
8.  If the issue occurs during installation at the time of OpenShift configuration, the node conversion resumes automatically. If the issue occurs during the node upsize, then retry the node addition.

**Parent topic:** [Node issues](../tshooting/sf_compute_troubleshooting.html "List of all troubleshooting and known issues while you work with the compute nodes.")


Backup & restore service from OpenShift Container Platform
==========================================================

List of issues in the Backup & Restore managed and monitored from the OpenShift® Container Platform console.

fusion-console plugin issue
---------------------------

Problem statement

The OpenShift Container Platform console pod gets into panic state initially with a connection error.

Cause

This error occurs because the inter-namespace communication gets blocked between OpenShift Container Platform console pod (`openshift-console` namespace) and `fusion-console` pod (running in IBM Storage Fusion operator install namespace).

Resolution

1.  Check whether any typo exists in the \*.apps definition or there is a bad name resolution.
2.  If the error persists even after you fix the issue in step 1, restart the Fusion proxy pods.
3.  Verify that no `NetworkPolicy/NetworkPolicies` or networking plugin configurations exist that can prevent or interfere with inter-namespace communication.

Known issue
-----------

The Fusion Native Console plugin follows the n, n-1 & n-2 support matrix of OpenShift Container Platform, where "n" is an even OpenShift Container Platform version. For example: Fusion Console 2.8 on IBM Storage Fusion 4.16, 4.15 and 4.14 releases. In case of versions that are not supported, run the following command to disable the plugin:

    oc patch -n ibm-spectrum-fusion-ns ISFConsolePlugin isf-console --type=merge -p '{"spec":{"enabled": 'false'}}'

Here, the `ibm-spectrum-fusion-ns` is the namespace where the IBM Storage Fusion operator is installed.

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")


Known issues and limitations
============================

Known issues and limitations in IBM Fusion Data Foundation.

The IBM Fusion Data Foundation dashboard includes inaccurate capacity information on the amount of data that is written to CephFS PVs. For more information about the issue, see [https://bugzilla.redhat.com/show\_bug.cgi?id=2252911](https://bugzilla.redhat.com/show_bug.cgi?id=2252911 "(Opens in a new tab or window)").

**Parent topic:** [IBM Fusion Data Foundation service error scenarios](../tshooting/sf_odf_troubleshooting.html "Use these troubleshooting information to know the problem and workaround when install or configure IBM Fusion Data Foundation service.")


Troubleshooting upgrade issues
==============================

Troubleshooting IBM Storage Fusion HCI System upgrade issues.

**Install strategy fails for Fusion Operator after cluster upgrade to Red Hat OpenShift Container Platform 4.15.3**
-------------------------------------------------------------------------------------------------------------------

Problem statement

The IBM Storage Fusion HCI System 2.7.2 is upgraded to 2.8.0 with Red Hat® OpenShift® Container Platform 4.14.x. If Red Hat OpenShift Container Platform is upgraded to 4.15.2 or higher in this setup, then the Fusion operator status in OperatorHub fails with the following error:

install strategy failed: rolebindings.rbac.authorization.k8s.io "isf-update-operator-controller-manager-service-auth-reader" already exists 

Cause

The error occurs because of a known Red Hat OpenShift Container Platform issue. For more information about the issue, see [https://issues.redhat.com/projects/OCPBUGS/issues/OCPBUGS-32311?filter=allopenissues](https://issues.redhat.com/projects/OCPBUGS/issues/OCPBUGS-32311?filter=allopenissues "(Opens in a new tab or window)").

Resolution

1.  Run the following command to check all the existing role bindings.
    
        oc get csv isf-operator.v2.8.0 -ojson | jq
                '.status.conditions[].message' -n ibm-spectrum-fusion-ns  
    
    Sample output of the command:
    
    "all requirements found, attempting install"
    "install strategy failed: rolebindings.rbac.authorization.k8s.io \\"isf-application-operator-controller-manager-service-auth-reader\\" already exists"
    "webhooks not installed"
    "all requirements found, attempting install"
    "install strategy failed: rolebindings.rbac.authorization.k8s.io \\"isf-application-operator-controller-manager-service-auth-reader\\" already exists"
    "webhooks not installed"
    "all requirements found, attempting install"
    "install strategy failed: rolebindings.rbac.authorization.k8s.io \\"isf-application-operator-controller-manager-service-auth-reader\\" already exists"
    
2.  Take back up of the YAMLs of the reported role bindings.
    
        oc get rolebinding isf-application-operator-controller-manager-service-auth-reader -n kube-system -o yaml > isf-application-operator-controller-manager-service-auth-reader_rb.yaml
        
    
3.  Run the following command to delete each reported role binding:
    
        oc delete rolebinding
                isf-application-operator-controller-manager-service-auth-reader -n kube-system
    
4.  Iterate through steps 1, 2, and 3 until the IBM Storage Fusion HCI System operator CSV reports Healthy and the Fusion operator status shows `Succeeded`.

catalogsource isf-catalog does not get updated with status
----------------------------------------------------------

Problem statement

If you upgrade from 4.14 to 4.15, then it is fusion-catalog and not isf-catalog.

Resolution

1.  Log in to the OpenShift Container Platform console as a cluster administrator.
2.  Create a new CatalogSource by using the YAML editor.
    
    Sample catalogsource YAML for online upgrade:
    
        
        apiVersion: operators.coreos.com/v1alpha1
        kind: CatalogSource
        metadata:
          name: fusion-catalog
          namespace: openshift-marketplace
        spec:
          displayName: IBM Storage Fusion Catalog
          image: 'icr.io/cpopen/isf-operator-catalog:2.8.0-linux.amd64'
          publisher: IBM
          sourceType: grpc
    
    Sample catalogsource YAML for offline upgrade:
    
        
        apiVersion: operators.coreos.com/v1alpha1
        kind: CatalogSource
        metadata:
          name: fusion-catalog
          namespace: openshift-marketplace
        spec:
          displayName: IBM Storage Fusion Catalog
          image: $TARGET_PATH/isf-operator-catalog:2.8.0-linux.amd64'
          publisher: IBM
          sourceType: grpc
    
3.  Save the YAML.
4.  Confirm that the CatalogSource 'fusion-catalog' is in Ready state:
    1.  Go to Home > Search.
    2.  Change namespace to `openshift-marketplace`.
    3.  In Resources, find `CatalogSource`.
    4.  From the list, select fusion-catalog. The Details tab opens by default.
    5.  Confirm that the status is Ready in the Details page.
5.  Go to Operators > Installed Operators and make sure that you select `ibm-spectrum-fusion-ns` project.
6.  From the Installed Operators list, select IBM Fusion that is on 2.7.2 version. The Details tab opens by default.
7.  Go to Subscription tab and check whether the Update approval is Manual or Automatic. If it is Automatic, change the Update approval to Manual.
8.  In the Update approval section, click edit icon and change the channel value to v2.0.
9.  Go to Actions and select Edit Subscription.
10.  In the YAML tab, update the value of the source in the `Spec` section to `fusion-catalog`.
11.  Save the YAML.
12.  Proceed with step [9](../upgrade/sf_upgrade_isf_operator_procedure.html#sf_upgrade_isf_operator__step9) of Upgrading IBM Storage Fusion HCI System management software topic.

Red Hat OpenShift Container Platform 4.13.24 does not progress beyond 80%
-------------------------------------------------------------------------

Problem statement

OpenShift Container Platform 4.13.24 does not progress beyond 80% and the OpenShift `service-ca` displays the following message:

Progressing: service-ca does not have available replicas

Resolution

Log in to OpenShift Container Platform by using the CLI and run the following command to fix the error related to the `service-ca` operator and its associated pods:

    oc adm policy remove-scc-from-group anyuid system:authenticated
    

For more information about this issue, see [https://access.redhat.com/solutions/5875621](https://access.redhat.com/solutions/5875621 "(Opens in a new tab or window)").

Restore failures post upgrade
-----------------------------

Problem statement

After upgrade, you might encounter some restore failures. Check whether the job logs of the failed restore jobs contain the following error:

    Invalid value: true: Privileged containers are not allowed, spec.containers[0].securityContext.capabilities.add: Invalid value: "SYS_ADMIN": capability may not be added, provider "nonroot":

Resolution

Contact IBM Support to resolve this known issue.

BMH shows registration error after OpenShift Container Platform upgrade 4.12.36 to 4.13.15 or higher
----------------------------------------------------------------------------------------------------

Problem statement

BMH shows registration error after OpenShift Container Platform upgrade 4.12.36 to 4.13.15 or higher. You can observe this error in the BMH of every node in the cluster.

Symptom

After IBM Storage Fusion HCI System 2.6.2 cluster is upgraded to OpenShift Container Platform 4.13, the existing BMH for compute nodes starts to show registration error. To view the error, go to console > Compute > Bare metal host.

Diagnosis

If you describe BMH by using the following command, you can see an error that indicates that field slaves is not recognized:

    oc -n openshift-machine-api describe bmh <bmh name>

Sample error output:

Cannot generate image: serde\_yaml::Error: unknown field \`slaves\`, expected one of \`mode\`, \`options\`, \`ports\`, \`port\` 

This error occurs because OpenShift Container Platform 4.13 uses `nmstate/nmcli` spec that does not recognize `slaves` keyword in the bond configuration.

Resolution

Run the following steps to resolve the registration error:

1.  For every such BMH, find the network secret. For example:
    
        oc -n openshift-machine-api get secret |grep network-secret
    
2.  Gt the secret. For example, `compute-1-ru5 bmh`:
    
        oc -n openshift-machine-api get secret compute-1-ru5-network-secret      -o json|jq '.data.nmstate' |tr -d '"'
        CiAgICBpbnRlcmZhY2VzOgogICAgLSBkZXNjcmlwdGlvbjogQm9uZCBjb25uZWN0aW9uIGVuc2xhdmluZyBiYXJlbWV0YWwgaW50ZXJmYWNlcwogICAgICBpcHY0OgogICAgICAgIGRoY3A6IHRydWUKICAgICAgICBlbmFibGVkOiB0cnVlCiAgICAgIGlwdjY6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgbGluay1hZ2dyZWdhdGlvbjoKICAgICAgICBtb2RlOiA4MDIuM2FkCiAgICAgICAgb3B0aW9uczoKICAgICAgICAgIGxhY3BfcmF0ZTogIjEiCiAgICAgICAgICBtaWltb246ICIxNDAiCiAgICAgICAgICB4bWl0X2hhc2hfcG9saWN5OiAiMSIKICAgICAgICBwb3J0czoKICAgICAgICAtIGVuczFmMG5wMAogICAgICAgIC0gZW5zMWYwbnAxCiAgICAgIG5hbWU6IGJvbmQwCiAgICAgIG1hYy1hZGRyZXNzOiAiMTA6NzA6ZmQ6Yjg6ZTY6NWUiCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBCb25kIGNvbm5lY3Rpb24gZW5zbGF2aW5nIGVuczNmMCBhbmQgZW5zM2YxCiAgICAgIGlwdjQ6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgaXB2NjoKICAgICAgICBhZGRyZXNzOgogICAgICAgIC0gaXA6IGZlODA6OmVhZWI6ZDNmZjpmZWZkOmNmYjgKICAgICAgICAgIHByZWZpeC1sZW5ndGg6IDY0CiAgICAgICAgLSBpcDogZmQ4YzoyMTVkOjE3OGU6YzBkZTplYWViOmQzZmY6ZmVmZDpjZmI4CiAgICAgICAgICBwcmVmaXgtbGVuZ3RoOiA2NAogICAgICAgIGF1dG9jb25mOiBmYWxzZQogICAgICAgIGRoY3A6IGZhbHNlCiAgICAgICAgZW5hYmxlZDogdHJ1ZQogICAgICBsaW5rLWFnZ3JlZ2F0aW9uOgogICAgICAgIG1vZGU6IDgwMi4zYWQKICAgICAgICBvcHRpb25zOgogICAgICAgICAgbWlpbW9uOiAiMTQwIgogICAgICAgICAgbGFjcF9yYXRlOiAiMSIKICAgICAgICAgIHhtaXRfaGFzaF9wb2xpY3k6ICIxIgogICAgICAgIHBvcnRzOgogICAgICAgIC0gZW5zM2YwbnAwCiAgICAgICAgLSBlbnMzZjFucDEKICAgICAgbXR1OiA5MDAwCiAgICAgIG5hbWU6IGJvbmQxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBzdG9yYWdlIHZsYW4gaW50ZXJmYWNlIGNvbm5lY3Rpb24gb24gdG9wIG9mIGJvbmQxCiAgICAgIG10dTogOTAwMAogICAgICBuYW1lOiBib25kMS4zMjAxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiB2bGFuCiAgICAgIHZsYW46CiAgICAgICAgYmFzZS1pZmFjZTogYm9uZDEKICAgICAgICBpZDogMzIwMQ==
    
3.  Base64 decode from the previous command:
    
     echo CiAgICBpbnRlcmZhY2VzOgogICAgLSBkZXNjcmlwdGlvbjogQm9uZCBjb25uZWN0aW9uIGVuc2xhdmluZyBiYXJlbWV0YWwgaW50ZXJmYWNlcwogICAgICBpcHY0OgogICAgICAgIGRoY3A6IHRydWUKICAgICAgICBlbmFibGVkOiB0cnVlCiAgICAgIGlwdjY6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgbGluay1hZ2dyZWdhdGlvbjoKICAgICAgICBtb2RlOiA4MDIuM2FkCiAgICAgICAgb3B0aW9uczoKICAgICAgICAgIGxhY3BfcmF0ZTogIjEiCiAgICAgICAgICBtaWltb246ICIxNDAiCiAgICAgICAgICB4bWl0X2hhc2hfcG9saWN5OiAiMSIKICAgICAgICBwb3J0czoKICAgICAgICAtIGVuczFmMG5wMAogICAgICAgIC0gZW5zMWYwbnAxCiAgICAgIG5hbWU6IGJvbmQwCiAgICAgIG1hYy1hZGRyZXNzOiAiMTA6NzA6ZmQ6Yjg6ZTY6NWUiCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBCb25kIGNvbm5lY3Rpb24gZW5zbGF2aW5nIGVuczNmMCBhbmQgZW5zM2YxCiAgICAgIGlwdjQ6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgaXB2NjoKICAgICAgICBhZGRyZXNzOgogICAgICAgIC0gaXA6IGZlODA6OmVhZWI6ZDNmZjpmZWZkOmNmYjgKICAgICAgICAgIHByZWZpeC1sZW5ndGg6IDY0CiAgICAgICAgLSBpcDogZmQ4YzoyMTVkOjE3OGU6YzBkZTplYWViOmQzZmY6ZmVmZDpjZmI4CiAgICAgICAgICBwcmVmaXgtbGVuZ3RoOiA2NAogICAgICAgIGF1dG9jb25mOiBmYWxzZQogICAgICAgIGRoY3A6IGZhbHNlCiAgICAgICAgZW5hYmxlZDogdHJ1ZQogICAgICBsaW5rLWFnZ3JlZ2F0aW9uOgogICAgICAgIG1vZGU6IDgwMi4zYWQKICAgICAgICBvcHRpb25zOgogICAgICAgICAgbWlpbW9uOiAiMTQwIgogICAgICAgICAgbGFjcF9yYXRlOiAiMSIKICAgICAgICAgIHhtaXRfaGFzaF9wb2xpY3k6ICIxIgogICAgICAgIHBvcnRzOgogICAgICAgIC0gZW5zM2YwbnAwCiAgICAgICAgLSBlbnMzZjFucDEKICAgICAgbXR1OiA5MDAwCiAgICAgIG5hbWU6IGJvbmQxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBzdG9yYWdlIHZsYW4gaW50ZXJmYWNlIGNvbm5lY3Rpb24gb24gdG9wIG9mIGJvbmQxCiAgICAgIG10dTogOTAwMAogICAgICBuYW1lOiBib25kMS4zMjAxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiB2bGFuCiAgICAgIHZsYW46CiAgICAgICAgYmFzZS1pZmFjZTogYm9uZDEKICAgICAgICBpZDogMzIwMQ==|base64 -d
    
        interfaces:
        - description: Bond connection enslaving baremetal interfaces
          ipv4:
            autoconf: true
            dhcp: true
            enabled: true
          ipv6:
            enabled: false
          link-aggregation:
            mode: 802.3ad
            options:
              lacp\_rate: "1"
              miimon: "140"
              xmit\_hash\_policy: "1"
            slaves:
            - ens1f0np0
            - ens1f0np1
          name: bond0
          mac-address: "10:70:fd:b8:e6:5e"
          state: up
          type: bond
        - description: Bond connection enslaving ens3f0 and ens3f1
          ipv4:
            enabled: false
          ipv6:
            address:
            - ip: fe80::eaeb:d3ff:fefd:cfb8
              prefix-length: 64
            - ip: fd8c:215d:178e:c0de:eaeb:d3ff:fefd:cfb8
              prefix-length: 64
            autoconf: false
            dhcp: false
            enabled: true
          link-aggregation:
            mode: 802.3ad
            options:
              miimon: "140"
              lacp\_rate: "1"
              xmit\_hash\_policy: "1"
            slaves:
            - ens3f0np0
            - ens3f1np1
          mtu: 9000
          name: bond1
          state: up
          type: bond
        - description: storage vlan interface connection on top of bond1
          mtu: 9000
          name: bond1.3201
          state: up
          type: vlan
          vlan:
            base-iface: bond1
            id: 3201
    
4.  In a YAML editor, paste the output in the previous step and replace two occurrences of `slaves` with `ports` and remove `autoconf: true` from ipv4 section:
    
    interfaces:
        - description: Bond connection enslaving baremetal interfaces
          ipv4:
            dhcp: true
            enabled: true
          ipv6:
            enabled: false
          link-aggregation:
            mode: 802.3ad
            options:
              lacp\_rate: "1"
              miimon: "140"
              xmit\_hash\_policy: "1"
            ports:
            - ens1f0np0
            - ens1f0np1
          name: bond0
          mac-address: "10:70:fd:b8:e6:5e"
          state: up
          type: bond
        - description: Bond connection enslaving ens3f0 and ens3f1
          ipv4:
            enabled: false
          ipv6:
            address:
            - ip: fe80::eaeb:d3ff:fefd:cfb8
              prefix-length: 64
            - ip: fd8c:215d:178e:c0de:eaeb:d3ff:fefd:cfb8
              prefix-length: 64
            autoconf: false
            dhcp: false
            enabled: true
          link-aggregation:
            mode: 802.3ad
            options:
              miimon: "140"
              lacp\_rate: "1"
              xmit\_hash\_policy: "1"
            ports:
            - ens3f0np0
            - ens3f1np1
          mtu: 9000
          name: bond1
          state: up
          type: bond
        - description: storage vlan interface connection on top of bond1
          mtu: 9000
          name: bond1.3201
          state: up
          type: vlan
          vlan:
            base-iface: bond1
            id: 3201
    
5.  base64 encode this changed YAML file content:
    
        cat /home/garg/slaves.yaml |base64 -w0
        aW50ZXJmYWNlczoKICAgIC0gZGVzY3JpcHRpb246IEJvbmQgY29ubmVjdGlvbiBlbnNsYXZpbmcgYmFyZW1ldGFsIGludGVyZmFjZXMKICAgICAgaXB2NDoKICAgICAgICBkaGNwOiB0cnVlCiAgICAgICAgZW5hYmxlZDogdHJ1ZQogICAgICBpcHY2OgogICAgICAgIGVuYWJsZWQ6IGZhbHNlCiAgICAgIGxpbmstYWdncmVnYXRpb246CiAgICAgICAgbW9kZTogODAyLjNhZAogICAgICAgIG9wdGlvbnM6CiAgICAgICAgICBsYWNwX3JhdGU6ICIxIgogICAgICAgICAgbWlpbW9uOiAiMTQwIgogICAgICAgICAgeG1pdF9oYXNoX3BvbGljeTogIjEiCiAgICAgICAgcG9ydHM6CiAgICAgICAgLSBlbnMxZjBucDAKICAgICAgICAtIGVuczFmMG5wMQogICAgICBuYW1lOiBib25kMAogICAgICBtYWMtYWRkcmVzczogIjEwOjcwOmZkOmI4OmU2OjVlIgogICAgICBzdGF0ZTogdXAKICAgICAgdHlwZTogYm9uZAogICAgLSBkZXNjcmlwdGlvbjogQm9uZCBjb25uZWN0aW9uIGVuc2xhdmluZyBlbnMzZjAgYW5kIGVuczNmMQogICAgICBpcHY0OgogICAgICAgIGVuYWJsZWQ6IGZhbHNlCiAgICAgIGlwdjY6CiAgICAgICAgYWRkcmVzczoKICAgICAgICAtIGlwOiBmZTgwOjplYWViOmQzZmY6ZmVmZDpjZmI4CiAgICAgICAgICBwcmVmaXgtbGVuZ3RoOiA2NAogICAgICAgIC0gaXA6IGZkOGM6MjE1ZDoxNzhlOmMwZGU6ZWFlYjpkM2ZmOmZlZmQ6Y2ZiOAogICAgICAgICAgcHJlZml4LWxlbmd0aDogNjQKICAgICAgICBhdXRvY29uZjogZmFsc2UKICAgICAgICBkaGNwOiBmYWxzZQogICAgICAgIGVuYWJsZWQ6IHRydWUKICAgICAgbGluay1hZ2dyZWdhdGlvbjoKICAgICAgICBtb2RlOiA4MDIuM2FkCiAgICAgICAgb3B0aW9uczoKICAgICAgICAgIG1paW1vbjogIjE0MCIKICAgICAgICAgIGxhY3BfcmF0ZTogIjEiCiAgICAgICAgICB4bWl0X2hhc2hfcG9saWN5OiAiMSIKICAgICAgICBwb3J0czoKICAgICAgICAtIGVuczNmMG5wMAogICAgICAgIC0gZW5zM2YxbnAxCiAgICAgIG10dTogOTAwMAogICAgICBuYW1lOiBib25kMQogICAgICBzdGF0ZTogdXAKICAgICAgdHlwZTogYm9uZAogICAgLSBkZXNjcmlwdGlvbjogc3RvcmFnZSB2bGFuIGludGVyZmFjZSBjb25uZWN0aW9uIG9uIHRvcCBvZiBib25kMQogICAgICBtdHU6IDkwMDAKICAgICAgbmFtZTogYm9uZDEuMzIwMQogICAgICBzdGF0ZTogdXAKICAgICAgdHlwZTogdmxhbgogICAgICB2bGFuOgogICAgICAgIGJhc2UtaWZhY2U6IGJvbmQxCiAgICAgICAgaWQ6IDMyMDEK
    
6.  Edit the network secret and replace spec.data.nmstate value with the base64 encoded value:
    
        oc -n openshift-machine-api  edit secret  compute-1-ru5-network-secret
    
    Example:
    
    apiVersion: v1
    data:
      nmstate: CiAgICBpbnRlcmZhY2VzOgogICAgLSBkZXNjcmlwdGlvbjogQm9uZCBjb25uZWN0aW9uIGVuc2xhdmluZyBiYXJlbWV0YWwgaW50ZXJmYWNlcwogICAgICBpcHY0OgogICAgICAgIGRoY3A6IHRydWUKICAgICAgICBlbmFibGVkOiB0cnVlCiAgICAgIGlwdjY6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgbGluay1hZ2dyZWdhdGlvbjoKICAgICAgICBtb2RlOiA4MDIuM2FkCiAgICAgICAgb3B0aW9uczoKICAgICAgICAgIGxhY3BfcmF0ZTogIjEiCiAgICAgICAgICBtaWltb246ICIxNDAiCiAgICAgICAgICB4bWl0X2hhc2hfcG9saWN5OiAiMSIKICAgICAgICBwb3J0czoKICAgICAgICAtIGVuczFmMG5wMAogICAgICAgIC0gZW5zMWYwbnAxCiAgICAgIG5hbWU6IGJvbmQwCiAgICAgIG1hYy1hZGRyZXNzOiAiMTA6NzA6ZmQ6Yjg6ZTY6NWUiCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBCb25kIGNvbm5lY3Rpb24gZW5zbGF2aW5nIGVuczNmMCBhbmQgZW5zM2YxCiAgICAgIGlwdjQ6CiAgICAgICAgZW5hYmxlZDogZmFsc2UKICAgICAgaXB2NjoKICAgICAgICBhZGRyZXNzOgogICAgICAgIC0gaXA6IGZlODA6OmVhZWI6ZDNmZjpmZWZkOmNmYjgKICAgICAgICAgIHByZWZpeC1sZW5ndGg6IDY0CiAgICAgICAgLSBpcDogZmQ4YzoyMTVkOjE3OGU6YzBkZTplYWViOmQzZmY6ZmVmZDpjZmI4CiAgICAgICAgICBwcmVmaXgtbGVuZ3RoOiA2NAogICAgICAgIGF1dG9jb25mOiBmYWxzZQogICAgICAgIGRoY3A6IGZhbHNlCiAgICAgICAgZW5hYmxlZDogdHJ1ZQogICAgICBsaW5rLWFnZ3JlZ2F0aW9uOgogICAgICAgIG1vZGU6IDgwMi4zYWQKICAgICAgICBvcHRpb25zOgogICAgICAgICAgbWlpbW9uOiAiMTQwIgogICAgICAgICAgbGFjcF9yYXRlOiAiMSIKICAgICAgICAgIHhtaXRfaGFzaF9wb2xpY3k6ICIxIgogICAgICAgIHBvcnRzOgogICAgICAgIC0gZW5zM2YwbnAwCiAgICAgICAgLSBlbnMzZjFucDEKICAgICAgbXR1OiA5MDAwCiAgICAgIG5hbWU6IGJvbmQxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiBib25kCiAgICAtIGRlc2NyaXB0aW9uOiBzdG9yYWdlIHZsYW4gaW50ZXJmYWNlIGNvbm5lY3Rpb24gb24gdG9wIG9mIGJvbmQxCiAgICAgIG10dTogOTAwMAogICAgICBuYW1lOiBib25kMS4zMjAxCiAgICAgIHN0YXRlOiB1cAogICAgICB0eXBlOiB2bGFuCiAgICAgIHZsYW46CiAgICAgICAgYmFzZS1pZmFjZTogYm9uZDEKICAgICAgICBpZDogMzIwMQ==
    kind: Secret
    metadata:
      creationTimestamp: "2023-11-06T05:13:06Z"
      labels:
        environment.metal3.io: baremetal
      name: compute-1-ru5-network-secret
      namespace: openshift-machine-api
      ownerReferences:
      - apiVersion: metal3.io/v1alpha1
        kind: PreprovisioningImage
        name: compute-1-ru5
        uid: 1591fbbd-b787-4c46-b37e-53cbd62d885e
      - apiVersion: metal3.io/v1alpha1
        kind: BareMetalHost
        name: compute-1-ru5
        uid: 0e03f8c3-6c01-4cd9-b499-03e4f67cd753
      resourceVersion: "8788604"
      uid: ee1af992-6c19-449b-8cb2-7315d1692498
    type: Opaque
    

Community operator catalog is shown as missing
----------------------------------------------

Resolution

If the Community operator catalog is shown as missing, then create it before you attempt upgrade.

Machine config roll out error
-----------------------------

Resolution

If an operation causes `machine config roll out` and gets stuck for a long time, then check whether the node to be updated is pingable and has an IP after restart. If there exist any DHCP or network issues that prevent the node from getting a hostname, then fix them and restart the node.

On-demand backup failures post upgrade
--------------------------------------

Problem statement

Post upgrade, on-demand backup failures might happen for existing applications.

Resolution

Do the following manual steps after you upgrade to avoid this problem:

1.  Run the following command to display the phase status of the backup policies associated with all your applications:
    
        oc get fpa -A
    
    Example output:
    
    oc get fpa -A
    
    NAMESPACE                NAME                                  PROVIDER     APPLICATION           BACKUPPOLICY      DATACONSISTENCY   PHASE             LASTBACKUPTIMESTAMP   CAPACITY
    ibm-spectrum-fusion-ns   deptest2-azure-hourly-30              isf-ibmspp   deptest2              azure-hourly-30                     Assigned          66m                   <no value>
    ibm-spectrum-fusion-ns   new-generic-1-azure-hourly-45         isf-ibmspp   new-generic-1         azure-hourly-45                     Assigned          21m                   <no value>
    ibm-spectrum-fusion-ns   new-mongo-project-1-azure-hourly-15   isf-ibmspp   new-mongo-project-1   azure-hourly-15                     InitializeError   81m                   <no value>
    ibm-spectrum-fusion-ns   new-mongo-project-azure-hourly-30     isf-ibmspp   new-mongo-project     azure-hourly-30                     Assigned          66m                   <no value>
    
2.  Verify whether your `policyassignment` CR corresponds to any application in `InitializeError` phase. In this example, the `new-mongo-project-1` application is in `InitializeError` phase.
3.  Log in to IBM Storage Fusion HCI user interface.
4.  Go to Applications > Backups tab.
5.  Unassign the backup policy that is assigned to the application in `InitializeError` phase and wait for its unassignment. In this example, unassign `azure-hourly-15` policy from `new-mongo-project-1` application.
6.  Reassign the backup policy.

ImagePull failure
-----------------

Resolution

If an `ImagePull` failure occurs due to intermittent network or registry issue during an upgrade, then restart the pod and retry. If the issue persists, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.").

DeadlineExceeded error
----------------------

Problem statement

IBM Cloud Paks foundational services operator ClusterServiceVersion (CSV) status shows Failed and its InstallPlan status shows Failed after the subscription gets created.

Resolution

If you notice that the operator installation or upgrade fails with `DeadlineExceeded error`, see [Operator installation or upgrade fails with DeadlineExceeded error](sf_deadlineexceeded.html "IBM Cloud Paks foundational services operator ClusterServiceVersion (CSV) status shows Failed and its InstallPlan status shows Failed after the subscription gets created.").

**Parent topic:** [Installation and upgrade issues](../tshooting/sf_install_tshooting.html "Use the troubleshooting tips and tricks in IBM Storage Fusion installation.")


Troubleshooting installation and upgrade issues in IBM Storage Fusion services
==============================================================================

Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.

*   **[Global Data Platform service issues](../tshooting/sf_gdp_installandupgrade_issues.html)**  
    Use this troubleshooting information to resolve install and upgrade problems that are related to Global Data Platform service.
*   **[Backup & Restore service install and upgrade issues](../tshooting/sf_br_installandupgrade_issues.html)**  
    Use this troubleshooting information to resolve install and upgrade problems that are related to Backup & Restore service.
*   **[Data Cataloging service issues](../tshooting/sf_dc_installandupgrade_issues.html)**  
    Use this troubleshooting information to resolve install and upgrade problems that are related to the Data Cataloging service.
*   **[Common service installation issues](../tshooting/sf_common_installandupgrade_issues.html)**  
    Use this troubleshooting information to resolve common install and upgrade problems related to IBM Storage Fusion services.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")



Add node failure
================

Node addition to the Red Hat® OpenShift® Container Platform cluster fails.

Severity
--------

Warning or Error

Explanation
-----------

Node addition to the Red Hat OpenShift Container Platform cluster reports a failure. As a part of the node addition process, IBM Storage Fusion HCI System validates and runs a few steps. If any of these validations or steps fail, a node addition failure gets reported.

Some of the reported failures can auto resolve after a few minutes because the IBM Storage Fusion HCI System retries the same validation and steps in every reconciliation cycle. You can view the actual failure reason in the status of the `computeprovisionworker` CR.

In the following example, the `message` field indicates a node DNS validation failure. Determine the failure message from the `computeprovisionworker` CR status, and follow the steps in the Recommended actions section.

    
    oc -n ibm-spectrum-fusion-ns get cpw provisionworker-compute-1-ru5 -o yaml
    apiVersion: install.isf.ibm.com/v1
    kind: ComputeProvisionWorker
    metadata:
     creationTimestamp: "2024-05-01T21:39:53Z"
     generation: 1
     name: provisionworker-compute-1-ru5
     namespace: ibm-spectrum-fusion-ns
     resourceVersion: "8733225"
     uid: 3aa59cb6-c032-4b31-98ec-26284ed4588a
    spec:
     location: RU5
     rackSerial: 8Y2DXXX
    status:
     hostname: compute-1-ru5.mycluster.mydomain.com
     installStatus: Installing
     ipAddress: XX.YY.ZZ.15
     location: RU5
     macAddress: 08:xx:eb:ff:yy:zz
     machineSetScaled: true
     message: DNS validation for node failed.
     messageCode: IOCPW0011
     name: provisionworker-compute-1-ru5
     nodeType: storage
     ocpRole: compute-1-ru5
     oneTimeNetworkBoot: Completed
     rackName: mycluster
     startTimeStamp: "2024-05-01T21:39:54Z"

Recommended actions
-------------------

DNS validation for node failed

The error indicates a node DNS validation failure, which can be intermittent because of an unsuccessful DNS call. It eventually succeeds in the next cycle. However, if the issue persists for more than 10 minutes, check the DHCP and DNS configurations to confirm that the correct FQDN to IP exists for the DNS lookup and reverse lookup. After you fix the issue, the node addition in IBM Storage Fusion HCI System proceeds automatically to the next step.

Unable to add a node and failed to set boot order

Usually, this failure gets resolved in the subsequent reconcile cycles. If it persists more than 10 minutes, then a Baseboard Management Controller (BMC) restart can resolve the issue. Sometimes, the IMM does not respond to ipmi or redfish commands. Contact IBM support to restart the BMC.

After your restart the BMC, the error gets resolved in the subsequent reconciliation cycles.

Failed to read userconfig secret object

The error indicates that an intermittent API error exists in reading the OpenShift Container Platform secret. It can occur due to a delay in the API response or some other network interruption. This error self-resolves automatically in the successive reconciliation cycles.

Failed to read kickstart configmap data

The error indicates that an intermittent API error exists in reading the OpenShift Container Platform configmap. It can occur due to a delay in the API response or some other network interruption. It self-resolves automatically in the successive reconciliation cycles.

Unable to add a node and the Red Hat OpenShift node did not transition to Ready state

The CSRs pending for approval is the most common reason for this error. The IBM Storage Fusion HCI System operator approves these CSRs as a part of the node addition process. If it does not occur as a rare case scenario, manually approve them to resolve the issue. Run the following commands to check and approve the pending CSRs:

1.  Run the following command to look for pending CSRs:
    
        oc get csr | grep -i pending
    
2.  Run the following command to approve pending CSRs:
    
        oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
    

Failed to scale machineset during new node addition

As a part of the Red Hat OpenShift cluster expansion process, a Bare Metal host object gets created for every node that is added to the cluster. This Bare Metal host object go through the following states: `registering` > `inspecting` > `available` > `provisioning` > `provisioned`.

After the Bare Metal host becomes `available`, the replica of the Bare Metal machineset CR in the OpenShift Container Platform cluster increases by one to start the machine creation process of that host. If that step fails, then you can see a reported error in the `computeprovisionworker` status. The Fusion operator tries to increase it in the next reconcile cycle again and that eventually succeeds. If it does not succeed in a rare case scenario, do the following steps to manually increase the replica count of the machineset CR.

1.  Run the following command to get the machineset name:
    
        oc get machinesets -n openshift-machine-api | tail -1 | awk '{print $1}'
    
2.  Run the following command to get the machineset current replica count:
    
        oc get machinesets -n openshift-machine-api | grep worker | awk '{print $2}'
    
3.  Run the following command to increase the replica by one:
    
        oc scale --replicas=<old replica count from step2> + 1 machineset <machineset name from step 1> -n openshift-machine-api
    

In the subsequent reconciliation cycles, the node addition process moves to the next steps and gets reflected in the IBM Storage Fusion HCI System user interface.

Node might contain an invalid hostname (localhost)

The error indicates that the added host did not get the expected fully qualified hostname assigned from DHCP. A common reason for this error is that the DHCP server does not get configured correctly. Check the following content in the DHCP server:

1.  Hostname option is set for this host entry
2.  No mistakes exist in the mac address that is used for this host entry
3.  DHCP service is running and it is reachable from the host

After you fix the issue on the DHCP side, restart the node by using the Bare Metal host object from the Red Hat OpenShift console. To restart the node, do the following steps from the OpenShift Container Platform web console:

1.  Go to Home > Search.
2.  Search for CRs with `ComputePowerOp`. It lists CRs and nodes in the cluster.
3.  Go to the CR of the node to restart and then go to the YAML tab.
4.  Add the following line to `Spec` to power off the node.
    
        powerOP: GracefulShutdown
    
    if you want to power on the node, change the line to the following line:
    
        powerOP: On
    

Unable to add one or more nodes as they contain an invalid hostname (localhost)

This error indicates that the host you want to add did not get the expected valid fully qualified hostname assigned from DHCP. A common reason for this error is that the DHCP server is not configured correctly. In a DHCP server, check whether the following conditions exist:

*   Hostname option is set for this host entry
*   No mistake in the mac address used for this host entry
*   DHCP service is running and DHCP is reachable from the host

After you fix the DHCP issues, restart the node to proceed with the node addition. To restart node, do the following steps from the Red Hat OpenShift Container Platform web console:

1.  Go to Home > Search.
2.  Search for CRs with `ComputePowerOp`.
3.  Go to the CR of the node to restart and then go to the YAML tab.
4.  Add the following line to `Spec` to power off the node.
    
        powerOP: GracefulShutdown
    
    if you want to power on the node, change the line to the following line:
    
        powerOP: On
    

**Parent topic:** [Node issues](../tshooting/sf_compute_troubleshooting.html "List of all troubleshooting and known issues while you work with the compute nodes.")



Backup issues
=============

List of backup issues in the Backup & Restore service of IBM Storage Fusion.

Failed to create snapshot content
---------------------------------

Problem statement

Failed to create snapshot content with the following error:

Cannot find CSI PersistentVolumeSource for directory-based static volume

Resolution

To resolve the error, see [https://www.ibm.com/docs/en/scalecsi/2.10?topic=snapshot-create-volumesnapshot](https://www.ibm.com/docs/en/scalecsi/2.10?topic=snapshot-create-volumesnapshot "(Opens in a new tab or window)").

Assign a backup policy operation fails
--------------------------------------

Problem statement

If you have a PolicyAssignment for an application on the hub and you create a PolicyAssignment for the same application on the spoke, then your attempt to assign a backup policy for the application fails. In both assignments, the application, backup policy, and short-form cluster name are the same. The current format of the PolicyAssignment CR name is `appName-backupPolicyName-shortFormClusterName`. The issue happens when the first string of the cluster names is identical. In this scenario, the creation gets rejected because the PolicyAssignment name exists in OpenShift® Container Platform.

For example:

Hub assignment creates `app1-bp1-apps`:

*   Application - `app1`
*   BackupPolicy - `bp1`
*   AppCluster - `apps.cluster1`

Spoke assignment creates `app1-bp1-apps` (The OpenShift Container Platform rejects it)

*   Application - `app1`
*   BackupPolicy - `bp1`
*   AppCluster - `apps.cluster2`

Resolution

To create the PolicyAssignment for the spoke application, delete the PolicyAssignment CR for the hub application assignment and attempt spoke application assignment again.

Backups do not work as defined in the backup policies
-----------------------------------------------------

Problem statement

Sometimes, backups do not work as defined in the backup policies, especially when you set hourly policies. For example, if you set a policy for two hours and it does not run every two hours, then gaps exist in the backup history. The possible reason might be that during pod crash and restart, scheduled jobs were not accounting for the time zone, causing gaps in run intervals.

Diagnosis

The following are the observed symptoms:

*   Policies with custom every X hour at minute YY schedules: the first scheduled run of this policy will run at minute YY after X hours + time zone offset from UTC instead of at minute YY after X hours.
*   Monthly and yearly policies run more frequently.

Resolution

You can start backups manually until the next scheduled time.

Backup & Restore service deployed in IBM Cloud Satellite
--------------------------------------------------------

Problem statement

You can encounter an error when you attempt backup operation on IBM Storage Fusion Backup & Restore service that is deployed in IBM Cloud® Satellite.

Diagnosis

Backup operations fail with the following log entries:

    
    level=error msg="Error backing up item" backup=<item> error="error executing custom action (groupResource=pods, namespace=<namespace>, name=<name>): rpc error: code = Unknown desc = configmaps \"config\" not found" error.file="/remote-source/velero/app/pkg/backup/item_backupper.go:326" error.function="github.com/vmware-tanzu/velero/pkg/backup.(*itemBackupper).executeActions" logSource="/remote-source/velero/app/pkg/backup/backup.go:417" name=<name>
    level=error msg="Error backing up item" backup=<item> error="error executing custom action (groupResource=replicasets.apps, namespace=<namespace>, name=<name>): rpc error: code = Unknown desc = configmaps \"config\" not found" error.file="/remote-source/velero/app/pkg/backup/item_backupper.go:326" error.function="github.com/vmware-tanzu/velero/pkg/backup.(*itemBackupper).executeActions" logSource="/remote-source/velero/app/pkg/backup/backup.go:417" name=<name>
    level=error msg="Error backing up item" backup=<item> error="error executing custom action (groupResource=deployments.apps, namespace=<namespace>, name=<name>): rpc error: code = Unknown desc = configmaps \"config\" not found" error.file="/remote-source/velero/app/pkg/backup/item_backupper.go:326" error.function="github.com/vmware-tanzu/velero/pkg/backup.(*itemBackupper).executeActions" logSource="/remote-source/velero/app/pkg/backup/backup.go:417" name=<name>
    

Cause

An issue exists with the default OADP plug-in and it must be disabled to continue.

Resolution

Do the following steps to disable the plug-in:

1.  In the OpenShift console, go to Administration > CustomerResourceDefinitions.
2.  Search for the CustomResourceDefiniton `DataProtectionApplication`.
3.  In the Instances tab, locate the instance that is named `velero`.
4.  Open the YAML file in edit mode for the instance.
5.  Under the entry `spec:velero:defaultPlugins`, remove the line for `openshift`.
6.  Save the YAML file.

Backup jobs are stuck in a running state for a long time and are not canceled
-----------------------------------------------------------------------------

Resolution

Do the following steps to resolve the issue:

1.  Ensure that all jobs are finished and the queue is empty before you do any disruptive actions like node restarts.
2.  If jobs are running for a long period and do not progress, follow the steps to delete the backup or restore CR directly.
    1.  Log in to IBM Storage Fusion.
    2.  Go to Backup & Restore > Jobs > Queue and get the name of the job that is stuck.
    3.  Run the following command to delete backup job.
        
            oc delete fbackup <job_name>
        
    4.  Run the following command to delete restore job.
        
            oc delete frestore <job_name>
        

Policy creation
---------------

Problem statement

Sometimes, when you create a backup policy, the following errors can occur:

Error: Policy daily-snapshot could not created. 

Resolution

Restart the `isf-data-protection-operator-controller-manager-* pod` in IBM Storage Fusion namespace. It triggers the recreation of the in-place-snapshot BackupStorageLocation CR.

Policy assignment from Backup & Restore service page of the OpenShift Container Platform console
------------------------------------------------------------------------------------------------

Problem statement

In the Backup & Restore service page of the OpenShift Container Platform console, the backup policy assignment to an application fails with a gateway timeout error.

Resolution

Use your IBM Storage Fusion user interface.

Backup of multiple VMs attempt is failed
----------------------------------------

Problem statement

This issue occurs when some VMs are in a migrating state. The OpenShift Container Platform does not support snapshot of the VMs in migrating state.

Resolution

Follow the steps to resolve this issue:

1.  Check whether the virtual machine is in a migrating state:
2.  Run the following command to check migrating VM.
    
        oc get virtualmachineinstancemigrations -A
    
    Example output:
    
    NAMESPACE            NAME                                          PHASE         VMI
    fb-bm1-fs-1-5g-10    rhel8-lesser-wildcat-migration-8fhbo          Failed        rhel8-lesser-wildcat
    vm-centipede-bm2     centos-stream9-chilly-hawk-migration-57jyk    Failed        centos-stream9-chilly-hawk
    vm-centos9-bm1-1     centos-stream9-instant-toad-migration-bfyz6   Failed        centos-stream9-instant-toad
    vm-centos9-bm1-1     centos-stream9-instant-toad-migration-d9547   Failed        centos-stream9-instant-toad
    vm-windows10-bm2-1   kubevirt-workload-update-4dm57                Failed        win10-zealous-unicorn
    vm-windows10-bm2-1   kubevirt-workload-update-f2s5w                Failed        win10-zealous-unicorn
    vm-windows10-bm2-1   kubevirt-workload-update-gt6nj                Failed        win10-zealous-unicorn
    vm-windows10-bm2-1   kubevirt-workload-update-rjwmn                Failed        win10-zealous-unicorn
    vm-windows10-bm2-1   kubevirt-workload-update-vfxfl                TargetReady   win10-zealous-unicorn
    vm-windows10-bm2-1   kubevirt-workload-update-z2thw                Failed        win10-zealous-unicorn
    vm-windows11-bm2-1   kubevirt-workload-update-9gr6v                Failed        win11-graceful-coyote
    vm-windows11-bm2-1   kubevirt-workload-update-clbck                Failed        win11-graceful-coyote
    vm-windows11-bm2-1   kubevirt-workload-update-j6pmx                Failed        win11-graceful-coyote
    vm-windows11-bm2-1   kubevirt-workload-update-sfbbx                Pending       win11-graceful-coyote
    vm-windows11-bm2-1   kubevirt-workload-update-th5dd                Failed        win11-graceful-coyote
    vm-windows11-bm2-1   kubevirt-workload-update-zl679                Failed        win11-graceful-coyote
    vm-windows11-bm2-2   kubevirt-workload-update-7dp6g                Failed        win11-conservative-moth
    vm-windows11-bm2-2   kubevirt-workload-update-9nb9m                TargetReady   win11-conservative-moth
    vm-windows11-bm2-2   kubevirt-workload-update-cdrf5                Failed        win11-conservative-moth
    vm-windows11-bm2-2   kubevirt-workload-update-dm8fz                Failed        win11-conservative-moth
    vm-windows11-bm2-2   kubevirt-workload-update-kwr6c                Failed        win11-conservative-moth
    vm-windows11-bm2-2   kubevirt-workload-update-zt8wx                Failed        win11-conservative-moth
    
3.  Exclude the migrating virtual machine from the backup. Reattempt it after the migration is complete.

Backup applications table does not show the new backup times for the backed-up applications
-------------------------------------------------------------------------------------------

Problem statement

The backup applications table does not show the new backup times for the backed-up applications.

Resolution

Go to the Applications and Jobs view to see the last successful backup job for a given application. For applications on the hub, the Applications table has the correct last backup time.

Backups are failing for the virtual machines
--------------------------------------------

Problem statement

The backups and snapshots are failing for the virtual machines that is mounted with second disk.

Resolution

1.  Run the following command to get disks details for the virtual machine.
    
        oc get virtualmachine -A -o json | jq '.items[] | [{name:.metadata.name, namespace:.metadata.namespace, volumes:.spec.template.spec.volumes}] | select(.[].volumes[].dataVolume | length > 1) | {name
        :.[].name, namespace:.[].namespace}'
    
    Example output:
    
    {
    "name": "rhel9-absent-basilisk",
    "namespace": "vmtesting"
    }
    
2.  If you find the virtual machines are mounted with second disk, then follow the steps mentioned in the [Red Hat solution](https://access.redhat.com/solutions/7041127 "(Opens in a new tab or window)") to resolve the issue.

Known issues and limitations
----------------------------

*   The OpenShift Container Platform cluster can have problems and become unusable. After you recover the cluster, rejoin the connections. For the steps to clean the connection and setup the connection between two clusters again, see [Connection setup after OpenShift Container Platform cluster recovery](sf_ocp_cluster_recover.html "The OpenShift Container Platform cluster can have problems and become unusable. After you recover the cluster, rejoin the connections.").OpenShift Container Platform cluster can have problems and become unusable.
*   The S3 bucket must not have an expiration policy or an archive rule. For more information about this known issue, see [S3 buckets must not enable expiration policies](sf_restore_issues.html#sf_restore_issues__s3).

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")



IBM Cloud Pak for Data backup and restore issues
================================================

This section lists the troubleshooting tips and tricks during backup and restore of IBM Cloud Pak for Data.

Original namespace as destination for IBM Cloud Pak for Data application
------------------------------------------------------------------------

Problem statement

OpenShift® restore from IBM Storage Protect Plus does not force you to select the original namespace as destination for IBM Cloud Pak for Data application.

Impact

From IBM Storage Protect Plus user interface, IBM Cloud Pak for Data restore fails whenever you do not select the original namespace as the target destination to restore to.

Resolution

Whenever you restore IBM Cloud Pak for Data from IBM Storage Protect Plus user interface, ensure that you select the original namespace as the restore destination.

These steps are applicable while you backup and test restore of IBM Cloud Pak for Data applications. For more information about the procedure, see [https://www.ibm.doc/docs/en/SSQNUZ\_4.5\_test/cpd/admin/online\_bar\_fusion\_spp.html](https://www.ibm.doc/docs/en/SSQNUZ_4.5_test/cpd/admin/online_bar_fusion_spp.html "(Opens in a new tab or window)").

Tethered namespace issues
-------------------------

Problem statement

For IBM Cloud Pak for Data Application type, you can tether secondary or other namespaces to the primary IBM Cloud Pak for Data platform namespace. However, after a successful restore, the IBM Storage Fusion application associated with the IBM Cloud Pak for Data namespace does not restore the tethered namespace.

Resolution

1.  Uninstall `cpdbr-oadp` and reinstall it.
2.  After you add the tethered namespaces, restart the `isf-application-controller-manager -xxxx` pod that is within the `ibm-spectrum-fusion-ns` to force a reconcile..
3.  In about five minutes, run the following command to verify whether the `fapp` is updated to include the tethered namespace in `spec.includedNamespaces`
    
        oc get fapp -n ibm-spectrum-fusion-ns cpd-instance -o yaml
    

IBM Cloud Pak for Data backup of zen namespace fails partially in IBM Storage Protect Plus user interface
---------------------------------------------------------------------------------------------------------

Problem statement

For an on-demand backup of a single application from a policy that has multiple applications assigned, a snapshot backup is taken only for that selected application but the copy backup is initiated for all of the applications assigned to the policy. This behavior causes the copy backup of the applications without a snapshot backup to fail. This issue does not impact the application selected for the ad-hoc backup.

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")



Backup & Restore service install and upgrade issues
===================================================

Use this troubleshooting information to resolve install and upgrade problems that are related to Backup & Restore service.

Application information not repopulated after service re-installation
---------------------------------------------------------------------

Problem statement

The application information gets removed during Backup & Restore service uninstallation. It does not get repopulated in MongoDB after the service re-installation. Restore jobs validation fail with the following error message.

postJobRequest: Application Info not returned from application service for applicationId

Resolution

The workaround is to restart the `isf-application-operator-controller-manager` pod in the IBM Storage Fusion namespace.

Backup & Restore server installation displays an Invalid Input error
--------------------------------------------------------------------

Problem statement

The Backup & Restore service deployment fails with the following error in the IBM Storage Fusion user interface:

    Install Error: Invalid Input: Both the api Server and bootstrapToken fields need to be populated

Resolution

As a resolution, try with a private browser window.

Backup & Restore hub service in a custom namespace
--------------------------------------------------

Problem statement

In general, the Backup & Restore service is installed or upgraded to the default `ibm-backup-restore` namespace. To avoid issues during the installation or upgrade of the service with custom namespace, do the resolution steps.

Resolution

1.  Go to Operators > Installed Operators > Fusion Service Definition.
2.  Open the `ibm-backup-restore-service` CR and navigate to the YAML tab.
3.  In spec.onboarding.parameters, search for parameter with **name** as namespace and change the defaultValue to the custom-namespace where the service must be installed.
    
    Example:
    
        
        parameters:
              - dataType: string
                name: namespace
                defaultValue: <custom-namespace>
                userInterface: false
                required: true
                descriptionCode: BMYSRV00003
                displayNameCode: BMYSRV00004
    
4.  Store the following YAML in server-fsi.yaml, and replace `<custom-namespace>` with the namespace where the service must be installed.
    
        
        apiVersion: service.isf.ibm.com/v1
        kind: FusionServiceInstance
        metadata:
          name: ibm-backup-restore-service-instance
          namespace: ibm-spectrum-fusion-ns
        spec:
          parameters:
            - name: doInstall
              provided: false
              value: 'true'
            - name: namespace
              provided: false
              value: <custom-namespace>
            - name: storageClass
              provided: true
              value: lvms-lmvg
          serviceDefinition: ibm-backup-restore-service
          triggerUpdate: false
          enabled: true
          doInstall: true
    
5.  Run the following command to apply the changes:
    
        oc apply -f server-fsi.yaml 
    
6.  For the upgrade procedure, upgrade the IBM Storage Fusion operator, repeat step [1](#sf_br_installandupgrade_issues__step1) of the resolution, and then upgrade the service.

Pods in Crashloopbackoff state after upgrade
--------------------------------------------

Problem statement

The Backup & Restore service health changes to unknown and two pods go into Crashlookbackoff state.

Resolution

In the resource settings of `guardian-dp-operator` pod that resides in `ibm-backup-restore` namespace, set the value of IBM Storage Fusion operator memory limits to 1000mi.

Example:

    
    resources:    
              limits:    
                cpu: 1000m    
                memory: 1000Mi    
              requests:    
                cpu: 500m    
                memory: 250Mi 
    

storage-operator pod crashes with `CrashLoopBackOff` status
-----------------------------------------------------------

Problems statement

The storage-operator pod crashes due to OOM error.

Resolution

To resolve the error, increase the memory limit from 300Mi to 500Mi.

1.  Log in to the Red Hat® OpenShift® web console as an administrator.
2.  Select the project `ibm-spectrum-fusion-ns`.
3.  Select the `isf-storage-operator-controller-manager-xxxx` pod, and go to the YAML tab.
4.  In the YAML, change the memory limit for `isf-storage-operator-controller-manager-xxxx` container from 300Mi to 500Mi.
    
        
        containers:
            - resources:
                limits:
                  cpu: 100m
                  memory: 500Mi
    
5.  Click Save.

Backup & Restore service goes into unknown state
------------------------------------------------

Problem statement

This Backup & Restore service might goes into unknown state when you upgrade IBM Storage Fusion from 2.6.x to 2.8.0.

Resolution

It automatically shows healthy on the IBM Storage Fusion user interface after you upgrade the Backup & Restore service.

MongoDB pod crashes with `CrashLoopBackOff` status
--------------------------------------------------

Problem statement

The MongoDB pod crashes due to OOM error.

Resolution

To resolve the error, increase the memory limit from 256Mi to 512Mi. Do the following steps to change the memory limit:

1.  Log in to the Red Hat OpenShift web console as an administrator.
2.  Go to Workloads > StatefulSet.
3.  Select the project `ibm-backup-restore`.
4.  Select the MongoDB pod, and go to the YAML tab.
5.  In the YAML, change the memory limit for MongoDB container from 256Mi to 512Mi.
6.  Click Save.

Backup & Restore stuck at 5% during upgrade
-------------------------------------------

Diagnosis

1.  In OpenShift user interface, go to the Installed Operator and filter on `ibm-backup-restore` namespace.
2.  Click `IBM Storage Fusion Backup and Restore Server`.
3.  Go to Subscriptions.
4.  If you see the following errors, then do the workaround steps to resolve the issue:
    
    "error validating existing CRs against new CRD's schema for "guardiancopyrestores.guardian.isf.ibm.com": error validating custom resource against new schema for GuardianCopyRestore"
    

Resolution

1.  From command line or command prompt, log in to the cluster and run the following commands to delete the `guardiancopyrestore` CRs and 2.6.0 cs:
    
        oc -n ibm-backup-restore delete guardiancopyrestore.guardian.isf.ibm.com --all
        oc -n ibm-backup-restore delete csv guardian-dm-operator.v2.6.0 
        
    
2.  From OpenShift Container Platform console, go to Installed Operator and filter on `ibm-backup-restore` namespace.
3.  Click `IBM Storage Fusion Backup and Restore Server`.
4.  Go to Subscriptions.
5.  Find the failing `installplan` and delete it.
6.  Go to Installed Operator and go to `ibm-backup-restore` namespace.
7.  Find `IBM Storage Fusion Backup and Restore Server` and click Upgrade available and approve the Install plan for `IBM Storage Fusion Backup and Restore Server`.
8.  Wait for the service upgrade to resume.

Backup & Restore service installation gets stuck after upgrade to IBM Storage Fusion 2.7.0
------------------------------------------------------------------------------------------

Problem statement

If the installation or upgrade of the Backup & Restore service does not reach 100% completion, then check for failed startup probes. Run the following command to look for any pods that are not in the READY state:

    oc get pods -n ibm-backup-restore

Example output:

NAME                                                              READY   STATUS      RESTARTS        AGE
applicationsvc-855746ffbf-2pmz7                                   0/1     Running     5 (2m8s ago)    17m
...
job-manager-845dc56b8d-r5w6j                                      0/1     Running     5 (2m50s ago)   17m
...

1.  Go to OpenShift Container Platform.
2.  Go to the Installed Operators view with the selected `ibm-backup-restore` project or namespace.
3.  Click IBM Storage Fusion Backup & Restore Server.
4.  Select the Data Protection Server tab and click the instance.
5.  Select the YAML tab and update the following:
    
        triggerUpgrade: false
    
    to
    
        triggerUpgrade: true
    
6.  Save and reload the YAML.
    
    After some time, all of the pods must be READY and the Backup & Restore service must show healthy in IBM Storage Fusion user interface.
    

Diagnosis

If found, then describe each of the pods and look at the event section to determine if there is a startup probe failure.

Example output showing probe failure:

Events:
  Type     Reason          Age                    From               Message
  ----     ------          ----                   ----               -------
  ... 
  Warning  Unhealthy       3m40s (x5 over 6m40s)  kubelet            Startup probe failed: HTTP probe failed with statuscode: 503

If one or more pods show startup probe failures, then patch the probes to provide additional start time. Update the following script and run to patch the probes. To determine the respective deployment names, update the DEPLOYMENT\_NAMES variable with the list of deployments that have failing startup probes and run the script.

    oc get deployment -n ibm-backup-restore

Example output:

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
...
applicationsvc                                0/1     1            1           12h
...
job-manager                                   0/1     1            1           12h
...

Now, check whether the pods are online. If the startup probes continue to fail, then increase the STARTUP\_DELAY\_SEC parameter and try again.

If it is an upgrade and the pods come online, but the progress does not reach 100% in the IBM Storage Fusion user interface even after 30 minutes, then you must update the Backup & Restore Server CR to re-trigger the upgrade.

Resolution

Update the Backup & Restore Server CR to re-trigger the upgrade

Backup & Restore service install and upgrade issue with the IBM Storage Protect Plus catalog source
---------------------------------------------------------------------------------------------------

Problem statement

The backups can fail with a `FailedValidation` error after you install or upgrade the IBM Storage Fusion and Backup & Restore service to 2.8.0.

validationErrors': \['an existing backup storage location wasn't specified at backup creation time and the default guardian-minio wasn't found.

Error: BackupStorageLocation.velero.io "guardian-minio" not found'

It occurs when the IBM Storage Protect Plus catalog exists in the OpenShift Container Platform clusters.

Resolution

The workaround is to upgrade the OADP, installed in the `ibm-backup-restore` namespace, to version 1.3.

**Parent topic:** [Troubleshooting installation and upgrade issues in IBM Storage Fusion services](../tshooting/sds_services_troubleshooting.html "Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.")

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")



Common service installation issues
==================================

Use this troubleshooting information to resolve common install and upgrade problems related to IBM Storage Fusion services.

ImagePull failure during installation or upgrade of any service
---------------------------------------------------------------

If an `ImagePull` failure occurs during the installation or upgrade of any service, then restart the pod and retry. If the issue persists, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.").

Rook-cephfs pods are in `CrashLoopBackOff` off state
----------------------------------------------------

Problem statement

Data Cataloging and Backup & Restore services that are installed, go into degraded state, and `rook-ceph-mds-ocs-storagecluster` pods are in `CrashLoopBackOff` state.

Resolution

Follow the steps to resolve the issue:

1.  If IBM Storage Fusion is installed with OpenShift® Container Platform or Data Foundation v4.10.x, then you can upgrade it to v4.11.x.
2.  The upgrade of OpenShift Container Platformor Data Foundation from v4.10.x to v4.11.x resolves the `CrashLoopBackOff` error in `rook-ceph-mds-ocs-storagecluster` pods and get it to running state. Eventually services also go to healthy state.
    
    Note: Also, you can upgrade OpenShift Container Platform or Data Foundation to further versions supported by IBM Storage Fusion.
    

Troubleshooting common installation issues
------------------------------------------

*   If you find an error saying `Configmap fusionplatform not found in fusion namespace` in the preparer operator logs, then the error does not have any impact and can be ignored from the `isf-prereq-operator-controller-manager-xxxx` pod logs
*   Problem statement
    
    Whenever the upgrade button is unavailable for any service
    
    Resolution
    
    During the offline upgrade, if you do not see the upgrade button for any of the services after upgrading IBM Storage Fusion operator, then check the `catalogsource` pod, and it should be in a running state. For any `imagepullbackoff` error, ensure you have completed mirroring and updated `imagecontentsourcepolicy`.
    

**Parent topic:** [Troubleshooting installation and upgrade issues in IBM Storage Fusion services](../tshooting/sds_services_troubleshooting.html "Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.")


Compute events and error codes
==============================

List of all the events that you might encounter in compute nodes.

For Call Home events, see [Compute Call Home events and error codes](../errorcodes/sf_compute_callhomevents.html "List of Call Home events that you might encounter in the compute.").

For the events that you might encounter during a compute node firmware upgrade, see [Upgrade events and error codes](sf_upgrade_error_codes.html "List of all the events that you might encounter during upgrade.").

For Informational, Warning, and Critical events, see the following:

*   **[Compute Call Home events and error codes](../errorcodes/sf_compute_callhomevents.html)**  
    List of Call Home events that you might encounter in the compute.
*   **[BMYCO0001](../errorcodes/BMYCO0001.html)**  
    Unable to login, unable to ping, invalid credentials.
*   **[BMYCO0004](../errorcodes/BMYCO0004.html)**  
    Failed to get hardware monitoring data for the node.
*   **[BMYCO0005](../errorcodes/BMYCO0005.html)**  
    The system board hardware is replaced on the existing node located at RU.
*   **[BMYCO0007](../errorcodes/BMYCO0007.html)**  
    Failed to set and update Processors.
*   **[BMYCO0008](../errorcodes/BMYCO0008.html)**  
    Default password successfully reset to a complex password upon first login.
*   **[BMYCO0010](../errorcodes/BMYCO0010.html)**  
    Firmware upgrade success.
*   **[BMYCO0011](../errorcodes/BMYCO0011.html)**  
    Unable to put the node into maintenance mode as scale cluster status is not healthy.
*   **[BMYCO0014](../errorcodes/BMYCO0014.html)**  
    IBM Storage Fusion software error recovery.
*   **[BMYCO0015](../errorcodes/BMYCO0015.html)**  
    Hardware validation error.
*   **[BMYCO0016](../errorcodes/BMYCO0016.html)**  
    Informational event for updating user about upsize success.
*   **[BMYCO0017](../errorcodes/BMYCO0017.html)**  
    Discovered a new node at RU, link-locations and rack serials or failed to get the location of a new node from rackinfo configmap.
*   **[BMYCO0019](../errorcodes/BMYCO0019.html)**  
    Power operation (on, off) does not succeed within the stipulated time of 75 seconds.
*   **[BMYCO0020](../errorcodes/BMYCO0020.html)**  
    Successfully performed power operation \[Gracefulshutdown/GracefulRestart\] on the node.
*   **[BMYCO0021](../errorcodes/BMYCO0021.html)**  
    Successfully performed power operation \[On\] on the node.
*   **[BMYCO0210](../errorcodes/BMYCO0210.html)**  
    Power off warning about successful power operation execution on the node.
*   **[BMYCO0211](../errorcodes/BMYCO0211.html)**  
    Power on information about the successful power operation execution on the node.
*   **[BMYXC0021](../errorcodes/BMYXC0021.html)**  
    OS timeout value exceeded.
*   **[BMYXC0022](../errorcodes/BMYXC0022.html)**  
    Refer to error conditions for a specific condition.
*   **[BMYXC0023](../errorcodes/BMYXC0023.html)**  
    Power off.
*   **[BMYXC0024](../errorcodes/BMYXC0024.html)**  
    Power on.
*   **[BMYXC0025](../errorcodes/BMYXC0025.html)**  
    System boot failure.
*   **[BMYXC0026](../errorcodes/BMYXC0026.html)**  
    OS loader timeout.
*   **[BMYXC0027](../errorcodes/BMYXC0027.html)**  
    Predictive Failure Analysis (PFA) information.
*   **[BMYXC0030](../errorcodes/BMYXC0030.html)**  
    Event remote login.
*   **[BMYXC0035](../errorcodes/BMYXC0035.html)**  
    System log 75% full.
*   **[BMYXC0037](../errorcodes/BMYXC0037.html)**  
    Network change notification.
*   **[BMYXC0038](../errorcodes/bmyxc0038.html)**  
    Host NIC link state down or up.
*   **[BMYXC0039](../errorcodes/bmyxc0039.html)**  
    Drive Hotplug.
*   **[BMYXC0060](../errorcodes/BMYXC0060.html)**  
    The component mentioned in the error message is in degraded state.
*   **[BMYXC0164](../errorcodes/BMYXC0164.html)**  
    Power error.
*   **[BMYXC0165](../errorcodes/BMYXC0165.html)**  
    Fan error.
*   **[BMYXC0255](../errorcodes/BMYXC0255.html)**  
    IP address is active.
*   **[BMYXC9999](../errorcodes/BMYXC9999.html)**  
    Trap server events raised for the hardware failures.
*   **[BMYFW0001E](../errorcodes/BMYFW0001.html)**  
    Failed to put a node into maintenance mode for the firmware update.
*   **[BMYFW0002E](../errorcodes/BMYFW0002.html)**  
    Failed to move a node out of maintenance mode after the firmware upgrade.
*   **[BMYFW0012E](../errorcodes/BMYFW0012.html)**  
    Firmware update operation timed out on the compute node.
*   **[BMYFW0013E](../errorcodes/BMYFW0013.html)**  
    Storage cluster does not turn healthy after a node firmware upgrade.
*   **[BMYFW0032E](../errorcodes/BMYFW0032.html)**  
    Failed to move a node into maintenance mode as the storage cluster status is unhealthy.
*   **[BMYFW0036E](../errorcodes/BMYFW0036.html)**  
    Unable to move a node to a maintenance mode due to an unsafe IBM Storage Scale core pod eviction.
*   **[BMYFW0037E](../errorcodes/BMYFW0037.html)**  
    Failed to move a node into a maintenance mode because an MCO rollout is in progress on the cluster.
*   **[BMYFW0041E](../errorcodes/BMYFW0041.html)**  
    Unable to move a node into maintenance mode. Failed to evict the workload pods from a node.
*   **[BMYPC5030](../errorcodes/BMYPC5030.html)**  
    The node is degraded due to an active event.
*   **[BMYPC6030](../errorcodes/BMYPC6030.html)**  
    Node hardware is not reachable.
*   **[BMYPC6060](../errorcodes/BMYPC6060.html)**  
    The node is in a critical state due to an active event.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")



Node issues
===========

List of all troubleshooting and known issues while you work with the compute nodes.

Known issues

The following known issues exist while you work with the compute nodes. The workarounds are included wherever possible. If you come across an issue that cannot be solved by using these instructions, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

*   [Compute node moved to failed state](#concept_m31_hkm_krb__computenodefailedstate)
*   [Management switch high availability is lost](#concept_m31_hkm_krb__mgmtswitchlost)
*   [Node goes into inspection error and the node is not visible](#concept_m31_hkm_krb__metalrestart)
*   [Firmware version for AFM nodes is missing on the IBM Storage Fusion nodes user interface page.](#concept_m31_hkm_krb__section_sw4_tnp_pbc)

Troubleshooting issues

The following troubleshooting issues exist while you work with the compute nodes with workarounds included wherever possible. If you come across an issue that cannot be solved by using these instructions, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

*   [mmaddnode command fails during node upsize on storage cluster](#concept_m31_hkm_krb__mmaddnode)
*   [NVMe drives missing on XCC but seen on CoreOS](#concept_m31_hkm_krb__nvme)
*   [Issues in node restart](#concept_m31_hkm_krb__nodereboot)
*   [GPU node capacity value shows 0](#concept_m31_hkm_krb__section_ww2_41r_byb)
*   [Compute nodes in a hung state](#concept_m31_hkm_krb__section_yby_kyq_byb)
*   [Pod migration gets stuck in the Terminating or ContainerCreating state](#concept_m31_hkm_krb__section_xrg_lyq_byb)
*   [Compute node can abruptly become unreachable](#concept_m31_hkm_krb__section_lfq_lyq_byb)
*   [Add rack workflow from base rack fails in node validation](#concept_m31_hkm_krb__section_pym_vrz_lyb)
*   [OpenShift Container Platform node status shows NotReady but IBM Storage Fusion HCI System user interface node status shows as Running](#concept_m31_hkm_krb__section_wwz_klf_yyb)
*   [Node goes missing from the IBM Storage Fusion HCI System rack graphical view](#concept_m31_hkm_krb__section_bvn_rx3_tzb)

mmaddnode command fails during node upsize on storage cluster
-------------------------------------------------------------

Resolution

After the initial failure of the mmaddnode command during node addition, run the following command to remove the files that do not get cleaned up properly:

    
    oc exec -it compute-1-ru13 -c config -- /bin/sh
    sh-5.1# rm /var/mmfs/gen/mmsdrfs
    

NVMe drives missing on XCC but seen on CoreOS
---------------------------------------------

Resolution

1.  Run resetsp to restart the BMC.
2.  If the step 1 does not resolve the issue, run OS reboot command to reboot the node.
3.  If both step 1 and step 2 do not resolve the issue, then contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") to replace the faulty drive or faulty NVMe backplane.

Issues in node restart
----------------------

Problem statement

Sometimes, after you restart a node, it may not change its state to ready and the `ovs-configuration.service` may fail.

Resolution

As a resolution, restart the node. For more information about Scale behavior during node restarts, see [Scale behavior during node restarts](sf_hci_common_issues.html#sf_hci_common_issues__pdbscalebehavior).

Compute node moved to failed state
----------------------------------

If the nodes display the following error message after power operation, contact IBM Support:

Failed to get storage information from the IMM of the node

Management switch high availability is lost
-------------------------------------------

When the management switch high availability is lost due to the management switch at RU18 being down, the cluster continues to function, but autodiscovery of nodes is not supported.

Node goes into inspection error and the node is not visible
-----------------------------------------------------------

This issue is observed with OpenShift® Container Platform 4.12. As a resolution, run the following command to restart the metal3 pods:

    oc -n openshift-machine-api get po |grep metal3|grep -v metal3-image|awk '{print $1}'|xargs oc -n openshift-machine-api delete po

Firmware version for AFM nodes is missing on the IBM Storage Fusion nodes user interface page.
----------------------------------------------------------------------------------------------

Important: This issue is applicable only for the IBM Storage Fusion 2.8.0.

Firmware version column on the Infrastructure > Nodes page, the firmware version details shows as blank for AFM nodes. It can be ignored as there is no firmware upgrade required in the IBM Storage Fusion 2.8.0 release.

GPU node capacity value shows 0
-------------------------------

Problem statement

It is observed that sometimes users might see GPU node capacity as 0 and also that the NFD discovery instance has the `nfd-worker-xxx` in a constant crashloopback.

Resolution

This issue was observed in OpenShift Container Platform release 4.10.21. Upgrade the OpenShift Container Platform to 4.10.61, reinstall the NFD operator with the NFD Discovery CR, and install the NVIDIA operator and create the cluster policy that detects the GPUs.

Compute nodes in a hung state
-----------------------------

Resolution

Steps to restart when the compute nodes get into a hung state:

As a prerequisite, you must have administrator access to Red Hat® OpenShift and OC (Red Hat OpenShift CLI) command access to the cluster.

1.  Log in to OpenShift UI.
2.  Go to Compute > Bare Metal Hosts.
3.  Make sure to select `openshift-machine-api` project on the Bare Metal Hosts page.
4.  From the ellipsis menu, click Power Off for the node.
5.  Power it on again and check whether the issue is resolved.

Pod migration gets stuck in the Terminating or ContainerCreating state
----------------------------------------------------------------------

Problem statement

A controller node goes down and the migration of the pod gets stuck in Terminating or ContainerCreating state.

Resolution

As a workaround, delete the container creating pod so that it can get scheduled in another available controller node.

Compute node can abruptly become unreachable
--------------------------------------------

Problem statement

A compute node can abruptly become unreachable either due to a power outage or network disruption.

Cause

The pods that run on that specific compute node might have not got evacuated cleanly and they remain stuck in `Terminating` or `Unknown` state.

Resolution

To ensure a complete evacuation and migration of the pod that is stuck on the failed node, force delete the pod manually by using the following command:

    oc delete pod <pod name> --grace-period=0 --force --namespace <namespace>

Add rack workflow from base rack fails in node validation
---------------------------------------------------------

Problem statement

Add rack workflow from base rack fails in node validation for the nodes of auxiliary rack in case of high availability cluster.

Cause

The issue might occur because monitoring failed for the nodes of the auxiliary rack.

Resolution

1.  Before performing the add rack operation, ensure that all the nodes on the auxiliary rack are healthy and that there are no warnings, critical events or any component failures on any of the nodes.
2.  Restart the BMC on the failing nodes to clear the events on the compute node.

OpenShift Container Platform node status shows NotReady but IBM Storage Fusion HCI System user interface node status shows as Running
-------------------------------------------------------------------------------------------------------------------------------------

Problem statement

The Running state of the node on the IBM Storage Fusion HCI System user interface indicates that the Baremetal node is healthy and running. Whenever you see IBM Storage Fusion HCI System node status is Running and OpenShift Container Platform node status is Not ready, then follow the steps in the resolution.

Resolution

1.  Log in to OpenShift user interface.
2.  Go to Compute > Bare Metal.
3.  From the ellipsis menu, click Power Off for the node.
4.  Power it on again and check if the issue is resolved.

Node goes missing from the IBM Storage Fusion HCI System rack graphical view
----------------------------------------------------------------------------

Problem statement

OpenShift Container Platform node status shows `Ready` even though the IBM Storage Fusion HCI System user interface node status shows the node state as `Error connecting to node`

For more information about node status, see [Monitoring hardware from IBM Storage Fusion HCI System user interface](../compute/sf_hardwaremonitoring_overview.html "The IBM Storage Fusion HCI System user interface provides an enhanced hardware monitoring experience. You can quickly determine a hardware component in the IBM Storage Fusion HCI System that needs your attention.").

This issue might occur due to the power or network issues.

Resolution

The node can be viewed graphically in the user interface of the IBM Storage Fusion HCI System rack after it gets into a healthy state. For more information, see [Monitoring hardware from IBM Storage Fusion HCI System user interface](../compute/sf_hardwaremonitoring_overview.html "The IBM Storage Fusion HCI System user interface provides an enhanced hardware monitoring experience. You can quickly determine a hardware component in the IBM Storage Fusion HCI System that needs your attention.").

1.  Get the name of the compute-operator pod name:
    
        oc get po -n ibm-spectrum-fusion-ns | grep isf-compute-operator
        
    
2.  Copy the OneCli folder from the operator pod:
    
        oc rsync -c manager isf-compute-operator-controller-manager-<pod name>:/fw/onecli
    
    Note: For the subsequent commands, create a separate session.
    
3.  Start a debug pod on the target node (BMC has error connecting to the node) that is in Ready state.
    
        oc debug node/<nodename>
    
4.  Get the debug pod name:
    
        oc get po -A -o wide | grep <nodename> | grep debug
    
5.  Copy the local copy of the OneCli folder to debug pod.
    
        oc rsync onecli pod/<debug pod>:/tmp
    
6.  From the other session, run the following command to restart the node:
    
        
        cd /tmp/onecli
        ./onecli misc rebootbmc
    
7.  Go to the Infrastructure > Nodes and wait for the following error message to go away:
    
    Error connecting to node
    
    It takes around three minutes for the error to disappear from the page.
    

*   **[Add node failure](../tshooting/sf_add_node_failure.html)**  
    Node addition to the Red Hat OpenShift Container Platform cluster fails.
*   **[Cleaning up localhost node from the IBM Storage Fusion HCI System cluster](../tshooting/cleaningup_localhostnode.html)**  
    Follow the instructions to clean up the localhost node from the IBM Storage Fusion HCI System cluster.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")



Contacting IBM Support Center
=============================

The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.

Before you begin
----------------

Points to note when you contact the IBM Support Center:

1.  For IBM Storage Fusion HCI System, ensure that you have your system’s serial number and its associated account number.For IBM Storage Fusion, a serial number is not required.
2.  Based on your issue, collect appropriate log files and diagnostic data.
3.  IBM Support provides a time period within which IBM representative returns your call. Be sure that the person that you identified as your contact can be reached in the phone number.
4.  An online case gets created to track the problem you are reporting. Note down the number for future reference and communication. If you need to make subsequent calls to discuss the problem, use the number to identify the problem.

About this task
---------------

If you enabled Call Home, then it connects your system to service representatives who can monitor issues and respond to problems efficiently and quickly to keep your system up and running. Enabling Call Home significantly improves your IBM Support experience. For more information about Call Home and Service level in Call Home, see [Enabling IBM Call Home for IBM Storage Fusion HCI System](../serviceability/sf_callhome.html "Steps to enable and disable Call Home from the IBM Storage Fusion HCI user interface. In addition, you can edit Call Home configuration details.")[Enabling Call Home](../serviceability/sf_sds_callhome.html "Steps to enable and disable Call Home from the user interface. In addition, you can edit Call Home configuration details.").

If you havean IBM Hardware and Software Maintenance service contract, then use this procedure to open a case on IBM Support site.

The following table lists alternative ways of contacting IBM Support Center:

Your location

Method of contacting the IBM Support Center

In the United States

Call **1-800-IBM-SERV** for support.

Outside the United States

Contact your local IBM Support Center or see the [Directory of worldwide contacts (www.ibm.com/planetwide).](http://www.ibm.com/planetwide "(Opens in a new tab or window)")

If you do not have an IBM Software Maintenance service contract, then contact your IBM sales representative to find out how to proceed. For non-IBM failures, follow the problem-reporting procedures provided with that product.

Procedure
---------

1.  Log in to [IBM support page](https://www.ibm.com/mysupport "(Opens in a new tab or window)") by using your IBMid and click Continue.
2.  In Let's troubleshoot, click Open a case tile.
3.  In the Open a case page, enter the following details:
    
    1.  Select a value for the Type of support that you need.
    2.  Enter the Case title to describe your case. The maximum length of the case title is 255 characters.
    3.  Enter IBM in Product manufacturer.
    4.  Enter Storage Fusion or Storage Fusion HCI Physical Appliance for Product.
    5.  For Storage Fusion HCI Physical Appliance, enter the Machine serial number.
    6.  Enter Product version.
    7.  Select Service Type.
    8.  Check Address where product is located.
    9.  Select Severity.
    10.  For Storage Fusion HCI Physical Appliance, enter Machine Type Model.
        
        Note: For Storage Fusion HCI Physical Appliance the machine models are currently: 9155-C00, 9155-C01, 9155-C04, 9155-C05, 9155-F01, 9155-G01, 9155-S01, 9155-S02, 9155-R42, 9155-TF5.
        
    11.  Enter System down (yes or no).
    12.  Enter Failing Component (Software or Hardware)
    13.  Enter Account name and Case description.
    14.  Enter Case contact phone number.
    15.  You can Add team members based on the need.
    
    Note: If a diagnostic file is required for this case, you can upload it after you submit this case.
    
4.  Go through the terms and conditions and click Submit case.

**Parent topic:** [Troubleshooting](../tshooting/sf_troubleshooting.html "The steps to troubleshoot an issue vary depending on the problem. To help make relevant information available to you as quickly as possible, the log files and other tools for troubleshooting problems are consolidated.")



Data Cataloging service issues
==============================

Use this troubleshooting information to resolve install and upgrade problems that are related to the Data Cataloging service.

Data cataloging upgrade stuck for more than 1 hour
--------------------------------------------------

Problem statement

Data Cataloging upgrade is stuck for more than 1 hour and the progress shows 15%.

Symptoms

The Data Cataloging upgrade runs for more than 1 hour and gets stuck at 15%. On the OpenShift® Console, the installed operator shows just the Data Cataloging operator and not `IBM DB2` and `AMQ-streams` operators under the `ibm-data-cataloging` namespace.

Also, the Subscription for the Data Cataloging operator shows InstallPlanPending, Reason "RequiresApproval.

Resolution

1.  In the OpenShift console, go to Operators > Installed Operators tab.
2.  Select IBM Storage Discover operator.
3.  Go to Subscriptions tab, and select InstallPlan.
4.  Approve the InstallPlan.
    
    There is an alternative method using OpenShift CLI as follows:
    
    1.  Run the following command to get the InstallPlan that is not approved.
        
            oc get ip -n ibm-data-cataloging -o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}'
        
    2.  Run the following command to get the InstallPlan name.
        
            oc patch installplan $(oc get ip -n ibm-data-cataloging -o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}') -n ibm-data-cataloging --type merge --patch '{"spec":{"approved":true}}'
        
    
5.  Wait for the upgrade process to complete.

Data Cataloging import service pod in CrashLoopBackOff state
------------------------------------------------------------

Diagnosis

1.  Check for CrashLoopBackOff on the import service pod.
    
        oc -n ibm-data-cataloging get pod -l role=import-service
        
    
2.  Confirm that the logs show permission denied error:
    
        oc -n ibm-data-cataloging logs -l role=import-service
    

Resolution

1.  Debug import service pod.
    
        oc -n ibm-data-cataloging debug deployment/isd-import-service --as-user=0
    
2.  Update the directory permissions.
    
        chmod 775 /uploads
        mkdir -p /uploads/failed_requests
        chmod 775 /uploads/failed_requests
        exit
    

Data Cataloging database schema job is not in a completed state during installation or upgrade
----------------------------------------------------------------------------------------------

Note: This procedure is applicable to recover DB2 that goes into an unavailable mode after the service installation and related to a degraded state of the Data Cataloging service.

Symptoms

The `isd-db2whrest` or `isd-db-schema` pods report a not ready or error state.

Run the following command to view the common logs:

    oc -n ibm-data-cataloging logs -l 'role in (db2whrest, db-schema)' --tail=200
    

Go through the logs to check whether the following error exists:

    Waiting on c-isd-db2u-engn-svc port 50001...
    
    db2whconn - ERROR - [FAILED]: [IBM][CLI Driver] SQL1224N The database manager is not able to accept new requests, has terminated all requests in progress, or has terminated the specified request because of an error or a forced interrupt. SQLSTATE=55032
    
    Connection refused
    

Resolution

1.  Restart Db2:
    
        
        oc -n ibm-data-cataloging rsh c-isd-db2u-0
        sudo wvcli system disable -m "Disable HA before Db2 maintenance"
        su - ${DB2INSTANCE}
        db2stop
        db2start
        db2 activate db BLUDB
        exit
        sudo wvcli system enable -m "Enable HA after Db2 maintenance"
    
2.  Confirm that Db2 HA-monitoring is active:
    
        
        sudo wvcli system status
        exit
        
    
3.  Check whether the problem occurred during installation or upgrade or a post installation problem.
4.  If this occurs during an upgrade or installation, recreate the `isd-db-schema` job and monitor the pod until it gets to a completed state.
    
        SCHEMA_OLD="isd-db-schema-old.json"
        SCHEMA_NEW="isd-db-schema-new.json"
        oc -n ibm-data-cataloging get job isd-db-schema -o json > $SCHEMA_OLD
        jq 'del(.spec.template.metadata.labels."controller-uid") | del(.spec.selector) | del (.status)' $SCHEMA_OLD > $SCHEMA_NEW
        oc -n ibm-data-cataloging delete job isd-db-schema
        oc -n ibm-data-cataloging apply -f $SCHEMA_NEW
        
    
        oc -n ibm-data-cataloging get pod | grep isd-db-schema 
        
    
5.  If this is a post installation problem, restart db2whrest.
    
        oc -n ibm-data-cataloging delete pod -l role=db2whrest
        
    

Data Cataloging installation is stuck for more than 1 hour
----------------------------------------------------------

Note: This procedure must be used only during installation problems, not for upgrades or any subsequent issues.

Symptoms

The Data Cataloging installation running for more than an hour, and it stuck somewhere between 35% and 80% (both inclusive).

Resolution

1.  Run the following command to scale down the operator.
    
        oc -n ibm-data-cataloging scale --replicas=0 deployment/spectrum-discover-operator
        
    
2.  Run the following command to scale down workloads.
    
        oc -n ibm-data-cataloging scale --replicas=0 deployment,statefulset -l component=discover
        
    
3.  Run the following command to remove the DB schema job if present.
    
        oc -n ibm-data-cataloging delete job isd-db-schema --ignore-not-found
        
    
4.  Run the following commands to delete the Db2 instance and the password secret.
    
        oc -n ibm-data-cataloging delete db2u isd
        oc -n ibm-data-cataloging delete secret c-isd-ldapblueadminpassword --ignore-not-found
    
5.  Wait until the Db2 pods and persistent volume claims get removed.
    
        oc -n ibm-data-cataloging get pod,pvc -o name | grep c-isd
        
    
6.  Run the following command to scale up the operator.
    
        oc -n ibm-data-cataloging scale --replicas=1 deployment/spectrum-discover-operator
        
    

Data Cataloging service is not installed successfully on IBM Storage Fusion HCI System with GPU nodes
-----------------------------------------------------------------------------------------------------

Problem statement

The data catalog service is in installing state for hours.

Resolution

To resolve the problem, do the following steps:

1.  Patch FSD with new affinity to not schedule `isd` workload on those nodes:
    
        oc -n <Fusion_namespace> patch fusionservicedefinitions.service.isf.ibm.com data-cataloging-service-definition  --patch "$(cat fsd_dcs_patch.yaml)" 
    
    The fsd\_dcs\_patch.yaml file:
    
        
        cat >> fsd_dcs_patch.yaml << EOF
        
        apiVersion: service.isf.ibm.com/v1
        kind: FusionServiceDefinition
        metadata:
          name: data-cataloging-service-definition
          namespace: <Fusion_namespace>
        spec:
          onboarding:
            parameters:
              - dataType: string
                defaultValue: ibm-data-cataloging
                descriptionCode: BMYSRV00003
                displayNameCode: BMYSRV00004
                name: namespace
                required: true
                userInterface: false
              - dataType: storageClass
                defaultValue: ''
                descriptionCode: BMYDC0300
                displayNameCode: BMYDC0301
                name: rwx_storage_class
                required: true
                userInterface: true
              - dataType: bool
                defaultValue: 'true'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: doInstall
                required: true
                userInterface: false
              - dataType: json
                defaultValue: '{"accept": true}'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: license
                required: true
                userInterface: false
              - dataType: json     
                defaultValue: '{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"nvidia.com/gpu","operator":"NotIn","values":["Exists"]}]}]}}}'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: affinity
                required: true
                userInterface: false
            
        EOF
        
    
    If the output shows this error message Error from server (UnsupportedediaType): the body of the request was in an unknown format - accepted media types include: application/json-patch+json, application/merge-patch+j son, application/apply-patch+yaml, then you need to follow the steps to resolve this issue:
    
    1.  Go to OpenShift Container Platform web console.
    2.  Go to Operators > Installed Operators tab, under `<Fusion_namespace>`, select the IBM Storage Fusion operator.
    3.  Select the IBM Storage Fusion service instance tab, and select the `data-cataloging-service-instance`.
    4.  Select the YAML tab, and edit the YAML file for the `data-cataloging-service-instance`. Under `spec.onboarding.parameters`, ensure you add the following lines.
        
            - dataType: json
                       defaultValue: '{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"isf.ibm.com/nodeType","operator":"NotIn","values":["gpu"]}]}]}}}'
                       descriptionCode: descriptionCode
                       displayNameCode: displayNameCode
                       name: affinity
                       required: true
                       userInterface: false
        
    
2.  Display the patch FSD:
    
        
        oc -n <Fusion_namespace> get fusionservicedefinitions.service.isf.ibm.com data-cataloging-service-definition -o yaml
    
3.  Install from the user interface.

Data Cataloging database pods stuck during initialization phase
---------------------------------------------------------------

Symptoms

Multiple Db2 pods use the host port 5002, which can cause pods to stay in the `init` phase.

Resolution

1.  Uninstall the Data Cataloging service. For procedure, see [Uninstalling Data Cataloging](../sdsinstall/sds_uninstall_data_cataloging.html "Steps to uninstall Data Cataloging by using command line interface.").
2.  Create a file with the Data Cataloging `FusionServiceDefinition` patch.
    
        cat >> fsd_dcs_patch.yaml << EOF
        apiVersion: service.isf.ibm.com/v1
        kind: FusionServiceDefinition
        metadata:
          name: data-cataloging-service-definition
          namespace: ibm-spectrum-fusion-ns
        spec:
          onboarding:
            parameters:
              - dataType: string
                defaultValue: ibm-data-cataloging
                descriptionCode: BMYSRV00003
                displayNameCode: BMYSRV00004
                name: namespace
                required: true
                userInterface: false
              - dataType: storageClass
                defaultValue: ''
                descriptionCode: BMYDC0300
                displayNameCode: BMYDC0301
                name: rwx_storage_class
                required: true
                userInterface: true
              - dataType: bool
                defaultValue: 'true'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: doInstall
                required: true
                userInterface: false
              - dataType: json
                defaultValue: '{"accept": true}'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: license
                required: true
                userInterface: false
              - dataType: json
                defaultValue: '{"size":1,"mln":2,"storage":{"activelogs":{"requests":"300Gi"},"data":{"requests":"600Gi"},"meta":{"requests":"100Gi"},"activelogs":{"tempts":"100Gi"}}}'
                descriptionCode: descriptionCode
                displayNameCode: displayNameCode
                name: dbwh
                required: true
                userInterface: false
        EOF
        
    
3.  Apply the patch to downsize the Db2 cluster.
    
        oc -n ibm-spectrum-fusion-ns patch fusionservicedefinitions.service.isf.ibm.com data-cataloging-service-definition --type=merge --patch-file fsd_dcs_patch.yaml
        
    
4.  Install Data Cataloging service from the IBM Storage Fusion user interface. For procedure, see [Data Cataloging](../services/sf_enable_discover.html "Data Cataloging service is a modern metadata management software that provides data insight for exabyte-scale heterogeneous file, object, backup, and archive storage on premises and in the cloud. It can help you manage your unstructured data by reducing the data storage costs, uncovering hidden data value, and reducing the risk of massive data stores.").

**Parent topic:** [Troubleshooting installation and upgrade issues in IBM Storage Fusion services](../tshooting/sds_services_troubleshooting.html "Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.")



**Operator installation or upgrade fails with DeadlineExceeded error**
======================================================================

IBM Cloud Paks foundational services operator ClusterServiceVersion (CSV) status shows Failed and its InstallPlan status shows Failed after the subscription gets created.

Symptom
-------

The foundational services operator CSV status shows as `Failed`. On the Red Hat® OpenShift® Container Platform cluster console, you see an error message similar to the following message:

Bundle unpacking failed. Reason: DeadlineExceeded, and Message: Job was active longer than specified deadline".

The InstallPlan status also shows as `Failed`.

1.  Get InstallPlans that have the `Failed` status.
    
         oc get subscription <failed-operator-subscription> -n <operator-namespace> -o jsonpath='{.status.installPlanRef}'
        
    
    See the following sample output:
    
    {"apiVersion":"operators.coreos.com/v1alpha1","kind":"InstallPlan","name":"install-cpqk2","namespace":"cloudpak-control","resourceVersion":"98650091","uid":"9f0210dd-a44f-454a-8b66-722b7520c838"}
    
2.  Confirm the status of the InstallPlan that you got in the previous command output.
    
         oc get installplan install-cpqk2 -n cloudpak-control -o jsonpath='{.status.installPlanRef}'
        
    
    Following is a sample output:
    
    Failed
    
3.  Inspect the InstallPlan.
    
         oc get installplan install-cpqk2 -n cloudpak-control -o yaml | grep "unpack job not completed"
        
    
    See the following sample output:
    
    bundle unpacking failed. Reason: DeadlineExceeded, and Message: Job was active longer than specified deadline
    

Cause
-----

Operator Lifecycle Manager (OLM) fails to unpack the operator bundle because the extract job failed (probably due to an issue of remote image access, or any other reason) and corrupted the configmap. Therefore, the operator manifest also most likely gets corrupted. When it is corrupted, any repeated installation attempts by using the same job and configmap fail.

In an air-gapped environment, in most cases the issue happens when the bundle image is unavailable in the private registry.

Resolution
----------

For more information about how to resolve the issue, see [Operator installation or upgrade fails with DeadlineExceeded in Red Hat OpenShift Container Platform](https://www.ibm.com/links?url=https%3A%2F%2Faccess.redhat.com%2Fsolutions%2F6459071 "(Opens in a new tab or window)"). Complete the following steps to resolve the issue:

1.  Find the corresponding job and configmap in the namespace where the CatalogSource is deployed. Narrow down your search by using the operator name or any other keyword.
    
         oc get job -n <catalog-namespace> -o json | jq -r '.items[] | select(.spec.template.spec.containers[].env[].value|contains ("<failed-operator-name>")) | .metadata.name'
        
    
2.  Delete the job and the corresponding configmap that you got in the previous command. In most cases, the job and configmap have the same name.
    
         oc delete job <job-name> -n <catalog-namespace>
        
    
         oc delete configmap <job-name> -n <catalog-namespace>
        
    
3.  Delete the `Failed` InstallPlan.
    
         oc delete installplan <operator-installplan-name> -n <operator-namespace>
        
    
4.  Delete the subscription and CSV of the `Failed` operator.
    
        oc delete subscription <name-of-the-operator-subscription> -n <operator-namespace>
        
    
        oc delete csv <name-of-the-corresponding-CSV> -n <operator-namespace>
        
    
5.  Delete the Operand Deployment Lifecycle Manager pod.
    
         oc delete pod -l name=operand-deployment-lifecycle-manager -n <your-foundational-services-namespace>
        
    

**Parent topic:** [Installation and upgrade issues](../tshooting/sf_install_tshooting.html "Use the troubleshooting tips and tricks in IBM Storage Fusion installation.")



Troubleshooting Hosted Control Plane clusters
=============================================

Use these troubleshooting information to know the problem and workaround for the installation of spoke cluster and

Hosted Control Plane in Red Hat documentation
---------------------------------------------

For Hosted Control Plane troubleshooting documentation by Red Hat®, see [Red Hat Documentation](https://docs.openshift.com/container-platform/4.15/hosted_control_planes/hcp-troubleshooting.html "(Opens in a new tab or window)").

Issues in installation of IBM Storage Fusion in a hosted cluster
----------------------------------------------------------------

Resolution

1.  Check the status of `manifestwork` `fusion-install` in `clusternamespace` of the hub cluster:
    
        
        oc login to the hub
        oc get manifestwork fusion-install -n <spoke_cluster_name> -o yaml
    
2.  If no error exists in the status of the `manifestwork`, check the status of IBM Storage Fusion installation in the spoke cluster:
    
        
        oc login to the spoke
        oc get csv -n ibm-spectrum-fusion-ns
    

Issues in the installation of Fusion Data Foundation in the hosted cluster
--------------------------------------------------------------------------

Resolution

As a resolution, check the status of `manifestwork` `odfclient-install` in `clusternamespace` in the hub cluster:

    
    oc login to the hub
    oc get manifestwork odfclient-install -n <spoke_cluster_name> -o yaml

Backup issues in Hosted Control Plane with Fusion Data Foundation
-----------------------------------------------------------------

Problem statement

Concurrent backups fail during a Velero snapshot with the following error message:

    The  operation has timed out because Velero has failed to report status.'
          However, the backup phase is updated as 'DataTransferfailed.

The timeout for Velero to pickup request is set to 30 minutes in the transaction manager.

Resolution

To resolve the issue, increase it to 60 minutes.

Hosted Control Plane cluster does not get created because of unavailability of IP addresses
-------------------------------------------------------------------------------------------

Resolution

Check whether the installed load balancer has sufficient IP addresses for the Hosted Control Plane cluster. Check the `IPAddressPool` object for the IP range. Run the following command to check whether an IP is available:

    oc get svc -A | grep LoadBalancer

Known issues and limitations
----------------------------

*   Random "image pull" failure can occur on the Hosted Control Plane due to secrets.
*   Sometimes, the hosted cluster status goes in offline mode after deletion. Contact IBM Support to resolve the issue.
*   The hcp destroy command can cause the hosted cluster to get stuck indefinitely during cleanup. Contact IBM Support to resolve the issue.
*   The YAML tab view on the OpenShift® Container Platform console is not working as expected during the installation of the multi cluster engine operator.
*   The hosted clusters expect the `storageprofile` of the used storage class, but it is not available during cluster creation.
*   The following issues occur whenever you remove the disks and place them back in the rack:
    *   Disks are not reflected in the node.
    *   LVM cluster and Data Foundation go into a degraded state.As a resolution, restart the affected node.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")



Global Data Platform service issues
===================================

Use this troubleshooting information to resolve install and upgrade problems that are related to Global Data Platform service.

Warning: Do NOT delete IBM Storage Scale pods. Deletion of Scale pods in many circumstances has implications on availability and data integrity.

IBM Storage Scale stuck with error
----------------------------------

Problem statement

If IBM Storage Scale installation stuck with the error ERROR Failed to define vdiskset {"controller": "filesystem", "controllerGroup": "scale.spectrum.ibm.com", "controllerKind": "Filesystem", "Filesystem": {"name":"ibmspectrum-fs","namespace":"ibm-spectrum-scale"}, "namespace": "ibm-spectrum-scale", "name": "ibmspectrum-fs", "reconcileID": "5fcf21a4-b488-44db-8db5-0f3c1ab695cd", "cmd": "/usr/lpp/mmfs/bin/mmvdisk vs define --vs ibmspectrum-fs-0 --rg rg1 --code 4+2P --bs 4M -

Resolution

As a resolution, run the following command to manually create a recovery group.

    mmvdisk recoverygroup create --recovery-group <RG name> --node-class <Node class name>

Upgrade does not complete as node pod is in `ContainerStatusUnknown` state
--------------------------------------------------------------------------

Problem statement

Global data platform upgrade does not complete because a compute node pod is in `ContainerStatusUnknown` state.

Workaround

Delete the pod in `ContainerStatusUnknown` state and restart the compute node.

Pod drain issues in Global data platform upgrade
------------------------------------------------

Diagnosis and resolution

If the upgrade of Global data platform gets struck, then do the following diagnose and resolve:

1.  Go through the logs from the scale operator.
2.  Check whether you observe the following error:
    
    ERROR Drain error when evicting pods
    
3.  If it is a drain error, then check whether the virtual machine's PVC were created ReadWriteOnce.
    
    The PVCs created by using ReadWriteOnce is not shareable or moveable and can cause the draining of the node.
    
    For example:
    
        oc get pods
        
        NAME                              READY        STATUS                 RESTARTS      AGE
        
        compute-0                         2/2         Running                  0             20h
        compute-1                         2/2         Running                  0             24h
        compute-13                        2/2         Running                  0             21h
        compute-14                        2/2         Running                  3             21h
        compute-2                         2/2         Running                  0             21h
        compute-3                         2/2         Running                  0             21h
        control-0                         2/2         Running                  0             23h
        control-1                         2/2         Running                  0             23h
        control-1-1                       2/2         Running                  0             23h
        control1-1-reboot                 0/1         ContainerStatusUnknown   1             23h
        control-2                         2/2         Running                  0             23h
        ibm-spectrum-scale-gui-0          4/4         Running                  0             20h
        ibm-spectrum-scale-gui-1          4/4         Running                  0             23h
        ibm-spectrum-scale-pmcollector-0  2/2         Running                  0             23h
        ibm-spectrum-scale-pmcollector-1  2/2         Running                  0             20h
        
    
4.  If virtual machine's PVC got created ReadWriteOnce, then stop the virtual machine to continue the upgrade of scale pods.
5.  If the upgrade fails further, then check whether the reason can be due to an orphaned `control-1-reboot` container left in the `ibm-spectrum-scale` namespace. Delete the container to resume and complete the installation.

IBM Storage Scale might get stuck during upgrade
------------------------------------------------

Resolution

Do the following steps:

1.  Shut down all applications that use storage.
2.  Scale down the IBM Storage Scale operator. Make the replica as 0 for deployment `ibm-spectrum-scale-controller-manager`.
    1.  Use `ibm-spectrum-scale-operator` project:
        
            oc project ibm-spectrum-scale-operator
        
    2.  Scale down the IBM Storage Scale operator.
        
            oc scale --replicas=0 deployment ibm-spectrum-scale-controller-manager -n ibm-spectrum-scale-operator
        
    3.  Run the following command to confirm resources are not found in `ibm-spectrum-scale-operator` namespace.
        
            oc get po
        
3.  Log in to the one of the IBM Storage Scale core pods and shut down the server.
    1.  Log in to the server:
        
            oc rsh compute-0
        
    2.  Shut down the server:
        
            mmshutdown -a
        
        Example output:
        
        Thu May 19 07:46:47 UTC 2022: mmshutdown: Starting force unmount of GPFS file systems
        Thu May 19 07:46:52 UTC 2022: mmshutdown: Shutting down GPFS daemons
        compute-13.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        compute-0.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        control-1.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        control-0.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        compute-1.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        compute-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        control-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Shutting down!
        compute-13.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  'shutdown' command about to kill process 1364
        compute-13.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading modules from /lib/modules/4.18.0-305.19.1.el8\_4.x86\_64/extra
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  'shutdown' command about to kill process 1276
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading modules from /lib/modules/4.18.0-305.19.1.el8\_4.x86\_64/extra
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  mmfsenv: Module mmfslinux is still in use.
        compute-13.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  mmfsenv: Module mmfslinux is still in use.
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfs26
        compute-13.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfs26
        compute-14.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unmount all GPFS file syste
        ..
        compute-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfs26
        control-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfs26
        control-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfslinux
        compute-0.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfslinux
        compute-2.ibm-spectrum-scale-core.ibm-spectrum-scale.svc.cluster.local:  Unloading module mmfslinux
        Thu May 19 07:47:03 UTC 2022: mmshutdown: Finished
        
4.  Check whether the states are down:
    
        mmgetstate -a
    
    Sample output:
    
    Node number  Node name          GPFS state  
    ---------------------------------------------
               1  control-0-daemon   down
               2  control-1-daemon   down
               3  control-2-daemon   down
               4  compute-14-daemon  down
               5  compute-13-daemon  down
               6  compute-0-daemon   down
               7  compute-1-daemon   down
               8  compute-2-daemon   down
    
5.  Delete all the pods in `ibm-spectrum-scale` namespace:
    
        oc delete pods --all -n ibm-spectrum-scale
    
6.  To verify, get the list of pods:
    
        oc get po -n ibm-spectrum-scale
    
    Sample output:
    
    NAME                              READY  STATUS   RESTARTS  AGE
     ibm-spectrum-scale-gui-0          0/4    Pending  0         7s
     ibm-spectrum-scale-pmcollector-0  0/2    Pending  0         7s
    
    Note: Only IBM Storage Scale pods get deleted and not GUI and `pmcollector`.
    

IBM Storage Scale upgrade gets stuck
------------------------------------

Cause

The IBM Storage Scale upgrade gets stuck because of `isf-storage-service`. It happens whenever the `scaleoutstatus` or `scaleupStatus` is **In Progress** in the Scale CR.

Important: Do not update the latest image for the `isf-storage-service` when a IBM Storage Scale operation is in progress.

Workaround

Steps to upgrade the IBM Storage Scale on the rack:

1.  Go to the `isf-storage-service` in `ibm-spectrum-fusion-ns` project and change the image to `cp.icr.io/cp/isf/isf-storage-services:2.2.0-latest` or run the following OC command.
    
        oc patch deploy isf-storage-service-dep -n ibm-spectrum-fusion-ns --patch='{"spec":{"template":{"spec":{"containers":[{"name": "app", "image":"cp.icr.io/cp/isf/isf-storage-services:2.2.0-latest"}]}}}}
    
2.  After the `isf-storage-service` pod goes to running state and points to the `cp.icr.io/cp/isf/isf-storage-services:2.2.0-latest` version, run the following API endpoint command in the `isf-storage-service` pod's terminal:
    
        curl -k https://isf-scale-svc/api/v1/upgradeExcludingOperator
    
    You can observe the new logs for `isf-storage-service` as well.
3.  Set the Edition to `erasure-code` in the `Daemon CR`:
    
        oc patch daemon ibm-spectrum-scale \
         --type='json' -n ibm-spectrum-scale \
         -p="[{'op': 'replace', 'path': '/spec/edition', 'value': "erasure-code"}]" 
    
4.  Run the following curl command to deploy the operator on the Red Hat® OpenShift® Container Platform cluster:
    
        curl -k https://isf-scale-svc/api/v1/upgradeWithOperator
    
    You can observe new logs for `isf-storage-service`.
5.  Wait for sometime and check whether the status of IBM Storage Scale core pod is in restart or running state in `ibm-spectrum-scale`.

CSI Pods experience scheduling problems post node drain
-------------------------------------------------------

Problem statement

Whenever a compute node is drained, an eviction of CSI PODs or sidecar PODs occur. The remaining available set of compute nodes cannot host the CSI PODs or sidecar PODs because of resource constraints at that point in time.

Resolution

Ensure that a functional system exists with available compute nodes, having sufficient resources to accommodate evicted CSI PODs or sidecar PODs.

Global Data Platform upgrade progress
-------------------------------------

Problem statement

During Global Data Platform upgrade, the progress percentage can go beyond 100% in a few cases. You can ignore this issue, as the percentage comes down to 100 % after the successful completion of the upgrade.

Scale installation might get stuck with ECE pods in `init` state
----------------------------------------------------------------

Problem statement

The Scale installation might get stuck with ECE pods in `init` state with the following error:

The node already belongs to a GPFS cluster.

Resolution

1.  Check whether the daemon-network IP of the pod is pingable.
2.  If it is not pingable, then restart all available TORs.
3.  After you restart TORs, check whether daemon-network IP is pingable.
4.  Run the following command to manually clean up the nodes:
    
    For the given worker node, run the following oc command:
    
        oc debug node/<openshift_worker_node> -T -- chroot /host sh -c "rm -rf /var/mmfs"
    
5.  Kill the pod and wait for it to come up again. The Scale installation resumes after all the ECE pods goes to running state.

Expansion rack fails for 4+2p setup
-----------------------------------

Problem statement

Sometimes, in a high-availability cluster, the expansion rack fails for 4+2p setup.

Resolution

*   Do the following workaround steps to resolve the issue:
    1.  In the OpenShift Container Platform user interface, go to Administration > CustomResourceDefinitions > Scale > Instances > storagemanager > Yaml and check whether the `scaleoutStatus: IN-PROGRESS`
    2.  If `scaleoutStatus: IN-PROGRESS`, go to Administration > CustomResourceDefinitions > RecoveryGroup > Instances and check for the created RecoveryGroups.
    3.  If more than two recovery groups are created, do the following actions:
        1.  Go to Administration > CustomResourceDefinitions > RecoveryGroup > Instances > ..
        2.  Find `RecoveryGroup` instance with suffix rg2.
        3.  Click the ellipsis menu for that instance and select the Delete RecoveryGroup option.
    4.  Do the following checks to validate whether the node upsize completed successfully:
        *   In the IBM Storage Fusion user interface, go to the IBM Storage Scale user interface by using the app switcher.
        *   Click Recovery groups on the IBM Spectrum Scale RAID tile.
        *   Click the entry of Recovery group server nodes.
        *   Check whether all the newly added nodes for expansion rack are listed and is in Healthy state.
        *   Create a sample PVC to check whether the storage can be provisioned.
    5.  Check for the labels on each recovery group.
        1.  In the OpenShift Container Platform console, go to Administration > CustomResourceDefinitions > RecoveryGroup > Instances > Yaml.
        2.  Check whether the YAML contains either of these two labels:
            *   `scale.spectrum.ibm.com/scale: up`
            *   `scale.spectrum.ibm.com/scale: out`
        3.  If it is yes for the previous step, remove those labels from each recovery group and click Save.

Known issues in installation
----------------------------

*   The display unit of filesystem block size in `storagesetupcr` is MiB and that of `ibmspectrum-fs` is M. As IBM Storage Scale also uses MiB format internally, you can ignore this inconsistency.

Known issues in upgrade
-----------------------

*   During upgrade, the IBM Storage Fusion rack user interface, IBM Storage Scale user interface, Grafana endpoint, and Applications are not reachable for sometime.
*   During the upgrade, an intermittent error `-1 node updated successfully out of 5 nodes` shows on the IBM Storage Fusion user interface in the upgrade details page.
*   During the upgrade, an intermittent error shows on the IBM Storage Fusion user interface that the progress percentage for the Global Data Platform decreased. It occurs, especially when the upgrade for one node is completed.
*   During the upgrade, an intermittent error `Global Data Platform upgrade failed` shows on the IBM Storage Fusion user interface in the upgrade details page. Ignore the error because it might recover during the next reconciliation.

**Parent topic:** [Troubleshooting installation and upgrade issues in IBM Storage Fusion services](../tshooting/sds_services_troubleshooting.html "Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.")



Hub and spoke connection issues
===============================

Procedure to debug issue in the hub and spoke connections. Backup & Restore service uses connection CR to setup hub and spoke connection.

You might encounter an error when you attempt setup connections between clusters.

Bootstrap token in init secret is not correct or expired: Unauthorized
----------------------------------------------------------------------

Problem statement

Connection setup fails with the following message in the connection CR:

    
    apiVersion: application.isf.ibm.com/v1
    kind: Connection
    metadata:
      name: <connection-name>
      namespace: <connection-namespace>
    spec:
      remoteCluster:
        apiEndpoint: <cluster api endpoint>
        connectionOperatorNamespace: <connection-namespace>
        heartBeatInterval: 10m
        initSecretName: <init-secret-name>
    status:
      conditions:
        - lastTransitionTime: '2023-06-15T02:31:01Z'
          message: 'Bootstrap token in init secret is not correct or expired: Unauthorized'
          reason: CreateBootstrapSecret
          status: 'False'
          type: BootstrapSecretAvaliable
      connectionFromRemoteClusterHealth:
        message: ''
        messageCode: ''
        messageType: ''
      connectionState: Failed
      connectionToRemoteClusterHealth:
        message: ''
        messageCode: ''
        messageType: ''
    

Cause

The bootstrap token in the `init` secret is not correct or expired.

Resolution

1.  Get the bootstrap token again.
    
        oc create token isf-application-operator-cluster-bootstrap -n <connection-namespace>
    
2.  Replace the token in `init` secret:
    
        oc edit secret <init-secret-name> -n <connection-namespace>
    

CA certificate of peer cluster is not correct
---------------------------------------------

Problem statement

The CA certificate of peer cluster is not correct error occurs in connection CR.

Cause

The CAcert in the configmap `kube-root-ca.crt` in namespace `kube-public` of the remote cluster is not correct.

Resolution

In the remote cluster, place the right CAcert in the configmap `kube-root-ca.crt` and namespace `kube-public`. Connection pkg also provides a customized configmap.

If it is not possible to place the right CAcert in configmap `kube-root-ca.crt` and namespace `kube-public`, then place the right CAcert in `custom-ca.crt` and Fusion namespace:

    
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: custom-ca.crt
      namespace: <connection-namespace>
    data:
      ca.crt: <right CAcert>
    

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")



Issues related to IBM Storage Fusion HCI System node drains
===========================================================

Use these common troubleshooting tips and tricks when you work with IBM Storage Fusion HCI System.

Issues related to draining IBM Storage Fusion HCI System node
-------------------------------------------------------------

Pod Distribution Budget (PDB) limits the number of Pods of a replicated application that are down simultaneously from voluntary disruptions. Scale uses PDB to ensure that storage does not get corrupted and minimum number nodes are available. You may experience delays or failure in node drain or node restart during any of the following operations:

Note: You must never reboot a node before it is drained successfully. Always perform node power operations from the IBM Storage Fusion HCI System user interface as it ensures successful node drain.

*   Configuration updates
    
    *   Red Hat® OpenShift® Machine Config Operator (MCO)
    *   IBM Storage Fusion component specification changes
    
*   Upgrade
    
    *   OpenShift Container Platform upgrades
    *   IBM Storage Fusion software upgrades
    *   Firmware upgrades
    
*   User Initiated
    
    *   Maintenance operation
    *   kdump enabling/disabling
    *   Operations that result in node restart
    *   Node drain
    

Cause

The PDB configuration is set by Scale to control the node drain so that the Scale cluster remains healthy always, and you can drain only an allowed quantity of nodes at any time. For more details, see [Scale behavior during node restarts](#sf_hci_common_issues__pdbscalebehavior).

Resolution

When node drain hangs due to any of the suggested operations, then do the following steps:

1.  Run the following command to identify the pod that causes the issue.
    
        oc describe cmt <target node name> -n ibm-spectrum-fusion-ns
    
    The command lists pods pending for eviction.
2.  Go through events of the node having the issue:
    
    Scenario where drains are prevented by an application
    
    If drains are prevented by an application, consult the owner of the application to proceed with the drain, and manually drain the node. For more information about the issue and to troubleshoot, see [Identifying applications preventing cluster maintenance](https://www.ibm.com/docs/en/scalecontainernative?topic=troubleshooting-identifying-applications-preventing-cluster-maintenance "(Opens in a new tab or window)").
    
    Scenario where drains are prevented by VM
    
    Check [Identifying applications preventing cluster maintenance](https://www.ibm.com/docs/en/scalecontainernative?topic=troubleshooting-identifying-applications-preventing-cluster-maintenance "(Opens in a new tab or window)") to determine the node that is waiting for reboot and check whether live migration is set up properly to allow VM to migrate to a different node. For more details about setting up live migrate, see [Virtual machine live migration](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/virtualization/live-migration "(Opens in a new tab or window)").
    
    Scenarios where issue is due to node maintenance
    
    *   The maintenance mode on a node can take from four minutes to thirty minutes to succeed. If the maintenance mode operation is taking more than 30 minutes, then Fusion continues to retry draining the pods with a warning event BMYCO0012, and gets timed out eventually. If you want to stop the retries, then delete the `Computemaintenance` CR instance.
        
        Run the following command to delete `Computemaintenance` CR instance.
        
            oc delete cmt <instnace-nodename> -n ibm-spectrum-fusion-ns
        
    *   If it takes a long time and you want to continue the operation, check the pod that is pending by using the oc get pods command. If the pod name belongs to IBM Storage Scale, see [Identifying applications preventing cluster maintenance](https://www.ibm.com/docs/en/scalecontainernative?topic=troubleshooting-identifying-applications-preventing-cluster-maintenance "(Opens in a new tab or window)"). If issue persists, collect Scale system health and Scale logs, and contact IBM support.
    
    If you want to understand Scale behavior during node reboots, see Scale behavior during node restarts section.
    

Scale behavior during node restarts
-----------------------------------

Restart of a node can happen due to various reasons such as an MCO roll out, user initiated, or as part of firmware or software upgrades. Scale has a maximum tolerance on the number of nodes that are in unavailable or in not ready state. It is based on the erasure code that is configured for the storage cluster. To stay within this tolerance, Scale uses the POD Disruption Budget (PDB) feature of OpenShift.

Scale CNSA implements PDB with a "maxUnavailable=0", which means that it does not allow a Scale pod to go down without its knowledge. Even with a "maxUnavailable=0" PDB, the design does allow for cluster updates. In this configuration, MCO is prevented from taking down the node and draining the CNSA core pod. The Scale CNSA pod detects the operation, drains the applications, and then exits itself, thus freeing the node up for the operation to continue.

If any application refuses to unmount, or if the scale core pod itself determines the cluster, then it goes into a bad or hung state. If it goes down, then things pause until the condition gets resolved.

For more information about identifying applications that prevent cluster maintenance, see [Identifying applications preventing cluster maintenance](https://www.ibm.com/docs/en/scalecontainernative?topic=troubleshooting-identifying-applications-preventing-cluster-maintenance "(Opens in a new tab or window)").

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")




Install events and error codes
==============================

List of all the events that you might encounter during installation.

For Call Home events, see [Install Call Home events and error codes](../errorcodes/sf_install_callhomeevents.html "List of Call Home events that you might encounter in the install.").

For Informational, Warning and Critical events, see the following:

*   **[Install Call Home events and error codes](../errorcodes/sf_install_callhomeevents.html)**  
    List of Call Home events that you might encounter in the install.
*   **[BMYIN1101](../errorcodes/BMYIN1101.html)**  
    Node addition to OpenShift® Container Platform is in progress.
*   **[BMYIN1104](../errorcodes/BMYIN1104.html)**  
    Bare metal host is not yet provisioned.
*   **[BMYIN1105](../errorcodes/BMYIN1105.html)**  
    OpenShift node is not yet ready.
*   **[BMYIN1106](../errorcodes/BMYIN1106.html)**  
    Node network configuration policy is not yet configured.
*   **[BMYIN1107](../errorcodes/BMYIN1107.html)**  
    Node is now added to OpenShift cluster.
*   **[BMYIN1109](../errorcodes/BMYIN1109.html)**  
    One time network boot and power off was successful.
*   **[BMYIN1117](../errorcodes/BMYIN1117.html)**  
    Waiting for new node to come up.
*   **[BMYIN1501](../errorcodes/BMYIN1501.html)**  
    Application has been created successfully.
*   **[BMYIN1502](../errorcodes/BMYIN1502.html)**  
    Installing service Global Data Platform.
*   **[BMYIN1503](../errorcodes/BMYIN1503.html)**  
    Installed service Global Data Platform.
*   **[BMYIN1504](../errorcodes/BMYIN1504.html)**  
    Installing service Backup & Restore.
*   **[BMYIN1505](../errorcodes/BMYIN1505.html)**  
    Installed service Backup & Restore.
*   **[BMYIN1508](../errorcodes/BMYIN1508.html)**  
    Service is installed.
*   **[BMYIN1509](../errorcodes/BMYIN1509.html)**  
    Service installation is complete.
*   **[BMYIN1510](../errorcodes/BMYIN1510.html)**  
    Service instance is discovered.
*   **[BMYIN1512](../errorcodes/BMYIN1512.html)**  
    Catalog source is created.
*   **[BMYIN1513](../errorcodes/BMYIN1513.html)**  
    Catalog source creation is completed.
*   **[BMYIN2102](../errorcodes/BMYIN2102.html)**  
    Location for the node is not specified.
*   **[BMYIN2103](../errorcodes/BMYIN2103.html)**  
    Failed to read kickstart config map data.
*   **[BMYIN2110](../errorcodes/BMYIN2110.html)**  
    Failed to read userconfig secret object.
*   **[BMYIN2111](../errorcodes/BMYIN2111.html)**  
    DNS validation failed for the node.
*   **[BMYIN2112](../errorcodes/BMYIN2112.html)**  
    Invalid name is used in ComputeProvisionWorker CR.
*   **[BMYIN2116](../errorcodes/BMYIN2116.html)**  
    Failed to scale machineset during new node addition.
*   **[BMYIN2117](../errorcodes/BMYIN2117.html)**  
    Rack serial for the node is not specified.
*   **[BMYIN2118](../errorcodes/BMYIN2118.html)**  
    Unable to read OpenShift Container Platform location for the node from kickstart configmap.
*   **[BMYIN2119](../errorcodes/BMYIN2119.html)**  
    Both location and rack serial for the node are not specified.
*   **[BMYIN2501](../errorcodes/BMYIN2501.html)**  
    Error installing service Global Data Platform.
*   **[BMYIN2502](../errorcodes/BMYIN2502.html)**  
    Error installing service Backup & Restore.
*   **[BMYIN3116](../errorcodes/BMYIN3116.html)**  
    Service is Unhealthy.
*   **[BMYIN3117](../errorcodes/BMYIN3117.html)**  
    Service Catalog source creation is failed.
*   **[BMYIN3119](../errorcodes/BMYIN3119.html)**  
    Service installation failed.
*   **[BMYPC5001](../errorcodes/BMYPC5001.html)**  
    Cluster operator is available but is in progress state.
*   **[BMYPC5002](../errorcodes/BMYPC5002.html)**  
    A `MachineConfigPool` is updating.
*   **[BMYPC5004](../errorcodes/BMYPC5004.html)**  
    A pod PodDisruptionBudget (PDB) is set to zero.
*   **[BMYPC5005](../errorcodes/BMYPC5005.html)**  
    Virtual machine is using PVC with non-RWX access mode.
*   **[BMYPC5006](../errorcodes/BMYPC5006.html)**  
    A pod in the `openshift-operator-lifecycle-manager` namespace is unhealthy.
*   **[BMYPC5010](../errorcodes/BMYPC5010.html)**  
    IBM Storage Fusion operator installation is in progress.
*   **[BMYPC5013](../errorcodes/BMYPC5013.html)**  
    Unable to check DNS resolution for the registry.
*   **[BMYPC6001](../errorcodes/BMYPC6001.html)**  
    Cluster operator is not healthy.
*   **[BMYPC6002](../errorcodes/BMYPC6002.html)**  
    A `MachineConfigPool` is degraded.
*   **[BMYPC6003](../errorcodes/BMYPC6003.html)**  
    A node is not ready or unscheduleable.
*   **[BMYPC6006](../errorcodes/BMYPC6006.html)**  
    A pod in the `openshift-operator-lifecycle-manager` namespace is not ready.
*   **[BMYPC6007](../errorcodes/BMYPC6007.html)**  
    A `Catalogsource` is not ready.
*   **[BMYPC6008](../errorcodes/BMYPC6008.html)**  
    Inaccessible registry.
*   **[BMYPC6010](../errorcodes/BMYPC6010.html)**  
    IBM Storage Fusion operator is in a `Failed` state or not found.
*   **[BMYPC6013](../errorcodes/BMYPC6013.html)**  
    The DNS resolution for image registry failed.
*   **[BMYPC6023](../errorcodes/BMYPC6023.html)**  
    `Pidslimit` on the nodes is less than 12228.
*   **[Configuration error](../errorcodes/INS0001F.html)**  
    **Message**: Encountered a configuration error. Red Hat® OpenShift Container Platform installation did not start successfully.
*   **[Timed out creating bootstrap VM](../errorcodes/INS0002F.html)**  
    **Message**: Bootstrap VM creation on provisioner node (also known as RU7 or compute-1-ru7) has not started.
*   **[Bootstrap VM creation did not complete](../errorcodes/INS0003F.html)**  
    **Message**: Bootstrap VM creation on the provisioner node (also known as RU7 or compute-1-ru7) did not complete within the expected time.
*   **[Failed control nodes inspection](../errorcodes/INS0004F.html)**  
    **Message**: Inspection failed for one or more control nodes.
*   **[Control nodes creation failed](../errorcodes/INS0005F.html)**  
    **Message**: Failed to create the Red Hat OpenShift Container Platform cluster control nodes.
*   **[Cluster bootstrapping failed](../errorcodes/INS0006F.html)**  
    
*   **[Catalog source pod is healthy](../errorcodes/INS0022F.html)**  
    **Message**: IBM Storage Fusion catalog source pod is not found or is not healthy.
*   **[Operator subscription isf-subscription not found](../errorcodes/INS0023F.html)**  
    **Message**: Subscription for IBM Storage Fusion operator not found.
*   **[Bootstrap VM is not pingable](../errorcodes/INS0090F.html)**  
    A temporary bootstrap VM is created on provisioner node (also known as RU7 or compute-1-ru7) to create Red Hat OpenShift Container Platform control plane.
*   **[Configuration error](../errorcodes/INS0207F.html)**  
    The certificate does not contain a wildcard domain for cluster.
*   **[Failed to install CSV for isf-operator](../errorcodes/INS0038F.html)**  
    When you install IBM Storage Fusion, the installation fails with "Failed to install CSV for ifs-operator" error.
*   **[Failed to instal IBM Storage Fusion operator](../errorcodes/INS0041F.html)**  
    Failed to install the IBM Storage Fusion operator. Check the log file /home/kni/logs/installoperator\_playbook.log for more details.
*   **[The catlogsource pod for Red Hat operators is not running](../errorcodes/INS0042F.html)**  
    The `catlogsource` pod for `redhat-operators` is not running in `openshift-marketplace` namespace. Check the log file %s for more details.
*   **[Failed to deploy nmstate-operator](../errorcodes/INS0043F.html)**  
    Failed to deploy `nmstate-operator`. Check the log file /home/kni/logs/installoperator\_playbook.log for more details.
*   **[Failed to install CSV for kubernetes-nmstate-operator](../errorcodes/INS0048F.html)**  
    While installing IBM Storage Fusion on HCI, the installation fails with the error message Failed to install CSV for kubernetes-nmstate-operator. Check log file /home/kni/logs/installoperator\_playbook.log for more details.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")



Installation and upgrade issues
===============================

Use the troubleshooting tips and tricks in IBM Storage Fusion installation.

Note: To log in to the provisioner node, as a kni user, ssh to compute-1-ru7. It is the same host that the IBM Service Support Representative (SSR) provided you before the stage 2 of the installation: `http://<host IP address>:3000/isfsetup`. For the actual step, see [Installation procedure](../install/sf_install.html#sf_install_new__step1).

*   You might observe unpredictable Red Hat® OpenShift® Container Platform cluster degradation issues that include but not limited to severe performance and operator degradation issues. To avoid these issues, go through the Red Hat recommendations. For example, the following list has some of the high-level considerations (not all inclusive):
    
    *   Your cluster must have sufficient CPU and RAM resources that are allocated to etcd and compute nodes.
    *   The etcd disks must be high-performing with low read or write latency and high IOPs. You can use [FIO test](https://access.redhat.com/solutions/4885641 "(Opens in a new tab or window)") to check IOPS for etcd disks.
    *   There must not be high network latency among etcd members.
    
    For more information and a complete list of considerations, see [https://docs.openshift.com/container-platform/4.15/scalability\_and\_performance/planning-your-environment-according-to-object-maximums.html](https://docs.openshift.com/container-platform/4.15/scalability_and_performance/planning-your-environment-according-to-object-maximums.html "(Opens in a new tab or window)").
*   If you notice that the operator installation or upgrade fails with `DeadlineExceeded error`, see [Operator installation or upgrade fails with DeadlineExceeded error](sf_deadlineexceeded.html "IBM Cloud Paks foundational services operator ClusterServiceVersion (CSV) status shows Failed and its InstallPlan status shows Failed after the subscription gets created.").

*   **[Troubleshooting installation issues](../tshooting/tshoot_install_upgrade.html)**  
    Troubleshooting IBM Storage Fusion HCI System installation issues.
*   **[Troubleshooting upgrade issues](../tshooting/kwnissues_upgrade.html)**  
    Troubleshooting IBM Storage Fusion HCI System upgrade issues.
*   **[Operator installation or upgrade fails with DeadlineExceeded error](../tshooting/sf_deadlineexceeded.html)**  
    IBM Cloud Paks foundational services operator ClusterServiceVersion (CSV) status shows Failed and its InstallPlan status shows Failed after the subscription gets created.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Serviceability events and error codes
=====================================

List of all the events that you might encounter that related to serviceability.

For Informational, Warning, and Critical events, see the following:

*   **[Serviceability Call Home events and error codes](../errorcodes/sf_serviceability_callhomeevents.html)**  
    List of Call Home events that you might encounter that related to serviceability.
*   **[BMYLC0001](../errorcodes/BMYLC0001.html)**  
    Remaining storage space is less than 30 percent.
*   **[BMYLC0002](../errorcodes/BMYLC0002.html)**  
    Remaining storage space is less than 10 percent.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")


Metro-DR events and error codes
===============================

List of all the events and error codes that you might encounter in Metro-DR setup.

*   **[Metro-DR event codes](../errorcodes/sf_metrodr_event_codes.html)**  
    List of all the event codes that you might encounter in Metro-DR setup.
*   **[Metro-DR error codes](../errorcodes/sf_metrodr_errorcodes.html)**  
    List of all the error codes that you might encounter in Metro-DR setup.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")


Metro-DR issues
===============

Use these troubleshooting information to resolve issues when you work with Metro-DR.

For issues related to the upgrade of Metro-DR setup, see [Installation and upgrade issues](sf_install_tshooting.html "Use the troubleshooting tips and tricks in IBM Storage Fusion installation.").

Failback issues
---------------

Problem statement

During unplanned failover, some applications can be in a "replication error" state after recovery of the failed site with fencing.

Resolution

*   If the application failover is not successful, then check for the `volumeattachments` of the application by using the following command.
    
        oc get volumeattachment -n <namespace name> | grep < pvc name of given application>
    
    If the `volumeattachments` exist, then try to delete by using the following command.
    
        oc delete volumeattachment -n < namespace name>
    

IBM Storage Fusion operator status is pending with errors
---------------------------------------------------------

Problem statement

The IBM Storage Fusion operator status is pending with the following errors in 2.7.2 version:

*   RequirementsNotMe
*   asyncreplications.dataprotection.isf.ibm.com

Resolution

Apply the following CRD YAML:

    
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      annotations:
        controller-gen.kubebuilder.io/version: v0.8.0
      creationTimestamp: null
      name: asyncreplications.dataprotection.isf.ibm.com
    spec:
      group: dataprotection.isf.ibm.com
      names:
        kind: AsyncReplication
        listKind: AsyncReplicationList
        plural: asyncreplications
        singular: asyncreplication
      scope: Namespaced
      versions:
      - name: v1
        schema:
          openAPIV3Schema:
            description: AsyncReplication is the Schema for the asyncreplications API
            properties:
              apiVersion:
                description: 'APIVersion defines the versioned schema of this representation
                  of an object. Servers should convert recognized schemas to the latest
                  internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
                type: string
              kind:
                description: 'Kind is a string value representing the REST resource this
                  object represents. Servers may infer this from the endpoint the client
                  submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
                type: string
              metadata:
                type: object
              spec:
                description: AsyncReplicationSpec defines the desired state of AsyncReplication
                properties:
                  consistencyGroup:
                    description: Foo is an example field of AsyncReplication. Edit asyncreplication_types.go
                      to remove/update
                    type: string
                  recoveryPointObjective:
                    type: string
                  remoteSite:
                    type: string
                  targetRole:
                    type: string
                type: object
              status:
                description: AsyncReplicationStatus defines the observed state of AsyncReplication
                properties:
                  currentRole:
                    description: 'INSERT ADDITIONAL STATUS FIELD - define observed state
                      of cluster Important: Run "make" to regenerate code after modifying
                      this file'
                    type: string
                  phase:
                    type: string
                type: object
            type: object
        served: true
        storage: true
        subresources:
          status: {}
    status:
      acceptedNames:
        kind: ""
        plural: ""
      conditions: []
      storedVersions: []

Global Data Platform upgrade fails in site 1
--------------------------------------------

Problem statement

*   Whenever you do the following upgrades on site 1 of the Metro-DR, the "Global Data platform service not active on all the nodes" error message gets displayed:
    *   IBM Storage Fusion 2.7.2 to 2.8.0
    *   Global Data Platform 5.19.0 to 5.2.0
*   When you upgrade IBM Storage Fusion 2.7.2 to 2.8.0, the Global Data Platform upgrade of 5.1.9.0 to 5.2.0.0 of Metro-DR Site 1 gets stuck with error "ECE service not active on all the nodes".

Resolution

Run the following command to patch Scale recovery group with 'Active' condition:

    oc patch recoverygroup rg201 -n ibm-spectrum-scale --type=merge --subresource status --type='json' -p '[{ "op": "add", "path": "/status/conditions/-","value": { "type": "Active", "status": "True", "message": "The recovery group is active", "reason": "Active", "lastTransitionTime":"'"`date +%FT%T`Z"'"}}]'
    

Less descriptor disks
---------------------

Problem statement

The site monitoring checks whether enough descriptor disks are assigned for each site. For multiple RGs per site, only one RG sees the descriptor disks. The other RG servers send disk\_missing that is interpreted as less descriptor disks.

Resolution

As a resolution, add the `site_fs_desc_fail` event to the "ignore list" to suppress the entire event. Run the command `mmchconfig mmhealth-events-ignore="site_fs_desc_fail" --force` once on the cluster and then run `mmsysmoncontrol restart` on the cluster manager node.

Latest tiebreaker version
-------------------------

After the upgrade of tiebreaker to the latest version, Metro-DR CR does not show the latest version. You can ignore this as it is a known issue.

Application removals fail in the local tab of failed site after recovery or reconnect
-------------------------------------------------------------------------------------

You can observe this issue only in case of fail over of fail back of applications on the surviving site. You can ignore this issue as the CR status is correct, and further relocation or fail back of these applications happen properly.

Add and Remove DR does not work as expected when a remote site is down
----------------------------------------------------------------------

When multiple DR protected applications exist in the local site and the other site goes down, the add and remove operations of disaster recovery takes longer than usual.

Connection setup after OpenShift Container Platform cluster recovery
--------------------------------------------------------------------

Problem statement

The OpenShift® Container Platform cluster can have problems and become unusable.

Resolution

After you recover the cluster, rejoin the connections. For the steps to clean the connection and setup the connection between two clusters again, see [Connection setup after OpenShift Container Platform cluster recovery](sf_ocp_cluster_recover.html "The OpenShift Container Platform cluster can have problems and become unusable. After you recover the cluster, rejoin the connections.").

IBM Storage Fusion HCI System installation hangs because of disaster recovery
-----------------------------------------------------------------------------

Resolution

If minio pod does not come up for a long time and the Metro-DR installation gets stuck even after the minio pod goes to the running state, then restart Metro-DR deployment in `ibm-spectrum-fusion-ns` (or your Fusion namespace) to trigger reconciliation.

RamenDR operator crashes after the restoration of a failed site
---------------------------------------------------------------

Resolution

If RamenDR operator crashes after the restoration of a failed site, then apply the following workaround:

1.  Retrieve `s3CompatibleEndpoint` from `s3StoreProfiles` of the local site.
2.  Retrieve the secret of local site that is specified in `s3SecretRef` of `s3StoreProfiles`.
3.  Run the following command from `isf-minio` pod in `ibm-spectrum-fusion-ns` namespace to connect to the local Minio server.
    
        mc alias set myminio <s3CompatibleEndpoint retrieved in step 1> <AWS_ACCESS_KEY_ID parameter of secret retrieved in step 1> < AWS_SECRET_ACCESS_KEY parameter of secret retrieved in step 1>
    
4.  Run the following command to find out all zero-bytes files in Minio store:
    
        mc ls myminio --recursive --insecure | grep -i VolumeReplicationGroup | grep 0B
    
5.  Run the following command to delete the zero-bytes file that is retrieved in step 3:
    
        mc rm <path of files retrieved in step 3>
    
6.  Restart `ramen-dr-cluster-operator` deployment in the `ibm-spectrum-fusion-ns` namespace.

If the Volumereplicationgroup's ReplicationState of the application fail over does not change to secondary after the restoration of a failed site, then do the following workaround steps:

1.  Go to Red Hat® OpenShift Container Platform web management console.
2.  Go to CustomResourceDefinitions > volumereplicationgroups.ramendr.openshift.io > instances.
3.  Edit YAMLs of selected `Volumereplicationgroups` and change `spec.replicationState = secondary`.
4.  Save YAMLs.

Failovers of concurrent applications take a long time after network recovery
----------------------------------------------------------------------------

Problem statement

During Disaster Recovery scenario, when either of the Metro-DR site is powered Off or network is disconnected.

Resolution

As a resolution, run the following steps before you initiate failover of applications on the surviving site:

1.  In Red Hat OpenShift Container Platform console, go to Workloads > ConfigMap.
2.  Open the `ramen-dr-cluster-operator-config` configmap in the `ibm-spectrum-fusion-ns` namespace.
3.  Add MaxConcurrentReconcile parameter in this configmap with value as 50.
4.  Save the changes and restart `ramen-dr-cluster-operator` deployment in the `ibm-spectrum-fusion-ns` namespace.

Connection between IBM Storage Scale pods
-----------------------------------------

Problem statement

Unable to ping between IBM Storage Scale pods after you reestablish the network connection between site 1 and site 2.

**Workaround:**

1.  Change in the CR `Spec` of the `installSubmariner` field to `uninstall`.
    
        oc edit mni
        
        Spec:
            installSubmariner: uninstall
    
2.  Delete the namespace, if it is not already deleted.
3.  Change in the CR `Spec` of the `installSubmariner` field to `install`.
    
        oc edit mni
        
        Spec:
            installSubmariner: install
    
4.  If the installation is not successful, then try to install the submariner by using the following manual steps:
    1.  Label the control nodes as gateways on both sites of Metro-DR.
        
            oc label node control-0.isf-rackc.rtp.raleigh.ibm.com submariner.io/gateway=true
            oc label node control-1.isf-rackc.rtp.raleigh.ibm.com submariner.io/gateway=true
            oc label node control-2.isf-rackc.rtp.raleigh.ibm.com submariner.io/gateway=true
        
    2.  Log in to MetroDR-operator pod.
        
            [root@hci-dev1 ~]# oc get po | grep metr
            isf-metrodr-operator-controller-manager-bddffd98-dhvgt            2/2     Running   0               13h
            [root@hci-dev1 ~]# oc rsh isf-metrodr-operator-controller-manager-bddffd98-dhvgt
        
    3.  Deploy the broker for submariner.
        
            subctl deploy-broker  --operator-debug  --kubeconfig=/tmp/site-1-kubeconfig
        
    4.  Join to broker.
        
            subctl join --kubeconfig=/tmp/site-2-kubeconfig --operator-debug --pod-debug  --clusterid=site-2  broker-info.subm 
            subctl join --kubeconfig=/tmp/site-1-kubeconfig  --operator-debug --pod-debug  --clusterid=site-1 broker-info.subm 
        

Disaster recovery connections in installing state for hours
-----------------------------------------------------------

Diagnosis

You might encounter authentication or authorization issues while you connect to the remote cluster. To confirm that the issue is related to authorization, check whether you see the following messages in the Metro-DR pod logs:

*   Incorrect token, need to regenerate kubeconfig file
    
*   UnAuthorized
    

Cause

*   If Metro-DR instance displays any of the following error messages, then do the following workaround:
    
        oc get mdr -oyaml
    
    “Submariner is not installed”
    
*       oc get mni -oyaml 
    
    “Failed to uninstall Submariner” 
    OR
    “Submariner installation failed”
    
*   Submariner connectivity might get affected when any of the nodes go to a bad state.
    
    If problems exist in pod connectivity between racks even after all nodes are back in working condition, then do the following workaround.
    

Resolution

1.  Update site 2 password on site 1 and regenerate the kubeconfig file:
    1.  Get the site 2 token from site 2 service account:
        
            [root@hci-dev1 ~]# oc get sa fusion-admin-controller-manager -oyaml
            secrets:
            - name: fusion-admin-controller-manager-dockercfg-vmp2m
            - name: fusion-admin-controller-manager-token-wj96d
            
            
            [root@hci-dev1 ~]# oc get secret fusion-admin-controller-manager-token-wj96d -oyaml
            data:
              token: site2Token 
        
    2.  Update the site 2 token in secret `isf-metrodr-config-secret` on site 1:
        
            [root@hci-dev1 ~]# oc edit secret isf-metrodr-config-secret
            Data:
               Site2KubePassword: site2Token
            
        
    3.  Log in to the Metro-DR DR pod on site 1 and delete the incorrect site 2 kubeconfig file:
        
            root@hci-dev1 ~]# oc get po | grep metro
            isf-metrodr-operator-controller-manager-7fd8c4fbc-wwkc8           2/2     Running             0             13h
            
            [root@hci-dev1 ~]# oc exec -ti isf-metrodr-operator-controller-manager-7fd8c4fbc-wwkc8 -c manager bash
            kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
            bash-4.4$ ls /tmp/
            ks-script-c9hd6mir  ks-script-jrnjxyf4	site-1-kubeconfig  site-2-kubeconfig
            bash-4.4$ rm -f /tmp/site-2-kubeconfig
            
        
2.  Update site 1 password on site 2 and regenerate the kubeconfig file:
    1.  Get the site 1 token from site 1 service account:
        
            [root@hci-dev1 ~]# oc get sa fusion-admin-controller-manager -oyaml
            secrets:
            - name: fusion-admin-controller-manager-dockercfg-vmp2m
            - name: fusion-admin-controller-manager-token-wj96d
            
            
            [root@hci-dev1 ~]# oc get secret fusion-admin-controller-manager-token-wj96d -oyaml
            data:
              token: site1Token 
        
    2.  Update the site 1 secret token in `isf-metrodr-config-secret` on site 2:
        
            [root@hci-dev1 ~]# oc edit secret isf-metrodr-config-secret
            Data:
               Site1KubePassword: site1Token
        
    3.  Log in to the Metro-DR pod on site 2 and delete the incorrect site 1 kubeconfig file:
        
            root@hci-dev1 ~]# oc get po | grep metro
            isf-metrodr-operator-controller-manager-7fd8c4fbc-wwkc8           2/2     Running             0             13h
            
            [root@hci-dev1 ~]# oc exec -ti isf-metrodr-operator-controller-manager-7fd8c4fbc-wwkc8 -c manager bash
            kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
            bash-4.4$ ls /tmp/
            ks-script-c9hd6mir  ks-script-jrnjxyf4	site-1-kubeconfig  site-2-kubeconfig
            bash-4.4$ rm -f /tmp/site-1-kubeconfig
        

Issues with more than 3 quorum nodes on a cluster
-------------------------------------------------

Problem statement

The following error occurs only when the OpenShift Container Platform contains more than 10 nodes for the initial installation in a Metro-DR:

    mmchnode: The number of quorum nodes exceeds the maximum (8) allowed.

Cause

The Scale works fine when both sites are healthy. As the quorum nodes are not balanced on both sites, when the site that has five quorum nodes goes down, the whole scale cluster goes down.

Resolution

1.  Run the following command to get the quorum nodes:
    
        oc get nodes -l scale.spectrum.ibm.com/designation=quorum
    
2.  Run the following command to remove the label from two non-control nodes:
    
        oc label nodes <node_name> scale.spectrum.ibm.com/designation-
    

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Network events error codes
==========================

List of all the events that you might encounter in network node.

For Call Home events, see [Network Call Home events and error codes](../errorcodes/sf_network_callhomevents.html "List of Call Home events that you might encounter in the network.").

For the events that you might encounter during a switch firmware upgrade, see [Upgrade events and error codes](sf_upgrade_error_codes.html "List of all the events that you might encounter during upgrade.").

For Informational, Warning, and Critical events, see the following:

*   **[Network Call Home events and error codes](../errorcodes/sf_network_callhomevents.html)**  
    List of Call Home events that you might encounter in the network.
*   **[BMYNW0001](../errorcodes/BMYNW0001.html)**  
    Switch interface up.
*   **[BMYNW0003](../errorcodes/BMYNW0003.html)**  
    Switch interface down.
*   **[BMYNW0005](../errorcodes/BMYNW0006.html)**  
    <switch-name:switch IBM Serial Number>: Fan is not OK
*   **[BMYNW0007](../errorcodes/BMYNW0007.html)**  
    <switch-name:switch IBM Serial Number>: Fan is OK
*   **[BMYNW0009](../errorcodes/BMYNW0009.html)**  
    <switch-name:switch IBM Serial Number>: PSU Fan is not OK.
*   **[BMYNW0011](../errorcodes/BMYNW0011.html)**  
    <switch-name:switch IBM Serial Number>: PSU Fan is OK.
*   **[BMYNW0013](../errorcodes/BMYNW0014.html)**  
    <switch-name:switch IBM Serial Number>: PSU temperature is not OK.
*   **[BMYNW0015](../errorcodes/BMYNW0015.html)**  
    <switch-name:switch IBM Serial Number>: PSU temperature is OK.
*   **[BMYNW0017](../errorcodes/BMYNW0018.html)**  
    <switch-name:switch IBM Serial Number>: Ambient temperature is not OK.
*   **[BMYNW0019](../errorcodes/BMYNW0019.html)**  
    <switch-name:switch IBM Serial Number>: Ambient temperature is OK.
*   **[BMYNW0021](../errorcodes/BMYNW0021.html)**  
    <switch-name:switch IBM Serial Number>: CPU Package temperature not OK.
*   **[BMYNW0023](../errorcodes/BMYNW0023.html)**  
    <switch-name:switch IBM Serial Number>: CPU Package temperature OK.
*   **[BMYNW0025](../errorcodes/BMYNW0026.html)**  
    <switch-name:switch IBM Serial Number>: CPU Core temperature not OK.
*   **[BMYNW0027](../errorcodes/BMYNW0027.html)**  
    <switch-name:switch IBM Serial Number>: CPU Core temperature OK.
*   **[BMYNW0029](../errorcodes/BMYNW0030.html)**  
    <switch-name:switch IBM Serial Number>: Port ambient temperature is not OK.
*   **[BMYNW0031](../errorcodes/BMYNW0031.html)**  
    <switch-name:switch IBM Serial Number>: Port Ambient temperature is OK.
*   **[BMYNW0033](../errorcodes/BMYNW0034.html)**  
    <switch-name:switch IBM Serial Number>: Main board ambient temperature is not OK
*   **[BMYNW0035](../errorcodes/BMYNW0035.html)**  
    <switch-name:switch IBM Serial Number>: Main Board Ambient temperature is OK.
*   **[BMYNW0037](../errorcodes/BMYNW0038.html)**  
    <switch-name:switch IBM Serial Number>: ASIC temperature is not OK.
*   **[BMYNW0039](../errorcodes/BMYNW0039.html)**  
    <switch-name:switch IBM Serial Number>: ASIC temperature is OK.
*   **[BMYNW0041](../errorcodes/BMYNW0042.html)**  
    <switch-name:switch IBM Serial Number>: Greater than 90% L3 entries.
*   **[BMYNW0043](../errorcodes/BMYNW0043.html)**  
    <switch-name:switch IBM Serial Number>: L3 entries count normal.
*   **[BMYNW0045](../errorcodes/BMYNW0046.html)**  
    <switch-name:switch IBM Serial Number>: Greater than 90% L2 entries.
*   **[BMYNW0047](../errorcodes/BMYNW0047.html)**  
    <switch-name:switch IBM Serial Number>: L2 entries count normal.
*   **[BMYNW0049](../errorcodes/BMYNW0050.html)**  
    <switch-name:switch IBM Serial Number>: Lesser than 5% memory available.
*   **[BMYNW0051](../errorcodes/BMYNW0051.html)**  
    <switch-name:switch IBM Serial Number>: Memory availability normal.
*   **[BMYNW0053](../errorcodes/BMYNW0053.html)**  
    <switch-name:switch IBM Serial Number>: CumulusLinkUP - port: <port number>.
*   **[BMYNW0055](../errorcodes/BMYNW0056.html)**  
    <switch-name:switch IBM Serial Number>: CumulusLinkDOWN - port: <port number>.
*   **[BMYNW0057](../errorcodes/BMYNW0057.html)**  
    <switch-name:switch IBM Serial Number> : 15 min Load Average too high (= value).
*   **[BMYNW0111](../errorcodes/BMYNW0111.html)**  
    Failed to login to the switch.
*   **[BMYNW0112](../errorcodes/BMYNW0112.html)**  
    Switch IP address is invalid.
*   **[BMYNW0113](../errorcodes/BMYNW0113.html)**  
    Secret name is missing in the switch CR.
*   **[BMYNW0114](../errorcodes/BMYNW0114.html)**  
    Switch secret is missing.
*   **[BMYNW0115](../errorcodes/BMYNW0115.html)**  
    Failed to add SNMP trap configuration in switch.
*   **[BMYNW0116](../errorcodes/BMYNW0116.html)**  
    Failed to create SNMP exporter resources.
*   **[BMYNW0117](../errorcodes/BMYNW0117.html)**  
    Spine switch is not reachable.
*   **[BMYNW0118](../errorcodes/BMYNW0118.html)**  
    Rack is isolated as both the High Speed Switches of the rack are not reachable.
*   **[BMYNW0131](../errorcodes/BMYNW0131.html)**  
    SNMP exporter image name or version information is missing.
*   **[BMYNW0231](../errorcodes/BMYNW0231.html)**  
    There are extra VLANs on the switch.
*   **[BMYNW0351](../errorcodes/BMYNW0351.html)**  
    There are extra links on the switch.
*   **[BMYNW0411](../errorcodes/BMYNW0411.html)**  
    Failed to create switch CR.
*   **[BMYNW0412](../errorcodes/BMYNW0412.html)**  
    Failed to create VLAN CR.
*   **[BMYNW0413](../errorcodes/BMYNW0413.html)**  
    Failed to create link CR.
*   **[BMYNW0431](../errorcodes/BMYNW0431.html)**  
    PolicyUpdateError.
*   **[BMYNW0451](../errorcodes/BMYNW0451.html)**  
    Kickstart config map is missing.
*   **[BMYNW0452](../errorcodes/BMYNW0452.html)**  
    Failed to read kickstart config map data.
*   **[BMYNW0453](../errorcodes/BMYNW0453.html)**  
    The rackinfoconfig config map is missing.
*   **[BMYNW0454](../errorcodes/BMYNW0454.html)**  
    Failed to read rackinfoconfig config map data.
*   **[BMYNW0455](../errorcodes/BMYNW0455.html)**  
    The user config-secret secret is missing.
*   **[BMYNW0457](../errorcodes/BMYNW0457.html)**  
    Failed to read appliance-info config map data.
*   **[Configuration error](../errorcodes/INS0001F.html)**  
    **Message**: Encountered a configuration error. Red Hat® OpenShift® Container Platform installation did not start successfully.
*   **[Timed out creating bootstrap VM](../errorcodes/INS0002F.html)**  
    **Message**: Bootstrap VM creation on provisioner node (also known as RU7 or compute-1-ru7) has not started.
*   **[Bootstrap VM creation did not complete](../errorcodes/INS0003F.html)**  
    **Message**: Bootstrap VM creation on the provisioner node (also known as RU7 or compute-1-ru7) did not complete within the expected time.
*   **[Failed control nodes inspection](../errorcodes/INS0004F.html)**  
    **Message**: Inspection failed for one or more control nodes.
*   **[Control nodes creation failed](../errorcodes/INS0005F.html)**  
    **Message**: Failed to create the Red Hat OpenShift Container Platform cluster control nodes.
*   **[Cluster bootstrapping failed](../errorcodes/INS0006F.html)**  
    
*   **[Catalog source pod is healthy](../errorcodes/INS0022F.html)**  
    **Message**: IBM Storage Fusion catalog source pod is not found or is not healthy.
*   **[Operator subscription isf-subscription not found](../errorcodes/INS0023F.html)**  
    **Message**: Subscription for IBM Storage Fusion operator not found.
*   **[Bootstrap VM is not pingable](../errorcodes/INS0090F.html)**  
    A temporary bootstrap VM is created on provisioner node (also known as RU7 or compute-1-ru7) to create Red Hat OpenShift Container Platform control plane.
*   **[Configuration error](../errorcodes/INS0207F.html)**  
    The certificate does not contain a wildcard domain for cluster.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")

Connection setup after OpenShift Container Platform cluster recovery
====================================================================

The OpenShift® Container Platform cluster can have problems and become unusable. After you recover the cluster, rejoin the connections.

Disaster recovery

*   Two clusters must be connected to failover and failback applications
*   Reconnect to another cluster

Backup & Restore

*   One hub cluster and more than one spoke clusters can be connected easily to manage application backups in multi clusters.
*   Reconnect to the hub or spoke

If the cluster recovers before the expiration time, then no action is needed as the cluster rejoins the connection automatically. If the cluster recovers after the client cert expires, then clean the connection and setup again to rejoin the recovered cluster. You can get the client cert effective time from the connection CR status, for example:

    oc get connection <connection_name> -n ibm-spectrum-fusion-ns

    - lastTransitionTime: '2024-01-03T10:23:20Z'
      message: >-
        client certificate rotated starting from 2024-01-03 10:18:31 +0000 UTC
        to 2024-01-19 13:12:00 +0000 UTC
      reason: ClientCertificateUpdated
      status: 'True'
      type: ClusterCertificateRotated

Clean the connection and setup the connection between cluster-a and cluster-b again.

1.  Clean the connection:
    
    Delete the connection CR in both cluster-a and cluster-b clusters:
    
    *   In cluster-a
        
            oc delete connection <connection_cr_with_cluster-b_endpoint_in_spec> -n ibm-spectrum-fusion-ns
        
    *   In cluster-b
        
            oc delete connection <connection_cr_with_cluster-a_endpoint_in_spec> -n ibm-spectrum-fusion-ns
        
2.  Set up the connection between clusters again.
    1.  Get the bootstrap token in cluster-a:
        
            kubectl create token isf-application-operator-cluster-bootstrap -n <namespace of isf-application-operator>
        
    2.  Create connection CR and `init` secret in cluster-b with the bootstrap token and API endpoint of cluster-a.
        
        For example, create `init` secret in cluster-b:
        
            apiVersion: v1
            kind: Secret
            metadata:
              name: init-<cluster-a>
              namespace: ibm-spectrum-fusion-ns 
            stringData:
              apiserver: <cluster-a api endpoint>
              bootstrapToken: <token generated in step 2.1>
            
        
3.  Create connection CR with this `init` secret in `spec` in cluster-b:
    
        apiVersion: application.isf.ibm.com/v1
        kind: Connection
        metadata:
          name: <connection-name>
          namespace: ibm-spectrum-fusion-ns 
        spec:
          remoteCluster
            apiEndpoint: <cluster-a api endpoint>
            initSecretName: init-<cluster-a>
        
    

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Red Hat OpenShift Data Foundation Object Storage Device (OSD) failure
=====================================================================

For any kind of failed storage devices on the clusters backed by local storage devices, you must replace the Red Hat® OpenShift® Data Foundation Object Storage Device (OSD).

If you encounter this issue, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

Before you begin

Red Hat recommends that replacement OSD devices are configured with similar infrastructure and resources to the device being replaced.

You can replace an OSD in Red Hat OpenShift Data Foundation deployed using local storage devices on the following infrastructures:

*   Bare Metal
*   VMware with local deployment
*   SystemZ
*   IBM Power Systems

Procedure

Do the following steps to check for the occurrence of Red Hat OpenShift Data Foundation OSD failure:

1.  Set the Red Hat OpenShift Data Foundation cluster to maintenance:
    
        oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode=true"
        
    
    Example output:
    
        [root@fu40 ~]# oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode=true"
        odfcluster.odf.isf.ibm.com/odfcluster labeled
        
    
2.  Identify the failed OSD:
    
    check whether the OSD failed by using any of the following methods:
    
    *   Log in to Red Hat OpenShift Container Platform web console and go to your storage system details page.
    *   In the Overview > Block and File tab, check the Status section for any warning in the Storage cluster.
    *   If the warnings indicate OSD down or degraded, then contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") to replace the Red Hat OpenShift Data Foundation failed OSD for your storage node in an internal-attached environment.
        
        Example warning message:
        
        1 osds down
        1 host (1 osds) down
        Degraded data redundancy: 333/999 objects degaded (33.333%), 81 pgs degraded
        
    *   Log in to IBM Storage Fusion user interface.
    *   Go to Data foundation page and check for warnings in the Health section for storage cluster.
    
    Alternatively, you can use the oc command to identify the OSD:
    
        oc get -n openshift-storage pods -l app=rook-ceph-osd -o wide
    
    Sample output:
    
        [root@fu40 ~]# oc get -n openshift-storage pods -l app=rook-ceph-osd -o wide
        NAME                               READY   STATUS             RESTARTS      AGE   IP             NODE   NOMINATED NODE   READINESS GATES
        rook-ceph-osd-0-6c99fc999b-2s9mr   1/2     CrashLoopBackOff   5 (17s ago)   17m   10.128.4.216   fu49   <none>           <none>
        rook-ceph-osd-1-764f9cff48-6gkg9   2/2     Running            0             16m   10.131.2.18    fu47   <none>           <none>
        rook-ceph-osd-2-5d9d5984dc-8gkrz   2/2     Running            0             16m   10.129.2.53    fu48   <none>           <none>
    
    In this example, `rook-ceph-osd-0-6c99fc999b-2s9mr` needs to be replaced and `fu49` is the Red Hat OpenShift Container Platform node on which the OSD is scheduled. And the failed OSD id is `0`.
    
    You can view the OSD details as well `ceph osd df` in the Ceph tools. And the failed OSD id is the same as in previous step.
    
3.  Scale down the OSD deployment
    
    Scale down the OSD deployment replica to 0
    
    Verify the OSD id from previous step, the `rook-ceph-osd-0-6c99fc999b-2s9mr` and pod id is `0`.
    
        osd_id_to_remove=<replace-it-with-osd-id>
        oc scale -n openshift-storage deployment rook-ceph-osd-${osd_id_to_remove} --replicas=0
    
    Example output:
    
        [root@fu40 ~]# osd_id_to_remove=0
        [root@fu40 ~]#  oc scale -n openshift-storage deployment rook-ceph-osd-${osd_id_to_remove} --replicas=0
        deployment.apps/rook-ceph-osd-0 scaled
    
    Waiting to the rook-ceph-osd pod is terminated
    
    Run the oc command to terminate `rook-ceph-osd` pod.
    
        oc get -n openshift-storage pods -l ceph-osd-id=${osd_id_to_remove}
    
    Example output:
    
        [root@fu40 ~]#  oc get -n openshift-storage pods -l ceph-osd-id=${osd_id_to_remove}
        NAME                               READY   STATUS        RESTARTS   AGE
        rook-ceph-osd-0-6c99fc999b-2s9mr   0/2     Terminating   6          20m
    
    Note: If the `rook-ceph-osd` pod is in terminating state and taking more time, then use the force option to delete the pod.
    
        oc delete -n openshift-storage pod rook-ceph-osd-0-6c99fc999b-2s9mr --grace-period=0 --force
    
    Example output:
    
        [root@fu40 ~]# oc delete -n openshift-storage pod rook-ceph-osd-0-6c99fc999b-2s9mr --grace-period=0 --force
        warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
        pod "rook-ceph-osd-0-6c99fc999b-2s9mr" force deleted
    
    Verify whether `rook-ceph-osd` is terminated.
    
        [root@fu40 ~]#  oc get -n openshift-storage pods -l ceph-osd-id=${osd_id_to_remove}
        No resources found in openshift-storage namespace.
    
4.  Remove the old OSD from the cluster.
    
    Delete any old `ocs-osd-removal` jobs
    
    Run the oc command to delete `ocs-osd-removal` jobs.
    
        oc delete -n openshift-storage job ocs-osd-removal-job
    
    Remove the old OSD from the cluster
    
    Ensure that you set the correct `osd_id_to_remove`.
    
    The FORCE\_OSD\_REMOVAL value must be changed to `true` in clusters that only have three OSDs, or clusters with insufficient space to restore all three replicas of the data after the OSD is removed.
    
    *   More than three OSDs
        
            oc process -n openshift-storage ocs-osd-removal -p FAILED_OSD_IDS=${osd_id_to_remove} FORCE_OSD_REMOVAL=true |oc create -n openshift-storage -f -
        
    *   Only three OSDs or insufficient space (force delete)
        
            oc process -n openshift-storage ocs-osd-removal -p FAILED_OSD_IDS=${osd_id_to_remove} FORCE_OSD_REMOVAL=true |oc create -n openshift-storage -f -
        
        Example output:
        
            [root@fu40 ~]# echo $osd_id_to_remove
            0
        
    
    Verify the OSD is removed
    
    Wait for the `ocs-osd-removal-job` pod is completed.
    
        [root@fu40 ~]#  oc get pod -l job-name=ocs-osd-removal-job -n openshift-storage
        NAME                        READY   STATUS      RESTARTS   AGE
        ocs-osd-removal-job-s4vhc   0/1     Completed   0          24s
    
    Double confirm the logs.
    
        [root@fu40 ~]#  oc logs -l job-name=ocs-osd-removal-job -n openshift-storage --tail=-1 | egrep -i 'completed removal'
        2022-11-25 16:08:49.858109 I | cephosd: completed removal of OSD 0
    
    The PVC will go to `Pending`, and the pv will be `Released`.
    
        openshift-storage   ocs-deviceset-ibm-spectrum-fusion-local-0-data-3nsk8j   Pending                                                                        ibm-spectrum-fusion-local     7m16s
    
        local-pv-a2879220                          600Gi      RWO            Delete           Released   openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b   ibm-spectrum-fusion-local              41m
    
    To locate the worker node, use the oc command to describe pv.
    
    For example, pv host name is fu49 `kubernetes.io/hostname=fu49`.
    
        [root@fu40 ~]# oc describe pv local-pv-a2879220
        Name:              local-pv-a2879220
        Labels:            kubernetes.io/hostname=fu49
                          storage.openshift.com/owner-kind=LocalVolumeSet
                          storage.openshift.com/owner-name=ibm-spectrum-fusion-local
                          storage.openshift.com/owner-namespace=openshift-local-storage
        Annotations:       pv.kubernetes.io/bound-by-controller: yes
                          pv.kubernetes.io/provisioned-by: local-volume-provisioner-fu49-96f64c0f-e5ed-4bb1-b4ff-cad610562f58
                          storage.openshift.com/device-id: scsi-36000c2913ba6a22c66120c73cb1edae6
                          storage.openshift.com/device-name: sdb
        Finalizers:        [kubernetes.io/pv-protection]
        StorageClass:      ibm-spectrum-fusion-local
        Status:            Released
        Claim:             openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b
        Reclaim Policy:    Delete
        Access Modes:      RWO
        VolumeMode:        Block
        Capacity:          600Gi
        Node Affinity:
          Required Terms:
            Term 0:        kubernetes.io/hostname in [fu49]
        Message:
        Source:
            Type:  LocalVolume (a persistent volume backed by local storage on a node)
            Path:  /mnt/local-storage/ibm-spectrum-fusion-local/scsi-36000c2913ba6a22c66120c73cb1edae6
        Events:
          Type     Reason              Age                  From     Message
          ----     ------              ----                 ----     -------
          Warning  VolumeFailedDelete  6m2s (x26 over 12m)  deleter  Error cleaning PV "local-pv-a2879220": failed to get volume mode of path "/mnt/local-storage/ibm-spectrum-fusion-local/scsi-36000c2913ba6a22c66120c73cb1edae6": Directory check for "/mnt/local-storage/ibm-spectrum-fusion-local/scsi-36000c2913ba6a22c66120c73cb1edae6" failed: open /mnt/local-storage/ibm-spectrum-fusion-local/scsi-36000c2913ba6a22c66120c73cb1edae6: no such file or directory
    
    Note: If the ocs-osd-removal-job fails and the pod is not in the expected completed state, check the pod logs for further debugging.
    
    Remove Encryption related configuration
    
    Remove the dm-crypt managed device-mapper mapping from the OSD devices that are removed from the respective Red Hat OpenShift Data Foundation nodes if encryption was enabled during installation.
    
    *   For each of the previously identified nodes, do the following:
        
            oc debug node/<node name>
            chroot /host
            dmsetup ls| grep <pvc name>
        
    *   Remove the mapped device.
        
            cryptsetup luksClose --debug --verbose ocs-deviceset-xxx-xxx-xxx-xxx-block-dmcrypt
        
        Example output:
        
            [root@fu40 ~]# oc debug nodes/fu49
            Starting pod/fu49-debug ...
            To use host binaries, run `chroot /host`
            If you don't see a command prompt, try pressing enter.
            sh-4.4# chroot /host
            sh-4.4# dmsetup ls
            ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt	(253:0)
            sh-4.4# cryptsetup luksClose --debug --verbose ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt
        
            # cryptsetup 2.3.3 processing "cryptsetup luksClose --debug --verbose ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt"
            # Running command close.
            # Locking memory.
            # Installing SIGINT/SIGTERM handler.
            # Unblocking interruption on signal.
            # Allocating crypt device context by device ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt.
            # Initialising device-mapper backend library.
            # dm version   [ opencount flush ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # Detected dm-ioctl version 4.43.0.
            # Detected dm-crypt version 1.21.0.
            # Device-mapper backend running with UDEV support enabled.
            # dm status ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount noflush ]   [16384] (*1)
            # Releasing device-mapper backend.
            # Allocating context for crypt device (none).
            # Initialising device-mapper backend library.
            Underlying device for crypt device ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt disappeared.
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm table ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush securedata ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm deps ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush ]   [16384] (*1)
            # LUKS device header not available.
            # Deactivating volume ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt.
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm status ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount noflush ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm table ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush securedata ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm deps ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # dm table ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush securedata ]   [16384] (*1)
            # dm versions   [ opencount flush ]   [16384] (*1)
            # Udev cookie 0xd4d9390 (semid 0) created
            # Udev cookie 0xd4d9390 (semid 0) incremented to 1
            # Udev cookie 0xd4d9390 (semid 0) incremented to 2
            # Udev cookie 0xd4d9390 (semid 0) assigned to REMOVE task(2) with flags DISABLE_LIBRARY_FALLBACK         (0x20)
            # dm remove ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b-block-dmcrypt  [ opencount flush retryremove ]   [16384] (*1)
            # Udev cookie 0xd4d9390 (semid 0) decremented to 1
            # Udev cookie 0xd4d9390 (semid 0) waiting for zero
        
    
    Find the persistent volume (PV) that need to be deleted
    
    Run the oc command to find the failed pv.
    
        oc get pv -l kubernetes.io/hostname=<failed-osds-worker-node-name>
    
    Example output:
    
        [root@fu40 ~]# oc get pv -l kubernetes.io/hostname=fu49
        NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                                                     STORAGECLASS                REASON   AGE
        local-pv-a2879220   600Gi      RWO            Delete           Released   openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m227b   ibm-spectrum-fusion-local            55m
    
    Delete the released persistent volume (PV)
    
    Run the oc command to delete the released pv.
    
        oc delete pv <pv_name>
    
    Example output:
    
        [root@fu40 ~]#  oc delete pv local-pv-a2879220
        persistentvolume "local-pv-a2879220" delete
    
5.  Add new OSD into the node.
    
    Add a new device physically to the node.
    
    Track the provisioning of persistent volume (PV)s for the devices that match the deviceInclusionSpec.
    
    It can take a few minutes to provision the PVs. Once the PV is identified, it adds itself to the cluster automatically.
    
    *   lvs spec
        
            oc -n openshift-local-storage describe localvolumeset ibm-spectrum-fusion-local
        
        Example output:
        
            ...
            Spec:
            Device Inclusion Spec:
                Device Types:
                disk
                part
                Max Size:  601Gi
                Min Size:  599Gi
            Node Selector:
                Node Selector Terms:
                Match Expressions:
                    Key:       cluster.ocs.openshift.io/openshift-storage
                    Operator:  In
                    Values:
        
    
    Delete the ocs-osd-removal-job
    
    Run the oc command to delete the ocs-osd-removal-job.
    
        ```
        oc delete -n openshift-storage job ocs-osd-removal-job
        ```
        ```
        [root@fu40 ~]# oc delete -n openshift-storage job ocs-osd-removal-job
        job.batch "ocs-osd-removal-job" deleted
        ```
    
6.  Verify that there is a new OSD running
    
    Verify new OSD pod is running
    
    Run the oc command to check the new OSD pod is running.
    
        oc get -n openshift-storage pods -l app=rook-ceph-osd
    
    Example output:
    
        [root@fu40 ~]# oc get -n openshift-storage pods -l app=rook-ceph-osd
        NAME                               READY   STATUS    RESTARTS   AGE
        rook-ceph-osd-0-7f99b8ccd5-ssj5w   2/2     Running   0          7m31s       <<-- This pod
        rook-ceph-osd-1-764f9cff48-6gkg9   2/2     Running   0          64m
        rook-ceph-osd-2-5d9d5984dc-8gkrz   2/2     Running   0          64m
    
    Tip: If the new OSD does not show as Running after a few minutes, restart the rook-ceph-operator pod to force a reconciliation.
    
        oc delete pod -n openshift-storage -l app=rook-ceph-operator
    
    Verify new PVC is created
    
    Run the oc command to check whether the pods are running.
    
        oc get pvc -n openshift-storage
    
    Example output:
    
        [root@fu40 ~]# oc get pvc -n openshift-storage
        NAME                                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                  AGE
        db-noobaa-db-pg-0                                       Bound    pvc-783036b5-ec40-41a7-91e5-9e179fd24cc3   50Gi       RWO            ocs-storagecluster-ceph-rbd   65m  <<--This one
        ocs-deviceset-ibm-spectrum-fusion-local-0-data-04vwvq   Bound    local-pv-b45b1d67                          600Gi      RWO            ibm-spectrum-fusion-local     66m
        ocs-deviceset-ibm-spectrum-fusion-local-0-data-24nj5t   Bound    local-pv-c3de9110                          600Gi      RWO            ibm-spectrum-fusion-local     66m
        ocs-deviceset-ibm-spectrum-fusion-local-0-data-3nsk8j   Bound    local-pv-1c9f3b11                          600Gi      RWO            ibm-spectrum-fusion-local     34m     
        [root@fu40 ~]#
        [root@fu40 ~]# oc get pv
        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                     STORAGECLASS                  REASON   AGE
        local-pv-1c9f3b11                          600Gi      RWO            Delete           Bound    openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-3nsk8j   ibm-spectrum-fusion-local              10m     <<--This one
        local-pv-b45b1d67                          600Gi      RWO            Delete           Bound    openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-04vwvq   ibm-spectrum-fusion-local              68m
        local-pv-c3de9110                          600Gi      RWO            Delete           Bound    openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-24nj5t   ibm-spectrum-fusion-local              68m
        pvc-783036b5-ec40-41a7-91e5-9e179fd24cc3   50Gi       RWO            Delete           Bound    openshift-storage/db-noobaa-db-pg-0                                       ocs-storagecluster-ceph-rbd            65m
    
    Verify the OSD Encryption settings
    
    If cluster wide encryption is enabled, ensure that the crypt keyword is next to the ocs-deviceset name.
    
        oc debug node/<new-node-name>  -- chroot /host lsblk -f
    
        oc debug node/<new-node-name> -- chroot /host dmsetup ls
    
    Example output:
    
        [root@fu40 ~]# oc debug node/fu49 -- chroot /host lsblk -f
        Starting pod/fu49-debug ...
        To use host binaries, run `chroot /host`
        NAME                                                                  FSTYPE      LABEL                                           UUID                                 MOUNTPOINT
        loop1                                                                 crypto_LUKS pvc_name=ocs-deviceset-ibm-spectrum-fusion-loca 6a8244eb-55d6-48cc-8e68-33436e512bc6
        loop2                                                                 crypto_LUKS pvc_name=ocs-deviceset-ibm-spectrum-fusion-loca fa228ec1-0b1d-43ad-8707-9ecd38bfb1f8
        sda
        |-sda1
        |-sda2                                                                vfat        EFI-SYSTEM                                      A084-4057
        |-sda3                                                                ext4        boot                                            7d757098-d548-4b7b-8c9a-3dd4f34ceca1 /boot
        `-sda4                                                                xfs         root                                            1cd39805-6936-458d-ae8c-39313bb71c95 /sysroot
        sdc                                                                   crypto_LUKS pvc_name=ocs-deviceset-ibm-spectrum-fusion-loca fa228ec1-0b1d-43ad-8707-9ecd38bfb1f8
        `-ocs-deviceset-ibm-spectrum-fusion-local-0-data-3nsk8j-block-dmcrypt
        sr0
        
        Removing debug pod ...
        [root@fu40 ~]# oc debug node/fu49 -- chroot /host dmsetup ls
        Starting pod/fu49-debug ...
        To use host binaries, run `chroot /host`
        ocs-deviceset-ibm-spectrum-fusion-local-0-data-3nsk8j-block-dmcrypt	(253:0)
        
        Removing debug pod ...
    
    Note: If verification steps fail, then contact Red Hat support.
    
    Exit maintenance mode
    
    Run the oc command to exit maintenance mode after all steps are completed.
    
        oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode-"
    
    Example output:
    
        [root@fu40 ~]# oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode-"
        odfcluster.odf.isf.ibm.com/odfcluster unlabeled
    
7.  Go to Data foundation page in IBM Storage Fusion user interface and check the health of the Storage cluster in the Health section.

**Parent topic:** [IBM Fusion Data Foundation service error scenarios](../tshooting/sf_odf_troubleshooting.html "Use these troubleshooting information to know the problem and workaround when install or configure IBM Fusion Data Foundation service.")


Red Hat OpenShift Data Foundation storage node failure
======================================================

You can do a node replacement proactively for an operational node and reactively for a failed node. For a failed node backed by local storage devices, you must replace the Red Hat® OpenShift® Data Foundation storage node.

Before you begin

Red Hat recommends that replacement nodes are configured with similar infrastructure, resources, and disks as the node planned for replacement.

Note: [Contact IBM Support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") before you proceed with any of these fixes.

Procedure

Do the following steps to check for the occurrence of Red Hat OpenShift Data Foundation storage node failure and identify the failed node:

1.  Set the Red Hat OpenShift Data Foundation cluster to maintenance mode:
    
        oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode=true"
        
    
    Example output:
    
        [root@fu40 ~]# oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode=true"
        odfcluster.odf.isf.ibm.com/odfcluster labeled
        
    
2.  Identify the failed node:
    
    1.  Log in to IBM Storage Fusion user interface.
    2.  Go to Data foundation page and check for warning in the Health section for storage cluster.
    
    Alternatively, you can use the oc command to identify the node:
    
        oc get node -l cluster.ocs.openshift.io/openshift-storage=
    
    Sample output:
    
        [root@fu71-f09-vm3 ~]# oc get node -l cluster.ocs.openshift.io/openshift-storage=
        NAME                               STATUS     ROLES    AGE   VERSION
        f09-prc4m-worker-cluster-b-9chb5   NotReady   worker   27d   v1.24.0+4f0dd4d
        f09-prc4m-worker-cluster-c-mfb77   Ready      worker   31d   v1.24.0+4f0dd4d
        f09-prc4m-worker-cluster-d-r5bxx   Ready      worker   27d   v1.24.0+4f0dd4d
    
3.  Identify the failed mon (if any) and Red Hat OpenShift Dedicated pods that are running in the node, which is planned for replacement:
    
    In an operational storage node environment:
    
        oc get pods -n openshift-storage -o wide | grep -i <node_name>
    
4.  If the storage node failed in a failed storage node, there is no `node_name` for the failed pods. Filter the `pending` pods instead.
    
        oc get pods -n openshift-storage -o wide | grep -i pending
        
    
    Example output: The mon deployment is `rook-ceph-mon-d`, and the Red Hat OpenShift Dedicated deployment is `ook-ceph-osd-0`.
    
        [root@fu71-f09-vm3 ~]# oc get pods -n openshift-storage -o wide | grep -i pending
         rook-ceph-mon-d-67686857d7-zv62c                                  0/2     Pending     0          8m50s   <none>            <none>                             <none>           <none>
         rook-ceph-osd-0-75b954c9bf-62xm4                                  0/2     Pending     0          8m50s   <none>            <none>                             <none>           <none>
        
    
5.  Remove the failed objects.
    
    Remove the failed node from `odfcluster` CR
    
        spec:
          autoScaleUp: false
          creator: CreatedByFusion
          deviceSets:
          - capacity: "0"
            count: 3
            name: ocs-deviceset-ibm-spectrum-fusion-local
            storageClass: ibm-spectrum-fusion-local
          encryption:
            keyManagementService: {}
          localVolumeSetSpec:
            deviceTypes:
            - disk
            - part
            size: 2Ti
          storageNodes:
          - f09-prc4m-worker-cluster-d-r5bxx
          - f09-prc4m-worker-cluster-c-mfb77
          - f09-prc4m-worker-cluster-b-9chb5 <<-- this one, remove this line.
        
    
    Remove the mon and Red Hat OpenShift Dedicated pods
    
    Scale down the deployments of the identified pods. The mon deployment is `rook-ceph-mon-d` and the Red Hat OpenShift Dedicated deployment is `rook-ceph-osd-0`.
    
        oc scale deployment rook-ceph-mon-d --replicas=0 -n openshift-storage
        oc scale deployment rook-ceph-osd-0 --replicas=0 -n openshift-storage
        
    
    Ensure that you confirm the values of `mon_id` and `osd_id`.
    
    Remove the crashcollector pods
    
    Remove the crashcollector pods ( if any). You must put scale replica to 0.
    
        oc scale deployment --selector=app=rook-ceph-crashcollector,node_name=<node_name>  --replicas=0 -n openshift-storage
        
    
    Mark the failed node as unschedulable
    
    Mark the node as `SchedulingDisabled`.
    
        oc adm cordon <node_name>
    
    Example command and output:
    
        oc adm cordon f09-prc4m-worker-cluster-b-9chb5
        node/f09-prc4m-worker-cluster-b-9chb5 cordoned
    
        oc get node -l cluster.ocs.openshift.io/openshift-storage=
    
    NAME                               STATUS                        ROLES    AGE   VERSION
    f09-prc4m-worker-cluster-b-9chb5   NotReady,SchedulingDisabled   worker   28d   v1.24.0+4f0dd4d
    f09-prc4m-worker-cluster-c-mfb77   Ready                         worker   31d   v1.24.0+4f0dd4d
    f09-prc4m-worker-cluster-d-r5bxx   Ready                         worker   27d   v1.24.0+4f0dd4d
    
    Remove the pods which are in Terminating state
    
    This step is for the failed storage node. You can ignore this step if your are removing an operational node.
    
        oc get pods -A -o wide | grep -i <node_name> |  awk '{if ($4 == "Terminating") system ("oc -n " $1 " delete pods " $2  " --grace-period=0 " " --force ")}'
    
    Example command and output:
    
        oc get pods -A -o wide | grep -i f09-prc4m-worker-cluster-b-9chb5 |  awk '{if ($4 == "Terminating") system ("oc -n " $1 " delete pods " $2  " --grace-period=0 " " --force ")}'
    
    warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
    pod "isf-data-protection-operator-controller-manager-5c7cf574d5ms4xx" force deleted
    
    Drain the node
    
        oc adm drain <node_name> --force --delete-emptydir-data=true --ignore-daemonsets
    
    Example command and output:
    
        oc adm drain f09-prc4m-worker-cluster-b-9chb5 --force --delete-emptydir-data=true --ignore-daemonsets
        node/f09-prc4m-worker-cluster-b-9chb5 already cordoned
    
    WARNING: ignoring DaemonSet-managed Pods: openshift-cluster-csi-drivers/vmware-vsphere-csi-driver-node-7696f, openshift-cluster-node-tuning-operator/tuned-fk949, openshift-dns/dns-default-gvv4m, openshift-dns/node-resolver-t5dk8, openshift-image-registry/node-ca-wtnp9, openshift-ingress-canary/ingress-canary-kgxts, openshift-local-storage/diskmaker-discovery-qkmh7, openshift-local-storage/diskmaker-manager-m9q42, openshift-machine-config-operator/machine-config-daemon-252j8, openshift-monitoring/node-exporter-cghwc, openshift-multus/multus-additional-cni-plugins-mkz4m, openshift-multus/multus-bz789, openshift-multus/network-metrics-daemon-57v5r, openshift-network-diagnostics/network-check-target-n6bhw, openshift-sdn/sdn-fhp47, openshift-storage/csi-cephfsplugin-5vsp9, openshift-storage/csi-rbdplugin-bfpfs
    node/f09-prc4m-worker-cluster-b-9chb5 drained
    
    Delete the node
    
    Delete the failed node:
    
        oc delete node <node_name>
    
    If you do not want to destroy this node for test purpose, you can remove the storage label `cluster.ocs.openshift.io/openshift-storage=`.
    
        oc label nodes/f09-prc4m-worker-cluster-b-9chb5 cluster.ocs.openshift.io/openshift-storage-
    
6.  Add new OpenShift compute nodes.
    1.  Create a new compute node and ensure that the new node is in ready state.
    2.  Update new node info in `odfcluster` CR.
        
        Edit the odfcluster cr and add new node name in `StorageNodes`
        
            oc edit odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster
            
        
            storageNodes:
                - f09-prc4m-worker-cluster-b-djplp  <<--- new node
                - f09-prc4m-worker-cluster-d-r5bxx
                - f09-prc4m-worker-cluster-c-mfb77
            
        
        Verify whether the new nodes are labeled successfully as storage node.
        
            oc get node -l cluster.ocs.openshift.io/openshift-storage
        
            NAME                               STATUS   ROLES    AGE   VERSION
            f09-prc4m-worker-cluster-b-djplp   Ready    worker   27d   v1.24.0+4f0dd4d <<-- this node
            f09-prc4m-worker-cluster-c-mfb77   Ready    worker   31d   v1.24.0+4f0dd4d
            f09-prc4m-worker-cluster-d-r5bxx   Ready    worker   27d   v1.24.0+4f0dd4d
        
        Verify whether the new PVs are created automatically. The local PV gets created automatically in a short time.
        
            oc get pv | grep Available
        
        local-pv-e97b23d7                          2Ti        RWO            Delete           Available                                                                             ibm-spectrum-fusion-local              2m49s
        
    3.  Replace the failed Red Hat OpenShift Dedicated disks.
        
        Remove the failed Red Hat OpenShift Dedicated from the cluster. You can also specify multiple failed ODs. Use the correct `failed_osd_id`.
        
        The `failed_osd_id` is the integer in the pod name immediately after the `rook-ceph-osd` prefix. You can add comma separated Red Hat OpenShift Dedicated IDs in the command to remove more than one Red Hat OpenShift Dedicated, for example, `FAILED_OSD_IDS=0,1,2`.
        
        Remove the failed Red Hat OpenShift Dedicated:
        
            
            oc process -n openshift-storage ocs-osd-removal \
            -p FAILED_OSD_IDS=<failed_osd_id> FORCE_OSD_REMOVAL=true | oc create -n openshift-storage -f -
        
        Example output:
        
            
            [root@fu71-f09-vm3 ~]# oc process -n openshift-storage ocs-osd-removal -p FAILED_OSD_IDS=0 FORCE_OSD_REMOVAL=true | oc create -n openshift-storage -f -
            Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "operator" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "operator" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "operator" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "operator" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
            job.batch/ocs-osd-removal-job created
        
        Check the status of the `ocs-osd-removal-job` pod to verify whether the Red Hat OpenShift Dedicated got removed successfully. A status of Completed confirms that the Red Hat OpenShift Dedicated removal job succeeded.
        
            oc get pod -l job-name=ocs-osd-removal-job -n openshift-storage
        
        Example output:
        
            [root@fu71-f09-vm3 ~]# oc get pod -n openshift-storage  | grep ocs-osd-removal
            ocs-osd-removal-job-ls65l                                         0/1     Completed   0          23s
            
        
        Ensure that the Red Hat OpenShift Dedicated removal is completed:
        
            oc logs -l job-name=ocs-osd-removal-job -n openshift-storage --tail=-1 | egrep -i 'completed removal'
            
            
        
        Example output:
        
            [root@fu71-f09-vm3 ~]# oc logs -l job-name=ocs-osd-removal-job -n openshift-storage --tail=-1 | egrep -i 'completed removal'
            2022-11-24 15:19:28.750910 I | cephosd: completed removal of OSD 0
        
        Delete the `ocs-osd-removal-job`:
        
            oc delete -n openshift-storage job ocs-osd-removal-job
        
        Delete the `Released PV` which is attached to the previous node.
        
            oc get pv | grep -i released
        
            
            local-pv-e4a12175                          2Ti        RWO            Delete           Released    openshift-storage/ocs-deviceset-ibm-spectrum-fusion-local-0-data-1m6gnz   ibm-spectrum-fusion-local              3h34m
            [root@fu71-f09-vm3 ~]# oc delete pv local-pv-e4a12175
            persistentvolume "local-pv-e4a12175" deleted
            
        
7.  Recover the failed objects.
    1.  Restart the mon deployment/pod:
        1.  Update the nodeSelector in deployment with new node.
            
                oc edit deployment -n openshift-storage rook-ceph-mon-d
            
                nodeSelector:
                        kubernetes.io/hostname: f09-prc4m-worker-cluster-b-djplp <<--new node
            
        2.  Scale the replica to 1 and wait till the mon pods are in running state.
            
                oc scale deployment rook-ceph-mon-d --replicas=1 -n openshift-storage
                
            
            Example:
            
                oc scale deployment rook-ceph-mon-d --replicas=1 -n openshift-storage
                    deployment.apps/rook-ceph-mon-d scaled
            
                
                [root@fu71-f09-vm3 ~]# oc get pod -n openshift-storage | grep mon
                    rook-ceph-mon-a-5bbb9dd98b-z54fx                                  2/2     Running     0          4m45s
                    rook-ceph-mon-b-7fdd8f958b-lk9g2                                  2/2     Running     0          5m18s
                    rook-ceph-mon-d-6945fbbfc5-nhhw8                                  2/2     Running     0          5m41s
            
    2.  Verify the Red Hat OpenShift Dedicated pods.
        
        Wait till all the pods are in running state.
        
            oc get pods -o wide -n openshift-storage| grep osd
            
        
            [root@fu71-f09-vm3 ~]# oc get pods -o wide -n openshift-storage| grep osd
            rook-ceph-osd-0-d559cc4fb-xspr8                                   2/2     Running     0          4m58s   10.129.6.49       f09-prc4m-worker-cluster-b-djplp   <none>           <none>
            rook-ceph-osd-1-6df7f9c669-n94md                                  2/2     Running     0          5m20s   10.128.6.95       f09-prc4m-worker-cluster-d-r5bxx   <none>           <none>
            rook-ceph-osd-2-5c5d48ff7c-sdd7l                                  2/2     Running     0          5m17s   10.129.4.125      f09-prc4m-worker-cluster-c-mfb77   <none>           <none>
            rook-ceph-osd-prepare-1bf7dd3d71fe899383e625dd0c27ea37-x9vtk      0/1     Completed   0          4h8m    10.128.6.79       f09-prc4m-worker-cluster-d-r5bxx   <none>           <none>
            rook-ceph-osd-prepare-24272b5641dc95baffc7932d78894e3c-zhz8m      0/1     Completed   0          5m24s   10.129.6.48       f09-prc4m-worker-cluster-b-djplp   <none>           <none>
            rook-ceph-osd-prepare-6f3c3b4626ec9888b6fbe5597afd55ea-zh7cp      0/1     Completed   0          4h8m    10.129.4.113      f09-prc4m-worker-cluster-c-mfb77   <none>           <none>
            
        
    3.  Verify the Red Hat OpenShift Dedicated encryption settings. If cluster wide encryption is enabled, make sure the “crypt” keyword beside the ocs-deviceset name(s)
        
            oc debug node/<new-node-name> -- chroot /host dmsetup ls
        
        Example output:
        
            # oc debug node/fu47 -- chroot /host dmsetup ls
            Starting pod/fu47-debug ...
            To use host binaries, run `chroot /host`
            ocs-deviceset-sc-lvs-0-data-0clwxf-block-dmcrypt	(253:0)
        
        If verification fails, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .
        
8.  Exit maintenance mode after all steps are completed.
    
        oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode-"
        
    
    Example output:
    
        [root@fu40 ~]# oc label odfclusters.odf.isf.ibm.com -n ibm-spectrum-fusion-ns odfcluster "odf.isf.ibm.com/maintenanceMode-"
        odfcluster.odf.isf.ibm.com/odfcluster unlabeled
        
    
9.  Go to Data foundation page in IBM Storage Fusion user interface and check the health of the Storage cluster in the Health section.

**Parent topic:** [IBM Fusion Data Foundation service error scenarios](../tshooting/sf_odf_troubleshooting.html "Use these troubleshooting information to know the problem and workaround when install or configure IBM Fusion Data Foundation service.")


IBM Fusion Data Foundation service error scenarios
==================================================

Use these troubleshooting information to know the problem and workaround when install or configure IBM Fusion Data Foundation service.

*   [Red Hat OpenShift Data Foundation storage node failure](sf_odf_storage_node_failure.html "You can do a node replacement proactively for an operational node and reactively for a failed node. For a failed node backed by local storage devices, you must replace the Red Hat OpenShift Data Foundation storage node.")
*   [Red Hat OpenShift Data Foundation Object Storage Device (OSD) failure](sf_odf_osd_failure.html "For any kind of failed storage devices on the clusters backed by local storage devices, you must replace the Red Hat OpenShift Data Foundation Object Storage Device (OSD).")
*   [Local storage operator unable to find candidate storage nodes](#sf_odf_troubleshooting__iso)
*   [IBM Fusion Data Foundation capacity cannot be loaded](#sf_odf_troubleshooting__capacity)
*   [IBM Fusion Data Foundation cluster fails due to pending StorageClusterPreparing stage](#sf_odf_troubleshooting__pvc)

Local storage operator unable to find candidate storage nodes
-------------------------------------------------------------

Problem statement

When you configure a IBM Fusion Data Foundation cluster, you do not find any candidate storage nodes.

Cause

When you configure IBM Fusion Data Foundation cluster, only compute nodes with available disks (SSD/NVMe or HDD) get displayed in the Data Foundation page of IBM Storage Fusion user interface. The following nodes get filtered out and do not display on the screen:

*   Nodes have SSD/NVMe or HDD disks but they are not in available state
*   The selected disk properties are not present in current node. For example, disk size or disk type.
*   The total disk count (with same disk size, disk type) is less than 3.

Steps to verify whether you have the correct storage node candidates

1.  In Red Hat OpenShift Container Platform console, go to Operators > Installed Operators.
2.  Verify whether the `LocalStorage` operator is installed successfully.
3.  Run the following command to get all the worker nodes:
    
        oc get node -l node-role.kubernetes.io/worker=
    
4.  Run the following command to check if discovery results are created for all worker nodes.
    
        oc get localvolumediscoveryresult -n openshift-local-storage
    
5.  Run the following command to confirm that none of the nodes have a IBM Fusion Data Foundation storage label:
    
        oc get node -l cluster.ocs.openshift.io/openshift-storage=
    

Note: In Linux on IBM zSystems platform, disks might be formatted and partitioned first. For more information about this behavior, see [Red Hat OpenShift Data Foundation on IBM Z and IBM LinuxONE - Reference Architecture](https://www.ibm.com/docs/en/linux-on-systems?topic=architecture-storage "(Opens in a new tab or window)") section 4.1.1 and 4.1.2.

If all the above checks pass, but the node still could not be seen in the IBM Storage Fusion user interface, then contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

IBM Fusion Data Foundation capacity cannot be loaded
----------------------------------------------------

If you encounter this issue in the Data foundation page of IBM Storage Fusion user interface, then contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

IBM Fusion Data Foundation cluster fails due to pending StorageClusterPreparing stage
-------------------------------------------------------------------------------------

Problem statement

Here, the PVC is not created and the `odfcluster` status shows as follows:

    
    conditions:
      - lastTransitionTime: "2022-12-01T15:09:47Z"
        message: storagecluster is not ready,install pending
        reason: StorageClusterPreparing
        status: "False"
        type: Ready
      phase: InProgress
      replica: 1
    

Diagnosis and resolution

To diagnose and fix the problem, do the following steps:

1.  Run the following command to open the `storagecluster` CR:
    
        oc get storageclusters.ocs.openshift.io -n openshift-storage ocs-storagecluster -o yaml
    
2.  Check whether the output of the command shows the following error message in the status:
    
    ConfigMap "ocs-kms-connection-details" not found'
    
    Output example:
    
        
        status:
        conditions:
        - lastHeartbeatTime: "2023-03-29T08:01:10Z"
        lastTransitionTime: "2023-03-29T07:49:47Z"
        message: 'Error while reconciling: some StorageClasses were skipped while waiting
        for pre-requisites to be met: [ocs-storagecluster-cephfs,ocs-storagecluster-ceph-rbd]'
        reason: ReconcileFailed
        status: "False"
        type: ReconcileComplete
    
3.  If you notice the error message, check the root-operator logs with the following command:
    
        oc logs -n openshift-storage $(oc get pod -n openshift-storage -l app=rook-ceph-operator -o name)
    
    Example output:
    
    2023-03-29 07:55:41.297073 E | ceph-cluster-controller: failed  to reconcile CephCluster  "openshift-storage/ocs-storagecluster-cephcluster". failed to reconcile
    cluster "ocs-storagecluster-cephcluster": failed to configure local ceph  cluster: failed to perform validation before cluster creation: failed  to validate kms connection details: failed to get backend version:  failed to list vault system mounts: Error making API
    request.
    URL: GET https://9.9.9.75:8200/v1/sys/mounts
    Code: 403. Errors: \* permission denied
    

*   **[Red Hat OpenShift Data Foundation storage node failure](../tshooting/sf_odf_storage_node_failure.html)**  
    You can do a node replacement proactively for an operational node and reactively for a failed node. For a failed node backed by local storage devices, you must replace the Red Hat OpenShift Data Foundation storage node.
*   **[Red Hat OpenShift Data Foundation Object Storage Device (OSD) failure](../tshooting/sf_odf_osd_failure.html)**  
    For any kind of failed storage devices on the clusters backed by local storage devices, you must replace the Red Hat OpenShift Data Foundation Object Storage Device (OSD).
*   **[Known issues and limitations](../tshooting/df_ceph_knownissues.html)**  
    Known issues and limitations in IBM Fusion Data Foundation.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Regional-DR issues
==================

Use these troubleshooting information to resolve issues when you work with Regional-DR.

Secondary does not reflect the expanded PVC size after volume expansion
-----------------------------------------------------------------------

Problem statement

The secondary site does not reflect the expanded PVC size after volume expansion on the primary site. PVC size is not synchronized between the two sites

Resolution

You can ignore this issue as the fileset reflects the expanded quota, and the application writes data with the expanded size.

Incorrect used capacity gets displayed on the secondary site after failover
---------------------------------------------------------------------------

Problem statement

After the failover of the application, PVC size is not synchronized between the two sites.

Resolution

Regional-DR is a technical preview in this release and it supports PVCs only with reclaim policy retain. You can ignore this issue as it does not impact the actual PVC size or you can clean up the PVC and PV on the secondary site.

Regional DR configuration fails between the two racks with an error in creating secret
--------------------------------------------------------------------------------------

Problem statement

This issue occurs only when a Backup & Restore is already setup and the snippet is copied from Spoke to the Hub cluster to setup the Regional DR.

Resolution

As a workaround, copy the code snippet from the Backup & Restore Hub cluster and pasted into the Backup & Restore Spoke cluster to setup the Regional DR.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Reconnecting OpenShift Container Platform cluster
=================================================

The OpenShift® Container Platform cluster that is connected in a disaster recovery set up or a Backup & Restore service with Hub and Spoke can face disasters, leading to a temporary unusable state. After recovery, it can still display an `Unhealthy` or `UnManaged` status in remote clusters. In such instances, this cluster must be reconnected to its corresponding connections.

About this task
---------------

If the cluster recovers before the expiration time, the cluster rejoins the connection automatically and no action is needed. However, if the cluster recovers after the expiration of the client cert, the connection must be cleaned and setup to rejoin the recovered cluster.

`connection-<id>`

Procedure
---------

1.  Save the name of connection CR in cluster-a (a cluster in the hub and spoke or DR clusters) in `connection-<id>` format.
2.  Clean the connection.
    
    For the procedure to clean the connection, see [Disabling the connection](../backuprestore/sf_disable_connection.html "Disable the connection between Hub and Spoke.").
    
3.  Setup the connection between clusters.
    1.  Get the bootstrap token in hub cluster.
        
            oc create token isf-application-operator-cluster-bootstrap -n <Fusion Namespace of cluster-b>
        
        Note: The version of oc command line must be greater or equal to 4.11.
        
    2.  Create init secret in cluster-a with the bootstrap token and API endpoint of cluster-b.
        
        For example:
        
            
            apiVersion: v1
            kind: Secret
            metadata:
               name: <Init Secret Name>
               namespace: <Fusion Namespace of cluster-a>
            stringData:
               apiserver: <cluster-b API Endpoint>
               bootstrapToken: <cluster-b Token Generated in Step 3.a>
        
    3.  Create connection CR in cluster-a with this init secret in spec:
        
            
            apiVersion: application.isf.ibm.com/v1
            kind: Connection
            metadata:
              name: <Connection Name Saved in Step 1>
              namespace: <Fusion Namespace of cluster-a>
            spec:
              remoteCluster:
                apiEndpoint: <cluster-b API Endpoint>
                initSecretName: <Init Secret Name>
                connectionOperatorNamespace: <Fusion Namespace in cluster-b>
        

**Parent topic:** [Backup and Restore](../backuprestore/sf_backuprestore.html "Using the IBM Storage Fusion Backup & Restore service, you can backup and restore applications and workloads.")


Restore issues
==============

List of restore issues in Backup & Restore service of IBM Storage Fusion.

exec format error
-----------------

Problem statement

Sometimes, you may observe the following error message:

    "exec <executable name>": exec format error

For example:

    The pod log is empty except for this message: exec /filebrowser 

The example error can be due to the wrong architecture of the container. For example, an amd64 container on s390x nodes or an s90x container on amd64 nodes.

Resolution

As a resolution, check whether the container that you want to restore and the local node architecture match.

Restore of namespaces that contains admission webhooks fails
------------------------------------------------------------

Problem statement

Restore of namespaces that contains admission webhooks fails.

Example error in IBM Storage Fusion restore job:

"Failed restore <some  resource>" "BMYBR0003
      RestorePvcsFailed There was an error when  processing the job in the Transaction Manager
      service"

Example error in Velero pod:

level=error msg="Namespace
      domino-platform, resource restore error: error restoring
      certificaterequests.cert-manager.io/domino-platform/hephaestus-buildkit-client-85k2v:
      Internal error occurred: failed calling webhook  "webhook.cert-manager.io": failed to call
      webhook: Post  "https://cert-manager-webhook.domino-platform.svc:443/mutate?timeout=10s\\\\":
      service "cert-manager-webhook" not found"

Resolution

1.  Identify the admission webhooks that is applicable to the namespace being restored:
    
        oc get mutatingwebhookconfigurations
        oc describe mutatingwebhookconfigurations
    
2.  Change the failure Policy parameter from `Fail` to `Ignore` to temporarily disable webhook validation prior to restore:
    
        failurePolicy: Ignore
    

Restore before upgrade fails with a BMYBR0003 error
---------------------------------------------------

Problem statement

When you try to restore backups before upgrade, it fails with a BMYBR0003 error.

Diagnosis

After you upgrade, your jobs may fail:

*   Backup jobs with the status:
    
    "Failed transferring data" "BMYBR0003 There was an error when processing the job in the Transaction Manager service"
    
*   Restore jobs with the status:
    
    "Failed restore <some resource>" "BMYBR0003 There was an error when processing the job in the Transaction Manager service"
    

Confirm the issue in the logs of the manager container of the Data Mover pod.

A sample error message:

2023-07-26T03:39:47Z	ERROR	Failed with error.	{"controller": "guardiancopyrestore", "controllerGroup": "guardian.isf.ibm.com", "controllerKind": "GuardianCopyRestore", "GuardianCopyRestore": {"name":"52a2abfb-ea9b-422f-a60d-fed59527d38e-r","namespace":"ibm-backup-restore"}, "namespace": "ibm-backup-restore", "name": "52a2abfb-ea9b-422f-a60d-fed59527d38e-r", "reconcileID": "c8642f40-c086-413f-a10e-6d6a85531337", "attempt": 2, "total attempts": 3, "error": "EOF"}
github.ibm.com/ProjectAbell/guardian-dm-operator/controllers/util.Retry
	/workspace/controllers/util/utils.go:39
github.ibm.com/ProjectAbell/guardian-dm-operator/controllers/kafka.(\*kafkaWriterConnection).PublishMessage
	/workspace/controllers/kafka/kafka\_native\_connection.go:71
github.ibm.com/ProjectAbell/guardian-dm-operator/controllers.(\*GuardianCopyRestoreReconciler).updateOverallCrStatus
	/workspace/controllers/status.go:191
github.ibm.com/ProjectAbell/guardian-dm-operator/controllers.(\*GuardianCopyRestoreReconciler).doRestore
	/workspace/controllers/guardiancopyrestore\_controller.go:187
github.ibm.com/ProjectAbell/guardian-dm-operator/controllers.(\*GuardianCopyRestoreReconciler).Reconcile
	/workspace/controllers/guardiancopyrestore\_controller.go:92
sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).Reconcile
	/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.14.6/pkg/internal/controller/controller.go:122
sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).reconcileHandler
	/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.14.6/pkg/internal/controller/controller.go:323
sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).processNextWorkItem
	/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.14.6/pkg/internal/controller/controller.go:274
sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).Start.func2.2
	/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.14.6/pkg/internal/controller/controller.go:235

Resolution:

Search for the `guardian-dm-controller-manager` and kill it. A new pod starts in a minute. After the pod reaches a healthy state, retry backup and restores.

"Failed restore snapshot" error occurs with applications using IBM Storage Scale storage PVCs
---------------------------------------------------------------------------------------------

*   A "Failed restore snapshot" error occurs with applications using IBM Storage Scale storage PVCs.
    
    Cause
    
    The "disk quota exceeded" error occurs whenever you restore from an object storage location having applications that use IBM Storage Scale PVC with a size less than 5 GB.
    
    Resolution
    
    Increase the IBM Storage Scale PVC size to a minimum of 5 GB and do a backup and restore operation.
    

Cannot restore multiple namespaces to a single alternative namespace
--------------------------------------------------------------------

You cannot restore multiple namespaces to a single alternative namespace. If you attempt such a restore, then the job fails. Example transaction manager log:

023-06-27 15:05:53,633\[TM\_5\]\[c2a4b768-2acc-4169-9c5f-f88e60f3be2b\]\[restoreguardian:resre\_application\_w\_recipe Line 172\]\[INFO\] - altNS: ns2, number of ns: 2

2023-06-27 15:05:53,633\[TM\_5\]\[c2a4b768-2acc-4169-9c5f-f88e60f3be2b\]\[restoreguardian:restore\_application\_w\_recipe Line 176\]\[ERROR\] - Alternate namespace specified, but more than one namespace was backed up

Restore to a cluster that does not have an identical storage class as the source cluster
----------------------------------------------------------------------------------------

You cannot restore to a cluster that does not have an identical storage class as the source cluster. However, the transaction manager still attempts to create PVCs with the non-existent storage class on the spoke cluster and eventually fails with `Failed restore snapshot` status.

Applications page does not show the details of the application
--------------------------------------------------------------

Problem statement

The new backed up applications page does not show the details of the application when you upgrade IBM Storage Fusion to the latest version while leaving the Backup & Restore service in the older version.

Resolution

As a resolution, it is recommended to upgrade the Backup & Restore service to the latest version after an IBM Storage Fusion upgrade.

S3 buckets must not enable expiration policies
----------------------------------------------

Backup & Restore currently uses restic to move the Velero backups to repositories on S3 buckets. When a restic repository gets initialized, it creates a configuration file and several subdirectories for its snapshots. As restic does not update it after initialization, the modification timestamp of this configuration file is never updated.

If you configure an expiration policy on the container, the restic configuration objects get deleted at the end of the expiration period. If you have an archive rule set, the configuration objects get archived after the time defined in the rule. In either case, the configuration is no longer accessible and subsequent backup and restore operations fail.

Note: The S3 buckets must not enable expiration policies. Also, the bucket must not have an archive rule set.

Virtual machine restore failure
-------------------------------

Problem statement

By default, the VirtualMachineClone objects are not restored due to the following unexpected behavior:

*   If you create a VirtualMachineClone object and delete the original virtual machine, then the restore fails because the object gets rejected.
*   If you create a VirtualMachineClone object and then delete the clone virtual machine, then the restore fails because the Virtualization ignores the `status.phase` "Succeeded" and clones the virtual machine again.
    
    As a result, the clone gets re-created every time you delete it.
    
*   If you create a VirtualMachineClone and then do a backup with the original and clone virtual machine, the restore fails because it ignores the `status.phase` "Succeeded" and tries to clone again to the virtual machine that exists.
    
    The Openshift Virtualization creates a snapshot of the original VirtualMachine, which adds an unwanted VirtualMachineSnasphot and a set of associated VolumeSnapshots after restore whose name starts with "tmp-". The clone operation does not complete and remains stuck in "RestoreInProgress" state because the requested VirtualMachine exists in the VirtualMachineClone.
    

Resolution

As a resolution, force the restore of the VirtualMachineClone objects by explicitly including it in the Recipe.

Change the OADP `DataProtectionApplication` object "velero" and add in the `spec.configuration.velero.args.restore-resource-priorities` field as follows:

    
        velero:
          args:
            restore-resource-priorities: "securitycontextconstraints,customresourcedefinitions,namespaces,managedcluster.cluster.open-cluster-management.io,managedcluster.clusterview.open-cluster-management.io,klusterletaddonconfig.agent.open-cluster-management.io,managedclusteraddon.addon.open-cluster-management.io,storageclasses,volumesnapshotclass.snapshot.storage.k8s.io,volumesnapshotcontents.snapshot.storage.k8s.io,volumesnapshots.snapshot.storage.k8s.io,datauploads.velero.io,persistentvolumes,persistentvolumeclaims,serviceaccounts,secrets,configmaps,limitranges,pods,replicasets.apps,clusterclasses.cluster.x-k8s.io,endpoints,services,-,clusterbootstraps.run.tanzu.vmware.com,clusters.cluster.x-k8s.io,clusterresourcesets.addons.cluster.x-k8s.io,virtualmachines.kubevirt.io,virtualmachineclones.clone.kubevirt.io"

Problem statement

Datamover operator must restore CephFS and IBM Storage Scale 5.2.0+ snapshots for backup by using the ReadOnlyMany access modes.

Resolution

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")



Service protection issues
=========================

List of service protection issues in Backup & Restore service of IBM Storage Fusion.

Service protection (scheduled) backups fail with Failed snapshot error
----------------------------------------------------------------------

Cause

Pod gets terminated during data export due to lack of resources on the node, where the pod that exports the service protection data is running.

Resolution

Address the lack of resources on the node issue and rerun the service protection backup.

User interface issues
---------------------

Problem statement

After a service protection restores the control plane and an application is restored, the following four columns do not populate on the backed up applications pane:

*   Backup status
*   Last backup on
*   Success rate
*   Backup capacity

Resolution

To populate those columns with data, run backup again for that application. Note that they do not include the counts before restore.

Known issue
-----------

*   Service protection can be configured on one cluster with both application and service backups. You can use the same bucket on cloud storage to configure service protection on a second cluster to restore service and application backups from the first cluster. Backups that no longer exist on cloud storage appear on the user interface, and a failure occurs during restore attempts of those backups.
    
    1.  If the first cluster remains as is, the retention period on the original backups can expire, and backups get removed from cloud storage. As the second cluster is unaware of the removal, it can attempt to remove the restored backups. The attempt fails because the backup on cloud storage no longer exists.
    2.  If you uninstall the Backup & Restore service from the first cluster, use the `-s` option to prevent `DeleteBackupRequest` CRs from getting created. If you do not set this option, the backups on cloud storage get removed, and the second cluster is again unaware that they no longer exist on the cloud storage.
    
    Note: The first deployment must not exist during the configuration of the second cluster.
    

**Parent topic:** [Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html "List of known Backup & Restore issues in IBM Storage Fusion.")

Serviceability known issues
===========================

List of all troubleshooting and known issues in Events, Log collection, and Call Home.

Events that go through the trap server do not get created in IBM Storage Fusion
-------------------------------------------------------------------------------

Problem statement

Sometimes, events that go through the trap server do not get created in IBM Storage Fusion.

Resolution

If you see the following details in the trapserver logs, follow the steps as a workaround:

    Listening for traps on 0.0.0.0:31620
     [fd8c:215d:178e:c0de:a94:efff:fef3:35cd]:41493 byte array parsed in is not a sequence
     [fd8c:215d:178e:c0de:a94:efff:fef3:3561]:35748 failed to parse sequence length0
     2021/10/13 14:20:48.124 [D] Recovered in resetServer, r=runtime error: slice bounds out of range [:-2592903709665718705]
    Listening for traps on 0.0.0.0:31620
     [fd8c:215d:178e:c0de:a94:efff:fef3:3555]:58314 failed to parse sequence length0
     [fd8c:215d:178e:c0de:a94:efff:fef3:35cd]:34801 byte array parsed in is not a sequence
     [fd8c:215d:178e:c0de:a94:efff:fef3:35cd]:52070 byte array parsed in is not a sequence
     [fd8c:215d:178e:c0de:a94:efff:fef3:3555]:47601 parse error
     [fd8c:215d:178e:c0de:a94:efff:fef3:3585]:56764 length parse error @ idx 2
     [fd8c:215d:178e:c0de:a94:efff:fef3:3399]:44497 failed to parse sequence length0
     [fd8c:215d:178e:c0de:a94:efff:fef3:35cd]:57884 failed to parse sequence length0
     [fd8c:215d:178e:c0de:a94:efff:fef3:3585]:59094 length parse error @ idx 2
     [fd8c:215d:178e:c0de:a94:efff:fef3:3555]:40214 parse error
     [fd8c:215d:178e:c0de:a94:efff:fef3:3399]:37409 byte array parsed in is not a sequence

*   Restart the trapserver pod in the `ibm-spectrum-fusion-ns`.
*   Run the following command to delete all the `ComputeMonitoring` CRs that are present in the `ibm-spectrum-fusion-ns` namespace.
    
        oc delete cmo --all -n ibm-spectrum-fusion-ns
    
    Wait for the custom resource instances to get recreated.
*   Restart BMC of all the compute nodes by executing `resetsp` command from the BMC command line.

Log status shows complete for downloaded log file with 0-bytes size
-------------------------------------------------------------------

Problem statement

The Log status shows complete for downloaded log file with of 0-bytes size. The status should be failed if logs are not collected, but it shows completed with 0-bytes size.

Cause

1.  Pods get evicted because of a lack of storage space.
2.  The ongoing jobs are taking time to get storage space, but if it takes more time, then they are marked as stale and automatically get cleaned up.

Resolution

There are two methods available to resolve this issue:

Method 1

1.  Delete the unnecessary logs through the IBM Storage Fusion HCI System user interface to get storage space. For more information, see [Delete a log package](https://ibmdocs-test.dcs.ibm.com/docs/en/SSG4YK_2.6/serviceability/sf_logs_page.html#sf_logs_page__info_vmc_1cz_txb "(Opens in a new tab or window)").
2.  Delete the on going jobs that are taking a long time and retry the job again.

Method 2

*   You can increase the log collector PVC size by following the steps:
    1.  Log in to the Red Hat® OpenShift® Container Platform web console.
    2.  Go to Storage > PersistentVoulmeClaims.
        
        The PersistentVolumeClaims page gets displayed.
        
    3.  Select the log collecter PVC that you want to modify.
    4.  Click the ellipsis icon and select Expand PVC.
        
        The Expand PersistentVolumeClaims page gets displayed.
        
    5.  Set the PVC value that you want and click Expand.

Known issues
------------

*   In the Logs page of the IBM Storage Fusion user interface, if you select System Health Check option during log collection, then it takes longer than usual time to complete. It is observed that the log collection process might take 20 to 25 minutes. In some cases, it can be due to many directories and multiple IMM log file collection process.
*   For IBM Storage Scale warning events, the fixed status might be incorrect.
*   Sometimes, in the Events page, the Source column of the events list might be incorrect.
*   Events are not created for IBM Storage Scale events with entity\_name fields that do not conform to the URL path components (^((\[A-Za-z0-9\]\[-A-Za-z0-9\_.\]\*)?\[A-Za-z0-9\])?$).
*   Call Home ticket creation can have a failed state when the system is not entitled or the Call Home server does not respond with the ticket number. Click Verify connection to check the test connection.
*   To prevent a deadlock condition, the event manager must be restarted every 24 hours. Run the following command to restart the event manager:
    
        oc rollout restart deployment eventmanager
    
*   Sometimes, the automatic upload of logs might not happen and the Call Home would fail. In such cases, manually upload the logs.
*   If you power off a control node wherein the trap server pod is running, then the migration of trap server pods to a different node fails. As a result, the trap server pod may get stuck in the terminating state, and some SNMP trap events may fail to capture and display.
*   In rare scenarios, the Events page comes up as empty because of a backend error due to a load with a 504 Gateway error. Generally, this page comes up after sometime automatically as the system recovers, so please try again after sometime.
*   One possible reason is log collector pod runs out of space and no space is available on it. This can happen when too many logs are collected in short period. The solution is to delete already collected logs from Fusion UI which are no longer required.
    
    **Workaround**
    
    *   From the title bar, click the help icon and select Support logs.
    *   Identify and delete logs through ellipsis menu in the Support logs page.
    
*   If Data Foundation log collection request or requests for multiple log packages collected simultaneously fails or stuck for more than 6 hours, then update the pod memory to 12000MiB instead of 6000MiB in the log collector deployment at `spec.containers[0].resources.limits.memory` and then attempt the log collection again one by one.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Global Data Platform service issues
===================================

This section lists the troubleshooting tips and tricks when you use IBM Storage Fusion storage.

Warning: Do NOT delete IBM Storage Scale pods. Deletion of Scale pods in many circumstances has implications on availability and data integrity.

Scale operator failed to add quorum label
-----------------------------------------

Problem statement

During Storage configuration, the recovery group status can be "`Waiting on some daemons in the recovery group to be restarted`".

Resolution

1.  Run the following command to check the count of the quorum node:
    
        oc get nodes -l scale.spectrum.ibm.com/designation=quorum
    
2.  Check the number of quorum nodes. Whenever the node count is less than 10, there must 3 quorum nodes. If not, apply the following quorum label for the missing nodes:
    
        scale.spectrum.ibm.com/designation=quorum
    

Remote filesystem connection goes to a Disconnected state
---------------------------------------------------------

Problem statement

If even one scale core pod is not ready due to node restart or other issues, then a problem occurs on the filesystem mount of that node and the filesystem CR reports errors. As a result, the filesystem goes to a Disconnected and the same is displayed on the IBM Storage Fusion user interface.

Resolution

Wait for the scale core pod to go to Ready state, and the filesystem automatically changes to Connected state.

PVCs stuck in pending state
---------------------------

Resolution

If PVCs are stuck in a pending state for a long time, restart Scale GUI pods.

Nodes with pods scheduled for storage results in a `crashloobackoff` or container creating errors
-------------------------------------------------------------------------------------------------

Problem statement

When install, upgrade, or upsize operations are in progress and the storage configuration is not yet done, the nodes with pods scheduled for storage results in a `crashloobackoff` or container creating errors.

Resolution

To resolve this issue, disable schedule on these nodes before you begin these operations, and schedule the pods on nodes that are configured for storage.

File system is not mounted on some nodes
----------------------------------------

Problem statement

After IBM Storage Scale upgrade, the file system is not mounted on some nodes.

Resolution

Do the following workaround steps:

1.  Delete the problematic pod where the file system is not mounted. Delete one pod at a time.
2.  Refresh the pod overview page until the pod appears. Run `watch oc get pods`.
3.  When the problematic pod comes up, set the replicas to 0 in the `Spec` section in the `ibm-spectrum-scale-operator` namespace. You must stop the operator deployment immediately. Ensure that no operator pod is running in `ibm-spectrum-scale-operator` namespace. Command for setting the operator replica to 0:
    
        oc patch deploy ibm-spectrum-scale-controller-manager \
         --type='json' -n ibm-spectrum-scale-operator \
         -p='[{"op": "replace", "path": "/spec/replicas", "value": 0}]'
        
    
        oc get deployment -n ibm-spectrum-scale-operator
    
    Sample output:
    
        
        I0420 16:56:23.579340    4184 request.go:645] Throttling request took 1.192496025s, request: GET:https://api.isf-rackk.rtp.raleigh.ibm.com:6443/apis/nmstate.io/v1beta1?timeout=32s
        NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
        ibm-spectrum-scale-controller-manager   0/0     0            0           28h
    
4.  Wait till the problematic pod is in `init` container state and run the following command:
    
        oc exec podname -- mmsdrrestore -p <any working core ip/name[Specify pod IP where FS is mounted]>
    
5.  After the previous step is successful, set the replicas to 1 again in the `ibm-spectrum-scale-operator` deployment.
    
    Ensure that the operator pod is running in `ibm-spectrum-scale-operator` namespace.
    
    Run the following command for setting the operator replica to 0:
    
        oc patch deploy ibm-spectrum-scale-controller-manager \
         --type='json' -n ibm-spectrum-scale-operator \
         -p='[{"op": "replace", "path": "/spec/replicas", "value": 1}]'
    
        oc get deployment -n ibm-spectrum-scale-operator
    
    Sample output:
    
        I0420 16:56:23.579340    4184 request.go:645] Throttling request took 1.192496025s, request: GET:https://api.isf-rackk.rtp.raleigh.ibm.com:6443/apis/nmstate.io/v1beta1?timeout=32s
        NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
        ibm-spectrum-scale-controller-manager   1/1     0            0           28h
    
6.  Wait for some time for the problematic pod to come in running state and file system gets mounted on this problematic node.

IBM Storage Scale user interface does not load properly or IBM Storage Scale CSI pods are in `CrashLoopBack` state
------------------------------------------------------------------------------------------------------------------

Problem statement

Sometimes, IBM Storage Scale user interface might not load properly or IBM Storage Scale CSI pods are in `CrashLoopBack` state.

Resolution

As a workaround, do the following steps:

1.  Restart the GUI pods and check whether it is working properly.
2.  If GUI pod restart does not open the IBM Storage Scale console, then restart the `ibm-spectrum-scale-controller-manager` pod and then restart GUI pod.

Not able to log in to IBM Storage Scale user interface
------------------------------------------------------

Cause

The secret that is mounted in GUI pods /etc/gui-oauth/oauth.properties file and secret present in the `ibm-spectrum-scale-gui-oauthclient` are different.

    gui-1
    sh-4.4$ cd /etc/gui-oauth/
    sh-4.4$ cat oauth.properties
    secret=tjCK6kE0pmCe2d8b7azw
    oauthServer=https://oauth-openshift.apps.isf-racka.rtp.raleigh.ibm.com
    redirectURI=https://ibm-spectrum-scale-gui-ibm-spectrum-scale.apps.isf-racka.rtp.raleigh.ibm.com/auth/redirect
    clientID=ibm-spectrum-scale-gui-oauthclient
    k8sAPIServer=https://172.30.0.1:443
    sh-4.4$
    gui_0
    secret=tjCK6kE0pmCe2d8b7azw
    oauthServer=https://oauth-openshift.apps.isf-racka.rtp.raleigh.ibm.com
    redirectURI=https://ibm-spectrum-scale-gui-ibm-spectrum-scale.apps.isf-racka.rtp.raleigh.ibm.com/auth/redirect
    clientID=ibm-spectrum-scale-gui-oauthclient
    k8sAPIServer=https://172.30.0.1:443
    Oauth client config (API Explorer -> OAuthClient -> ibm-spectrum-scale-gui-oauthclient)
    kind: OAuthClient
    apiVersion: oauth.openshift.io/v1
    metadata:
      name: ibm-spectrum-scale-gui-oauthclient
      uid: a9eb9d85-33b9-4cd7-99c8-ae0213032380
      resourceVersion: ‘314932’
      creationTimestamp: ‘2022-03-29T17:09:38Z’
      labels:
        app.kubernetes.io/instance: ibm-spectrum-scale
        app.kubernetes.io/name: gui
      ownerReferences:
    apiVersion: scale.spectrum.ibm.com/v1beta1
          kind: Gui
          name: ibm-spectrum-scale-gui
          uid: da23c696-5c71-405b-b9ac-c7d0ccb4dd43
          controller: true
          blockOwnerDeletion: true
      managedFields:
    manager: ibm-spectrum-scale/ibm-spectrum-scale-gui
          operation: Apply
          apiVersion: oauth.openshift.io/v1
          time: ‘2022-03-29T17:09:38Z’
          fieldsType: FieldsV1
          fieldsV1:
            ‘f:grantMethod’: {}
            ‘f:metadata’:
              ‘f:labels’:
                ‘f:app.kubernetes.io/instance’: {}
                ‘f:app.kubernetes.io/name’: {}
              ‘f:ownerReferences’:
                ‘k:{“uid”:“da23c696-5c71-405b-b9ac-c7d0ccb4dd43"}’:
                  .: {}
                  ‘f:apiVersion’: {}
                  ‘f:blockOwnerDeletion’: {}
                  ‘f:controller’: {}
                  ‘f:kind’: {}
                  ‘f:name’: {}
                  ‘f:uid’: {}
            ‘f:redirectURIs’:
              ‘v:“https://ibm-spectrum-scale-gui-ibm-spectrum-scale.apps.isf-racka.rtp.raleigh.ibm.com/auth/redirect”’: {}
            ‘f:secret’: {}
    *secret: bk2_q1FqwhVhoNqOl7ob*
    redirectURIs:
    >-
        https://ibm-spectrum-scale-gui-ibm-spectrum-scale.apps.isf-racka.rtp.raleigh.ibm.com/auth/redirect
    grantMethod: auto
    
    

Resolution

As a resolution, do the following steps:

1.  As a resolution, restart the GUI pods.
2.  If the previous step (step 1) does not resolve the issue, then restart IBM Storage Scale Operator pod and then restart the GUI pods.

Download log files of all deployments
-------------------------------------

If you cannot download log files of all deployments, individually download the log files.

Issues in node drain and node restart
-------------------------------------

Resolution

To resolve such node related issues, see [Issues related to IBM Storage Fusion HCI System node drains](sf_hci_common_issues.html "Use these common troubleshooting tips and tricks when you work with IBM Storage Fusion HCI System.").

IBM Storage Scale cluster failed to recover after powering off and on the compute nodes
---------------------------------------------------------------------------------------

Diagnosis

Identify if the nodes joining the cluster are in an out-of-tolerance situation.

1.  mmhealth reported UNKNOWN RG status on some nodes.
2.  mmrepquota commands have been generated repeatedly, likely by the GUI, which has caused many long waits.
3.  Due to the large number of long waits, the cluster manager node was not responding to any ts cmd other than tsctl nqStatus.
    
    As a resolution, follow the steps defined in the [Recover storage cluster from an unplanned multi-node restart or failure](../storage/sf_stg_cluster_failure.html "The service of IBM Storage Scale file system might hang if the storage nodes are restarted.").
    

MCO rollout is not progressing for more than 5 hours during the scale upgrade
-----------------------------------------------------------------------------

Cause

Whenever the nodes are over committed and no pod limit is available, you might see delays in the policy rollout, ICSP, pull secret, and MCO rollout changes.

Resolution

As a resolution, do the following step:

*   IBM Storage Fusion HCI System nodes allow the maximum number of pods on a node as defined by OpenShift® Container Platform default and ensure to keep that count within the limit. Otherwise, you might see this issue.

IBM Storage Scale cluster status is in a critical state after power on/off the rack
-----------------------------------------------------------------------------------

Cause

IBM Storage Scale cluster status is in critical state due to `unmounted_fs_check` is present on the `filesystem` CR.

Resolution

As a resolution, wait until this `filesystem` CR status gets cleared; after that, the IBM Storage Scale cluster status automatically returns to a healthy state. If the issue persists for a long time, then contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.") .

Disk upsize issues
------------------

*   Manual steps to test the `/dev` mount:
    
    After you add a new NVMe disk to the server, the new disk cannot be discovered in the IBM Storage Scale pod. The disks do not show up with /dev/nvme\* device names in the POD, IBM Storage Scale need the device name /dev/nvme\* to access the disks. Without the /dev/nvme\* device name, the new disks cannot be added into recovery group.
    
    Resolution
    
    Do the following manual steps to mount disks to /dev in the pod.
    
    1.  Log in to each IBM Storage Scale storage pods.
        
            oc exec -it <pod name> -c <container> /bin/sh
        
    2.  Stop the GPFS daemons on one or more nodes:
        
            mmshutdown
        
    3.  Mount `/dev` from the host running the following command:
        
            mount -t devtmpfs none /dev
        
    4.  Validate that the content of /dev is from host and you can see sub directories like block, bus, char.
        
            mmstartup


Troubleshooting issues in IBM Storage Fusion HCI System
=======================================================

This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.

*   **[Collecting log files of final installation](../install/sf_collect_logs_stg3_ops.html)**  
    Three operators get involved in the final installation. The `Installer` operator adds compute nodes to the OpenShift® Container Platform cluster, and orchestrates the overall final installation. The `Storage` operator installs the IBM Storage Scale cluster.
*   **[Installation and upgrade issues](../tshooting/sf_install_tshooting.html)**  
    Use the troubleshooting tips and tricks in IBM Storage Fusion installation.
*   **[Metro-DR issues](../tshooting/sf_metrodr_tshooting.html)**  
    Use these troubleshooting information to resolve issues when you work with Metro-DR.
*   **[Regional-DR issues](../tshooting/sf_rdr_tshooting.html)**  
    Use these troubleshooting information to resolve issues when you work with Regional-DR.
*   **[Networking issues](../network/sf_network_tshooting.html)**  
    A list of all troubleshooting and known issues that exist in the networking of IBM Storage Fusion HCI System.
*   **[Node issues](../tshooting/sf_compute_troubleshooting.html)**  
    List of all troubleshooting and known issues while you work with the compute nodes.
*   **[Serviceability known issues](../tshooting/sf_serviceability_tshooting.html)**  
    List of all troubleshooting and known issues in Events, Log collection, and Call Home.
*   **[IBM Storage Fusion HCI System user interface issues](../tshooting/sf_tshooting_ui_issues.html)**  
    A list of all known issues in the IBM Storage Fusion user interface.
*   **[Issues related to IBM Storage Fusion HCI System node drains](../tshooting/sf_hci_common_issues.html)**  
    Use these common troubleshooting tips and tricks when you work with IBM Storage Fusion HCI System.
*   **[Troubleshooting installation and upgrade issues in IBM Storage Fusion services](../tshooting/sds_services_troubleshooting.html)**  
    Use these troubleshooting information to know install and upgrade problems related to IBM Storage Fusion services.
*   **[IBM Fusion Data Foundation service error scenarios](../tshooting/sf_odf_troubleshooting.html)**  
    Use these troubleshooting information to know the problem and workaround when install or configure IBM Fusion Data Foundation service.
*   **[Troubleshooting Backup & Restore service issues](../sf_guardian_known_issues.html)**  
    List of known Backup & Restore issues in IBM Storage Fusion.
*   **[IBM Storage Fusion Data Cataloging known issues](../discover/datacataloging_knownissues.html)**  
    List of all issues in the Data Cataloging service along with its resolution.
*   **[Troubleshooting Hosted Control Plane clusters](../tshooting/sf_df_cons_mode_hcp_cluster.html)**  
    Use these troubleshooting information to know the problem and workaround for the installation of spoke cluster and
*   **[IBM Cloud Satellite diagnosis and troubleshooting](../ICS/sf_ics_troubleshooting.html)**  
    Use troubleshooting information to isolate and resolve issues. It also includes FAQs for your common questions around IBM Cloud® Satellite in IBM Storage Fusion.
*   **[Connection setup after OpenShift Container Platform cluster recovery](../tshooting/sf_ocp_cluster_recover.html)**  
    The OpenShift Container Platform cluster can have problems and become unusable. After you recover the cluster, rejoin the connections.

**Parent topic:** [Troubleshooting](../tshooting/sf_troubleshooting.html "The steps to troubleshoot an issue vary depending on the problem. To help make relevant information available to you as quickly as possible, the log files and other tools for troubleshooting problems are consolidated.")


Troubleshooting
===============

The steps to troubleshoot an issue vary depending on the problem. To help make relevant information available to you as quickly as possible, the log files and other tools for troubleshooting problems are consolidated.

*   **[Contacting IBM Support Center](../tshooting/sf_contact_ibm_support.html)**  
    The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.
*   **[How to provide feedback](../sf_feedback.html)**  
    
*   **[Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html)**  
    This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.
*   **[Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html)**  
    This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.


IBM Storage Fusion HCI System user interface issues
===================================================

A list of all known issues in the IBM Storage Fusion user interface.

Known issues in the node management pages
-----------------------------------------

*   If you use the IBM Storage Fusion user interface to move a node to maintenance mode, then the user interface might show the status of the node as Ready.
*   Only one node can go down at a time for maintenance and power down operation. If you place a second node in maintenance, then the user interface does not show any error message instead shows the node with status as Ready (maintenance mode in progress).

General issues
--------------

*   To close or terminate all open sessions in IBM Storage Fusion, log out of both IBM Storage Fusion UI console and OpenShift® Container Platform UI console.
*   Sometimes during upgrades and the nodes are down, a red toast notification appear on the user interface when you configure the remote filesystem even though file system is recovered and shows connected. In such case, you need to be explicitly close the notification.

**Parent topic:** [Troubleshooting issues in IBM Storage Fusion HCI System](../tshooting/sf_troubleshooting_HCI.html "This topic covers the most common issues and their workarounds, which you might encounter while you work with the IBM Storage Fusion HCI System appliance.")


Upgrade events and error codes
==============================

List of all the events that you might encounter during upgrade.

*   **[BMYUP1103](../errorcodes/BMYUP1103.html)**  
    Upgrade of Red Hat® OpenShift® cluster is in progress.
*   **[BMYUP1104](../errorcodes/BMYUP1104.html)**  
    Red Hat OpenShift cluster upgrade is complete.
*   **[BMYUP1105](../errorcodes/BMYUP1105.html)**  
    IBM Storage Fusion firmware upgrade available for nodes.
*   **[BMYUP1106](../errorcodes/BMYUP1106.html)**  
    IBM Storage Fusion firmware upgrade available for switches.
*   **[BMYUP1107](../errorcodes/BMYUP1107.html)**  
    Firmware update succeeded for the compute node.
*   **[BMYUP1108](../errorcodes/BMYUP1108.html)**  
    Firmware update is in progress on the compute node.
*   **[BMYUP1109](../errorcodes/BMYUP1109.html)**  
    Firmware update succeeded for the switch.
*   **[BMYUP1110](../errorcodes/BMYUP1110.html)**  
    Firmware Update of the switch is in progress.
*   **[BMYUP1111](../errorcodes/BMYUP1111.html)**  
    Started upgrade of the Global Data Platform service.
*   **[BMYUP1112](../errorcodes/BMYUP1112.html)**  
    Completed upgrade of the Global Data Platform service.
*   **[BMYUP1113](../errorcodes/BMYUP1113.html)**  
    Started upgrade of the Data Protection service.
*   **[BMYUP1114](../errorcodes/BMYUP1114.html)**  
    Completed upgrade of the Data Protection service.
*   **[BMYUP1117](../errorcodes/BMYUP1117.html)**  
    IBM Storage Fusion service upgrade available for the on-boarded service.
*   **[BMYUP1118](../errorcodes/BMYUP1118.html)**  
    Service upgrade is available.
*   **[BMYUP1119](../errorcodes/BMYUP1119.html)**  
    Service upgrade in progress.
*   **[BMYUP1120](../errorcodes/BMYUP1120.html)**  
    Service upgrade is completed.
*   **[BMYUP1121](../errorcodes/BMYUP1121.html)**  
    Operator Upgrade in progress.
*   **[BMYUP1123](../errorcodes/BMYUP1123.html)**  
    Operator Upgrade completed.
*   **[BMYUP2106](../errorcodes/BMYUP2106.html)**  
    Firmware update Job failed for the compute node.
*   **[BMYUP2107](../errorcodes/BMYUP2107.html)**  
    Firmware Update of the switch has failed.
*   **[BMYUP2108](../errorcodes/BMYUP2108.html)**  
    Failed to login to the switch with default user credentials after firmware upgrade.
*   **[BMYUP2109](../errorcodes/BMYUP2109.html)**  
    Failed to change the password of ISFUSER on the switch.
*   **[BMYUP2110](../errorcodes/BMYUP2110.html)**  
    Failed to login to the switch with ISFUSER credentials.
*   **[BMYUP2111](../errorcodes/BMYUP2111.html)**  
    Failed to login to switch during firmware upgrade.
*   **[BMYUP3101](../errorcodes/BMYUP3101.html)**  
    Failed to upgrade the Global Data Platform service.
*   **[BMYUP3102](../errorcodes/BMYUP3102.html)**  
    Failed to upgrade Backup & Restore service.
*   **[BMYUP3104](../errorcodes/BMYUP3104.html)**  
    Service upgrade is failing.
*   **[BMYUP3105](../errorcodes/BMYUP3105.html)**  
    Operator Upgrade Failed.

**Parent topic:** [Events and error codes message references in IBM Storage Fusion HCI System](../sf_event_error_messages.html "This reference information provides a consolidated view of all the events or error messages that can be displayed when the IBM Storage Fusion system might result in an event or an error.")


Troubleshooting installation issues
===================================

Troubleshooting IBM Storage Fusion HCI System installation issues.

Empty node list in Network precheck wizard
------------------------------------------

Problem statement

The Network validation wizard page (Network setup stage 1) of the IBM Storage Fusion HCI System installation can have an empty node list with the Finish button in enabled state. The InlineNotification status may also show a Connection complete! status with a green checkmark that suggests you can proceed to the next step.

Similarly, Network precheck wizard page (Red Hat OpenShift installation stage 2) of IBM Storage Fusion HCI System may have an empty node list with the Next button enabled.

Resolution

1.  If you run into this scenario, confirm that the nodes are connected before you proceed:
    
    Check the response of the endpoint `/verifydhcp`
    
2.  Find the failed nodes from the response.
3.  Manually verify the configuration of the node in DHCP and DNS.
    
    The IP addresses of the nodes can either be incorrect or not reachable.
    
4.  Fix the issue and restart Network setup (stage 1) installation or Red Hat OpenShift installation (stage 2).

Nodes added as local host or local domain to Red Hat OpenShift Container Platform cluster
-----------------------------------------------------------------------------------------

Resolution

If the Red Hat® OpenShift® installation fails, then retry OpenShift installation wizard. If problem persists, contact IBM Support.

`ISF node exporter` pod in `container creating` state
-----------------------------------------------------

Problem statement

You might encounter `ISF node exporter` pod in `container creating` state at the end of stage 2 installation. An example pod in `container creating` state:

2022-05-16T12:06:07.538Z ERROR controller-runtime.manager.controller.node Reconciler error {"reconciler group": "storage.isf.ibm.com", "reconciler kind": "Node", "name": "isf-node-exporter", "namespace": "openshift-monitoring", "error": "pod is not in ready state isf-node-exporter-5bc7bb6587-vcr2n Skipping the installation process"} sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).processNextWorkItem /workspace/vendor/sigs.k8s.io/controller-runtime/pkg/internal/controller/controller.go:253 sigs.k8s.io/controller-runtime/pkg/internal/controller.(\*Controller).Start.func2.2 /workspace/vendor/sigs.k8s.io/controller-runtime/pkg/internal/controller/controller.go:214

Resolution

Ignore this message and proceed to work with next steps as the error gets resolved by itself.

ImagePull failure during an installation:
-----------------------------------------

Cause

If an `ImagePull` failure occurs due to intermittent network or registry issue during IBM Storage Fusion HCI installation, then restart the pod and retry.

Resolution

If the issue persists, contact [IBM support](sf_contact_ibm_support.html "The IBM® Support Center is available for various types of IBM hardware and software problems that IBM Storage Fusion customers might encounter.").

Pull a container image from the `registry.connect.redhat.com`
-------------------------------------------------------------

Problem statement

If you pull a container image from the `registry.connect.redhat.com`, it redirects to AWS S3. It is a known issue in Red Hat.

Diagnostic steps

To verify the error message, do the following steps:

1.  Log in to Red Hat OpenShift Container Platform control node.
2.  Use the following commands to manually pull an image from `registry.connect.redhat.com`:
    
        
        $ podman login registry.connect.redhat.com
        $ podman pull registry.connect.redhat.com/seldonio/seldon-core-operator-bundle:latest
    
    Note: An example image is used for illustration.
    

Resolution

A portion of the content is hosted on `registry.connect.redhat.com` by using the following AWS S3 bucket: `rhc4tp-prod-z8cxf-image-registry-us-east-1-evenkyleffocxqvofrk.s3.dualstack.us-east-1.amazonaws.com`. Allow this domain so that OpenShift Container Platform can access it in your firewall.

Known issues
------------

*   During Red Hat OpenShift cluster creation, you cannot download logs but can monitor them on the user interface and download them after installation.
*   If you observe an error `Configmap fusion platform not found in fusion namespace` in the prereq operator logs, then the error does not have any impact and can be ignored from the `isf-prereq-operator-controller-manager-xxxx` pod logs.

**Parent topic:** [Installation and upgrade issues](../tshooting/sf_install_tshooting.html "Use the troubleshooting tips and tricks in IBM Storage Fusion installation.")

