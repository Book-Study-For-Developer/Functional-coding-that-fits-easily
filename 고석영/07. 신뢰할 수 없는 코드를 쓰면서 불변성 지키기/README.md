## 레거시 코드와 불변성

- 상황: 블랙 프라이데이 행사를 위한 코드 추가
- 문제: 블랙 프라이데이 행사 코드는 많은 곳에서 장바구니 데이터를 변경하고 있음

```ts
const add_item_to_cart = (name, price) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)
  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)
  black_friday_promotion(shopping_cart) // 블랙 프라이데이 행사를 위한 코드
}
```

- 추가된 함수를 호출하면 카피-온-라이트 원칙을 지킬 수 없는 상태
- `black_friday_promotion()` 함수 또한 고칠 수 없는 상태
- **방어적 복사**를 통해 카피-온-라이트 원칙을 지키면서 안전하게 함수를 사용할 수 있음!

## 우리가 만든 카피-온-라이트 코드는 신뢰할 수 없는 코드와 상호작용해야 합니다

- 불변성이 지켜지는 안전지대(safe zone)
- 블랙 프라이데이 행사 함수는 안전지대 밖에 있지만, 요구 사항을 구현하려면 우리가 만든 코드에서 안전하지 않은 함수를 사용해야 함
- 따라서 블랙 프라이데이 함수의 입력과 출력을 통해 안전지대에 있는 코드와 데이터를 주고받아야 함
- 안전지대 밖으로 나가는 데이터는 잠재적으로 바뀔 수 있음 -> 신뢰할 수 없는 코드가 데이터를 변경할 수 있는 가능성 존재
- 마찬가지로 신뢰할 수 없는 코드에서 안전지대로 들어오는 데이터 역시 잠재적으로 변경 가능성 존재
- 문제는 불변성을 지키면서 데이터를 주고받는 방법을 찾아야 함

- 카피-온-라이트 패턴은 데이터를 바꾸기 전에 복사
- 무엇이 바뀌는지 알기 때문에 무엇을 복사해야 할지 예상할 수 있음
- 어떤 일이 일어날지 정확히 알지 못할 때 데이터가 바뀌는 것을 완벽히 막아주는 원칙이 필요 -> **방어적 복사**

## 방어적 복사는 원본이 바뀌는 것을 막아줍니다

> 들어오고 나가는 데이터의 복사본을 만드는 것이 방어적 복사가 동작하는 방식

## 바뀔 수도 있는 데이터가 신뢰할 수 없는 코드에서 안전지대로 들어오는 경우

- 들어온 데이터로 깊은 복사를 만들고 변경 가능한 원본은 버린다.
- 신뢰할 수 있는 코드만 복사본을 쓰기 때문에 데이터 변경 x

## 안전지대에서 나가는 데이터

- 나가는 데이터에 깊은 복사
- 신뢰할 수 없는 코드가 값을 변경할 가능성 차단

## 방어적 복사의 목적

- 안전지대의 불변성 유지
- 바뀔 수도 있는 데이터가 안전지대로 들어올 가능성 차단

## 방어적 복사 구현하기

> 방어적 복사를 사용하면 데이터가 바뀌는 것을 막아 불변성을 지킬 수 있습니다. 원본이 바뀌지 않도록 막아주기 때문입니다.

```ts
const add_item_to_cart = ({ name, price }: Item) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)
  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)

  const cart_copy = deepCopy(shopping_cart) // 데이터를 전달하기 전에 복사
  black_friday_promotion(cart_copy)
  shopping_cart = deepCopy(cart_copy) // 데이터를 전달한 후에 복사
}
```

## 방어적 복사 규칙

- 방어적 복사는 데이터를 변경할 수도 있는 코드와 불변성 코드 사이에 데이터를 주고받기 위한 원칙
- 이러한 규칙을 따르면 불변성 원칙을 지키면서 신뢰할 수 없는 코드와 상호작용 가능

**규칙 1: 데이터가 안전한 코드에서 나갈 때 복사하기**

1. 불변성 데이터를 위한 깊은 복사본 생성
2. 신뢰할 수 없는 코드로 복사본 전달

