# fluentd-kafka

fluentd와 kafka를 연결하는 예제

- Kafka 사용시 매번 Producer, Consumer를 직접 코딩하여 만드는 것은 비효율적이다.
- 강력한 파이프라인 도구 중 하나인 fluentd로 Producer, Consumer를 대체할 수 있는 유스케이스가 많다.
- 이를 위한 사전테스트, 템플릿을 다루는 레포지토리

## Requirement

- K3s
- Helm

## fluentd

```sh
# fluentd
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-fluentd bitnami/fluentd --version 5.19.0

# kafka
# broker 3, connect 1, private ip, KRAFT
helm install test https://github.com/YunanJeong/simple-kafka-deploy/releases/download/v2.0.3/skafka-2.0.3.tgz \
-f https://github.com/YunanJeong/simple-kafka-deploy/releases/download/v2.0.3/kraft-multi.yaml
```

## 플러그인 gem 설치

### 온라인

```yaml
# 필요한 gem을 value에 기술하면 자동설치
aggregator:
  extraGems: # []
    - name: fluent-plugin-beats 
      version: 2.1.2  # default: latest
```

### 오프라인

- 필요한 gem파일을 로컬호스트의 정해진 경로에 미리 배치하고, 볼륨마운트 기능으로 컨테이너에 배포

```sh
# 플러그인파일들을 호스트마운트 경로로 옮기기
sudo cp -r plugins/ /etc/
```

- 이후 다음 과정으로 배포된다. 샘플파일 `myvalues.yaml` 참고
  1. 로컬 gem경로가 initContainer에 마운트
  2. initContainer에서 gem이 설치됨
  3. 설치된 파일내역은 initContainer와 App. Container 간 공유경로(볼륨)으로 복사됨

```yaml
# 사용되는 value 섹션들
aggregator:
  initContainers: []
  extraVolumes: []
  extraVolumeMounts: []
```

## MEMO

- 커뮤니티에선 fluentd를 aggregator와 forwarder로 구분하는 데, fluentd 공식은 아니다.
- 공식 fluentd(td-agent)처럼 단일앱으로만 실행하려면, aggregator만 활성화
- forwarder는 DaemomSet으로 구현되어있으므로 비활성화
- kafka, prometheus, elasticsearch 등 주요 플랫폼은 이미 gem 플러그인이 내장되어 있어, 추가 설치가 필요없음
