# 함수형 반복

## 이번 장에서 살펴볼 내용

- 함수형 도구 map(), filter(), reduce() 에 대해 알아보기
- 배열에 대한 반복문을 함수형 도구로 바꾸는 방법에 대해 알아보기
- 함수형 도구를 어떻게 구현하는지 알아보기

## 예제를 통해 map 함수 도출하기

고객 커뮤니케이션팀에 할당된 코드들
```ts
type Customers = Customer[];
type Goods = Coupon[];
type Bests = Coupon[];

const emailsForCustomers = (customers: Customers, goods: Goods, bests: Bests) => {
  const emails = [];
  for(let i = 0; i < customers.length; i++) {
    const customer = customers[i];
    const email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  }
  return emails;
}

const biggestPurchasePerCustomer = (customers: Customers) => {
  const purchases = [];
  for(let i = 0; i < customers.length; i++) {
    const customer = customers[i];
    const purchase = biggestPurchase(customer);
    purchases.push(purchase);
  }
  return purchases;
}

const customerFullNames = (customers: Customers) => {
  const fullNames = [];
  for(let i = 0; i < customers.length; i++) {
    const cust = customers[i];
    const name = cust.firstName + " " + cust.lastName;
    fullNames.push(name);
  }
  return fullNames;
}

const customerCities = (customers: Customers) => {
  const cities = [];
  for(let i = 0; i < customers.length; i++) {
    const customer = customers[i];
    const city = customer.address.city;
    cities.push(city);
  }
  return cities;
}
```

코드의 형태는 반환되는 배열에 넣을 값을 만드는 부분만 다름
  - `emailForCustomer(customer, goods, bests);`
  - `biggestPurchase(customer);`
  - `cust.firstName + " " + cust.lastName;`
  - `customer.address.city;`   

전장에서 만든 `forEach` 함수로 개선한다면 아래와 같이 개선 가능
```ts
const emailsForCustomers = (customers: Customers, goods: Goods, bests: Bests) => {
  const emails = [];
  forEach(customers, (customer) => {
    const email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  })
  return emails;
}
```

개선한 버전도 반환되는 배열에 넣을 값을 만드는 부분만 다른건 동일하기 때문에 `map`함수를 만들어서 개선
```ts
const emailsForCustomers = (customers: Customers, goods: Goods, bests: Bests) => {
  return <Customers, (customer: Customer) => Email[]>map(customers, (customer) => {
    return emailForCustomer(customer, goods, bests);
  });
}

const map = <T, U>(array:T, f:U) => {
  const newArray = [];
  forEach(array, (element) => {
    newArray.push(f(element));
  });
  return newArray;
}
```
> ![TIP]
> 실제 map의 타입은
> ```ts
> interface Array<T>{
>   map<U>(callbackfn: (value: T, index: number, array: T[]) => U, thisArg?: any): U[];
> }
> ```
> 으로 따로 안적어도 추론이 됨

`map`함수의 결과를 반환해서 중복을 최소화 했음

```js
// 개선된 함수 보기(중복 최소화를 강조하기 위해 javascript로 ㅎㅎ)
const emailsForCustomers = (customers, goods, bests) => {
  return map(customers, (customer) => emailForCustomer(customer, goods, bests));  
}

const biggestPurchasePerCustomer = (customers) => {
  return map(customers, (customer) => biggestPurchase(customer));
}

const customerFullNames = (customers) => {
  return map(customers, (customer) => customer.firstName + " " + customer.lastName);
}

const customerCities = (customers) => {
  return map(customers, (customer) => customer.address.city);
}
```

### 함수형 도구 map()
- map() 에 넘기는 함수가 계산일 때 가장 사용하기 쉬움
- map() 에 계산을 넘기면 map도 계산 이지만, 액션을 넘기면 map도 액션이 됨
- 전달하는 함수의 인자 이름은 어떤 이름을 붙여도 되지만, 명확하기 하기 위해 인자가 무엇인지 알려주는 변수명을 사용

## 함수를 전달하는 세가지 방법

1. 전역으로 정의하기
  함수의 이름으로 프로그램 어디서나 사용가능
2. 지역적으로 정의하기
  지역 범위 안에서 정의하고 이름을 붙이며, 지역 안에서만 사용가능
