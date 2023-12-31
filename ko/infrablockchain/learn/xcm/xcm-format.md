---
title: XCM 형식
description:
keywords:
  - XCM 지침
  - 레지스터
  - 타입
  - 오류
---

본 문서는 XCM 형식에 대한 정보를 제공합니다.

## 지침

[XCM 통신](./xcm.md)에서 배운 대로, XCM 실행기는 수신 컨센서스 시스템에서 실행되는 가상 머신에서 명령어의 정렬된 집합을 실행하는 프로그램입니다.
일부 명령어는 다른 명령어에 종속적입니다.
`Instruction` enum에서 명령어가 나열된 순서는 이러한 종속성 중 일부를 반영합니다.
예를 들어, 자산은 다른 곳에 예금하기 전에 보유 레지스터에 추가되어야 합니다.
일반적으로 수신 시스템에 보낼 메시지를 구성할 때에도 명령어에 대해 비슷한 순서를 사용합니다.
그러나 편의를 위해 이 참조 섹션에서는 명령어를 정의된 순서가 아닌 알파벳 순서로 나열합니다.

### BuyExecution

현재 메시지의 실행을 위해 보유 레지스터에서 자산을 제거하여 지불합니다.
`fees` 매개변수를 지정하여 보유 레지스터에서 실행 수수료를 지불하기 위해 제거할 자산을 식별해야 합니다.
또한 최대 구매 수수료를 위해 `weight_limit`를 지정할 수 있습니다.
지정한 `weight_limit`가 메시지의 예상 가중치보다 낮으면 XCM 실행기는 `TooExpensive` 오류로 실행을 중지합니다.

| 매개변수      | 설명                                                                                      |
| :------------- | :---------------------------------------------------------------------------------------- |
| `fees`         | 트랜잭션 수수료를 지불하기 위해 보유 레지스터에서 제거할 자산을 지정합니다.                     |
| `weight_limit` | 실행 수수료를 지불하기 위해 구매할 최대 가중치를 지정합니다. 제한을 지정하지 않으면 가중치는 제거할 보유 레지스터에서 지정한 최대치까지 무제한으로 처리됩니다. |

다음 예제는 BuyExecution 명령어의 설정을 보여줍니다:

```text
{
  BuyExecution: {
    fees: {
      id: {
        Concrete: {
          parents: 0
          interior: Here
        }
      }
      fun: {
        Fungible: 1,000,000
      }
    }
    weightLimit: {
      Limited: 1,000,000
    }
  }
}
```

다음 예제는 Rust 프로그램에서 명령어를 사용하는 방법을 보여줍니다:

```rust
BuyExecution { fees, weight_limit } => {
    if let Some(weight) = Option::<u64>::from(weight_limit) {
        // `fees`의 자산을 최대 `weight`까지 사용하여 지불합니다.
        let max_fee =
            self.holding.try_take(fees.into()).map_err(|_| XcmError::NotHoldingFees)?;
        let unspent = self.trader.buy_weight(weight, max_fee)?;
        self.holding.subsume_assets(unspent);
    }
    Ok(())
},
```

### ClaimAsset

원본 레지스터에서 식별된 위치를 대신하여 보유 중인 자산을 생성합니다.
`assets` 매개변수를 지정하여 청구할 자산을 식별해야 합니다.
지정한 자산은 주어진 `ticket`과 함께 원본이 청구할 수 있는 자산과 정확히 일치해야 합니다.
`ticket`은 `MultiLocation` 타입을 사용하여 지정해야 합니다.
자산에 대한 청구 티켓은 청구할 자산을 찾는 데 도움을 주는 추상 식별자입니다.

| 매개변수 | 설명                                        |
| :-------- | :------------------------------------------ |
| `assets`  | 청구할 자산을 지정합니다.                   |
| `ticket`  | 청구할 자산을 식별하는 데 도움을 주는 위치를 지정합니다. |

### ClearError

오류 레지스터를 지웁니다.
이 명령어를 사용하여 오류 레지스터에서 마지막 오류를 수동으로 지울 수 있습니다.

