## 6장 목표

- 데이터가 바뀌지 않도록 하기 위해 `카피-온-라이트` 적용하기
- 배열과 객체를 데이터에 쓸 수 있는 `카피-온-라이트` 동작 만들기
- 깊이 중첩된 데이터도 `카피-온-라이트`가 잘 동작하게 만들기

### 모든 동작을 불변형으로 만들 수 있을까?

제품 추가할 때 `slice` 메서드를 활용해서 복사본을 만들어 값을 바꾸어 리턴하는 **카피-온-라이트**를 사용했다.

하지만 [제품 이름으로 제품 구매 수량 바꾸기]는 쉽지 않아 보인다. 장바구니 배열 안에 제품 객체의 정보를 바꿔야 하기 때문이다. - 중첩된 데이터

> [!NOTE]
> 데이터 구조 안에 데이터 구조가 있는 경우를 말한다. 객체 안에 객체가 있든, 객체 안에 배열이 있든, 배열 안에 객체가 있든 모두 중첩된 데이터다.

### 동작을 읽기 또는 쓰기 또는 둘 다로 분류하기

어려운 중첩된 데이터에 대해 불변 동작을 구현하기 위해 동작을 1. 읽기, 2. 쓰기, 3. 읽기와 쓰기로 분류해볼 수 있다.

1. 읽기
   - 데이터를 바꾸지 않고 정보를 꺼내기
   - 데이터가 바뀌지 않아서 다루기 쉽다.
2. 쓰기
   - 읽기를 제외한 모든 동작
   - 어떻게든 데이터를 바꾼다
   - 바뀌는 값은 어디서 사용될지 모르기 때문에 바뀌지 않도록 원칙이 필요하다.

| 장바구니 동작                       | 분류 |
| ----------------------------------- | ---- |
| 제품 개수 가져오기                  | 읽기 |
| 제품 이름으로 제품 가져오기         | 읽기 |
| 제품 추가하기                       | 쓰기 |
| 제품 이름으로 제품 빼기             | 쓰기 |
| 제품 이름으로 제품 구매 수량 바꾸기 | 쓰기 |

`쓰기` 동작은 불변성 원칙에 따라 구현해야 한다. 하스켈(Haskell)이나 클로저(Clojure) 같은 언어는 카피-온-라이트를 쓰고 있지만, 자바스크립트는 기본적으로 변경 가능한 데이터 구조를 사용하기 때문에 `🚨직접 구현이 필요하다.🚨`

### 카피-온-라이트 원칙 세 단계

1. 복사본 만들기
2. 복사본 변경하기(원하는 만큼)
3. 복사본 리턴하기

```ts
function add_element_last(array, elem) {
  const new_array = array.slice(); // 1. 복사본 만들기
  new_array.push(elem); // 2. 복사본 변경하기
  return new_array; // 3. 복사본 리턴하기
}
```

위 코드의 동작은 다음과 같다.

1. 먼저 배열을 복사했고 기존 배열은 변경하지 않았다.
2. 복사본은 함수 범위에 있기 때문에 다른 코드에서 값을 바꾸기 위해 접근할 수 없다.
3. 복사본을 변경하고 나서 함수를 나간다. 이후에는 값을 바꿀 수 없다.

> [!NOTE]
> add_element_last 함수는 데이터를 바꾸지 않았고 정보를 리턴했기 때문에 읽기다!

### 카피-온-라이트로 쓰기를 읽기로 바꾸기

```ts
// 장바구니를 바꾸는 다른 동작
function remove_item_by_name(cart, name) {
  let idx = null;
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) idx = i;
  }
  if (idx !== null) cart.splice(idx, 1); // 장바구니를 변경
}
```

> [!WARNING]
> splice 메서드는 배열에서 항목을 삭제하는 메서드인데 함수 실행으로 장바구니 자체를 변경시킨다. 장바구니를 변경 불가능한 데이터로 쓰기 위해서는 slice 메서드를 활용해서 복사본을 만들어야 한다.

```ts
function remove_item_by_name(cart, name) {
  const new_cart = cart.slice();
  let idx = null;
  for (let i = 0; i < new_cart.length; i++) {
    if (new_cart[i].name === name) idx = i;
  }
  if (idx !== null) new_cart.splice(idx, 1);
  return new_cart;
}

// 전역변수의 변경을 remove_item_by_name 내에서 하지 않도록 바꿨기 때문에 실행 후 리턴값을 할당해야 한다.
function delete_handler(name) {
  shopping_cart = remove_item_by_name(shopping_cart, name);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}
```

