## 코드의 냄새: 함수 이름에 있는 암묵적 인자

- 코드의 냄새는 일급 값으로 바꾸면 표현력이 더 좋아짐
- 함수 이름에 있는 암묵적 인자는 코드의 냄새가 됨

```js
function setPriceByName(cart, name, price) {
  var item = cart[name]
  var newItem = objectSet(item, 'price', price)
  var newCart = objectSet(cart, name, newItem)
  return newCart
}

function setQuantityByName(cart, name, quant) {
  var item = cart[name]
  var newItem = objectSet(item, 'quantity', quant)
  var newCart = objectSet(cart, name, newItem)
  return newCart
}

function setShippingByName(cart, name, ship) {
  var item = cart[name]
  var newItem = objectSet(item, 'shipping', ship)
  var newCart = objectSet(cart, name, newItem)
  return newCart
}

function setTaxByName(cart, name, tax) {
  var item = cart[name]
  var newItem = objectSet(item, 'tax', tax)
  var newCart = objectSet(cart, name, newItem)
  return newCart
}

function objectSet(object, key, value) {
  var copy = Object.assign({}, object)
  copy[key] = value
  return copy
}
```

### 요구사항에 맞춘 함수들의 문제점

- 문자열 정도의 일부분만 다르고 거의 똑같이 생김
- 함수들의 차이점, 즉 필드를 결정하는 문자열이 함수 이름에 있음
- 함수 이름에 있는 일부가 인자처럼 동작하고 있음 **→ 함수 이름에 있는 암묵적 인자**

### 함수 이름에 있는 암묵적 인자 냄새가 가지는 특징

- 거의 똑같이 구현된 함수가 있다.
- 함수 이름이 구현에 있는 다른 부분을 가리킨다.

## 리팩토링: 암묵적 인자를 드러내기

- 기본적인 아이디어는 암묵적 인자를 명시적인 인자로 바꾸는 것
- 드러낸다

1. 함수 이름에 있는 암묵적 인자를 확인하기
2. 명시적인 인자를 추가하기
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾸기
4. 함수를 호출하는 곳을 고치기

### `setFieldByName()` 함수로 리팩토링

- 어떤 필드값이든 설정할 수 있는 함수로 리팩토링

#### 리팩토링 전

```tsx
// 사용
cart = setPriceByName(cart, 'shoe', 13)
cart = setQuantityByName(cart, 'shoe', 3)
cart = setShippingByName(cart, 'shoe', 0)
cart = setShippingByName(cart, 'shoe', 2.34)
```

#### 리팩토링 후

```tsx
const setFieldByName = (cart: Cart, name: string, field: string, value: number) => {
  const item = cart[name]
  const newItem = objectSet(item, field, value)
  const newCart = objectSet(cart, name, newItem)

  return newCart
}

// 사용
cart = setFieldByName(cart, 'shoe', 'price', 13)
cart = setFieldByName(cart, 'shoe', 'quantity', 3)
cart = setFieldByName(cart, 'shoe', 'shipping', 0)
cart = setFieldByName(cart, 'shoe', 'tax', 2.34)
```

- 이제 필드명은 암묵적인 이름에서 **인자로 넘길 수 있는 일급 값**이 되었음

## 일급인 것과 일급이 아닌 것을 구별하기

### 자바스크립트에서 일급이 아닌 것

- 수식 연산자
- 반복문
- 조건문
- try/catch 블록

### 일급으로 할 수 있는 것

- 변수에 할당
- 함수의 인자로 넘기기
- 함수의 리턴값으로 받기
- 배열이나 객체에 담기

### 필드명을 문자열로 사용하면 버그가 생기지 않을까요?

- 문제를 해결할 수 있는 방법
  - 컴파일 타임에 검사하기 → 타입스크립트 사용하기 🤗
  - 런타임에 검사하기 → 유효성 검사하기

```tsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'] as const
type ValidItemField = (typeof validItemFields)[number] // 컴파일 타임에 검사하기

// 런타임에 검사하기
const isValidItemField = (field: string) => validItemFields.includes(field as ValidItemField)

const setFieldByName = (cart: Cart, name: string, field: string, value: number) => {
  // 런타임에 검사하기
  if (!isValidItemField(field)) {
    throw new Error(`Not a valid item field: '${field}'.`)
  }

  const item = cart[name]
  const newItem = objectSet(item, field, value)
  const newCart = objectSet(cart, name, newItem)

  return newCart
}
```

### 일급 필드를 사용하면 API를 바꾸기 더 어렵나요?

어질…

```tsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'] as const
type ValidItemField = (typeof validItemFields)[number] // 컴파일 타임에 검사하기

const translations: { [key in ValidItemField]?: string } = { quantity: 'number' }

// 런타임에 검사하기
const isValidItemField = (field: string) => validItemFields.includes(field as ValidItemField)

const setFieldByName = (cart: Cart, name: string, field: string, value: number) => {
  // 런타임에 검사하기
  if (!isValidItemField(field)) {
    throw new Error(`Not a valid item field: '${field}'.`)
  }

  if (Object.hasOwn(translations, field)) {
    field = translations[field as ValidItemField]
  }

  const item = cart[name]
  const newItem = objectSet(item, field, value)
  const newCart = objectSet(cart, name, newItem)

  return newCart
}
```

### 객체와 배열을 너무 많이 쓰게 됩니다

> 장바구니와 제품처럼 일반적인 엔티티는 객체와 배열처럼 일반적인 데이터 구조를 사용해야 한다.

- 데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용
- 데이터를 데이터 그대로 사용하면 여러 가지 방법으로 해석 가능 **→ 데이터 지향**
- 데이터 지향: 이벤트와 엔티티에 대한 사실을 표현하기 위해 일반 데이터 구조를 사용하는 프로그래밍 형식

## 어떤 문법이든 일급 함수로 바꿀 수 있습니다

- 반복문을 말 그대로 일급 함수를 인자로 받는 함수인 고차함수로 바꾸기
- 고차함수: 인자로 함수를 받거나 리턴값으로 함수를 리턴할 수 있는 함수
- **고차함수를 이용하여 본문을 콜백으로 바꿀 수 있음**

```tsx
// 준비하고 먹기
for (var i = 0; i < foods.length; i++) {
  var food = foods[i]
  cook(food)
  eat(food)
}

// 설거지하기
for (var i = 0; i < dishes.length; i++) {
  var dish = dishes[i]
  wash(dish)
  dry(dish)
  putAway(dish)
}
```

### 1. 코드를 함수로 감싸기

- 반복문을 코드로 감싸 공통 로직을 제외한 나머지 부분을 넣을 수 있는 구멍 만들기

```tsx
function cookAndEatFoods() {
  for (var i = 0; i < foods.length; i++) {
    var food = foods[i]
    cook(food)
    eat(food)
  }
}

function cleanDishes() {
  for (var i = 0; i < dishes.length; i++) {
    var dish = dishes[i]
    wash(dish)
    dry(dish)
    putAway(dish)
  }
}

cookAndEatFoods()
cleanDishes()
```

### 2. 더 일반적인 이름으로 바꾸기

- 지역변수의 이름이 매우 구체적이므로 더 일반적인 이름으로 바꾸기

```tsx
function cookAndEatArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    cook(item)
    eat(item)
  }
}

function cleanArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    wash(item)
    dry(item)
    putAway(item)
  }
}

cookAndEatArray(foods)
cleanArray(dishes)
```

### 3. 암묵적 인자를 드러내기

- 암묵적 인자를 명시적 인자로 변경하기

```tsx
function cookAndEatArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    cookAndEat(item)
  }
}

function cookAndEat(food) {
  cook(food)
  eat(food)
}

function cleanArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    clean(item)
  }
}

function clean(dish) {
  wash(dish)
  dry(dish)
  putAway(dish)
}

cookAndEatArray(foods)
cleanArray(dishes)
```

### 4. 함수 추출하기

- 반복문 안에 있는 본문을 함수로 빼내 한 줄로 만들기

