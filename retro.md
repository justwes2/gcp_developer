#### Retrospective

_I achieved a provisional pass on the GCP Developer Professional Exam. This is my observations out of the exam. This isn't a tied to specific questions, if something was just one question, it wouldn't really stick out. This is about trends and themes across the exam._

- Sathish Vj's [notes](https://medium.com/@sathishvj/notes-from-my-beta-google-cloud-professional-cloud-developer-exam-e5826f6e5de1) are quite good, I recommend reviewing all the docs he links to.
- Understand LB deployments- GCP has a lot of entities involved in an LB deployment (instance templates, managed instance groups, backends, etc). Be familiar with how to provision the full stack, as it were.
- Remember the IP addresses and firewall rules needed for health checks. I didn't have to recognize the IPs from a range of conflicting values, but know what those IPs are, and why they matter. 
- `kubectl`: know the subcommands. Understand how logs work. 
- IAM Permissions: Understand how service accounts work _across projects_. I had a few questions about permissions for resources in project and a b interacting. 
- Datastore: Be familiar with best practices- how to use transactions, when to use ancestor queries. 
- Bigquery: 
    - batching- I wasn't explicitly asked about batching, but one of the solutions was based on if batching made sense, so don't just look for the word. 
    - DML: know what combinations can run concurrently.  
- GDPR: Know, at a high level, what design is needed to implement compliant apps. 
- GitOps: Understand how to implement a pipeline from source control to a build tool to a container registry. Most of the time, this revolves around GCP products, but occasionally Docker Hub and Jenkins were mentioned, so know what they are analogous to. 
- `gcloud` vs `gsutil`: `gsutil` is the storage utility. If its not interacting with a bucket, you won't need this. Also pretty sure they made up some commands line names. 
- Troubleshooting: What should you do when a vm is unresponsive to ssh? Its covered in the study guide- check the serial port output, and attach the disk to another vm to diagnose further.
- IAP came up a few times- I didn't need to know all the support mechanism, but all of them product a JWT. 
- Dataproc: I got a few questions, so know what it is, what the use cases are, and how to manage/import/export data.
- Case study: I got about 10 questions on the case study. Know the data bases- what to use when (spanner vs sql vs data store vs big query)
- This exam was more focused on remembering specific details than the PCA- which often was 'Here are four right choices, pick the _most_ correct one. Definitely be able to rattle off use cases and limitations for all the services (I know, good luck). 

I hope this helps. If anything in the guide is inaccurate, please do open an issue or a PR!