### ClearOrigin

원본 레지스터를 지웁니다.
이 명령어를 사용하여 원래 원본의 권한이 나중에 오는 명령어에게서 가져가지 못하도록 할 수 있습니다.
예를 들어, 신뢰할 수 없는 소스에서 중계되는 명령어가 있는 경우(ReserveAssetDeposited와 같은 경우가 자주 있음), ClearOrigin을 사용하여 원래 원본이 명령어를 실행하는 데 사용되지 않도록 할 수 있습니다.

다음 예제는 기존 메시지에 ReserveAssetDeposited와 ClearOrigin 명령어를 추가하는 방법을 보여줍니다:

```rust
let mut message = vec![ReserveAssetDeposited(assets), ClearOrigin];
message.extend(xcm.0.into_iter());
```

### DepositAsset

보유 레지스터에서 지정한 자산을 제거하고 지정된 `beneficiary`의 소유로 사슬 상에 동등한 자산을 예금합니다.
`assets`를 `MultiAssetFilter` 타입을 사용하여 제거할 자산을 지정해야 합니다.

| 매개변수     | 설명                                                                                                                                                                                                                                                                                                                                                                                 |
| :------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets`      | 보유 레지스터에서 제거할 자산을 지정합니다.                                                                                                                                                                                                                                                                                                                                         |
| `max_assets`  | 보유 레지스터에서 제거할 고유 자산 또는 자산 인스턴스의 최대 수를 지정합니다. 지정한 `assets`와 일치하는 고유 자산 또는 자산 인스턴스의 첫 `max_assets` 개만 제거되며, 표준 자산 순서에 따라 우선순위가 지정됩니다. 추가로 고유 자산 또는 자산 인스턴스가 있는 경우 보유 레지스터에 그대로 남게 됩니다. |
| `beneficiary` | 자산의 새로운 소유자를 지정합니다.                                                                                                                                                                                                                                                                                                                                                   |

다음 예제는 DepositAsset 명령어가 포함된 간단한 메시지를 보여줍니다:

```rust
ParaA::execute_with(|| {
        let message = Xcm(vec![
            WithdrawAsset((Here, send_amount).into()),
            buy_execution((Here, send_amount)),
            DepositAsset { assets: All.into(), max_assets: 1, beneficiary: Parachain(2).into() },
        ]);
        // 출금 및 예금 전송
        assert_ok!(ParachainPalletXcm::send_xcm(Here, Parent, message.clone()));
    });
```

### DepositReserveAsset

보유 레지스터에서 자산을 제거하고 sovereign 계정에 동등한 자산을 예금합니다.
이 명령어는 또한 주권 자산 예금 메시지의 대상에게 후속 메시지를 보냅니다.

| 매개변수     | 설명                                                                                                                                                                                                                                                                                                                                                                                 |
| :------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets`      | 보유 레지스터에서 제거할 자산을 지정합니다.                                                                                                                                                                                                                                                                                                                                         |
| `max_assets`  | 보유 레지스터에서 제거할 고유 자산 또는 자산 인스턴스의 최대 수를 지정합니다. 지정한 `assets`와 일치하는 고유 자산 또는 자산 인스턴스의 첫 `max_assets` 개만 제거되며, 표준 자산 순서에 따라 우선순위가 지정됩니다. 추가로 고유 자산 또는 자산 인스턴스가 있는 경우 보유 레지스터에 그대로 남게 됩니다. |
| `destination` | 자산의 소유자가 될 위치를 지정합니다. 이 위치의 sovereign 계정은 해당 자산을 인출하고 효과를 실행합니다. 일반적으로 한 자산/체인 조합에는 일반적으로 하나의 유효한 위치만 있습니다.                                                                                                                                                                                      |
| `xcm`         | ReserveAssetDeposited 명령어 이후에 `destination` 위치에서 실행할 추가 명령어를 지정합니다.                                                                                                                                                                                                                                                                                          |

### DescendOrigin