3. 인라인으로 정의하기
  함수를 사용하는 곳에서 바로 정의하며, 문맥에서 한 번만 쓰는 짧은 함수에 사용



## 예제를 통해 filter 함수 도출하기

고객 커뮤니케이션팀에 할당된 코드들
```ts
const selectBestCustomers = (customers: Customers) => {
  const newArray = [];
  forEach(customers, (customer) => {
    if(customer.purchases.length >= 3)
      newArray.push(customer);
  });
  return newArray;
}

const selectCustomersAfter = (customers: Customers, date: Date) => {
  const newArray = [];
  forEach(customers, (customer) => {
    if(customer.signupDate > date)
      newArray.push(customer);
  });
  return newArray;
}

const selectCustomersBefore = (customers: Customers, date: Date) => {
  const newArray = [];
  forEach(customers, (customer) => {
    if(customer.signupDate < date)
      newArray.push(customer);
  });
  return newArray;
}

const singlePurchaseCustomers = (customers: Customers) => {
  const newArray = [];
  forEach(customers, (customer) => {
    if(customer.purchases.length === 1)
      newArray.push(customer);
  });
  return newArray;
}
```

코드의 형태는 if문으로 검사하는 부분만 다름
  - `customer.purchases.length >= 3`
  - `customer.signupDate > date`
  - `customer.signupDate < date`
  - `customer.purchases.length === 1`   

filter함수를 만들어서 개선
```js
const selectBestCustomers = (customers) => {
  return filter(customers, (customer) => customer.purchases.length >= 3)
}

const filter = (array, f) => {
  const newArray = [];
  forEach(array, (element) => {
    if(f(element)){
      newArray.push(element);
    }
  })
  return newArray;
}
```

filter도 map과 같이 계산일 때 가장 사용하기 쉬움

## 예제를 통해 reduce 함수 도출하기

고객 커뮤니케이션팀에 할당된 코드들
```ts
const countAllPurchases = (customers: Customers) => {
  const total = 0;
  forEach(customers, (customer) => {
    total = total + customer.purchases.length;
  });
  return total;
}

const concatenateArrays = (arrays: Array[]) => {
  const result = [];
  forEach(arrays, (array) => {
    result = result.concat(array);
  });
  return result;
}

const customersPerCity = (customers: Customers) => {
  const cities = {};
  forEach(customers, (customer) => {
    cities[customer.address.city] += 1;
  });
  return cities;
}

const biggestPurchase = (purchases: Purchases) => {
  const biggest = {total:0};
  forEach(purchases, (purchase) => {
    biggest = biggest.total > purchase.total ? biggest : purchase;
  });
  return total;
}
```

코드의 형태는 초기값과 합치는 부분만 다름
  - `0`, `+ customer.purchases.length`
  - `[]`, `.concat(array)`
  - `{}`, `[customer.address.city] += 1`
  - `{total:0}`, `biggest.total > purchase.total ? biggest : purchase`

reduce함수를 만들어서 개선
```js
const countAllPurchases = (customers) => {
  return reduce(customers, 0, (total, customer) => total + customer.purchase.length);
}

const reduce = (array, init, f) => {
  const accum = init;
  forEach(array, (element) => {
    accum = f(accum, element);
  });
  return accum;
}
```

reduce는 배열을 순회하면서 값을 누적함

## reduce()로 할 수 있는 것들

- reduce로 map과 filter를 만들 수 있음
- 실행 취소/실행 복귀
- 테스트할 때 사용자 입력을 다시 실행하기
- 시간 여행 디버깅
- 회계 감사 추적

### reduce로 map과 filter 만들어보기

```js
const map = (array, f) => {
  return reduce(array, [], function(ret, item) {
    ret.push(item);
    return ret;
  });
}

const filter = (array, f) => {
  return reduce(array, [], function(ret, item) {
    if(f(item))
      ret.push(item);
    return ret;
  });
}
```

## 세 가지 함수형 도구를 비교하기

1. map()
    어떤 배열의 모든 항목에 함수를 적용해 새로운 배열로 바꿈
2. filter()
    어떤 배열의 하위 집합을 선택해 새로운 배열로 만듬
3. reduce()
    어떤 배열의 항목을 조합해 최종값을 만듬