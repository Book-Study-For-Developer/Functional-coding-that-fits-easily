## 온라인 쇼핑몰 서비스 코드를 살펴보며 액션에서 계산 빼내기

```tsx
const shopping_cart = [];
let shopping_cart_total = 0;

// 장바구니에 제품을 담는 함수
function add_item_to_cart(name: string, price: number) {
  shopping_cart.push({
    name: name,
    price: price
  });
  calc_cart_total();
}

// 카트 금액을 변경시키는 함수
function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  // 금액 합계 반영을 위한 DOM 업데이트
  set_cart_total_dom();
}
```

### 새로운 요구사항이 계속해서 들어온다.

**무료 배송하기 기능 추가**

구매 합계가 20달러 이상이면, 무료 배송을 해주려고 하기 때문에 **구매 버튼 옆에 무료배송 아이콘을 표시**해 준다.
절차적인 방법으로 먼저 구현한 방법이다.

```tsx
// 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for(var i = 0; i < buy_buttons.length; i++) {
    var button = buy_buttons[i];
    var item = button.item;
    
    // 버튼의 노출 여부 결정하는 로직
    if(item.price + shopping_cart_total >= 20)
      button.show_free_shipping_icon();
    else
      button.hide_free_shipping_icon();
  }
}

// 기존 함수 아래에 추가
function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  set_cart_total_dom();
  update_shipping_icons();
}
```

장바구니의 합계에 따른 세금 계산

```tsx
// 세금을 계산해주는 함수
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.10);
}

// 기존 함수 아래에 추가
function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
```

### 지금 코드는 비지니스 로직을 테스트하기가 어렵다.

1. 브라우저 설정하기
2. 페이지 로드하기
3. 장바구니에 제품 담기 버튼 클릭
4. DOM이 업데이트될 때까지 기다리기
5. DOM에서 값 가져오기
6. 가져온 문자열 값을 숫자로 바꾸기
7. 예상하는 값과 비교하기

**🔥 개선 방안**

- **DOM 업데이트**와 **비지니스 로직**은 분리되어야 한다.
- **전역변수**가 없어야 한다.

지금까지 만든 코드는 **모든 것이 액션**으로 이뤄져있다.

### 함수에는 입력과 출력이 있다.

입력과 출력은 **명시적이거나 암묵적**일 수 있는데, **암묵적 입력과 출력**이 있으면 액션이 된다.

```tsx
var total = 0;

function add_to_total(amount) {
  console.log("Old total: " + total); // 암묵적 입력 -> 전역변수를 읽기 때문
  total += amount; // 암묵적 출력 -> 전역변수를 바꾸기 때문
  return total; 
}
```

## 액션에서 계산을 빼내자

암묵적 입력과 출력이 가득한 코드이기 때문에 리팩토링을 진행해야 한다.

```tsx
// 카트 금액을 변경시키는 함수
function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  // 금액 합계 반영을 위한 DOM 업데이트
  set_cart_total_dom();
}

function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart);
  // 금액 합계 반영을 위한 DOM 업데이트
  set_cart_total_dom();
}

// 암묵적 입력과 출력을 모두 없애면서 리팩토링을 진행한 함수
function calc_total(cart: Cart) {
  // 책의 코드와는 다릅니다.
	const total = cart.reduce((accumulator, currentValue) => {
		return acc + currentValue.price;
	}, 0);
	return total;
}
```

**`add_item_to_cart()`함수**도 계산을 뽑아내어 리팩토링을 진행하자

전에 말했던 copy-on-write를 사용하는 방식이다.

```tsx
function add_item_to_cart(name: string, price: number) {
  shopping_cart.push({
    name: name,
    price: price
  });
  calc_cart_total();
}

function add_item_to_cart(name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

// 책에서는 slice()를 사용했다.
function add_item(cart: Cart, name: string, price: number) {
	const new_cart = cart.slice();
	new_cart.push({ name, price });
	
	return new_cart;
}

// 나였다면..? 얕은 복사로도 가능한 수준이라 이렇게 했을 듯 하다.
function add_item(cart: Cart, name: string, price: number) {
	const new_cart = [...cart, { name, price }];
	return new_cart;
}
```

**세금을 계산하는 함수**도 리팩토링을 진행하자

```tsx
// 세금을 계산해주는 함수
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.10);
}

// 세금을 계산해주는 함수
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

// 0.10도 상수로 분리하면 더 좋을 듯 하다.
const TAX_FEE = 0.10;
function calc_tax(amount: number) { 
	return amount * TAX_FEE;
}
```

updating_shipping_icons() 함수도 계산을 추출해보자.

```tsx
// 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  for(var i = 0; i < buy_buttons.length; i++) {
    var button = buy_buttons[i];
    var item = button.item;
    
    // 버튼의 노출 여부 결정하는 로직
    if(item.price + shopping_cart_total >= 20)
      button.show_free_shipping_icon();
    else
      button.hide_free_shipping_icon();
  }
}

// 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  const buy_buttons = get_buy_buttons_dom();
  buy_buttons.forEach((buy_button) => {
    const { item } = buy_button;
    
    // 버튼의 노출 여부 결정하는 로직
    if(gets_free_shipping(shopping_cart_total, item.price)) {
      buy_button.show_free_shipping_icon();
    } else {
      buy_button.hide_free_shipping_icon();
    }
  })
}

const FREE_SHIPPING_AMOUNT = 20;
function gets_free_shipping(total: number, item_price: number) {
	return item_price + total >= FREE_SHIPPING_AMOUNT;
}
```

최종적으로 리팩토링 된 함수

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
  // 책의 코드와는 다릅니다.
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

(전역 변수들도 사용하지 않게 바꾸고 싶다..ㅎ)