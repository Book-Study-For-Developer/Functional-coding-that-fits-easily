## 배열에 대한 카피-온-라이트 리팩토링

1. 본문과 앞부분, 뒷부분을 확인하기

```tsx
function arraySet(array: Array, idx: number, value: string) {
	const copy = array.slice(); // 앞부분
	copy[idx] = value; // 본문
	return copy; // 뒷부분
}
```

1. 함수 빼내기

```tsx
function withArrayCopy(array: Array) {
	const copy = array.slice(); // 앞부분
	copy[idx] = value; // 본문
	return copy; // 뒷부분
}
```

1. 콜백 빼내기

```tsx
function withArrayCopy<T>(array: Array<T>, modify: (copy: T) => void) {
	const copy = array.slice(); // 앞부분
	modify(copy); // 콜백으로 빼내기
	return copy; // 뒷부분
}
```

최종적으로 리팩토링 된 코드

```tsx
function arraySet(array: Array, idx: number, value: string) {
	return withArrayCopy(array, function(copy) {
		copy[idx] = value;
	})
}
```

원래 코드는 두줄밖에 되지 않았지만 리팩토링을 진행하면서 코드가 더 증가했다.
그러나 리팩토링을 했기 때문에 한곳에서 관리 할 수 있게 되었다.

- 표준화된 원칙
- 새로운 동작에 원칙을 적용할 수 있음
- 여러 개를 변경할 때 최적화

위의 3가지를 얻을 수 있게 되었다.

### 이외에도 적용해볼 수 있는 연습 문제들

```tsx
// try/catch 문
function tryCatch(f: VoidFunction, errorHandler: (error: Error) => void) {
	try {
		return f();
	} catch (error) {
		return erorHandler(error);
	}
}

// when
function when(test: boolean, then: VoidFunction) {
	if (test) {
		return then();
	}
}

// IF
function IF(test: boolean, then: VoidFunction, ELSE: VoidFunction) {
	if (test) {
		return then();
	} else {
		return ELSE();
	}
}

```

함수로 만들어서 좋은건 OK.. 그런데 예를 들어 else if가 더 필요하다던가 또는 switch case문으로 처리해야한다면..? 이때도 또 다른 함수를 만들어서 처리해야하는 건가?

## 함수를 리턴하는 함수

로그를 남기기 위해서 모든 함수에 비슷한 try/catch문을 감싸서 실행을 해야했다.

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

// 둘은 거의 동일한 코드로 반복된다.
// 이를 해결하기 위해 반복되는 코드를 캡슐화 한다.
function withLogging(f: VoidFunction) {
	try {
		f();
	} catch (error) {
		logToSnapErrors(error);
	}
}
```

`withlogging` 을 통해 함수를 캡슐화 하더라도 여전히 두가지 문제가 존재

- 어떤 부분에 로그를 남기는 것을 깜빡할 수 있다.
    - 수동으로 함수를 적용해야 하다보니 휴먼 에러가 발생할 수 있다. 🤦‍♀️
- 모든 코드에 수동으로 withLogging() 함수를 적용해야 한다.
    - 모든 코드에 직접 적용해야 한다. 🥹

1. 이름을 명확하게 바꾼다.

```tsx
try {
	saveUserDataNoLogging(user);
} catch (error) {
	logToSnapErrors(error);
}
```

1. 로그를 남기는 함수

```tsx
function saveUserDataWithLogging(user: User) {
	try {
		saveUserDataNoLogging(user);
	} catch (error) {
		logToSnapErrors(error);
	}
}
function fetchProductWithLogging(user: User) {
	try {
		fetchProductNoLogging(user);
	} catch (error) {
		logToSnapErrors(error);
	}
}
```

로그를 남기는 함수를 명시적으로 표기하여 사용할 때 로그가 남을 것을 **이름으로 예상**할 수 있다.
그러나 여전히 두 함수는 동일하게 생겼고 중복이 존재한다.

중복을 없애보자 🔫

1. 먼저 이름을 없애고 익명함수로 만든다.

```tsx
function(arg) {
	try {
		fetchProductNoLogging(arg);
	} catch (error) {
		logToSnapErrors(error);
	}
}
```

1. 앞부분, 본문, 뒷부분을 명확히 알아냈으니 함수 본문을 콜백으로 바꾸자

```tsx
function wrapLogging(f) { // 함수를 인자로 받는다.
 return function (arg) { // 함수로 감싼 코드는 나중에 실행된다.
		try {
			f(arg);
		} catch (error) {
			logToSnapErrors(error);
		}
	}
}

// 더 자주 쓰는 화살표 함수로 바꾸면? 
const wrapLogging = (f) => {
  return (arg) => {
	  try {
			f(arg);
		} catch (error) {
			logToSnapErrors(error);
		}
  }
}

const saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
```

어떤 함수라도 같은 방식으로 로그를 남기는 함수로 쉽게 바꿀 수 있게 되었다.

**여기서 드는 의문 🤔** 

리팩토링을 하는 이유는 아래 두가지 이유였다.

- **어떤 부분에 로그를 남기는 것을 깜빡할 수 있다.**
- **모든 코드에 수동으로 withLogging() 함수를 적용해야 한다.**

리팩토링을 진행하면서 과연 이게 해소가 된건가? 의문을 가져서 한번 다시 Recap…해보기

어떤 부분에 로그를 남기는 것을 깜빡할 수 있다. ⇒  이름을 명시적으로 바꾸면서 해결을 한것으로 이해함.
모든 코드에 수동으로 withLogging() 함수를 적용해야 한다. ⇒  수동으로 하는 것은 변함이 없는 듯 하고 withLogging을 사용하는 대신 함수에 wrapLogging을 감싸서 해결함.

다른 분들의 의견도 궁금합니다! ⁇

### 고차 함수를 통해 많은 것을 얻을 수 있는데 전체를 고차 함수로 만들면 안되는 것인가?

고차 함수는 강력한 기능이다. 그러나 **비용이 따른다.** 능숙하게 쓸 줄 알아야 하지만 그 것을 더 좋은 코드를 만드는데 사용해야 한다.

즉, 고차 함수를 만드는 것이 정말 필요한 함수인지 의문을 가지고 사용하는 것이 좋다.