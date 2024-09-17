# 일급 함수 1

## 이번 장에서 살펴볼 내용

- 왜 일급 값이 좋은지 알아보기
- 문법을 일급 함수로 만드는 방법에 대해 알아보기
- 고차 함수로 문법을 감싸는 방법을 알아보기
- 일급 함수와 고차 함수를 사용한 리팩터링 두 개를 살펴보기

## 앞으로 배울 코드의 냄새와 중복을 없애 추상화를 잘할 수 있는 리펙터링 두 개를 간략히 살펴보기

### 코드의 냄새: 함수 이름에 있는 암묵적 인자

함수 본문에서 사용하는 어떤 값이 함수 이름에 나타난다면 **함수 이름에 있는 암묵적 인자**는 코드의 냄개가 되며 일급 값으로 바꾸면 표현력이 더 좋아짐   

<br />

**특징**   
1. 거의 똑같이 구현된 함수가 있다
2. 함수 이름이 구현에 있는 다른 부분을 가리킨다

### 리팩터링: 암묵적 인자를 드러내기

암묵적 인자가 일급 값이 되도록 함수에 인자를 추가   

### 리팩터링: 함수 본문을 콜백으로 바꾸기

함수 본문에 어떤 부분(비슷한 함수에 있는 서로 다른 부분)을 콜백으로 바꿈, 이 방법은 원래 있던 코드를 고차 함수로 만드는 강력한 방법   

## 마케팅팀은 여전히 개발팀과 협의해야 함

**코드의 냄새와 리팩터링을 알아보기 위한 예시 상황**   
이전 장에서 추상화벽을 이용해 대부분은 개발팀과 협의 없이 일할 수 있었지만, 주어진 API로는 할 수 없는 일이 있어서 새로운 API를 개발팀에 요청해야 됨   

**요구사항**
1. 장바구니에 있는 제품 값을 설정하는 기능
2. 장바구니에 있는 제품 개수를 설정하는 기능
3. 장바구니에 있는 제품에 배송을 설정하는 기능

마케팅팀이 요청한 요구사항은 모두 비슷하며, 코드로 구현해도 비슷할 것   

## 코드의 냄새: 함수 이름에 있는 암묵적 인자

```ts
const setPriceByName = (cart: Cart, name: string, price: number) => {
  const item = cart[name];
  const newItem = objectSet(item, 'price', price);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}

// 똑같은 형태의 함수
const setQuantityByName = (cart: Cart, name: string, price: number) => { ... }
const setShippingByName = (cart: Cart, name: string, price: number) => { ... }
const setTaxByName = (cart: Cart, name: string, price: number) => { ... }

cart = setPriceByName(cart, "shoe", 13);
cart = setQuantityByName(cart, "shoe", 3);
cart = setShippingByName(cart, "shoe", 0);
cart = setTaxByName(cart, "shoe", 2.34);
```

4개의 함수가 거의 똑같이 생긴 함수로 중복의 문제가 있음   
함수 이름에 있는 암묵적 인자가 있어서 코드의 냄새가 있음   
**값을 명시적으로 전달하지 않고 함수 이름의 일부로 "전달"하고 있음**

## 리팩터링: 암묵적 인자를 드러내기

함수 이름의 일부가 암묵적 인자로 사용되고 있다면 암묵적 인자를 명시적인 인자로 바꾸는 것

<br />

**리팩터링 단계**   
1. 함수 이름에 있는 암묵적 인자를 확인
    `price`, `quantity`, `shipping`, `tax` 등
2. 명시적인 인자를 추가
    `field` 라는 인자를 추가
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 변경
    `const newItem = objectSet(item, 'price', price);` 에서 price 로 하드코딩 된 곳을 `field`로 변경
4. 함수를 부르는 곳을 고침
    `field`를 전달하도록 변경

```ts
const setFieldByName = (cart, name, field, value) => {
  const item = cart[name];
  const newItem = objectSet(item, field, value);
  const newCart = objectSet(cart, name, newItem);
  return newCart;
}

cart = setFieldByName(cart, "shoe", 'price', 13);
cart = setFieldByName(cart, "shoe", 'quantity', 3);
cart = setFieldByName(cart, "shoe", 'shipping', 0);
cart = setFieldByName(cart, "shoe", 'tax', 2.34);
```

암묵적인 이름은 인자로 넘길 수 있는 값, `일급값`으로 만듬

> [!NOTE]
> 일급인 것과 일급이 아닌 것을 구별하기   
> 일급인 것   
> 변수에 할당되는 것, 함수의 인자로 넘길수 있는 것, 함수의 리턴값으로 받을 수 있는 것, 배열이나 객체에 담을수 있는것   
> 일급이 아닌 것    
> 수식 연산자, 반복문, 조건문, try/catch 블록   

<br />

> [!NOTE]
> 문자열로 전달한 필드명이 버그가 되지 않도록 하는 방법   
> 1. 타입스크립트를 사용해서 컴파일 단계에서 버그를 잡는다   
> 2. throw문과 catch 문을 이용해서 예외처리를 한다   

