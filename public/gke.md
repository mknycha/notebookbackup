## Kuberenetes Managed Storage Services

Why use it?
Because then Google handles availability, performance and security of your data

Self-managed containerized storage vs. GKE Managed Storage Services:
You can Volumes and PersistentVolumes abstractions for durable file storage, or as a backing store for databases, or other storage services that you deploy and manage yourself.
E.g. if you have a MySQL server, you will have to fully maintain it yourself.

In order to use it, your application needs to use Google Cloud Platform API. You need to enable the relevant API, and give the credentials to your app - this means your app needs to have a separate Cloud IAM service account. Your app will need to provide the service account key with every request to Cloud API.
Separate account is safer, because it’s easier to revoke access.

Cloud storage - object storage services
Used for:
- files
- archives
- unstructued files (blob storage)
- backups
Can’t be used for:
- Data querying
- file system (do not use it as a persistent volume)
Cloud Storages Storage Classes (4) offer you different trade-offs of price and geographic redundancy.

Databases
Databases managed by GCP.
Bigtable - Google’s NoSQL database designed for scale. Scales lineary. Super fast data retrieval.
Firestore - NoSQL database, automatically scales and replicate data for high availability. Offers guarantees for transactions.
Can be used to non-homogenic data.
Offers ACID complaints - so it offers consistent state. Can be used for instance for session management, persistent cache.
Cloud SQL - sql database service for MySQL and PostgreSQL. It manages data replication and it performs backups for you. Cloud SQL also supports automatic Failovers for high availability. Your applications running in GKE access Cloud SQL using the Cloud SQL proxy. That's a piece of software you add to your application to provide secure access to Cloud SQL database instances without the need to configure SSL or whitelist IP addresses.
Cloud spanner - relational, offers transactions guarantees. Unlike Cloud SQL, it’s globally distributed and horizontally scalable.
Bigquery - supports SQL queries, but it’s a columnar store. That means it's optimized for analyzing data sets rather than for making row by row queries and changes.

## Kuberenetes logging and monitoring

### Stackdriver
Stackdriver - suite of reconnaissance tools: for debugging, logging and monitoring applications and infrastructure. Logging is support is built-in for most of the GCP services (like AppEngine) and can be used with an agent for others (AWS EC2 or GCP Compute Engine). Includes configurable dashboard for showing metrics and stuff
Metrics - value representing performance that you can monitor (like disk usage)
Events - represent actions (e.g. scaling up or down deployments in a cluster)
Stakcdriver debugging:
	- trace - see the bottlenecks and latency
	- error reporting - captures the error information and stack (like rollbar)
	- debugger - walk through the code and indentify how the value changes during code execution (breakpoints are called watchpoints in stackdriver)
Stackdriver is cool, because it does not impact the performance much.
If you want to use stackdriver for multiple GCP projects, good practice is to create a separate project just for the stackdriver.

### Logging
Passive form of system monitoring, collects the events logs.
Tow ways to view logs: view the logs with the kubectl command, or GCP console - stackdriver. Stack driver stores the logs for 30 days.
For long term storage or complex analysis, export the logs to a google cloud storage.
The basic GKE logs such as the system component logs, standard output and the standard error messages are stored in the /var/log directory on the nodes and can be queried using the kubectl logs command or by directly accessing the /var/log directory on the nodes.
Why use stackdriver for logging?
- Because it makes the logs available for more time than just kubectl
- Broaden visibility of GKE events
- Single interface for all the clusters, nodes and services
Logs from Kubernetes system components such as kubectl and kube proxy are stored in each node's file system in its /var/log directory.
Since the logs are stored in the above directory, to avoid node saturation with logs, the are automatically streamed to stackdriver and rotated.
Only the last 5 days logs file are stored on the node itself.
Since the node or a pod can go down, logs must be stored outside (cluster level logging). In GKE handled by an integration with a stackdriver. Logging agents are preinstalled on the node and configured by default.
To filter logs, you can use directly stackdriver logging API or the stackdriver console.
Stackdriver uses FluentD (logs aggregator, also adds metadata, setup using DaemonSet and ConfigMap for configuration), as a node logging agent.
You can create custom metrics that checks the occurance of particular events.
You can also setup alert policies that causes some action to be taken when customer metrics value changes.

### Monitoring
Site reliability engineering - Google approach to DevOps. Basically everything is based on monitoring data.
Monitoring is especially important for loosly coupled services.
We monitor Clusters, Nodes and Pods.
Pod monitoring consists of system metrics (deployments, health checks), container metrics (resource usage), application specific metrics.
Stackdriver can filter the logs based on resources labels.
You can use Prometheus for collecting additional, custom metrics from the application and pass them to stackdriver.

### Probes
Probes allow us to check statuses of pods.
Liveness - is the container running?
Readiness - is the container accepting requests?
Three types of probes handlers: command (e.g. print a file from a pod), http (get request) & tcp (kubelet attempts to make a tcp connection).

