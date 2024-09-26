## 쿠폰 이메일 처리 로직 리팩토링

```js
function emailsForCustomers(customers, goods, bests) {
  var emails = []
  for (var i = 0; i < customers.length; i++) {
    var customer = customers[i]
    var email = emailForCustomer(customer, goods, bests)
    emails.push(email)
  }
  return emails
}
```

### `forEach()`를 사용하여 리팩토링

```js
function emailsForCustomers(customers, goods, bests) {
  var emails = []
  forEach(customers, function (customer) {
    var email = emailForCustomer(customer, goods, bests)
    emails.push(email)
  })
  return emails
}
```

### `map()`을 사용하여 리팩토링

#### `forEach()`를 `map()`으로 빼내기

```js
function map(array, f) {
  var newArray = []
  forEach(array, function (element) {
    newArray.push(f(element))
  })
  return newArray
}
```

#### `map()`에 콜백으로 본문 전달하기

```js
function emailsForCustomers(customers, function(customer) {
  return emailForCustomer(customer, goods, bests)
})
```

## 함수형 도구 1: `map()`

> [!NOTE]
>
> `map()`은 x 값이 있는 배열을 y 값이 있는 배열로 변환한다.
>
> 이때 x를 인자로 받아 y를 리턴하는 즉, x를 y로 바꾸는 `xToY()`함수를 필요로 한다.

### 함수를 전달하는 세 가지 방법

#### 전역으로 정의하기

- 함수를 전역으로 정의하고 이름을 붙이면 나중에도 사용 가능

```js
function greet(name) {
  return `Hello, ${name}`
}
```

#### 지역적으로 정의하기

- 함수를 지역 범위 안에서 정의하고 이름을 붙이면 이름은 가지되, 범위 밖에서는 사용할 수 없음
- 지역적으로 쓰고 싶지만 이름이 필요한 경우 유용

```js
function greetEveryBody(friends) {
  var greeting
  if (language === 'English') greeting = 'Hello'
  else greeting = 'Salut, '

  var greet = function (name) {
    return greeting + name
  }

  return map(friends, greet)
}
```

#### 인라인으로 정의하기

- 함수 사용하는 곳에서 바로 정의하며 주로 익명 함수를 사용
- 문맥에서 한 번만 쓰는 짧은 함수에 유용

```js
var friendGreeting = map(friendsNames, function (name) {
  return `Hello, ${name}`
})
```

### 예제: 모든 고객의 이메일 주소

- 가진 것: 고객 배열
- 필요한 것: 고객 이메일 주소 배열
- 함수: 고객 하나를 받아 고객 이메일 주소를 리턴하는 함수

```js
map(customers, function (customer) {
  return customer.email
})
```

### 예제: 모든 고객의 성, 이름, 주소가 담긴 객체

- 가진 것: 고객 배열
- 필요한 것: 고객의 성, 이름, 주소가 담긴 객체 배열
- 함수: 고객 하나를 받아 고객의 성, 이름, 주소가 담긴 객체를 리턴하는 함수

```js
map(customers, customer => ({
  firstName: customer.firstName,
  lastName: customer.lastName,
  address: customer.address,
}))
```

> [!WARNING]
>
> `map()`은 리턴값인 배열에 들어 있는 항목을 확인하지 않기 때문에 주의해야 함

## 함수형 도구 2: `filter()`

- filter()는 X 값이 있는 배열을 받아서 순서를 유지하면서 통과하는 Y 값이 있는 배열 반환
- 배열에서 일부 항목을 선택하는 함수
- 항목이 x인 배열에서 filter()를 사용해도 결과는 여전히 항목이 x인 배열
- 항목을 선택하기 위해 x를 받아 불리언 타입을 리턴하는 함수(술어)를 전달
  - true인 경우 항목을 유지하고 false인 경우에는 항목을 없애기

```js
function selectBestCustomers(customers) {
  var newArray = []
  forEach(customers, function (customer) {
    if (customer.purchases.length >= 3) newArray.push(customer)
  })
  return newArray
}
```

### `filter()` 함수 도출하기: 우수 고객 목록

#### `forEach()`를 `filter()`로 빼내기

```jsx
function filter(array, f) {
	var newArray = []
	forEach(array, funtion(element) {
		if(f(element))
			newArray.push(element)
	})
	return newArray
}
```

#### `filter()`에 콜백으로 본문 전달하기

```js
function selectBestCustomer(customers) {
  return filter(customers, function (customer) {
    return customer.purchases.length >= 3
  })
}
```

### 예제: 아무것도 구입하지 않은 고객

