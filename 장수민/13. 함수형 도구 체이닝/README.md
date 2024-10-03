## 함수형 도구 체이닝(Chapter 13)

12장에서는 함수형 도구 '하나'로 작업을 진행했지만, 계산이 더 복잡해지면 그럴 수 없다.

그래서 🎈여러 단계를 하나로 엮은 **체인**으로 복합적인 계산을 표현하는 방법에 대해서 알아보려고 한다. 🎈

---

### 고객 커뮤니케이션팀은 계속 일하고 있습니다

**요구사항** : 우수 고객이 가장 많은 비용을 쓸 것으로 생각하고 있습니다. 그래서 각각의 우수 고객(3개 이상 구매)의 구매 중 가장 비싼 구매를 알려주세요.

이 과정을 여러 단계로 나누고 순서대로 나열하자면,

1. 우수 고객(3개 이상 구매)을 거릅니다. - `filter()`
2. 우수 고객을 가장 비싼 구매로 바꿉니다 - `map()`

우수 고객을 거르는 코드는 다음과 같다.

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });
}
```

다음으로 할 일은 각 고객의 가장 비싼 구매를 가져와서 배열에 담는 일이다.

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });

  const biggestPurchases = map(bestCustomers, function (customer) {
    return ...;
  });
}
```

가장 큰 수를 찾는 방법은 reduce()를 사용할 수 있다.

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });

  const biggestPurchases = map(bestCustomers, function (customer) {
    return reduce(
      customer.purchases,
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
}
```

위 코드는 잘 동작하는 코드이지만, 콜백 지옥때문에 함수가 너무 커졌다...! 코드를 더 깨끗하게 만들어보자!

reduce 함수 부분을 `maxKey`라는 이름의 콜백으로 분리하면 다음과 같다.

```tsx
maxKey(customer.purchases, { total: 0 }, function (purchase) {
  return purchase.total;
});

function maxKey(array, init, f) {
  return reduce(array, init, function (biggestSoFar, element) {
    if (f(biggestSoFar) > f(element)) {
      return biggestSoFar;
    } else {
      return element;
    }
  });
}
```

그러면 원래 코드는 다음과 같아진다.

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, function (customer) {
    return customer.purchases.length >= 3;
  });

  const biggestPurchases = map(bestCustomers, function (customer) {
    return maxKey(bestCustomers, { total: 0 }, function (purchase) {
      return purchase.total;
    });
  });
}
```

---

### 체인을 명확하게 만들기 1: 단계에 이름 붙이기

각 단계의 고차 함수를 빼내 이름을 붙일 수 있다.

```tsx
// 🔔콜백 지옥으로 인해 떨어진 가독성을 async-await로 개선하는 느낌과 비슷합니다...!🔔

function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = selectBestCustomers(customers); // 우수 고객을 고른다라는 의미가 명확히 드러남.
  const biggestPurchases = getBiggestPurchases(bestCustomers); // 우수 고객들의 가장 큰 구매를 얻는다는 의미가 명확히 드러남.
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

### 체인을 명확하게 만들기 2: 콜백에 이름 붙이기

익명 함수를 자주 쓰고 있어서 재사용하기 어렵다. 콜백을 빼내고 이름을 붙여 재사용할 수 있는 함수로 만들어보자.

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, isGoodCustomer);
  const biggestPurchases = map(customers, getBiggestPurchase);
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

호출 그래프의 아래쪽에 위치하므로 재사용하기 좋은 코드다!

### 체인을 명확하게 만들기 3: 두 방법을 비교

**방법 1: 단계에 이름 붙이기** vs **방법 2: 콜백에 이름 붙이기**

책에서 얘기하길,

- 일반적으로 두 번째 방법(콜백에 이름 붙이기)이 더 명확하다고 한다.
- 이름을 붙인 두 번째 방법이 재사용하기도 더 좋다.
- 인라인 함수를 콜백으로 사용하면 단계가 중첩되는데, 이를 막을 수 있다.
- (개인적으로는) 함수를 작성하면서 중괄호를 열면 줄이 넘어가는데, 이러한 문제를 막을 수 있어서 좋다고 생각한다...!

> [!NOTE]
> 하지만, 사용하는 언어의 문법과 문맥에 따라 달라지므로, 두 가지 방법을 모두 시도해서 어떤 방법이 더 좋은지 코드를 비교해 결정하자!

---

### 연습 문제

**요구 사항**: 마케팅팀은 구매 금액이 최소 100달러를 넘고(AND) 두 번 이상 구매한 고객을 찾으려고 합니다. 두 가지 조건을 만족하는 고객을 큰 손이라고 합니다. 함수형 도구를 체이닝해서 bigSpenders() 함수를 만들어 보세요. 깔끔하고 읽기 쉽게 만들어야 합니다.

```tsx
function bigSpenders(customers) {
  const withBigPurchases = filter(customers, hasBigPurchase); // 최소 100달러를 넘는지 한 번 거르고
  const withSpendTwice = filter(withBigPurchases, doSpendTwice); // 위 조건을 만족하는 고객 중에 두 번 이상 구매했는지 또 거르기
  return withSpendTwice; // 🎃 하지만, 이러면 읽는 사람 입장에서는 두 번 이상 구매한 고객 배열만 반환됐다고 생각할 수 있지 않을까요?
}

function hasBigPurchase(customer) {
  return filter(customer.purchases, doSpendMoreThan100).length > 0;
}

function doSpendMoreThan100(purchase) {
  return purchase.total > 100;
}

function doSpendTwice(customer) {
  return customer.purchases.length >= 2;
}
```

> [!NOTE]
> filter와 map 고차 함수는 모두 새로운 배열을 만드므로, 비효율적이라고 생각할 수 있지만 대부분 문제가 되지 않는다! 만들어진 배열이 필요 없을 때 가비지 컬렉터가 빠르게 처리하기 때문이다.

만약 비효율적인 경우라면 **스트림 결합(stream fusion)**을 사용해볼 수 있다. 병목이 생겼을 때만 사용하는 것이 좋고, 대부분의 경우에는 여러 단계를 사용하는 것이 더 명확하고 읽기 쉽다!

```tsx
// 스트림 결합이 이뤄지지 않았을 때 - 가비지 컬렉터가 필요
const names = map(customers, getFullName);
const nameLengths = map(names, stringLength);

// map()을 한 번 사용해도 똑같이 동작하도록 만들 수 있다. - 가비지 컬렉터 필요 없음.
const nameLengths = map(customers, function (customer) {
  return stringLength(getFullName(customer)); // 스트림 결합
});
```

---

### 반복문을 함수형 도구로 리팩터링하기

어떤 때에는 기존에 있는 반복문을 함수형 도구로 리팩터링해야 하는 경우가 있다.

**전략 1: 이해하고 다시 만들기**

단순히 반복문을 읽고 어떤 일을 하는지 파악한 다음에 구현을 잊어버리는 것이다.

**전략 2: 단서를 찾아 리팩터링**

반복문을 하나씩 선택한 다음 함수형 도구 체인으로 바꾸면 된다. 이 때, 안쪽 반복문이 리팩터링을 시작하기 좋은 위치이다.

#### 팁 1: 데이터 만들기

```tsx
const answer = [];
const window = 5;

for (let i = 0; i < array.length; i++) {
  let sum = 0;
  let count = 0;
  for (let w = 0; w < window; w++) {
    // 배열에 들어있는 값으로 순회를 하진 않는다.
    const idx = i + w; // 배열로 만들지 않는다.
    if (idx < array.length) {
      sum += array[idx]; // 배열로 따로 만들지 않는다.
      count += 1;
    }
  }
  answer.push(sum / count);
}
```

이 범위의 값을 배열로 만들어 반복해보자.

```tsx
const answer = [];
const window = 5;

for (let i = 0; i < array.length; i++) {
  let sum = 0;
  let count = 0;
  const subarray = array.slice(i, i + window); // 배열로 따로 만든다.
  for (let w = 0; w < subarray.length; w++) {
    sum += array[w];
    count += 1;
  }
  answer.push(sum / count);
}
```

#### 팁 2: 한 번에 전체 배열을 조작하기

average() 함수를 재사용해서 리팩터링하자.

```tsx
const answer = [];
const window = 5;

for (let i = 0; i < array.length; i++) {
  const subarray = array.slice(i, i + window);
  answer.push(average(subarray));
}
```

#### 팁 3: 작은 단계로 나누기

배열의 항목이 아니라 인덱스를 가지고 반복해야 하는 문제가 있어서 더 작은 단계로 나눠야 한다.

```tsx
const indices = [];

for (let i = 0; i < array.length; i++) indices.push(i);

const window = 5;

const answer = map(indices, function (i) {
  const subarray = array.slice(i, i + window);
  return average(subarray);
});
```

하지만 map()에 전달된 콜백 함수가 하는 일은 두 가지 이므로 이를 나누면 더 명확해진다!

```tsx
function range(start, end) {
  const ret = [];
  for (let i = start; i < end; i++) {
    ret.push(i);
  }
  return ret;
}

const window = 5;

const indices = range(0, array.length);
const windows = map(indices, function (i) {
  return array.slice(i, i + window);
});
const answer = map(windows, average);
```

### 체이닝 디버깅을 위한 팁

1. 구체적인 것을 유지하기
   - 각 단계에서 어떤 것을 하고 있는지 알기 쉽게 이름을 잘 지어야 한다.(map과 같은 고차 함수를 쓸 때, 변수명을 잘 짓자...!)
2. 출력해보기
3. 타입을 따라가 보기
