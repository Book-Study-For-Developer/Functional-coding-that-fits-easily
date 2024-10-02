## 예제를 통해 map() 함수를 도출하기

```tsx
function emailsForCustomers(customers, goods, bests) {
	const emails = [];
	forEach(customers, function(customers) {
		const email = emailForCustomer(customer, goods, bests);
		emails.push(email);
	})
	return emails;
}

// 콜백으로 바꾼 버전
function emailsForCustomers(customers, goods, bests) {
	return map(customers, function (customers) {
		return emailForCustomer(customer, goods, bests);
	});
}

function map(array, f) {
	const newArray = [];
	
	forEach(array, function(element) {
		newArray.push(f(element));
	});
	
	return newArray;
}
```

일반적으로 사용할 수 있는 반복 함수 map을 만들었다.

**map을 사용할 때 주의해야 할 점**

데이터의 값이 null이나 undefined라면 배열안에 null 값이 들어가게 될 것이다.

그렇기에 이를 해결하기 위해서는 조심해서 사용하거나, null을 허용하지 않는 언어를 사용하는 것이다.
(filter를 사용하면 도움이 된다.)

```tsx
const customers = [{
    name: '고객1',
    email: 'customer1@email.com'
},{
    name: '고객2',
    email: 'customer2@email.com'
},{
    name: '고객3',
    email: null
},{
    name: '고객4',
    email: 'customer4@email.com'
}]

const greetForCustomer = customers.map((customer) => customer.email)

// 결과
// 3번 고객의 경우 null 값이 들어가버림
['customer1@email.com', 'customer2@email.com', null, 'customer4@email.com']
```

## 예제를 통해 filter() 함수 도출하기

```tsx
function selectBestCustomers(customers) {
  const newArray = [];
  forEach(customers, function(customer) {
    if(customer.purchases.length >= 3)
      newArray.push(customer);
  });
  return newArray;
}

// 콜백으로 바꾼 버전
function selectBestCustomers(customers) {
  return filter(customers, function(customer) {
    return customer.purchases.length >= 3;
  });
}

function filter(array, f) {
  const newArray = [];
  forEach(array, function(element) {
    if(f(element))
      newArray.push(element);
  });
  return newArray;
}
```

filter를 사용하게 되면 앞에 설명했던 map 함수에 주의해야 했던 점을 해결할 수 있다.

```tsx
const customers = [{
    name: '고객1',
    email: 'customer1@email.com'
},{
    name: '고객2',
    email: 'customer2@email.com'
},{
    name: '고객3',
    email: null
},{
    name: '고객4',
    email: 'customer4@email.com'
}]

const greetForCustomer = customers.filter((customer) => customer.email !== null)

// 결과
// null인 고객은 제외하고 나타남
['customer1@email.com', 'customer2@email.com', 'customer4@email.com']
```

## 예제를 통해 reduce() 도출하기

```tsx
function countAllPurchases(customers) {
  const total = 0;
  forEach(customers, function(customer) {
    total = total + customer.purchases.length;
  });
  return total;
}

// 콜백으로 바꾼 버전
function countAllPurchases(customers) {
  return reduce(customers, 0, function(total, customer) {
    return total + customer.purchase.length;
  });
}

function reduce(array, init, f) {
  const accum = init;
  forEach(array, function(element) {
    accum = f(accum, element);
  });
  return accum;
}
```

reduce를 사용할 때 주의해야할 점

인자의 순서를 조심해야 함,

- 계산이 어떤 값에서 시작되는가? ⇒ 더하기는 0, 곱하기는 1
- 배열이 비어있다면 어떤 값을 리턴할 것인가?

> **여기까지 느낀 점.. 😎**
그냥 javascript 고차함수에 대해서 직접 구현하고 설명한 느낌이 강했다. (저자가 es5이전에 썼나? 했는데 뒤에보니(13장) 저자도 javascript 내장함수를 알고 있는데 아마 책에서는 다른 언어에서도 사용할 수 있는 것을 보여주기 위해 직접 구현한 것 같다)
> 

reduce로 할 수 있는 것들

- 실행 취소/실행 복귀
- 테스트할 때 사용자 입력 다시 실행하기(사용자의 입력이 계속해서 업데이트될때 이값을 최신값으로 맞출 수 있다.)
- 시간 여행 디버깅: 어떤 특정한 시점으로 되돌릴 수 있다.
- 회계 감사 추적: 과거에 어떤 일이 있었는지 기록할 수 있다.(accumulate를 사용하기 때문?)

### reduce로 map과 filter 만들기

javascript에서 쓸 수 있는 고차함수로 만들었다(책은 직접 구현)

```tsx
function map(array, f) {
  return array.reduce((acc, x) => {
    acc.push(f(x));
    return acc;
  }, []);
}
// 사용 예시
map([1, 2], ((number) => number + 1))
// 결과 [2, 3]

function filter(array, f) {
  return array.reduce((acc, x) => {
	  if(f(x)) {
		  acc.push(x);
	  }
    return acc;
  }, []);
}
// 사용 예시
filter([1, 2], ((number) => number % 2 === 0))
// 결과 [2]
```