현재 원본 레지스터의 값의 컨텍스트 내에서 일부 내부 위치로 원본을 변경합니다.

| 매개변수  | 설명                                                                                   |
| :--------- | :------------------------------------------------------------------------------------- |
| `interior` | 원본 레지스터에 배치할 내부 위치를 지정합니다.                                        |

### ExchangeAsset

보유 레지스터의 자산을 `give` 매개변수로 지정한 금액까지 감소시키고 `receive` 매개변수로 지정한 대체 자산의 최소 금액만큼 보유 레지스터의 자산을 증가시킵니다.

| 매개변수 | 설명                                                                                                                                                                                                                                                                                                                                                                           |
| :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `give`    | 보유 레지스터에서 제거할 자산을 지정합니다.                                                                                                                                                                                                                                                                                                                                 |
| `receive` | 보유 레지스터에서 증가시킬 자산을 지정합니다. `receive` 매개변수에 지정된 펀지블 자산은 표현된 것보다 큰 금액으로 증가시킬 수 있지만, 보유 레지스터에는 지정되지 않은 자산이 증가할 수 없습니다. |

### HrmpChannelAccepted

수신자에 의해 열린 채널 요청이 수락되었음을 알리는 알림 메시지를 전송합니다.
이 알림을 전송한 후에는 릴레이 체인 세션이 변경될 때 채널이 열립니다.
이 메시지는 릴레이 체인에서 직접 발생해야 하며, 릴레이 체인에서 파라체인으로 전송될 것입니다.

| 매개변수    | 설명                                                                                                       |
| :----------- | :--------------------------------------------------------------------------------------------------------- |
| `recipient` | 이전의 채널 개방 요청을 수락한 수신 파라체인을 식별하는 파라체인 식별자를 지정합니다. |

### HrmpChannelClosing

채널 개방 요청을 시작한 송신자가 채널을 닫기로 결정했음을 수신자에게 알리는 메시지를 전송합니다.
이 알림을 전송한 후에는 릴레이 체인 세션이 변경될 때 채널이 닫힙니다.
이 메시지는 릴레이 체인에서 직접 발생해야 하며, 릴레이 체인에서 파라체인으로 전송될 것입니다.

| 매개변수    | 설명                                                                                   |
| :----------- | :------------------------------------------------------------------------------------- |
| `initiator` | 채널 닫기 작업을 시작한 파라체인을 식별하는 파라체인 식별자를 지정합니다.                |
| `sender`    | 닫히는 채널의 송신자 측 파라체인을 식별하는 파라체인 식별자를 지정합니다.                 |
| `recipient` | 닫히는 채널의 수신자 측 파라체인을 식별하는 파라체인 식별자를 지정합니다.                 |

### HrmpNewChannelOpenRequest

파라체인에서 릴레이 체인에게 다른 파라체인과의 통신을 위해 새로운 채널을 개방하기 위한 요청을 전송합니다.
HRMP 전송 프로토콜을 사용하여 전달되는 메시지는 항상 릴레이 체인을 통해 라우팅됩니다.
이 메시지는 릴레이 체인에서 직접 발생해야 하며, 릴레이 체인에서 파라체인으로 전송될 것입니다.

| 매개변수             | 설명                                                                                                      |
| :-------------------- | :-------------------------------------------------------------------------------------------------------- |
| `sender`              | 개방될 채널의 송신자를 지정합니다. 또한 채널 개방의 시작자입니다.                                          |
| `max_message_size`    | 송신자가 제안한 메시지의 최대 크기를 지정합니다.                                                            |
| `max_capacity`        | 채널에 대기할 수 있는 최대 메시지 수를 지정합니다.                                                          |

### InitiateReserveWithdraw

보유 레지스터의 값을 자산으로 감소시키고 WithdrawAsset로 시작하는 XCM 명령어를 예약 위치로 전송합니다.

