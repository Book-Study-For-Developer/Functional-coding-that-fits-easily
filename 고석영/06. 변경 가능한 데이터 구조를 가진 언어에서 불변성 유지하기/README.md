## 동작을 읽기,쓰기 또는 둘 다로 분류하기

#### 읽기

- 데이터에서 정보를 가져오기
- 데이터를 바꾸지 않음

#### 쓰기

- 어떻게든 데이터를 바꿈
- 추가하기, 빼기, 바꾸기 등

## 카피-온-라이트 원칙 세 단계

> [!NOTE]
> 카피 온 라이트는 쓰기를 읽기로 바꿉니다.

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

```ts
const add_element_last = <T>(array: T[], element: T) => [...array, element]
```

1. 배열을 복사했고 기존 배열은 변경하지 않음
2. 복사본은 함수 범위에 있기 때문에 다른 코드에서 값을 변경하기 위해 접근할 수 없음
3. 복사본 변경 후 함수를 나가므로(리턴) 이후에 값을 변경할 수 없음

> 데이터를 변경하지 않고 정보를 리턴하면 읽기!

### 카피-온-라이트로 쓰기를 읽기로 바꾸기

#### `remove_item_by_name()`에 카피 온 라이트 적용

```ts
const remove_item_by_name = (cart: Item[], name: string) => {
  let idx = null
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) idx = i
  }
  if (idx !== null) cart.splice(idx, 1)
}
```

#### 장바구니 복사하기

```ts
const remove_item_by_name = (cart: Item[], name: string) => {
  const new_cart = cart.slice()

  let idx = null
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) idx = i
  }
  if (idx !== null) cart.splice(idx, 1)
}
```

#### 복사본 사용하기

```ts
const remove_item_by_name = (cart: Item[], name: string) => {
  const new_cart = cart.slice()

  let idx = null
  for (let i = 0; i < new_cart.length; i++) {
    if (new_cart[i].name === name) idx = i
  }
  if (idx !== null) new_cart.splice(idx, 1)
}
```

#### 복사본 리턴하기

```ts
const remove_item_by_name = (cart: Item[], name: string) => {
  const new_cart = cart.slice()

  let idx = null
  for (let i = 0; i < new_cart.length; i++) {
    if (new_cart[i].name === name) idx = i
  }
  if (idx !== null) new_cart.splice(idx, 1)

  return new_cart
}
```

#### 함수 호출 부분 변경하기

```ts
const delete_handler = (name: string) => {
  shopping_cart = remove_item_by_name(shopping_cart, name)
  const total = calc_total(shopping_cart)

  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)
}
```

#### (+) filter 메서드를 사용하면 이러지 않아도 된다..

```ts
const remove_item_by_name = (cart: Item[], name: string) => {
  return cart.filter(item => item.name !== name)
}
```

## 읽기도 하면서 쓰기도 하는 동작은 어떻게 해야 할까요?

어떤 동작은 읽고 변경하는 일을 동시에 한다. 이런 동작은 값을 변경하고 리턴한다.

- 읽기와 쓰기 함수로 각각 분리하기
- 값을 두 개 리턴하는 함수로 만들기

> 선택할 수 있다면 첫 번째 방법이 더 좋은 접근 방법이다. 책임이 확실히 분리되기 때문이다.

### 읽기 함수와 쓰기 함수로 분리하기

> [!NOTE]
> 읽기와 쓰기를 분리하는 접근 방법은 **분리된 함수를 따로 쓸 수 있기 때문에** 더 좋은 접근 방법
>
> - 필요한 것만 선택이 가능하기 때문!

1. 쓰기에서 읽기를 분리하기
2. 쓰리에 카피-온-라이트를 적용해 읽기로 바꾸기

### 배열에 대한 카피-온-라이트: `shift()`

#### 읽기와 쓰기 동작으로 분리하기

```ts
// 읽기
const first_element = array => array[0]

// 쓰기
const drop_first = array => {
  array.shift() // shift 메서드를 실행하고 결과 값 무시
}
```

#### 쓰기 동작을 카피 온 라이트로 바꾸기

```ts
const drop_first = array => {
  const array_copy = [...array]
  array_copy.shift()

  return array_copy
}
```

#### 값 두 개를 리턴하는 함수로 만들기

1. 먼저 동작을 새로운 함수로 감싸기
2. 읽기와 쓰기를 함께 하는 함수를 읽기만 하는 함수로 바꾸기

