---
title: 라이브러리 소개
description:
keywords:
---

노드 템플릿을 사용할 때는 기본 구성 요소가 이미 조립되어 사용 가능하므로 사용되는 하부 아키텍처나 라이브러리에 대해 알 필요가 없습니다.
그러나 커스텀 블록체인을 설계하고 구축하려면 사용 가능한 라이브러리에 익숙해지고 이러한 다른 라이브러리가 무엇을 하는지 알아야 할 수도 있습니다.

[아키텍처](../basic/architecture.md)에서는 Substrate 노드의 핵심 구성 요소와 노드의 다른 부분이 서로 다른 책임을 가지는 방식에 대해 배웠습니다.
보다 기술적인 수준에서는 노드의 다른 계층 간의 책임 분리가 Substrate 기반 블록체인을 구축하는 데 사용되는 핵심 라이브러리에 반영됩니다.
다음 다이어그램은 라이브러리가 외부 노드와 런타임 책임을 반영하고 두 가지 간의 통신 계층을 제공하는 **원시(primitives)** 라이브러리를 보여줍니다.

![외부 노드와 런타임을 위한 핵심 노드 라이브러리](/media/images/docs/infrablockchain/learn/substrate/learn/libraries.png)

## 핵심 노드 라이브러리

Substrate 노드가 합의 및 블록 실행과 같은 네트워크 책임을 처리할 수 있도록 하는 라이브러리는 `sc_` 접두사를 사용하는 Rust 크레이트입니다.
예를 들어, [`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html) 라이브러리는 Substrate 블록체인을 위한 네트워킹 계층을 구축하고 네트워크 참가자와 트랜잭션 풀 간의 통신을 관리합니다.

외부 노드와 런타임 간의 통신 계층을 제공하는 라이브러리는 `sp_` 접두사를 사용하는 Rust 크레이트입니다.
이러한 라이브러리는 외부 노드와 런타임이 상호 작용하는 활동을 조율합니다.
예를 들어, [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html) 라이브러리는 Rust의 표준 라이브러리에서 유용한 원시(primitives)를 가져와 런타임에 의존하는 코드와 함께 사용할 수 있게 합니다.

런타임 로직을 구축하고 런타임으로 전달되는 정보를 인코딩하고 디코딩하는 데 사용되는 라이브러리는 `frame_` 접두사를 사용하는 Rust 크레이트입니다.
`frame_*` 라이브러리는 런타임을 위한 인프라를 제공합니다.
예를 들어, [`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html) 라이브러리는 다른 Substrate 구성 요소와 상호 작용하기 위한 기본 함수 집합을 제공하며, [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html)는 런타임 저장소 항목, 오류 및 이벤트를 선언할 수 있게 합니다.

`frame_*` 라이브러리가 제공하는 인프라 외에도 런타임에는 하나 이상의 `pallet_*` 라이브러리를 포함할 수 있습니다.
`pallet_` 접두사를 사용하는 각 Rust 크레이트는 단일 FRAME 모듈을 나타냅니다.
대부분의 경우, `pallet_*` 라이브러리를 사용하여 프로젝트에 맞게 블록체인에 통합하려는 기능을 조립합니다.

`sp_*` 핵심 라이브러리가 노출하는 원시(primitives)를 사용하여 Substrate 런타임을 구축할 수도 있습니다.
그러나 `frame_*` 또는 `pallet_*` 라이브러리를 사용하는 것이 Substrate 런타임을 구성하는 가장 효율적인 방법입니다.

## 모듈식 아키텍처

핵심 라이브러리의 분리는 블록체인 로직을 작성하기 위한 유연하고 모듈식 아키텍처를 제공합니다.
원시(primitives) 라이브러리는 외부 노드와 런타임이 서로 직접 통신하지 않고도 구축할 수 있는 기반을 제공합니다.
원시(primitives) 유형과 특성은 별도의 크레이트에서 노출되므로 순환 종속성 문제를 발생시키지 않고 외부 노드 및 런타임 구성 요소에서 사용할 수 있습니다.

## 프론트엔드 라이브러리

Substrate 기반 블록체인을 구축하는 데 사용할 수 있는 핵심 라이브러리 외에도 Substrate 노드와 상호 작용하기 위해 사용할 수 있는 클라이언트 라이브러리가 있습니다.
클라이언트 라이브러리를 사용하여 응용 프로그램별 프론트엔드를 구축할 수 있습니다.
일반적으로 클라이언트 라이브러리가 노출하는 기능은 Substrate 원격 프로시저 호출(RPC) API 위에 구현됩니다.
응용 프로그램을 구축하기 위한 메타데이터 및 프론트엔드 라이브러리에 대한 자세한 정보는 [응용 프로그램 개발](../../build/application-development.md#rpc-api)을 참조하십시오.

## 다음 단계로 넘어가기

Substrate 노드를 구축하고 상호 작용하기 위해 사용되는 라이브러리에 익숙해졌으므로 더 깊이있게 라이브러리를 탐색해 볼 수 있습니다.
각 라이브러리에 대한 기술적인 세부 정보를 알아보려면 해당 라이브러리의 [Rust API](https://paritytech.github.io/substrate/master/) 문서를 검토해야 합니다.