## 동작을 읽기, 쓰기 또는 둘 다로 분류하기

- 읽기
  - 데이터에서 정보를 가져옴
  - 데이터를 바꾸지 않음
- 쓰기
  - 데이터를 바꿈

장바구니 동작

```tsx
1. 제품 개수 가져오기 - 읽기
2. 제품 이름으로 제품 가져오기 - 읽기
3. 제품 추가하기 - 쓰기
4. 제품 이름으로 제품 빼기 - 쓰기
5. 제품 이름으로 제품 구매 수량 바꾸기 - 쓰기
```

쓰기 동작은 불변성의 원칙에 따라 구현해야 함

→ **카피-온-라이트**

제품에 대한 동작

```tsx
1. 가격 설정하기 - 쓰기
2. 가격 가져오기 - 읽기
3. 이름 가져오기 - 읽기
```

## 카피-온-라이트 원칙 세단계

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

```tsx
function add_elemet_last(array, elem) {
  const new_array = array.slice(); // 1. 복사본 만들기
  new_array.push(elem); // 2. 복사본 변경하기
  return new_array; // 3. 복사본 리턴하기
}
```

⇒ 데이터를 바꾸지 않았고 정보를 리턴했기 때문에 쓰기임

## 카피-온-라이트로 쓰기를 읽기로 바꾸기

```tsx
// 장바구니에서 제품을 빼는 함수
function remove_item_by_name(cart, name) {
  let idx = null;
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      idx = i;
    }
  }
  if (idx !== null) {
    cart.splice(idx, 1); // idx위치에 있는 항목 한개 삭제
  }
}

// 카피-온-라이트 원칙 적용
function remove_item_by_name(cart, name) {
  const new_cart = cart.slice();
  let idx = null;
  for (let i = 0; i < new_cart.length; i++) {
    if (new_cart[i].name === name) {
      idx = i;
    }
  }
  if (idx !== null) {
    new_cart.splice(idx, 1);
    return new_cart;
  }
}

function delete_handler(name) {
  // remove_item_by_name함수를 사용하는 곳에서 전역변수를 변경
  shopping_cart = remove_item_by_name(shopping_cart, name);
  const total = calc_total(shopping_cart);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}
```

### 연습 문제

```tsx
const mailing_list = [];

function add_concat(email) {
  mailing_list.push(email);
}

function submit_form_handler(event) {
  const form = event.target;
  const email = form.elements["email"].value;
  add_concat(email);
}

// 리팩토링
const mailing_list = [];

function add_concat(mailing_list, email) {
  let list_copy = mailing_list.slice();
  list_copy.push(email);
  return copy_email;
}

function submit_form_handler(event) {
  let form = event.target;
  let email = form.elements["email"].value;
  mailing_list = add_concat(mailing_list, email);
}
```

## 쓰기를 하면서 읽기도 하는 동작

- 읽기와 쓰기 함수로 분리
- 함수에서 값을 두개 리턴

shift() 메서드

```tsx
function first_element(array) {
  return array[0];
}

function drop_first(array) {
  const array_copy = array.slice();
  array_copy = shift();
  return array_copy;
}

function shift(array) {
  return array.shift();
}

function shift(array) {
  const array_copy = array.slice();
  const first = array_copy.shift();
  return {
    first: first,
    array: array_copy,
  };
}
```

pop 메서드

```tsx
function last_element(array) {
  return array[array.length - 1];
}

function drop_least(array) {
  array.pop();
}

function drop_last(array) {
  const array_copy = array.slice();
  array_copy.pop();
  return array_copy;
}

function pop(array) {
  const array_copy = array.slice();
  const first = array_copy.pop();
  return {
    first: first,
    array: array_copy,
  };
}
```

## 불변 데이터 구조를 읽는 것은 계산입니다

- 변경 가능한 데이터를 읽는 것은 액션이다
- 쓰기는 데이터를 변경 가능한 구조로 만든다
- 어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터이다
- 불변 데이터 구조를 읽는 것은 계산이다
- 쓰기를 읽기로 바꾸면 코드에 계산이 많아진다.

## 불변 데이터 구조는 충분히 빠릅니다

- 언제든 최적화 가능
- 가비지 콜렉터는 매우 빠름
- 생각보다 많이 복사하지 않음
- 함수형 프로그래밍 언어에는 빠른 구현체가 있음

## 객체에 대한 카피-온-라이트

배열은 slice() 메서드로 복사본 만들 수 있음

객체는 Object.assign()으로 복사 구현

```tsx
function setPrice(item, new_price){
 CONST item_copy = Object.assign({}, item);
 item_copy.price = new_price;
 return item_copy;
}
```

## 중첩된 쓰기를 읽기로 바꾸기

```tsx
function setPriceByName(cart, name, price) {
  const cartCopy = cart.slice(); // 배열 복사
  for (let i = 0; i < cartCopy.length; i++) {
    if (cartCopy[i].name === name) {
      cartCopy[i] = setPrice(cartCopy[i], price);
    }
    return cartCopy;
  }
}

function setPrice(item, new_price) {
  const item_copy = Object.assign({}, item); // 객체 복사
  item_copy.price = new_price;
  return item_price;
}
```

복사본은 배열 한개와 객체 한개

나머지 객체 두개는 중첩된 데이터에 얕은 복사를 했기 때문에 그 결과가 구조적으로 공유 됨

## 요점 정리

- 함수형 프로그래밍에서 불변 데이터가 필요하며 계산에서는 변경 가능한 데이터에 쓰기를 할 수 없음
- 카피-온-라이트는 데이터를 불변형으로 유지할 수 있는 원칙임
- 복사본을 만들고 원본 대신 복사본을 변경 하는 것!
- 카피-온-라이트는 값을 변경하기 전에 얕은 복사를 하고 리턴함
- 기본적인 배열과 객체에 대한 동작은 카피-온-라이트 버전을 만들어 주는 것이 좋음
