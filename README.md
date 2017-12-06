# Getting-Started-With-DSE-On-DCOS

Since point releases are being released at an untimed release schedule at this time please see DataStax Enterprise on DC/OS to access the latest docs and features.  As of this writing the current release is DSE 2.0.4-5.1.2

First things first (tongue) after you have a working DC/OS Cluster
Lets install the the DC/OS CLI 
Visit the DC/OS UI, Click the cluster name at the top left of the UI and click install CLI

Copy, paste and run the code snippet into a terminal for your OS


Installing DSE and OpsCenter on DC/OS
Installation with Default Settings
The service comes with defaults for testing, but  should not be used in production. 
Each DSE Node has:
– 2.5 CPUs
– 2500 MB mem
– Ports
The OpsCenter Node has:
– 1 instance
– 2 CPUs
– 6000 MB mem
– 1 10240 MB disk
– Ports   
From the DC/OS Dashboard website, DSE and OpsCenter may be installed with a default configuration as follows:
Visit http://yourcluster.com/ to view the DC/OS Dashboard
Navigate to Catalog and find the datastax-dse package.
Click Deploy to use default settings.
To use the built-in OpsCenter, repeat steps 2 and 3, but substitute datastax-ops for datastax-dse.
NOTE: DSE should not be deployed to production with the default settings. In production, the DSE Nodes should be operated with 32 GB of memory and 16 GB of heap.
Installation with Custom Settings
You may customize settings at initial install. 
From the CLI, DSE may be installed with a custom configuration as follows:
dcos package install datastax-dse --options=your-dse-options.json
OpsCenter can be installed with a custom configuration the same way:
dcos package install datastax-ops --options=your-datastax-ops-options.json
For more information about building the options.json file, see the DC/OS Documentation.
From the DC/OS Dashboard , DSE may be installed with a custom configuration as follows:
Visit http://yourcluster.com/ to view the DC/OS Dashboard. Navigate to Universe => Packages and find the datastax-dse package.  Click Install, then in the pop up dialog click Advanced to see the customization dialog.
Make your changes to the default configuration in the customization dialog, then click Review.
Examine the configuration summary for any needed changes. Click Back to make changes, or Install to confirm the settings and install the service.
If installing OpsCenter, repeat the above steps, but substitute datastax-ops for datastax-dse.
Now Lets install our specific CLI's for DC/OS
In your terminal run the following 
dcos package install --cli datastax-dse
dcos package install --cli datastax-ops

Managing and Updating Configuration
You will not manually edit config files, DC/OS manages the config files and pushes the values to the nodes. Some but not all configuration files and values are exposed as Environment Variables.  New values are being exposed in each point release.  
To manage these values from the UI you will do the following.
Visit http://yourcluster.com/ to view the DC/OS Dashboard. Navigate to Services => click the  at the far right of the service you want to update and select edit.
Here you can change Service settings, Network settings, Volumes, Health Checks and Environment settings.
After making any changes you will click  Review your settings and click .  This will push the new values to each nodes config files and restart the service.
Configurations are stored and can be rolled back to previous versions.
*** (DC/OS 1.10 Enterprise and above)***
To roll back changes to a previous version  do the following
Visit http://yourcluster.com/ to view the DC/OS Dashboard. Navigate to Services => click on the services name => go to the Configuration tab 
Click the black arrow next to Active and a list of configurations will appear.
 
Select the configuration you would like to roll back to and click Apply

Working with the CLI and getting a shell so we can load some data
Lets see the endpoints in you cluster.  Endpoints are the DSE services running on the nodes
**If you changed the default service_id you will need to add the --name=<service_id> to the end of your command**
dcos datastax-dse endpoints
[
"gremlin-agent",
"native-client",
"solr-admin",
"spark-worker-webui",
"thrift-client"
]
Now lets look at the cluster nodes
dcos datastax-dse endpoints native-client
{
 "address": [
 "10.200.181.204:9042",
 "10.200.181.216:9042",
 "10.200.181.208:9042"
 ],
 "dns": [
 "dse-0-node.dse.autoip.dcos.thisdcos.directory:9042",
 "dse-1-node.dse.autoip.dcos.thisdcos.directory:9042",
 "dse-2-node.dse.autoip.dcos.thisdcos.directory:9042"
 ]
}