| 매개변수 | 설명                                                                                                                                                                                                                                                                                                                                                                                     |
| :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets`  | 보유 레지스터에서 제거할 자산을 지정합니다.                                                                                                                                                                                                                                                                                                                                             |
| `reserve` | 모든 지정된 자산에 대한 예약 역할을 하는 유효한 위치를 지정합니다. 이 위치의 sovereign 계정은 해당 자산을 인출하고 효과를 실행합니다. 일반적으로 한 자산/체인 조합에는 일반적으로 하나의 유효한 위치만 있습니다.                                                                                                                                                                                  |
| `xcm`     | 자산이 `reserve` 위치에서 인출된 후에 실행할 추가 명령어를 지정합니다.                                                                                                                                                                                                                                                                                                                    |

### InitiateTeleport

보유 레지스터에서 자산을 제거하고 지정된 대상 위치로 ReceiveTeleportedAsset 명령어로 시작하는 메시지를 전송합니다.

참고: 대상 위치는 모든 자산에 대한 유효한 텔레포트 원본으로 원본을 존중해야 합니다.
그렇지 않으면 자산이 손실될 수 있습니다.

| 매개변수     | 설명                                                                                                                                                                                                                                                                                                                                                                                     |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets`      | 보유 레지스터에서 제거할 자산을 지정합니다.                                                                                                                                                                                                                                                                                                                                             |
| `destination` | 이 위치에서 발생하는 텔레포트를 존중하는 유효한 위치를 지정합니다.                                                                                                                                                                                                                                                                                                                         |
| `xcm`         | ReceiveTeleportedAsset 명령어 이후에 `destination` 위치에서 실행할 추가 명령어를 지정합니다.                                                                                                                                                                                                                                                                                             |

### QueryHolding

자산 값이 보유 내용 또는 일부 내용과 동일한 QueryResponse 메시지를 전송합니다.

| 매개변수             | 설명                                                                                                                                           |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| `query_id`            | QueryResponse 메시지의 `query_id` 필드에 사용되는 쿼리의 식별자를 지정합니다.                                                                     |
| `destination`         | QueryResponse 메시지를 전송할 위치를 지정합니다.                                                                                                 |
| `assets`              | 보고할 자산에 대한 필터를 지정합니다.                                                                                                          |
| `max_response_weight` | QueryResponse 메시지의 `max_weight` 필드에 사용할 값을 지정합니다.                                                                               |

다음 예제는 QueryHolding 명령어를 보여줍니다. 이 명령어는 식별자 `query_id_set`을 가진 QueryResponse를 반환합니다:

```rust
ParaA::execute_with(|| {
	let message = Xcm(vec![
			QueryHolding {
				query_id: query_id_set,
				dest: Parachain(1).into(),
				assets: All.into(),
				max_response_weight: 1_000_000_000,
			},
	]);
});
```

### QueryResponse

원본으로부터 예상되는 정보를 제공합니다.

| 매개변수  | 설명                                                                                                  |
| :--------- | :---------------------------------------------------------------------------------------------------- |
| `query_id` | 이 메시지를 보내는 쿼리의 식별자를 지정합니다.                                                        |

| `response`: 쿼리 명령으로부터 발생한 메시지의 내용을 표현합니다.
| `max_weight` | 이 응답을 처리하는 데 사용할 최대 가중치를 지정합니다. 지정한 가중치보다 더 많은 가중치가 필요한 경우 오류가 반환됩니다. 지정한 가중치보다 적게 필요한 경우 실행 시점에 나머지 가중치가 초과 가중치 레지스터에 추가될 수 있습니다.

`Response` 타입은 `QueryResponse` XCM 명령어에서 정보 내용을 표현하는 데 사용됩니다.
쿼리에 따라 다음과 같은 다른 데이터 유형을 나타낼 수 있습니다:

- Null
- Assets { assets: MultiAssets }
- ExecutionResult { result: Result<(), (u32, Error)> }
- Version { version: Compact }

다음 예제는 `QueryResponse` 메시지가 수신되었는지 확인하고 응답의 정보가 새로 예금된 자산인지 확인하는 방법을 보여줍니다:

