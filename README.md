# kafka-registry

Kafka 스키마 레지스트리 정의 저장소입니다.

이 저장소는 Kafka 기반 마이크로서비스 간 통신을 위한 Avro 스키마를 관리합니다.  
Schema Registry를 기반으로 데이터 계약(Contract)을 정리하고 버전 관리합니다.

## 🗂️ 디렉토리 구조

```
com/
  example/
    kafka_schemas/
      /order
        OrderEvent.avsc
        StockUpdateEvent.avsc
      /test
        LoadTestEvent.avsc
```

- 각 `.avsc` 파일은 Avro 스키마 정의이며, `namespace`에 따라 디렉토리 구조를 맞춥니다.

## 🛠️ 사용 방법

1. 이 레포를 서브모듈로 추가하거나, `.avsc` 파일을 직접 복사합니다.
   ```bash
   git submodule add https://github.com/Team-Project-MSA-InnerArchitecture/kafka-registry.git src/main/avro-schemas
   ```

2. Spring 또는 Kafka Producer/Consumer 설정에서 Avro 스키마 경로로 지정합니다.
   ```groovy
   avro {
       fieldVisibility = "PRIVATE"
   }
   ```

3. 각 서비스는 `build.gradle`이나 CI/CD에서 `.avsc` 파일을 자동 컴파일하여 사용합니다.

## 📦 스키마 예시

```json
{
  "type": "record",
  "name": "OrderEvent",
  "namespace": "com.example.kafka_schemas",
  "fields": [
    { "name": "orderId", "type": "int" },
    { "name": "status", "type": "string" },
    { "name": "qty", "type": "int" }
  ]
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
