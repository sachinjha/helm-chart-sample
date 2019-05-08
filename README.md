# ibm-nginx-dev

[NGINX](https://www.nginx.com/) is a free and open-source web server which can also be used as a reverse proxy, load balancer and HTTP cache.

## Introduction

This chart creates an [NGINX](https://www.nginx.com/) deployment to host simple static content.

## Chart Details

This chart will do the following:

* Create a fixed size set of NGINX servers using a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Create a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) to export NGINX to the cluster.
* Optionally, use a [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-a-configmap) to inject a nginx.conf file.
* Optionally, an image that extends the official nginx images that already contains configuration and/or content to host.

## Prerequisites

* Kubernetes 1.9 or later
* Tiller 2.7.2 or later
* Existing PersistentVolumeClaims or PersistentVolumes if mounting static content or configuration files.

## Resources Required

The chart deploys pods consuming minimum resources as specified in the resources configuration parameter (default: Memory: 256Mi, CPU: 100m)

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
helm install --name my-release --repo local-charts my-nginx-chart
```

The command deploys `my-nginx-chart` on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

### Verifying the Chart

See NOTES.txt associated with this chart for verification instructions

### Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
helm delete --purge my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release. If Persistent Volumes were used, they may or may not be deleted based upon their reclaim policy.

## Configuration

The following tables lists the configurable parameters of the ibm-nginx-dev chart and their default values.

| Parameter                  | Description                                     | Default                                                    |
| -----------------------    | ---------------------------------------------   | ---------------------------------------------------------- |
| `image.repository`         | Image repository                                | `nginx`                                                    |
| `image.pullPolicy`         | Image pull policy                               | `Always` if `imageTag` is `latest`, else `IfNotPresent`    |
| `image.tag`                | Image tag                                       | `1.13.9-alpine`                                            |
| `replicaCount`             | Number of deployment replicas                   | `1`                                                        |
| `configMapName`            | Name of the ConfigMap with the nginx.conf file  | ``                                                         |
| `service.port`             | TCP port NGINX will bind to.                    | `80`                                                       |
| `htmlPVC.enabled`          | Use a volume that contins static content files  | `false`                                                    |
| `htmlPVC.accessMode`       | Access mode for the volume                      | `ReadOnlyMany`                                             |
| `htmlPVC.existingClaimName` | Name of an existing volume claim to use        | ``                                                         |
| `htmlPVC.selector.label`   | The label to use when selecting a volume        | ``                                                         |
| `htmlPVC.selector.value`   | The label value to match when selecting a volume | ``                                                        |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
helm install --name my-release \
  --set replicaCount=2 \
    --repo local-charts my-nginx-chart
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
helm install --name my-release --repo local-charts my-nginx-chart -f values.yaml 
```

> **Tip**: You can use the default values.yaml

## Storage

Storage can be used to inject  content such as html files into the chart. These can
be used by creating your content in a persistent volume and then either creating an volume claim and referencing it by name with an
`existingClaimName` parameter or by using the `selector.label` and `selector.value` parameters to have the chart create a volume claim
to select the volume.

## Limitations

None.

## Documentation

### Server content by creating a Docker image with content

To inject content and configuration, you do not have to use the volume configurations. Instead, you can just create your own image by extending the
`nginx` image. It could easily be down with a `Dockerfile` similar to this:

```Dockerfile
FROM nginx:1.13.9-alpine

# add all the files in the html directory to the image
ADD html /usr/share/nginx/html/

# add all the configuration files in the conf.d directory to the image
ADD conf.d /etc/nginx/conf.d/
```

### Inject a nginx.conf file with a ConfigMap

To use a Kubernetes ConfigMap to serve as the NGINX configuration, you need to create a file with content similar to:

`my-nginx-configmap.yaml`

```shell
kind: ConfigMap
apiVersion: v1
metadata:
  name: my-nginx-configmap
data:
  nginx.conf: |-
    events {
      worker_connections  1024;
    }

    http {
      server {
          listen 80;

          location = / {
            return 200 "hello world";
          }
      }
    }
```

The create the ConfigMap:

```shell
kubectl create -f my-nginx-configmap.yaml
```

The set the configMapName of your Helm installation:

```shell
helm install --name my-release --set configMapName=my-nginx-configmap stable/ibm-nginx-dev
```