# 07. 신뢰할 수 없는 코드를 쓰면서 불변성 지키기

- [방어적 복사를 사용해 레거시 코드에서 불변성 지키기](#방어적-복사를-사용해-레거시-코드에서-불변성-지키기)
- [일반적으로 쓰이는 방어적 복사](#일반적으로-쓰이는-방어적-복사)
- [카피-온-라이트와 방어적 복사 비교](#카피-온-라이트와-방어적-복사-비교)
  - [얕은 복사와 깊은 복사 비교](#얕은-복사와-깊은-복사-비교)

# 방어적 복사를 사용해 레거시 코드에서 불변성 지키기

- 방어적 복사

  - 동작 방식: 들어오고 나가는 데이터의 깊은 복사본 만듦
    - 들어오는 데이터 보호: 신뢰할 수 없는 들어온 데이터로 깊은 복사본 만듦, 변경 가능한 원본은 버림. 신뢰할 수 있는 코드만 복사본 사용
    - 나가는 데이터 보호: 깊은 복사본을 만들어 데이터 내보냄. 신뢰할 수 없는 코드가 나가는 데이터 값을 변경 못하도록.
  - 목적: 안전지대에 불변성 유지. 가변성 있는 데이터가 안전지대로 들어오지 못하도록 하는 것
  - 규칙
    1. 데이터가 안전한 코드에서 나갈 때 복사하기
       1. 불변성 데이터를 위한 깊은 복사본 만듦
       2. 신뢰할 수 없는 코드로 복사본 전달
    2. 안전한 코드로 데이터가 들어올 때 복사하기
       1. 변경될 수도 있는 데이터가 들어오면 바로 깊은 복사본 만들어 안전한 코드로 전달
       2. 복사본을 안전한 코드에서 사용
  - 구현

    - 기존 코드
      ```tsx
      function add_item_to_cart(name, price) {
        shopping_cart = add_item(shopping_cart, name, price);
        var total = calc_total(shopping_cart);
        set_cart_total_dom(total);
        update_shipping_icons(shopping_cart);
        update_tax_dom(total);
        // 명세 추가
        black_friday_promotion(shopping_cart);
      }
      ```
    - 방어적 복사 적용
      ```tsx
      function add_item_to_cart(name, price) {
        // ...
        // 넘기기 전 복사
        var cart_copy = deepCopy(shopping_cart);
        black_friday_promotion(cart_copy);
        // 들어오는 데이터를 위한 복사
        shopping_cart = deepCopy(cart_copy);
      }
      ```
    - 방어적 복사 코드 분리해 신뢰할 수 없는 코드 감싸기

      ```tsx
      function add_item_to_cart(name, price) {
        // ...
        shopping_cart = black_friday_promotion_safe(shopping_cart);
      }

      function black_friday_promotion_safe(cart) {
        var cart_copy = deepCopy(cart);
        black_friday_promotion(cart_copy);
        return deepCopy(cart_copy);
      }
      ```

      - 분리 이유
        - 나중에 코드 보면 복사본 만든 이유 모를 수 있음
        - black_friday_promotion() 코드를 나중에 쓸 수도 있음
        - 코드가 하는 일 명확하게 만들기 위함

# 일반적으로 쓰이는 방어적 복사

- 웹 API 속 방어적 복사
  - API에 요청으로 들어오고 응답으로 보내는 JSON 데이터는 깊은 복사본
  - 방어적 복사로 서로 다른 코드와 원칙을 가진 서비스들이 문제 없이 통신 가능
  - 비공유 아키텍처: shared nothing architecture. 모듈이 서로 통신하기 위해 방어적 복사 공유함. 모듈이 어떤 데이터의 참조도 공유하지 않음.
- 함수형 프로그래밍 언어인 얼랭과 엘릭서에서의 방어적 복사
  - 두 프로세스가 서로 메시지 주고받을 때 데이터 복사. 고가용성 보장
- 방어적 복사 예제

  ```tsx
  function payrollCalc(employees) {
    // ...
    return payrollChecks;
  }

  function payrollCalcSafe(employees) {
    const copied_employees = deepCopy(employees);
    const payrollChecks = payrollCalc(copied_employees);
    return deepCopy(payrollChecks);
  }
  ```

  ```tsx
  userChanges.subscribe(function (user) {
    const userCopy = deepCopy(user);
    // 안전지대에서 데이터가 나가지 않음 -> 데이터 복사할 필요 없음
    procssUser(userCopy);
  });
  ```

# 카피-온-라이트와 방어적 복사 비교

|                                            | 카피-온-라이트                          | 방어적 복사                              |
| ------------------------------------------ | --------------------------------------- | ---------------------------------------- |
| 사용 시점                                  | 통제 가능한 데이터 변경                 | 신뢰할 수 없는 코드와 데이터 주고받기    |
| 사용 장소                                  | 안전지대 안 어디서나<br/> 카피-온-라이트가 불변성 가진 안전지대 만듦 | 안전지대의 경계에서 데이터가 오고 갈 때 |
| 복사 방식                                  | 얕은 복사 (비용 저렴)                   | 깊은 복사 (비용 비쌈)                    |
| 복사 시점                                  | 데이터가 바뀔 때마다                    | 안전지대에서 데이터가 들어오고 나갈 때만 |
| 규칙                                       | 1. 바꿀 데이터의 얕은 복사 만듦      <br/>2. 복사본 변경<br/>3. 복사본 리턴 | 1. 안전지대로 들어오는 데이터에 깊은 복사 만듦<br/>2. 안전지대에서 나가는 데이터에 깊은 복사 만듦 |

## 얕은 복사와 깊은 복사 비교

|                           | 얕은 복사                                                          | 깊은 복사                                                                                 |
| ------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| 원본과의 데이터 구조 공유 | O                                                                  | X. 어떠한 데이터 구조도 공유하지 않음 (→ 신뢰할 수 없는 코드로부터 원본 데이터 보호 가능) |
| 복사 대상                 | 바뀐 부분만 복사. 바뀌지 않은 값이면 원본과 복사본이 데이터 공유함 | 모든 것 복사. 중첩된 모든 객체 배열 다 복사.                                              |
| 구현 난이도               | 비교적 용이                                                        | 어려움 → lodash 라이브러리의 .cloneDeep() 사용 권장                                       |

→ 카피-온-라이트를 쓸 수 없는 곳에만 방어적 복사를 사용하는 게 좋음
