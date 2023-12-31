---
title: 아키텍처와 Rust 라이브러리
description: Substrate 노드의 핵심 구성 요소를 소개합니다.
keywords:
---

[블록체인 기본](../basic/blockchain-basics.md)에서 언급한 바와 같이, 블록체인은 서로 통신하는 컴퓨터들의 탈중앙화된 네트워크인 노드들에 의존합니다.

노드는 어떤 블록체인이든 핵심 구성 요소이기 때문에, Substrate 노드가 독특하게 만드는 요소들, 기본적으로 제공되는 핵심 서비스와 라이브러리들, 그리고 노드를 다양한 프로젝트 목표에 맞게 사용자 정의하고 확장하는 방법을 이해하는 것이 중요합니다.

## 클라이언트와 런타임

Substrate 노드는 크게 두 가지 주요 부분으로 구성됩니다:

- **코어 클라이언트**와 **외부 노드 서비스**로 구성된 클라이언트는 피어 검색, 트랜잭션 요청 관리, 피어와의 합의 도달, RPC 호출에 응답하는 등 네트워크 활동을 처리합니다.

- 블록체인의 상태 전이 기능을 실행하는 비즈니스 로직을 포함하는 **런타임**입니다.

다음 다이어그램은 아키텍처와 Substrate가 블록체인을 구축하기 위한 모듈화된 프레임워크를 제공하는 방식을 시각화하기 위해 단순화된 형태로 분리된 책임을 보여줍니다.

![Substrate 아키텍처](/media/images/docs/infrablockchain/learn/substrate/learn/simplified-architecture.png)

## 클라이언트 외부 노드 서비스

코어 클라이언트에는 런타임 외부에서 발생하는 활동을 처리하는 여러 외부 노드 서비스가 포함되어 있습니다.
예를 들어, 코어 클라이언트의 외부 노드 서비스는 피어 검색, 트랜잭션 풀 관리, 합의 도달을 위한 다른 노드와의 통신, 외부 세계로부터의 RPC 요청에 응답하는 등의 역할을 합니다.

코어 클라이언트 서비스가 처리하는 가장 중요한 활동 중 일부는 다음과 같은 구성 요소들과 관련이 있습니다:

- [저장소](../frame/state-transitions-and-storage.md): 외부 노드는 간단하고 매우 효율적인 Key-Value 저장소 계층을 사용하여 Substrate 블록체인의 진화하는 상태를 유지합니다.

