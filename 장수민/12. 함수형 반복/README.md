## 함수형 반복

### 예제를 통해 map() 함수를 도출하기

고객 커뮤니케이션팀에 할당된 비슷한 코드를 살펴보자.

```tsx
function emailsForCustomers(customers, good, bests) {
  const emails = [];
  forEach(customers, function (customer) {
    const email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  });
  return emails;
}

function biggestPurchasePerCustomer(customers) {
  const purchases = [];
  forEach(customers, function (customer) {
    const purchase = biggestPurchase(customer);
    purchases.push(purchase);
  });
  return purchases;
}

// customerFullNames, customerCities 함수 생략
```

비슷한 코드에 **함수 본문을 콜백으로 바꾸기 리팩터링**을 진행해 볼 수 있다!

> [!NOTE]
> 본문이란 꽤나 비슷한 함수 내에 핵심이면서, 다른 부분을 가리키는 것 같다...!

```tsx
// 콜백으로 바꾼 버전
function emailsForCustomers(customers, good, bests) {
  return map(customers, function (customer) {
    return emailForCustomer(customer, goods, bests);
  });
}

function map(array, f) {
  const newArray = [];
  forEach(array, function (element) {
    newArray.push(f(element)); // 콜백 함수 호출
  });
  return newArray;
}
```

### 함수형 도구: map()

🚨`map()`🚨은 이 장에서 소개하는 세 가지 함수형 도구 중 하나이다.

```tsx
function map(array, f) {
  // 배열과 함수를 인자로 받는다.
  const newArray = []; // 빈 배열을 만든다.
  forEach(array, function (element) {
    newArray.push(f(element)); // 원래 배열의 항목으로 새로운 항목을 만들기 위해서 f() 함수를 호출하고, 새로운 배열에 추가한다.
  });
  return newArray; // 새로운 배열을 리턴한다.
}
```

고로, `map()`함수를 사용하면 값 하나를 바꾸는 함수를 배열 전체를 바꾸는 데 사용할 수 있다.

> [!WARNING]
> map()에 넘기는 함수가 액션이면 map()은 항목의 개수만큼 액션을 호출하게 되므로, map()을 사용하는 코드도 액션이 된다...!

고차 함수에 익숙하지 않다면 어려울 수도 있는 내용을 다뤄본다.

```tsx
function emailsForCustomers(customers, goods, bests) {
  return map(customers, function (customer) {
    // customer는 위에서 전달된 인자도 아닌데, 어디서 나왔죠?
    return emailForCustomer(customer, goods, bests);
  });
}

// 그에 대한 답변으로는,
// map()에 넘기는 customers 배열에는 고객 정보가 있을 것으로 기대한다.
// customer가 아닌 다른 이름의 인자더라도 동작하지만, 명확하게 하기 위해 customer라는 인자명을 사용한다.
```

> [!WARNING]
> map() 함수를 사용할 때 주의할 점은, 리턴값인 배열에 들어 있는 항목을 확인하지 않기 때문에 null이나 undefined 값이 들어갈 수도 있다. 조심해서 사용하거나 null을 허용하지 않는 언어를 사용하자.

---

### 예제를 통해 filter() 함수 도출하기

고객 커뮤니케이션팀에 할당된 비슷한 코드를 살펴보자.

```tsx
function selectBestCustomers(customers) {
  const newArray = [];
  forEach(customers, function (customer) {
    if (customer.purchases.length >= 3) newArray.push(customer);
  });
  return newArray;
}

function selectCustomersAfter(customers, date) {
  const newArray = [];
  forEach(customers, function (customer) {
    if (customer.signupDate > date) newArray.push(customer);
  });
  return newArray;
}

// selectCustomersBefore, singlePurchaseCustomers 함수 생략
```

이번에도 '함수 본문을 콜백으로 바꾸기' 리팩토링을 진행해보자.