> [!TIP]
> 값을 복사해서 변경하고 리턴하는 패턴은 일반적인 카피-온-라이트이므로 일반화한 함수를 만들어두면 여러 곳에서 재사용할 수 있다.

### 연습 문제 - 메일링 리스트에 연락처를 추가하는 코드

```ts
const mailing_list = [];

function add_contact(email: string) {
  mailing_list.push(email);
}

function submit_form_handler(event: Event) {
  const form = event.target;
  const email = form.elements["email"].value;
  add_contact(email);
}
```

위 코드를 카피-온-라이트 형식으로 바꿔보면,

```ts
const mailing_list = []; // 전역 변수

function add_contact(mailing_list, email) {
  const new_mailing_list = mailing_list.slice();
  new_mailing_list.push(email);
  return new_mailing_list;
}

function submit_form_handler(event: Event) {
  const form = event.target;
  const email = form.elements["email"].value;
  mailing_list = add_contact(mailing_list, email);
}
```

### 쓰기를 하면서 읽기도 하는 동작은 어떻게 해야 할까?

shift와 같은 메서드는 배열의 가장 앞에 있는 값을 반환도 하면서 실제 배열을 변경하기도 한다.

이 때는 `1. 읽기와 쓰기 함수로 각각 분리`하는 방법과 `2. 함수에서 값을 두 개 리턴`하는 방법으로 접근해볼 수 있다.

### 1. 읽기와 쓰기 함수로 각각 분리하기

1. 읽기와 쓰기 동작으로 분리하기

```ts
// 읽기 동작
function first_element(array) {
  return array[0];
}

// 쓰기 동작: 반환하는 값 무시
function drop_first(array) {
  array.shift();
}
```

2. 쓰기 동작을 카피-온-라이트로 바꾸기

```ts
function drop_first(array) {
  const array_copy = array.slice();
  array_copy.shift();
  return array_copy;
}
```

### 2. 함수에서 값을 두 개 리턴하기

1. 동작을 감싸기: .shift() 메서드를 바꿀 수 있도록 새로운 함수로 감싸기

```ts
function shift(array) {
  return array.shift();
}
```

2. 읽으면서 쓰기도 하는 함수를 읽기 함수로 바꾸기

```ts
function shift(array) {
  const array_copy = array.slice();
  const first = array_copy.shift();
  // 값 두 개 리턴을 위해 객체를 사용
  return {
    first: first,
    array: array_copy,
  };
}
```

이렇게 하면, shift의 인자로 들어간 array의 원본은 변경하지 않으면서(불변을 지키면서) 원래 .shift() 메서드의 읽기 동작과 쓰기 동작을 모두 반환값으로 얻을 수 있다.

### 연습문제 - .pop() 메서드를 카피-온-라이트 버전으로 만들기

```ts
// 읽기 동작
function get_last_element(array) {
  return array[array.length - 1];
}

// 쓰기 동작
function drop_last(array) {
  const array_copy = array.slice();
  array_copy.pop();
  return array_copy;
}

// 값 두 개를 리턴하는 함수 만들기
function pop(array) {
  return {
    last: get_last_element(array),
    array: drop_last(array),
  };
}
```

---

### 불변 데이터 구조를 읽는 것은 계산이다.

- **변경 가능한 데이터를 읽는 것은 액션이다.** 대표적인 예로는 어디에서 바뀔 지 예측할 수 없는 전역 변수를 읽는 것과 같다.
- **쓰기는 데이터를 변경 가능한 구조로 만든다.**
- **어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터이다.**
- **불변 데이터 구조를 읽는 것은 계산이다.**
- **쓰기를 읽기로 바꾸면 코드에 계산이 많아진다.** 데이터 구조를 불변형으로 만들수록 계산이 많아지고 액션이 줄어든다.

### 하지만 데이터가 모두 불변형이면 시간에 따라 변하는 상태는 어떻게 다루지?

결국 애플리케이션은 시간에 따라 변하는 상태가 있고 우리가 다루는 예제에서는 `shopping_cart`라는 전역 변수가 그러하다.

카피-온-라이트 원칙을 적용하지 않았을 때는 `shopping_cart` 전역 변수를 직접 변경하는 경우가 많았지만 함수를 쓰기에서 읽기로 바꾸면서 `🚨교체🚨`하는 방식을 적용하고 있다.

### 근데 불변 데이터 구조는 변경 가능한 데이터 구조보다 메모리를 더 쓰고 많이 느리지 않나?

반박하는 주장으로,

1. 언제든 최적화할 수 있다.
   - 미리 계획해서 최적화하기보다는, 불변 데이터 구조를 사용하고 속도가 느린 부분이 있다면 그 때 최적화하자.
