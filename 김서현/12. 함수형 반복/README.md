# 12. 함수형 반복

- [map()](#map)
- [filter()](#filter)
- [reduce()](#reduce)

## map()

```tsx
function map(array, f) {
  // 배열, 함수를 받음
  var newArray = []; // 빈 배열
  forEach(array, function (element) {
    // 원래 배열 항목으로 새로운 항목을 만듦
    newArray.push(f(element));
  });
  // 새로운 배열 리턴
  return newArray;
}

map(customers, (customer) => customer.email);
```

- X(어떤 값의 집합) 값이 있는 배열을 Y(또 다른 값의 집합) 값이 있는 배열로 변환
- X를 Y로 바꾸는 함수: X를 인자로 받아 Y를 리턴
- map()에 계산을 넘기면 map()을 사용하는 코드도 계산임, 액션을 넘기면 액션이 됨


---


## filter()

```tsx
function filter(array, f) {
  // 배열, 함수를 받음
  var newArray = [];
  forEach(array, function (element) {
    // f()를 호출해 항목을 결과 배열에 넣을지 확인
    if (f(element)) {
      // 조건에 맞으면 원래 항목을 결과 배열에 넣음
      newArray.push(element);
    }
  });
  // 결과 배열 리턴
  return newArray;
}

var emailsWithoutNulls = filter(emailsWithNulls, (email) => email !== null);
```

- 원래 배열을 가지고 새로운 배열을 만드는 고차 함수
- 항목이 X인 배열에 filter()를 사용해도 결과는 여전히 항목이 X인 배열이지만 남길 배열을 선택하기에 원래 가진 항목보다 적을 수 있음.
- 전달하는 함수가 계산일 때 가장 사용하기 쉬움


---


## reduce()

```tsx
function reduce(array, init, f) {
  // 배열, 추깃값, 누적 함수를 받음
  // 누적된 값을 초기화
  var accumulator = init;
  forEach(array, function (element) {
    // 누적 값 계산하기 위해 현재 값과 배열 항목으로 f() 함수 부름
    accumulator = f(accumulator, element);
  });
  // 누적된 값 리턴
  return accumulator;
}

// reduce()에 전달하는 함수는 인자 두 개 (현재 누적된 값, 배열의 현재 항목) 받음
reduce(strings, "", (accum, string) => accum + string);
```

- 배열 순회하며 값을 누적해가는 고차 함수
- 전달하는 함수를 통해 누적하는 방법 결정
- reduce()에 전달하는 함수는 인자로 현재 누적된 값과 배열의 현재 항목을 받고 첫 번째 인자와 같은 타입의 값을 리턴해야 함
- 초깃값 결정할 때 주의
  - 계산이 어떤 값에서 시작되는가 - 더하기면 초깃값 0, 곱하기 하면 초깃값 1
  - 배열이 비어 있다면 어떤 값을 리턴할 것인가
- reduce()로 할 수 있는 것들
  - map(), filter() 만들기
  - 실행 취소/실행 복귀 - 실행 취소: 리스트의 마지막 사용자 입력을 없애는 것
  - 테스트할 때 사용자 입력을 다시 실행하기 - 초깃값은 시스템 처음 상태이고 사용자 입력이 순서대로 리스트에 있을 때 reduce()로 모든 값 합쳐 현재 상태 만들 수 있음
  - 시간 여행 디버깅 - 어떤 언어는 변경 사항을 어떤 시점으로 되돌릴 수 있음. 뭔가 잘못 동작하면 특정 시점 상태의 값 보관, 문제 고치고 새로운 코드로 실행해볼 수있음.
  - 회계 감사 추적
