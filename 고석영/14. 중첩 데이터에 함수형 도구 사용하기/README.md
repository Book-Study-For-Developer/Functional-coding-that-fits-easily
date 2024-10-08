## 객체를 다루기 위한 고차 함수

```tsx
function incrementField(item, field) {
  const value = item[field]
  const newValue = value + 1
  const newItem = objectSet(item, field, newValue)
  return newItem
}

function decrementField(item, field) {
  const value = item[field]
  const newValue = value - 1
  const newItem = objectSet(item, field, newValue)
  return newItem
}

function doubleField(item, field) {
  const value = item[field]
  const newValue = value * 2
  const newItem = objectSet(item, field, newValue)
  return newItem
}

function halveField(item, field) {
  const value = item[field]
  const newValue = value / 2
  const newItem = objectSet(item, field, newValue)
  return newItem
}
```

### 문제점

- increment, decrement, double, halve처럼 동작 이름이 함수 이름에 있음
- 비슷한 동작이 조금씩만 다르고 중복되고 있음

## `update()` 도출하기

- 동작 이름이 함수 이름에 있음 → **암묵적 인자를 드러내기**로 리팩터링
- 함수의 중복 → **함수 본문을 콜백으로 바꾸기**로 리팩터링

> [!NOTE]
>
> 고차 함수 `update()`에는 **바꾸려는 객체, 키, 동작**을 함수에 넘기면 됨

```tsx
function update(object, key, modify) {
  const value = object[key]
  const newValue = modify(value)
  const newObject = objectSet(object, key, newValue)
  return newObject
}
```

### 값을 바꾸기 위해 `update()` 사용하기

```tsx
const employee = {
  name: 'Kim',
  salary: 120000,
}

function raise10Percent(salary) {
  return salary * 1.1
}

update(employee, 'salary', raise10Percent)
// { name: 'Kim', salary: 132000 }
```

- `update()`: 특정 값을 다루는 동작을 받아 특정키가 있는 해시 맵에 적용하는 함수
  - `raise10Percent()`를 직원 객체(해시 맵)에 사용
- 이것은 중첩된 문맥 안에 있는 값에 함수를 적용하는 것으로 볼 수 있음

> ❓ `update()`가 원본 해시 맵을 바꾸나요?
>
> - 카피-온-라이트 원칙을 사용해 원본 해시 맵의 복사본을 변경해 리턴하기 때문에 원본 해시 맵을 바꾸지 않음

> ❓ 원본을 바꾸지 않는다면 어떻게 사용하나요?
>
> - 리턴한 값을 원래 변수에 다시 할당해서 사용

### 리팩터링: 조회하고 변경하고 설정하는 것을 `update()`로 교체하기

- `update()`로 암묵적 인자를 드러내기 + 함수 본문을 콜백으로 바꾸기를 한 번에 진행할 수 있음
- 중첩된 객체에 사용하기 좋음
- 교체 방법
  1. 조회하고 바꾸고 설정하는 것을 찾기
  2. 바꾸는 동작을 콜백으로 전달해서 `update()`로 교체하기

## 함수형 도구: `update()`

- 객체를 다루는 함수형 도구
- 하나의 키에 하나의 값을 변경
- `update()`에 필요한 인자
  - 객체
  - 변경할 값이 어디 있는지 알려주는 키
  - 값을 변경하는 동작이 서술된 함수
- `update()`에 전달할 함수는 계산이어야 하고 현재 값을 새로운 값으로 리턴

> 아무래도.. 전자는 액션을 전달하면 액션이 되서 그런 것 같고... 후자는 카피-온-라이트가 적용된 듯

```tsx
function update(object: Record<string, any>, key: string, modify: (value: any) => any) {
  const value = object[key]
  const newValue = modify(value)
  const newObject = objectSet(object, key, newValue)
  return newObject
}

function incrementField(item, field) {
  return update(item, field, value => value + 1)
}
```

## 중첩된 데이터에 `update()` 사용하기

### 숨은 `update()` 찾기

