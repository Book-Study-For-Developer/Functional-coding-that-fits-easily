4장에서 리팩토링한 함수

```tsx
const TAX_FEE = 0.10;
const FREE_SHIPPING_AMOUNT = 20;

let shopping_cart = [];
let shopping_cart_total = 0;

// 액션
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}
function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    if(gets_free_shipping(shopping_cart_total, item.price)) {
      buy_button.show_free_shipping_icon();
    } else {
      buy_button.hide_free_shipping_icon();
    }
  })
}
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

// 계산
function gets_free_shipping(total: number, item_price: number) {
	return item_price + total >= FREE_SHIPPING_AMOUNT;
}
function calc_tax(amount: number) { 
	return amount * TAX_FEE;
}
function calc_total(cart: Cart) {
	const total = cart.reduce((accumulator, currentValue) => {
		return acc + currentValue.price;
	}, 0);
	return total;
}
function add_item(cart: Cart, name: string, price: number) {
	const new_cart = [...cart, { name, price }];
	return new_cart;
}
```

## 비지니스 요구 사항과 함수를 맞추기

함수에 맞지 않는 인자를 비지니스 요구사항에 맞는 인자로 변경하는 리팩토링을 진행해본다.

```tsx
// 계산
function gets_free_shipping(cart: Cart) {
	return calc_total(cart) >= FREE_SHIPPING_AMOUNT;
}

function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    // 새로운 장바구니를 가져오기
    const new_cart = add_item(shopping_cart, item.name, item.price);
    
    if(gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  })
}
```

<aside>
💡 **잠깐 쉬어가기**

Q. **코드 라인수가 늘어나면 좋은 코드인가?**
함수의 라인수는 늘어났지만, 함수의 크기는 작아졌기 때문에 이해하기 쉬워졌다.
****즉, 응집력있고 재사용하기 쉽다.

**Q. `add_item()` 함수를 부를 때 마다 배열을 복사하는데, 괜찮을까?**
GC는 생각보다 효율적으로 처리하고, 복사할 때 잃는 것보다 얻는 것이 더 많다.
섣부른 최적화는 하지 않는게 더 중요하다.

나였다면?
나도 당연히 새로운 함수를 원본을 훼손시키는 것보다 복사해서 쓰는 것을 선호한다.
그렇기에 위처럼 함수를 작게 나누면서 작업하는 것이 먼저 선행되야 하는 것 같다.

</aside>

### 🍎 원칙: 암묵적 입력과 출력은 적을수록 좋다.

- 암묵적 입력: 인자가 아닌 모든 입력
- 암묵적 출력: 리턴값이 아닌 모든 출력

**암묵적 입력/출력이 없는 함수는 계산**이며 많을 수록 **결함도가 높은 컴포넌트**라고 할 수 있다.

즉, 액션을 계산으로 바꾸지는 못하더라도 암묵적 입력과 출력을 줄이면 테스트에 용이하고 재사용하기 좋다.

## 암묵적 입력과 출력 줄이기

암묵적 입력과 출력을 줄여서 코드를 리팩토링해보려고 한다.

**리팩토링의 목적: 암묵적 입력을 제거하자(전역변수 사용을 제거하자)**

```tsx
function update_shipping_icons(cart: Cart) {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    const new_cart = add_item(cart, item.name, item.price);
    
    if(gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  })
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons(shopping_cart);
  update_tax_dom();
}
```

함수 내부의 암묵적 입력을 제거하여 **명시적 입력으로 변경**하였다.

**리팩토링의 목적: 전역변수의 사용을 제거하자(**`shopping_cart`, `shopping_cart_total`)

```tsx
function add_item_to_cart(cart: Cart, name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total(shopping_cart);
}

function calc_cart_total(cart: Cart) {
  const total = calc_total(cart);
  set_cart_total_dom(total);
  update_shipping_icons(cart);
  update_tax_dom(total);
  shopping_cart_total = total;
}

function update_shipping_icons(cart: Cart) {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    const new_cart = add_item(cart, item.name, item.price);
    
    if(gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  })
}

function update_tax_dom(total: number) {
  set_tax_dom(calc_tax(total));
}
```