```rust
ParaA::execute_with(|| {
	assert_eq!(
			parachain::MsgQueue::received_dmp(),
			vec![Xcm(vec![QueryResponse {
					query_id: query_id_set,
					response: Response::Assets(MultiAssets::new()),
					max_weight: 1_000_000_000,
			}])],
	);
});
```

### ReceiveTeleportedAsset

현재 원본에서 제거된 자산과 동등한 자산을 보유 레지스터에 누적합니다.
원본은 이 메시지를 보내는 결과로 자산을 제거했음을 신뢰해야 합니다.

| 매개 변수 | 설명                                                         |
| :-------- | :----------------------------------------------------------- |
| `assets`  | 원본에서 제거된 자산을 지정합니다.                             |

### RefundSurplus

환불된 가중치 레지스터를 초과 가중치 레지스터의 값으로 증가시킵니다.
이 명령은 이전에 BuyExecution 명령을 사용하여 지불한 수수료를 환불된 가중치 레지스터에 추가된 금액과 일치하도록 홀딩 레지스터로 이동시킵니다.

### ReportError

오류 레지스터의 내용을 지정된 `destination`에 XCM을 사용하여 보고합니다.
XCM의 결과와 함께 `destination`에 지정된 `query_id` 및 QueryResponse 메시지의 유형인 ExecutionOutcome의 메시지가 전송됩니다.

| 매개 변수             | 설명                                                                                   |
| :-------------------- | :------------------------------------------------------------------------------------- |
| `query_id`            | QueryResponse 메시지의 `query_id` 필드에 사용할 값입니다.                              |
| `destination`         | QueryResponse 메시지를 전송할 위치를 지정합니다.                                       |
| `max_response_weight` | QueryResponse 메시지의 `max_weight` 필드에 사용할 값입니다.                            |

### ReserveAssetDeposited

원본 레지스터의 값에서 받은 자산을 나타내기 위해 홀딩 레지스터에 파생 자산을 추가합니다.
원본은 자산의 예비로 작동할 수 있도록 신뢰할 수 있어야 합니다.

| 매개 변수 | 설명                                                                                                        |
| :-------- | :---------------------------------------------------------------------------------------------------------- |
| `assets`  | 로컬 합의 시스템의 sovereign 계정에 수신된 자산을 지정합니다.                                                     |

### SetAppendix

부록 레지스터를 설정합니다.
부록 레지스터는 현재 프로그램이 실행을 완료한 후에 실행되어야 하는 코드를 제공합니다.
현재 프로그램이 성공적으로 종료되거나 오류가 발생한 경우 오류 처리기가 비어 있는 경우 부록 레지스터가 지워지고 그 내용이 프로그램 레지스터를 대체하는 데 사용됩니다.
이 명령의 예상 가중치에는 `appendix` 코드의 예상 가중치가 포함되어야 합니다.
실행 시에 부록의 예상 가중치가 변경되기 전에 초과 가중치 레지스터의 예상 가중치가 증가해야 합니다.

| 매개 변수  | 설명                                               |
| :--------- | :------------------------------------------------ |
| `appendix` | Appendix 레지스터를 설정할 값입니다. (Xcm 형식) |

### SetErrorHandler

오류 처리기 레지스터를 설정합니다.
프로그램이 오류를 만나면 이 레지스터가 지워지고 그 내용이 프로그램 레지스터를 대체하는 데 사용됩니다.
이 명령의 예상 가중치에는 `error_handler` 코드의 예상 가중치가 포함되어야 합니다.
실행 시에 초과 가중치 레지스터의 예상 가중치가 변경되기 전에 오류 처리기의 예상 가중치가 증가해야 합니다.

| 매개 변수       | 설명                                                       |
| :-------------- | :-------------------------------------------------------- |
| `error_handler` | 오류 처리기 레지스터에 설정할 값입니다. |

### SubscribeVersion

응답 필드에 XCM 버전을 지정하는 QueryResponse 메시지를 Origin에 전송합니다.
로컬 합의에 대한 업그레이드로 인해 XCM의 최신 버전이 지원되는 경우 유사한 응답이 나와야 합니다.