**규칙 2: 안전한 코드로 데이터가 들어올 때 복사하기**

1. 변경될 수도 있는 데이터가 들어오면 바로 깊은 복사본 생성해 안전한 코드로 전달
2. 복사본을 안전한 코드에서 사용

### 신뢰할 수 없는 코드 감싸기

- 재사용 및 코드 파악 용이를 위해 방어적 복사를 분리해 새로운 함수 만들기
- 이렇게 하면 데이터가 바뀌지 않고 코드가 하는 일이 명확해짐

```ts
const black_friday_promotion_safe = cart => {
  const cart_copy = deepCopy(cart)
  black_friday_promotion(cart_copy)
  return deepCopy(cart_copy)
}
```

- 방어적 복사를 분리해 만든 함수 사용하기

```ts
const add_item_to_cart = ({ name, price }: Item) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)
  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)

  shopping_cart = black_friday_promotion_safe(shopping_cart)
}
```

## 방어적 복사가 익숙할 수도 있습니다

### 웹 API속에 방어적 복사

- 대부분의 웹 기반 API는 암묵적으로 방어적 복사
- JSON 데이터가 API에 요청으로 들어오면 클라이언트는 데이터를 인터넷을 통해 API로 보내려고 직렬화함 → JSON 데이터는 **깊은 복사본**
- 서비스가 잘 동작하면 JSON으로 응답 → 이때도 역시 **깊은 복사본**
- 서비스에 들어올 때와 나갈 때 데이터를 복사한 것
- 마이크로서비스나 서비스 지향 시스템이 서로 통신할 때 방어적 복사를 한다는 점은 장점
- 방어적 복사로 서로 다른 코드와 원칙을 가진 서비스들이 문제없이 통신할 수 있음

### 얼랭과 엘릭서에서 방어적 복사

- 방어적 복사는 얼랭 시스템이 고가용성을 보장하는 핵심 기능

## 카피-온-라이트와 방어적 복사를 비교해 봅시다

|        카피-온-라이트         |                      방어적 복사                      |
| :---------------------------: | :---------------------------------------------------: |
| 통제할 수 있는 데이터 변경 시 |      신뢰할 수 없는 코드와 데이터 주고받을 경우       |
|  안전지대 어디서나 사용 가능  |        안전지대의 경계에서 데이터가 오고 갈 때        |
|           얕은 복사           |                       깊은 복사                       |
|  규칙: 복사본 생성→변경→리턴  | 규칙: 데이터가 들어오고 나갈 때 마다 깊은 복사본 생성 |

## 깊은 복사는 얕은 복사보다 비쌉니다

- 깊은 복사는 원본과 어떤 데이터 구조도 공유하지 않기 때문에 비싼 메모리 비용이 든다.
- 그래서 모든 곳에서 사용하는 것이 아닌 카피-온-라이트를 사용할 수 없는 곳에서만 사용하는 것이 좋다.

## 자바스크립트에서 깊은 복사 방법

> [!NOTE]
> 책에서 추천한 lodash와 같은 깊은 복사를 지원하는 라이브러리를 사용하라고 안내하고 있는데요.
> 관련해서 찾아보니 [structuredClone()](https://developer.mozilla.org/ko/docs/Web/API/structuredClone)라는 전역함수가 2022년부터 지원되었다고 합니다.
>
> 몇 가지 기능이나 제한 사항이 있어 주의가 필요하다고 하지만 안정화가 되면 추후에 사용도 가능하지 않을까..? 싶네요. 관련해서 같이 볼 만한 글도 공유합니다!
>
> - [구조화된 Clone을 사용하여 자바스크립트 딥 복사 ](https://web.dev/articles/structured-clone?hl=ko)

## 결론

- 방어적 복사: 불변성을 유지할 수 있는 강력하고 더 일반적인 원칙
- 불변성을 스스로 구현할 수 있기 때문에 더 강력하다.
- 더 많은 데이터를 복사해야 하므로 비용이 많이 든다.
- 카피-온-라이트와 함께 사용하면 필요할 때 언제든 적용할 수 있는 강력함과 얕은 복사로 인한 효율성에 대한 장점 모두 얻을 수 있다!