하나씩 살펴보면서 중복된 코드나 불필요한 코드가 있는지 보도록 하자

- `calc_cart_total()` 는 불필요한 코드가 존재
    - 전역변수에 할당하는 부분이 존재한다.

```tsx
function add_item_to_cart(cart: Cart, name: string, price: number) {
  // 장바구니에 물건을 추가
  shopping_cart = add_item(shopping_cart, name, price);
  // 총합을 계산
  const total = calc_total(shopping_cart);
  // 총액 DOM에 반영
  set_cart_total_dom(total);
  // 아이콘 변경
  update_shipping_icons(shopping_cart);
  // 세금 DOM에 반영
  update_tax_dom(total);
}

// 함수를 제거
~~function calc_cart_total(cart: Cart) {
  const total = calc_total(cart);
  set_cart_total_dom(total);
  update_shipping_icons(cart);
  update_tax_dom(total);
  shopping_cart_total = total;
}~~
```

> **💡실무였다면?**
>
> DOM에 직접적으로 업데이트 하는건 리액트의 담당이 대신해줄테니, useState의 상태값을 업데이트만 해준다면 알아서 반영이 될 것이다.
> 

## 계산 분류하기

크게 액션, 계산, 데이터로 분류했지만, 계산을 의미있게 더욱 분류해보자

```
C - cart에 대한 동작
I - item에 대한 동작
B - 비지니스 로직
```

```tsx
// B
function gets_free_shipping(total: number, item_price: number) {
	return item_price + total >= FREE_SHIPPING_AMOUNT;
}

// B
function calc_tax(amount: number) { 
	return amount * TAX_FEE;
}

// C, I, B
function calc_total(cart: Cart) {
	const total = cart.reduce((accumulator, currentValue) => {
		return acc + currentValue.price;
	}, 0);
	return total;
}

// C, I
// 내가 생각한 방법
function add_item(cart: Cart, name: string, price: number) {
	const new_cart = [...cart, { name, price }];
	return new_cart;
}

// 책의 코드
function add_item(cart: Cart, name: string, price: number) {
	const new_cart = cart.slice();
	new_cart.push({
		name: name,
		price: price
	})
	return new_cart;
}
```

단순히 계산함수로 볼 때는 같은 계층이지만, 이렇게 나누면 계층이 엉켜있는 것을 볼 수 있다.

그렇다면 엉켜있는 계층을 풀어보자.

### 🍎 원칙: 설계는 엉켜있는 코드를 푸는 것이다.

함수를 사용하여 관심사를 분리하고, 인자를 넘기고 그 값을 사용하는 방법을 분리한다. 이렇게 분리된 것은 언제든 **쉽게 조합**할 수 있다.

```tsx
function add_item(cart: Cart, name: string, price: number) {
  // 1, 2, 3. 배열을 복사하여, item을 만들고, 추가한다.
	const new_cart = [...cart, { name, price }];
	return new_cart; // 4. 복사본을 리턴
}

function add_item(cart: Cart, name: string, price: number) {
	const new_cart = cart.slice(); // 1. 배열을 복사
	new_cart.push({ // 3. 복사본에 item을 추가
		name: name, // 2. item 객체를 만든다.
		price: price
	})
	return new_cart; // 4. 복사본을 리턴
}

// 분리해보자
function make_cart_item(name: string, price: number) {
	return { name, price };
}

type Item = {
	name: string,
	price: number
}

function add_item(cart: Cart, item: Item) {
	const new_cart = [...cart, item];
	return new_cart;
}

function add_item(cart: Cart, item: Item) {
	const new_cart = cart.slice();
	new_cart.push(item)
	return new_cart;
}

// 사용하는 쪽에서 만들어서 넘겨줌
add_item(shopping_cart, make_cart_item("shoes", 3.45));
```

add_item은 일반적으로 item을 추가하는 함수이지만 **장바구니를 받도록 되어있기에 조금 더 일반적인 방법**으로 바꾸는 것이 좋아보인다.

```tsx
function add_elemnt_last<T>(array: Array, item: T) {
	const new_array = [...array, item];
	return new_array;
}

function add_elemnt_last<T>(array: Array, elem: T) {
	const new_array = array.slice();
	new_array.push(elem);
	
	return new_array;
}
```