```tsx
function incrementSize(item) {
  const options = item.options // 조회
  const size = options.size // 조회
  const newSize = size + 1 // 변경
  const newOptions = objectSet(options, 'size', newSize) // 설정
  const newItem = objectSet(item, 'options', newOptions) // 설정
  return newItem
}
```

### 가운데에 끼어 있는 조회, 변경, 설정 먼저 `update()`로 교체하기

```tsx
const size = options.size // 조회
const newSize = size + 1 // 변경
const newOptions = objectSet(options, 'size', newSize) // 설정
```

```tsx
function incrementSize(item) {
  const options = item.options // 조회
  const newOptions = update(options, 'size', size => size + 1) // 변경
  const newItem = objectSet(item, 'options', newOptions) // 설정
  return newItem
}
```

```tsx
function incrementSize(item) {
  return update(item, 'options', options => {
    return update(options, 'size', size => size + 1)
  })
}
```

- 변경으로 바뀌면서 다시금 조회, 변경, 설정 리팩토링을 할 수 있음
- `update()`를 중첩해서 부르면 더 깊은 단계로 중첩된 객체에도 사용할 수 있음

## 이중으로 중첩된 객체

### `updateOption()` 도출하기

#### 암묵적 인자가 있는 `update()`를 명시적 인자로 변경한 `updateOption()`

```tsx
function updateOption(item, option, modify) {
  return update(item, 'options', function (options) {
    return update(options, option, modify)
  })
}
```

> 암묵적 인자를 명시적으로 변경했지만 여전히 함수 이름에 있는 인자(options)를 본문에서 참조하고 있음 → 코드의 냄새

### `update2()` 도출하기

#### 암묵적 인자가 있는 `updateOptions()`을 명시적 인자로 변경한 `update2()`

```tsx
function update2(object, key1, key2, modify) {
  return update(object, key1, function (value1) {
    return update(value1, key2, modify)
  })
}

function increment(item) {
  return update2(item, 'options', 'size', size => size + 1)
}
```

- `update2()`는 두 단계로 중첩된 어떤 객체에도 쓸 수 있는 함수
- 값에 접근하기 위한 key1, key2를 모아 리스트로 만들면 경로(path)라고 부를 수 있음 → 경로로 중첩된 데이터의 어떤 부분을 가리키는지 표현할 수 있음

## 삼중으로 중첩된 객체

### `incrementSizeByName()`을 만드는 네 가지 방법

1. `update()`와 `incrementSize()`로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, incrementSize)
}
```

2. `update()`와 `update2()`로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, function (item) {
    return update2(item, 'options', 'size', function (size) {
      return size + 1
    })
  })
}
```

3. `update3()`로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, function (item) {
    return update(item, 'options', function (options) {
      return update(options, 'size', function (size) {
        return size + 1
      })
    })
  })
}
```

4. 조회하고 바꾸고 설정하는 것을 직접 만들기

```tsx
function incrementSizeByName(cart, name) {
  const item = cart[name]
  const options = item.options
  const size = options.size
  const newSize = size + 1
  const newOptions = objectSet(options, 'size', newSize)
  const newItem = objectSet(item, 'options', newOptions)
  const newCart = objectSet(cart, name, newItem)
  return newCart
}
```

> 결국 4가지 다 각자의 문제점이 있겠지만 이 중에서 고르라면 2번째가 제일 나은 것 같고,
> 3번째는... 안 좋은 코드의 예시를 본 것 같은 기분이다 😂

### `update3()` 도출하기

```tsx
function update3(object, key1, key2, key3, modify) {
  return update(object, key1, function (object2) {
    return update2(object2, key2, key3, modify)
  })
}

