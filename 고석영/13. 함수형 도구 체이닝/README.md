## 체이닝

> [!NOTE]
>
> 여러 단계를 하나로 조합하는 것

### 체이닝 적용하기

> 각각의 우수 고객(3개 이상 구매)의 구매 중 가장 비싼 구매

1. 우수 고객(3개 이상 구매) 거르기(filter)
2. 우수 고객을 가장 비싼 구매로 바꾸기(map)

#### 함수 정의하기

```js
const biggestPurchasesBestCustomers = customers => {}
```

### 우수 고객 거르기

```js
const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = customers.filter(customer => customer.purchases.length >= 3)
}
```

#### 각 고객의 가장 비싼 구매 배열에 담기

```js
const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = customers.filter(customer => customer.purchases.length >= 3)

  const biggestPurchases = bestCustomers.map(customer => {})
}
```

#### 가장 비싼 구매 찾기

```js
const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = customers.filter(customer => customer.purchases.length >= 3)

  const biggestPurchases = bestCustomers.map(customer => {
    return customer.purchases.reduce(
      (biggestSoFar, purchase) => {
        return biggestSoFar.total > purchase.total ? biggestSoFar : purchase
      },
      { total: 0 },
    )
  })

  return biggestPurchases
}
```

> 잘 동작하지만 콜백이 여러 개 중첩되어 함수가 너무 커지고 복잡해졌다. 리팩토링 해보자.

#### total 값을 가져오는 부분을 콜백으로 분리하기

```js
const maxKey = (array, init, f) =>
  array.reduce((biggestSoFar, element) => {
    return f(biggestSoFar) > f(element) ? biggestSoFar : element
  }, init)

maxKey(customer.purchases, { total: 0 }, purchase => purchase.total)
```

#### `maxKey()` 적용하기

```js
const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = customers.filter(customer => customer.purchases.length >= 3)

  const biggestPurchases = bestCustomers.map(customer => {
    return maxKey(customer.purchases, { total: 0 }, purchase => purchase.total)
  })

  return biggestPurchases
}
```

- `maxKey()`로 코드가 의미하는 바가 명확해짐 → 배열에서 가장 큰 값을 선택한다는 의미

> 코드에 중첩된 리턴 구문이 있는 콜백이 있다. 이는 코드가 어떤 일을 하는지 알기 어렵게 만든다. 체인을 명확하게 만들면서 개선해보자.

## 체인을 명확하게 만들기

### 1. 단계에 이름 붙이기

- 😊 각 단계에 이름을 붙이면 훨씬 명확해지고 함수의 구현도 알아보기 쉬움
- 😞 인라인으로 정의된 콜백 함수를 재사용할 수 없음

```js
const selectBestCustomer = customers => customers.filter(customer => customer.purchases.length >= 3)

const getBiggestPurchase = customer =>
  maxKey(customer.purchases, { total: 0 }, purchase => purchase.total)
const getBiggestPurchases = customers => customers.map(customer => getBiggestPurchases)

const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = selectBestCustomer(customers)
  const biggestPurchases = getBiggestPurchases(bestCustomers)

  return biggestPurchases
}
```

### 2. 콜백에 이름 붙이기

- 콜백을 빼내면 고차 함수에 고객 배열에 넘길 때도 쓸 수 있고, 고객 하나에도 사용할 수 있는 재사용이 가능한 함수가 됨
  - 앞에서 만든 selectBestCutsomer 함수의 경우 고객 배열만 넘기는 경우에만 사용 가능
- 😊 재사용하기 좋고 직관적인 코드

```js
const isGoodCustomer = customer => customer.purchases.length >= 3

const getPurchaseTotal = purchase => purchase.total
const getBiggestPurchase = customer => maxKey(customer.purchases, { total: 0 }, getPurchaseTotal)

const biggestPurchasesBestCustomers = customers => {
  const bestCustomers = customers.filter(isGoodCustomer)
  const biggestPurchases = bestCustomers.map(getBiggestPurchase)

  return biggestPurchases
}
```

### 3. 단계에 이름 붙이기 vs 콜백에 이름 붙이기

