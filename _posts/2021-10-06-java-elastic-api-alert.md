---
layout: post
title: Elastic API 알람 모듈 개발기
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/jeaa-1.png
author: csupreme19
categories: Development
tags: [Java, Spring, Spring boot, Scheduler, Elasticsearch, Kibana, Slack, Webhook]

---

# Elastic API 알람 모듈 개발기

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

Spring boot 기반 엘라스틱서치 알람 모듈 개발 경험을 정리해봤어요.

---

## 배경

Elastic Kibana의 Alert 기능을 사용하려면 유료 구독형 라이선스(Gold License 이상)가 필요한데

현재 Basic License를 사용중이기 때문에 알림 기능을 사용할 수 없었어요.

메트릭 정보를 수집하고 판단하여 slack, mail, 문자 등으로 알람을 줄 수 있는 모듈을 개발하기로 제안하였어요.

---

## 모니터링 요소

### 1. Monitoring(입력)

폴링방식으로 시간 주기 스케줄러를 돌려 API를 호출하여 메트릭 정보를 확인한다.

1. #### VM Metrics

2. #### Kubernetes Metrics

3. #### Service Metrics

4. #### APM

### 2. Alert(출력)

---
## Monitoring(입력)

모니터링 요소는 크게 3가지로 나뉜다고 생각했어요.

##### 1. VM Metrics
![jeaa-1.png]({{ "/assets/img/contents/jeaa-1.png"}})

서버 자원, 네트워크 사용량 등의 메트릭 정보 모니터링 

##### 2. Kubernetes Metrics
![jeaa-2.png]({{ "/assets/img/contents/jeaa-2.png"}})

쿠버네티스 클러스터 상태, 파드 재시작, 레플리카 수 등 API 서버 및 메트릭 정보 모니터링

##### 3. Opensource Metrics
![jeaa-3.png]({{ "/assets/img/contents/jeaa-3.png"}})

오픈소스 health, 지연 등 메트릭 정보 모니터링

##### 4. APM
![jeaa-4.png]({{ "/assets/img/contents/jeaa-4.png"}})

Application Transaction, Erros, Latency 정보 모니터링

---
## Alert(출력)

##### 1. SMS

##### ![jeaa-5.jpeg]({{ "/assets/img/contents/jeaa-5.jpeg"}})

사내 문자 서버 및 에이전트 사용중이에요.

##### 2. Slack
![jeaa-6.png]({{ "/assets/img/contents/jeaa-6.png"}})

Slack Webhook API 요청을 통해 구현 예정이에요.

---
## 개발 스택

- OpenJDK 8(Java 8)
- Spring Boot 2.4.2
- Gradle
- JUnit
- Elasticsearch REST Client
- Kubernetes

---

## 호출 흐름

<div class="mermaid">
  flowchart LR
  	A[Node]
  	B[Node]
  	C[Node]
  	D[Elasticsearch]
  	E[Elastic API Module]
  	F[Admin]
  	A & B & C --metrics--> D
  	E --query--> D
  	E --scheduler--> E
  	D -.response.-> E
  	E --alert--> F
</div>

---

## 환경구성

### 1. Elasticsearch Java REST Client

[Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)

Elasticsearch에서는 공식적으로 REST 호출을 할 수 있는 자바 클라이언트 라이브러리를 제공해요.

- Java Low Level REST Client
- Java High Level REST Client

두 명세를 제공하며 엘라스틱서치 query dsl을 그대로 사용하기 위하여 Low level REST Client를 사용할 예정이에요.

#### `build.gradle` dependency 추가

```groovy
dependencies {
	implementation 'org.elasticsearch.client:elasticsearch-rest-client:7.12.1'
  // 참고) High REST Client
  // implementation 'org.elasticsearch.client:elasticsearch-rest-high-level-client:7.15.2'
}
```

