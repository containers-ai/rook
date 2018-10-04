## What is Alameda

Alameda is an intelligent resource orchestrator for Kubernetes, providing the features of autonomous balancing, scaling, and scheduling by using machine learning. Alameda learns the continuing changes of computing resources from K8S clusters, predicts the future computing resoruce demands for pods and nodes, and intelligently orchestrates the underlying computing resources without manual configuration.

For more details, please refer to https://github.com/containers-ai/Alameda

## Features

The main purpose of Alameda is to orchestrate computing resources orchestration by utilizaing AI-powered prediction capability. With this, IT admins can leave one of the hardest problems of running Kubernetes to Alameda. 

Our first take is to provide the following features, which we consier they are extremely usable for Rook. 
- Disk health and life expectancy prediction  
    Based on a disk's S.M.A.R.T. value, Alameda predicts how bad a commodity disk is going to fail in a near future. Rook can stop provisioning volumes from a critical status disk.
    
- Automatic resource configuration for pods  
    Alameda includes a community edition AI Engine which generates loading data in the future time. The AI Engine learns patterns from the historical performance metrics of each node and pods running on it. For example, it predicts CPU metrics of next 24 hours in 1-hour internal. With this prediction info of nodes and pods, Alameda can automatically configure pod resource settings without human interventino. Alameda will provides the following features:

>- Continously autoamte compute resoruce (i.e. CPU and memory) requests and limits configurations for pods. It takes both a node and pods running on this node into consideration. For Rook, it means users don't need to configure for pods of mgr, mon, and osd.
>- Recommend predicted low loading time window for a predicted bad disk replacement

## How Alameda works

1. Alameda data collector gets metrics from Prometheus (e.g. CPU, memory, Ceph metrics)  
No Alameda Agent is needed.
2. Alameda AI engine generates resource prediction
3. Alameda resource operator monitors Rook cluster CRD  
Alameda will monitor rook CRDs with Alameda annotations. For example, Rook user can add ```container.ai/autoscale``` and ```container.ai/diskFailurePrediction``` annotations in their *cluster.yaml* as:
<pre>
    apiVersion: v1
    kind: Namespace
    metadata:
      name: rook
    ---
    apiVersion: rook.io/v1alpha1
    kind: Cluster
    metadata:
      name: rook
      namespace: rook
      <b>annocations:
        container.ai/autoscale: true
        container.ai/diskFailurePrediction: true
        container.ai/capacityTrendingPrediction: true</b>
    spec:
      versionTag: v0.5.1
      dataDirHostPath:
      storage:
        useAllNodes: true
        useAllDevices: false
        storeConfig:
          storeType: filestore
          databaseSizeMB: 1024
          journalSizeMB: 1024
</pre>

4. Alameda generates resource operation planning for Rook cluster
5. Alameda update cluster CRD  
Alameda will (1) update CRD spec, or (2) update CRD spec with new definitions of planning. No matter they are (1) or (2), Rook needs to change the logic of watching CRD.

![work_flow](./Alameda_work_with_Rook.png)

