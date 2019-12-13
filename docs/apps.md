Sometimes it's hard to figure out what exactly the least privileges for a certain app are and we resort to giving `cluster-admin` permissions for apps running in our Kubernetes cluster. This is great news for attackers, but not so great news for the fine folks who are on-call for the cluster. The good practice is to use tools such as [audit2rbac](https://github.com/liggitt/audit2rbac) to right-size the permissions.

In the following, we have a look at sensible RBAC settings for common apps.

## Prometheus

## Elasticsearch

## MySQL

## Kafka

## Istio

## Kubeflow
