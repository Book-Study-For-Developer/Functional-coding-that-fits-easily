## 레거시 코드와 불변성

한달에 한번씩하는 블랙 프라이데이 세일 준비하기

요구사항: 고객이 장바구니에 제품을 담을 때 행사 가격이 적용되도록 하기

블랙 프라이데이 관련 코드는 오래전에 만들었고, 카피-온-라이트가 적용되어있지 않으며, 이 코드는 장바구니 데이터를 변경한다.

지금 당장 고칠 수 없고 그대로 사용해야하는 레거시 코드임

```tsx
function add_item_to_cart(name: string, price: number) {
  const item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
  // 블랙프라이데이 관련 코드
  black_friday_promotion(shopping_cart);
}
```

블랙프라이데이 관련 코드를 추가하면서 카피-온-라이트 원칙을 지킬 수 없음

⇒ 방어적 복사 defensive copy

## 우리가 만든 카피-온-라이트 코드는 신뢰할 수 없는 코드와 상호작용해야 합니다

지금까지 우리가 만든 코드는 불변성이 지켜지는 안전지대에 있어 걱정 없이 사용 가능

블랙 프라이데이 행사 관련 함수는 안전지대 밖에 있어 그대로 사용한다면 이 함수의 입력과 출력을 통해 안전지대에 있는 코드와 데이터를 주고 받게 된다

그러면 안전지대 밖으로 나가는 데이터는 바뀔 수 있고, 안전지대로 들어오는 데이터도 바뀔 수 있어 카피-온-라이트 원칙을 지킬 수 없다

_🤔 블랙 프라이데이 행사 함수도 카피-온-라이트를 적용 하면?_

카피-온-라이트 패턴은 데이터를 바꾸기 전에 복사하기 떄문에 무엇이 바뀌는지 알 수 없는 레거시 코드(블랙프라이데이 코드)에서는 예상하기가 힘들다

여기서 **데이터가 바뀌는 것을 완벽히 막아주는 방어적 복사 원칙**이 필요

## 방어적 복사는 원본이 바뀌는 것을 막아줍니다

**들어오고 나가는 데이터의 복사본 만들기**

- 안전지대에 불변성 유지
- 바뀔 수도 있는 데이터가 안전지대로 들어오지 못하도록 하는 것

## 방어적 복사 구현하기

```tsx
function add_item_to_cart(name, price) {
  var item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);
  var total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
  // 데이터 전달하기 전에 복사
  var cart_copy = deepCopy(shopping_cart);
  black_friday_promotion(cart_copy);
  // 데이터를 전달하고 나서 복사(들어오는 데이터를 위한)
  shopping_cart = deepCopy(cart_copy);
}
```

## 방어적 복사 규칙

규칙1. 데이터가 안전한 코드에서 나갈 때 복사하기

1. 불변성 데이터를 위한 깊은 복사본 만들기

2. 신뢰할 수 없는 코드로 복사본을 전달

규칙2. 안전한 코드로 데이터가 들어올 때 복사하기

1. 변경될 수도 있는 데이터가 드어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달

2. 복사본을 안전한 코드에서 사용

## 신뢰할 수 없는 코드 감싸기

나중에 복사본을 만드는 이유를 모를 수 있고, `black_friday_promotion()`코드 재사용할 수 도 있기 때문에 코드 분리해 새로운 함수로 만들기

```tsx
function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
  shopping_cart = black_friday_promotion_safe(shopping_cart);
}

function black_friday_promotion_safe(cart) {
  const cart_copy = deepCopy(cart);
  black_Friday_promotion(cart_copy);
  return deepCopy(cart_copy);
}
```

## 방어적 복사가 익숙할 수도 있습니다

### 웹 API 속 방어적 복사

웹 API는 암묵적으로 방어적 복사를 한다!

클라이언트는 데이터를 인터넷을 통해 API로 보내려고 직렬화하는데 이때 JSON 데이터는 깊은 복사본임

잘 동작한다면 내려오는 응답 JSON도 깊은 복사본임

서비스에 들어올 때와 나갈 때 데이터를 복사 한 것!

### 얼랭과 엘릭서에서 방어적 복사

얼렝Erlan과 엘릭서Elxir(함수형 프로그래밍 언어)에서도 방어적 복사를 잘 구현함

얼렝Erlan

- 두 프로세스가 서로 메시지를 주고 받을 때 수신자의 메일박스에 메시지가 복사됨
- 프로세스에서 데이터가 나갈 때도 데이터를 복사함

얼랭과 엘릭서는 처음들어봤다..😂

## 카피-온-라이트와 방어적 복사 비교

|           | 카피-온-라이트                                                         | 방어적 복사                                                                                       |
| --------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 언제      | 통제할 수 있는 데이터를 바꿀 때                                        | 신뢰할 수 없는 코드와 데이터를 주고받아야 할 때                                                   |
| 어디서    | 안전지대 어디에서나 (카피-온-라이트가 불변성을 가진 안전지대로 만든다) | 안전지대의 경계에서 데이터가 오고 갈 때                                                           |
| 복사 방식 | 얕은 복사                                                              | 깊은 복사                                                                                         |
| 규칙      | 1. 바꿀 데이터의 얕은 복사를 만듦 2. 복사본을 변경 3. 복사본을 리턴    | 1. 안전지대로 들어오는 데이터에 깊은 복사를 만듦 2. 안전지대에서 나가는 데이터에 깊은 복사를 만듦 |

## 자바스크립트에서 깊은 복사를 구현하는 것은 어렵습니다

Lodash 라이브러리에 있는 깊은 복사 함수를 쓰는 것을 추천

Loadash에 .cloneDeep() 함수는 중첩된 데이터에 깊은 복사를 함

## 카피-온-라이트와 방어적 복사중에 어떤 원칙이 더 중요할까?

더 중요한 원칙은 없고 두가지 원칙을 상황에 따라 적절하게 사용해야한다

## 요점 정리

- 방어적 복사는 불변성을 구현하는 원칙이며 데이터가 들어오고 나갈 때 복사본을 만든다
- 방어적 복사는 깊은 복사를 해서 카피-온-라이트보다 비용이 더 든다
- 카피-온-라이트와 다르게 방어적 복사는 불변성 원칙을 구현하지 않은 코드로부터 데이터를 보호해준다
- 복사본이 많이 필요하지 않기 때문에 카피-온-라이트를 더 많이 사용함
- 방어적 복사는 신뢰할 수 없는 코드와 함께 사용할 때만 사용
- 깊은 복사는 위에서 아래로 중첩된 데이터 전체를 복사하며 얕은 복사는 필요한 부분만 최소한으로 복사

💬 지금 회사에서는 히스토리 파악이 대부분 가능하고, 변경도 가능해서 장바구니 코드를 리팩토링 하면서 블랙프라이데이 코드를 사용해야 한다면 블랙 프라이데이 코드도 리팩토링하는 편인데, 상황에 따라 건드릴 수 없는 레거시도 있겠구나 싶다. 나는 지금 실무에서 대부분 스프레드 연산자를 통해 얕은 복사만 하는 편인데 다른 분들은 어떤걸 어떻게 쓰고 계시는지 궁금하다.
