## 새로운 요구사항에 따른 체이닝 기법

새로운 요구사항 

- 각각의 우수 고객(3개 이상 구매)의 구매 중 가장 비싼 구매를 알려주세요.

1. 우수 고객을 거른다(filter)

```tsx
function biggestPurchasesBestCustomers(customers) {
	const bestCustomers = filter(customers, function(customer) {
		return customer.purchases.length >= 3;
	});
}
```

1. 우수 고객을 가장 비싼 구매로 바꾼다(map)

```tsx
function biggestPurchasesBestCustomers(customers) {
	const bestCustomers = filter(customers, function(customer) {
		return customer.purchases.length >= 3;
	});
	
	const biggestPurchases = map(bestCustomers, function(customer) {
    // 가장 비싼 구매를 찾는 코드
		return reduce(customer.purchase, { total: 0 }, function(biggestSoFar, purchase) {
			if (biggestSoFar.total > purchase.total) {
				return biggestSoFar;
			} else {
				return purchase;
			}
		}
	}
}
```

위의 코드는 잘 동작하는 코드이지만 콜백이 중첩되어 좋지 않은 코드가 되었다.

중첩된 코드를 콜백으로 분리하자

```tsx
function biggestPurchasesBestCustomers(customers) {
	const bestCustomers = filter(customers, function(customer) {
		return customer.purchases.length >= 3;
	});
	
	const biggestPurchases = map(bestCustomers, function(customer) {
    // 가장 비싼 구매를 찾는 코드
		return maxKey(customer.purchase, { total: 0 }, function(purchase) {
			return purchase.total;
		})
}

function maxKey(array, init, f) {
	return reduce(array, init, function(biggestSoFar, element) {
		if (biggestSoFar.total > purchase.total) {
				return biggestSoFar;
		} else {
				return purchase;
		}
	})
}
```

reduce는 배열의 값을 조합한다는 의미 말고 특별한 의미가 없다. 
반면에 maxKey의 함수는 더 구체적인 함수이고 배열에서 가장 큰 값을 선택한다는 의미가 담겨있다.

### 체인을 명확하게 만들기 1: 단계에 이름 붙이기

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = selectBestCustomers(customers); // 1단계
  const biggestPurchases = getBiggestPurchases(bestCustomers); // 2단게
  return biggestPurchases;
}

function selectBestCustomers(customers) {
  return filter(customers, function(customer) {
    return customer.purchases.length >= 3;
  });
}

function getBiggestPurchases(customers) {
  return map(customers, getBiggestPurchase);
}

function getBiggestPurchase(customer) {
  return maxKey(customer.purchases, {total: 0}, function(purchase) {
    return purchase.total;
  });
}
```

콜백 함수가 여전히 인라인으로 사용되고 있다. ⇒ 재사용이 어려움

### 체인을 명확하게 만들기 2: 콜백에 이름 붙이기

```tsx
function biggestPurchasesBestCustomers(customers) {
  const bestCustomers = filter(customers, isGoodCustomer);
  const biggestPurchases = map(bestCustomers, getBiggestPurchase);
  return biggestPurchases;
}

function isGoodCustomer(customer) {
  return customer.purchases.length >= 3;
}

function getBiggestPurchase(customer) {
  return maxKey(customer.purchases, {total: 0}, getPurchaseTotal);
}

function getPurchaseTotal(purchase) {
  return purchase.total;
}
```

콜백 자체에 이름을 붙여서 재사용하기 쉬운 코드로 분리한다.

### 체인을 명확하게 만들기 3: 두 방법을 비교

두 방법 중에 어떤 것이 좋은지 비교해 결정하는 것이 좋다.

일반적으로 ‘콜백에 이름 붙이는 방법’ 이 더 명확하다. 
(음.. 나는 첫번째가 더 좋아보이긴 한다. 내부 구현은 숨기고 보는 쪽에서는 함수명만 보고 파악하도록?)

## 반복문을 함수형 도구로 리팩터링하기

기존의 반복문이 많은 코드를 리팩토링 해보자

```tsx
const answer = [];
const window = 5;

for(let i = 0; i < array.length; i++) {
  const sum = 0;
  const count = 0;
  for(let w = 0; w < window; w++) {
	  const idx = i + w;
	  if (idx < array.length) {
		  sum += array[idx];
			count += 1;
	  }
  }
  answer.push(sum/count);
}
```

### 1. 데이터 만들기

```tsx
const answer = [];
const window = 5;

for(let i = 0; i < array.length; i++) {
  const sum = 0;
  const count = 0;
  // 범위에 있는 값을 배열로 만들어서 반복시키면 중첩이 사라지게 된다.
  const subarray = array.slice(i, i + window);
  for(let w = 0; w < subarray.length; w++) {
    sum += subarray[w];
    count += 1;
  }
  answer.push(sum/count);
}
```

### 2. 한 번에 전체 배열을 조작하기

```tsx
const answer = [];
const window = 5;

