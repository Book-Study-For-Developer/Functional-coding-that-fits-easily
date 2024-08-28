> [!NOTE]
> 4장(액션에서 계산 빼내기)의 목표

- 어떻게 함수로 정보가 들어가고 나오는지 살펴보기
- 테스트하기 쉽고 재사용성이 좋은 코드를 만들기 위한 함수형 기술에 대해 알아보기
- 액션에서 계산을 빼내는 방법 배우기

## 액션에서 계산 빼내기

```js
// 원래 금액 합계를 업데이트하는 함수
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom(); // 금액 합계를 반영하기 위한 DOM 업데이트 함수
  update_shipping_icons(); // 무료 배송 아이콘을 띄울지 말지 결정하는 함수
  update_tax_dom(); // 금액 합계에 따라 달라지는 세금 업데이트 함수
}
```

```js
// 바꾼 코드
// 기존의 calc_cart_total 함수에서 계산(calc_total) 빼내기
// '서브루틴 추출하기'라고도 표현
function calc_cart_total() {
  calc_total();
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

function calc_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
}
```

```js
// 추출한 서브루틴은 여전히 액션이다. 그 이유는, shopping_cart_total 전역변숫값을 바꾸고 있고, shopping_cart를 읽고 있다.
// 암묵적 입력, 출력을 모두 명시적 입력, 출력으로 바꾼다면 아래와 같다.
function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

// 아래는 이제 계산이다. 모든 입력은 인자이고 모든 출력은 리턴값이다.
function calc_total(cart) {
  let total = 0;
  for (let i = 0; i < cart.length; i++) {
    const item = cart[i];
    total += item.price;
  }
  return total;
}
```

> [!NOTE]
> 이렇게 액션에서 계산을 빼면서 생기는 효과는 여러가지가 있다.

- DOM 업데이트와 비즈니스 규칙이 분리되었다. 위 코드에서 장바구니 합계 금액을 계산하는 것은 비즈니스 규칙이었다.
- 전역변수를 없애면서 테스트가 쉬워졌다.
- 전역변수에 의존하지 않아도 된다.
- 암묵적 출력이 함수의 리턴값으로 바뀌었다.
- 함수가 결괏값을 리턴한다.

## 액션에서 또 다른 계산 빼내기

```js
// 원래 코드
function add_item_to_cart(name, price) {
  shopping_cart.push({
    name: name,
    price: price,
  });

  calc_cart_total();
}

// 바꾼 코드
function add_item_to_cart(name, price) {
  add_item(shopping_cart, name, price);
  calc_cart_total();
}

function add_item(cart, name, price) {
  cart.push({
    name: name,
    price: price,
  });
}
```

> [!WARNING]
> 하지만 추출한 서브 루틴(add_item)은 액션이다. 왜냐하면 인자로 전달된 전역 변수 shopping_cart는 참조형이고 add_item 내에서 전역 변수를 변경(push)하는 암묵적 출력이다.

```js
// 암묵적 출력을 명시적 출력으로 바꾸기 위해서
// 참조형인 배열을 복사해서 바꾸고 출력하는 불변성을 구현한다.
function add_item_to_cart(name, price) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

// 이제 add_item 함수는 계산이 되었다!
function add_item(cart, name, price) {
  const new_cart = cart.slice();
  new_cart.push({
    name: name,
    price: price,
  });
  return new_cart;
}
```

## 쉬는 시간

- `Q1` : 코드가 더 많아진 것 같습니다. 잘하고 있는 것인가요? 코드가 적어야 더 좋은 것이 아닌가요?
  - 일반적으로 더 적은 코드가 좋은 것은 맞지만, 액션에서 계산을 분리하면서 코드를 테스트하기 쉬워졌고 재사용하기 좋아졌다.
- `Q2` : 재사용성을 높이고 쉽게 테스트할 수 있는 것이 함수형 프로그래밍으로 얻을 수 있는 전부인가요?
  - 이외에도, 동시성이나 설계, 데이터 모델링 측면에서 좋은 점들에 대해 더 설명할 것이다.
