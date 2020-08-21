# HPA using Cloudwatch Metrics.

**What is HPA ?** 

HPA is Horizontal Pod Autoscaler, It is used to auto scale up and down number of kubernetes pods based on observed CPU utilization or any other Custom metrics. So basically the HPA will help us auto scale our pods as per the observed resource from a particular metric we are using in our cluster. 

**What is Cloudwatch ?** 

Amazon CloudWatch is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events, providing you with a unified view of AWS resources, applications, and services that run on AWS and on-premises servers.

**Why are we using the Cloudwatch metric for HPA ?**  

Cloudwatch is an effective monitoring tool. Majority of the KOPS using customers might be using the cloud as AWS, so probably most of our metric and log collection should happen through Cloudwatch. HPA using cloudwatch will be effective in that way to auto scale our applications pods, as the developers might be more confident in creating Custom metrics for CloudWatch. 

Cool !!!  How can we go ahead with this HPA setup ?

**What we need to proceed.** 

 * Working KOPS cluster.
 * A custom cloudwatch metric to test. 
 * Kubernetes Custom Metrics API and External Metrics API for AWS CloudWatch metrics.
 



**Step by step procedure.**  

1. Create a IAM policy with following policy templete and attach the same to node and worker roles of cluster.

++++++++++++++++++++++++++++++++++++++++++++
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:GetMetricData"
            ],
            "Resource": "*"
        }
    ]
}    


+++++++++++++++++++++++++++++++++++++++++++++++

Make sure you use service account creation as below in adapter.yaml.

++++++++++++++++++
kind: ServiceAccount
  apiVersion: v1
  metadata:
    annotations:
      eks.amazonaws.com/role-arn: arn:aws:iam::awsaccountID:role/policyname
    name: k8s-cloudwatch-adapter
    namespace: custom-metrics
++++++++++++++++++


2. Integrate the Cloudwatch adapter. 

    kubectl apply -f https://raw.githubusercontent.com/awslabs/k8s-cloudwatch-adapter/master/deploy/adapter.yaml

This creates a new namespace custom-metrics and deploys the necessary ClusterRole, Service Account, Role Binding, along with the deployment of the adapter.

 3. Verifying the deployment using below command. 


    kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1" | jq .

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "external.metrics.k8s.io/v1beta1",
  "resources": [
  ]
}
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

If the result is like above, then adapter installation is fine. 

So we have completed the Adapter configuration within our cluster. 
Deploy a sample application otherwise we need a deployed application which has a custom metric already configured. 

  4. Create a custom metric yaml and apply. Example as below.

  > kubectl apply -f https://github.com/liminpb/hpa-cloudwatch/blob/master/external.yaml


  5. Next steps to create a HPA for the custom metric. 

  > kubectl apply -f https://github.com/liminpb/hpa-cloudwatch/blob/master/hpa.yaml


Using this HPA, it will trigger the metric target value specified and scale  in an out the number of pods between 1 to 5.

We can go ahead with a stress test and verify the HPA performance using below command. 

		 > kubectl get hpa -w