for(let i = 0; i < array.length; i++) {
  const subarray = array.slice(i, i + window);
  // average 함수를 재사용한다.
  answer.push(average(subarray));
}
```

평균을 구하는 함수를 호출하여 내부 반복문을 제거한다.

### 3. 작은 단계로 나누기

인덱스를 가지고 반복문을 돌려야 하기 때문에 인덱스를 통해 배열을 만드는 것으로 해결한다.

```tsx
const answer = [];

const indices = [];
for(let i = 0; i < array.length; i++) {
	indices.push(i);
}

const window = 5;

const answer = map(indicies, function(i) {
  const subarray = array.slice(i, i + window);
  answer.push(average(subarray));
})

// map 안에서 하고 있는 일의 단계를 더 쪼개자
const indicis = range(0, array.length); // 인덱스 배열 생성 
const windows = map(indicies, function(i) { // 하위 배열 만들기
	return array.slice(i, i + window);
})
const answer = map(windows, average); // 평균 계산하기
```

최종적으로 절차적인 원래 코드에서 함수형 도구를 사용한 코드로 만들었다.

- 재사용하기 쉬워짐
- 테스트하기 쉬워짐
- 유지보수하기 쉬워짐

## 체이닝 디버깅을 위한 팁

고차 함수를 사용하는 것은 매우 추상적이기때문에 디버깅을 할때 어려울 수 있다.

- **구체적인 것을 유지하기**

단계가 많아질 수록 잊어버리기 쉬워진다. 각단계마다 어떤 것을 하는지 알기 쉽게 이름을 지어야 한다.

- **출력해보기**

중간에 데이터를 찍어보면서 다음 단계를 추가하는 것이 좋다 (ㅋㅋ 역시 모를땐 console.log)

- **타입을 따라가 보기**

각 단계에서 만들어지는 타입을 따라가보면 된다. 즉 콜백이 리턴하는 타입을 잘 확인하자(typescript를 잘쓰자?)

## 값을 만들기 위한 reduce

```tsx
const itemsAdded = ["shirt", "shoes", "shirt", "socks", "hat", "...."];

const shoppingCart = reduce(itemsAdded, {}, function(cart, item) {
  if(!cart[item]) {
	  // 추가하려고 하는 제품이 장바구니에 없으면 가져오기
    return add_item(cart, {name: item, quantity: 1, price: priceLookup(item)});
  } else {
    // 추가하려는 제품이 장바구니에 있으면 그대로 추가하기
    const quantity = cart[item].quantity;
    return setFieldByName(cart, item, 'quantity', quantity + 1);
  }
});

// 이를 약간의 리팩토링을 더 해보면,,
const shoppingCart = reduce(itemsAdded, {}, addOne);

function addOne(cart, item) {
  if(!cart[item]) {
	  // 추가하려고 하는 제품이 장바구니에 없으면 가져오기
    return add_item(cart, {name: item, quantity: 1, price: priceLookup(item)});
  } else {
    // 추가하려는 제품이 장바구니에 있으면 그대로 추가하기
    const quantity = cart[item].quantity;
    return setFieldByName(cart, item, 'quantity', quantity + 1);
  }
}
```

위의 코드가 의미하는 바를 더 살펴보자면, 제품을 추가한 기록이 모두 있어서 어느 시점이라도 장바구니를 만들 수 있다. 

예를 들어 되돌리기를 구현해야 할 경우 배열에서 마지막 항목만 없애주면 된다. 

```tsx
function removeOne(cart, item) {
  if(!cart[item])
    return cart;
  else {
    const quantity = cart[item].quantity;
    if(quantity === 1) {
      return remove_item_by_name(cart, item);
    } else {
      return setFieldByName(cart, item, 'quantity', quantity - 1);
    }
  }
}
```

제품을 삭제하는 것 까지 만들었으니 이제 제품을 추가했는지 삭제했는지만 기록하게 되면 제품을 처리하는데 용이해진다.

```tsx
const itemOps = [['add', "shirt"],
               ['add', "shoes"],
               ['remove', "shirt"],
               ['add', "socks"],
               ['remove', "hat"],
              ];

const shoppingCart = reduce(itemOps, {}, function(cart, itemOp) {
  const op = itemOp[0];
  const item = itemOp[1];
  if(op === 'add') return addOne(cart, item);
  if(op === 'remove') return removeOne(cart, item);
});
```

이때 인자(itemOp)를 데이터로 표현했다. 배열의 동작 이름과 제품 이름의 인자를 넣어 동작을 **완전한 데이터로 표현**했다.