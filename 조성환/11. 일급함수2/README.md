# 일급함수 2

## 이번 장에서 살펴볼 내용

- 함수 본문을 콜백으로 바꾸기 리팩터링에 대해 더 알아보기
- 함수를 리턴하는 함수가 가진 강력한 힘을 이해하기
- 고차 함수에 익숙해지기 위해 여러 고차 함수를 만들어 보기

## 코드 냄새 하나와 리팩터링 두 개

### 코드의 냄새: 함수 이름에 있는 암묵적 인자

함수 본문에서 사용하는 어떤 값이 함수 이름에 나타난다면 함수 이름에 있는 암묵적 인자는 코드의 냄새

1. 거의 똑같이 구현된 함수가 있음
2. 함수 이름이 구현에 있는 다른 부분을 가리킴

### 리팩터링: 암묵적 인자를 드러내기

암묵적 인자가 일급 값이 되도록 함수에 인자를 추가   
단계
1. 함수 이름에 있는 암묵족 인자를 확인
2. 명시적인 인자를 추가
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 변경
4. 함수를 호출하는 곳을 변경

### 리팩터링: 함수 본문을 콜백으로 바꾸기

함수 본문에 비슷한 함수에 있는 서로 다른 부분을 콜백으로 바꿈   
원래 있던 코드를 고차 함수로 만드는 강력한 방법   
단계
1. 본문에서 바꿀 부분의 앞부분과 뒷부분을 확인한다.
2. 리팩터링 할 코드를 함수로 빼낸다.
3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼낸다.

## 배열에 대한 카피-온-라이트 리팩터링

카피-온-라이트 리팩터링 단계   
1. 복사본을 만듬
2. 복사본을 변경
3. 복사본을 리턴

함수 본문을 콜백으로 바꾸기 단계
1. 본문과 앞부분, 뒷부분을 확인하기
2. 함수 빼내기
3. 콜백 빼내기

예제   
```ts
const arraySet = <T>(array: T[], idx: number, value: T) => {
  const copy = array.slice();
  copy[idx] = value;
  return copy;
}

const push = <T>(array: T[], elem: T) => {
  const copy = array.slice();
  copy.push(elem);
  return copy;
}

const dropLast = <T>(array: T[]) => {
  const copy = array.slice();
  copy.pop();
  return copy;
}

const dropFirst = <T>(array: T[]) => {
  const copy = array.slice();
  copy.shift();
  return copy;
}
```

1. 본문과 앞부분, 뒷부분을 확인   
    앞부분: `const copy = array.slice();`
    뒷부분: `return copy;`
    본문: 여러 함수들의 달라지는 부분이 있다면 그건 본문이 되어야 함

2. 함수 빼내기
    ```ts
    const withArrayCopy = <T>(array:T) => {
      const copy = array.slice();
      copy.shift(); // 본문
      return copy;
    }
    ```
3. 콜백 빼내기
    ```ts
    const withArrayCopy = <T>(array:T, modify: (copy: T) => void) => {
      const copy = array.slice();
      modify(copy)
      return copy;
    }
    ```

**사용하기**
```ts
const arraySet = <T>(array: T[], idx: number, value: T) => {
  return withArrayCopy(array, (copy) => {
    copy[idx] = value;
  })
}

const push = <T>(array: T[], elem: T) => {
  return withArrayCopy(array, (copy) => {
    copy.push(elem);
  })
}

const dropLast = <T>(array: T[]) => {
  return withArrayCopy(array, (copy) => {
    copy.pop();
  })
}

const dropFirst = <T>(array: T[]) => {
  return withArrayCopy(array, (copy) => {
    copy.shift();
  })
}

const withArrayCopy = <T>(array:T, modify: (copy: T) => void) => {
  const copy = array.slice();
  modify(copy)
  return copy;
}
```

이렇게 함수로 추출하면 좋은점
1. 빠른 정렬 라이브러리를 찾았다면 쉽게 적용이 가능
    ```ts
    const sortedArray = withArrayCopy(array, (copy) => {
      SuperSorter.sort(copy);
    })
    ```
