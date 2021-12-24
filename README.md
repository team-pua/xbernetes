# xbernetes
A Kubernetes API Server that spans multiple regions or accounts on AWS.

# Background
Started from a container orchestration system, Kubernetes is now becoming a popular distributed system framework with the growth of its tooling and ecosystems.
Developers can efficiently build their own controllers and encapsulate the reconciliation logics in a CR (customized resource).
With the thrive of Kubernetes community, many softwares are now delivered together with an operator that automates maintenance operations.
Basically an operator will subscribe to the Kubernetes API server, watch and react to events from some certain CRs.
Cloud SaaS vendors also want to leverage this programing paradigm, but the problem is, those SaaS vendors typically don't have ownership of the API server from the tenants, so it's challenging to manage CRs in tenant's clusters.
Typically the vendor need to access tenants' API server or deploy an agent in tenants' clusters that subscribe to tenant's API server.

# Motivation
To manage CRs in tenants' cluster, currently we need to establish network connectivity between SaaS vendors and tenants.
This is challenging, costly and insecure.
The challenge is that if vendor and tenants' VPCs are from different regions on AWS, we need to deploy extra resources such as transit gateway to route the traffic.
The cost for those infra could accumulate, and the the complexity of the system would grow as well.
It's difficult to troubleshoot and maintain the system especially when something goes wrong.
Finally, it's not ideal to expose API server from vendor or tenant to its counterpart since it increases attack surface.
It would be nice if we can get rid of the network connectivity requirement between vendor and tenant, and switch to a data-driven approach to transfer information.

# Solution
The root cause for the issue above is that, the storage backend for API server (_i.e._ etcd) is coupled with network infrastructure.
The API server itself is stateless.
Tenant cannot directly access vendor's API server simply because we cannot strech the backend etcd cluster across vendor and tenant's VPCs directly, and that's why we need extra works to establish network connectivity between the two.
To solve this problem, we can replace etcd with AWS services that are natively availble across regions such as S3, SNS and SQS.
By deploying a stateless API server that connects to those AWS services in both vendor and tenant's VPCs, we no longer need to establish network connectivity between VPCs.

# Limitations
There're limitations for this approach.
Typically, it's required for API server to have connectivity to each node or pod in the cluster.
This networking model for typical kubernetes might not be met in this scenario.
Some subresources (_e.g._ proxy, log) or functions (_e.g._ health check) might not work.
In general, we're not targetting at getting conformance with standard kubernetes but focusing at the use case described above.

