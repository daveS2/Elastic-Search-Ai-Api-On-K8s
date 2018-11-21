# ElasticSearchAiApiOnK8s
Experimental work looking at ES and ONS AI API on kubernetes in Google Cloud Platform

# Prerequesites

* [Helm install](https://docs.helm.sh/using_helm/)
* [Kubectl installed](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

# Steps to set-up Helm

1. Install helm on cluster `helm init`
2. Check with `kubectl get deploy,svc tiller-deploy -n kube-system` should see the deployment
3. Create a service account with ServiceAccount.yml: `kubectl apply -f helm-service-account.yml`
4. Run `KUBE_EDITOR="nano" kubectl edit deploy --namespace kube-system tiller-deploy`
5. And add the line `serviceAccount: helm` into  spec/template/spec
6. Check with `kubectl describe deploy --namespace kube-system tiller-deploy`

# Helm install of ES

1. Run: `helm install --name <name> stable/elasticsearch`
or run, customised for ONS: `helm install --name census-ai-elasticsearch stable/elasticsearch  --set data.persistence.size=60Gi,appVersion=5.6.12,image.repository=dave2/elasticsearch,image.tag=5.6.12gcpplugin`
2. Expose the service `kubectl expose svc <name> --type=LoadBalancer --name=elastic-search`
or run, customised for ONS: `kubectl expose svc census-ai-elasticsearch-client --type=LoadBalancer --name=elastic-search`

# Install ONS AI API

1. Update `ai-api-deployment.yml` to the correct IP for ES
2. Run `kubectl apply -f ai-api-deployment.yml.yml`

# Optional create custom ES containter:
1. Clone repo: https://github.com/elastic/elasticsearch-docker
2. `cd <that repo>`
3. Build the image: `docker build -t yourrepo/imagename:tag .`
4. Get the image ID `docker images`
5. Run the image: `docker run <images id>`
6. Run `docker ps` and grab the container id
7. Enter the container: `docker exec -it <containter id> /bin/bash`
8. Enter the image customise the build for example: `sudo bin/elasticsearch-plugin install repository-gcs`
9. Rebuild `docker build -t yourrepo/imagename:tag .`
10. Push to your repo: `docker push`
11. Add image repo to helm install `image.repository=yourrepo/imagename,image.tag=tag`

# Optional, install cerebro (ES UI)

1. `helm install stable/cerebro`
2. `kubectl expose deployment <deployment name> --type=LoadBalancer --name=elastic-search-ui`

# Optional create Google Cloud Storage Repository

## Prequsites

* GCS plugin installed within ES container(see above to customise ES containers)
* Correct ES container deployed

1. Run:
``` 
curl -X PUT "<ip:port>/_snapshot/<repo name>" -H 'Content-Type: application/json' -d'
{
  "type": "gcs",
  "settings": {
    "bucket": "<bucketname>"
  }
}
```
2. ES will then backup from that bucket

