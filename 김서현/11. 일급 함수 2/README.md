# 11. 일급 함수 2

- [카피-온-라이트 리팩터링하기](#카피-온-라이트-리팩터링하기)
- [함수를 리턴하는 함수 - 고차 함수](#함수를-리턴하는-함수---고차-함수)

# 카피-온-라이트 리팩터링하기

1. 본문과 앞부분, 뒷부분을 확인하기

   ```tsx
   // 리팩터링할 함수
   function arraySet(array, idx, value) {
     var copy = array.slice();
     copy[idx] = value;
     return copy;
   }

   function push(array, elem) {
     var copy = array.slice();
     copy.push(elem);
     return copy;
   }

   function drop_last(array) {
     var array_copy = array.slice();
     array_copy.pop();
     return array_copy;
   }

   function drop_first(array) {
     var array_copy = array.slice();
     array_copy.shift();
     return array_copy;
   }
   ```

2. 함수 빼내기

   ```tsx
   function arraySet(array, idx, value) {
     return withArrayCopy(array);
   }

   function withArrayCopy(array) {
     var copy = array.slice(); // 앞부분
     copy[idx] = value; // 본문
     return copy; // 뒷부분
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
     var copy = array.slice();
     modify(copy); // 콜백
     return copy;
   }
   ```

- 리팩터링으로 얻은 것
  1. 표준화된 원칙
  2. 새로운 동작에 원칙을 적용할 수 있음
  3. 여러 개를 변경할 때 최적화

---

# 함수를 리턴하는 함수 - 고차 함수

- 로그를 남기려면 남겨야 하는 모든 함수를 다 감싸야 함 → 문제점

  1. 어떤 부분에 로그를 남기는 것을 깜빡할 수 있음
  2. 모든 코드에 수동으로 withLogging() 함수 적용해야 함

  ```tsx
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

  ```tsx
  function withLogging(f) {
    try {
      f();
    } catch (error) {
      logToSnapErrors(error);
    }
  }

  withLogging(function () {
    saveUserData(user);
  });

  withLogging(function () {
    fetchProduct(productID);
  });
  ```

- 함수 본문을 콜백으로 바꾸기 리팩터링 적용

  - 원래 함수를 고차 함수로 전달 → 새로운 함수 리턴

  ```tsx
  // 함수를 인자로 받음
  function wrapLogging(f) {
    // 함수를 리턴
    return function (arg) {
      try {
        f(arg);
      } catch (error) {
        logToSnapErrors(error);
      }
    };
  }

  // 로그를 남기지 않는 함수를 wrapLogging으로 감쌈
  // 로그를 남기지 않는 버전을 남기는 버전으로 쉽게 바꿈
  var saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
  var saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
  var fetchProductWithLogging = wrapLogging(fetchProductNoLogging);
  ```
