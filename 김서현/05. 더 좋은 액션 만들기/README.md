# 05. 더 좋은 액션 만들기

- [암묵적 입력과 출력은 줄이기](#암묵적-입력과-출력은-줄이기)
- [엉켜있는 코드 풀기](#엉켜있는-코드-풀기)

## 암묵적 입력과 출력은 줄이기

- 비즈니스 요구 사항과 맞추기: 합계 금액과 제품 가격에 대한 무료 배송 여부가 아니고 주문 결과가 무료 배송인지 확인해야
- 암묵적 입력과 출력을 줄이고 인자로 받는 식으로 변경 - 전역변수 사용 안 함

```tsx
// 변경 가능한 전역 변수 - 액션
var shopping_cart = [];
// var shopping_cart_total = 0;

// ! 계산 - cart, item에 대한 동작
function add_item(cart, name, price) {
  var new_cart = cart.slice();
  new_cart.push({ name: name, price: price });
  return new_cart;
}

// 액션
function add_item_to_cart(name, price) {
  shopping_cart = add_item(shopping_cart, name, price);
  // calc_cart_total();
  // calc_cart_total(shopping_cart); // ? 인자로 받음
  // ? calc_cart_total(cart) 내부에 있던 함수들을 여기로 옮김
  var total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}

// function get_free_shipping(total, item_price) {
//   return item_price + total >= 20;
// }

// ! 계산 - 비즈니스 규칙
function gets_free_shipping(cart) {
  return calc_total(cart) >= 20; // ? 별도의 계산 함수 사용
}

// 액션
// ? 전역변수 대신 인자 사용
function update_shipping_icons(cart) {
  var buttons = get_buy_buttons_dom();
  for (var i = 0; i < buttons.length; i++) {
    var button = buttons[i];
    var item = button.item;

    // ? 추가할 제품이 들어있는 새 장바구니 만듦
    var new_cart = add_item(cart, item.name, item.price);
    // if (get_free_shipping(shopping_cart_total, item.price)) {
    if (gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// ! 계산 - 비즈니스 규칙
function calc_tax(amount) {
  return amount * 0.1;
}

// function update_tax_dom() {
//   set_tax_dom(calc_tax(shopping_cart_total));
// }

// 액션
// ? 인자로 받음
function update_tax_dom(total) {
  set_tax_dom(calc_tax(total));
}

// ! 계산 - cart, item에 대한 동작, 비즈니스 규칙
function calc_total(cart) {
  var total = 0;
  for (var i = 0; i < cart.length; i++) {
    var item = cart[i];
    total += item.price;
  }
  return total;
}

// ? 추가됨
function set_cart_total_dom(total) {
  //...
  total;
  //...
}

// 액션
// ? 전역 변수 쓰던 걸 다 인자로 전달
// ? add_item_to_cart() 내부로 옮김
// function calc_cart_total(cart) {
//   var total = calc_total(cart);
//   set_cart_total_dom(total);
//   update_shipping_icons(cart);
//   update_tax_dom(total);
//   shopping_cart_total = total;
// }
```

---

## 엉켜있는 코드 풀기

- 함수 분리의 장점
  - 재사용하기 쉬움
  - 유지보수하기 쉬움
  - 테스트하기 쉬움
- 각 함수가 하나의 일만 하도록 분리하기

```tsx
// 변경 가능한 전역 변수 - 액션
var shopping_cart = [];

// function add_item(cart, name, price) {
//   // 1. 배열 복사
//   var new_cart = cart.slice();
//   // 2. item 객체 만듦
//   // 3. 복사본에 item 추가
//   new_cart.push({ name: name, price: price });
//   // 4. 복사본 리턴
//   return new_cart;
// }

// ! 계산 - item에 대한 동작
// ? item에 관한 코드를 별도의 함수로 분리
function make_cart_item(name, price) {
  // 2. item 객체 만듦
  return {
    name: name,
    price: price,
  };
}

// ! 계산 - 배열 유틸리티
// 일반적인 함수이기에 이름도 일반적인 이름으로 변경
function add_element_last(array, elem) {
  // ? 1, 3, 4는 값을 바꿀 때 복사하는 카피-온-라이트를 구현한 부분이기에 함께 두는 게 좋음
  // 1. 배열 복사
  var new_array = array.slice();
  // 3. 복사본에 item 추가
  new_array.push(elem);
  // 4. 복사본 리턴
  return new_array;
}

// ! 계산 - cart에 대한 동작
// ? 일반적인 이름으로 변경해 분리한 add_element_last 호출
function add_item(cart, item) {
  return add_element_last(cart, item);
}

// 액션 - 전역 변수
function add_item_to_cart(name, price) {
  // var shopping_cart = add_item(shopping_cart, name, price);
  // ? make_cart_item 호출해서 넘겨줌
  var item = make_cart_item(name, price);
  // 전역 변수 읽기는 액션
  shopping_cart = add_item(shopping_cart, item);

  var total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}

// ! 계산 - 비즈니스 규칙
function gets_free_shipping(cart) {
  return calc_total(cart) >= 20;
}

// 액션 - DOM 수정
function update_shipping_icons(cart) {
  var buttons = get_buy_buttons_dom();
  for (var i = 0; i < buttons.length; i++) {
    var button = buttons[i];
    var item = button.item;
    var new_cart = add_item(cart, item);
    if (gets_free_shipping(new_cart)) {
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  }
}

// ! 계산 - 비즈니스 규칙
function calc_tax(amount) {
  return amount * 0.1;
}

// 액션 - DOM 수정
function update_tax_dom(total) {
  set_tax_dom(calc_tax(total));
}

// ! 계산 - cart, item에 대한 동작, 비즈니스 규칙
function calc_total(cart) {
  var total = 0;
  for (var i = 0; i < cart.length; i++) {
    var item = cart[i];
    total += item.price;
  }
  return total;
}

function set_cart_total_dom(total) {
  //...
  total;
  //...
}
```

- update_shipping_icons 함수 분리하기

  - 함수
    ```tsx
    function update_shipping_icons(cart) {
      // 1. 모든 버튼을 가져오기
      var buttons = get_buy_buttons_dom();
      // 2. 버튼을 가지고 반복하기
      for (var i = 0; i < buttons.length; i++) {
        var button = buttons[i];
        // 3. 버튼에 관련된 제품들 가져오기
        var item = button.item;
        // 4. 가져온 제품으로 새 장바구니 만들기
        var new_cart = add_item(cart, item);
        // 5. 장바구니가 무료 배송이 필요한지 확인하기
        // 6. 아이콘 표시하거나 감추기
        if (gets_free_shipping(new_cart)) {
          button.show_free_shipping_icon();
        } else {
          button.hide_free_shipping_icon();
        }
      }
    }
    ```
  - 예시 방안

    ```tsx
    // ? 구매하기 버튼 관련 동작
    function update_shipping_icons(cart) {
      // 1. 모든 버튼을 가져오기
      var buy_buttons = get_buy_buttons_dom();
      // 2. 버튼을 가지고 반복하기
      for (var i = 0; i < buy_buttons.length; i++) {
        var button = buy_buttons[i];
        // 3. 버튼에 관련된 제품들 가져오기
        var item = button.item;
        var hasFreeShipping = gets_free_shipping_with_item(cart, item);
        set_free_shipping_icon(button, hasFreeShipping);
      }
    }

    // ? cart와 item 관련 동작
    function gets_free_shipping_with_item(cart, item) {
      // 4. 가져온 제품으로 새 장바구니 만들기
      var new_cart = add_item(cart, item);
      // 5. 장바구니가 무료 배송이 필요한지 확인하기
      return gets_free_shipping(new_cart);
    }

    // ? DOM 관련 동작
    function set_free_shipping_icon(button, isShown) {
      // 6. 아이콘 표시하거나 감추기
      if (isShown) button.show_free_shipping_icon();
      else button.hide_free_shipping_icon();
    }
    ```