- `Q3` : 앞에서는 다른 곳에서 쓰기 위해 계산을 분리했습니다. 다른 곳에서 쓰지 않더라도 계산으로 분리하는 것이 중요한 것인가요?
  - 함수형 프로그래밍의 목적 중에 어떤 것을 분리해서 더 작게 만들려는 의도가 분명히 있다. 작은 것은 테스트하기 쉽고 재사용과 이해하기가 쉽다.
- `Q4` : 계산으로 바꾼 함수 안에서 아직도 변수를 변경하고 있습니다. 함수형 프로그래밍에서는 모든 것이 불변값이어야 한다고 들었는데요. 어떻게 설명할 수 있나요?
  - 생성할 때 초기화 단계가 불변값이 생성된 이후에 바뀌지 않아야 한다는 법칙에 위배되지만 피할 수 없다. 추후에 값이 바뀌지 않게 하는 원칙을 배운다.

## 계산 추출을 단계별로 알아보기

1. 계산 코드를 찾아 빼낸다.
2. 새 함수에 암묵적 입력과 출력을 찾는다.
3. 암묵적 입력은 인자로, 암묵적 출력은 리턴값으로 바꾼다.

## 연습 문제 - 세금 계산하는 부분 추출하기

```js
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1);
}
```

**먼저, 계산 코드를 찾아 빼낸다.**

```js
function update_tax_dom() {
  set_tax_dom(calc_tax());
}

function calc_tax() {
  return shopping_cart_total * 0.1;
}
```

**다음으로는, 새 함수에 암묵적 입력과 출력을 찾는다.**
calc_tax 함수에서 참조하고 있는 shopping_cart_total이 암묵적 입력에 해당한다.

**마지막으로, 암묵적 입력은 인자, 그리고 암묵적 출력은 리턴값으로 바꾼다.**

```js
// shopping_cart_total이 암묵적 입력이므로 인자로 바꾼다.
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

function calc_tax(total) {
  return total * 0.1;
}
```

**이제 calc_tax에는 비즈니스 규칙만 남아있으므로 테스트하기도 쉽고 재사용하기도 쉽다.**

## 연습 문제 - 무료 배송인지 확인하는 계산 추출하기

```js
// 원래 코드
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;
    if (item.price + shopping_cart_total >= 20)
      button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}
```

**먼저 계산 코드를 찾아 빼내기**

```js
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;
    if (is_free_shipping(item.price)) button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}

function is_free_shipping(price) {
  return price + shopping_cart_total >= 20;
}
```

**빼낸 계산 코드에서 암묵적 입력, 출력 찾아내기**
`shopping_cart_total`라는 전역 변수를 읽었으므로 암묵적 입력을 처리해야 한다.

**암묵적 입력은 인자로 바꾸기**

```js
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;
    if (is_free_shipping(shopping_cart_total, item.price))
      button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}

function is_free_shipping(total, price) {
  return price + total >= 20;
}
```

## 요점 정리

- 액션은 암묵적 입력이나 출력을 가지고 있다.
- 계산의 정의에 따르면 계산은 암묵적 입력과 출력이 없는 순수 함수여야 한다.
- 전역변수 같은 공유 변수는 일반적으로 암묵적 입력이나 출력이 된다.
- 암묵적 입력은 인자로 바꿀 수 있다.
- 암묵적 출력은 리턴값으로 바꿀 수 있다.
- 함수형 원칙을 적용하면 액션은 줄어들고 계산은 늘어난다.

## 의문이 드는 점

- React에서 사용하는 함수 컴포넌트는 함수의 일종인데, 함수 컴포넌트에 전달하는 인자(props)가 여러 하위 컴포넌트를 건너 뛰어 전달되는 경우 props drilling이라는 문제가 생겨서 전역 context를 활용하는 해결법을 사용하곤 한다. **그렇다면 함수형 프로그래밍을 고수한다면 전역 context을 읽고 쓰는 일은 암묵적 입력과 출력에 해당되므로 사용하면 안되는 패턴인걸까?**
