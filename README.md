# kubernetes-k8s-lab14
Your Kubernetes Cluster

For this scenario, Katacoda has just started a fresh Kubernetes cluster for you. Verify it's ready for your use.

kubectl version --short && \
kubectl get componentstatus && \
kubectl get nodes && \
kubectl cluster-info

The Helm package manager used for installing applications on Kubernetes is also available.

helm version --short

Kubernetes Dashboard

You can administer your cluster with the kubectl CLI tool or use the visual Kubernetes Dashboard. Use this script to access the protected Dashboard.

token.sh

######

Deploy ElasticSearch

Elasticsearch is a RESTful search engine based on the Lucene library. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. Elasticsearch is open source and developed in Java.

Create a namespace for the installation target.

kubectl create namespace logs

Add the chart repository for the Helm chart to be installed.

helm repo add elastic https://helm.elastic.co

Deploy the public Helm chart for ElasticSearch. The chart's default settings are appropriately opinionated for production deployment. Here, some of the default settings are downsized to fit in this Katacoda cluster.

helm install elasticsearch elastic/elasticsearch \
--version=7.9.0 \
--namespace=logs \
-f elastic-values.yaml

ElasticsSearch is starting and will be available in a few moments. In the meantime, continue to the next installation step.

######
Deploy Fluent Bit

Fluent Bit is an open source specialized data collector. It provides built-in metrics and general purpose output interfaces for centralized collectors such as Fluentd. Create the configuration for Fluent Bit.

Install Fluent Bit and pass the ElasticSearch service endpoint as a chart parameter. This chart will install a DaemonSet that will start a Fluent Bit pod on each node. With this, each Fluent Bit service will collect the logs from each node and stream it to ElasticSearch.

Add the chart repository for the Helm chart to be installed.

helm repo add fluent https://fluent.github.io/helm-charts

Install the chart.

helm install fluent-bit fluent/fluent-bit \
  --version 0.6.3 \
  --namespace=logs

Fluent Bit is starting and will become available in a few moments. In the meantime, continue to the next installation step.

######
Deploy Kibana

Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack. Do anything from tracking query load to understanding the way requests flow through your apps.

Deploy Kibana. The service will be on a NodePort at 31000.

helm install kibana elastic/kibana \
  --version=7.9.0 \
  --namespace=logs \
  --set service.type=NodePort \
  --set service.nodePort=31000

    Security caution. This NodePort exposes the logging to the outside world intentionally for demonstration purposes. However, for production Kubernetes clusters never expose the Kibana dashboard service to the world without any authentication.

Kibana is starting and will become available in a few minutes.

######

Verify Running Stack

All three installations of ElasticSearch, Fluent Bit, and Kibana are either still initializing or fully available.

To inspect the status of these deployments run this watch.

watch kubectl get deployments,pods,services --namespace=logs

Once complete, the Pods will move to the Running state. The full stack is not ready until all the Deployment statuses move to the Available (1) state.

While observing the progress, be patient, as it takes time for the stack to initialize, even with this small configuration.

When all Deployments report Available and the Pods report Running use this clear to break out of the watch or press
+

.

You know have a full EFK stack running. Granted this stack smaller and not configure to he highly available or with access protection, but it comprises a functional solution to get started.
######
Generate Log Events

Run this container to start generating random log events.

kubectl run random-logger --image=chentex/random-logger

    Thank you to Vicente Zepeda for providing this beautifully simple container.

The log events will look something like this.

...
INFO takes the value and converts it to string.
DEBUG first loop completed.
ERROR something happened in this execution.
WARN variable not in use.
...

Inspect the actual log events now being generated with this log command.

kubectl logs pod/random-logger

Don't be alarmed by the messages, these are just samples.


######

View Log Events

Katacoda has exposed the NodePort 31000 to access Kibana from your browser.

Access Kibana. There is also a tab above the command-line area labeled Kibana that takes you to the same Kibana portal.
Security

    Tip: There are no credentials to access this EFK stack through Kibana. For real deployments, you would never expose this type of information without at least an authentication wall. Logs typically reveal lots of dirty laundry and attack vector opportunities.

Kibana Portal

New information and logs are currently streaming into Elasticsearch from various components. You can use the Portal to create filters to find only the logs emanating from the random-logger.

To see the logs collected from the random-logger follow these steps in the Kibana portal.

    When Kibana appears for the first time there will be a brief animation while it initializes.
    On the Welcome page click Explore on my own.
    From the left-hand drop-down menu (≡) select the Discover item.
    Click on the button Create index pattern on the top.
    In the form field Index pattern enter logstash-*
    It should read "Success!" and Click the > Next step button on the right.
    In the next form, from the dropdown labeled Time Filter field name, select @timestamp.
    From the bottom-right of the form select Create index pattern.
    In a moment a list of fields will appear.
    Again, from the left-hand drop-down menu (≡) select the Discover item.
    On the right is a listing of all the log events. On the left, is a list of available fields to choose for filtering.
    Filter the log list by first choosing the _kubernetes.podname field. When you hover over or click on the word _kubernetes.podname, click the Add button to the right of the label.
    The filter selection is added to the Selected fields list. Click on the filter and select the magnifying glass (🔍) with the plus sign (+) next to random-logger.
    Now only then events from the random-logger appear.
    From the available field list, select and add the log field.

The log list now is filtered to show log events from the random-logger service. You can expand each event to reveal further details.

From here you can start to appreciate the amount of information this stack provides. More information is in the Kibana documentation.
######






