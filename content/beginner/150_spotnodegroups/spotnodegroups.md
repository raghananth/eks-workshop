---
title: "Add Spot managed node group"
date: 2021-03-08T10:00:00-08:00
weight: 10
draft: false
---
We have our EKS cluster and nodes already, but we need some Spot Instances configured to run the workload. We will be creating a Spot managed node group to utilize Spot Instances. Managed node groups automatically create a label - eks.amazonaws.com/capacityType - to identify which nodes are Spot Instances and which are On-Demand Instances so that we can schedule the appropriate workloads to run on Spot Instances. We will use `eksctl` to launch new nodes running on Spot Instances that will connect to the EKS cluster.

First, we can check that the current nodes are running On-Demand by checking the **eks.amazonaws.com/capacityType** label is set to **ON_DEMAND**. The output of the command shows the **CAPACITYTYPE** for the current nodes is set to **ON_DEMAND**.

```bash
kubectl get nodes \
  --label-columns=eks.amazonaws.com/capacityType \
  --selector=eks.amazonaws.com/capacityType=ON_DEMAND
```

![OnDemand Output](/images/spotworkers/spot_get_od.png)

#### Create Spot managed node group

We will now create a Spot managed node group using the --spot option in `eksctl create nodegroup` command.

```bash
eksctl create nodegroup \
  --cluster=eksworkshop-eksctl --region=${AWS_REGION} \
  --managed --spot --name=ng-spot \
  --instance-types=m5.large,m4.large,m5d.large,m5a.large,m5ad.large,m5n.large,m5dn.large
```

{{% notice warning %}}
Note, the instance types above might not be present in your region. To select instance types that meet that criteria in your region, you could run `eksctl create nodegroup` command with Instance Selector options flags to help you select compatible instance types for your application to run on. Execute the command `eksctl create nodegroup --cluster=eksworkshop-eksctl --region=${AWS_REGION} --managed --spot --name=ng-spot --instance-selector-vcpus=2 --instance-selector-memory=8 --instance-selector-cpu-architecture=x86_64` to get a diversified selection of instance types available in your region of choice that meet the criteria of being similar to m4.large (in vCPU and memory terms). Benefits of running instance selector options include filtering of instances supported by EKS in a particular region and access to the EC2 Instance types based on vCPU and memory requirements.
{{% /notice %}}

Spot managed node group creates a label **eks.amazonaws.com/capacityType** and sets it to **SPOT** for the nodes.

The Spot managed node group created follows Spot best practices including using [capacity-optimized](https://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/) as the spotAllocationStrategy in the underlying EC2 Auto Scaling group, which will launch instances from the Spot Instance pools with the most available capacity, aiming to decrease the number of Spot interruptions in our cluster (when EC2 needs the capacity back).

{{% notice info %}}
The creation of the nodes will take about 3 minutes.
{{% /notice %}}

#### Confirm the Nodes

Confirm that the new nodes joined the cluster correctly. You should see 2 more nodes added to the cluster.

```bash
kubectl get nodes --sort-by=.metadata.creationTimestamp
```

![All Nodes](/images/spotworkers/spot_get_nodes.png)
You can use the **eks.amazonaws.com/capacityType** to identify the lifecycle of the nodes. The output of this command should return 2 nodes with the **CAPACITYTYPE** set to **SPOT**.

```bash
kubectl get nodes \
  --label-columns=eks.amazonaws.com/capacityType \
  --selector=eks.amazonaws.com/capacityType=SPOT
```

![Spot Output](/images/spotworkers/spot_get_spot.png)


