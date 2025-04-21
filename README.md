# kafka-registry

Kafka 스키마 레지스트리 정의 저장소입니다.

이 저장소는 Kafka 기반 마이크로서비스 간 통신을 위한 Avro 스키마를 관리합니다.  
Schema Registry를 기반으로 데이터 계약(Contract)을 정리하고 버전 관리합니다.

## 🗂️ 디렉토리 구조

```
src/
  main/
    avro/
      /order
        OrderEvent.avsc
        StockUpdateEvent.avsc
      /test
        LoadTestEvent.avsc
```

- 각 `.avsc` 파일은 Avro 스키마 정의이며, `namespace`로 빌드 시 객체를 저장할 경로를 지정합니다.
- `src/main/avro`는 빌드 도구가 avro를 찾는 기본 경로이며, 변경할 시 따로 명시해줘야 합니다.

## 🛠️ 사용 방법

1. 이 레포를 서브모듈로 추가하거나, `.avsc` 파일을 직접 복사합니다.
   ```bash
   git submodule add https://github.com/Team-Project-MSA-InnerArchitecture/kafka-registry.git src/main/avro-schemas
   ```

2. 빌드 도구에 설정을 추가합니다. (gradle 예시)
   ```groovy
   plugins {
     id 'com.github.davidmc24.gradle.plugin.avro' version '1.5.0'
   }
    
   repositories {
     mavenCentral()
     maven { url 'https://packages.confluent.io/maven/' }
   }
    
   dependencies {
     //kafka
     implementation 'org.springframework.kafka:spring-kafka'
     implementation 'com.fasterxml.jackson.core:jackson-databind'
   
     //avro
     implementation 'org.apache.avro:avro:1.11.4'
     implementation 'io.confluent:kafka-avro-serializer:7.6.0'
   }
    
   avro {
     fieldVisibility = "PRIVATE"
   }
   ```

3. 카프카 서버를 설정 파일에 등록합니다. (yml 예시)
```YAML
spring:
  kafka:
    bootstrap-servers: 43.200.90.54:9092
    consumer:
      group-id: your-service-name
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: io.confluent.kafka.serializers.KafkaAvroDeserializer
    properties:
      schema.registry.url: http://43.200.90.54:8081
      specific.avro.reader: true
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: io.confluent.kafka.serializers.KafkaAvroSerializer
```

## 📦 스키마 예시

```json
{
  "type": "record",
  "name": "TestEvent",
  "namespace": "com.example.kafka_schemas",
  "fields": [
    { "name": "id", "type": "int" },
    { "name": "message", "type": "string" },
    {
      "name": "user",
      "type": {
        "type": "record",
        "name": "User",
        "fields": [
          { "name": "userId", "type": "int" },
          { "name": "username", "type": "string" }
        ]
      }
    },
    {
      "name": "tags",
      "type": { "type": "array", "items": "string" }
    }
  ]
}
```
- 해당 스키마는 다음과 같은 자바 객체로 변환됩니다. 
```java
public class TestEvent extends SpecificRecordBase {
    private int id;
    private String message;
    private User user;
    private List<String> tags;
}
```

## 🤝 PR 규칙

- 새로운 스키마 추가 및 변경은 반드시 PR로 진행합니다.
- PR 시 `namespace`, `name`, `field` 변경 사항을 명확히 기술해주세요.
- 스키마 변경 시 각 MS팀 간의 호환성(backward compatibility)을 고려해야 합니다.
  
  -> 이너 아키텍처 팀에서 각 팀의 호환성 체크 후 머지하겠습니다.

## 🔗 참고

- [Confluent Schema Registry Docs](https://docs.confluent.io/platform/current/schema-registry/index.html)
- [Avro Specification](https://avro.apache.org/docs/current/spec.html)