| 매개 변수             | 설명                                                                                   |
| :-------------------- | :------------------------------------------------------------------------------------- |
| `query_id`            | QueryResponse 메시지의 `query_id` 필드에 사용할 값입니다.                              |
| `max_response_weight` | QueryResponse 메시지의 `max_weight` 필드에 사용할 값입니다.                            |

### Transact

인코딩된 호출을 `origin_type` 매개 변수로 지정한 컨텍스트를 사용하여 디스패치 원본으로 사용하여 전송합니다.

| 매개 변수     | 설명                                                                                                                                                                                                                                                                                                                                                           |
| :------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `origin_type` | 메시지 원본을 디스패치 원본으로 표현하기 위한 컨텍스트를 지정합니다.                                                                                                                                                                                                                                                                                             |
| `max_weight`  | 인코딩된 호출을 디스패치하는 동안 사용할 최대 가중치를 지정합니다. 지정한 가중치보다 더 많은 가중치가 필요한 경우 실행이 중지되고 오류가 반환됩니다. 지정한 가중치보다 적은 가중치가 필요한 경우 실행 시에 초과 가중치 레지스터에 차이가 추가될 수 있습니다.                                                                                                                  |
| `call`        | 수신 시스템에서 실행할 인코딩된 트랜잭션을 지정합니다.                                                                                                                                                                                                                                                                                                             |

### TransferAsset

원본의 소유권에서 자산을 인출하고 수취인의 소유권 아래에 동등한 자산을 예금합니다.

| 매개 변수     | 설명                                     |
| :------------ | :--------------------------------------- |
| `assets`      | 이전할 자산을 지정합니다.                 |
| `beneficiary` | 자산의 새 소유자를 지정합니다.            |

### TransferReserveAsset

현재 원본의 소유권에서 자산을 인출하고 대상의 sovereign 계정이 자산을 소유하고 자산의 효과적인 수혜자 및 예비 자산 예금 메시지의 알림 대상이 되도록 동등한 자산을 예금합니다.
이 명령은 ReserveAssetDeposited와 `xcm` 매개 변수로 지정된 모든 명령을 지정된 대상으로 추가 메시지를 보냅니다.

