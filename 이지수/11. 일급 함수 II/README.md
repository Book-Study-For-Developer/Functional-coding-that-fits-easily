## 카피-온-라이트 리팩터링하기

**함수 본문을 콜백으로 바꾸기 단계**

1. 본문과 앞부분, 뒷부분을 확인하기
2. 함수를 빼내기
3. 콜백 빼내기

**카피-온-라이트 단계**

1. 복사본 만들기
2. 복사본 변경
3. 복사본 리턴

## 배열에 대한 카피-온-라이트 리팩터링

1. 본문과 앞부분, 뒷부분 확인하기

```tsx
function arraySet(array, idx, value) {
  const copy = array.slice(); // 앞부분
  copy[idx] = value; // 본문
  return copy; // 뒷부분
}
```

1. 함수 빼내기

```tsx
function arraySet(array, idx, value) {
  return withArrayCopy(array);
}

function withArrayCopy() {
  const copy = array.slice(); // 앞부분
  copy[idx] = value; // 본문
  return copy; // 뒷부분
}
```

1. 콜백으로 빼내기

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

리팩토링을 하면서 코드가 늘어났지만 똑같은 코드를 여기저기 만들지 않아도 되고, 카피-온-라이트 원칙에 대한 코드를 한 곳에서 관리할 수 있음

리팩토링으로 얻은 것

- 표준화된 원칙
- 새로운 동작에 원칙을 적용할 수 있음
- 여러 개를 변경할 때 최적화

## 함수를 리턴하는 함수

중복된 try/catch 구문을 없앴지만 여전히 모든 코드에 withLogging()을 사용하고 있음

```tsx
function withLogging(f) {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging(function () {
  fetchProduct(productID);
});
```

여전히 남은 두가지 문제

- 어떤 부분에 로그를 남기는 것을 깜빡할 수 있다
- 모든 코드에 수동으로 withLogging()함수를 적용해야함

1. 이름을 명확하게 바꿈

   ```tsx
   try {
     fetchProductNotLogging(productId);
   } catch (error) {
     logToSnapErrors(error);
   }

   function fetchProductWithLogging(productId) {
     try {
       fetchProductNoLogging(productId);
     } catch (error) {
       logToSnapErros(error);
     }
   }
   ```

   함수 이름으로 로그를 남기는지 안남기는지 예상할 수 있지만 새로운 중복이 생김

2. 중복 제거

   1. 익명 함수

      ```tsx
      function(arg) {
      	try {
      		fetchProductNoLogging(arg); // 본문
      	} catch (error) {
      		logToSnapErrors(error);
      	}
      }
      ```

   2. 함수 본문을 콜백으로 바꾸기

      ```tsx
      function wrapLogging(f) {
        return function (arg) {
          try {
            f(arg);
          } catch (error) {
            logToSnapErrors(error);
          }
        };
      }

      const saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
      const fetchProductWithLogging = wrapLogging(fetchProductNoLogging);
      ```

어떤 함수라도 같은 방식으로 로그를 남기는 함수로 쉽게 바꿀 수 있음
