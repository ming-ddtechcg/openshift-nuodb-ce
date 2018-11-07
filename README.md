# NuoDB OpenShift CE Templates

Public repository for the NuoDB CE OpenShift templates.

## NuoDB Community Edition (CE) in OpenShift Overview

The NuoDB CE OpenShift template is available in two options (1) using ephemeral storage and (2) using persistent storage. With ephemeral storage, if the Admin Service pod or Storage Manager pod are stopped or deleted, the database will be removed with the pod. With persistent storage, if either pod is stopped or deleted, the database state is preserved and available again (just as it was) on automatic pod restart.  The Enterprise Edition already supports persistent storage, which can be made available by simply contacting us. We will also be releasing additional CE templates with persistent storage options. For more information about NuoDB running in Red Hat OpenShift, including System Requirements, please see our documentation section, [Deploying NuoDB in OpenShift Environments](https://doc.nuodb.com/Latest/Default.htm#Deploying-NuoDB-with-OpenShift.htm "NuoDB Documentation")

## Image Availability

The templates reference images stored in the RHCC / Registry. Other images
exist in case you don't have a RHCC account. The templates are parameterized
so you can configure which image is used. The set of permissible images are as
follows:

| image  | operating system | default  |
|---|---|---|
| registry.connect.redhat.com/nuodb/nuodb-ce:latest  | RHEL 7.5 | yes |
| nuodb/nuodb-ce:latest | CentOS 7.5 | no |

## Environment Considerations

We recommend that you perform your demo using three (3) or four (4) nodes.
However, because your clusters may have more nodes than that, and you may
want to dedicate particular nodes for testing NuoDB, the templates are set
up to permit isolating NuoDB to specified nodes using labels.

While you can run the CE template on a single host, this is not recommended
as you will end up running the transaction engine, storage manager, administration
tier, and client sample application -- all on one host.

## Deploying NuoDB CE in OpenShift

### STEP 1.
Create a project name “nuodb” by clicking the OpenShift “Create Project” button. If you like, you can also add a template display name of “NuoDB CE”.

### STEP 2.
Disable Transparent Huge Pages (THP) on the servers you will run NuoDB containers. For more information on this topic, see our OpenShift Prerequisite Configuration page, section titled “Disabling Transparent Huge Pages (THP)”.

### STEP 3.
Select the servers you want to use for NuoDB by assigning them OpenShift labels. The database install will start four container pods. We recommend you label at least three servers using the oc label command. For example:
      
      $ oc label node nuodb.com/zone=nuodb

      If you are deploying the persistent storage template run these addition two commands from your master node.
          
          For one storage node, label it for storage use using this command
              $ oc label node <node name> nuodb.com/node-type=storage
          Create the local persistent storage disk class and volume
              $ oc create -f local-disk-class.yaml
          
### STEP 4.
Create an image pull secret which allows the template to pull the NuoDB CE container image from RHCC. From the OpenShift left bar menu, click “Resources”, then “Secrets”, and “Add Secret”. Enter values:

      Secret Type = Image Secret
      Secret Name = pull-secret
      Authentication Type = Image Registry Credentials
      Image Registry Server Address = registry.connect.redhat.com
      Username = (your RH login)
      Password = (your RH password)
      Email = (your email address)
      Link secret to a service account (check this box)
      Service Account = Default

### STEP 5.
Import the NuoDB CE template of your choice into OpenShift by navigating to the Overview Tab, clicking “Import YAML/JSON,” and running the import.

### STEP 6.
Process the template by navigating back to the Overview Tab and clicking “Select from Project”. Choose the NuoDB CE template and follow the installation prompts.

It will only take a few moments for the database to start! You will see one pod each for the following database processes:

      Administrative Service (Admin)
      Storage Manager (SM)
      Transaction Engine (TE)
      Insights


#### Additional Information - Server Node Labeling Commands

To permit isolating the NuoDB CE demo onto dedicated OpenShift nodes, you
must add one label if you're running the `ephemeral` template, and two
labels if you are running the `persistent` template. In each case, the
values for the labels are irrelevant for the CE version, as the templates
simply check for their existence. The following table outlines the labels,
their applicability, etc:

| label  | description  | ephemeral  | persistent  | default |
|---|---|---|---|---|
| nuodb.com/zone  | availability zone or set of nodes to run nuodb on  | yes  | yes  | none |
| nuodb.com/node-type  | type of node, typically `storage` |  no | yes  | none |

Label three or four nodes with the `nuodb.com/zone` label, any value.

Label ONLY one node with the `nuodb.com/node-type` label, with any value.

All NuoDB pods will run on those labeled nodes only.

For example, to list and add labels:

```
$ oc get nodes -L nuodb.com/zone -L nuodb.com/node-type

$ oc label node ip-10-0-2-152.ec2.internal nuodb.com/zone=nuodb
node "ip-10-0-2-152.ec2.internal" labeled
```

To delete the labels when you're all done:

```bash
$ oc label node ip-10-0-2-152.ec2.internal nuodb.com/zone-
node "ip-10-0-2-152.ec2.internal" labeled
```

Your cluster should look something like this in the end with regards to labeling:

```
$ oc get nodes -L nuodb.com/zone -L nuodb.com/node-type
NAME                         STATUS    ROLES     AGE       VERSION             ZONE      NODE-TYPE
ip-10-0-1-139.ec2.internal   Ready     compute   17h       v1.9.1+a0ce1bc657   a         storage
ip-10-0-1-237.ec2.internal   Ready     compute   17h       v1.9.1+a0ce1bc657   a         
ip-10-0-1-74.ec2.internal    Ready     compute   17h       v1.9.1+a0ce1bc657   a         
ip-10-0-1-93.ec2.internal    Ready     compute   17h       v1.9.1+a0ce1bc657   a         
ip-10-0-2-152.ec2.internal   Ready     compute   17h       v1.9.1+a0ce1bc657             
ip-10-0-2-222.ec2.internal   Ready     compute   17h       v1.9.1+a0ce1bc657             
ip-10-0-2-225.ec2.internal   Ready     compute   17h       v1.9.1+a0ce1bc657             
ip-10-0-2-73.ec2.internal    Ready     master    17h       v1.9.1+a0ce1bc657             
ip-10-0-2-81.ec2.internal    Ready     compute   17h       v1.9.1+a0ce1bc657            
```

## Cleaning Up

```bash
$ oc delete project nuodb
project "nuodb" deleted
```
