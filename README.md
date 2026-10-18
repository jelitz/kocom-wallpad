[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge)](https://github.com/hacs/integration)

# 코콤 월패드 Home Assistant 통합 (개인 커스텀 포크)

[ninthsword/kocom-wallpad](https://github.com/ninthsword/kocom-wallpad)를 코콤 KHN-V100JA 월패드와 EW11 환경에 맞춰 고쳐 쓰는 개인 포크입니다. 문제가 생기거나 필요한 기능이 생기면 월패드 버스를 캡처해 동작을 확인하고 고칩니다. 바꾼 내용은 모두 KHN-V100JA 한 대에서만 확인했습니다.

같은 월패드를 쓰거나 비슷한 문제를 겪고 있다면 이 포크를 설치하거나 필요한 커밋만 가져다 쓸 수 있습니다. 다른 코콤 월패드라면 원본을 쓰고, 아래 "바꾼 것"과 "KHN-V100JA에서 확인한 것"은 참고만 하세요.

## 따로 고쳐 쓰는 이유

원본 통합을 KHN-V100JA에 붙이자 켜진 조명이 꺼짐으로 보였습니다. 조명을 한 번 누르면 켜짐과 꺼짐이 반복됐고, HA를 재시작하면 한동안 조명과 엘리베이터 호출을 쓸 수 없었습니다. 이 월패드가 원본이 가정한 것과 다른 값을 보내고, 조명 명령 뒤에 별도 해제 패킷을 기다렸기 때문입니다.

서버에서 파일을 직접 고치면 HACS 업데이트 때 덮어써집니다. 그래서 고친 내용을 커밋으로 남기고 HACS가 이 레포에서 받아 가도록 포크를 만들었습니다.

## 바꾼 것

| 문제 | 바꾼 내용 |
|---|---|
| 조명 상태 값이 0xFF가 아니면 꺼짐으로 보임 | 0이 아니면 켜짐으로 판단 |
| 일괄소등(0EFF)이 일반 조명처럼 조작됨 | 일괄소등 상태를 따로 두고 읽기 전용으로 표시 |
| 조명 명령 재시도 때문에 켜짐·꺼짐이 반복됨 | 조명 명령은 재시도 없이 한 번만 보냄 |
| 조명을 켜고 끈 뒤 월패드가 일괄소등 해제 패킷을 기다림 | 조명 명령 1초 뒤 해제 패킷(0EFF 66)을 보냄. 일괄소등이 켜져 있으면 보내지 않음 |
| HA 재시작 직후 조명·엘리베이터 호출·귀가 감지가 사용 불가로 표시됨 | EW11 연결만 살아 있으면 사용 가능으로 표시 |
| 재시작 뒤 조명·환기·가스 상태를 모름 | 연결할 때 조명 방·환기·가스밸브에 상태 조회(0x3A)를 보냄 |
| 환기 CO2 값이 몇 시간씩 갱신되지 않음 | 환기 상태를 5분마다 다시 조회 |
| 환기 프리셋 이름이 영어로 보임 | 한국어 번역 추가 |
| 0x60 신호가 움직임 감지기로 잡히고, 한 번 켜지면 꺼지지 않음 | 귀가 감지 센서로 표시하고, 04 01은 켜짐, 2초쯤 뒤 오는 04 00은 꺼짐으로 처리. 재시작할 때는 꺼짐으로 시작 |

## KHN-V100JA에서 확인한 것

KHN-V100JA 한 대에서 버스를 캡처해 확인한 내용입니다. 다른 월패드에서는 다를 수 있습니다.

- 0x60 신호는 현관문이 열린 뒤 거실 재실센서가 움직임을 잡을 때만 옵니다. 문만 여닫거나 거실에서만 움직이면 아무 신호도 없습니다. 그래서 현관문 열림과 거실 재실을 따로 받을 수는 없고, 둘이 함께 일어난 귀가만 받을 수 있습니다.
- 엘리베이터는 호출만 됩니다. 버스에 방향·층수 정보가 오지 않습니다.
- 실내 공기질(0x98) 기기는 없습니다. CO2 값은 환기 응답에 들어 있습니다.
- 환기 프리셋은 버스에서 한 번이라도 본 모드만 선택지에 나옵니다. 이 월패드에서는 환기·자동·바이패스 세 가지입니다.
- 현관 도어락 문열림은 버스 패킷을 다시 보내는 방법으로는 열리지 않았습니다. 월패드에서 문을 열 때 버스에 0x33 신호가 보이지만, 실제로 문을 여는 신호는 다른 선으로 나가는 것으로 보입니다.

## 설치

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=jelitz&repository=kocom-wallpad&category=Integration)

1. HACS 사용자 저장소에 `https://github.com/jelitz/kocom-wallpad`를 통합(Integration)으로 추가하거나 위 배지를 누릅니다.
2. 내려받은 뒤 HA를 재시작합니다.
3. **설정 > 기기 및 서비스 > 통합구성요소 추가하기**에서 `코콤 월패드`를 검색해 추가합니다.
4. 호스트에 EW11의 IP 주소, 포트에 EW11 포트(기본값 8899)를 넣습니다.

원본 ninthsword 포크를 이미 쓰고 있다면, HACS 사용자 저장소에서 원본을 빼고 이 레포를 추가해 내려받은 뒤 HA를 재시작하면 됩니다. 통합 설정과 엔티티는 그대로 남습니다.

HA 2026.8.0 이상이 필요합니다. 이 포크는 HA 2026.10.2(Docker)에서 쓰고 있습니다.

## 지원 기기

| 기기 | 원본 지원 | KHN-V100JA에서 확인 |
|---|---|---|
| 조명 | O | O |
| 일괄소등 | O | O (읽기 전용) |
| 난방 | O | O |
| 환기 | O | O |
| 가스밸브 | O (잠금만) | O |
| 엘리베이터 | O | 호출만 |
| 귀가 감지 (0x60) | O (모션으로 표시) | O |
| 실내 공기질 | O | 없음 |
| 조명 디밍, 콘센트, 에어컨 | O | 확인 안 함 |
| 인터폰 | X | X |

처음 추가한 기기는 한 번 켜거나 꺼야 HA에 등록될 수 있습니다.

## 문제가 생기면

`configuration.yaml`에 아래 내용을 넣고 HA를 재시작하면 패킷 로그를 볼 수 있습니다. 확인이 끝나면 지웁니다.

```yaml
logger:
  default: info
  logs:
    custom_components.kocom_wallpad: debug
```

이 포크에서 생긴 문제는 [Issues](https://github.com/jelitz/kocom-wallpad/issues)에 남겨 주세요. 이 통합을 쓰다 생기는 문제는 책임지지 않습니다.

## 개발과 테스트

일괄소등 해제 패킷, 연결 중 사용 가능 표시, 연결 시 상태 조회, 귀가 감지는 `tests/`에 테스트가 있습니다. 귀가 감지 테스트에는 실제로 캡처한 패킷을 씁니다. 코드를 고친 뒤에는 아래 두 가지를 돌립니다.

```sh
python -B -m unittest discover -s tests
ruff check custom_components tests
```

원본에 있던 HA 버전별 검증 절차(Pyright, 2026.9·2026.10 호환성 검사)는 [docs/development.md](docs/development.md)로 옮겼습니다.

## 출처와 라이선스

- 원작: [lunDreame/kocom-wallpad](https://github.com/lunDreame/kocom-wallpad). 지금은 보관(archive) 상태이고 라이선스 파일이 내려갔습니다.
- 이 포크의 기반: [ninthsword/kocom-wallpad](https://github.com/ninthsword/kocom-wallpad). MIT 라이선스가 그대로 남아 있습니다.
- 라이선스: MIT. 원저작권 표기(Copyright (c) 2024 lunDreame)와 [LICENSE](./LICENSE) 파일은 그대로 둡니다.