```ts
const shift = array => {
  const array_copy = [...array]
  const first = array_copy.shift()

  return {
    first: first,
    array: array_copy,
  }
}
```

#### 첫 번째 접근 방식을 사용해 두 값을 객체로 조합하기

```ts
const shift = array => {
  return {
    first: first_element(array),
    array: drop_first(array),
  }
}
```

### 배열에 대한 카피-온-라이트: `pop()`

#### 읽기 함수와 쓰기 함수로 분리하기

```ts
const last_element = array => array[array.length - 1]

const drop_last = array => {
  const array_copy = [...array]
  array_copy.pop()
  return array_copy
}
```

#### 값 두 개를 리턴하는 함수로 만들기

```ts
const pop = array => {
  const array_copy = [...array]
  const last = array_copy.pop()

  return {
    last: last,
    array: array_copy,
  }
}
```

### 배열에 대한 카피-온-라이트: `push()`

#### 카피-온-라이트 적용하기

```ts
const push = (array, element) => {
  const array_copy = [...array]
  array_copy.push(element)
  return array_copy
}
```

### 불변 데이터 구조를 읽는 것은 계산이다.

- 변경 가능한 데이터를 읽는 것은 액션이다.
- 쓰기는 데이터를 바꾸기 때문에 데이터를 변경 가능한 구조로 만든다.
- 어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터다.
- 불변 데이터 구조를 읽는 것은 계산이다.
- 데이터 구조를 불변형으로 만들수록 코드에 계산이 많아지고 액션은 줄어든다.

### 어플리케이션에는 시간에 따라 변하는 상태가 있다.

- `shopping_cart` 전역변수
  - 애플리케이션에서 시간에 따라 변하는 상태
  - 장바구니가 바뀔 때마다 새 값이 할당된다. 즉, 새 값으로 교체(swapping)된다.
- 교체: 읽기, 바꾸기, 쓰기

### 불변 데이터 구조는 충분히 빠릅니다

- 언제든 최적화 할 수 있다.
- 가비지 콜렉터는 매우 빠르다.
- 구조적 공유를 하는 얕은 복사를 하기 때문에 생각보다 많이 복사하지 않는다.
- 함수형 프로그래밍 언어엔 빠른 구현체가 있다.

### 객체의 카피-온-라이트

책에서는 `Object.assign()`을 사용해서 객체를 복사하는 방법을 제시하고 있지만
스프레드 연산자가 (1) 익숙하고 (2) 동작에 별 차이가 없기 때문에 스프레드 연산자를 사용했다.

#### 제품 가격 설정하기

```ts
const setPrice = (item, new_price) => {
  return { ...item_copy, price: new_price }
}
```

#### 객체에 값 설정하기

```ts
const objectSet = (object, key, value) => {
  return { ...object, key: value }
}
```

#### 객체의 키로 키/값 쌍 지우기

```ts
const objectDelete = (object, key) => {
  const object_copy = { ...object }
  delete object_copy[key]
  return object_copy
}
```

### 중첩된 쓰기 읽기로 바꾸기

```ts
const setPriceByName = (cart: Item[], name: string, price: number) => {
  // 배열의 카피-온-라이트
  const cartCopy = [...cart]
  // 중첩된 항목: 객체의 카피-온-라이트
  cartCopy.map(item => (item.name === name ? setPrice(item, price) : item))

  return cartCopy
}
```

### 어떤 복사본이 생겼을까요?

```ts
const setPrice = (item: Item, new_price: number) => {
  return { ...item, price: new_price } // 객체 복사
}

const setPriceByName = (cart: Item[], name: string, price: number) => {
  const cartCopy = [...cart] // 배열 복사
  cartCopy.map(item => (item.name === name ? setPrice(item, price) : item)) // 반복하다가 찾는 제품을 만나면 그때 setPrice()를 부름

  return cartCopy
}
```

- 중첩된 데이터에 얕은 복사를 하기 때문에 찾고자 하는 데이터만 복사하고 나머지는 구조적으로 공유한다.
  - 배열 하나 (장바구니)
  - 객체 하나 (티셔츠)
  - ...나머지 공유 (신발, 양말)
- 구조적 공유에서 공유된 복사본이 변경되지 않는 한 안전하다.
  - 값을 바꿀 때는 복사본을 만들고 있기 때문에 공유된 값은 변경되지 않는다고 확신할 수 있다!