### 데이터 지향
**데이터 지향이라고 하는 중요한 원칙**   
데이터를 데이터 그대로 사용하는 것의 중요한 장점은 여러 가지 방법으로 해석할 수 있다는 점이다. 제한된 API로 정의하면 데이터를 제대로 활용할 수 없다. 데이터가 미래에 어떤 방법으로 해석될지 미리 알 수 없기 때문에 필요할 때 알맞은 방법으로 해석할 수 있어야 한다.

## 리팩터링: 함수 본문을 콜백으로 바꾸기

단계   
1. 코드를 함수로 감싸기
2. 더 일반적인 이름으로 바꾸기
3. 암묵적 인자를 드러내기
4. 함수 추출하기
5. 암묵적 인자를 드러내기

### 예시 1

```ts
// 준비하고 먹기
for(let i = 0; i < foods.length; i++) {
  const food = foods[i];
  cook(food);
  eat(food);
}

// 설거지하기
for(let i = 0; i < dishes.length; i++) {
  const dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

1. 코드를 함수로 감싸기
    ```ts
    // 준비하고 먹기
    const cookAndEatFoods = () => {
      for(let i = 0; i < foods.length; i++) {
        const food = foods[i];
        cook(food);
        eat(food);
      }
    }
    cookAndEatFoods();

    // 설거지하기
    const cleanDishes = () => {
      for(let i = 0; i < dishes.length; i++) {
        const dish = dishes[i];
        wash(dish);
        dry(dish);
        putAway(dish);
      }
    }
    cleanDishes();
    ```

2. 더 일반적인 이름으로 바꾸기
    ```ts
    // 준비하고 먹기
    const cookAndEatFoods = () => {
      for(let i = 0; i < foods.length; i++) {
        const item = foods[i];
        cook(item);
        eat(item);
      }
    }
    cookAndEatFoods();

    // 설거지하기
    const cleanDishes = () => {
      for(let i = 0; i < dishes.length; i++) {
        const item = dishes[i];
        wash(item);
        dry(item);
        putAway(item);
      }
    }
    cleanDishes();
    ```

3. 암묵적 인자를 드러내기
    ```ts
    // 준비하고 먹기
    const cookAndEatArray = <T>(array: T) => {
      for(let i = 0; i < array.length; i++) {
        const item = array[i];
        cook(item);
        eat(item);
      }
    }
    cookAndEatArray(foods);

    // 설거지하기
    const cleanArray = <T>(array: T) => {
      for(let i = 0; i < array.length; i++) {
        const item = array[i];
        wash(item);
        dry(item);
        putAway(item);
      }
    }
    cleanArray(dishes);
    ```

4. 함수 추출하기
    ```ts
    // 준비하고 먹기
    const cookAndEatArray = <T>(array: T) => {
      for(let i = 0; i < array.length; i++) {
        const item = array[i];
        cookAndEat(item);
      }
    }

    const cookAndEat = (food) => {
      cook(food);
      eat(food);
    }

    cookAndEatArray(foods);

    // 설거지하기
    const cleanArray = <T>(array: T) => {
      for(let i = 0; i < array.length; i++) {
        const item = array[i];
        clean(item);
      }
    }

    const clean = (dish) => {
      wash(dish);
      dry(dish);
      putAway(dish);
    }

    cleanArray(dishes);
    ```

5. 암묵적 인자를 드러내기
    ```ts
    const forEach = <T>(array: T, f: () => void) => {
      for(let i = 0; i < array.length; i++) {
        const item = array[i];
        f(item);
      }
    }

    forEach(foods, (food) => {
      cook(food);
      eat(food);
    });
    forEach(dishes, (dish) => {
      wash(dish);
      dry(dish);
      putAway(dish);
    });
    ```

위와 같은 단계를 거처 **함수 본문을 콜백으로 바꾸기** 리팩토링을 할 수 있음

### 예시 2

```ts
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

```ts
const withLogging = <T>(f: T) => {
  try {
    f();
  } catch(error) {
    logToSnapErrors(error);
  }
} 
withLogging(() => saveUserData(user));
withLogging(() => fetchProduct(productId));
```
**함수로 감싸서 넘기는 이유**   
코드가 바로 실행되서는 안되고 콜백으로 함수의 실행을 미루는 방법을 사용하기 위해서

## 요점 정리
- 일급 값은 변수에 저장할 수 있고 인자로 전달하거나 함수의 리턴값으로 사용할 수 있음
- 언어에는 일급이 아닌 기능이 많이 있음, 함수로 감싸 일급으로 만듬
- 일급 함수는 어떤 단계 이상의 함수형 프로그래밍을 하는데 필요
- 고차 함수는 다른 함수에 인자로 넘기거나 리턴값으로 받을 수 있는 함수
- 함수 이름에 있는 암묵적 인자는 코드의 냄새, 암묵적 인자 드러내기로 개선 가능
- 동작을 추상화 하기 위해서 본문을 콜백으로 바꾸기 리팩토링 사용