2. 함수는 동작을 최적화 하기 더 쉬움
    ```ts
    // 중간 복사본을 만듬 - 배열을 4번 복사
    const a1 = dropFirst(array);
    const a2 = push(a1, 10);
    const a3 = push(a2, 11);
    const a4 = arraySet(a3, 0, 42);

    // 복사본을 하나만 만듬 - 배열을 한번만 복사하고 복사본을 4번 바꿈
    const a4 = withArrayCopy(array, (copy) => {
      copy.shift();
      copy.push(10);
      copy.push(11);
      copy[0] = 42;
    })
    ```

### 객체에 대한 카피-온-라이드 리팩터링

예시
```ts
const objectSet = <T>(object: {[key in string]: T}, key: string, value:T) => {
  const copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}

const objectDelete = <T>(object:T, key: string) => {
  const copy = Object.assign({}, object);
  delete copy[key];
  return copy
}
```

리팩터링 적용 후
```ts
const objectSet = <T>(object: {[key in string]: T}, key: string, value:T) => {
  return withObjectCopy(object, (copy) => {
    copy[key] = value;
  });
}

const objectDelete = <T>(object:T, key: string) => {
  return withObjectCopy(object, (copy) => {
    delete copy[key];
  });
}

const withObjectCopy = <T>(object: T, modify: (copy: T) => void) => {
  const copy = Object.assign({}, object);
  modify(copy);
  return copy;
}
```
## 함수를 리턴하는 함수

이전에 추가된 에러 로그를 찍었던 함수들
```ts
// 처음 코드
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}
try {
  fetchProduct(productId);
} catch (error) {
  logToSnapErrors(error);
}

// 함수로 개선 했던 코드
const withLogging = (f: () => void) => {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}
withLogging(() => {
  saveUserData(user);
})
withLogging(() => {
  fetchProduct(productId);
})
```

**아직 남아 있는 문제**   
- 어떤 부분에는 로그를 남기는 것을 깜빡할 수 있음
- 모든 코드에 수동으로 withLogging() 함수를 적용해야 됨

### 함수를 리턴하는 함수로 개선하기

```ts
// 처음 코드
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}
try {
  fetchProduct(productId);
} catch (error) {
  logToSnapErrors(error);
}
```

함수를 구분하기 위해 이름을 명확하게 변경(로그를 남기지 않는 함수라는걸 명시)

```ts
// 로그를 찍지 않는 함수라고 명시
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

// 함수로 감싸주기, 로그를 찍는 함수라는걸 명시
const saveUserDataWithLogging = (user: User) => {
  try {
    saveUserDataNoLogging(user);
  } catch (error) {
    logToSnapErrors(error);
  }
}
const fetchProductWithLogging = (productId: string) => {
  try {
    fetchProductNoLogging(productId);
  } catch (error) {
    logToSnapErrors(error);
  }
}
```

**함수 본문을 콜백으로 바꾸기 리팩터링 적용**
```ts
// 익명함수로 반복되는 본문을 바꿈
<T>(arg: T) => {
  try {
    f(arg); // 아직 지정안한 함수
  } catch (error) {
    logToSnapErrors(error);
  }
}

// 고차 함수로 만든 함수를 리턴
// TODO: 타입 지정하기
const wrapLogging = (f) => {
  return (arg) => {
    try {
      f(arg); // 아직 지정안한 함수
    } catch (error) {
      logToSnapErrors(error);
    }
  }
}

const saveUserDataWithLogging = wrapLogging(saveUserDateNoLogging);
const fetchProductWithLogging = wrapLogging(fetchProductNoLogging);
saveUserDataWithLogging(user)
fetchProductWithLogging(productId)
```

함수를 리턴하는 함수를 사용해서   
반환하는 함수를 할당 후 사용   
할당된 함수를 사용하기 때문에 반복되는 코드가 많이 개선됨   
함수를 리턴하는 함수는 **함수 팩토리**와 같음

## 요점 정리

- 고차 함수로 패턴이나 원칙을 코드로 만들 수 있음
- 고차 함수로 함수를 리턴하는 함수를 만들 수 있음
- 고차 함수는 많은 중복 코드를 없애 주지만 가독성을 해칠 수 있음