function incrementSizeByName(cart, name) {
  return update3(cart, name, 'options', 'size', size => size + 1)
}
```

## 반복되는 패턴을 찾고 `nestedUpdate()` 도출하기

> 최고 마케팅 담당자가 `update4()`, `update5()`를 만들어달라고 한다. 이러다간 `update21()`까지 만들어야 할 판.. 패턴을 찾아서 중첩된 개수에 상관없이 쓸 수 있는 함수를 만들어보자.

### 반복되는 패턴

- `updateX()`를 만들려고 한다면 `update()` 안에 `updateX-1()`을 부르기
- `update()`는 첫 번째 키만 사용
- `updateX-1()`에서 나머지 키와 modify 함수를 사용

#### 그렇다면.. `update0()`인 경우는?

- 패턴이 다르게 적용됨!
- 사용하는 키가 없기 때문에 키가 한 개 필요한 `update()`를 부를 수 없음
- X-1이 -1이 되기 때문에 경로 길이를 표현할 수 없음
- 조회나 설정 없이 그냥 변경만 하기 때문에 modify 함수를 바로 호출

```tsx
function update0(value, modify) {
  return modify(value)
}
```

### `nestedUpdate()` 도출하기

#### 중첩된 횟수를 명시적 인자인 depth로 변경하기

```tsx
function updateX(object, depth, key1, key2, key3, modify) {
  return update(object, key1, function (value1) {
    return updateX(value1, dept - 1, key2, key3, modify)
  })
}
```

#### 깊이와 키 개수 맞추기

- **키의 개수와 순서**가 중요 → **배열** 자료 구조가 적절
- 첫 번째 키로 `update()`를 호출한 뒤, 나머지 키로 재귀 함수를 호출

```tsx
function updateX(object, keys, modify) {
  const [key1, ...restOfKeys] = keys
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify())
  })
}
```

#### 키가 더 이상 없는 경우 예외처리하기

```tsx
function updateX(object, keys, modify) {
  if (keys.length === 0) {
    return modify(object) // 재귀 호출이 없음
  }

  const [key1, ...restOfKeys] = keys
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify())
  })
}
```

#### 일반적인 함수명인 nestedUpdate()로 변경하기

```tsx
function nestedUpdate(object, keys, modify) {
  if (keys.length === 0) {
    // 종료 조건
    return modify(object)
  }

  const [key1, ...restOfKeys] = keys
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify()) // 재귀 호출
  })
}
```

#### `nestedUpdate()` 사용하기

```tsx
function incrementSizeByName(cart, name) {
  return nestedUpdate(cart, [name, 'options', 'size'], size => size + 1)
}
```

## 안전한 재귀 사용법

- 종료 조건
- 재귀 호출
- 종료 조건에 다가가기

## 깊이 중첩된 데이터에 추상화 벽 사용하기

```tsx
httpGet('http://my-blog/com/api/category/blog', function (blogCategory) {
  renderCategory(nestedUpdate(blogCategory, ['posts', '12', 'author', 'name'], capitalize))
})
```

- `[’posts’, ‘12’, ‘author’, ‘name’]`
  - 키 경로가 길면 중간 객체가 어떤 키를 가졌는지 기억하기 어려움

> 추상화 벽이 필요한 시점!

### 추상화 벽으로 데이터 구조 줄이기

#### 특정 포스트의 id를 기준으로 블로그를 변경하는 함수

```tsx
function updatePostsById(category, id, modifyPost) {
  return nestedUpdate(category, ['posts', id], modifyPost)
}
```

#### 글쓴이를 수정하는 함수

```tsx
function updateAuthor(post, modifyUser) {
  return update(post, 'author', modifyUser)
}
```

#### 사용자 이름을 대문자로 바꾸는 함수

```tsx
function capitalizeUserName(user) {
  return update(user, 'name', capitalize)
}
```

#### 함수를 합치고 사용하기

```tsx
httpGet(
  'http://my-blog/com/api/category/blog',
  updatePostsById(blogCategory, '12', function (post) {
    return updateAuthor(post, capitalizeUserName)
  }),
)
```

#### 추상화 벽을 사용했을 때 얻을 수 있는 이점

- 기억해야 할 것이 줄어들었음
- 동작의 이름이 있으므로 각각의 동작을 기억하기 쉬움

## 지금까지 배운 고차 함수 리뷰

- 배열을 반복할 때 for 반복문 대신 사용하기
- 중첩된 데이터를 효율적으로 다루기
- 카피-온-라이트 원칙 적용하기
- try/catch 로깅 규칙을 코드화
