# ADR-000: 프로세스 · 스레드 학습 범위와 구현 순서

- 상태: 학습 예정 — 코드는 대부분 있지만 실행 결과 기록이 없다. 나중에 코드를 함께 읽으며 한 단계씩 배우고, 그때 결과를 남긴다.
- 날짜: 2026-11-18(구현 순서 문서를 이 ADR로 정리한 날. 원래 문서는 2025-09-21 커밋 "feat : add IMPLEMENTATION_ORDER.md"에서 들어왔다)
- 성격: 설계 · 범위 정리. 원문은 [`docs/archive/IMPLEMENTATION_ORDER.md`](../archive/IMPLEMENTATION_ORDER.md)로 옮겼다. 원문 내용은 그대로 두고 머리말 1곳만 더했다(§7 대응표).

## 1. 이 랩이 묻는 것

같은 일을 프로세스, 플랫폼 스레드, 가상 스레드로 할 때 생성 비용 · 컨텍스트 스위칭 · 메모리 · 처리량이 실제로 어떻게 다른가. 그리고 운영체제 교과서의 개념(상태 전이, 동기화, 스케줄링, IPC)이 JVM에서 어떻게 보이는가.

이론 정리는 [`docs/theory.md`](../theory.md)에 있다.

## 2. 원문에 설계안이 두 벌 있다

`IMPLEMENTATION_ORDER.md` 한 파일에 서로 다른 구현 순서 두 개가 이어 붙어 있다. 두 번째 안은 "Phase 1" 제목 없이 "### 목표"로 바로 시작한다(보존 원문 701줄, "각 언어의 특성을 살려 구현하고…" 문단 바로 뒤).

| | 첫 번째 안 (보존 원문 1~700줄) | 두 번째 안 (보존 원문 701줄~끝) |
|---|---|---|
| 언어 · 프레임워크 | Java(Spring Boot) + Python(FastAPI)을 함께 만들어 비교 | Java만 |
| 단계 | 8단계: 설정 → Process → Thread → 동기화 → 웹 프레임워크 통합 → 고급 기능 → 성능 비교 → 통합 · 문서화 | 7단계: 공통 유틸 → Process → Thread → 동기화 → 고급(IPC · 스레드 풀 · 스케줄링) → 성능 측정 → 통합(Main · 모니터링 · 보고서) |
| 패키지 | `com.study.processthread.*` (web · config 포함) | `com.study.*` (process · thread · comparison · common) |
| 일정 · 체크리스트 | 단계별 예상 소요 시간(1~4일) 표, Week 1~5 체크리스트 | 단계별 체크리스트 7개, 완성도 체크리스트 |
| 지금 코드와의 관계 | `python-fastapi/`, `docker/` 파일 일부가 이 안을 따른다 | `java-spring/src/main/java/com/study/` 아래 코드가 이 안을 따른다 |

원문의 "예상 소요 시간"은 일정 예상이고 측정값이 아니다. 체크리스트는 두 안 모두 전부 미체크다.

## 3. 범위

| 넣는 것 | 넣지 않는 것 |
|---------|-------------|
| 프로세스 생성 · 모니터링 · 관리, IPC(파이프 · 소켓 · 공유 메모리 · 메시지 큐) | 여러 머신에 걸친 분산 처리(원문 "다음 단계") |
| 스레드 생명주기, 플랫폼 · 가상 스레드, 스레드 풀, Structured Concurrency · Scoped Value | Kubernetes 스케줄링, Actor 모델, Kotlin 코루틴(원문 "다음 단계") |
| 동기화(mutex · semaphore · monitor · ReentrantLock · ReadWriteLock · StampedLock), 데드락 · 경쟁 조건 | |
| 스케줄링 알고리즘 시뮬레이션(FCFS · SJF · RR · Priority · MLFQ) | |
| 성능 비교(생성 · 컨텍스트 스위칭 · I/O · CPU · 확장성 · 처리량), 리소스 분석 | |

## 4. 선택지

