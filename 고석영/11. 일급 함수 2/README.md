## 카피-온-라이트 리팩터링

### 함수 본문을 콜백으로 바꾸는 단계

1. 본문과 앞부분, 뒷부분을 확인하기
2. 함수 빼내기
3. 콜백 빼내기

### 카피-온-라이트 단계

1. 복사본을 만듭니다.
2. 복사본을 변경합니다.
3. 복사본을 리턴합니다.

> 카피-온-라이트의 규칙은 복사본을 만들고 복사본을 변경한 다음 복사본을 리턴하는 것이다.
> 여기서 달라지는 부분은 변경하는 방법이고, 복사본을 만들고 리턴하는 동작은 항상 같다.

## 배열에 대한 카피-온-라이트 리팩터링

### 본문과 앞부분, 뒷부분을 확인하기

```tsx
const arraySet = (array, idx, value) => {
  const copy = [...array] // 본문 앞부분
  copy[idx] = value // 본문
  return copy // 본문 뒷부분
}
```

### 함수 빼내기

```tsx
const withArrayCopy = array => {
  const copy = [...array]
  copy[idx] = value
  return copy
}

const arraySet = (array, idx, value) => {
  return withArrayCopy(array)
}
```

### 콜백 빼내기

```tsx
const withArrayCopy = <T,>(array: T[], modify: (copy: T[]) => void) => {
  const copy = [...array]
  modify(copy)
  return copy
}

const arraySet = <T,>(array: T[], idx: number, value: T): T[] => {
  return withArrayCopy(array, copy => {
    copy[idx] = value
  })
}
```

### **리팩터링으로 얻은 것**

> 코드는 늘었지만, 카피-온-라이트 원칙에 대한 코드를 한 곳에서 관리할 수 있습니다 .

1. 표준화된 원칙
2. 새로운 동작에 원칙을 적용할 수 있음
3. 여러 개를 변경할 때 최적화

## 함수를 리턴하는 함수

```tsx
try {
  saveUserData(user)
} catch (error) {
  logToSnapErrors(error)
}
```

### `withLogging()` 함수로 반복되는 코드 캡슐화

```tsx
function withLogging(f) {
  try {
    f()
  } catch (error) {
    logToSnapErrors(error)
  }
}

withLogging(function () {
  saveUserData(user)
})
```

#### 아직 남아있는 문제점

- 어떤 부분에 로그를 남기는 것을 깜빡할 수 있음
- 모든 코드에 수동으로 함수를 적용해야 하는 것은 동일함..

> 만약에 코드를 감싸지 않고 그냥 함수를 호출할 수 있다면..?

### 코드를 감싸지 않고 함수를 호출하기

#### 이름을 명확하게 바꿈

```tsx
try {
  saveUserDataNoLogging(user)
} catch (error) {
  logToSnapErrors(error)
}
```

#### 로그를 남기는 함수

```tsx
function saveUserDataWithLogging(user) {
  try {
    saveUserDataNoLogging(user)
  } catch (error) {
    logToSnapErrors(error)
  }
}
```

- 로그가 남는다는 걸 함수 이름으로 명시해 로그 남기기를 깜빡하지 않을 수 있음

#### 로그를 남기는 함수를 일반적인 이름으로 변경하기

```tsx
function (arg) {
	try {
		saveUserDataNoLogging(arg)
	} catch (error) {
		logToSnapErrors(error)
	}
}
```

#### 함수 본문을 콜백으로 바꾸기

```tsx
function wrapLogging(f) {
  return function (args) {
    try {
      f(arg)
    } catch (error) {
      logToSnapErrors(error)
    }
  }
}

const saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging)
```

- 중복되는 함수 본문을 콜백으로 바꿔 어떤 함수라도 같은 방식으로 로그를 남기는 함수로 쉽게 변경할 수 있음

### `saveUserDataWithLogging()` 함수 시각화

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0283-02.jpg)

### (+) 연습문제 (feat. 클로저, 커링)

```tsx
const makeAdder = n => x => n + x

const increment = makeAdder(1)
const plus10 = makeAdder(10)
const sum = makeAdder(1)(2)

increment(10) // 11
plus10(12) //12
console.log(sum) // 3
```
