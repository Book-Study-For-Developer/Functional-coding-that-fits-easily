## 일급 함수 I

> 문제 상황 : 마케팅팀은 여전히 필요한 API가 많다. 새로운 API를 개발팀에게 요청해야 한다.

### 코드 냄새: 함수 이름에 있는 암묵적 인자

```tsx
// 이름으로 아이템 가격 정하기
function setPriceByName(cart, name, price) {
  const item = cart[name];
  const newItem = objectSet(item, "price", price);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}

// 이름으로 아이템 배송비 정하기
function setShipppingByName(cart, name, ship) {
  const item = cart[name];
  const newItem = objectSet(item, "shippping", ship);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}

// setQuantityByName, setTaxByName은 생략

function objectSet(object, key, value) {
  const copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}
```

위 두 개의 함수만 봐도 objectSet에 전달하는 'price', 'shipping' 등 문자열만 다르다.

분명하게 보이는 두 개의 문제는,

1. 여기 있는 함수는 거의 똑같이 생겼다. 즉, **중복**이 일어난다.
2. 필드를 결정하는 문자열이 함수 이름에 있다. **함수 이름에 있는 암묵적 인자**라고 부른다.

> [!NOTE]
> 함수 구현이 거의 똑같은데, 함수 이름만으로 구현의 차이가 있다면 코드의 냄새가 난다고 볼 수 있다!

### 리팩터링: 암묵적 인자를 드러내기

**기본적인 아이디어는 암묵적 인자를 명시적인 인자로 바꾸는 것이다.** 즉, 암묵적인 것을 명시적으로 바꾼다!

그 세부 단게는,

1. 함수 이름에 있는 암묵적 인자를 확인한다.
2. 명시적인 인자를 추가한다.
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾼다.
4. 함수를 부르는 곳을 고친다.

```tsx
// 리팩토링 후: Price나 Shippping등 암묵적인 표현을 Field로 변경한다.
function setFieldByName(cart, name, field, value) {
  const item = cart[name];
  const newItem = objectSet(item, field, value);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}

// 호출부
cart = setFieldByName(cart, "shoe", "price", 13);
cart = setFieldByName(cart, "shoe", "quantity", 3);
cart = setFieldByName(cart, "shoe", "shipping", 0);
cart = setFieldByName(cart, "shoe", "tax", 2.34);
```

위 처럼 리팩터링을 하면서 얻을 수 있는 효과는,

1. 비슷한 함수를 모두 일반적인 함수 하나로 바꿨기 때문에, 이전에 많던 함수는 필요가 없어졌다.
2. 필드명을 **일급 값**으로 만들었다. 암묵적인 이름(price, shippping 등)은 인자로 넘길 수 있는 값이 되었다.

> [!NOTE]
> 일급 값(first-class value)은 언어에 있는 다른 값처럼 쓸 수 있다.

### 일급인 것과 아닌 것을 구별하기

자바스크립트에서 일급이라 할 수 있는 것은 다음과 같다.

1. 숫자: 함수에 인자로 넘길 수 있고 함수의 리턴값으로 받을 수 있다.
2. 문자열
3. 불리언값
4. 배열
5. 객체
6. **함수**

일급이 아닌 것은 다음과 같다.

1. 수식 연산자(+, -, \*, /): 변수에 담을 방법이 없다.
2. if 키워드(조건문)
3. for 키워드(반복문)
4. try/catch 블록

이전의 리팩토링 사례에서처럼, 함수명 일부(price, shippping)을 값처럼 쓸 수 있는 방법은 없기 때문에 함수명의 일부를 인자로 바꿔 일급으로 만들었다.

### 필드명을 문자열로 사용하면 버그가 생기지 않을까?

오타가 발생하여 충분히 걱정할 만한 일이다.

그래서 이를 해결할 수 있는 두 가지 방안은,

1. 컴파일 타임에 검사하기
   - 타입스크립트와 같이 정적 타입 시스템 차용하기
2. 런타임에 검사하기
   - 함수를 실행할 때 동작하는 지 확인하기

```tsx
// 런타임 검사 방법으로 필드명이 올바른지 확인하는 코드
const validItemFields = ["price", "quantity", "shippping", "tax"];

function setFieldByName(cart, name, field, value) {
  if (!validItemFields.includes(field))
    throw "Not a valid item field: " + "'" + field + "'.";
  const item = cart[name];
  const newItem = objectSet(item, field, value);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

### 일급 필드를 사용하면 API를 바꾸기 더 어렵나요?

문제가 될 여지가 있는 안건이 몇 가지 있다.

1. 엔티티 필드명을 일급으로 만들어 사용하는 것이 세부 구현을 밖으로 노출하는 것이 아닌지 걱정할 수도 있다.
2. 추상화 벽 아래에서 정의한 필드명이 벽 위에 있는 사람들에게 전달되는 것은 추상화 벽의 원칙을 위반할 수도 있다.
3. API 문서에 필드명을 명시하면 영원히 필드명을 바꿀 수 없다.

그에 대한 답변으로는,

1. 필드명을 계속 유지해야 한다. 그렇다고 해서 구현이 외부에 노출된 것은 아니다.
2. 내부에서 정의한 필드명이 바뀐다고 해도 사용하는 사람들이 원래 필드명을 그대로 사용할 수 있게 바꿔주면 된다.

```tsx
const validItemFields = ["price", "quantity", "shippping", "tax", "number"];
const translations = { quantity: "number" };

