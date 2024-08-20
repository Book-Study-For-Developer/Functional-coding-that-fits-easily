## 온라인 쇼핑몰 코드 보기

장바구니에 담겨 있는 제품의 금액 합계를 볼 수 있는 기능

```tsx
const shopping_cart: { name: string; price: number }[] = [];
let shopping_cart_total = 0;

// 장바구니에 제품 담는 함수
function add_item_to_cart(name: string, price: number) {
  shopping_cart.push({ name, price });
  calc_cart_total();
}

// 장바구니 합계 금액을 업데이트 하는 함수
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom();
}
```

### 새로운 요구사항 1 : 무료 배송비 계산하기

- 구매 합계가 20달러 이상이면 무료 배송 해주려고 함
- 장바구니에 넣으면 20달러 이상이 되는 제품의 구매 버튼 옆에 무료 배송 아이콘 표시

절차적 방법

```tsx
// 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    if (item.price + shopping_cart_total >= 20) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// 장바구니 합계 금액을 업데이트 하는 함수 (앞에서 만든 함수)
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom();
  update_shipping_icons(); // 추가
}
```

### 새로운 요구사항 2 : 세금 계산하기

장바구니 금액의 합계가 바뀔 때마다 세금을 다시 계산하기

```tsx
// 세금 계산하는 함수
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1);
}

// 장바구니 합계 금액을 업데이트 하는 함수 (앞에서 만든 함수)
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom(); // 추가
}
```

### 테스트 하기가 어려움

```tsx
function update_tax_dom() {
	set_tax_dom(**shopping_cart_total * 0.10**);
}
```

위 코드에서 테스트해야하는 비즈니스 규칙 (total \* 0.10)

- set_tax_dom을 얻는 방법은 DOM에서 값을 가져오는 방법
- shopping_cart_total의 전역변숫값을 설정해야함

테스트를 더 쉽게 하려면?

- DOM 업데이트와 비즈니스 규칙은 분리
- 전역변수가 없어야 함

### 재사용하기가 어려움

결제팀과 배송팀에서 재사용하기가 어려움

- 지금 코드는 장바구니 정보를 전역변수에서 읽어오고 있지만, 타팀에서는 데이터베이스에서 읽어와야함
- 지금 코드는 DOM을 직접 바꾸고 있지만, 타팀에서는 영수증과 운송장을 출력해야함

```tsx
function update_shipping_icons () {
	const buy_buttons = get_buy_buttons_dom();
	  for (let i = 0; i < buy_buttons.length; i++) {
	    const button = buy_buttons[i];
	    const item = button.item;

	    if (item.price + shopping_cart_total **>= 20**) {
	      button.show_free_shipping_icon(); // DOM이 있어야 실행 가능
	    } else {
	      button.hide_free_shipping_icon(); // DOM이 있어야 실행 가능
	    }
	  }
}
```

타팀에서 사용하려는 비즈니스 규칙 (≥ 20)

- 전역변수인 shopping_cart_total이 있어야 실행 가능
-
- 리턴값이 없어서 결과를 받을 수 없음

재사용 하려면?

- 전역변수에 의존 X
- DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안됨
- 함수가 결과값을 리턴해야함

전체 코드

```tsx
const shopping_cart: { name: string; price: number }[] = [];
let shopping_cart_total = 0;

// 장바구니에 제품 담는 함수
function add_item_to_cart(name: string, price: number) {
  shopping_cart.push({ name, price });
  calc_cart_total();
}

// 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    if (item.price + shopping_cart_total >= 20) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// 세금 계산하는 함수
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1);
}

// 장바구니 합계 금액을 업데이트 하는 함수
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
```

- 전역 변수는 변경 가능하기 때문에 → 액션
- 전역 변수 변경하는 것 → 액션
- DOM에서 읽는 것 → 액션
- DOM을 바꾸는 것 → 액션

코드 전체가 액션이다

### 함수에는 입력과 출력이 있다

```tsx
let total = 0;

function add_to_total(amount) {
  // amount: 명시적 입력
  console.log("old total: ", total); // total: 암묵적 입력(전역변수 읽기 때문)
  total += amount; // 암묵적 출력(전역변수 변경하기 때문)
  return total; // 명시적 출력
}
```

⇒ 입력과 출력은 명시적이거나 암묵적일 수 있으며, 함수에 **암묵적 입력과 출력이 있으면 액션**이 됨!!

## 액션에서 계산 빼내기

장바구니에 담긴 제품 금액 합계 계산

```tsx
// 기존 코드
function calc_cart_total() {
  shopping_cart_total = 0;
  for (let i = 0; i < shopping_cart.length; i++) {
    const item = shopping_cart[i];
    shopping_cart_total += item.price;
  }

  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

// 리팩토링한 코드

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);

  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

// 암묵적 입력과 출력을 모두 없애면서 리팩토링을 진행한 함수
function calc_total(cart: Cart) {
  let total = 0;
  cart.forEach((item) => {
    total += item.price;
  });
  return total;
}
```

add_item_to_cart에서도 계산 빼내기

```tsx
// 기존 코드
function add_item_to_cart(name: string, price: number) {
  shopping_cart.push({ name, price });
  calc_cart_total();
}

// 리팩토링한 코드
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

function add_item(cart: Cart, name: string, price: number) {
  const new_cart = cart.slice(); // 복사본을 만들어 할당
  new_cart.push({ name, price });
  return new_cart;
}
```

어떤 값을 바꿀 때 그 값을 복사해서 바꾸는 카피 온 라이트 copy-on-write 방법을 사용하여 불변성 구현

update_tax_dom에서도 계산 빼내기

```tsx
// 기존 코드
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1);
}

// 리팩토링한 코드
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

function calc_tax(amount: number) {
  return total * 0.1;
}
```

update_shipping_icon에서도 계산 빼내기

```tsx
// 기존 코드
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for (let i = 0; i < buy_buttons.length; i++) {
    const button = buy_buttons[i];
    const item = button.item;

    if (item.price + shopping_cart_total >= 20) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// 리팩토링한 코드
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

function get_free_shipping(total, item_price) {
  return item_price + total >= 20;
}
```

리팩토링한 전체 코드

```tsx
interface Cart {
  name: string;
  price: number;
}

const shopping_cart: Cart[] = [];
let shopping_cart_total = 0;

// 장바구니에 항목 추가하고 총합 계산하는 함수 A
function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

//장바구니의 총합 계산 및 업데이트하는 함수 A
function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

// 무료 배송 아이콘 업데이트 하는 함수 A
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

// 세금 DOM을 업데이트 A
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

// 장바구니 총합 계산 C
function calc_total(cart: Cart[]) {
  let total = 0;
  cart.forEach((item) => {
    total += item.price;
  });
  return total;
}

// 장바구니에 새 항목 추가 C
function add_item(cart: Cart[], name: string, price: number) {
  const new_cart = cart.slice();
  new_cart.push({ name, price });
  return new_cart;
}

// 무료 배송 조건을 판단 C
function get_free_shipping(total, item_price) {
  return item_price + total >= 20;
}

// 세금 계산 C
function calc_tax(amount: number) {
  return total * 0.1;
}
```

## 요점 정리

- 액션은 암묵적인 입력 또는 출력을 가지고 있음
- 계산은 암묵적인 입력이나 출력이 없어야함
- 전역변수는 암묵적 입력 또는 출력이 됨
- 암묵적 입력은 인자로 바꿀 수 있음
- 암묵적 출력은 리턴값으로 바꿀 수 있음
- 함수형 원칙을 사용하면 액션은 줄어들고 계산은 늘어남
