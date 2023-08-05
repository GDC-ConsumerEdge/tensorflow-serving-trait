# Overview

This repostiory is used to add tensorflow-serving trait to an ACM-enabled Anthos cluster.

This repository references a Google Cloud Storage (GCS) bucket to pull Tensorflow models from. The bucket and models need to be generated before using this trait in a TF Serving to a cluster.

## Pre-Reqs
* GCS Bucket name containing TensorFlow "SavedModel" and tf-lite models
* GSA (`tf-model-gsa`) with read-only access to bucket
* GSA JSON creds of `tf-model-gsa`

## Default Values
This trait has some default values and conventions that may not fit your conditions.
* Default GSA is `tf-model-gsa`
* Default GCS bucket name prefix `tf-models`

> NOTE: GCS buckets have universal naming therefore this repo assumes using a prefix (ie: `tf-models-xxxx`)

# Trait Installation

## How to apply Trait

Must apply the following code snippet. This code will create a new `RepoSync` object pointing to the Cluster Trait Repo, and create an `ExternalSecret` that allows the `RootSync` to reference a private repository.

> :note: This can be applied via ACM updates IF the Primary Root Repo is `unstructured`, otherwise a manual `kubectl apply -f <file.yaml>` is required to install this CTR.

```yaml
apiVersion: configsync.gke.io/v1beta1
kind: RootSync
metadata:
  name: tensorflow-serving-trait-sync
  namespace: config-management-system
spec:
  sourceFormat: "unstructured"
  git:
    repo: "https://gitlab.com/gcp-solutions-public/retail-edge/available-cluster-traits/tensorflow-serving-anthos.git"
    branch: "main"
    dir: "/config"
    auth: "token"
    secretRef:
      name: tensorflow-serving-git-creds               # matches the ExternalSecret spec.target.name below

---

apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: tensorflow-serving-git-creds-es
  namespace: config-management-system
spec:
  refreshInterval: 1h
  secretStoreRef:
    kind: ClusterSecretStore
    name: gcp-secret-store
  target:                                        # K8s secret definition
    name: tensorflow-serving-git-creds           ############# Matches the secretRef above
    creationPolicy: Owner
  data:
  - secretKey: username                          # K8s secret key name inside secret
    remoteRef:
      key: tensorflow-serving-access-token-creds #  GCP Secret Name
      property: username                         # field inside GCP Secret
  - secretKey: token                             # K8s secret key name inside secret
    remoteRef:
      key: tensorflow-serving-access-token-creds #  GCP Secret Name
      property: token                            # field inside GCP Secret

```

### Create GCP Secret for git-creds

Create the GCP Secret Manager secret used by `ExternalSecret` to proxy for K8s `Secret`

```
export PROJECT_ID=<your google project id>
export SCM_TOKEN_TOKEN=<your gitlab personal-access token value>
export SCM TOKEN_USER=<your gitlab personal-access token user>
export REPO_STRUCTURE="hierarchy" # or "unstructred"

gcloud secrets create tensorflow-serving-access-token-creds --replication-policy="automatic" --project="${PROJECT_ID}"
echo -n "{\"token\"{{':'}} \"${SCM_TOKEN_TOKEN}\", \"username\"{{':'}} \"${SCM_TOKEN_USER}\"}" | gcloud secrets versions add tensorflow-serving-access-token-creds --project="${PROJECT_ID}" --data-file=-

```

### Local Validation

Assuming `nomos` is installed (via `gcloud components install nomos`)

> IF using "hierarchy" ACM repo (default)
```
nomos vet --no-api-server-check --path config/
```

> IF using "unstructured" ACM repos
```
nomos vet --no-api-server-check --source-format "unstructred" --path config/
```

## Model Configuration & Convention

This CTR assumes the solution deploying to has a certain GCS bucket structure and fits a convention.

### Model Folder Structure

All model vesions should be placed inside of a folder structure using a folder name as the version.

```
gs://[bucket_name]/version/saved_mode.pb
```

New versions add on using the convention


### Docker method

Using this link to find the version of nomos-docker:  https://cloud.google.com/anthos-config-management/docs/how-to/updating-private-registry#expandable-1

```
docker pull gcr.io/config-management-release/nomos:latest
docker run -it -v $(pwd):/code/ gcr.io/config-management-release/nomos:latest nomos vet --no-api-server-check --path /code/config/
```

# Tensorflow Setup

## Create GCP Secret with access to GCS Bucket

The following creates a GCPSM Secret containing a `JSON` credentials key of a GSA with access to the image bucket. You will need the name of the GSA that has access.

```
export PROJECT_ID=<your google project id>
export GSA_NAME="tf-model-gsa"
export GCS_BUCKET="consumer-edge-public-aiml-samples" # replace with your bucket

# Create GSA
gcloud iam service-accounts create ${GSA_NAME} --description="GSA used to read models from GCS buckets and server to TF Serving" --display-name="TF Serving Model Agent"
# Create new GSA key
gcloud iam service-accounts keys create ./gcr-gsa-key.json --iam-account=${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
# Create GCP Secret Manager Secret
gcloud secrets create tf-lite-gcr-creds --replication-policy="automatic" --project="${PROJECT_ID}" --data-file="./gcr-gsa-key.json"
# Grant GSA access to bucket
gsutil iam ch serviceAccount:${GSA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com:objectViewer,legacyBucketReader gs://${GCS_BUCKET}
```


# ACM Overview (optional)

See [our documentation](https://cloud.google.com/anthos-config-management/docs/repo) for how to use each subdirectory.
