# Cloud Scheduler Source

This repository implements an Event Source for (Knative Eventing)[http://github.com/knative/eventing]
defined with a CustomResourceDefinition (CRD). This Event Source represents
(Google Cloud Scheduler)[https://cloud.google.com/scheduler/]. Point is to demonstrate an Event Source that
does not live in the (Knative Eventing Sources)[http://github.com/knative/eventing-sources] that can be
independently maintained, deployed and so forth.

This particular example demonstrates how to perform basic operations such as:

* Create a Cloud Scheduler Job when a Source is created
* Delete a Job when that Source is deleted
* Update a Job when the Source spec changes

## Details

Actual implementation contacts the Cloud Scheduler API and creates a Job
as specified in the CloudSechedulerSource CRD Spec. Upon success a Knative service is created
to receive calls from the Cloud Scheduler and will then forward them to the Channel.


## Purpose

Provide an Event Source that allows subscribing to Cloud Scheduler and processing them
in Knative.

Another purpose is to serve as an example of how to build an Event Source using a
(Kubernetes Sample Controller)[https://github.com/kubernetes/sample-controller] as a starting point.

## Installation 

```sh
gcloud services enable 
```


1. Create GCP [Service Account(s)](https://console.cloud.google.com/iam-admin/serviceaccounts/project). You can either create one with both permissions or two different ones for least privilege. If you create only one, then use the permissions for the `Source`'s Service Account (which is a superset of the Receive Adapter's permission) and provide the same key in both secrets.
    - The `Source`'s Service Account.
        1. Determine the Service Account to use, or create a new one.
        1. Give that Service Account the 'Pub/Sub Editor' role on your GCP project.
        1. Download a new JSON private key for that Service Account.
        1. Create a secret for the downloaded key:
            
            ```shell
            kubectl -n knative-sources create secret generic cloudscheduler-source-key --from-file=key.json=PATH_TO_KEY_FILE.json
            ```
            
    - Note that you can change the secret's name and the secret's key, but will need to modify some things later on.

## Running

1. Setup [Knative serving](https://github.com/knative/docs/tree/master/install/README.md) or
1. Setup [Knative eventing](https://github.com/knative/docs/tree/master/eventing)

**Prerequisite**: Since the sample-controller uses `apps/v1` deployments, the Kubernetes cluster version should be greater than 1.9.

```sh
# assumes you have a working kubeconfig, not required if operating in-cluster
$ go build -o sample-controller .
$ ./sample-controller -kubeconfig=$HOME/.kube/config

# create a CustomResourceDefinition
$ kubectl create -f artifacts/examples/crd.yaml

# create a custom resource of type Foo
$ kubectl create -f artifacts/examples/example-foo.yaml

# check deployments created through the custom resource
$ kubectl get deployments
```

### Example

The CRD in [`crd-status-subresource.yaml`](./artifacts/examples/crd-status-subresource.yaml) enables the `/status` subresource
for custom resources.
This means that [`UpdateStatus`](./controller.go#L330) can be used by the controller to update only the status part of the custom resource.

To understand why only the status part of the custom resource should be updated, please refer to the [Kubernetes API conventions](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status).

In the above steps, use `crd-status-subresource.yaml` to create the CRD:

```sh
# create a CustomResourceDefinition supporting the status subresource
$ kubectl create -f artifacts/examples/crd-status-subresource.yaml
```

## Cleanup

You can clean up the created CustomResourceDefinition with:

    $ kubectl delete crd foos.samplecontroller.k8s.io

