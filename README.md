# Dockerfiles

This repo is a fork from https://github.com/elastic/dockerfiles that modifies the Elasticsearch 6.8 Dockerfile to provide multiarchitecture support for the amd64 and arch64 platforms. [See this branch comparison for all the changes](https://github.com/ArcadiaPower/elasticsearch-dockerfiles/compare/6.8...6.8-arm-compatible).

The [elasticsearch-docker-arm64 repo](https://github.com/hsxsix/elasticsearch-docker-arm64) was used as a guide for making the neccesary changes.

[Note that since the Xpack Machine Learning feature does not support ARM](https://issues.apache.org/jira/browse/FLINK-14126), it has been disabled in the included `elasticsearch.yml`.

## Multi-archetiecture builds

```
docker buildx create --name amd-arm-builder
docker buildx use amd-arm-builder
docker buildx build --platform linux/amd64,linux/arm64 --tag YOUR_TAG --push .
```
