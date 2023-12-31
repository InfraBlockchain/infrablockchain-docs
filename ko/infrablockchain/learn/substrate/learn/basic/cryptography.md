---
title: 암호학
description: Substrate에서 사용되는 해시 알고리즘과 암호화 서명 방식에 대한 정보를 요약합니다.
keywords:
  - 암호학
  - 해싱
  - 서명
  - ECDSA
  - Ed25519
  - SR25519
  - 계층적 결정론적 키
  - 키 파생
---

암호학은 합의 시스템, 데이터 무결성 및 사용자 보안에 대한 수학적 검증 가능성을 제공합니다. 블록체인과 관련된 암호학의 기본 응용 프로그램을 이해하는 것은 일반 개발자에게 필수적이지만, 수학적인 과정 자체는 필요하지 않을 수 있습니다. 이 페이지는 Parity 및 생태계 전반에 걸쳐 암호학의 다양한 구현에 대한 기본적인 컨텍스트를 제공합니다.

## 해시 함수

**해싱**은 2개의 임의의 고유한 숫자 입력을 사용하여 임의의 데이터와 32바이트 참조 사이에 일대일 매핑을 생성하는 수학적인 과정입니다. 해시 함수를 사용하면 간단한 텍스트, 이미지 또는 기타 형식의 파일을 포함한 모든 데이터에 고유한 식별자가 부여됩니다. 해싱은 데이터 무결성을 검증하고 전자 서명을 생성하며 비밀번호를 안전하게 저장하는 데 사용됩니다. 이러한 매핑 형태는 '비둘기집 원리'라고도 알려져 있으며, 주로 대규모 데이터 세트에서 데이터를 효율적이고 검증 가능하게 식별하기 위해 구현됩니다.

이러한 함수들은 **결정론적**이므로 동일한 입력은 항상 동일한 출력을 생성합니다. 이는 두 개의 다른 컴퓨터가 동일한 데이터에 동의할 수 있도록 보장하는 데 중요합니다. 이러한 함수는 목적에 따라 빠르게 또는 느리게 설계될 수 있습니다. 속도가 중요한 경우 빠른 해시 함수가 사용되고, 보안이 우선인 경우 느린 해시 함수가 사용됩니다. 느린 해시 함수는 데이터를 찾기 위해 필요한 작업량을 증가시켜 무차별 대입 공격의 성공을 완화하는 데에도 사용됩니다.

### 충돌 저항성

블록체인에서 해시 함수는 또한 **충돌 저항성**을 제공하기 위해 사용됩니다. 이는 공격자가 두 개의 동일한 값을 찾아 암호화된 객체에 액세스하기 위해 두 개의 숫자 입력을 계산하거나 제어하는 것입니다. 부분적인 충돌의 경우, 비슷한 방법이 적용되지만 전체가 아닌 처음 몇 비트를 공유하는 두 값을 찾으려고 시도합니다.

부분적인 충돌 저항성만 구현하는 것은 계산적으로 가볍고 충돌 가능성의 가능성에 대한 꽤 강력한 보호를 제공하지만, 국가 차원의 잘 자원화된 상대방과 마주치는 경우에는 보안 수준이 낮은 옵션입니다. 상당한 계산 능력을 사용하여 처음 몇 자릿수를 무차별 대입 공격을 통과하는 것은 상당히 쉽기 때문입니다. 그렇지만 일반적인 공격 벡터(즉, 악의적인 주체)에 대해서는 허용할 수 있습니다.

### Blake2

새로운 블록체인 프로토콜이나 생태계를 설계할 때 사용되는 암호학 방법의 계산 비용을 고려하는 것이 중요합니다. Substrate는 효율성과 프로세서 부하를 우선시하여 Blake2를 사용합니다.

Blake2는 SHA2보다 동등하거나 더 높은 보안을 제공하면서도 다른 비교 가능한 알고리즘보다 훨씬 빠릅니다. 정확한 벤치마크를 결정하는 것은 하드웨어 사양에 크게 의존하기 때문에 Substrate에 대한 가장 큰 긍정적인 영향은 새로운 노드가 체인과 동기화하는 데 필요한 시간과 리소스를 크게 줄이는 데 있으며, 덜 중요하지만 검증에 필요한 시간도 줄일 수 있습니다.

