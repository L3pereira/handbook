---
description: >-
  Storage nodes accept user uploads and do long term archiving of user data
  objects, as well as sharing these objects with bandwidth nodes which do cached
  low latency last mile on-demand delivery of dat
---

# Storage Node

## Introduction

Storage nodes

## API

### files/{id}

#### GET

Returns a media file.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/files/{id}" method="get" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

#### HEAD

Returns media file headers.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="undefined" method="undefined" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### files

Upload data.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/files" method="post" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### authToken

Get auth token.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/authToken" method="post" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### state/data-objects

Returns all local data objects.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/state/data-objects" method="get" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### state/bags/{bagId}/data-objects

Returns local data objects for a bag.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/state/bags/{bagId}/data-objects" method="get" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### version

Returns server version.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/version" method="get" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

### state/data

Returns local uploading directory stats.

{% swagger src="https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml" path="/state/data" method="get" %}
[https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml](https://raw.githubusercontent.com/Joystream/joystream/giza_staging/storage-node-v2/src/api-spec/openapi.yaml)
{% endswagger %}

## Scenarios

WIP.
