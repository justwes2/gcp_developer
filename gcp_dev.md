# GCP Developer Exam Review

### Pre Exam Questions
1. You have a service running on Compute Engine virtual machine instances behind a global load balancer. You need to ensure that when the instance fails, it is recovered. What should you do?
 
    Set up health checks in the managed instance group configuration- the managed instance group health check will recreate instances when they fail. This is the *platform native* way to do this. [Docs](https://cloud.google.com/compute/docs/instance-groups/creating-groups-of-managed-instances#monitoring_groups%29)

2. You have an application that accepts inputs from users. The application needs to kick off different background tasks based on these inputs. You want to allow for automated asynchronous execution of these tasks as soon as input is submitted by the user. Which product should you use?

   [Cloud Tasks](https://cloud.google.com/tasks/docs/dual-overview)- "lets an applications perform developer-defined pieces of work, called tasks, outside of a user request". It is asynchronous- there are no strong guarantees on FIFO or timing. Tasks(items of work) are added to queues. Tasks are performed later by workers. Tasks manages issues like user-facing latency costs, crashes, resouce limitations, and reentry mgmt. Typically used to speed user response by pushing database updates to worker/task. 

3. As part of their expansion, [HipLocal](goo.gl/NpR3y2) is creating new projects in order to separate resources. They want to build a system to automate enabling of their APIs. What should they do?

   Use the service management API to enable the compute API- Case study shows use of VMs, not storage. Don't need to create a new services. Service Mgmt allows service producers to publish their services so they can be discovered/used by service consumers([Docs](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rest/))

4. Your team is using App Engine to write every Cloud Pub/Sub message to both a Cloud Storage object and a BigQuery table. You want to achieve the greatest resource efficiency. Which architecture should you implement?

   One topic, 2 subscriptions, 2 app engines:
   ```
                 -Pub/Sub Push Subscription - App Engine/Write to BigQuery
   Pub/Sub Topic-
                 -Pub/Sub Push Subscription - App Engine/Write to Cloud Storage
    ```
    Because each App Engine can run/fail/retry independently. Using a single stream, if one write fails, it could lead to duplication

5.  Your teammate has asked you to review the code below. Its purpose is to query account entities in Cloud Datastore for those with a balance greater than 10000 and an age less than 4. Which improvement should you suggest your teammate make?
![alt text](https://lh3.googleusercontent.com/rlfMjBQ84eIEeHBy52q14N54IFPz0DQS7Sx9sVht7kVbMtIdw2X7Ig75TcoJi9_V1sCMU7ORDA=w740 "DataStore Query")
   Send two queries- one for balances over 10000, and another for ages less than 4- and compute the intersection- two inequality comparisons aren't permitted in a Datastore query and it requires two queries to be merged

6. Your organization has grown, and new teams need access to manage network connectivity within and across projects. You are now seeing intermittent timeout errors in your application. You want to find the cause of the problem. What should you do?

   Configure VPC flow logs for each of the subnets in your VPC- uses the substrate specific logging to capture everything

### [Study Guide](https://cloud.google.com/certification/guides/cloud-developer/)

#### Section 1: Designing highly scalable, availible, and reliable cloud-native applications
1.1 Designing performant applications and APIs
- IaaS vs Container as a Service vs PaaS (e.g. autoscaling implications)
    - IaaS: VMs, managed service groups for autoscaling, can configure for HA, failover
    - CaaS: GCE, autoscales, failover, etc managed
    - PaaS: Cloud Funtions, all ops managed by platform, infintely scalable
- Portablity vs. platform-specific design: As a rule, the more a solution is 'managed', the less portable it is. VMs move as images accross providers fairly well, but Serverless Functions must be rewritten to accomidate the specific CSP. Containers are an interesting case- the container itself is designed to be portable, and K8s should be portable by design. (Not enough exposure to validate)
- Evaluating different services and technologies:TODO wow this is frustratingly vague. Things to look for:
    - Level of mgmt: as services are more managed, it decreases ops workload. However, sometimes this sacrifices flexiblity for more specialized configuration. (I.e. a managed DB service patches according to the service's schedule, not when the app is ready to support(not a great ex, but that's the idea))
    - Cost: TCO, managed services can be more expensive, but *can* deliver more value. 
    - HA/DR/Failover options: is the tech regional? Can it support multiregional solutions? How much of that is managed
- Operationg system versions and base runtimes of services:
    - VMs: Marketplace images support most anything you can imagine, if its not there, roll your own image. 
    - Containers: GAE Standard supports: Python 2.7/3.7, Java 8, PHP 5.5/7.2, Go 1.9/1.11, and Nodejs (not sure what version). GAE also supports custom containers with flexible runtimes. 
    - Cloud Funtions: Node.js 6/8, Python 3.7, Go 1.11
    - Cloud SQL: MySQL 5.5/5.6/5.7, Postgres 9.6
- Georaphic distribution of Google Cloud services: GCP has a bunch of regions, adding more all the time. Each region has at least 2 zone, which are geographically separate. VPCs can span regions, subnets can span zones. Different services have different geographic coverage.
- Microservices: Unlike traditional 'monoliths', microservice architectures involve units of functionality being broken up and managed separately. By losely coupling systems, app teams can move more quickly, and update services in smaller, tighter release cycles.
- Definging a key structure for high write applications using Cloud Storage, Cloud BigTable, Cloud Spanner, or Cloud SQL: All of these services are managed to different degrees. Ensuring that the key will not result in lots of write activity in a single shard of the database/storage (hotspotting) is essential. The best key will depend on the nature of the data and the application/queries. If a shard is getting 'hot', consider hashing additional information into the key to break up the writes.
- Session management: Managed services like GAE and GKE, as well as google's load balancers, have sticky session as a managed feature so that users will interact with the same container throughout the duration of an interaction, to improve performance. 
- Deploying and securing an API with cloud endpoints: Endpoints is an API Gateway style api manager that allows logging, securing, and monitoring of endpoints. It is availible in OpenAPI, gRPC, and App Engine standard flavors. See the [Docs](https://cloud.google.com/endpoints/docs/)
- Loosely coupled applciations using asynchronous Cloud Pub/Sub events: Having services push to pub/sub allows supports reliable, scalable service designs. Once an events is written to a pub/sub subscription, a worker of whatever flavor will pick it up when able. This prevents data loss if a service is unavailible. Additionally, if an event is not marked as completed before it times out, it will be picked up by another worker. This prevents data loss when workers fail or error out. TODO confirm visibility timeout
- Health checks: Managed instance groups and K8s use health checks to ensure availiblity. If a vm or container fails a health check, those services can spin up another resource to ensure healthy compute resources are always availible
- Google-recommended practices and documentation: Google has lots of docs full of lots of opinions. Good luck with that. Most service docs will have a best practices page/section to refer to. 

1.2 Designing secure applications
- Applicable regulatory requirements and legislation: Varies depending on location (GDPR), industry (HIPPA), and other stuff. See [Docs](https://privacy.google.com/businesses/compliance/#!?modal_active=none)
- Security mechanisms that protect services and resoures: varies from service to service. IAM can be used to limit user access and communication between resources. Ensure that buckets aren't public, firewalls aren't open. 
- Storing and rotating secrets: IAM service account keys are automatically rotated. GCP's [Secrets Managment](https://cloud.google.com/solutions/secrets-management/) can be used for other secrets. Rotate your keys/secrets. Third party products like vault also exist- not sure google cares.
- IAM roles for users/groups/service accounts: All IAM entities can be assingned a role at the project level (can also be asigned at the folder or org level). Using roles can limit access to principle of least privilige. 
- HTTPS certificates: You can put an SSL cert on a load balancer for HTTPS traffic. GCP will manage certs, or you can bring your own. You can also attach an SSL cert to an App Engine
- Google-recommended practices and documentation: [tl;dr](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations#secure-apps-and-data): Ensure VPCs are secured (service controls), put HTTPS load balancing in place, use Cloud Armor service, Use Cloud Identity-Aware Proxy service
1.3 Managing application data: 
- Defining database schema for Google-managed databases (Datastore, Spanner, BigTable, BigQuery): 
    - [Datastore](https://cloud.google.com/datastore/docs/concepts/entities): objects are called entities, entities have properties. Doesn't need schema upfront. Doc DB will just hold hierachical info by unique key. 
    - [Spanner](https://cloud.google.com/spanner/docs/schema-and-data-model): Much like traditional SQL db. Specify schema of table on creation. Data strongly typed. Can define secondary indexes.
    - [BT](https://cloud.google.com/bigtable/docs/schema-design): Very similar to RDBMS. Each table has one index- the row key. Rows are sorted lexicographically by row key. All operations are atomic at the row level. Reads and writes should be distributed evenly across table. Keep all info for entity in single row. Related entities should be stored in adject rows. BT tables are sparse (create lots of mostly empty columns)
    - [BQ](https://cloud.google.com/bigquery/docs/schemas): specific when you load data into table and when you create empty table. Can auto-generate schema, or retrieve from source.
- Choosing data storage options based on use case considerations:
    - Cloud storage signed URLs for user uploaded content: provide user with a URL to get access to a bucket for a set duration without needing to provide a google account. [Docs](https://cloud.google.com/storage/docs/access-control/signed-urls)
    - Using Cloud Storage to run a static website: Connect bucket to Cloud DNS address to serve static content (usually based on an `index.html`). All website content must be publiclly accessable for this to work.
    - Structured vs. Unstructured data: Structured data should go somewhere relational- SQL/Spanner, unstructured should go Datastore or BT (for massive IoT streaming datasets)
    - ACID Transactions vs. analytics processing: Spanner offers ACID++, SQL is ACID, Datastore is ACID, BT is **not** ACID, BQ is ACID
    - Data volume: [Persistant disk](https://cloud.google.com/compute/docs/disks/) popped into my head, but probably more about [quantity](https://cloud.google.com/solutions/transferring-big-data-sets-to-gcp) of data. 
    ![alt text](https://cloud.google.com/solutions/images/tran5.png "data transfer")
    Use `gsutil` for smaller amounts, google transfer service for GCS or other CSP, and get a tranfer appliance for large amounts of data in a datacenter
    - Frequency of data access in Cloud Storage: Multiregional, Regional, Nearline, Coldline
- Working with data ingestion systems (Cloud pub/sub, storage transfer service):
    - Pub/Sub: subscription messaging system: Data is published to subscription, picked up, processed, and deleted
    - Storage transfer service: Hits http endpoints to sync data with other cloud blob storage solutions (S3)
- Following google-recommended practicies and docs: service by service. [Cloud Spanner](https://cloud.google.com/spanner/docs/best-practice-list)
1.4 Re-Architecting applications from local services to Google Cloud Platform
- Using managed services: Managed services are a gradient (VM->Managed VM group->GKE->GCE->Cloud Functions). Using managed services reduces ops overhead/costs. Managed services are more expensive, usually. Additionally, managed services are not as flexible in some cases.
- Using the strangler pattern for migration: The [strangler pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/strangler) is where a legacy application is migrated in pieces. A specific unit of functionality is identified, rearchitected, and replaced. Then another piece of functionality is identified, and the pattern repeats until the entire application has been migrated/modernized, and the old system can be decomissioned.  
- Google-recommended practices and documentation: Building scalable and resilient applications: [docs](https://cloud.google.com/solutions/scalable-and-resilient-apps). There are some whitepapers [here](https://cloud.google.com/solutions/migration-center/)
#### Section 2: Building and Testing Applications
2.1 Setting up your development environment
- Emulating GCP services for local application development: With GCP's scalablity, it is possible to have a dev environment where condition mirror production very closely. When that is not feasable, containers are a good way to make sure local dev matches the GCP environment very closely. Infrastructure as Code is a good tool to make sure resources are created the same way each time they are created, so that dev machines function closely matches prod function.
- Creating GCP projects: Projects are entities that contain and constrain resources. Every application should have a project per environment (dev, test, prod, etc). Projects allow for easy management of cost and access. 
2.2 Building a continuous integration pipeline:
- Creating a Cloud Source Repository and committing code to it: You can, but in a world with coca cola (github), why would you drink rite cola (CSR)? Source code repositories are a good thing. Use GCP's if it make sense for your org/project, but I can't find a use case where github wouldn't work just as well, if not better.
- Creating container images from code: Using docker (because what else would you use?), create a Dockerfile with the configurations needed for the container (base image, ports opened, scripts to run, etc). Use Cloud Build to build the image based on the Dockerfile:

    `gcloud builds submit --tag gcr.io/[PROJECT_ID]/[IMAGE_NAME] .`

Run from the directory (cloudshell) where the Dockerfile is located. More in [Docs](https://cloud.google.com/cloud-build/docs/quickstart-docker)
- Developing unit tests for all code written: All major coding languages and frameworks have testing associated. Building tests into the build process is a good way to make sure poor quality code doesn't make it to deployment. There are a number of ways to impliment testing. All code written should include tests in the PR before the code is merged or a ticket is closed. Code reviews are important to make sure that tests are adequete. There are tools to test test coverage. Once tests are being written and reviewed, the pipeline should run all tests before building new artifacts/deploying new versions. 
- Developing an integration pipeline using services (e.g. Cloud Build, Container Registry) to deploy the application to the target environment (e.g. development, testing, staging): For example, once PRs are merged into github/cloud source repository, a pipeline can build the container image, run all tests against the image, and if it passes, push the image into the container registry of the project associated with the desired environment- once a build in dev passes all tests, the image can be added to the container registry in the test account, for the testing team to vet, before approving to the production account. (Having a dedicated testing team is an anti-pattern in agile, but let's assume that not every team building pipelines in the cloud is perfectly agile yet)
- Reviewing test results of a continuous integration pipeline: The goal of a CI pipeline is going to vary depending on context- some orgs will push for completely automated builds where the test results are logged somewhere, perhaps posted to slack, and life moves on. Others are going to need a workflow where once tests have run, they are reviewed by an approving entitiy before promotion. 
2.3 Testing
- Performance testing: Per [tutorialspoint](https://www.tutorialspoint.com/software_testing_dictionary/performance_testing.htm), performance testing is testing of system parameters under workload. Tests measure scalablity, reliablity, and resource usage. Performance testing techniques include Load testing, Stress testing, Soak testing, and Spike testing.
- [Integration testing](https://martinfowler.com/bliki/IntegrationTest.html): tests how well independently developed units function in the system, versus unit testing, which tests how the units of code behave. 
-[Load testing](https://www.digitalocean.com/community/tutorials/an-introduction-to-load-testing): A subset of peformance testing, this involves simulating high load on the resources and measuring how they respond. Do autoscaling groups scale? Can the load balancer handle all the requests? Is the database keeping up will all the reads and writes? These are things to look for in load testing. 
2.4 Writing Code
- Algorithm design: Wow, that's multiple sememsters worth of material. When writing code, designing systems using the best algorithms will make systems more performant, decrease latency, and lead to a better user experence. There's a ton of content on that, like [this](https://www.geeksforgeeks.org/fundamentals-of-algorithms/). 
- Modern application patterns: In GCP, microservices is the big one. Layer or tiered models are also common- a web tier, a backend, and a database, for a simple example. Monoliths are still around, but in the context of the exam are proabably not considered modern. Most web frameworks support an MVC model- model-view-controller. This is similar to the tiered approach. More [here](https://techbeacon.com/app-dev-testing/top-5-software-architecture-patterns-how-make-right-choice)
- Efficiency: One quality of well designed systems is efficient- how much time/compute resources does it take to acheieve the desired output. Slack is a good example of an inefficient app- it can take gigs of ram on the client machine to run. 
- Agile methodology: its a thing. Sprints, stand ups, retros, scrum. 
#### Section 3: Deploying applications
3.1 Implementing appropriate deployment strategies based on the target compute environment (Compute, GKE, GAE). Strategies include:

   - Blue/green deployments: create a new resource target (VM, managed VM group, K8s service, GAE service) and cut the load balancer/routing to the new version. 
   - Traffic-splitting deployments: create the two services as above, but instead of a total cutover, split the traffic between the two services. You can use this for A/B testing on UI changes, for example.
   - Rolling deployments: This process involves slowly replacing instances running the old version with new ones. 
   - Canary deployments: A small amount of traffic is sent to the new service. If there are any issues, that traffic can quickly be sent back to the stable version without widespread issues for end users.

As long as service resources are behind a google load balancer, most of these strategies can be supported for most compute services. 

3.2 Deploying applications and services on Compute Engine.
- Launching a compute instance using GCP Console and Cloud SDK (gcloud)(e.g., assign disks, availiblity policy, ssh keys):
   - In the [console](https://cloud.google.com/compute/docs/instances/create-start-instance)
   - Using gcloud:
   ```
   gcloud compute instances create [INSTANCE_NAME] \
   --image-family [IMAGE_FAMILY] \
   --image-project [IMAGE_PROJECT] \
   --subnet [SUBNET_NAME] \
   --zone [ZONE_NAME]
   ```
- Moving a persistant disk to a different VM: Detach the disk from the first VM (ensure the disk was unmounted so no data loss occured), attach the disk to the new VM (you will then need to mount the disk inside the VM). If the target VM is in a differnt zone, use the `gcloud compute disks move` command.
- Creating an autoscaled managed instance group using an instance template: See the [Docs](https://cloud.google.com/compute/docs/instance-groups/creating-groups-of-managed-instances). tl;dr, you can create an instance template from an image- public or private. When creating the managed instance group, designate the desired template, and that is the image that will be used to autoscale the group.
- Generating/uploading a custom ssh key for instances: You can use [OS Login](https://cloud.google.com/compute/docs/instances/managing-instance-access) to manage ssh access to linux instances using IAM roles. If that is not an effective solution (not all users needing SSH access have IAM credentials), you can [manage ssh keys in metadata](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys). Run the `gcloud compute project-info add-metadata` to add the users' public keys who should have access. 
- Configuring a VM for Stackdriver monitoring and logging: Install the [agent](https://cloud.google.com/logging/docs/agent/installation). You can [configure](https://cloud.google.com/logging/docs/agent/configuration) to agent, but it will function out of the box. You can access your logs in the console or via the api.
- Creating an instance with a startup script that installs software: Use the following command:
    ```
    gcloud compute instances create example-instance \
        --metadata-from-file startup-script=PATH/TO/FILE/install.sh
    ```
   You can store the file locally or in a bucket. More [info](https://cloud.google.com/compute/docs/startupscript)
- Creating custom metadata tags: You can attach tags/labels to resources to filter for cost, routing, access, and other funtionality. More in the [Docs](https://cloud.google.com/compute/docs/labeling-resources).
- Creating a loadbalancer for Compute Engine instances: GCP offers a number of different load balancers, per [Docs](https://cloud.google.com/load-balancing/docs/load-balancing-overview):
    - [HTTPS](https://cloud.google.com/load-balancing/docs/https/): The most robust. Lots of features for routing rules, service based routing, global availiblity, and so on
    - SSL: Global, TCP with SSL offload, external
    - TCP Proxy: TCP without offload(doesn't preserve client IPs), global, external
    - Network TCP/UDP: No ssl offload, preserves client IPs, regional, external
    - Internal TCP/UDP: Regional, internal

3.3 Deploying applications and services on Google Kubernetes Engine.
- Deploying a GKE cluster: `gcloud container clusters create [arguments]`. [Clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster) can be zonal, regional, private (nodes not accessable on internet), and alpha (not recommended for public use)
- Deploying a containerized application to GKE: Create a [Deployment](https://cloud.google.com/kubernetes-engine/docs/concepts/deployment). A deployment is a set of identical pods running a set of containers defined in a manifest (YAML) file. Here's a [tutorial](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)
-  Configuring GKE application monitoring and logging: [StackDriver](https://cloud.google.com/monitoring/kubernetes-engine/) supports monitoring for GKE. Legacy StackDriver (GA) and SD Kubernetes Monitoring (beta) are offered. There is a Stackdriver K8s Monitoring Console dashboard showing metrics. 
- Creating a load balancer for GKE instances: GKE supports TCP/UDP and HTTP(S) load balancers for public access. TCP load balancers are not aware of individual HTTP(S) requests, and do not feature health checks. HTTP(S) loadbalancers use Ingress, and are sensitive to requests to make context aware decisions. They feature URL maps the TLS termination. GKE automatically configures health checks. More [here](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer). Internal load balancers are a service that can be configured like [so](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balancing). The service's `spec` will include `type: LoadBalancer` in the `service.yaml`. See example:
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: [SERVICE_NAME]
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
    labels:
        [KEY]: [VALUE]
    spec:
    type: LoadBalancer
    loadBalancerIP: [IP_ADDRESS] # if omitted, an IP is generated
    loadBalancerSourceRanges:
    - [IP_RANGE] # defaults to 0.0.0.0/0
    ports:
    - name: [PORT_NAME]
        port: 9000
        protocol: TCP # default; can also specify UDP
    selector:
        [KEY]: [VALUE] # label selector for Pods to target
    ```

- Building a container image using Cloud build: Create a build config `.yaml` file, like [so](https://cloud.google.com/cloud-build/docs/configuring-builds/create-basic-configuration). Add steps to the file, will existing images, and arguments with specific instructions. Once the config file looks right, you can use the `images` field to determine where the build will be stored. 

3.4 Deploying  an application to App Engine. 
- Scaling configuration: You can set scaling to determine the inital number of instances (don't like the overlap of terminology), how instances are created or stopped, and how long an instance has to handle a request. You can use auto scaling for dynamic instances, manual scaling for resident instances, and basic scaling for dynamic instaces. For dynamic instances, the App Engine scheduler decides if a request can be managed by existing instances or if another one must be created. You can use metrics such as target_cpu_utilization, target_throughput_utilization, and max_concurrent_requests to optimize scaling. Each instance has its own queue of requests. Scaling is configured in the `app.yaml` for the version of the service. More [here](https://cloud.google.com/appengine/docs/standard/python/how-instances-are-managed).
- Versions: App Engine deployments are versioned. Traffic splitting can be done along versions for canary deployments. This streamlines rollback when issues arise.
- Blue/green deployment: Since deployments are versioned, you can cut 100% of traffic from one version of the deployment to the new one, once it has been approved for prod use. 

3.5 Depoloying a Cloud Function
- Cloud Functions that are triggered via an event (e.g., Cloud Pub/Sub events, Cloud Storage object change notification events): When configuring a Cloud Function, it can subscribe to a pub/sub topic, and new writes can trigger the function, passing in information. Changes to objects in a bucket (upload, deletion, etc) can also trigger stuff.
- Cloud Functions that are invoked via HTTP: All Cloud Functions have an HTTP endpoint. Hitting that endpoint can trigger the function, and it will return the output of the function. More [here](https://cloud.google.com/functions/docs/calling/http).

3.6 Creating data storage resources
- Creating a Cloud Repository: Use github, or gitlab, or whatever. I defy you to show a use case where Cloud Repository is the best solution. In that [case...](https://cloud.google.com/source-repositories/docs/creating-an-empty-repository)
-  Creating a Cloud SQL instance: Can be [done](https://cloud.google.com/sql/docs/mysql/create-manage-databases) via the console, gcloud, or api.
- Creating compostie indexes in Cloud Datastore: Composite indexes index multiple property values per indexed entity- they support complex queries and are defined in the index config file (`index.yaml`). These are needed for the following: 
    - Queries with ancestor and inequality filters
    - Queries with one or more inequality filters on a property and one or more equality filters on other properties
    - Queries with a sort order on keys in descending order
    - Queries with multiple sort orders
    - Queries with one or more filters and one or more sort orders

    Per [docs](https://cloud.google.com/datastore/docs/concepts/indexes)
- Creating BigQuery datasets: [Can be done](https://cloud.google.com/bigquery/docs/datasets#bigquery_create_dataset-cli) via the console, command line, or API. When creating a dataset, its location is immutable. 
- Planing and deploying Cloud Spanner: [Whitepapers](https://cloud.google.com/spanner/docs/whitepapers) for later. Create an instance, a database within the instance, and schema for tables within the DB and insert data, per [Docs](https://cloud.google.com/spanner/docs/quickstart-console). You can query and write to the DB via the gcloud sdk in nodejs, python, and go, [per](https://cloud.google.com/spanner/docs/use-cloud-functions#functions-calling-spanner-python). 
- Creating a Cloud Storage Bucket: Go to cloud storage, click 'create bucket', choose a globally unique name.
- Creating a Cloud Storage Bucket and selecting appropriate storage class: When creating bucket, select from one of: multiregional, regional, nearline, and coldline. You can set up lifecycle policies to turn older objects to cheaper classes. Not all objects in the bucket must be of the same class, but regional buckets and multi regional buckets cannot support the other class (but both support nearline and coldline).
- Creating a Pub/Sub topic: create the topic in the UI or programmatically, then create a subscription. You can push messages to the topic and pick them up via the subscription. 

3.7 Deploying and implementing networking resources
- Creating an auto mode VPC with subnets: An auto mode VPC will have a subnet in each region. Custom VPC can be created with subnets only in designated zones.

- Creating ingress and egress firewall rules for a VPC (e.g., IP subnets, Tags, Service accounts): Firewall rules can target traffic using the following conditions:
    - All instances in the network TODO vs subnet
    - Instances with a matching network tag (a different entity than labels, a key value pair for grouping related resources)
    - Instances with a specific service account
    
    More [here](https://cloud.google.com/vpc/docs/firewalls)
- Setting up a domain using Cloud DNS: You will need to:
    - Get a domain name through a registrar. Google offers this service, but you can get it from others as well.
    - Get an IP address to point the A record to.
    - Create a managed public zone: a container for DNS records of the name suffix. It has a set of name servers that respond to queries. 
    - Create a managed private zone: like the public one, but only visible from specified VPCs.
    - Create the record for the IP address and the A record.
    - Create a CNAME record
    - Update domain name servers to push new recoreds. 

    Find the full version [here](https://cloud.google.com/dns/docs/quickstart).

3.8 Automating resource provisioning with Deployment Manager

Personally I prefer cloud agnostic tools (see ongoing github/cloud repo feud). I would look to Jenkins, Terraform, and Chef for this. [But...](https://cloud.google.com/deployment-manager/docs/quickstart)

You define your resources in `.yaml` files. The file can contain templates, which are similar to terraform modules- boilerplate resources that can be called in the resource file. A template is written in python or jinja2. You deploy resources using `gcloud`. Once the resource collection has been created, a manifest is created. This file, like terraform state files, holds the desired state configuration. The manifest is updated to reflect updated runs of the the deployment file. You can destroy resources in the file the same way. There is a [repository](https://github.com/GoogleCloudPlatform/deploymentmanager-samples) of samples (hosted on github) for various types of resources. See the [Docs](https://cloud.google.com/deployment-manager/docs/fundamentals)

3.9 Managing Service accounts
- Creating a service account with a minimum number of scopes required: After creating the service account, it should be added to only the roles with the permissions it will need (priniple of least privilage). For example, if a server needs to read data from files in a bucket, it should only have read rights to Cloud Storage, and no other rights. TODO can you add conditional rights to a specific resouce? See [Docs](https://cloud.google.com/iam/docs/granting-roles-to-service-accounts)
- Downloading and using a service account private key file: You can create a public/private key pair associated with a service role like [so](https://cloud.google.com/iam/docs/creating-managing-service-account-keys). Keys generated in the console vs the api will have slighly different structures. Make sure the workflow is standardize to avoid errors. This key can be used for non-GCP resorces to authenticate into the environment with that service account's permissions. 
#### Section 4: Integrating Google Cloud Platform Services
4.1 Integrating an application with Data and Storage services
- Enabling BigQuery and setting permissions on a dataset: There are a number of [permissions and roles](https://cloud.google.com/bigquery/docs/access-control) associated with BigQuery. Setting up a service account with the desired rights will allow compute resources to access BigQuery data at the desired level (read, write, delete, etc)
- Writing a SQL query to retrieve data from relational databases: All coding languages and frameworks will have their own tools/libraries for interfacing with SQL databases. Set the database endpoint to the one provided by GCP, and construct queries in SQL. Cloud Spanner allows you to query the db using the [SDK](https://cloud.google.com/spanner/docs/reference/libraries#client-libraries-install-python).
- Analyzing data using BigQuery: This is a devloper exam, not an analyst one. You can construct SQL-like queries in BigQuery, but for robust analysis, you will need an analytics tool, like [Datalab](https://cloud.google.com/datalab/docs/).
- Fetching data from various databases: Managed databases have SDKs that can be used for queries in addition to a more conventional interface library in appilcation code. 
- Enabling Cloud SQL and configuring an instance: Using this [tutorial](https://cloud.google.com/sql/docs/mysql/quickstart), you can create an instance using the console (or gcloud or the api). After the instance has been created, use `gcloud sql connect <instance_name> --user=root`, provide the root password used when creating the instances, and you will be brought to the MySQL or PostgreSQL prompt. You can create databases and upload data using standard SQL. 
- Connecting to a Cloud SQL instance: Use the `gcloud sql connect` command above. 
- Enabling Cloud Spanner and configuring an instance: You can create an instances and configure the schema etc using the [console](https://cloud.google.com/spanner/docs/quickstart-console). 
- Creating an application that uses Cloud Spanner: Cloud Spanner has client libraries in most major programming languages. See [more](https://cloud.google.com/spanner/docs/reference/libraries#client-libraries-install-python) on writing code to interface.
- Configuring a Cloud Pub/Sub push subscription to call an endpoint: The server [pushes](https://cloud.google.com/pubsub/docs/subscriber#push-subscription) messages in the subscription to preconfigured endpoints as HTTPS requests. If the server does not recieve a sucess code from the endpoint, it resends the message. You can create a push subscription (pull by default) using the following [command](https://cloud.google.com/pubsub/docs/admin#subscriber-create-push-gcloud): `gcloud pubsub subscriptions create mySubscription --topic myTopic --push-endpoint="https://myapp.appspot.com/push"`
- Connecting to and running a CloudSQL query: You can connect to a MySQL instance using the MySQL (or psql client for postgresql) client installed on another server, like [so](https://cloud.google.com/sql/docs/mysql/connect-admin-ip). Additionally, you can use `gcloud sql connect [INSTANCE_ID] --user=root` in the cloudshell to connect. You can run queries using the client. Connecting from different compute resources will have different [tools/methods](https://cloud.google.com/sql/docs/mysql/external-connection-methods).
- Storing and retriving objects from Google Storage: You can use the console, or the gcloud SDK to read and write objects to buckets.
- Publishing and consuming from Data Ingestion sources: [pub/sub](https://cloud.google.com/pubsub/) is a good multipurpose solution for this use case. Any data source (app, batch, logs) can write to it, and it can be picked up by many different services without data loss. TODO more
-  Reading and updating an entity in a Cloud Datastore transaction from an application: Use the gcloud SDK: 
    - `get(key, missing=None, deferred=None, **transaction=var**, eventual=False)` ([Python](https://googleapis.github.io/google-cloud-python/latest/datastore/client.html#google.cloud.datastore.client.Client.get))
    - `put(entity)` to [add or update](https://googleapis.github.io/google-cloud-python/latest/datastore/client.html#google.cloud.datastore.client.Client.put)
- Using CLI tools: every action in GCP is an api call and the gcloud cli can make those calls. The library is huge, but its generally something like `gcloud <service> <action> <options>`. Some services have special cli tools, like Cloud Storage (`gsutil`) and BigQuery (`bq`).
- Provisioning and configuring networks: [Lot](https://cloud.google.com/vpc/docs/vpc) to cover here. GCP uses software defined networking, so VPC can be global, and subnets are regional. GCP offers a shared VPC where a host account holds the VPC, but other service projects can deploy resources into that VPC. Useful for enterprise architecture where app teams need their own projects to work in, but security and networking can maintain control over the network. VPCs have firewall rules to control traffic. VPCs can be peered to one another for easy access. They can be connected to hybrid environments using Cloud VPN or Interconnect. 

4.2 Integrating an application with Compute services
- Implementing service discovery in GKE, GAE, and GCE: GCP includes service discovery in the form of a metadata server. You can configure project metadata with shared environment variables, and query the metadata server endpoint (cURL).[This](https://medium.com/google-cloud/service-discovery-and-configuration-on-google-cloud-platform-spoiler-it-s-built-in-c741eef6fec2) is an older article talking about its use in for VMs. See the [Docs](https://cloud.google.com/compute/docs/storing-retrieving-metadata?hl=en). In the context of containers, service discovery refers to the types of containers as [services](https://medium.com/google-cloud/running-a-simple-kubernetes-frontend-backend-service-on-google-cloud-platform-85eb0346f600).
- Writing an application that publishes/consumes from Cloud Pub/Sub: use the gcloud sdk to publish and consume. You can also hit the https endpoints to access that functionality.
- Reading instance metadata to obtain application configuration: Instance [metadata](https://cloud.google.com/compute/docs/storing-retrieving-metadata) usually includes information about the infrastructure (IP, VM class, applicable service accouts, etc). You can add custom metadata in the form of labels (as opposued to network tags). You can use this to create startup and shutdown scripts. Application configuration would have to be added as custom metadata, which could then be queried for post provisioning, maintanance, governance, and other tasks. (I couldn't find any info on app info in metadata).
- Authenticating users by using Oauth2 Web Flow and Identity Aware Proxy:
    - Oauth: A widely used auth framework. See [more](https://cloud.google.com/community/tutorials/understanding-oauth2-and-deploy-a-basic-auth-srv-to-cloud-functions).
    ![alt text](https://storage.googleapis.com/gcp-community/tutorials/understanding-oauth2-and-deploy-a-basic-auth-srv-to-cloud-functions/ac.png 'oAuth flow')
    - IAP: is a GCP service that uses identity and context to sign in to apps and vms. See the [tutorial](https://cloud.google.com/iap/docs/app-engine-quickstart). You can allow users access by whitelisting them. You can also add conditions based on location, and add admin rights depending on the URL path. Check the [quick demo](https://www.youtube.com/watch?v=XqMY-rPk3MY).
- Using the CLI tools: Same info as previous section.
- Configuring Compute services network settings (e.g., subnet, firewall ingress/egress, public/private IPs): subnet and firewall rules have already been answered. All Compute instances have an internal IP that allows resoureces within the network to access them. Public IPs are optional- they can be removed. They allow access from outside the network. Multiple IPs can be added by adding additional virtual NICs. 

4.3 Intgrating Google Cloud APIs with applications.



### NB:
- Cloud SQL not HA cross regionally
- Bigtable not globally availible


### Glossary (terms I don't know/Stuff I need to understand better):
- Cloud Memorystore
- [SQL Union operator](https://www.techonthenet.com/sql/union.php)
- [SQL Cross Join](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#join-types)
- SQL Unnest
- Service Mgmt Api
- Wireshark
- Cloud tasks
- Cloud Composer
- DataStore Queries
- VPC flow logs
- [VPC service controls](https://cloud.google.com/vpc-service-controls/docs/)- Allows user to constrain managed services (buckets, BigTable, BigQuery) within VPC
- [Cloud Armor](https://cloud.google.com/armor/)- defense at scale against DDoS attacks
- [Cloud Identity-Aware Proxy](https://cloud.google.com/iap/docs/)- Uses identity and context to allow secure auth without VPN. Works for App Engine, Compute and GKE