Blake2에 대한 종합적인 내용은 [공식 문서](https://www.blake2.net/blake2.pdf)를 참조하십시오.

## 암호학의 종류

암호학 알고리즘은 **대칭 암호**과 **비대칭 암호** 두 가지 방식으로 구현됩니다.

### 대칭 암호

**대칭 암호화**는 비대칭 암호학과 달리 일방 함수에 기반하지 않는 암호학의 한 분야입니다. 평문의 암호화와 암호문의 복호화에 동일한 암호키를 사용합니다.

대칭 암호학은 역사를 통해 사용된 암호화 방식입니다. 예를 들어 Enigma 암호와 Caesar 암호가 있습니다. 현재에도 널리 사용되며, web2 및 web3 애플리케이션에서 찾을 수 있습니다. 하나의 키만 있으며, 수신자는 포함된 정보에 액세스하기 위해 해당 키에도 액세스해야 합니다.

### 비대칭 암호

**비대칭 암호화**는 두 개의 다른 키, 즉 키 쌍을 사용하는 암호학의 한 유형입니다. 공개 키는 평문을 암호화하는 데 사용되고, 개인 키는 암호문을 복호화하는 데 사용됩니다.

공개 키는 수신자의 개인 키와 때때로 설정된 비밀번호를 사용하여 암호화된 고정 길이 메시지를 복호화하는 데 사용됩니다. 공개 키는 해당 개인 키가 데이터를 생성하는 데 사용되었음을 암호학적으로 확인하는 데 사용될 수 있습니다. 이는 **전자 서명**과 같은 용도로 사용됩니다. 이는 신원, 소유권 및 속성에 대한 명백한 영향을 미치며, web2 및 web3 모두에서 다양한 프로토콜에서 사용됩니다.

### 트레이드오프와 타협

대칭 암호학은 비대칭 암호학이 제공하는 동일한 수준의 보안을 달성하기 위해 키의 비트 수를 줄이고 더 빠르게 동작합니다. 그러나 통신이 이루어지기 전에 공유 비밀이 필요하므로 무결성과 잠재적인 타협점이 발생합니다. 반면에 비대칭 암호학은 사전에 비밀을 공유할 필요가 없으므로 훨씬 더 우수한 최종 사용자 보안을 제공할 수 있습니다.

비대칭 암호학의 엔지니어링 문제를 극복하기 위해 하이브리드 대칭 및 비대칭 암호학이 종종 사용됩니다. 비대칭 암호학은 더 느리고 더 많은 비트의 키가 필요하기 때문에 상대적으로 가벼운 대칭 암호를 사용하여 메시지에 대한 '중요한 작업'을 수행합니다.

## 전자 서명

전자 서명은 비대칭 키 쌍을 사용하여 문서나 메시지의 진위성을 검증하는 방법입니다. 발신자나 서명자의 문서나 메시지가 전송 중에 변경되지 않았는지 확인하고, 수신자가 해당 데이터가 정확하고 예상한 발신자로부터 온 것임을 검증하는 데 사용됩니다.

전자 서명은 수학과 암호학에 대한 낮은 수준의 이해만으로도 수행할 수 있습니다. 개념적인 예를 들어 체크를 서명할 때는 체크가 여러 번 현금화될 수 없도록 기대됩니다. 이는 서명 시스템의 기능이 아니라 체크 직렬화 시스템의 특징입니다. 은행은 체크의 일련번호가 이미 사용되지 않았는지 확인합니다. 전자 서명은 이러한 두 가지 개념을 결합하여 _서명 자체_가 재현될 수 없는 고유한 암호학적 지문을 통해 직렬화를 제공합니다.

펜과 종이 서명과 달리 전자 서명의 지식은 다른 서명을 생성하는 데 사용될 수 없습니다. 전자 서명은 종이에 서명을 스캔하여 문서에 붙이는 것보다 안전하므로 관료적인 프로세스에서 자주 사용됩니다.

Substrate는 여러 가지 다른 암호학적 방식을 제공하며, [`Pair` trait](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Pair.html)를 구현하는 모든 것을 지원할 수 있도록 일반화되어 있습니다.

## 타원 곡선

블록체인 기술은 블록 제안 및 검증을 위해 여러 개의 키가 서명을 생성할 수 있는 능력을 요구합니다. 이를 위해 타원 곡선 전자 서명 알고리즘(ECDSA)과 Schnorr 서명(Schnorr signatures)이 가장 일반적으로 사용되는 방법입니다. ECDSA는 훨씬 더 간단한 구현이지만, Schnorr 서명은 다중 서명에 대해 더 효율적입니다.

Schnorr 서명은 [ECDSA](#ecdsa)/EdDSA 방식보다 명확한 기능을 제공합니다:

- 계층적 결정론적 키 파생에 더 적합합니다.
- [서명 집계](https://bitcoincore.org/en/2017/03/23/schnorr-signature-aggregation/)를 통해 원시(raw) 다중 서명을 지원합니다.
- 일반적으로 오용에 더 저항합니다.

Schnorr 서명을 사용할 때 ECDSA에 비해 얻는 이점 중 하나는 두 가지 모두 64바이트를 필요로 한다는 점입니다. 그러나 ECDSA 서명만이 공개 키를 전달합니다.

### 다양한 구현

#### ECDSA

Substrate는 [secp256k1](https://en.bitcoin.it/wiki/Secp256k1) 곡선을 사용하는 [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) 서명 방식을 제공합니다. 이는 [Bitcoin](https://en.wikipedia.org/wiki/Bitcoin) 및 [Ethereum](https://en.wikipedia.org/wiki/Ethereum)에서 보안에 사용되는 동일한 암호학적 알고리즘입니다.

#### Ed25519

[Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519)은 [Curve25519](https://en.wikipedia.org/wiki/Curve25519)을 사용하는 EdDSA 서명 방식입니다.
보안을 저해하지 않고 매우 높은 속도를 달성하기 위해 여러 수준의 설계 및 구현에서 신중하게 설계되었습니다.

#### SR25519

[SR25519](https://research.web3.foundation/Polkadot/security/keys/accounts-more)은 [Ed25519](#ed25519)과 동일한 기본 곡선을 사용합니다.
그러나 EdDSA 방식 대신 Schnorr 서명을 사용합니다.

## 다음으로 어디로 가야 할까요

- [Polkadot의 암호학](https://wiki.polkadot.network/docs/en/learn-cryptography).
- [W3F의 연구: 암호학](https://research.web3.foundation/crypto).
- 새로운 해시 알고리즘을 구현하기 위한 [`Hash`](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.Hash.html) trait.
- 새로운 암호학적 방식을 구현하기 위한 [`Pair`](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Pair.html) trait.