```tsx
function selectBestCustomers(customers) {
  return filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });
}

function filter(array, f) {
  // 배열과 콜백 함수를 받는다.
  const newArray = []; // 빈 배열을 만든다.
  forEach(array, function (element) {
    if (f(element)) newArray.push(element); // f()를 호출해 항목을 결과 배열에 넣을지 확인한다. 맞으면 결과 배열에 넣는다.
  });
  return newArray; // 결과 배열을 리턴한다.
}
```

고로, `filter()`는 배열에서 일부 항목을 선택하는 함수로 볼 수 있다.
그렇기 때문에,

- 원래 가진 항목보다 더 적을 수도 있다.
- 항목을 선택하기 위해서 불리언 타입을 리턴하는 함수(**술어**)를 전달해야 한다.

> [!NOTE]
> 앞에서 map()을 사용할 때 결과 배열에 null이 있을 수도 있었다. 만약 null을 없애고자 한다면 filter() 함수를 사용하면 된다!

---

### reduce()가 필요한 상황(책에는 없는 목차)

모든 고객이 지금까지 얼마나 구매했는지 알고 싶다...! 각 고객의 구매 목록은 고객 레코드에 있으므로, 전부 더하면 전체 구매 수가 나온다.

### 예제를 통해 reduce() 도출하기

```tsx
function countAllPurchases(customers) {
  let total = 0;
  forEach(customers, function (customer) {
    total = total + customer.purchases.length;
  });
  return total;
}

function concatenateArrays(arrays) {
  const result = [];
  forEach(arrays, function (array) {
    result = result.concat(array);
  });
  return result;
}

// customersPerCity, biggestPurchase 함수 생략
```

이번에도 '함수 본문을 콜백으로 바꾸기' 리팩토링을 진행해보자.

```tsx
function countAllPurchases(customers) {
  return reduce(customers, 0, function (total, customer) {
    // 자바스크립트 내장 고차함수는 init 값이 뒤에 있다.
    return total + customer.purchases.length;
  });
}

function reduce(array, init, f) {
  // 배열과 초깃값, 누적 함수를 받는다.
  let accum = init; // 누적된 값을 초기화 한다.
  forEach(array, function (element) {
    accum = f(accum, element); // 누적 값을 계산하기 위해 현재 값과 배열 항목으로 f() 함수를 호출한다.
  });
  return accum; // 누적된 값을 리턴한다.
}
```

🚨`reduce()`🚨 함수에서 누적한다는 것은 추상적인 개념이다. 연산이 계속해서 누적될 수도 있고, 해시 맵이나 문자열을 합치고, 배열을 평탄화할 수도 있다.

> [!WARNING]
> reduce()를 사용할 때 주의할 점으로는, 1. 인자의 순서, 2. 초깃값이다. reduce()에 전달하는 인자의 갯수가 2개이기 때문에 혼동하지 않도록 해야 하고, 곱하기를 제대로 수행하려면 초깃값이 1이 되어야 하는 것처럼 초깃값을 잘 설정해야 한다.

reduce()가 강력한 이유는 해당 함수를 통해 map()이나 filter()를 구현할 수 있지만, 역으로는 불가능하다.

바꿔보자면,

```tsx
function map(array, f) {
  return reduce(array, [], function (ret, item) {
    return ret.concat(f([item])); // 불변 함수를 사용
  });
}

function map1(array, f) {
  return reduce(array, [], function (ret, item) {
    ret.push(f(item)); // 조금 더 효율적인 변이 함수를 사용
    return ret;
  });
}

function filter(array, f) {
  return reduce(array, [], function (ret, item) {
    if (f(item)) return ret.concat([item]); // 불변 함수를 사용
    return ret;
  });
}

function filter1(array, f) {
  return reduce(array, [], function (ret, item) {
    if (f(item)) ret.push(item); // 조금 더 효율적인 변이 함수를 사용
    return ret;
  });
}

// 변이 함수를 넘기는 경우 계산이라는 규칙을 어길 수도 있지만, 변이가 지역적으로 일어나기 때문에 여전히 계산이라는 규칙을 유지한다!
```
