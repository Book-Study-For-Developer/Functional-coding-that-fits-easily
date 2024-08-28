## 5장의 목표!

**액션에서 암묵적 입력과 출력을 줄여 설계를 개선하는 방법 알아보기**

---

### 비즈니스 요구 사항과 설계를 맞추기

```ts
// 아래 함수는 비즈니스 요구 사항과 맞지 않는 부분이 있다.
function gets_free_shipping(total: number, item_price: number) {
  return item_price + total >= 20;
}

function calc_total(cart) {
  let total = 0;
  for (let i = 0; i < cart.length; i++) {
    const item = cart[i];
    total += item.price; // 장바구니 합계를 계산하는 비즈니스 규칙 중복
  }
  return total;
}
```

> [!WARNING]
> 합계 금액(total)과 제품 가격(item_price)에 대한 무료 배송 여부가 아니라 주문 결과가 무료 배송인지 확인해야 한다.

```ts
type ItemType = {
  name: string;
  price: number;
};

// 함수 시그니처를 변경함으로써 비즈니스 요구 사항을 코드에 반영했다.
// 비즈니스 규칙 중복도 없앴다.
function gets_free_shipping(cart: ItemType[]) {
  return calc_total(cart) >= 20;
}

// 새로운 함수를 적용
function update_shipping_icons() {
  const buttons = get_buy_buttons_dom();
  buttons.forEach((button) => {
    const item = button.item;
    const new_cart = add_item(shopping_cart, item.name, item.price);
    if (gets_free_shipping(new_cart)) button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  });
}
```

> [!WARNING]
> 여전히 update_shipping_icons 함수에는 암묵적 입력과 출력이 있다. 테스트를 쉽게 하기 위해 액션을 최대한 계산으로 바꾸고, 모듈화된 컴포넌트로 만들어야 한다.

```ts
type ItemType = {
  name: string;
  price: number;
};

function update_shipping_icons(cart: ItemType[]) {
  const buttons = get_buy_buttons_dom();
  buttons.forEach((button) => {
    const item = button.item;
    const new_cart = add_item(cart, item.name, item.price); // 전역 변수 shopping_cart 제거
    if (gets_free_shipping(new_cart)) button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  });
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons(shopping_cart);
  update_tax_dom();
}
```

---

### 계산 분류하기

**의미 있는 계층에 대해 알아보기 위해 계산을 분류해보자.**

> [!NOTE]
> 함수를 사용하면 관심사를 자연스럽게 분리할 수 있다. 함수는 인자로 넘기는 값과 그 값을 사용하는 방법을 분리한다. 분리된 것은 언제나 쉽게 조합할 수 있지만, 잘 분리하는 방법을 찾기는 어렵다.

### add_item()을 분리해 더 좋은 설계 만들기

```ts
type ItemType = {
  name: string;
  price: number;
};

// 아래 함수는 네 부분으로 분리할 수 있다.
function add_item(cart: ItemType[], name: string, price: number) {
  const new_cart = cart.slice(); // 1. 배열 복사
  new_cart.push({ name: name, price: price }); // 2. 객체 리터럴로 item 객체 만들기, 3. 복사본에 item 추가
  return new_cart; // 4. 복사본 반환
}
```

코드를 쪼개본다면,

```ts
// 2. 객체 리터럴로 item 객체 만들기
function make_cart_item(name: string, price: number) {
  return {
    name: name,
    price: price,
  };
}

function add_item(cart: ItemType[], item: ItemType) {
  const new_cart = cart.slice(); // 1. 배열 복사
  new_cart.push(item); // 3. 복사본에 item 추가
  return new_cart; // 4. 복사본 반환
}
```

이렇게 쪼갠 코드는 다음과 같은 효과가 생긴다.

1. **cart를 해시 맵 같은 자료 구조로 변경할 때, 바꿀 부분이 적다.**
2. **값을 바꿀 때 복사하는 카피-온-라이트를 한 부분에 구현해서 알아보기 쉽다.**

### 카피-온-라이트 패턴 빼내기

add_item은 `카피-온-라이트`를 사용해서 배열에 항목을 추가하는 함수가 되었기 때문에 일반적인 배열과 항목에도 사용할 수 있다.

그렇기 때문에 조금 더 일반적인 이름으로 바꿔서 재사용 가능한 유틸리티 함수를 만들 수 있다. 장바구니에 제품을 추가하는 것뿐만 아니라 배열에 항목을 추가할 때 재사용할 수 있다.

```ts
function add_element_last(array, elem) {
  const new_array = array.slice();
  new_array.push(elem);
  return new_array;
}
```

### add_item() 사용하기

