## 일급 함수 II

### 카피-온-라이트 리팩터링하기

복사하고, 복사본을 변경하고, 복사본을 리턴하는 카피-온-라이트 패턴에 **함수 본문을 콜백으로 바꾸기 리팩터링**을 적용해보자.

1. 본문과 앞부분, 뒷부분 확인하기

```tsx
function arraySet(array, idx, value) {
  const copy = array.slice();
  copy[idx] = value;
  return copy;
}

function push(array, elem) {
  const copy = array.slice();
  copy.push(elem);
  return copy;
}

// drop_last 함수와 drop_first 함수는 생략
```

`arraySet`과 `push`함수는 복사하는 앞부분과 변경하는 본문, 그리고 복사본을 리턴하는 뒷부분이 비슷해 보인다.

2. 함수 빼내기

```tsx
function arraySet(array, idx, value) {
  return withArrayCopy(array);
}

function withArrayCopy(array) {
  const copy = array.slice();
  // 아직 정의되지 않은 idx와 value를 쓰고 있어서 동작하지 않음.
  // copy[idx] = value;
  return copy;
}
```

3. 콜백 빼내기

```tsx
function arraySet(array, idx, value) {
  return withArrayCopy(array, function (copy) {
    copy[idx] = value;
  });
}

function withArrayCopy(array, modify) {
  const copy = array.slice();
  modify(copy);
  return copy;
}
```

리팩터링으로 얻은 것은 다음과 같다.

1. 표준화된 원칙
2. 새로운 동작에 원칙을 적용할 수 있다.
3. 여러 개를 변경할 때 최적화 할 수 있다.

특히, 느리고 메모리를 많이 쓰는 카피-온-리이트를 한 번만 수행하고도 원하는 작업을 진행할 수 있다.

```tsx
// 중간 복사본을 4개나 만드는 호출부
const a1 = drop_first(array);
const a2 = push(a1, 10);
const a3 = push(a2, 11);
const a4 = arraySet(a3, 0, 42);

// 복사본을 한 번만 만들고도 위 작업을 모두 수행
const a = withArrayCopy(array, function (copy) {
  copy.shift();
  copy.push(10);
  copy.push(11);
  copy[0] = 42;
});
```

### 연습 문제 - 객체에 사용할 수 있는 버전 만들기

```tsx
// 원래 코드
function objectSet(object, key, value) {
  const copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}

function objectDelete(object, key) {
  const copy = Object.assign({}, object);
  delete copy[key];
  return copy;
}
```

```tsx
// 리팩터링하고 난 후 코드
function withObjectCopy(object, modify) {
  const copy = Object.assign({}, object);
  modify(copy);
  return copy;
}

function objectSet(object, key, value) {
  return withObjectCopy(object, function (copy) {
    copy[key] = value;
  });
}

function objectDelete(object, key) {
  return withObjectCopy(object, function (copy) {
    delete copy[key];
  });
}
```

### 연습 문제 - if 구문 리팩터링해서 when()이라는 함수로 만들어 보기

```tsx
if (array.length === 0) {
  console.log("Array is empty");
}

if (hasItem(cart, "shoes")) {
  return setPriceByName(cart, "shoes", 0);
}
```

```tsx
function when(test, then) {
  if (test) {
    return then();
  }
}

// 사용 예시
when(array.length === 0, function () {
  console.log("Array is empty");
});

when(hasItem(cart, "shoes"), function () {
  return setPriceByName(cart, "shoes", 0);
});
```

### 함수를 리턴하는 함수

```tsx
// 원래 코드
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}

// 리팩터링한 코드
function withLogging(f) {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}
```

이전에 중복된 try/catch 블록을 없앴지만, _1. 모든 코드에 `withLogging()`을 적용해야 하는 문제가 생겼다._ 그리고, _2. 어떤 부분에 로그를 남기는 것을 깜빡할 수도 있다._

코드를 감싸지 않고도 그냥 함수를 호출할 수 있는 방법을 알아보자.

1. 이름을 명확하게 바꾸기 - 로그를 남기지 않는다는 것을 이름에 표현하기

```tsx
try {
  saveUserDataNoLogging(user);
} catch (error) {
  logToSnapErrors(error);
}

try {
  fetchProductNoLogging(productId);
} catch (error) {
  logToSnapErrors(error);
}
```

2. 전체 코드는 로그를 남기므로 함수로 빼서 로그를 남기고 있는 버전이라는 이름 붙이기

```tsx
function saveUserDataWithLogging(user) {
  try {
    saveUserDataNoLogging(user);
  } catch (error) {
    logToSnapErrors(error);
  }
}

function fetchProductWithLogging(productId) {
  try {
    fetchProductNoLogging(productId);
  } catch (error) {
    logToSnapErrors(error);
  }
}
```

3. 중복 없애고, 함수 본문을 콜백으로 바꾸기

```tsx
function wrapLogging(f) {
  return function (arg) {
    // arg로 인자를 받는 부분까지 중복을 없앴다.
    try {
      f(arg);
    } catch (error) {
      logToSnapErrors(error);
    }
  };
}
```

이제 로그를 남기지 않는 버전을 로그를 남기는 버전으로 쉽게 바꿀 수 있다!

```tsx
const saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
const fetchProductWithLogging = wrapLogging(fetchProductNoLogging);
```

> [!NOTE]
> 함수를 리턴하는 함수는 함수 팩토리와 같다!

### 연습 문제 - 예외가 발생했을 때 에러를 무시하는 함수를 만드는 함수 만들기...

```tsx
// 원래 코드
try {
  codeThatMightThrow();
} catch (e) {
  // 에러를 무시하고 아무것도 하지 않음
}

// 풀이
function wrapIgnoreErrors(f) {
  return function (a1, a2, a3) {
    try {
      return f(a1, a2, a3);
    } catch (e) {
      return null;
    }
  };
}
```

### 쉬는 시간

**Q. 고차 함수를 통해 함수를 리턴하면 많은 것을 할 수 있을 것 같다. 전체 프로그램을 고차 함수로 만들면 안될까?**

- 고차 함수로 프로그램을 만들면 더 일반적으로 만들 수 있고, 복잡한 퍼즐을 풀어서 똑똑해진 것 같은 느낌을 받을 수 있다. 하지만 좋은 엔지니어링은 퍼즐을 푸는 것이 아니라, 효과적으로 문제를 해결하는 것이다!
- 고차 함수로 만든 좋은 방법을 찾았다면 항상 직관적인 방법과 비교해보자. 어떤 코드가 더 읽기 쉬운지, 얼마나 많은 중복 코드를 없앴는지, 코드가 하는 일이 무엇인지 쉽게 알 수 있는지 판단하는 과정이 필요하다.
