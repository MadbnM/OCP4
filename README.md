# OCP4


## ElasticSearch and Logging

https://www.redhat.com/en/blog/openshift-container-storage-4-and-logging-elasticsearch-and-cluster-logging-operators


Elasticsearch Storage Selection Guidance
There are multiple factors that an OpenShift Admin or a Developer should consider while deploying OpenShift Cluster Logging Service or Elasticsearch in general on OpenShift. Table 2 summarizes such factors in accordance with different Elasticsearch replication policies and available storage options on OpenShift.

Based on the summary and empirical evidence described in the following tables and later sections of this document, we recommend using OpenShift Container Storage for OpenShift Logging Service and general-purpose Elasticsearch deployments. Elasticsearch offers replication at per index level to provide some data resilience. Additional data resilience can be provided by deploying Elasticsearch on top of a reliable storage service layer offering further resilience capabilities. This additional data resilience can enhance Elasticsearch service availability during broader infrastructure failure scenarios mentioned in appendix Table 12.

![image](https://user-images.githubusercontent.com/81707384/210450854-4fc5fda3-66ce-4d4d-90fb-4a1cc9358cc5.png)


Table 1: Recommended storage option for OpenShift Cluster Logging (Elasticsearch) use case

![image](https://user-images.githubusercontent.com/81707384/210450939-fb587a45-c3d7-480b-b664-263d1758950a.png)

Table 2: Feature comparison summary for choosing the optimal storage for OpenShift Cluster Logging (Elasticsearch)

Key Measures of Resilience for Elasticsearch
We captured the following key measures of resilience and performance to inform this brief:

Steady-state ES index append median throughput for OpenShift Container Storage and EBS storage classes.

Degraded-state ES index append median throughput, completion time, index append error % and latency.

Workload Benchmarking Results Summary
To measure ES resiliency, we have simulated ES node failure while the client is performing index append operation to the ES cluster. For details about the benchmarking environment and methodology refer to the appendix. As depicted in Chart 1 ES, when deployed on OpenShift Container Storage, did not show any sign of workload failure, the index-append error rate found to be 0 (highly desirable by applications).

In a slightly different test where ES was deployed on EBS storage class, resulted in an error rate of 100% as well as higher latency, completion time and lower index-append throughput. This degraded resiliency during the infrastructure failure situation (failed node in this case) can be attributed to the principal design of the AWS Elastic Block Storage (EBS) service. AWS EBS is a zonal service and is limited to a single AZ within a region, as such during failure when OpenShift spawns a new ES pod in a different AZ or site, the pod is unable to cross mount Persistent Volume across AZ, as result, ES cluster enters ERR state.

OpenShift Container Storage does not exhibit this behavior as it supports cross-AZ persistent storage, which can withstand a variety of infrastructure failure scenarios as mentioned in Appendix Table 12. In the public cloud OpenShift environment, OpenShift Container Storage nodes are deployed in multi-availability zones within a region, forming a distributed and highly available storage cluster. As such storage provisioned by OpenShift Container Storage is available across availability zones and can seamlessly handle failure of AZs without losing data or impacting application service availability, helping applications consuming OpenShift Container Storage storage enjoy higher resiliency.

Chart 1: Elasticsearch workload characterization steady vs degraded state
Chart 1: Elasticsearch workload characterization steady vs degraded state

Why choose OpenShift Container Storage for OpenShift Logging Service
Whether you choose to deploy OpenShift on-premise or in the public cloud, you require persistent storage to aggregate the logs from your OpenShift Container Platform cluster, such as node system logs, application container logs, and so forth. OpenShift cluster logging component logStore implements Elasticsearch to store and index the logs for an OpenShift cluster.

Table 3 summarizes the comparison between the current default storage (emptyDir), OpenShift Container Storage and AWS EBS for OpenShift Logging Service.

Table 3 : Storage options comparison for OpenShift Logging Service
Table 3 : Storage options comparison for OpenShift Logging Service

OpenShift Logging is a key service of the OpenShift Container Platform monitoring stack, hence data persistence for logging service is not optional rather it is mandatory. Comparing across available storage options in the public cloud as well as on-premise, OpenShift Container Storage stands out to be the ideal storage option for providing data persistence to OpenShift logging service.

As summarized in Table 4, Multi availability-zone storage resiliency, multi interface support as well as a common storage experience across infrastructure types are some of the most prominent features of OpenShift Container Storage, which makes it ideal for being used as the preferred storage backend for OpenShift logging service.

Table 4 : Storage options comparison for OpenShift Logging Service across infrastructure
Table 4 : Storage options comparison for OpenShift Logging Service across infrastructure
Appendix
Benchmarking Environment
The benchmarking environment consists of one OpenShift cluster configured with OpenShift Container Storage and EBS storage classes. To test Elasticsearch on different storage classes, we created two OpenShift projects one for ES on OpenShift Container Storage storage class and ES on EBS storage classes. For workload generation, we used the Rally tool running within each OpenShift project. The following tables further describe the benchmarking environment.

Tables 5 through 7
Figure 1: Red Hat OpenShift Cluster Logging powered by OpenShift Container Storage
Figure 1: Red Hat OpenShift Cluster Logging powered by OpenShift Container Storage
Benchmark Overview
For this testing we used OpenShift 4.3 cluster for base infrastructure, OpenShift Container Storage 4.3 / EBS for storage and Cluster Logging operator for logstor (Elasticsearch) component. The cluster logging operator, logstor was re-configured multiple times with different storage classes and redundancy policies, as described in Table 2. Each Elasticsearch pod was provided with 1 x 200GB PV. For workload generation we used rally tool with up to two instances. We ran a myriad of test combinations where we modulated different settings including replication factor, ES shards count and storage class backends. The results shown above depict the results in each category across various tests.

Feature Comparison Matrix
We ran several different tests to help you choose the right storage for your Elasticsearch deployment. Table 8 summarizes our results in the form of a rating against each feature and storage backend. Refer to Appendix Tables 10, 11 and 12 to understand the redundancy policies, rating reference and failure scenarios used here. To help you make the most out of this matrix, below is a one liner description of resiliency and reliability in the context of our results.

ES Resiliency: Ability of Elasticsearch to withstand certain types of failure and heal itself (higher the better).

ES Reliability: In case of failure, it's the ability of Elasticsearch service to remain functional from consumer perspective (higher the better).

Table 8: Feature comparison matrix for choosing the optimal storage for Openshift Cluster Logging (Elasticsearch)
Table 8: Feature comparison matrix for choosing the optimal storage for Openshift Cluster Logging (Elasticsearch)

Basic performance details
The goal of this study was to understand the resiliency characteristics of the Elasticsearch cluster when deployed using different storage backends, the results for the same has been described above.

In the remaining time, we ran a basic test to understand the performance penalty imposed by different Elasticsearch native redundancy levels. We did not intend to stress test the Elasticsearch cluster or storage subsystem with this test. Chart 2 describes the performance of Elasticsearch at different redundancy levels when a single client workload was applied. We did not observe any bottleneck on the storage subsystem as well as on Elasticsearch cluster resources.

Chart 2: Comparing elasticsearch redundancy level performance
Chart 2: Comparing Elasticsearch redundancy level performance

Miscellaneous details
Table 7 summarizes the recommended and configurable storage technologies for OpenShift Container Platform hosted logging service and Table 8 describes different policy types that could be specified in Custom Resource Definition (CRD) to provide data redundancy and resilience to failure.

Tables 9 and 10
Table 9 describes the rating reference used to compare different test results and Table 10 lists possible degraded state scenarios that could trigger degradation in the Elasticsearch cluster’s health.
Tables 11 & 12