- 가진 것: 고객 배열
- 필요한 것: 아무것도 구입하지 않은 고객 배열
- 함수: 고객 하나를 받아 아무것도 구입하지 않았다면 true를 리턴

```js
filter(customers, function (customer) {
  return customer.purchases.length === 0
})
```

### 예제: 테스트 그룹과 아닌 그룹을 나누기

- 가진 것: 고객 배열
- 필요한 것: 테스트 그룹 고객 배열과 테스트 그룹이 아닌 고객 배열
- 함수: 고객 하나를 받아 고객의 아이디가 3으로 나누어 떨어지는지의 여부에 따라 고객을 구분

```js
const testGroup = filter(customers, customer => customer.id % 3 === 0)
const nonTestGroup = filter(customers, customer => customer.id % 3 !== 0)
```

## 함수형 도구3: `reduce()`

- 배열을 순회하면서 값을 누적
- 전달하는 함수를 통해 누적하는 방법을 결정
- 콜백은 누적하고 있는 현재 값과 반복하고 있는 현재 배열의 항목을 인자로 받은 뒤, 새로운 누적값을 리턴
- reduce()에 전달하는 함수의 인자: 현재 누적된 값, 배열의 현재 항목
  - 첫 번째 인자와 같은 타입의 값을 리턴해야 함

### 예제 1: 모든 고객의 전체 구매 수

```js
function countAllPurchases(customers) {
	val total = 0;
	forEach(customers, function(customer) {
		total = total + customer.purchases.length
	}
	return total
}
```

#### `forEach()`를 `reduce()`로 빼내기

```jsx
function reduce(array, init, f) {
  var accum = init
  forEach(array, function (element) {
    accum = f(init, element)
  })
  return accum
}
```

#### `reduce()`에 콜백으로 본문 전달하기

```js
function countAllPurchases(customers) {
  return reduce(customers, 0, function (total, customer) {
    return total + customer.purchases.length
  })
}
```

### 예제 2: 문자열 합치기

- 가진 것: 문자열 배열
- 필요한 것: 배열에 있는 모든 문자열을 하나로 합친 문자열
- 함수: 누적된 문자열과 배열에 있는 현재 문자열을 받아서 합치는 함수

```js
reduce(strings, '', function (accum, string) {
  return accum + string
})
```

> `reduce()`는 주어진 초기 값을 가지고 배열에 있는 모든 항목을 하나의 값으로!

> [!WARNING]
>
> #### `reduce()` 함수를 사용할 때 두 가지 주의 사항
>
> - 인자의 순서
> - 초기 값을 결정하는 방법
>   - 계산이 어떤 값에서 시작되는가?
>   - 빈 배열을 사용하면 어떤 값을 리턴할 것인가?
>   - 비즈니스 규칙이 있는가?

### 예제 3: 더하기와 곱하기

#### 더하기

```js
function sum(numbers) {
  return reduce(numbers, 0, (total, num) => total + num)
}
```

#### 곱하기

```js
function produce(numbers) {
  return reduce(numbers, 0, (total, num) => total * num)
}
```

### 예제 4: `min()`, `max()`

#### `min()`

```js
function min(numbers) {
  return reduce(numbers, Number.MAX_VALUE, (min, num) => (min < num ? min : num))
}
```

#### `max()`

```js
function max(numbers) {
  return reduce(numbers, Number.MIN_VALUE, (max, num) => (max > num ? max : num))
}
```

### `reduce()`로 할 수 있는 것들

- 실행 취소/복귀
  - 리스트 형태의 사용자 입력에 reduce()를 적용한 것이 현재 상태
  - 실행 취소는 리스트의 마지막 사용자 입력을 없애는 것
- 테스트할 때 사용자 입력을 다시 실행하기
  - 시스템의 처음 상태가 초기 값이고 사용자 입력이 순서대로 리스트에 있을 때 reduce()로 모든 값을 합쳐 현재 상태를 만들 수 있음
- 시간 여행 디버깅
  - 뭔가 잘못 동작하는 경우 특정 시점 상태의 값을 보관할 수 있음
- 회계 감사 추적
  - 특정 시점에 시스템 상태를 알고 싶은 경우

## 세 가지 함수형 도구 비교하기

- `map()` : 어떤 **배열**의 모든 항목에 함수를 적용해 새로운 **배열**로 반환
- `filter()` : 어떤 **배열**의 하위 집합을 선택해 새로운 **배열**로 반환
- `reduce()` : 어떤 **배열**의 항목을 조합해 **최종값**을 반환
