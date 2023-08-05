# Overview

|Port | Protocol | Transport | Description |
|-----|----------|-----------|-------------|
| 8501 | HTTP | TCP | Restulf API |
| 8500 | gRPC | TCP | gRPC API |

## Images
Images need to be smaller than 1.5 Mb in base64 format

## APIs

Obtain the IP of the service
```
export IP_ADDRESS="IP of the LoadBalancer Service" # kubectl get svc -n tf-serving
```

### Is OK (Running)
```bash
curl http://${IP_ADDRESS}:8501/v1/models/hat-no-hat-v1/versions/1
```

## Bash

1. Create Base64 version of a picture
```
base64 -w 0 < first-file.jpg > first-inference.json

base64 -w 0 < second-hat-pic.jpg > second-inference.json
```

1. Create a file called "first-inference.json"

```json
{
    "signature_name": "serving_default",
    "instances": [{
        "image_bytes": {
            "b64":
            "valueOfThImageHereInBase64"
        },
        "key": "hat"
    }]
}
```

```bash
curl \
    -X POST \
    -H "Content-Type: application/json" \
    http://192.168.8.23:8501/v1/models/hat-no-hat-v1/versions/1:predict \
    -d "@second-inference.json"
```


### Find the values of the model

1. pull the `saved_model.pb` down locally

1. Run the `saved_model_cli` (you have to install tfserving binaries first)

```bash
saved_model_cli show --dir . --tag_set serve --signature_def serving_default

The given SavedModel SignatureDef contains the following input(s):
  inputs['image_bytes'] tensor_info:
      dtype: DT_STRING
      shape: (-1)
      name: encoded_image_string_tensor:0
  inputs['key'] tensor_info:
      dtype: DT_STRING
      shape: (-1)
      name: key:0
The given SavedModel SignatureDef contains the following output(s):
  outputs['detection_boxes'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 40, 4)
      name: detection_boxes:0
  outputs['detection_classes'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 40)
      name: detection_classes:0
  outputs['detection_classes_as_text'] tensor_info:
      dtype: DT_STRING
      shape: (-1, 40)
      name: detection_classes_as_text:0
  outputs['detection_multiclass_scores'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 40, 2)
      name: detection_multiclass_scores:0
  outputs['detection_scores'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 40)
      name: detection_scores:0
  outputs['image_info'] tensor_info:
      dtype: DT_INT32
      shape: (-1, 6)
      name: Tile_1:0
  outputs['key'] tensor_info:
      dtype: DT_STRING
      shape: (-1)
      name: Identity:0
  outputs['num_detections'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1)
      name: num_detections:0
Method name is: tensorflow/serving/predict

```