| 항목 | 선택지 | 지금 상태 |
|------|--------|----------|
| 비교 대상 언어 | Java만(두 번째 안) · Java + Python(첫 번째 안) | 정하지 않았다. Java 코드는 대부분 있고, Python은 프로세스 생성 하나와 API 뼈대만 있다 |
| 실행 형태 | 클래스마다 `main`으로 데모 실행 · `com.study.Main` 하나에서 고르는 CLI · Spring Boot 앱 | 클래스마다 `main`이 있다. `Main` · `DemoRunner`는 없다. `java-spring/build.gradle`의 `mainClass`(`com.study.processthread.ProcessThreadApplication`)를 가리키는 클래스도 없다 |
| 벤치마크 방법 | JMH · 직접 `System.nanoTime` 측정 | `java-spring/build.gradle`에 JMH 플러그인이 있다. 결과를 어떻게 남길지는 정하지 않았다 |
| 측정 환경 | 로컬 PC · 실험 서버 | 정하지 않았다. 잴 때 하드웨어 · JDK 버전을 함께 남긴다 |

## 5. 지금 있는 것

- `java-spring/src/main/java/com/study/` 아래 클래스 71개(공통 유틸 · 모니터, process, thread, comparison). 두 번째 안의 1~6단계 클래스는 `ProducerConsumer`를 빼고 있다.
- 테스트: `ChildProcessTest`, `ProcessCreatorTest` 두 개.
- `python-fastapi/`: `process_creator.py`, API 엔드포인트 파일, 단위 테스트 하나.
- `docker/`(compose · Dockerfile · prometheus 설정), `scripts/`(`.bat` 위주, `run-all.sh` 하나).
- 실행 결과 기록: 없다. README에도 측정값이 없다. README의 "100만 개의 Virtual Thread" 같은 숫자는 시나리오 목표다.

## 6. 아직 없는 것

README의 프로젝트 구조와 원문이 약속하지만 저장소에 없는 것이다. README의 구조도는 저장소 루트에 `src/`가 있는 것처럼 그렸지만, 실제 코드는 `java-spring/src/` 아래에 있다.

- 진입점: `com.study.Main`, `com.study.DemoRunner`, `ProcessThreadApplication`
- 클래스: `ProducerConsumer`, `ProcessOrchestrator`, `StructuredTaskScope`(대신 `StructuredTaskScopeDemo`가 있다)
- 테스트: README 구조도의 테스트 중 위 두 개를 뺀 나머지 전부(`ProcessFactoryTest`, `SocketIPCTest`, `MutexDemoTest`, `DeadlockDetectorTest`, `FCFSSchedulerTest`, `CreationBenchmarkTest` 등 29개)와 `test-application.properties`, `test-data/`
- 리소스: `benchmark.properties`, `demo-scripts/`
- 문서: `docs/architecture/`(system-design · UML), `docs/experiments/experiment-01~05`, `docs/results/`(benchmark-results · performance-analysis · charts)
- 스크립트: `run-process-demo.sh`, `run-thread-demo.sh`, `run-virtual-thread-demo.sh`, `run-all-benchmarks.sh`, `generate-report.sh`, `setup-environment.sh`
- 루트 파일: `build.gradle`, `gradle.properties`, `LICENSE`, `logs/`
- 첫 번째 안의 `shared/`(공통 벤치마크 시나리오 · 대시보드 · 비교 문서), Python의 thread · comparison 모듈과 벤치마크

## 7. 원문과의 대응

| 원문 절 | 이 ADR |
|--------|--------|
| 개요 · 프로젝트 구조 개요 · 구현 단계 개요(예상 소요 시간 표) | §2 |
| 첫 번째 안 Phase 1~8 | §2 표, §3, §4 |
| 구현 순서 체크리스트(Week 1~5) | §2 (전부 미체크) |
| 주요 비교 포인트 · 개발 환경 설정 · 테스트 전략 | 원문에 둠 |
| 두 번째 안 Phase 1~7 | §2 표, §3, §5, §6 |
| 구현 팁과 주의사항(단계별 체크리스트 · 개발 환경 · 코딩 규칙 · 디버깅 팁 · 자주 발생하는 문제) | 원문에 둠, 체크리스트는 §2 |
| 완성도 체크리스트 | §2 (전부 미체크) |
| 다음 단계 | §3 넣지 않는 것 |

## 8. 다음 할 일

1. §4의 비교 대상 언어와 실행 형태를 정한다.
2. 두 번째 안 순서대로 코드를 함께 읽으며, 단계마다 실제로 돌린 결과(명령 · 환경 · 출력)를 남긴다.
3. 결과가 생기면 이 ADR의 상태와 README 상단 절을 바꾼다.