2. 가비지 콜렉터는 매우 빠르다.
   - 가비지 콜렉터 성능은 꽤나 개선됐기 때문에 잘 쓰면 된다.
3. 생각보다 많이 복사하지 않는다.
   - 데이터 구조의 최상위 단계만 복사하는 **얕은 복사(혹은 구조적 공유)**는 참조만 복사된다.
4. 함수형 프로그래밍 언어에는 빠른 구현체가 있다.
   - 불변 데이터 구조를 지원하는 언어에서는 데이터 구조를 복사할 때 최대한 많은 구조를 공유한다. 그래서 더 적은 메모리를 사용하고 결국 가비지 콜렉터의 부담을 줄인다.

### 이번에는 배열이 아닌 객체에 대한 카피-온-라이트

`Object.assign()` 메서드를 활용하면 객체의 모든 키와 값을 복사할 수 있다.

```ts
// 카피-온-라이트가 적용된 setPrice 함수
function setPrice(item, new_price) {
   const item_copy = Object,assign({}, item);
   item_copy.price = new_price;
   return item_copy;
}
```

> [!NOTE]
> 얕은 복사(shallow copy)는 중첩된 데이터 구조에서 최상위 데이터만 복사한다. 예를 들어 객체가 나열된 배열이라면 얕은 복사는 배열만 복사하고 객체는 참조로 공유한다. 다른 말로는 구조적 공유라고도 하는데, 메모리를 적게 사용하고 모든 것을 복사하는 것보다 빠르다.

### 중첩된 쓰기를 읽기로 바꾸기

```ts
// 원래 코드
function setPriceByName(cart, name, price) {
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      cart[i].price = price; // 불변성이 깨졌다.
    }
  }
}

// 카피-온-라이트 버전
function setPriceByName(cart, name, price) {
  var cartCopy = cart.slice();
  for (let i = 0; i < cartCopy.length; i++) {
    if (cartCopy[i].name === name) {
      cartCopy[i] = setPrice(cartCopy[i], price); // 객체에 대한 카피-온-라이트 적용하기
    }
  }
  return cartCopy;
}
```

> [!WARNING]
> 하지만 위 코드에서 `cartCopy[i]`에 할당하여 직접 변경하는 부분에서 결국 불변성이 깨졌다고 볼 수 있다. `🚨최하위부터 최상위까지 중첩된 데이터 구조의 모든 부분이 불변형이어야 한다.🚨`

### 어떤 복사본이 생겼을까?

```ts
// 카피-온-라이트 버전
function setPriceByName(cart, name, price) {
   var cartCopy = cart.slice();
   for (let i = 0; i < cartCopy.length; i++) {
      if (cartCopy[i].name === name) {
         cartCopy[i] = setPrice(cartCopy[i], price); // 객체에 대한 카피-온-라이트 적용하기
      }
   }
   return cartCopy;
}

function setPrice(item, new_price) {
   const item_copy = Object,assign({}, item);
   item_copy.price = new_price;
   return item_copy;
}
```

장바구니 안에 제품이 세 개 있다고 가정하면 하나의 배열(장바구니) 안에 객체 세 개(티셔츠, 신발, 양말)의 중첩된 데이터라고 할 수 있다.

티셔츠 가격을 바꾸기 위해 `shopping_cart = setPriceByName(shopping_cart, "t-shirt", 13);`을 실행하면 복사본은 **배열 하나와 객체 하나이다.**

### 왜 나머지 두 개의 객체는 복사하지 않았을까?

얕은 복사란 최상위 데이터 구조만 복사했으므로 slice 메서드를 활용하면 배열(장바구니)은 복사했지만, 복사된 배열의 객체 원소는 원래 객체를 가리킨다.

setPrice 함수는 티셔츠 객체의 얕은 복사본을 만들어서 가격을 변경한다. 그리고 `cartCopy[i]`에 할당하면서 직접 값을 할당한다. 반면, 나머지 두 객체는 변경하지도 복사하지도 않았기 때문에 **구조적 공유**를 하고 있다고 볼 수 있다.

---

### 느낀 점

- 배열이나 객체를 쓰는 동작으로 바꾸지 않기 위해 읽기 함수를 만든다는 게 어려운 일인 것 같다.
- 개인적으로, 불변성을 지키기 때문에 무엇이 좋은지 완전히 받아들이기 어려운 상태다.
- 적어도, 객체에 대한 카피-온-라이트 패턴을 사용하는 것은 오랜 시간이 걸릴 것 같다. 왜냐하면 프로퍼티에 쓰기 동작은 겨우 한 줄인데, 읽기 함수를 만듦으로써 5줄을 더 작성해야 한다.
