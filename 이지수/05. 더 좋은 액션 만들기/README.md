## 비즈니스 요구 사항과 설계 맞추기

요구사항

- 장바구니에 담긴 제품을 주문할 때 무료 배송인지 확인하는 것

코드 구현 함수

- 장바구니로 무료 배송인지 확인 X
- 제품의 합계와 가격으로 확인

비즈니스 요구사항과 맞지 않음

## 비즈니스 요구 사항과 함수 맞추기

변경 전 코드

```tsx
function gets_free_shipping(total, item_price) {
  return item_price + total >= 20;
}

function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    if (get_free_shipping(shopping_cart_total, item.price)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}
```

변경 후 코드

```tsx
function gets_free_shipping(cart) {
  return calc_total(cart) >= 20;
}

function update_shipping_icons() {
  const buy_buttons = gets_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    const new_cart = add_item(shopping_cart, item.name, item.price);
    if (gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}
```

- 원래 있던 장바구니를 직접 변경하지 않고 복사본을 만들어 프로그래밍함

## 원칙: 암묵적 입력과 출력은 적을수록 좋습니다

## 암묵적 입력과 출력 줄이기

원래 코드

```tsx
function update_shipping_icons() {
  const buy_buttons = gets_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    const new_cart = add_item(shopping_cart, item.name, item.price);
    if (gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
```

명시적 인자로 바꾼 코드

```tsx
function update_shipping_icons(cart) {
  const buy_buttons = gets_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    const new_cart = add_item(cart, item.name, item.price);
    if (gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons(shopping_cart);
  update_tax_dom();
}
```

### 연습문제

```tsx
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons(shopping_cart);
  update_tax_dom();
}

function set_cart_total_dom() {
	...
	shopping_cart_total
	...
}

function update_shipping_icons(cart){
	const buy_buttons = gets_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    const new_cart = add_item(cart, item.name, item.price)
    if(gets_free_shipping(new_cart)){
	    button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}
```

변경 후 코드

```tsx
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total(shopping_cart);
}

function calc_cart_total(cart) {
  const total = calc_total(cart);
  set_cart_total_dom(total);
  update_shipping_icons(cart);
  update_tax_dom(total);
  shopping_cart_total = total;
}

function set_cart_total_dom(total) {
	...
	total
	...
}

function update_shipping_icons(cart){
	const buy_buttons = gets_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    const new_cart = add_item(cart, item.name, item.price)
    if(gets_free_shipping(new_cart)){
	    button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

function update_tax_dom(total) {
  set_tax_dom(calc_tax(total));
}
```

문제점

- 사용하지 않는 shopping_cart_total 전역변수
- 과해 보이는 calc_cart_total()

위의 문제 개선한 코드

```tsx
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  const total = calc_total(cart);
  set_cart_total_dom(total);
  update_shipping_icons(cart);
  update_tax_dom(total);
}
```

## 계산 분류하기

장바구니 구조 (C), 제품에 대한 구조 (I), 비즈니스 규칙에 대한 함수 (B)

```tsx
// C, I
function add_item(cart: Cart[], name: string, price: number) {
  const new_cart = cart.slice();
  new_cart.push({ name, price });
  return new_cart;
}

// C, I, B
function calc_total(cart: Cart[]) {
  let total = 0;
  cart.forEach((item) => {
    total += item.price;
  });
  return total;
}

// B
function gets_free_shipping(total, item_price) {
  return item_price + total >= 20;
}

// B
function calc_tax(amount: number) {
  return amount * 0.1;
}
```

## 원칙: 설계는 엉켜있는 코드를 푸는 것이다

- 재사용하기 쉽다
- 유지보수 하기 쉽다
- 테스트하기 쉽다

## add_item()을 분리해 더 좋은 설계 만들기

```tsx
function add_item(cart: Cart[], name: string, price: number) {
  // 1. 배열을 복사
  const new_cart = cart.slice();
  // 2. item 객체를 만듦  3. 복사본에 item 추가
  new_cart.push({ name, price });
  // 4. 복사본 리턴
  return new_cart;
}
```

item에 관한 코드를 별도의 함수로 분리하기

```tsx
// item 구조만 알고 있는 함수
function make_cart_item(name: string, price: number) {
  return { name, price };
}

// cart 구조만 알고 있는 함수
function add_item(cart, item) {
  // 1. 배열을 복사
  const new_cart = cart.slice();
  // 3. 복사본에 item 추가
  new_cart.push(item);
  // 4. 복사본 리턴
  return new_cart;
}

add_item(shopping_cart, make_cart_item("shoe", 3.45));
```

1, 3, 4 단계는 카피-온-라이트를 구현한 부분이기 때문에 함께 두는 것이 좋음

## 카피-온-라이트 패턴을 빼내기

원래 코드

```tsx
function add_item(cart, item) {
  const new_cart = cart.slice();
  new_cart.push(item);
  return new_cart;
}
```

일반적인 이름으로 바꾼 코드

```tsx
function add_elemet_last(array, elem) {
  const new_array = array.slice();
  new_array.push(elem);
  return new_array;
}
```

add_item()함수 다시 만들기

```tsx
function add_item(cart, item) {
  add_element_last(cart, item);
}
```

add_elemet_last()는 어떤 배열이나 항목에도 쓸 수 있는 재사용 가능한 유틸리티 함수

## add_item() 사용하기

```tsx
functiom add_to_cart(name: string, price: number){
	const item = make_cart_item(name, price);
	shopping_cart = add_item(shopping_cart, item);

	const total = calc_total(cart);
  set_cart_total_dom(total);
  update_shipping_icons(cart);
  update_tax_dom(total);
}
```

## 계산을 분류하기

```tsx
C: cart에 대한 동작
I: item에 대한 동작
B: 비즈니스 규칙
A: 배열 유틸리티

// A
function add_elemet_last(array, elem) {
	const new_array = array.slice();
  new_array.push(elem);
  return new_array;
}

// C
function add_item(cart, item) {
	add_element_last(cart, item);
}

// I
function make_cart_item(name: string, price: number) {
	return { name, price }
}

// C, I, B
function calc_total(cart: Cart[]) {
  let total = 0;
  cart.forEach((item) => {
    total += item.price;
  });
  return total;
}

// B
function gets_free_shipping(total, item_price) {
  return item_price + total >= 20;
}

// B
function calc_tax(amount: number) {
  return amount * 0.1;
}

```

## 요점 정리

- 암묵적 입력과 출력은 인자와 리턴값으로 바꿔 없애는 것이 좋음
- 설계는 엉켜있는 것을 푸는 것
- 엉켜 있는 것을 풀어 각 함수가 하나의 일만 하도록 하면 개념을 중심으로 쉽게 구성 할 수 있음
