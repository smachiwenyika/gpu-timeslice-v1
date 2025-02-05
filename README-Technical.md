a. Enabling GPU Slicing on EKS:

   1. Deploy NVIDIA Device Plugin with Time-Slicing:
   
    - Deploy the NVIDIA Device Plugin:

       helm repo add nvidia https://nvidia.github.io/k8s-device-plugin
       helm repo update

    - Install the plugin with time-slicing enabled:
    
       helm install nvidia-device-plugin nvidia/k8s-device-plugin --set timeSlicing.enabled=true


   2. Configure Pods to Use Shared GPUs:
      - In your pod specifications, request GPU resources as needed specify the following in your pod's resource limits: 

     resources:
       limits:
         nvidia.com/gpu: 1
    
b. Integrating GPU Slicing with Karpenter Autoscaler:

   1. Install Karpenter:
      - follow the official Karpenter installation guide to deploy Karpenter in your EKS cluster.

   2. Define a Karpenter Provisioner for GPU Nodes:
      - Create a provisioner configuration that specifies the requirements for GPU nodes. 

   ---
     apiVersion: karpenter.sh/v1alpha5
     kind: Provisioner
     metadata:
       name: gpu-provisioner
     spec:
       requirements:
         - key: node.kubernetes.io/instance-type
           operator: In
           values: ["g4dn.xlarge", "g4dn.2xlarge", "g4dn.4xlarge", "g4dn.12xlarge"]
         - key: karpenter.sh/capacity-type
           operator: In
           values: ["on-demand"]
       provider:
         subnetSelector:
           karpenter.sh/discovery: ${CLUSTER_NAME}
         securityGroupSelector:
           karpenter.sh/discovery: ${CLUSTER_NAME}
       labels:
         nvidia.com/gpu: "true"
       taints:
         - key: nvidia.com/gpu
           effect: NoSchedule
       limits:
         resources:
           cpu: 1000
       ttlSecondsAfterEmpty: 300
     
3. Deploy Workloads with GPU Requirements:

      - When deploying workloads that require GPU resources, ensure that your pod specifications include the necessary tolerations and resource requests:     
   
      ---
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: gpu-workload
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: gpu-workload
          template:
            metadata:
              labels:
                app: gpu-workload
            spec:
              tolerations:
                - key: nvidia.com/gpu
                  operator: Exists
                  effect: NoSchedule
              containers:
                - name: gpu-container
                  image: your-gpu-enabled-image
                  resources:
                    limits:
                      nvidia.com/gpu: 1
    