일반적인 이름의 유틸리티 함수를 장바구니에 제품을 추가하는 목적을 갖는 `add_item`에서 사용하도록 한다.

```ts
function add_item(cart, item) {
  return add_element_last(cart, item);
}

function add_item_to_cart(name: string, price: number) {
  const item = make_cart_item(name, price); // item 객체를 만드는 계산
  shopping_cart = add_item(shopping_cart, item); // add_item 계산 호출
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}
```

> 가벼운 질문

**Q. 왜 계산을 유틸리티와 장바구니, 비즈니스 규칙으로 다시 나누는 것인지?**

- 최종적으로 코드를 구분된 그룹과 분리된 계층으로 구성하기 위함이다. 나중에 다룰 설계 기술을 미리 익히기 위해 나누었다.

**Q. 비즈니스 규칙과 장바구니 기능은 어떤 차이가 있는지? 전자상거래를 만드는 것이라면 장바구니에 관한 것은 모두 비즈니스 규칙이 아닌지?**

- 장바구니가 전자상거래 서비스에서 사용하는 일반적인 개념이라면, 비즈니스 규칙은 다르다. MegaMart와 모두 똑같은 무료 배송 규칙이 있으리라 기대하지 않는다.

**Q. 비즈니스 규칙과 장바구니에 대한 동작에 모두 속하는 함수도 있을 수 있나요?**

- 계층 관점에서 보면 코드 냄새다. 비즈니스 규칙은 장바구니 구조와 같은 하위 계층보다 빠르게 바뀌기 때문에 차차 분리해나가야 한다.

### 연습문제 - update_shipping_icons() 함수 분류하기

```ts
function update_shipping_icons(cart) {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;
    const new_cart = add_item(cart, item);
    if (gets_free_shipping(new_cart)) button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}
```

함수가 하는 일은,

1. 모든 버튼을 가져오기(`구매하기 버튼 관련 동작`)
2. 버튼을 가지고 반복하기(`구매하기 버튼 관련 동작`)
3. 버튼에 관련된 제품을 가져오기(`구매하기 버튼 관련 동작`)
4. 가져온 제품을 가지고 새 장바구니 만들기(`cart와 item 관련 동작`)
5. 장바구니가 무료 배송이 필요한지 확인하기(`cart와 item 관련 동작`)
6. 아이콘 표시하거나 감추기(`DOM 관련 동작`)

이 함수를 하나의 분류에만 속하도록 바꾸면,

```ts
type ItemType = {
  name: string;
  price: number;
};

function update_shipping_icons(cart: ItemType[]) {
  const buy_buttons = get_buy_buttons_dom(); // 1. 모든 버튼을 가져오기

  buy_buttons.forEach((button) => {
    // 2. 버튼을 가지고 반복하기
    const item = button.item; // 3. 버튼에 관련된 제품을 가져오기
    const isFreeShipping = gets_free_shipping_with_item(cart, item);
    set_free_shipping_icon(button, isFreeShipping);
  });
}

function gets_free_shipping_with_item(cart: ItemType[], item: ItemType) {
  const new_cart = add_item(cart, item); // 4. 가져온 제품을 가지고 새 장바구니 만들기
  return gets_free_shipping(new_cart); // 5. 장바구니가 무료 배송이 필요한지 확인하기
}

function set_free_shipping_icon(button, isShown) {
  if (isShown) button.show_free_shipping_icon(); // 6. 아이콘 표시하거나 감추기
  else button.hide_free_shipping_icon();
}
```

> [!NOTE]
> 액션에서 계산을 추출하고 계산을 분류하는 과정 속에서 재사용할 수 있는 유용한 인터페이스 함수가 많이 생겼다.

---

## 느낀 점

- 함수를 쪼갤수록 함수가 늘어나기 때문에 당연히 전체적인 코드의 양은 늘어나지만, 함수 하나만 놓고 보면 코드의 길이가 이전보다 짧아지기 때문에 함수가 덜 복잡해보이는 효과가 있는 것 같다.
- 알고리즘 문제를 풀이할 때도 그렇지만, 시간에 쫓겨 중복되는 비즈니스 규칙을 분리해서 재사용이 가능하도록 했던 기억이 많이 없기 때문에 단순히 더하기나 빼기와 같은 연산을 중복으로 사용하는 경우를 민감하게 받아들이는 연습이 필요하다는 생각이 든다.
- 액션에서 계산을 추출하는 것이 끝이 아니라, 계산에서 하고 있는 **여러 가지 일을 파악**해서 분리하는 일이 중요하기 때문에 함수형 프로그래밍을 숙지하는데 짧지 않은 시간이 걸리겠다는 생각이 든다.
