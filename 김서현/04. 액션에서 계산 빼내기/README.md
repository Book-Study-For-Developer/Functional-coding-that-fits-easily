# 04. 액션에서 계산 빼내기

- [절차형 코드로 본 장바구니 서비스](#절차형-코드로-본-장바구니-서비스)
- [함수형 코드로 리팩토링하기: 액션에서 계산 빼내기](#함수형-코드로-리팩토링하기-액션에서-계산-빼내기)

## 절차형 코드로 본 장바구니 서비스

- 장바구니에 담겨 있는 금액 합계 볼 수 있음
- 추가된 명세
    - 구매 합계가 20달러 이상이면 무료 배송
    - 세금 계산하기

```jsx
// 변경 가능한 전역 변수 - 액션
var shopping_cart = [];
var shopping_cart_total = 0;

// 장바구니에 물건 담기 - 전역 변수 바꾸는 액션
function add_item_to_cart(name, price) {
  shopping_cart.push({ name: name, price: price });
  calc_cart_total(); // 장바구니 제품 바뀌었기에 합계 업데이트
}

// 추가 명세 1: 합계 금액이 20달러 이상이면 무료 배송 아이콘 표시 - DOM에서 읽는 액션
function update_shipping_icons() {
  var buy_buttons = get_buy_buttons_dom(); // 모든 구매 버튼 DOM
  for (var i = 0; i < buy_buttons.length; i++) {
    var button = buy_buttons[i];
    var item = button.item;

    // 무료 배송 가능 여부에 따라 버튼 보여주거나 보여주지 않음
    if (item.price + shopping_cart_total >= 20) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// 추가 명세 2: 세금 계산 - DOM 바꾸는 액션
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1);
}

// 장바구니 합계 금액 계산 - 전역변수 바꾸는 액션
function calc_cart_total() {
  shopping_cart_total = 0;
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price; // 모든 제품값 더하기
  }

  set_cart_total_dom(); // DOM에 금액 합계 업데이트
  update_shipping_icons() // 추가 명세 1. 아이콘 업데이트
  update_tax_dom(); // 추가 명세 2. 세금 계산
}
```

### 🍭 위 코드의 문제점

- 비즈니스 규칙을 테스트하기 어려움
    - 결제팀, 배송팀은 전역변수에 접근하는 게 아니라 데이터베이스에서 장바구니 정보 읽어 와야 함
    - 결과 보여주기 위해 DOM을 직접 바꾸고 있지만, 결제팀은 영수증을, 배송팀은 운송장을 출력해야

### 🍭 개선점

- DOM 업데이트`[암묵적 출력]`와 비즈니스 규칙은 분리되어야 (DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안 됨)
- 전역 변수가 없어야 (전역변수에 의존하지 않아야) - 전역 변수 읽기`[암묵적 입력]`는 함수 인자로 바꾸면 됨, 전역 변수 변경`[암묵적 출력]`하는 동작은 함수 리턴값으로 바꾸면 됨
- 함수가 결괏값을 리턴해야 `[명시적 출력]`
- 암묵적 입력(인자 외 다른 입력)과 압묵적 출력(리턴값 외 다른 출력)이 있으면 액션이 됨 → 명시적 입력(인자)과 명시적 출력(리턴값)이 있으면 계산이 됨

<br />

## 함수형 코드로 리팩토링하기: 액션에서 계산 빼내기

서브루틴 추출하기. 기존 코드에서 동작 바꾸지 않은 채 코드는 고침

```jsx
// 변경 가능한 전역 변수 - 액션
var shopping_cart = [];
var shopping_cart_total = 0;

// ! 계산
function add_item(cart, name, price) {
  // 전역 변수 대신 인자인 cart 사용
  // 직접 배열을 바꾸는 대신 복사본을 만들어 지역 변수에 할당
  var new_cart = cart.slice();
  // 복사본 변경해 복사본을 리턴
  new_cart.push({ name: name, price: price });
  return new_cart;
}

// 장바구니에 물건 담기 - 전역 변수 바꾸는 액션
function add_item_to_cart(name, price) {
  shopping_cart = add_item(shopping_cart, name, price);
  calc_cart_total();
}

// ! 계산
function get_free_shipping(total, item_price) {
  return item_price + total >= 20;
}

// DOM에서 읽는, 전역 변수 읽는 액션
function update_shipping_icons() {
  var buttons = get_buy_buttons_dom();
  for (var i = 0; i < buttons.length; i++) {
    var button = buttons[i];
    var item = button.item;

    if (get_free_shipping(shopping_cart_total, item.price)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// ! 계산
function calc_tax(amount) {
  return amount * 0.1;
}

// DOM 바꾸는, 전역 변수 읽는 액션
function update_tax_dom() {
  set_tax_dom(calc_tax(shopping_cart_total));
}

// ! 계산. 전역변수 대신 인자 사용
function calc_total(cart) {
  var total = 0; // 지역 변수를 사용하고 리턴
  for (var i = 0; i < cart.length; i++) {
    var item = cart[i];
    total += item.price;
  }
  return total;
}

// 전역변수 읽는 액션
function calc_cart_total() {
  shopping_cart_total = calc_total(shopping_cart); // 리턴값을 받아 전역 변수에 할당
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
```

### 🍭 단계별 계산 추출

1. 계산 코드를 찾아 빼냄 - 새로운 함수로 만들고 원래 코드에서 빼낸 부분에 만든 함수를 부르도록 바꿈
2. 새 함수에 암묵적 입력과 출력을 찾음
    1. 암묵적 입력: 함수를 부르는 동안 결과에 영향을 줄 수 있는 것
        - 입력 - 인자를 포함한 함수 밖에 있는 변수 읽기, DB에서 값 가져오기
    2. 암묵적 출력: 함수 호출의 결과로 영향을 받는 것
        - 출력 - 리턴값 포함해 전역변수 바꿈, 공유 객체 바꿈, 웹 요청 보냄
3. 압묵적 입력은 인자로, 암묵적 출력은 리턴값으로 변경
    - 인자와 리턴값은 바뀌지 않는 불변값임
