## 새로운 요구사항

각각의 우수 고객의 구매 중 가장 비싼 구매를 알려주세요

1. 우수 고객을 거른다(filter)

   ```tsx
   function biggestPurchasesBestCustomers(customers) {
     const bestCustomers = filter(customers, function (customer) {
       return customer.purchases.length >= 3;
     });
   }
   ```

2. 우수 고객을 가장 비싼 구매로 바꾼다(map)

   ```tsx
   function biggestPurchasesBestCustomers(customers) {
     const bestCustomers = filter(customers, function (customer) {
       return customer.purchases.length >= 3;
     });

     const biggestPurchases = map(bestCustomers, function (customer) {
       return reduce(
         customer.purchase,
         { total: 0 },
         function (biggestSoFar, purchase) {
           if (biggestSoFar.total > purchase.total) {
             return biggestSoFar;
           } else {
             return purchase;
           }
         }
       );
     });
     return biggestPurchases;
   }
   ```

위의 코드는 잘 동작하는 코드이지만, 콜백이 여러개가 중첩되어 읽기 어렵다

```tsx
// 콜백으로 빼내기
function maxKey(array, init, f) {
  return reduce(array, init, function (biggestSoFar, element) {
    if (f(biggestSoFar) > f(element)) return biggestSoFar;
    else return element;
  });
}

function biggestPurchasesBestCustomers(customers) {
  var bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });
  var biggestPurchases = map(bestCustomers, function (customer) {
    return maxKey(customer.purchases, { total: 0 }, function (purchase) {
      return purchase.total;
    });
  });
  return biggestPurchases;
}
```

## 체인을 명확하게 만들기 1: 단계에 이름 붙이기

각 단계의 고차 함수를 빼내 이름을 붙이기

```tsx
function biggestPurchasesBestCustomers(customers) {
  var bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });
  var biggestPurchases = map(bestCustomers, function (customer) {
    return maxKey(customer.purchases, { total: 0 }, function (purchase) {
      return purchase.total;
    });
  });
  return biggestPurchases;
}

function biggestPurchasesBestCustomers(customers) {
  var bestCustomers = selectBestCustomers(customers);
  var biggestPurchases = getBiggestPurchases(bestCustomers);
  return biggestPurchases;
}

function selectBestCustomers(customers) {
  return filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });
}

function getBiggestPurchases(customers) {
  return map(customers, getBiggestPurchase);
}

function getBiggestPurchase(customer) {
  return maxKey(customer.purchases, { total: 0 }, function (purchase) {
    return purchase.total;
  });
}
```

각 단계에 이름을 붙이면 훨씬 명확해지고 숨어있던 두 함수의 구현도 알아보기 쉽다

그러나, 콜백 함수는 여전히 인라인으로 사용하고 있고, 인라인으로 정의된 콜백 함수는 재사용할 수 없다

## 체인을 명확하게 만들기 2: 콜백에 이름 붙이기

콜백을 빼내어 이름 붙여 재사용할 수 있는 함수로 만들기

```tsx
function biggestPurchasesBestCustomers(customers) {
  var bestCustomers = filter(customers, isGoodCustomer);
  var biggestPurchases = map(bestCustomers, getBiggestPurchase);
  return biggestPurchases;
}

function isGoodCustomer(customer) {
  return customer.purchases.length >= 3;
}

function getBiggestPurchase(customer) {
  return maxKey(customer.purchases, { total: 0 }, getPurchaseTotal);
}

function getPurchaseTotal(purchase) {
  return purchase.total;
}
```

## 체인을 명확하게 만들기 3: 두 방법을 비교

단계에 이름 붙이기 vs 콜백에 이름 붙이기

→ 일반적으로 ‘콜백에 이름 붙이기’ 방법이 더 명확하다

- 재사용하기 good
- 단계가 중첩되는 것도 막을 수 있음

💬 나는 첫번째 방법을 더 많이 사용하고 있던 것 같다.

## 예제: 한번만 구매한 고객의 이메일 목록

```tsx
가진 것: 전체 고객 배열
필요한 것: 한 번만 구매한 고객들의 이메일 목록
계획:
1. 한번만 구매한 고객을 거른다(filter)
2. 고객 목록을 이메일 목록으로 바꾼다(map)
```

```tsx
const firstTimers = filter(customers, function (customer) {
  return customer.purchases.length === 1;
});
const firstTimerEmails = map(firstTimers, function (customer) {
  return customer.email;
});

// 콜백에 이름 붙이기
const firstTimers = filter(customers, isFirstTimer);
const firstTimerEmails = map(firstTimers, getCustomerEmail);

function isFirstTimer(customer) {
  return customer.purchases.length === 1;
}

function getCustomerEmail(customer) {
  return customer.email;
}
```

- **스트림결합**

map, filter, reduce 체인을 최적화 하는 것

