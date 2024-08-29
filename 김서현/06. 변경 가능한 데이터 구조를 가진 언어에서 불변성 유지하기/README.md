# 06. 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기

- [쓰기와 읽기 동작](#쓰기와-읽기-동작)
- [카피-온-라이트](#카피-온-라이트)
  - [카피-온-라이트로 쓰기를 읽기로 바꾸기](#카피-온-라이트로-쓰기를-읽기로-바꾸기)
  - [카피-온-라이트로 쓰기와 읽기를 하는 동작을 읽기로 바꾸기](#카피-온-라이트로-쓰기와-읽기를-하는-동작을-읽기로-바꾸기)
  - [객체에 카피-온-라이트 적용하기](#객체에-카피-온-라이트-적용하기)
- [불변 데이터 구조](#불변-데이터-구조)
- [중첩된 쓰기를 읽기로 바꾸기](#중첩된-쓰기를-읽기로-바꾸기)

# 쓰기와 읽기 동작

- 동작
  - 읽기 - 데이터에서 정보를 가져옴. 데이터 바꾸지 않음
  - 쓰기 - 데이터 바꿈
    - 불변성 원칙에 따라 구현해야
- 장바구니와 제품에 대한 동작
  - 장바구니에 대한 동작
    1. 제품 개수 가져오기 - 읽기
    2. 제품 이름으로 제품 가져오기 - 읽기
    3. 제품 추가하기 - 읽기
    4. 제품 이름으로 제품 빼기 - 쓰기
    5. 제품 이름으로 제품 구매 수량 바꾸기 - 쓰기 ← 중첩된 데이터에 대한 동작
  - 제품에 대한 동작
    1. 가격 설정하기 - 쓰기
    2. 가격 가져오기 - 읽기
    3. 이름 가져오기 - 읽기

<br />
 
# 카피-온-라이트

- : 데이터를 불변형으로 유지할 수 있는 원칙. 복사본을 만들고 원본 대신 복사본을 변경함
- 카피-온-라이트 원칙 세 단계
  1. 복사본 만들기
  2. 복사본 변경하기(원하는 만큼)
  3. 복사본 리턴하기


<br />
 
## 카피-온-라이트로 쓰기를 읽기로 바꾸기

- 장바구니에 대한 동작

  ```tsx
  // ? 기존 코드
  function remove_item_by_name(cart, name) {
    var idx = null;
    for (var i = 0; i < cart.length; i++) {
      if (cart[i].name === name) idx = i;
    }
    if (idx !== null) cart.splice(idx, 1);
  }

  function delete_handler(name) {
    // 전역변수 변경
    remove_item_by_name(shopping_cart, name);
    var total = calc_total(shopping_cart);
    set_cart_total_dom(total);
    update_shipping_icons(shopping_cart);
    update_tax_dom(total);
  }

  // ? 카피-온-라이트 적용한 코드
  function remove_item_by_name(cart, name) {
    // 1. 복사본 만들기
    var new_cart = cart.slice();
    var idx = null;
    // 2. 복사본 변경하기
    for (var i = 0; i < new_cart.length; i++) {
      if (new_cart[i].name === name) idx = i;
    }
    if (idx !== null) new_cart.splice(idx, 1);
    // 3. 복사본 리턴하기
    return new_cart;
  }

  function delete_handler(name) {
    // 함수 사용처에서 전역변수 변경
    shopping_cart = remove_item_by_name(shopping_cart, name);
    var total = calc_total(shopping_cart);
    set_cart_total_dom(total);
    update_shipping_icons(shopping_cart);
    update_tax_dom(total);
  }
  ```

- 재사용 쉽도록 일반화 가능한 코드를 카피-온-라이트 적용해 분리하기

  ```tsx
  // ? 기존 코드
  function removeItems(array, idx, count) {
    array.splice(idx, count);
  }

  function remove_item_by_name(cart, name) {
    var new_cart = cart.slice();
    var idx = null;
    for (var i = 0; i < new_cart.length; i++) {
      if (new_cart[i].name === name) idx = i;
    }
    if (idx !== null) new_cart.removeItems(idx, 1);
    return new_cart;
  }

  // ? Copy-on-write를 splice 부분에 적용
  function removeItems(array, idx, count) {
    var copy = array.slice();
    copy.splice(idx, count);
    return copy;
  }

  function remove_item_by_name(cart, name) {
    // ? removeItems이 배열을 복사하기에 여기에서 복사할 필요 없음
    var idx = null;
    for (var i = 0; i < cart.length; i++) {
      if (cart[i].name === name) idx = i;
    }
    if (idx !== null) return removeItems(cart, idx, 1);
    return cart;
  }
  ```

<br />
 
## 카피-온-라이트로 쓰기와 읽기를 하는 동작을 읽기로 바꾸기

- 두 가지 접근 방법

  1. 함수를 분리하기 - 읽기 함수, 쓰기 함수

     1. 읽기와 쓰기 동작으로 분리하기

        ```tsx
        // ? 배열 첫 번째 항목 읽기
        function first_element(array) {
          return array[0];
        }

        // ? 결괏값 리턴 없이 쓰기만 수행
        function drop_first(array) {
          array.shift();
        }
        ```

     2. 쓰기 동작을 카피-온-라이트로 바꾸기

        ```tsx
        function drop_first(array) {
          var array_copy = array.slice();
          array_copy.shift();
          return array_copy;
        }
        ```

  2. 함수에서 값을 두 개 리턴하기

     1. 동작을 감싸기

        ```tsx
        function shift(array) {
          return array.shift();
        }
        ```

     2. 읽으면서 쓰기도 하는 함수를 읽기 함수로 바꾸기

        ```tsx
        // 1안 - 인자 활용해 수행 후 지운 항목과 변경된 배열 함께 리턴
        function shift(array) {
          var array_copy = array.slice();
          var first = array_copy.shift();
          return {
            first: first, // 지운 첫 번째 항목
            array: array_copy, // 변경된 배열
          };
        }

        // 2안 - 함수를 분리하기 방식 활용해 두 값을 객체로 조합
        function shift(array) {
          return {
            first: first_element(array),
            array: drop_first(array),
          };
        }
        ```

<br />
 
## 객체에 카피-온-라이트 적용하기

```tsx
// 기존 코드
var a = { x: 1 };
delete a["x"];
a;

// Copy-on-write 적용 코드
function objectDelete(object, key) {
  var copy = Object.assign({}, object);
  delete copy[key];
  return copy;
}
```

<br />
 
# 불변 데이터 구조

- 쓰기를 읽기로 바꾸면 코드에 계산이 많아지고 액션은 줄어듦
  - 변경 가능한 데이터를 읽는 것 - 액션
  - 데이터를 변경 가능한 구조로 만드는 것 - 쓰기
  - 변경 불가능한 데이터: 쓰기가 없는 데이터
  - 불변 데이터 구조를 읽는 것 - 계산
- 불변 데이터 구조의 속도
  - 불변 데이터 구조는 변경 가능한 데이터 구조보다 메모리를 더 많이 쓰고 느림
  - 반박
    - 언제든 최적화 가능 - 일단 불변 데이터구조 사용하고 속도 느리면 그때 최적화
    - 가비지 콜렉터는 매우 빠름
    - 생각보다 복사 많이 안 함. 복사해도 얕은 복사(참조만 복사,구조적 공유)만 수행
    - 함수형 프로그래밍 언어에는 빠른 구현체가 있음

<br />
 
# 중첩된 쓰기를 읽기로 바꾸기

값을 바꿀 땐 복사본을 만들기에 공유된 값은 변경되지 않음

- 중첩된 데이터
  - 데이터 구조 안에 데이터 구조가 있는 경우. - 배열 안에 객체가 있는 경우 등
  - 깊이 중첩(deeply nested): 중첩이 이어짐. - 객체 안에 객체 안에 배열 안에 객체가 있는 등

```tsx
// 기존 코드
function setPrice(item, new_price) {
  item.price = new_price;
}

// Copy-on-write 적용 코드
function setPrice(item, new_price) {
  var item_copy = Object.assign({}, item); // 객체 복사
  item_copy.price = new_price; // 복사본 변경
  return item_copy; // 리턴
}
```

```tsx
// 기존 코드
function setPriceByName(cart, name, price) {
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      cart[i].price = price;
    }
  }
}

// Copy-on-write 적용 코드
function setPriceByName(cart, name, price) {
  // 배열 복사
  var cartCopy = cart.slice();
  for (var i = 0; i < cartCopy.length; i++) {
    // ? 중첩된 항목을 바꾸기 위해 카피-온-라이트 동작 부름
    if (cartCopy[i].name === name) {
      cartCopy[i] = setPrice(cartCopy[i], price);
    }
  }
  return cartCopy;
}
```
