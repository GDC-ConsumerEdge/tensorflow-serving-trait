# Overview

Models should be built in a preferred method as long as they are TensorFlow models. Exporting the model to `saved_model.pb` or `model-name.tflite` (NOTE: `tflite` are experimental) and placing them in a GCS bucket with a sequence of folders representing versions and containing the model. Models can be arbitrary (/1 vs /2 vs /3 can all be different models) however best practices would be to use a single bucket per model and to use configuration to host different models on the same inference server.

```bash
.
├── 1
│   └── saved_model.pb
├── 2
│   └── saved_model.pb
├── 3
│   └── saved_model.pb
├── 4
│   └── saved_model.pb
└── 5
    └── saved_model.pb
```


## Update / Setup Models

1. Check the current models used by tf-serving
  - `gcloud storage ls --all-versions --long gs://[GCS where models are exported]`
2. Download new version locally (optional, can use gcloud storage to copy between buckets instead)
  - `gcloud storage cp gs://[GCS Bucket w/ exported models]/[folder-containing-model]/.../saved_model.pb .`
3. Create a new version for tf-serving
  - `gcloud storage cp saved_model.pb gs://[tf-serfving-model-bucket]/[version-number]/saved_model.pb` # where "#" is the version number in sequence
4. Update configuration in tf-serving CTR to expose the version
  - modify the `/config/tf-serving/default-configmap-model-config.yaml` <!-- TODO: Move the configuration to the container of the CTR, not in the CTR -->
  - commit and push
  - bounce the pods so the new config picks up version
5. Use tf-serving service using:
  - export HOST="tf-serving-service.default.svc.cluster.local" or IP
  - Label is likely "latest". Can substitute "/labels/<label>" for "/versions/<version-number>" if version is known
  - curl http://${HOST}:8501/v1/models/<model-name>/labels/<label>/metadata