| 매개 변수     | 설명                                                                                                                                                                                    |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets`      | 이전할 자산을 지정합니다.                                                                                                                                                                 |
| `destination` | 자산을 소유할 대상의 위치를 지정합니다. 이는 자산의 효과적인 수혜자 및 예비 자산 예금 메시지의 알림 대상이 됩니다.                                                                      |
| `xcm`         | ReserveAssetDeposited 명령을 따르는 명령을 지정합니다. 이 명령은 대상으로 전달되는 ReserveAssetDeposited 명령과 `xcm` 매개 변수로 지정된 명령과 함께 추가 메시지를 보냅니다. |

다음 예제는 두 개의 추가 명령을 포함하는 메시지를 포함하는 TransferReserveAsset 명령을 사용하는 방법을 보여줍니다.

```rust
let mut message = Xcm(vec![TransferReserveAsset {
    assets,
    dest,
    xcm: Xcm(vec![
        BuyExecution { fees, weight_limit: Unlimited },
        DepositAsset { assets: Wild(All), max_assets, beneficiary },
    ]),
}]);
```

### Trap

Trap 유형의 오류를 발생시킵니다.

| 매개 변수 | 설명                                                               |
| :-------- | :---------------------------------------------------------------- |
| `id`      | 발생한 오류의 매개 변수로 사용할 값을 지정합니다.                  |

### UnsubscribeVersion

이전 SubscribeVersion 명령의 효과를 취소합니다.

### WithdrawAsset

로컬 합의 시스템의 송신 계정에서 지정된 자산을 제거하고 홀딩 레지스터의 값에 추가합니다.

| 매개 변수 | 설명                                                                                          |
| :-------- | :-------------------------------------------------------------------------------------------- |
| `assets`  | 송신자에서 제거할 자산을 지정합니다. 자산은 원본 레지스터의 계정이 소유해야 합니다. |

## 레지스터

대부분의 XCVM 레지스터는 직접 수정할 수 없습니다.
레지스터는 특정 값으로 설정되어 시작되며 특정 명령, 특정 상황 또는 특정 규칙에 따라만 조작될 수 있습니다.
XCVM에는 다음과 같은 레지스터가 포함되어 있습니다.

| 레지스터           | 설명                                                                                           |
| :---------------- | :--------------------------------------------------------------------------------------------- |
| Origin            | 현재 프로그램이 실행 중인 권한의 위치를 저장합니다.                                             |
| Holding           | XCVM의 제어 하에 있는데 체인에 표시되지 않는 자산의 수를 저장합니다.                            |
| Surplus weight    | 이전에 계산된 가중치의 과대 추정을 저장합니다.                                                   |
| Refunded weight   | 환불된 가중치의 일부를 저장합니다.                                                               |
| Programme         | 현재 실행 중인 XCM 명령 세트를 저장합니다. 이 레지스터는 크로스-컨센서스 가상 머신에서 프로그램을 보유합니다. |
| Programme counter | 현재 실행 중인 명령의 인덱스를 저장합니다. 값은 각 성공적으로 실행된 명령의 끝에서 1씩 증가합니다. 프로그램 레지스터가 변경될 때마다 레지스터는 0으로 재설정됩니다. |
| Error             | 프로그램 실행 중 발생한 마지막 알려진 오류에 대한 정보를 저장합니다.                              |
| Error handler     | 프로그램에 오류가 발생한 경우 실행될 코드를 저장합니다.                                          |
| Appendix          | 현재 프로그램이 종료된 후 실행될 코드를 저장합니다.                                             |

## 원본

일부 경우에는 XCM 명령을 실행하는 데 사용되는 원본을 조작하여 특정 작업을 수행하거나 특정 XCM 명령을 실행하는 데 필요한 권한을 더 많이 또는 더 적게 부여할 수 있습니다.
다음 원본 유형을 사용하여 XCM 명령의 원본이 해석되는 방식을 조작할 수 있습니다.

| 원본 유형      | 설명                                                                                                                                           |
| :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Native         | 로컬 런타임 프레임워크에서 발신자에 대한 기본 디스패치 원본 표현을 사용합니다. 대부분의 체인에서 이는 체인에서 온 경우 `Parachain` 또는 `Relay` 원본입니다. |
| SovereignAccount | 발신자의 sovereign 계정을 사용합니다. 대부분의 체인에서 이는 `Signed` 원본입니다.                                                                       |
| Superuser      | 슈퍼유저 계정을 사용합니다. 일반 계정은 이 원본을 사용할 수 없으며 많은 경우 어떤 계정도 사용할 수 없습니다. 대부분의 체인에서 이는 `Root` 원본입니다.   |
| Xcm            | XCM 원본 및 디스패치 원본의 `MultiLocation`을 직접 인코딩하여 사용합니다. 대부분의 체인에서 이는 `pallet_xcm::Origin::Xcm` 유형입니다.               |

다른 원본을 사용하는 것의 영향은 호출하는 코드에 따라 다릅니다.
체인 로직 및 권한에 따라 원하는 원본을 생성할 수 없을 수 있습니다.
예를 들어 대부분의 사용자는 슈퍼유저 원본을 생성할 수 없지만 체인 로직에서는 가능합니다.

다음 예제는 Transact 명령에서 원본 유형을 지정하는 방법을 보여줍니다.

```text
Transact {
	origin_type: OriginKind::SovereignAccount,
	require_weight_at_most: weight,
	call: call.encode().into(),
},
```

원본 변환에 대한 추가 예제는 [origin_conversion](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/xcm/xcm-builder/src/origin_conversion.rs) 모듈을 참조하십시오.

## 오류

XCM 명령을 실행하는 프로그램이 문제를 만나면 실행이 중지되고 다음 오류 중 하나로 문제의 유형을 식별합니다:

| 오류 유형                       | 설명                                                                                                                                                             |
| :------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Overflow = 0`                   | 명령이 산술 오버플로우를 발생시켰습니다.                                                                                                                           |
| `Unimplemented = 1`              | 해당 명령은 일부러 지원되지 않습니다.                                                                                                                           |
| `UntrustedReserveLocation = 2`   | 원본 레지스터에 예약 전송 알림에 대한 유효한 값이 포함되어 있지 않습니다.                                                                                  |
| `UntrustedTeleportLocation = 3`  | 원본 레지스터에 텔레포트 알림에 대한 유효한 값이 포함되어 있지 않습니다.                                                                                          |
| `MultiLocationFull = 4`          | `MultiLocation` 값이 더 이상 하위로 내려갈 수 없을 만큼 큽니다.                                                                                                                 |
| `MultiLocationNotInvertible = 5` | `MultiLocation` 값이 로컬 위치의 알려진 조상 수보다 더 많은 부모로 올라갑니다.                                                                |
| `BadOrigin = 6`                  | 원본 레지스터에 명령을 실행하기 위한 유효한 값이 포함되어 있지 않습니다.                                                                                        |
| `InvalidLocation = 7`            | 위치 매개변수가 명령에 대한 유효한 값이 아닙니다.                                                                                                        |
| `AssetNotFound = 8`              | 지정된 자산을 찾을 수 없거나 명령에 지정된 위치에서 유효하지 않습니다.                                                                        |
| `FailedToTransactAsset = 9`      | 자산 거래(예: 자산 인출 또는 입금 명령)가 실패했습니다. 대부분의 경우, 이러한 유형의 오류는 유형 변환 문제로 인해 발생합니다. |
| `NotWithdrawable = 10`           | 지정된 자산을 인출할 수 없습니다. 소유권, 자산 가용성 또는 권한 부족 등의 이유로 인할 수 있습니다.                                                       |
| `LocationCannotHold = 11`        | 특정 위치의 소유권 아래에 자산을 예금할 수 없습니다.                                                                                    |
| `ExceedsMaxMessageSize = 12`     | 전송 프로토콜에서 지원하는 최대 메시지 크기를 초과하려고 시도했습니다.                                                      |
| `DestinationUnsupported = 13`    | 전송 대상에서 지원되는 형식으로 변환할 수 없는 메시지를 전송하려고 시도했습니다.                                                             |
| `Transport = 14`                 | 대상은 경로 지정 가능하지만 전송 메커니즘에 문제가 있습니다.                                                                                         |
| `Unroutable = 15`                | 대상이 경로 지정할 수 없음을 알려져 있습니다.                                                                                                                              |
| `UnknownClaim = 16`              | ClaimAsset 명령에 지정된 클레임이 인식되지 않거나 찾을 수 없습니다.                                                                                  |
| `FailedToDecode = 17`            | Transact 명령에 지정된 함수를 디코딩할 수 없습니다.                                                                                                   |
| `TooMuchWeightRequired = 18`     | Transact 명령에 지정된 함수가 허용된 가중치 제한을 초과할 수 있습니다.                                                                              |
| `NotHoldingFees = 19`            | BuyExecution 명령에서 사용할 수 있는 지불 가능한 수수료가 보유 레지스터에 포함되어 있지 않습니다.                                                                          |
| `TooExpensive = 20`              | BuyExecution 명령에서 가중치를 구매하기 위해 선언된 수수료가 충분하지 않습니다.                                                                                  |
| `Trap(u64) = 21`                 | Trap 명령에 의해 의도적으로 오류가 발생합니다. 코드가 포함됩니다.                                                                                     |
| `ExpectationFalse = 22`          | ExpectAsset, ExpectError 및 ExpectOrigin에서 기대값이 참이 아닌 경우 사용됩니다.                                                                                     |

구체적인 식별자는 명령이 실행되는 맥락에 상대적으로 합의 시스템 내에서 자산을 고유하게 식별합니다.