# 함수형 도구 체이닝

## 이번 장에서 살펴볼 내용
- 복합적인 쿼리로 데이터를 조회하기 위해 함수형 도구를 조합하는 방법을 배운다
- 복잡한 반복문을 함수형 도구 체인으로 바꾸는 방법을 이해한다
- 데이터 변환 파이프라인을 만들아 작업을 수행하는 방법을 배운다.

## 우수고객(3개 이상 구매)의 구매 중 가장 비싼 구매를 알려주세요

1. 3개 이상 구매한 우수 고객을 찾아낸다. (filter)
2. 우수 고객의 가장 비싼 구매로 바꾼다. (map)

이렇게 여러 단계를 하나로 조합하는 것을 `체이닝` 이라고 한다.

### 코드로 구현하면

아래와 같이 각 단계를 코드로 구현
```ts
const biggestPurchasesBestCustomers = (customers: Customers) => {
  // 우수고객 찾기
  const bestCustomers = filter(customers, (customer) => {
    return customer.purchases.length >= 3;
  })

  // 비싼 구매로 바꾸기
  const biggestPurchases = map(bestCustomers, (customer) => {
    return maxKey(customer.purchases, {total: 0}, (purchase) => {
      return purchase.total
    })
  })

  return biggestPurchases;
}
```

### 체인을 명확하게 만들기 첫번째 방법(단계에 이름 붙이기)
```ts
const biggestPurchasesBestCustomers = (customers: Customers) => {  
  const bestCustomers = selectBestCustomers(customers);
  const biggestPurchases = getBiggestPurchases(customers);
  return biggestPurchases;
}

const selectBestCustomers = (customers: Customers) => {
  return filter(customers, () => {
    return customer.purchases.length >= 3;
  })
}

const getBiggestPurchases = (customers: Customers) => {
  return map(customers, getBiggestPurchase);
}

const getBiggestPurchase = (customer: Customer) => {
  return maxKey(customer.purchases, {total: 0}, (purchase) => {
      return purchase.total;
    })
}
```
각 단계를 함수로 추출해서 그 함수를 호출해주는 방식

### 체인을 명확하게 만들기 두번째 방법(콜백에 이름 붙이기)
```ts
const biggestPurchasesBestCustomers = (customers: Customers) => {
  const bestCustomers = filter(customers, isGoodCustomer);
  const biggestPurchases = map(bestCustomers, getBiggestPurchase);
  return biggestPurchases;
}

const isGoodCustomer = (customer: Customer) => {
  return customer.purchases.length >= 3;
}

const getBiggestPurchase = (customer: Customer) => {
  return maxKey(customer.purchases, {total: 0}, getPurchaseTotal)
}

const getPurchaseTotal = (purchase: Purchase) => {
  return purchase.total;
}
```

### 체인을 명확하게 만들기 방법 비교
일반적으로 두번째 방법이 더 명확하며,   
고차 함수를 그대로 쓰는 첫번째 방법보다 이름을 붙인 두번째 방법이 재사용하기도 좋음

> [!NOTE]
> 저도 가독성을 개선하기 위한 리팩토링으로 위 두가지 방법을 사용했었던거 같은데,
> 명확한 기준이 없어서 첫번째 방법을 쓴곳도 있고 두번째 방법을 쓴곳도 있었던 것 같습니다.
> 예시 코드를 보니 두번째 방법이 조건이 바뀌었을때 더 집중하기도 좋고, 책에서 말한대로 재사용하기도 좋을것 같네요
> 앞으로는 기준을 가지고 조금 더 생각해보고 추출해야 겠습니다!

### 한 번만 구매한 고객의 이메일 목록
```ts
const firstTimers = filter(customers, isFirstTimer);
const firstTimerEmails = map(firstTimers, getCustomerEmail);

const isFirstTimer = (customer: Customer) => {
  return customer.purchases.length === 1;
}

const getCustomerEmail = (customer: Customer) => {
  return customer.email
}
```
더 작은 단위의 함수로 추출

## 함수형 도구가 비효울적인 경우에

체인을 최적화 하는 것을 `스트림 결합`이라고 합니다.

```ts
// map 두번 사용
const names = map(customers, getFullName);
const nameLengths = map(names, stringLength);

// map 한번 사용
const nameLengths = map(customers, (customer) => {
  return stringLength(getFullName(customer));
})
```
map을 한번만 사용하도록 최적화를 진행한 내용이다.   
현대의 가비지 컬렉터는 매우 빠르기 때문에, 꼭 최적화가 필요한 경우에만 중첩으로 사용하고 그렇지 않을 때는 **나누는 것이 더 명확하고 읽기 좋다**   
(filter, reduce도 동일)

## 반복문을 함수형 도구로 리팩터링

### 단서를 찾아 리팩터링

