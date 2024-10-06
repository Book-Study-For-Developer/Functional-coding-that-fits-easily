## MegaMart 에서 커뮤니케이션팀 만들기

첫번째 요구사항

데이터 요청: 쿠폰 이메일 처리

```tsx
// 3장에서 나왔던 코드
function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  for (var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  }
  return emails;
}

// forEach() 함수로 바꾼 코드
function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  forEach(customers, function (customer) {
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  });
  return emails;
}
```

map()

배열을 하나씩 변환해서 같은 길이의 배열로 만들어 주는 함수

```tsx
function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  for (var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  }
  return emails;
}

function biggestPurchasePerCustomer(customers) {
  var purchases = [];
  for (var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var purchase = biggestPurchase(customer);
    purchases.push(purchase);
  }
  return purchases;
}

function customerFullNames(customers) {
  var fullNames = [];
  for (var i = 0; i < customers.length; i++) {
    var cust = customers[i];
    var name = cust.firstName + " " + cust.lastName;
    fullNames.push(name);
  }
  return fullNames;
}

function customerCities(customers) {
  var cities = [];
  for (var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var city = customer.address.city;
    cities.push(city);
  }
  return cities;
}
// 위 네개의 함수의 코드가 매우 비슷함

// 콜백으로 바꾼 코드
function emailsForCustomers(customers, goods, bests) {
  return map(customers, function (customer) {
    return emailForCustomer(customer, goods, bests);
  });
}

function map(array, f) {
  var newArray = [];
  forEach(array, function (element) {
    newArray.push(f(element));
  });
  return newArray;
}
```

## 함수형 도구: map()

!https://drek4537l1klr.cloudfront.net/normand/Figures/f0315-01.jpg

```tsx
function map(array, f) {
  // 배열과 함수를 인자로 받고
  let newArray = []; // 빈 배열을 만든다
  forEach(array, function (element) {
    // 원래 배열 항목으로 새로운 항목을 만들기 위해 f()함수를 부르고
    // 원래 배열 항목에 해당하는 새로운 항목을 추가한다
    newArray.push(f(element));
  });
  return newArray; // 새로운 배열을 리턴한다
}
```

map()에 넘기는 함수가 계산일 때 사용하기가 쉽다

### 함수를 전달하는 세가지 방법

1. 전역으로 정의하기

   어디서나 이름으로 함수를 참조 가능

   ```tsx
   function greet(name) {
     return "Hello, " + name;
   }
   ```

2. 지역으로 정의하기

   함수 안에서 이름을 붙여 정의하고 해당 함수 안에서 참조 가능

   ```tsx
   function greetEverybody(friends) {
     let greeting;
     if (language === "English") {
       greeting = "Hello, ";
     } else {
       greeting = "Salut, ";
     }

     let greet = function (name) {
       return greeting + name;
     };
     return map(friends, greet);
   }
   ```

3. 인라인으로 정의하기

   함수를 사용하는 곳에서 정의

   ```tsx
   const friendFreetings = map(friendsNames, function (name) {
     return "Hello, " + name;
   });
   ```

### 예제: 모든 고객의 이메일 주소

```
가진것: **고객** 배열
필요한 것: **고객 이메일 주소** 배열
함수: **고객** 하나를 받아 고객 **이메일 주소**를 리턴하는 함수
```

```tsx
map(customers, () => customer.email);
```

⚠️주의

고객 데이터에 이메일이 없어서 customer.email의 값이 null이나 undefined면 결과 배열 안에 null이 들어갈 것임

## 함수형 도구: filter()

!https://drek4537l1klr.cloudfront.net/normand/Figures/f0315-02.jpg

```tsx
function filter(array, f) {
  // 배열과 함수를 받고
  let newArray = []; // 빈 배열을 만들고
  forEach(array, function (element) {
    if (f(element)) {
      // f()를 호출해 항목을 결과 배열에 넣을지 확인하고
      newArray.push(element); //조건에 맞으면 원래 항목을 결과 배열에 넣고
    }
  });
  return newArray; // 결과 배열을 리턴
}
```

불리언 타입을 리턴하는 함수를 전달하여 결과 배열에 넣을지 확인한다

map()처럼 전달하는 함수가 계산일 때 사용하기 쉽다

### 예제: 아무것도 구입하지 않은 고객

```
가진것: **고객** 배열
필요한 것: **아무것도 구입하지 않은 고객** 배열
함수: **고객** 하나를 받아 **아무것도 구입하지 않았다면** true를 리턴
```

```
filter(customers, (customer) => customer.purchases.length === 0)
```

⚠️주의

```tsx
const allEmails = map(customers, function (customer) {
  return customer.email; // 고객 이메일이 null이면 결과에 null이 들어감
});

const emailsWithoutNulls = filter(emailsWithNulls, function (email) {
  return email !== null; // filter를 사용해 null을 없앰
});
```

## 함수형 도구: reduce()

!https://drek4537l1klr.cloudfront.net/normand/Figures/f0315-03.jpg

```tsx
function reduce(array, init, f) {
  // 배열과 초기값, 누적 함수를 받아
  let accum = init; // 누적된 값을 초기화
  forEach(array, function (element) {
    accum = f(accum, element); // 누적 값을 계산하기 위해 현재 값과 배열 항목으로 f()함수를 부른다
  });
  return accum; // 누적값 리턴
}
```

배열을 순회하면서 값을 누적

```tsx
function countAllPurchase(customers) {
  return reduce(customers, 0, function (total, customer) {
    return total + customer.purchase.length;
  });
}

// reduce에 배열 customers, 초기값 0 전달
// reduce에 전달하는 함수는 인자가 2개이어야 하고, 리턴 값은 첫번째 인자와 타입이 같아야 함
// 지금까지 누적한 합계와 현재 고객이 구입한 제품의 개수를 더한 값을 리턴
```

### 예제: 문자열 합치기

```
가진것: **문자열** 배열
필요한 것: 배열에 있는 모든 문자열을 하나로 **합친** 문자열
함수: **누적된 문자열**과 **배열에 있는 현재 문자열**을 받아서 합치는 함수
```

```tsx
reduce(strings, '', (accum, string) => accum + string;)
```

⚠️주의

1. 인자의 순서

   인자가 3개이고, reduce에 전달하는 함수의 인자는 2개이기 때문에 순서 혼동 주의

2. 초기값을 결정하는 방법

   초기값은 동작과 문맥에 따라 다르지만 아래 두가지 질문에 같은 답을 해야한다

   - 계산이 어떤 값에서 시작되는가?
   - 배열이 비어있다면 어떤 값을 리턴할 것인가?

### reduce()로 할 수 있는 것들

- 실행 취소/ 실행 복귀
- 테스트할 때 사용자 입력을 다시 실행하기
- 시간 여행 디버깅
- 회계 감사 추적

💬 별 생각 없이 익숙하게 사용하던 map, filter, reduce 함수를 뜯어 볼 수 있는 기회가 되어서 좋았다 😊