Now lets get a shell so we can load some data and run other commands such as nodetool and dse 
Look back at that endpoint data from the dse endpoints command to find the name of one of your DSE nodes. Reference that node in the task exec command and DC/OS to get a shell.
dcos task exec -it dse-0-node /bin/bash

Now you have a shell we need to set up our environment
export TERM=xterm-256color
export HOME=/mnt/mesos/sandbox

Who wants to cqlsh so you can load some data? Don't hold the applause back (wink)
Grab an IP address of one of the native-client endpoints.

cqlsh 10.200.181.204


Troubleshooting
Accessing Logs
Logs for the Scheduler, DSE Nodes, OpsCenter instance (if built-in is enabled), instance may all be browsed via the DC/OS Dashboard.
Scheduler logs are useful for determining why a task isn't being launched (this is under the purview of the Scheduler).
OpsCenter logs are useful for examining problems in the OpsCenter dashboard.
DSE Node logs are useful for examining problems in DSE itself.
In all cases, logs are generally piped to files named stdout and/or stderr.
To view logs for a given node, perform the following steps:
Visit http://yourcluster.com/ to view the DC/OS Dashboard.
Navigate to Services and click on the service to be examined (default datastax-dse).
In the list of tasks for the service, click on the task to be examined (scheduler is named after the service, OpsCenter is opscenter-0-node, DSE nodes are each dse-#-node).
In the task details, click on the Logs tab to go into the log viewer. By default you will see stdout, but stderr is also useful. Use the pull-down in the upper right to select the file to be examined.
In case of problems with accessing the DC/OS Dashboard, logs may also be accessed via the Mesos UI:
Visit http://yourcluster.com/mesos to view the Mesos UI.
Click the Frameworks tab in the upper left to get a list of services running in the cluster.
Navigate into the correct Framework for your needs. The Scheduler runs under marathon with a task name matching the service name (default datastax-dse). Meanwhile DSE nodes and OpsCenter run under a Framework whose name matches the service name (default datastax-dse).
You should now see two lists of tasks. Active Tasks are what's currently running, and Completed Tasks are what has since exited. Click on the Sandbox link for the task you wish to examine.
The Sandbox view will list files named stdout and stderr. Click the file names to view the files in the browser, or click Download to download them to your system for local examination. Note that very old tasks will have their Sandbox automatically deleted to limit disk space usage.
Working with a Down Node
If you encounter a down node lets try to start it and watch the logs
Lets say node 0 is down and you need to restart it, you will do this from the cli
dcos datastax-dse pod restart dse-0

What happens if the node will not come up? Maybe a node in the DC/OS cluster is down,  WE CAN REPLACE IT!!
Lets say node 0 is down and you need to replace it, you will do this from the cli
dcos datastax-dse pod replace dse-0
Once the provisioning is complete, get a shell described earlier to any node and use the nodetool removenode command

Restarting OpsCenter
dcos datastax-ops pod restart opscenter-0
Restarting the DSE Cluster
From the cli run 
dcos marathon app restart <service_name>
Gathering information from the CLI
Find the version of DSE or OpsCenter installed
From the cli run the following 
dcos marathon app show <service_name> | grep "DCOS_PACKAGE_VERSION"
We may should check the docker container to make sure they are running our release along with  DSE/OpsCenter versions 
From the cli run the following 
dcos marathon app show <service_name> | grep -w '"DCOS_PACKAGE_VERSION"\|"DSE_DOCKER_IMAGE"'
    "DSE_DOCKER_IMAGE": "dspn/10517a0c8e9b11e79ef5600308a463aa-1:5.1.2",
    "DCOS_PACKAGE_VERSION": "2.0.4-5.1.2",
Our docker image should always be
dspn/10517a0c8e9b11e79ef5600308a463aa-1:5.1.2