> [Maven Central](https://search.maven.org/search?q=g:org.elasticsearch.client)

<br>

#### `ElasticsearchRestClientUtil` 작성

```java
@Slf4j
@Component
public class ElasticsearchRestClientUtil {
	
	private static RestClient restClient;
	
	@Autowired
	EnvironmentConfig env;
	
	private ElasticsearchRestClientUtil() {
	}
	
	@PostConstruct
	public void init() {
		final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
		credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials("elastic", "changeme"));
		
		RestClientBuilder builder = RestClient.builder(new HttpHost(env.getElasticHost(), env.getElasticPort(), "http"));
		builder.setFailureListener(new RestClient.FailureListener() {
			@Override
			public void onFailure(Node node) {
				log.error("Elasticsearch node fail: {}, {}, {}, {}", node.getHost(), node.getName(), node.getRoles(), node.getVersion());
			}
		});
		
		builder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
			@Override
			public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
				return requestConfigBuilder.setSocketTimeout(10000); 
			}
		});
		
		builder.setHttpClientConfigCallback(new HttpClientConfigCallback() {
	        @Override
	        public HttpAsyncClientBuilder customizeHttpClient(
	                HttpAsyncClientBuilder httpClientBuilder) {
	            return httpClientBuilder
	                .setDefaultCredentialsProvider(credentialsProvider);
	        }
	    });
		
		restClient = builder.build();
	}
	
	public static Response request(String method, String url, Map<String, String> param, String jsonBody) {
		if(StringUtils.isEmpty(url)) {
			url = "/";
		}
		Request request = new Request(method, url);
		if(!ObjectUtils.isEmpty(param)) {
			request.addParameters(param);
		}
		if(!ObjectUtils.isEmpty(jsonBody)) {
			request.setJsonEntity(jsonBody);
		}
		
		Response response = null;
		
		try {
			response  = restClient.performRequest(request);
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		return response;
	}
}
```

1. 엘라스틱서치 라이브러리에서 제공하는 `RestClientBuilder`로 `RestClient` 생성 & 초기화
2. 이후 `ElasticsearchRestClientUtil`에서 초기화된 Singleton 객체 `RestClient` 사용
3. 요청시 파라미터와 json포맷 body를 받도록 설계
4. 주입된 `EnvironmentConfig` Bean 사용을 위하여 `@PostConstruct`에서 초기화

<br>

#### Query 상수 생성

```java
public class ElasticConstants {
	public static final String URL_METRICBEAT_SEARCH = "/metricbeat-*/_search";
	public static final String URL_FILEBEAT_SEARCH = "/filebeat-*/_search";
	
	public static final String QUERY_CHECKVM_VMLIST = "{\n" + 
			"  \"query\": {\n" + 
			"    \"bool\": {\n" + 
			"      \"must\": [\n" + 
			"        {\n" + 
			"          \"range\": {\n" + 
			"            \"@timestamp\": {\n" + 
			"              \"gte\": \"now-30s\"\n" + 
			"              , \"lte\": \"now\"\n" + 
			"              , \"boost\": 2\n" + 
			"            }\n" + 
			"          }\n" + 
			"        },\n" + 
			"        {\n" + 
			"          \"exists\": {\n" + 
			"            \"field\": \"host.name\"\n" + 
			"          }\n" + 
			"        }\n" + 
			"      ]\n" + 
			"    }\n" + 
			"  },\n" + 
			"  \"size\": 0," + 
			"  \"aggs\": {\n" + 
			"    \"alive\": {\n" + 
			"      \"cardinality\": {\n" + 
			"        \"field\": \"host.name\"\n" + 
			"      }\n" + 
			"    },\n" + 
			"    \n" + 
			"    \"metric_count\": {\n" + 
			"      \"terms\": {\n" + 
			"        \"field\": \"host.name\"\n" + 
			"        , \"size\": 40\n" + 
			"        , \"order\": {\n" + 
			"          \"_key\": \"asc\"\n" + 
			"        }\n" + 
			"      }\n" + 
			"    }\n" + 
			"  }\n" + 
			"}";
	
	...
}
```



#### 호출 예제

```java
import org.elasticsearch.client.Response;

...

		Response elasticResponse = ElasticsearchRestClientUtil.request("POST"
				, ElasticConstants.URL_METRICBEAT_SEARCH + "?filter_path=hits.hits.fields"
				, null
				, String.format(ElasticConstants.QUERY_CHECKVM_LASTDOC, vm));

		if(ObjectUtils.isEmpty(elasticResponse)) {
			return null;
		}
		
		JSONObject json = null;
		String item = "";
		try {
			json = new JSONObject(EntityUtils.toString(elasticResponse.getEntity()));
		} catch (JSONException | ParseException | IOException e) {
			e.printStackTrace();
		}
    
		if(!ObjectUtils.isEmpty(json) && !json.isEmpty()) {
			item = json.getJSONObject("hits")
					.getJSONArray("hits").getJSONObject(0)
					.getJSONObject("fields")
					.getJSONArray("@timestamp")
					.get(0).toString();
		}

```

```json
// 쿼리 응답 예시
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

Elasticsearch API 쿼리 조회 후 받은 응답값 JSON 파싱했어요.

<br>

## Spring Scheduler 설정

```java
@EnableScheduling
public class RestScheduler {
	private static final double CHECKVM_RESOURCES_MEMORY_LIMIT = 0.8;
	private static final double CHECKVM_RESOURCES_CPU_LIMIT = 0.8;
	private static final int CHECKVM_RESOURCES_SCHEDULE_RATE_SEC = 30;
	
	@Autowired
	ElasticsearchService elasticsearchService;
	
	@Autowired
	AlertService alertService;
	
	@Autowired
	EnvironmentConfig env;
  
	@Scheduled(fixedRate=CHECKVM_RESOURCES_SCHEDULE_RATE_SEC * 1000)
	public int checkVMResourcesSchedule() {
		ListResult<Map<String, String>> response = elasticsearchService.checkVMResources(CHECKVM_RESOURCES_MEMORY_LIMIT, CHECKVM_RESOURCES_CPU_LIMIT);
		if(response.getCode() == 200) {
			return 1;
		}
		List<Map<String, String>> list = response.getList();
		
		StringBuilder sb = new StringBuilder()
				.append("서버 자원 알림");
		
		for(Map<String, String> map : list) {
			if(map.isEmpty()) {
				continue;
			}
			sb.append(StringUtil.formatSafe("\n%s:", map.get("key")));
			
			Double memory = Double.valueOf(map.get("memory"));
			Double cpu = Double.valueOf(map.get("cpu"));

			if(memory > CHECKVM_RESOURCES_MEMORY_LIMIT * 100) {
				sb.append(StringUtil.formatSafe(" mem: %.2f%%", memory));
			}
			
			if(cpu > CHECKVM_RESOURCES_CPU_LIMIT * 100) {
				sb.append(StringUtil.formatSafe(" cpu: %.2f%%", cpu));
			}
		}
		
		String msg = sb.toString();

		int result = 0;
    int result1 = alertService.sendSms(msg);
    int result2 = alertService.sendSlackAlert(msg);
    result = result1 & result2;
		
		return result;
	}
}
```

> 서비스 구현체의 상세한 비즈니스 로직은 생략하였어요.

1. `@EnableScheduling` 어노테이션으로 Spring Scheduler 활성화
2. `@Scheduled(fixedRate=CHECKVM_RESOURCES_SCHEDULE_RATE_SEC * 1000)` 스케줄러 등록 및 주기 설정

<br>

### Slack Webhook 설정

[Incoming Webhooks](https://api.slack.com/messaging/webhooks) 참고하세요.

#### 호출 예시

```java
String uri = env.getSlackWebhookUrl;
		
Map<String, Object> requestMap = new HashMap<>();
requestMap.put("text", message);

HttpRequestInfo requestInfo = new HttpRequestInfo();
requestInfo.setUri(uri);
requestInfo.setMethod("POST");
requestInfo.setConnTimeout(apiConnTimeout);
requestInfo.setReadTimeout(apiReadTimeout);
try {
  requestInfo.setSendData(ObjectMapperUtil.toJsonString(requestMap));
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
requestInfo.setResponseCode(HttpURLConnection.HTTP_UNAVAILABLE);

String responseInfo = "";

try {
  responseInfo = RestTemplateUtil.http(requestInfo, MediaType.APPLICATION_JSON_VALUE);
} catch (Exception e) {
  e.printStackTrace();
}

return StringUtils.equals(responseInfo, "ok") ? 1 : 0;
```

![jeaa-6.png]({{ "/assets/img/contents/jeaa-6.png"}})

응답을 확인할 수 있어요.

---

## Elasticsearch Query DSL

쿼리의 경우 별도 문서에 따로 정리했어요.

> [Elasticsearch 모니터링 쿼리 예제]({% post_url 2021-10-06-elasticsearch-monitoring-query-sample %}) 참고하세요.

---

## Reference

1. [Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)
2. [Incoming Webhooks](https://api.slack.com/messaging/webhooks)