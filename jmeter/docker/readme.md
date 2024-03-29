# Overview
This project is a very lean implementation of a Kubernetes JMeter cluster.  It will allow for the execution of a JMeter test script on an arbitrarily sized [JMeter test rig](https://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html#distributed-testing) and then will generate:
- A [JMeter DashBoard Report](http://jmeter.apache.org/usermanual/generating-dashboard.html#generation)
- [JMeter Test Log \(JTL\)](https://jmeter.apache.org/usermanual/get-started.html#non_gui)
   
To provide a simple method of deleting the test rig after the test execution we are using the [Kubernetes Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) feature which allows for an expedited method to delete everything created.

[**kubectl delete namespace *NameSpacdName***](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete)

## Features
- Certified on [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/)
- PowerShell scripts automate all of the features 
  - All samples that I found assumed a Linux client and no PoserShell
  - Linux based containers whicdh allow for K8S native pod clustering
- PowerShell script to create your own docker images if you don't want to use mine at https://cloud.docker.com/u/shadowpic
- Build the cluster from Docker root containers

## Dependendies
- Docker for Windwos Desktop: https://docs.docker.com/docker-for-windows/install/
- Kitematic: https://github.com/docker/kitematic
- AKS: https://azure.microsoft.com/en-us/services/kubernetes-service/
- To create a basic load test
  - JmMter 5.x: https://jmeter.apache.org/download_jmeter.cgi
  - JMeter Plugins: https://jmeter-plugins.org/
  - A web site somewhere that you can break and not get in trouble.  :)

## Scripts

### Building the Docker Images
**File Name:** builddocker.ps1
Must be hand edited to point this to your Docker repository and then update the relevent scripts which is beyond the scope of this document.

### Creating the K8S JMeter Cluster
**File Name:** jmeter_cluster_create.ps1
This will create 1 JMeter Master pod and 2 or more JMeter Slave pods.  It also creates the K8S master configurtion map and the JMeter Slaves service.  Once the script is completed you can check the status of the Pods with the following kubectl command:

**kubectl -n \<K8S NameSpace\> get pods**

- -tenant <K8S NameSpace> 
  - Will create a K8S NameSpace and use that to create and deploy all services
- -ScaleSlaves [integer larger than 2]
  - OPTIONAL parameter which allows for a cluster larger than the default of 1 master and 2 slaves
  
### Running the Test
**File Name:** run_test.ps1
- -tenant <K8S NameSpace> 
- -TestName < full or relative path to the JMeter test script >
- -ReportFolder < folder name to publish the results of the test >

### Cleaning Up

To delete tbe cluster you run the following command:
**kubectl delete namespace NAMESPCENAME**

## Supporting files

- Docker Files
  - jmeterbase-docker
    - Creates a JMeter docker base image which is used to create the Master and Slave Docker images
  - jmetermaster-docker
    - Creates the JMeter master docker image
  - jmeterslave-docker
    - Creates the JMeter slave docker image
- K8S files
  - jmeter_master_configmap.yaml
    - configures the entry point for the JMeter master to enumerate the slave pods
  - jmeter_master_deploy.yaml
    - K8S master pod definition
  - jmeter_slaves_deploy.yaml  
    - K8S slave(s) pod definition                                           
  - jmeter_slaves_svc.yaml
    - K8S service which, among other things, allows the jmeter_master_configmap to enumerate the slave pods
- Miscellaneous
  - load_test_run
    - the linux shell script used to execute the tests
    - CRITICAL NOTE:  must be in linux line endings and not DOS                                                      

# References
- https://kubernauts.io/en/
- http://www.testautomationguru.com/jmeter-distributed-load-testing-using-docker/
- https://www.blazemeter.com/blog/make-use-of-docker-with-jmeter-learn-how 