이렇게 변경을 하게 되면 배열 내에 항목을 추가하고 싶을 때 재사용할 수 있는 유틸리티 함수로 만들어지게 되었다.

> **💡실무였다면?**
>
> 일반화된 함수를 만들었지만, 이렇게까지 작게 만들어 재사용할 일이 많을까 생각이 들었다.
> 너무 과도한 일반화는 오히려 힘들지 않을까..?
> 

최종적으로 만들어진 코드

```
C - cart에 대한 동작
I - item에 대한 동작
B - 비지니스 로직
A - 배열 유틸리티
```

```tsx
// A
function add_elemnt_last<T>(array: Array, item: T) {
	const new_array = [...array, item];
	return new_array;
}

// C
function add_item(cart: Cart, item: Item) {
	return add_elemnt_last<Item>(cart, item);
}

// I
function make_cart_item(name: string, price: number) {
	return { name, price };
}

// C, I, B
function calc_total(cart: Cart) {
	const total = cart.reduce((accumulator, currentValue) => {
		return acc + currentValue.price;
	}, 0);
	return total;
}

// B
function gets_free_shipping(total: number, item_price: number) {
	return item_price + total >= FREE_SHIPPING_AMOUNT;
}

// B
function calc_tax(amount: number) { 
	return amount * TAX_FEE;
}
```

> **💡 비지니스 규칙(로직)이란?**
> 비지니스 규칙이란 특정 서비스에서 적용되는 특별한 로직이라고 보면된다.
예를 들어 장바구니 기능은 전자상거래 서비스에는 모두 존재할 거라고 생각하지만, 특정 규칙에 의해 무료 배송이 되는 규칙을 비지니스 규칙이라고 부른다.
> 

### `update_shipping_icons()` 함수를 의미있는 계층으로 분류해보자.

```tsx
function update_shipping_icons(cart: Cart) {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    const hasFreeShipping = gets_free_shipping_with_item(cart, item);
    set_free_shipping_icon(buy_button, hasFreeShipping);
  })
}

// cart와 Item 관련 동작
function gets_free_shipping_with_item(cart, item) {
	const new_cart = add_item(cart, item);
	return gets_free_shipping(new_cart);
}

// DOM 관련 동작
// NOTE: React를 사용했다면 상태를 업데이트만 한다면 리렌더링을 통해 화면상에서 제거될 것이다.
function set_free_shipping_icon(button, isShown) {
	if(isShown) {
		button.show_free_shipping_icon();
	} else {
		button.hide_free_shipping_icon();
	}
}
```

**최종적으로 리팩토링 된 함수**

```tsx
const TAX_FEE = 0.10;
const FREE_SHIPPING_AMOUNT = 20;

let shopping_cart = [];

// 액션
function add_item_to_cart(cart: Cart, name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}
function update_shipping_icons(cart: Cart) {
  const buy_buttons = get_buy_buttons_dom();
  
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    const hasFreeShipping = gets_free_shipping_with_item(cart, item);
    set_free_shipping_icon(buy_button, hasFreeShipping);
  })
}
function update_tax_dom(total: number) {
  set_tax_dom(calc_tax(total));
}

// 계산
function add_elemnt_last<T>(array: Array, item: T) {
	const new_array = [...array, item];
	return new_array;
}
function add_item(cart: Cart, item: Item) {
	return add_elemnt_last<Item>(cart, item);
}
function make_cart_item(name: string, price: number) {
	return { name, price };
}
function calc_total(cart: Cart) {
	const total = cart.reduce((accumulator, currentValue) => {
		return acc + currentValue.price;
	}, 0);
	return total;
}
function gets_free_shipping(total: number, item_price: number) {
	return item_price + total >= FREE_SHIPPING_AMOUNT;
}
function calc_tax(amount: number) { 
	return amount * TAX_FEE;
}
function gets_free_shipping_with_item(cart, item) {
	const new_cart = add_item(cart, item);
	return gets_free_shipping(new_cart);
}
function set_free_shipping_icon(button, isShown) {
	if(isShown) {
		button.show_free_shipping_icon();
	} else {
		button.hide_free_shipping_icon();
	}
}
```