> 일반적으로 두 번째 방법이 더 명확하다.
> 그리고 고차 함수를 그대로 쓰는 첫 번째 방법보다 이름을 붙이 두 번째 방법이 재사용하기도 더 좋다.
> 인라인 대신 이름을 붙여 콜백을 사용하면 단계가 중첩되는 것도 막을 수 있다.

## 예제: 한 번만 구매한 도구로 리팩터링하기

- 가진 것: 전체 고객 배열
- 필요한 것: 한 번만 구매한 고객들의 이메일 목록
- 계획
  1. 한 번만 구매한 고객 거르기(filter)
  2. 고객 목록을 이메일 목록으로 바꾸기(map)

```js
const firstTimers = customers.filter(customer => customer.purchases.length === 1)

const firstTimerEmails = firstTimers.map(customer => customer.email)
```

#### 콜백에 이름을 붙이기

```js
const isFirstTimer = customer => customer.purchases.length === 1
const firstTimers = customers.filter(isFirstTimer)

const getCustomerEmail = customer => customer.email
const firstTimerEmails = firstTimers.map(getCustomerEmail)
```

### 스트림 결합

- `map()`, `filter()`, `reduce()` 체인 최적화하기

```js
// 하나의 값에 map() 두번 사용
const names = map(customers, getFullName)
const nameLengths = map(names, StringLength)

// map() 한번 사용
const nameLengths = map(customers, customer => StringLength(getFullName(customer)))
```

---

## 반복문을 함수형 도구로 리팩토링하기

> 새로운 요구 사항이 아니라 기존에 있던 반복문을 함수형 도구로 리팩토링 해보자.
>
> → 반복문을 하나씩 선택한 다음 함수형 도구 체인으로 바꾸기!

```js
const answer = []
const window = 5

for (let i = 0; i < array.length; i++) {
  let sum = 0
  let count = 0
  for (let w = 0; w < window; w++) {
    const idx = i + w
    if (idx < array.length) {
      sum += array[idx]
      count += 1
    }
  }
  answer.push(sum / count)
}
```

- 원래 배열 크기만큼 answer 배열에 항목 추가 → `map()` 단서
- 배열을 돌면서 항목을 값 하나로 만들고 있음 → `reduce()` 단서

### 팁 1: 데이터 만들기

> [!NOTE]
>
> 데이터를 배열에 넣으면 함수형 도구를 쓸 수 있다.

#### 어떤 범위에 있는 값을 배열로 만들어 반복하기

```js
const answer = []
const window = 5

for (let i = 0; i < array.length; i++) {
  let sum = 0
  let count = 0
  const subarray = array.slice(i, i + window)
  for (let w = 0; w < subarray.length; w++) {
    sum += subarray[w]
    count += 1
  }
  answer.push(sum / count)
}
```

- `w`: 배열에 들어있는 값은 아님
- `idx`: 계속 바뀌지만 배열로 만들지는 않음
- `array[idx]`: 배열에 있는 작은 범위의 값이지만 배열로 따로 만들지는 않음

### 팁 2: 한 번에 전체 배열을 조작하기

- 하위 배열을 만들었기 때문에 일부 배열이 아닌 배열 전체를 반복할 수 있음
- 평균을 구하는 `average()` 함수를 재사용하여 반복문 제거

```js
const answer = []
const window = 5

for (let i = 0; i < array.length; i++) {
  const subarray = array.slice(i, i + window)
  answer.push(average(subarray)) // sum / count -> 평균
}
```

### 팁 3: 작은 단계로 나누기

#### 인덱스가 들어 있는 배열 만들기

- 반복문 안에서 배열 항목을 이용하고 있지 않아 필요한 인덱스가 들어 있는 배열 필요
- 만들어진 인덱스 배열 전체에 함수형 도구를 사용

```js
const indices = array.map((_, i) => i)
const window = 5

const answer = indices.map(i => {
  // 두 가지 일을 하는 함수
  const subarray = array.slice(i, i + window) // (1) 하위 배열을 만들기
  return average(subarray) // (2) 평균 계산하기
})
```

#### 두 가지 일을 하는 함수를 나누기

