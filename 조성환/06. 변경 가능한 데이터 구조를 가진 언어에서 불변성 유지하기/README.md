# 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기

## 이번장에서 살펴볼 내용

- 데이터가 바뀌지 않도록 하기 위해 카피-온-라이트를 적용합니다.
- 배열과 객체를 데이터에 쓸 수 있는 카피-온-라이트 동작을 만듭니다.
- 깊이 중첩된 데이터도 카피-온-라이트가 잘 동작하게 만듭니다.

## 동작을 읽기, 쓰기 또는 둘 다로 분류하기

- `읽기` 동작은 데이터를 바꾸지 않고 정보를 꺼내는 것
- `쓰기` 동작은 데이터를 바꾸는 것으로 불변성의 원칙에 따라 구현해야 함
- `읽기쓰기` 를 동시에 하는 경우도 있음

**자바스크립트는 기본적으로 변경 가능한 데이터 구조를 사용**하기 때문에 불변성 원칙을 적용하려면 직접구현   
```js
const 원본배열 = [1,2,3,4,5];
const changeValue = (받아온배열) => {
  받아온배열[2] = 6;  
}
changeValue(원본배열);
console.log(원본배열); // [1,2,6,4,5]
```

## 카피-온-라이트 원칙 세 단계

1. 복사본 만들기
2. 복사본 변경하기(원하는 만큼)
3. 복사본 리턴하기

카피-온-라이트 예시(지난장)
```ts
const addElementLast = <T>(array: T[], elem:T) => {
  const newArray = array.slice(); // 1.복사본 만들기
  newArray.push(elem); // 2.복사본 변경하기
  return newArray; // 3.복사본 리턴하기
}
```
addElementLast 함수는 받아오는 기존의 데이터를 바꾸지 않고, 정보를 리턴했기 때문에 `읽기`

> [!NOTE]
> 카피-온-라이트는 쓰기를 읽기로 바꿉니다. 👍🏻

## 카피-온-라이트로 쓰기를 읽기로 바꾸기

```ts
const removeItemByName = (cart: Cart, name: string) => {
  const idx: null | number = null;
  cart.forEach((item) => {
    if(item.name === name){
      idx = i;
    }
  })
  if(idx !== null){
    cart.splice(idx, 1); // cart 배열의 idx 위치의 요소를 하나 제거 (cart를 변경 -> cart 쓰기)
  }
}
```
removeItemByName함수는 **인자 cart로 넘기는 배열을 변경**   
장바구니를 **변경 불가능한 데이터로 쓰기 위해 카피-온-라이트를 적용**   

```ts
const removeItemByName = (cart: Cart, name: string) => {
  const newCart = cart.slice(); // 1.복사본 만들기
  const idx: null | number = null;
  newCart.forEach((item) => {
    if(item.name === name){
      idx = i;
    }
  })
  if(idx !== null){
    newCart.splice(idx, 1); // 2.복사본 변경하기
  }
  return newCart; // 3.복사본 리턴하기
}
```
이제 removeItemByName 함수는 읽기 함수가 되었음 (내부에 생성한 복사본을 가지고 조작하고, 외부 데이터를 무엇도 변경하지 않음)

```ts
// 원래는 이렇게 shoppingCart 데이터를 바꿨다면
removeItemByName(shoppingCart, name);

// 쓰기를 읽기로 바꾼 다음에는
shoppingCart = removeItemByName(shoppingCart, name); // 함수 자체는 읽기만 하고 할당하면서 쓰기
```
   
## 쓰기를 하면서 읽기도 하는 동작
**example**   
shift는 첫번째 요소를 제거하고 반환함
```ts
const a = [1,2,3,4];
const b = a.shift();
console.log(b) // 1 (읽기의 결과물)
console.log(a) // [2,3,4] (쓰기의 결과물)
```

두가지 접근 방법
1. 읽기와 쓰기 함수로 각각 분리한다.
2. 함수에서 값을 두 개 리턴한다.

### 읽기와 쓰기 동작으로 분리하기
읽기
```ts
const firstElement = (array: number[]) => {
  return array[0];
}
```
쓰기
```ts
// 일단 그대로 shift 사용
const dropFirst = (array: number[]) => {
  array.shift();
}

// 그리고 카피-온-라이트 적용
const dropFirst = (array: number[]) => {
  const arrayCopy = array.slice(); // 1.복사본을 만든다
  arrayCopy.shift(); // 2.복사본을 변경한다
  return arrayCopy; // 3.복사본을 리턴한다
}
```

### 값을 두개 리턴하는 함수로 만들기

```ts
const shift = (array: number[]) => {
  const arrayCopy = array.slice();
  const first = arrayCopy.shift();
  return {
    first, // 읽기한거 반환
    array: arrayCopy // 복사한 배열을 변경한거라 쓰기가 아니라 읽기
  }
}
```

> [!NOTE]
> 카피-온-라이트로 쓰기를 읽기로 바꾸는 방법   
> 두가지 접근 방법   
> 1. 읽기와 쓰기 함수로 각각 분리한다.   
> 2. 함수에서 값을 두 개 리턴한다.   

## 불변 구조를 읽는 것은 계산

- **변경 가능한 데이터를 읽는 것은 액션**
- 쓰기는 **데이터를 변경 가능한 구조로 만듬**
- 어떤 데이터에 **쓰기가 없다면 데이터는 변경 불가능한 데이터**
- **불변 데이터 구조를 읽는 것은 계산**
- 쓰기를 **읽기로 바꾸면 코드에 계산이 많아짐**