```tsx
function cookAndEatArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    cookAndEat(item)
  }
}

function cookAndEat(food) {
  cook(food)
  eat(food)
}

function cleanArray(array) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    clean(item)
  }
}

function clean(dish) {
  wash(dish)
  dry(dish)
  putAway(dish)
}

cookAndEatArray(foods)
cleanArray(dishes)
```

### 5. 암묵적 인자를 드러내기

```tsx
function operateOnArray(array, f) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    f(item)
  }
}

function cookAndEat(food) {
  cook(food)
  eat(food)
}

function clean(dish) {
  wash(dish)
  dry(dish)
  putAway(dish)
}

operateOnArray(foods, cookAndEat)
operateOnArray(dishes, clean)
```

### (+) `forEach()` 로 변경하고 익명 함수를 인자로 받기

- 앞에서 만든 `operateOnArray()` 함수를 자바스크립트처럼 `forEach()`로 함수 이름 변경
- `forEach()` 함수는 함수를 인자로 받으므로 고차 함수

```tsx
function forEach(array, f) {
  for (var i = 0; i < array.length; i++) {
    var item = array[i]
    f(item)
  }
}
```

#### 익명 함수를 인자로 받기

```tsx
forEach(foods, function (food) {
  cook(food)
  eat(food)
})

forEach(dishes, function (dish) {
  wash(dish)
  dry(dish)
  putAway(dish)
})
```

> 너무 많은 단계가 있어서 단계를 하나로 합친 **함수 본문을 콜백으로 바꾸기**로 리팩토링해 보자.
> 여러 단계를 줄여서 할 수 있는 리팩토링이고 이를 새로운 로깅 시스템에 적용해 보자.

## 리팩토링: 함수 본문을 콜백으로 바꾸기

```tsx
try {
  saveUserData(user) // 여기만 계속 변경됨
} catch (error) {
  logToSnapErrors(error)
}
```

### 1. 본문과 본문의 앞부분과 뒷부분을 구분하기

```tsx
try {
  // 본문 앞부분
  saveUserData(user) // 본문 (계속 변경되는 부분)
} catch (error) {
  // 본문 뒷부분
  logToSnapErrors(error)
}
```

### 2. 전체를 함수로 빼내기

- `withLogging()` 함수로 만들고 부르기

```tsx
function withLogging() {
  try {
    saveUserData(user) // 본문
  } catch (error) {
    logToSnapErrors(error)
  }
}

withLogging()
```

### 3. 본문 부분을 빼낸 함수의 인자로 전달한 함수(콜백 함수)로 바꾸기

- 원래 본문이 있던 곳에서는 인자로 받은 함수를 호출
- 본문에 전달할 함수는 익명 함수로 전달

```tsx
function withLogging(f) {
  try {
    f() // 본문이 있던 곳에서 인자로 받은 함수를 호출
  } catch (error) {
    logToSnapErrors(error)
  }
}

withLogging(function () {
  // 익명 함수
  saveUserData(user)
})
```

### 콜백으로 익명 함수 전달하기

#### 전역으로 정의하기

- 함수를 전역적으로 정의하고 이름을 붙이면 나중에도 사용 가능

```tsx
function saveCurrentUserData() {
  saveUserData(user)
}
```

#### 지역적으로 정의하기

- 함수를 지역 범위 안에서 정의하고 이름을 붙이면 이름은 가지되, 범위 밖에서는 사용할 수 없음

```tsx
function someFunction() {
  const saveCurrentUserDat = function saveCurrentUserData() {
    saveUserData(user)
  }
}
```

#### 인라인으로 정의하기

- 함수 사용하는 곳에서 바로 정의하며 주로 익명 함수를 사용

```tsx
withLogging(function () {
  saveUserData(user)
})
```

### 본문을 함수로 감싸서 넘기는 이유

- 코드가 바로 실행되면 안 되기 때문에 함수의 실행을 미루기 위해 그냥 넘겼음
- 함수 안에 담아 실행을 미뤄 전달한 함수는 필요한 상황에 맞게 호출될 수 있게 조정할 수 있음