**데이터 만들기**
```ts
const answer = [];
const window = 5;
for(let i = 0; i < array.length; i++) {
  let sum = 0;
  let count = 0;
  for(let w = 0; w < window; w++) { 
    const idx = i + w; // 인덱스 부터 window 만큼의 갯수를 반복
    if(idx < array.length){
      sum += array[idx];
      count += 1;
    }
  }
  answer.push(sum/count);
}
```
안쪽 반복문은 array에 있는 값들 중 어떤 범위의 값을 반복, **범위로 새로운 배열**을 만들어서 개선   
   
**배열 전체를 다루기**
```ts
const answer = [];
const window = 5;
for(let i = 0; i < array.length; i++) {
  let sum = 0;
  let count = 0;
  const subArray = array.slice(i, i + window); //새로운 배열을 만들어서 배열 전체를 다루도록 함
  answer.push(average(subArray));
}
```
slice로 새로운 배열을 만들었기 때문에 함수형 도구를 사용할수 있어 안쪽 반복문을 없앰
   
**작은 단계로 나누기**
```ts
const window = 5;
const indices = range(0, array.length);
const windows = map(indices, (i) => { // 바로 평균을 구할수 있게 windows를 생성하는 단계로 나눔
  return array.slice(i, i + window);
})
const answer = map(windows, average); // window로 평균을 구하는 단계
```
작은 단계들로 나누어 더 명확해짐   
모든 반복문을 함수형 도구로 체이닝 함

### 절차적 코드와 함수형 코드 비교

```ts
// 절차적 코드
const answer = [];
const window = 5;
for(let i = 0; i < array.length; i++) {
  let sum = 0;
  let count = 0;
  for(let w = 0; w < window; w++) {  // 중첩된 반복문
    const idx = i + w; // 인덱스 계산
    if(idx < array.length){
      sum += array[idx]; // 지역변수 변경
      count += 1;
    }
  }
  answer.push(sum/count);
}

// 함수형 코드
const window = 5;
const indices = range(0, array.length); // 1. 인덱스 배열을 만듬
const windows = map(indices, (i) => { // 2. window를 만듬
  return array.slice(i, i + window);
})
const answer = map(windows, average); // 3. 평균값을 구함
```


절차적 코드일 때는 반복문이 중첩되고 인덱스를 계산하며 지역변수를 바꾸었지만,   
함수형 코드를 이용해 각 단계를 명확하게 만듬   

## 체이닝 디버깅을 위한 팁

1. 구체적인 것을 유지하기
데이터를 처리하는 과정에서 데이터가 어떻게 생겼는지 잊어버리기 쉽기 때문에 의미를 나타내는 이름을 붙여야 됨
2. 출력해보기
각 단계 사이에 console.log를 찍어서 어떻게 데이터가 변화하고 있는지 찍어본다
3. 타입을 따라가 보기
지정한 타입으로 데이터를 유추해볼수 있음

## 값을 만들기 위한 reduce()
장바구니 데이터를 읽어버렸지만 로깅으로 복구하는 상황으로 살펴보기

```ts
const itemsAdded = ["shirt", "shoes", ...]
const shoppingCart = reduce(itemsAdded, {}, addOne)
const addOne = (cart: Cart, item: Item) => {
  if(!cart[item]) {
    return addItem(cart, {name: item, quantity:1, price: priceLookup(item)});
  } else {
    const quantity = cart[item].quantity;
    return setFieldByName(cart, item, 'quantity', quantity + 1);
  }
}
```
장바구니에 추가하는건 충족했지만 삭제는 할수 없음

### 데이터를 사용해 창의적으로 만들기

로그를 아래와 같이 쌓았다면 삭제도 구현 가능
```ts
const itemOps = [
  ["add", "shirt"],
  ["add", "shoes"],
  ["remove", "shirt"],
  ...
]
```
```ts
const shoppingCart = reduce(itemOps, {}, (itemOp) => {
  const [op, item] = itemOp
  if(op === "add") return addOne(cart, item);
  if(op === "remove") return removeOne(cart, item);
})
const addOne = (cart: Cart, item: Item) => {
  if(!cart[item]) {
    return addItem(cart, {name: item, quantity:1, price: priceLookup(item)});
  } else {
    const quantity = cart[item].quantity;
    return setFieldByName(cart, item, 'quantity', quantity + 1);
  }
}
const removeOne = (cart: Cart, item: Item) => {
  if(!cart[item]) {
    return cart;
  } else {
    const quantity = cart[item].quantity;
    if(quantity === 1) {
      return removeItemByName(cart, item);
    } else {
      return setFieldByName(cart, item, 'quantity', quantity - 1);
    }
  }
}
```
인자를 데이터로 표현했다는 점이 중요   
추가나 삭제에 대한 동작도 데이터로 표현   
이런 방법은 함수형 프로그래밍에서 자주 사용하는 방법

> [!NOTE]
> "인자를 데이터로 만들면 함수형 도구를 체이닝 하기 좋다"
> 라는 말을 **함수를 실행하기 위해 참고 할 수 있는 데이터를 만들면** 체이닝으로 묶기 좋다 라고 이해했습니다.