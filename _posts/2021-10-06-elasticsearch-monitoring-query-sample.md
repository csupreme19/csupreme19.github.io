---
layout: post
title: Elasticsearch 모니터링 쿼리 예제
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/titles/elastic-logo.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Query, DSL, Elastic, ELK]

---

# Elasticsearch 모니터링 쿼리 예제

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

Elasticsearch metric을 조회하여 host, k8s 등을 모니터링 할 수 있도록 쿼리를 작성해보았다.

---

## Elasticsearch Query DSL

### 1. VM Metric Monitoring

VM 서버 자체가 실패한 경우와 마지막 메트릭 수집 시점 알림

VM 리소스 정보 임계점이 넘으면 알림(Disk, CPU, Memory)

#### 1. VM health check

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=aggregations.alive,aggregations.metric_count.buckets
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-30s"
              , "lte": "now"
              , "boost": 2
            }
          }
        },
        {
          "exists": {
            "field": "host.name"
          }
        }
      ]
    }
  },
  "size": 0, 
  "aggs": {
    "alive": {
      "cardinality": {
        "field": "host.name"
      }
    },
    
    "metric_count": {
      "terms": {
        "field": "host.name"
        , "size": 40
        , "order": {
          "_key": "asc"
        }
      }
    }
  }
}
```

최근 30초간 메트릭 수집이 없었으면 죽었다고 판단

#### 응답 예시

```json
{
  "aggregations" : {
    "alive" : {
      "value" : 12
    },
    "metric_count" : {
      "buckets" : [
        {
          "key" : "vm-agent",
          "doc_count" : 32
        },
        ...
        {
          "key" : "vm-proxy",
          "doc_count" : 34
        }
      ]
    }
  }
}
```

#### 2. VM fail time check

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=hits.hits.fields
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "host.name": "{검색할 vm명}"
          }
        }
      ]
    }
  },
  "fields": [
    {
      "field": "@timestamp",
      "format": "epoch_millis"
    }
  ],
  "size": 1,
  "_source": [
    "false"
  ],
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

해당 host의 최근 수집 시간 조회

#### 응답 예시

```json
{
  "hits": {
    "hits": [
      {
        "fields": {
          "@timestamp": [
            "1634172060569"
          ]
        }
      }
    ]
  }
}
```

#### 3. VM resource check

##### 1. Memory, CPU 사용량

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=**.key,**.cpu_max,**.memory_max
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-30s",
              "lte": "now",
              "boost": 2
            }
          }
        },
        {
          "exists": {
            "field": "host.name"
          }
        },
        {
          "bool": {
            "should": [
              {
                "match": {
                  "event.dataset": "system.cpu"
                }
              },
              {
                "match": {
                  "event.dataset": "system.memory"
                }
              },
              {
                "match": {
                  "event.dataset": "system.filesystem"
                }
              }
            ]
          }
        }
      ]
    }
  },
  "size": 0,
  "_source": "@false",
  "aggs": {
    "group": {
      "terms": {
        "field": "host.name",
        "size": 40,
        "order": {
          "memory_max": "desc"
        }
      },
      "aggs": {
        "cpu_user": {
          "max": {
            "field": "system.cpu.user.pct"
          }
        },
        "cpu_system": {
          "max": {
            "field": "system.cpu.system.pct"
          }
        },
        "cpu_cores": {
          "max": {
            "field": "system.cpu.cores"
          }
        },
        "cpu_max": {
          "bucket_script": {
            "buckets_path": {
              "user": "cpu_user",
              "system": "cpu_system",
              "n": "cpu_cores"
            },
            "script": "params.n > 0 ? (params.user+params.system)/params.n : null"
          }
        },
        "memory_max": {
          "max": {
            "field": "system.memory.actual.used.pct"
          }
        },
        "bucket_filter": {
          "bucket_selector": {
            "buckets_path": {
              "cpu_max": "cpu_max",
              "memory_max": "memory_max"
            },
            "script": "params.memory_max > 0.5 || params.cpu_max > 0.5"
          }
        }
      }
    }
  }
}
```

각 host별 최근 30초간 자원 최대 사용량

##### cpu_max

$$system.cpu.user.pct+system.cpu.system.pct/system.cpu.cores$$

##### memory_max

$$system.memory.actual.used.pct$$

#### 응답 예시

```json
{
  "aggregations" : {
    "group" : {
      "buckets" : [
        {
          "key" : "vm-1",
          "memory_max" : {
            "value" : 0.5680000000000001
          },
          "cpu_max" : {
            "value" : 0.37224999999999997
          }
        },
        {
          "key" : "vm-2",
          "memory_max" : {
            "value" : 0.519
          },
          "cpu_max" : {
            "value" : 0.028624999999999998
          }
        }
      ]
    }
  }
}

```

##### 2. 디스크 사용량(파티션별)

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=**.key,**.disk_max.*.used
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-120s",
              "lte": "now",
              "boost": 2
            }
          }
        },
        {
          "exists": {
            "field": "host.name"
          }
        },
        {
          "match": {
            "event.dataset": "system.filesystem"
          }
        }
      ]
    }
  },
  "size": 0,
  "_source": "@false",
  "aggs": {
    "group": {
      "terms": {
        "field": "host.name",
        "size": 40
      },
      "aggs": {
        "disk_max": {
          "terms": {
            "field": "system.filesystem.mount_point",
            "exclude": ".*docker.*|.*/var/lib/lxcfs.*",
            "size": 10,
            "order": {
              "used.value": "desc"
            }
          },
          "aggs": {
            "used": {
              "max": {
                "field": "system.filesystem.used.pct"
              }
            },
            "bucket_filter": {
              "bucket_selector": {
                "buckets_path": {
                  "disk_max": "used"
                },
                "script": "params.disk_max > 0.8"
              }
            }
          }
        }
      }
    }
  }
}
```

최근 2분간 호스트별 파티션별 디스크 사용량이 80%를 초과하면 해당 파티션과 사용률 조회

#### 응답 예시

```json
{
  "aggregations" : {
    "group" : {
      "buckets" : [
        {
          "key" : "vm-5",
          "disk_max" : {
            "buckets" : [
              {
                "key" : "/",
                "used" : {
                  "value" : 0.828
                }
              }
            ]
          }
        },
        {
          "key" : "vm-6",
          "disk_max" : {
            "buckets" : [
              {
                "key" : "/",
                "used" : {
                  "value" : 0.841
                }
              }
            ]
          }
        }
      ]
    }
  }
}