```tsx
// 값 하나에 map 2번 사용
const names = map(customers, getFullName);
const nameLengths = map(names, stringLength);

// map 1번 사용
const nameLengths = map(customers, function (customer) {
  return stringLength(getFullName(customer));
});
```

## 반복문을 함수형 도구로 리팩터링하기

기존에 있던 반복문을 함수형 도구로 리팩터링하기

반복문을 하나씩 선택한 다음 함수형 도구 체인으로 바꾸기

```tsx
var answer = [];
var window = 5;
for (var i = 0; i < array.length; i++) {
  var sum = 0;
  var count = 0;
  for (var w = 0; w < window; w++) {
    var idx = i + w;
    if (idx < array.length) {
      sum += array[idx];
      count += 1;
    }
  }
  answer.push(sum / count);
}
```

원래 배열 크기만큼 answer 배열에 항목을 추가하고 있다 → map

배열을 돌면서 항목을 값 하나로 만들고 있음 → reduce

## 팁1: 데이터 만들기

데이터를 배열에 넣으면 함수형 도구를 쓸 수 있다는 것

```tsx
var answer = [];
var window = 5;

for (var i = 0; i < array.length; i++) {
  var sum = 0;
  var count = 0;
  for (var w = 0; w < window; w++) {
    var idx = i + w;
    if (idx < array.length) {
      sum += array[idx];
      count += 1;
    }
  }
  answer.push(sum / count);
}

// 어떤 범위에 값을 배열로 만들어 반복하기

var answer = [];
var window = 5;

for (var i = 0; i < array.length; i++) {
  var sum = 0;
  var count = 0;
  var subarray = array.slice(i, i + window);
  for (var w = 0; w < subarray.length; w++) {
    sum += subarray[w];
    count += 1;
  }
  answer.push(sum / count);
}
```

## 팁2: 한 번에 전체 배열을 조작하기

하위 배열을 만들었기 때문에 일부 배열이 아닌 배열 전체를 반복할 수 있음

average함수를 재사용해서 반복문 바꾸기

```tsx
var answer = [];
var window = 5;

for (var i = 0; i < array.length; i++) {
  var subarray = array.slice(i, i + window);
  answer.push(average(subarray));
}
```

## 팁3: 작은 단계로 나누기

하위 배열을 만들기 위해 반복문의 인덱스를 사용

인덱스를 생성하는 작은 단계를 만들고 각 항목마다 인덱스를 가지고 콜백을 부른다

```tsx
var indices = [];

for (var i = 0; i < array.length; i++) indices.push(i);

var window = 5;
var answer = map(indices, function (i) {
  // 인덱스 배열 생성
  var subarray = array.slice(i, i + window); // 하위 배열 만들기
  return average(subarray); // 평균 계산
});
```

## 체이닝 팁 요약

- 데이터 만들기
- 배열 전체를 다루기
- 작은 단계로 만들기
- 보너스: 조건문을 filter로 바꾸기
- 보너스: 유용한 함수로 추출하기
- 보너스: 개선을 위해 실험하기

## 체이닝 디버깅을 위한 팁

- 구체적인 것을 유지하기
- 출력해보기
- 타입을 따라가보기

## 값을 만들기 위한 reduce()

reduce는 값을 요약하기도 하지만 값을 만들기도 한다

```tsx
var itemsAdded = ["shirt", "shoes", "shirt", "socks", "hat", "...."];

var shoppingCart = reduce(itemsAdded, {}, function (cart, item) {
  if (!cart[item])
    return add_item(cart, {
      name: item,
      quantity: 1,
      price: priceLookup(item),
    });
  else {
    var quantity = cart[item].quantity;
    return setFieldByName(cart, item, "quantity", quantity + 1);
  }
});
```

콜백함수를 빼내 이름을 붙이기

```tsx
var shoppingCart = reduce(itemsAdded, {}, addOne);

function addOne(cart, item) {
  if (!cart[item])
    return add_item(cart, {
      name: item,
      quantity: 1,
      price: priceLookup(item),
    });
  else {
    var quantity = cart[item].quantity;
    return setFieldByName(cart, item, "quantity", quantity + 1);
  }
}
```

## 데이터를 사용해 창의적으로 만들기

제품을 추가한 경우와 삭제한 경우 모두 처리

```tsx
var itemOps = [
  ["add", "shirt"],
  ["add", "shoes"],
  ["remove", "shirt"],
  ["add", "socks"],
  ["remove", "hat"],
];

var shoppingCart = reduce(itemOps, {}, function (cart, itemOp) {
  var op = itemOp[0];
  var item = itemOp[1];
  if (op === "add") return addOne(cart, item);
  if (op === "remove") return removeOne(cart, item);
});

function removeOne(cart, item) {
  if (!cart[item]) return cart;
  else {
    var quantity = cart[item].quantity;
    if (quantity === 1) return remove_item_by_name(cart, item);
    else return setFieldByName(cart, item, "quantity", quantity - 1);
  }
}
```