function setFieldByName(cart, name, field, value) {
  if (!validItemFields.includes(field))
    throw "Not a valid item field: " + "'" + field + "'.";
  // 필드명이 일급이기 때문에 가능하다.
  if (translations.hasOwnProperty(field)) field = translations[field];
  const item = cart[name];
  const newItem = objectSet(item, field, value);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

### 지루한 연습문제

**해답 1**

```tsx
function multiplyByNum(x, num) {
  return x * num;
}
```

**해답 2**

```tsx
function incrementFieldByName(cart, field, name) {
  const item = cart[name];
  const value = item[field];
  const newValue = value + 1;
  const newItem = objectSet(item, field, newQuantity);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

### 객체와 배열을 너무 많이 쓰게 된다.

자바스크립트의 특성 상, 일급 필드를 사용하면 객체를 많이 쓴다고 느낄 수 있다.

**중요한 것은 데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용하고 있다는 점이다.** 장바구니와 제품 엔티티는 매우 일반적이기 때문에, 정해진 인터페이스로 접근하게끔 하는 것이 아니라 여러 가지 방법으로 해석할 수 있게 제한된 API를 오히려 사용하지 않는다.

이는 **🚨데이터 지향🚨**이라고 하는 중요한 원칙이다!

---

### 반복문 예제: 먹고 치우기

고차함수(다른 함수를 인자로 받는 함수)를 만들어서 반복문 문법을 보지 않게 하자!

```tsx
// 준비하고 먹기
for (let i = 0; i < foods.length; i++) {
  const food = foods[i];
  cook(food);
  eat(food);
}

// 설거지하기
for (let i = 0; i < dishes.length; i++) {
  const dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

먼저, 설명할 수 있는 이름을 붙이자.

```tsx
function cookAndEatFoods() {
  for (let i = 0; i < foods.length; i++) {
    const food = foods[i];
    cook(food);
    eat(food);
  }
}

function cleanDishes() {
  for (let i = 0; i < dishes.length; i++) {
    const dish = dishes[i];
    wash(dish);
    dry(dish);
    putAway(dish);
  }
}
```

매우 구체적인 지역변수의 이름을 일반적인 이름으로 바꾸고, 함수 이름에 있는 암묵적 인자 냄새를 없애자.

```tsx
function cookAndEatArray(array) {
  for (let i = 0; i < array.length; i++) {
    const item = array[i];
    cook(item);
    eat(item);
  }
}

function cleanArray(array) {
  for (let i = 0; i < array.length; i++) {
    const item = array[i];
    wash(item);
    dry(item);
    putAway(item);
  }
}

// 호출부
cookAndEatArray(foods);
cleanArray(dishes);
```

반복문 안에 있는 본문을 분리하자.

```tsx
function cookAndEatArray(array) {
  for (let i = 0; i < array.length; i++) {
    const item = array[i];
    cookAndEat(item);
  }
}

function cookAndEat(food) {
  cook(food);
  eat(food);
}

function cleanArray(array) {
  for (let i = 0; i < array.length; i++) {
    const item = array[i];
    clean(item);
  }
}

function clean(dish) {
  wash(item);
  dry(item);
  putAway(item);
}

// 호출부
cookAndEatArray(foods);
cleanArray(dishes);
```

다시, 함수 이름에 있는 암묵적 인자 냄새를 없애자.

```tsx
// 이제 forEach는 함수 f를 인자로 받는 고차 함수다!
function forEach(array, f) {
  for (let i = 0; i < array.length; i++) {
    const item = array[i];
    f(item);
  }
}

function cookAndEat(food) {
  cook(food);
  eat(food);
}

function clean(dish) {
  wash(item);
  dry(item);
  putAway(item);
}

// 호출부
forEach(foods, cookAndEat);
forEach(dishes, clean);
```

이렇게 바꾸면서 얻을 수 있는 효과는, **코드를 추상화해서 반복문을 여러번 쓰지 않아도 된다는 점이다.**

### 리팩터링: 함수 본문을 콜백으로 바꾸기

try/catch 블럭이 여기저기 널려있는 문제를 발견했다.

```tsx
try {
  // 앞부분
  saveUserData(user); // 본문
} catch (error) {
  // 뒷부분: 어디에서나 똑같이 사용
  logToSnapErrors(error);
}

try {
  // 앞부분
  fetchProduct(productId); // 본문
} catch (error) {
  // 뒷부분: 어디에서나 똑같이 사용
  logToSnapErrors(error);
}
```

함수 본문을 콜백으로 바꾸기라는 리팩토링으로 중복을 없애보자.

```tsx
function withLogging(f) {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}

// 인라인으로 함수를 정의한 이유는,
// 함수가 바로 실행되면 안 되기 때문이다!
// 함수의 실행을 미루는 일반적인 방법이다.
withLogging(function () {
  saveUserData(user);
});
```