- [피어 간 네트워킹](../basic/networks-and-nodes.md): 외부 노드는 다른 네트워크 참가자들과 통신하기 위해 Rust 구현인 [`libp2p` 네트워크 스택](https://libp2p.io/)을 사용합니다.

- [합의](../basic/consensus.md): 외부 노드는 다른 네트워크 참가자들과 통신하여 블록체인의 상태에 대해 합의를 이끌어냅니다.

- [원격 프로시저 호출 (RPC) API](../../build/remote-procedure-calls.md): 외부 노드는 블록체인 사용자가 네트워크와 상호작용할 수 있도록 들어오는 HTTP 및 WebSocket 요청을 수락합니다.

- [텔레메트리](../../../../devops/monitor-node-metrics.md): 외부 노드는 내장된 [Prometheus](https://prometheus.io/) 서버를 통해 노드 메트릭에 대한 수집 및 액세스를 제공합니다.

- [실행 환경](../../build/build-process.md): 외부 노드는 런타임이 사용할 실행 환경인 WebAssembly 또는 네이티브 Rust를 선택하고 선택한 런타임에 호출을 보냅니다.

Substrate는 이러한 활동을 처리하기 위한 기본 구현을 제공합니다.
원칙적으로, 어떤 구성 요소의 기본 구현을 수정하거나 대체할 수 있습니다.
실제로는, 대부분의 애플리케이션은 기본 블록체인 기능에 대한 변경이 필요하지 않지만, Substrate를 사용하면 필요한 경우 변경할 수 있으므로 필요한 곳에서 혁신할 수 있습니다.

이러한 작업을 수행하기 위해서는 클라이언트 노드 서비스가 런타임과 통신해야 합니다.
이 통신은 전용 [런타임 API](../frame/runtime-apis.md)를 호출하여 처리됩니다.

## 런타임

런타임은 트랜잭션이 유효한지 아닌지를 결정하고, 블록체인 상태의 변경을 처리하는 역할을 담당합니다.
외부에서 들어오는 요청은 클라이언트를 통해 런타임으로 전달되며, 런타임은 상태 전이 함수를 실행하고 결과 상태를 저장하는 역할을 담당합니다.

런타임은 받은 함수를 실행함으로써 트랜잭션이 블록에 포함되고, 블록이 외부 노드로 반환되어 전파되거나 다른 노드로 가져오는 방식을 제어합니다.
즉, 런타임은 체인 상에서 발생하는 모든 일을 처리하는 역할을 담당합니다.
또한 Substrate 블록체인을 구축하기 위한 노드의 핵심 구성 요소입니다.

Substrate 런타임은 [WebAssembly (Wasm)](../basic/glossary.md#webassembly-wasm) 바이트 코드로 컴파일될 수 있도록 설계되었습니다.
이 설계 결정은 다음을 가능하게 합니다:

- 포크 없는 업그레이드 지원.
- 다중 플랫폼 호환성.
- 런타임 유효성 검사.
- 릴레이 체인 합의 메커니즘에 대한 검증 증명.

외부 노드가 런타임에 정보를 제공할 수 있는 방법과 마찬가지로, 런타임은 외부 노드나 외부 세계와 통신하기 위해 전용 [호스트 함수](https://paritytech.github.io/substrate/master/sp_io/index.html)를 사용합니다.

## 핵심 라이브러리

노드 템플릿에서 블록체인의 많은 측면은 기본 구현으로 구성됩니다.
예를 들어, 네트워킹 레이어, 데이터베이스, 합의 메커니즘에 대한 기본 구현이 제공되어 많은 사용자 정의 없이 블록체인을 실행할 수 있습니다.
그러나 기본 아키텍처의 기반이 되는 라이브러리들은 사용자 정의 블록체인 구성 요소를 정의하는 데 많은 유연성을 제공합니다.

노드가 코어 클라이언트와 런타임이라는 두 가지 주요 부분으로 구성되어 다른 서비스를 제공하는 것처럼, Substrate 라이브러리는 다음과 같은 세 가지 주요 책임 영역으로 나뉘어 있습니다:

- 외부 노드 서비스를 위한 코어 클라이언트 라이브러리.
- 런타임을 위한 FRAME 라이브러리.
- 라이브러리 간 통신을 위한 기본 기능과 인터페이스를 제공하는 Primitive 라이브러리.

다음 다이어그램은 라이브러리가 코어 클라이언트 외부 노드와 런타임 책임을 반영하고, **Primitive** 라이브러리가 두 가지 간의 통신 계층을 제공하는 방식을 보여줍니다.

![코어 노드 라이브러리: 외부 노드와 런타임](/media/images/docs/infrablockchain/learn/substrate/learn/libraries.png)

### 코어 클라이언트 라이브러리

Substrate 노드가 네트워크 책임, 합의 및 블록 실행과 같은 역할을 수행할 수 있도록 하는 라이브러리는 크레이트 이름에 `sc_` 접두사를 사용하는 Rust 크레이트입니다.
예를 들어, [`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html) 라이브러리는 Substrate 블록체인을 위한 네트워킹 레이어를 구축하고, 네트워크 참가자와 트랜잭션 풀 간의 통신을 관리합니다.

### 런타임을 위한 FRAME 라이브러리

런타임 로직을 구축하고 런타임으로 전달되는 정보를 인코딩하고 디코딩하는 데 사용되는 라이브러리는 크레이트 이름에 `frame_` 접두사를 사용하는 Rust 크레이트입니다.

`frame_*` 라이브러리는 런타임을 위한 인프라를 제공합니다.
예를 들어, [`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html) 라이브러리는 다른 Substrate 구성 요소와 상호작용하기 위한 기본 함수 집합을 제공하며, [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html)는 런타임 저장소 항목, 오류 및 이벤트를 선언할 수 있도록 지원합니다.

`frame_*` 라이브러리가 제공하는 인프라 외에도, 런타임에는 하나 이상의 `pallet_*` 라이브러리를 포함할 수 있습니다.
`pallet_` 접두사를 사용하는 각 Rust 크레이트는 하나의 FRAME 모듈을 나타냅니다.
대부분의 경우, 프로젝트에 맞게 블록체인에 통합할 기능을 조합하기 위해 `pallet_*` 라이브러리를 사용합니다.

**Primitive** 라이브러리를 사용하지 않고 Substrate 런타임을 구축할 수도 있습니다.
그러나 `frame_*` 또는 `pallet_*` 라이브러리를 사용하는 것이 Substrate 런타임을 구성하는 가장 효율적인 방법입니다.

### Primitive 라이브러리

Substrate 아키텍처의 가장 낮은 수준에는 핵심 클라이언트 서비스와 런타임 간의 통신을 위한 Primitive 라이브러리가 있습니다.
Primitive 라이브러리는 크레이트 이름에 `sp_` 접두사를 사용하는 Rust 크레이트입니다.

Primitive 라이브러리는 핵심 클라이언트 서비스나 런타임이 서로 통신하거나 작업을 수행하기 위해 사용할 수 있는 인터페이스를 노출하는 가장 낮은 수준의 추상화를 제공합니다.

예를 들어:

- [`sp_arithmetic`](https://paritytech.github.io/substrate/master/sp_arithmetic/index.html) 라이브러리는 런타임에서 사용할 고정 소수점 산술 Primitive와 타입을 정의합니다.
- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html) 라이브러리는 공유 가능한 Substrate 타입 집합을 제공합니다.
- [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html) 라이브러리는 Rust 표준 라이브러리에서 가져온 Primitive를 내보내어 런타임에 의존하는 모든 코드에서 사용할 수 있도록 합니다.

## 모듈화된 아키텍처

Substrate 라이브러리의 분리는 블록체인 로직을 작성하기 위한 유연하고 모듈화된 아키텍처를 제공합니다.
Primitive 라이브러리는 코어 클라이언트와 런타임이 서로 직접 통신하지 않고도 구축할 수 있는 기반을 제공합니다.
Primitive 타입과 트레이트는 각각 별도의 크레이트로 노출되므로, 외부 노드 서비스와 런타임 구성 요소에서 순환 종속성 문제를 발생시키지 않고 사용할 수 있습니다.

## 다음 단계로 넘어가기

Substrate 노드를 구축하고 상호작용하기 위해 사용되는 아키텍처와 라이브러리에 익숙해졌으므로, 라이브러리를 더 깊이 탐색해볼 수 있습니다.
각 라이브러리에 대한 기술적인 세부 정보를 알아보려면 해당 라이브러리의 [Rust API](https://paritytech.github.io/substrate/master/) 문서를 검토해야 합니다.