```

### 2. Kubernetes Cluster Monitoring

Kubernetes 클러스터의 health와 Deployment, pod 장애 상태 알림

#### 1. Kubernetes deployment

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=aggregations.group.buckets.key,aggregations.group.buckets.group_docs.hits.hits._source
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-30s",
              "lte": "now",
              "boost": 2
            }
          }
        },
        {
          "script": {
            "script": "doc['kubernetes.deployment.replicas.desired'].value > doc['kubernetes.deployment.replicas.available'].value"
          }
        },
        {
          "match": {
            "event.dataset": "kubernetes.deployment"
          }
        }
      ]
    }
  },
  "size": 0,
  "_source": "@false",
  "aggs": {
    "group": {
      "terms": {
        "field": "kubernetes.deployment.name",
        "size": 100
      },
      "aggs": {
        "group_docs": {
          "top_hits": {
            "size": 1,
            "_source": [
              "kubernetes.namespace",
              "kubernetes.deployment.replicas"
            ],
            "sort": [
              {
                "@timestamp": {
                  "order": "desc"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

k8s deployment 워크로드 상태 조회

최근 30초간 desired pod > available pod인 deployment만 조회한다.

#### 응답 예시

```json
{
  "aggregations" : {
    "group" : {
      "buckets" : [
        {
          "key" : "yourservice-deploy",
          "group_docs" : {
            "hits" : {
              "hits" : [
                {
                  "_source" : {
                    "kubernetes" : {
                      "namespace" : "production",
                      "deployment" : {
                        "replicas" : {
                          "desired" : 2,
                          "unavailable" : 1,
                          "available" : 1,
                          "updated" : 2
                        }
                      }
                    }
                  }
                }
              ]
            }
          }
        }
      ]
    }
  }
}

```

#### 3. Kubernetes pod

#### 요청 예시

```json
POST metricbeat-*/_search?filter_path=aggregations.group.buckets.key,aggregations.group.buckets.*.hits.hits._source
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-60s",
              "lte": "now",
              "boost": 2
            }
          }
        },
        {
          "match": {
            "event.dataset": "kubernetes.container"
          }
        },
        {
          "exists": {
            "field": "kubernetes.container.status.reason"
          }
        },
        {
          "bool": {
            "should": [
              {
                "range": {
                  "kubernetes.container.status.restarts": {
                    "gte": "1"
                  }
                }
              },
              {
                "match": {
                  "kubernetes.container.status.reason": "ErrImagePull"
                }
              },
              {
                "match": {
                  "kubernetes.container.status.reason": "ImagePullBackOff"
                }
              }
            ]
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "kubernetes.container.status.reason": "Completed"
          }
        },
        {
          "match": {
            "kubernetes.container.status.reason": "ContainerCreating"
          }
        },
        {
          "match": {
            "kubernetes.container.status.reason": "PodInitializing"
          }
        }
      ]
    }
  },
  "size": 0,
  "_source": "@false",
  "aggs": {
    "group": {
      "terms": {
        "field": "kubernetes.pod.name",
        "size": 100
      },
      "aggs": {
        "reason": {
          "top_hits": {
            "size": 1,
            "_source": [
              "kubernetes.namespace",
              "kubernetes.container.status.reason",
              "kubernetes.container.status.restarts"
            ],
            "sort": [
              {
                "@timestamp": {
                  "order": "desc"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

최근 1분간 재시작 장애가 발생하는 pod 조회

아래와 같은 상태는 정상 상태라고 판단하였다.

- Completed 
  - initContainer, Job, 일회성 pod들은 해당 상태로 라이프사이클이 종료된다.
- ContainerCreating
  - available이 아니지만 컨테이너 생성중인 정상 상태
- PodInitializing
  - 위와 마찬가지

>[파드 장애 판별](/2021/10/14/kubernetes-pod-fail-test)

#### 응답 예시

```json
{
  "aggregations" : {
    "group" : {
      "buckets" : [
        {
          "key" : "dummy-pod",
          "reason" : {
            "hits" : {
              "hits" : [
                {
                  "_source" : {
                    "kubernetes" : {
                      "container" : {
                        "status" : {
                          "reason" : "CrashLoopBackOff",
                          "restarts" : 10
                        }
                      },
                      "namespace" : "default"
                    }
                  }
                }
              ]
            }
          }
        },
        {
          "key" : "dummy-pod2",
          "reason" : {
            "hits" : {
              "hits" : [
                {
                  "_source" : {
                    "kubernetes" : {
                      "container" : {
                        "status" : {
                          "reason" : "ImagePullBackOff",
                          "restarts" : 9
                        }
                      },
                      "namespace" : "default"
                    }
                  }
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```


---

## Reference

1. [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
2. [Bucket script aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline-bucket-script-aggregation.html)
3. [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)