```js
const indices = array.map((_, i) => i)
const window = 5

const windows = indices.map(i => array.slice(i, i + window)) // (1) 하위 배열 만들기
const answer = windows.map(average) // (2) 평균 계산하기
```

#### 재사용이 가능한 `range()` 함수 만들기

```js
function range(start, end) {
  const ret = []
  for (let i = start; i < end; i++) ret.push(i)
  return ret
}

const indices = range(0, array.length)
```

## 절차적 코드와 함수형 코드 비교

#### 절차적 코드를 함수형 코드로 바꾸면서 얻은 이점

- 각 단계로 나누면서 코드를 글로도 쓸 수 있음 → 명확해짐
- 재사용이 가능한 함수형 도구도 생김 → 유연해짐

## 체이닝 디버깅을 위한 팁

- 구체적인 것을 유지하기
- 출력해보기
- 타입을 따라가보기

## 다양한 함수형 도구와 라이브러리

### 함수형 도구

#### `pluck()`

- 특정 필드값을 가져오기

```js
// 책
const prices = (products, 'price')

// FxTS
const iter = pluck('age', [{ age: 21 }, { age: 22 }, { age: 23 }])
```

#### `concat()`

- 배열 합치기

```js
// 책
const purchaseArrays = pluck(customers, 'purchases')
const allPurchases = concart(purchasesArrays)

// FxTS
const iter = concat([1, 2], [3, 4])
```

#### `pipe()`

- 데이터 흐름 처리를 제어할 수 있음
- 앞에서 나온 `map()`, `reduce()` 등도 체이닝해서 쓸 수 있지만 배열에만 쓸 수 있다면 `pipe()` 는 다양한 데이터 유형에도 사용 가능
- 왼쪽에서 오른쪽으로 읽어가면서 데이터 흐름을 볼 수 있음

```js
// FxTS
pipe(
  [1, 2, 3, 4, 5],
  map(a => a + 10),
  filter(a => a % 2 === 0),
  toArray,
) // [12, 14]
```

## 값을 만들기 위한 `reduce()`

> [!NOTE]
>
> `reduce()`는 값을 요약하기도 하지만 값을 만들기도 한다.

### 제품명으로 장바구니 살리기

#### `reduce()`로 장바구니 만들기

```js
const itemAdded = ['shirt', 'shoes', 'shirt', 'socks', 'hat']

const shoppingCart = itemAdded.reduce((cart, item) => {
  if (cart[item]) add_item(cart, { name: item, quantity: 1, price: priceLookup(item) })
  else {
    const quantity = cart[item].quantity
    return setFieldByName(cart, item, 'quantity', quantity + 1)
  }
}, {})
```

#### `addOne()` 함수로 빼내기

```tsx
const itemAdded = ['shirt', 'shoes', 'shirt', 'socks', 'hat']

const addOne = (cart, item) => {
  if (cart[item]) add_item(cart, { name: item, quantity: 1, price: priceLookup(item) })
  else {
    const quantity = cart[item].quantity
    return setFieldByName(cart, item, 'quantity', quantity + 1)
  }
}

const shoppingCart = itemAdded.reduce(addOne, {})
```

### 제품을 추가 및 삭제 처리하기

```tsx
const itemOps = [
  ['add', 'shirt'],
  ['add', 'shoes'],
  ['remove', 'shirt'],
  ['add', 'socks'],
  ['remove', 'hat'],
]

const addOne = (cart, item) => {
  if (!cart[item]) add_item(cart, { name: item, quantity: 1, price: priceLookup(item) })
  else {
    const quantity = cart[item].quantity
    return setFieldByName(cart, item, 'quantity', quantity + 1)
  }
}

const removeOne = (cart, item) => {
  if (!cart[item]) return cart
  else {
    const quantity = cart[item].quantity
    if (quantity === 1) remove_item_by_name(cart, item)
    else setFieldByName(cart, item, 'quantity', quantity - 1)
  }
}

const shoppingCart = itemOps.reduce((cart, itemOp) => {
  const [op, item] = itemOp
  if (op === 'add') addOne(cart, item)
  if (op === 'remove') removeOne(cart, item)
}, {})
```
