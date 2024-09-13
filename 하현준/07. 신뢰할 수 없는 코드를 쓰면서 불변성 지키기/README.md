### 방어적 복사

안전지대 밖으로 나가는 데이터는 잠재적으로 바뀔 수 있고, 마찬가지로 신뢰할 수 없는 코드에서 안전지대로 들어오는 데이터 역시 잠재적으로 바뀔 수 있다.

이를 위해 데이터가 바뀌는 것을 완벽히 막아주는 원칙인 방어적 복사가 필요하다.

## 방어적 복사는 원본이 바뀌는 것을 막아준다.

들어오고 나가는 데이터의 복사본을 만드는 것 방어적 복사가 동작하는 방식.

**방어적 복사 규칙**

1. 데이터가 안전한 코드에서 나갈 때 복사하기
2. 안전한 코드로 데이터가 들어올 때 복사하기

**방어적 복사 구현하기**

```tsx
function add_item_to_cart(cart: Cart, name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
  
  // 넘기기 전에 복사
  const cart_copy = deepCopy(shopping_cart);
  black_friday_promotion(cart_copy);
  // 들어오는 데이터를 위한 복사
  shopping_cart = deepCopy(cart_copy);
}
```

코드를 보고 복사본을 만드는 이유를 모를 수 있기 때문에 새로운 함수로 분리

```tsx
function add_item_to_cart(cart: Cart, name: string, price: number) {
  // ...
  
	shopping_cart = black_friday_promotion(shopping_cart);
}

function black_friday_promotion(cart: Cart) {
  const cart_copy = deepCopy(shopping_cart);
  black_friday_promotion(cart_copy);
  return deepCopy(cart_copy);
}
```

## 우리에겐 익숙한 방어적 복사?

### 웹 API속에 방어적 복사

API는 암묵적으로 방어적 복사를 한다. 

클라이언트는 데이터를 인터넷을 통해 API로 보내려고 직렬화하는데 이때 **JSON 데이터는 깊은 복사본**이다.

### 얼랭(Erlang)과 엘릭서(Elixir)에서 방어적 복사

얼랭과 엘릭서는 방어적 복사를 잘 구현했다.

- Erlang은 Elixir가 지원하지 않는 동작을 일부 지원한다. 예를 들어 Erlang에서는 논리연산자인 `and` 및 `or` 를 지원하지만, Elixir는 그렇지 않다.
- Erlang에서는 `:` 를 활용해 함수를 부른다, 그러나 Elixir는 `.` 연산자를 통해 함수를 부른다.
- Elixir에서는 변수를 한번 이상 지정 가능하지만, Erlang에서는 변수를 한번 이상 지정할 수 없다. 만약 그렇게 하려한다면 Erlang은 에러를 발생시킬 것이다.
- Elixir는 인자(argument)에 대한 기본값(default value)가 정의되어 있으나 Erlang은 그렇지 않다. 전체적으로 두 언어는 신뢰성이 높고 유지보수성이 높다. 최근에는 Elixir가 둘 중 더 많이 활용되는데, Elixir가 기능적으로 지원하는 것이 더 많고, 문법이 단순하며, 스케일링이 용이하기 때문이다.

```elixir
iex> list = [{:c, 1}, {:d, 2}]
[c: 1, d: 2]

iex> list == [c: 1, d: 2]
true

iex> String.length("elixir")
6
```

|  | 카피-온-라이트 | 방어적 복사 |
| --- | --- | --- |
| 언제 쓰나요? | 통제할 수 있는 데이터를 바꿀 때 | 신뢰할 수 없는 코드와 데이터를 주고받아야 할 때 방어적 복사를 쓴다. |
| 어디서 쓰나요? | 안전지대 어디서나 쓸 수 있다. <br/> 카피-온-라이트를 안전지대를 만드는 역할을 한다. | 안전지대에 경계에서 데이터가 오고갈 때 사용 |
| 복사 방식 | 얕은 복사 | 깊은 복사 |
| 규칙 | 1. 바꿀 데이터의 얕은 복사를 만든다.<br/>2. 복사본을 변경한다.<br/>3. 복사본을 리턴한다. | 1. 안전지대로 들어오는 데이터에 깊은 복사를 만든다. <br/> 2. 안전지대에서 나가는 데이터에 깊은 복사를 만든다. |

### 자바스크립트에서 깊은 복사를 구현하는 것은 어렵다.

```tsx
function deepCopy(thing) {
  if(Array.isArray(thing)) {
    // 배열일 경우
    var copy = [];
    for(var i = 0; i < thing.length; i++)
      copy.push(deepCopy(thing[i]));
    return copy;
  } else if (thing === null) {
    // null
    return null;
  } else if(typeof thing === "object") {
    // 객체일 경우
    var copy = {};
    var keys = Object.keys(thing);
    for(var i = 0; i < keys.length; i++) {
      var key = keys[i];
      // 재귀함수 실행
      copy[key] = deepCopy(thing[key]);
    }
    return copy;
  } else {
    // 문자열, 숫자, 불리언, 함수
    return thing;
  }
}
```

그렇다면 lodash의 cloneDeep와 toss에서 만든 라이브러리를 살펴보자.

<details>
<summary>
<a href="https://github.com/lodash/lodash">lodash의 cloneDeep</a></summary>
<a href="https://github.com/lodash/lodash/blob/main/src/cloneDeep.ts">
cloneDeep.js</a> 

<pre>

    import baseClone from './.internal/baseClone.js';
    
    /** Used to compose bitmasks for cloning. */
    const CLONE_DEEP_FLAG = 1;
    const CLONE_SYMBOLS_FLAG = 4;
    
    /**
     * This method is like `clone` except that it recursively clones `value`.
     * Object inheritance is preserved.
     *
     * @since 1.0.0
     * @category Lang
     * @param {*} value The value to recursively clone.
     * @returns {*} Returns the deep cloned value.
     * @see clone
     * @example
     *
     * const objects = [{ 'a': 1 }, { 'b': 2 }]
     *
     * const deep = cloneDeep(objects)
     * console.log(deep[0] === objects[0])
     * // => false
     */
    function cloneDeep(value) {
        return baseClone(value, CLONE_DEEP_FLAG | CLONE_SYMBOLS_FLAG);
    }
    
    export default cloneDeep;

</pre>

<br />
<a href="https://github.com/lodash/lodash/blob/main/src/.internal/baseClone.ts">baseClone.js</a>
<pre>
    
    import Stack from './Stack.js'
    import arrayEach from './arrayEach.js'
    import assignValue from './assignValue.js'
    import cloneBuffer from './cloneBuffer.js'
    import copyArray from './copyArray.js'
    import copyObject from './copyObject.js'
    import cloneArrayBuffer from './cloneArrayBuffer.js'
    import cloneDataView from './cloneDataView.js'
    import cloneRegExp from './cloneRegExp.js'
    import cloneSymbol from './cloneSymbol.js'
    import cloneTypedArray from './cloneTypedArray.js'
    import copySymbols from './copySymbols.js'
    import copySymbolsIn from './copySymbolsIn.js'
    import getAllKeys from './getAllKeys.js'
    import getAllKeysIn from './getAllKeysIn.js'
    import getTag from './getTag.js'
    import initCloneObject from './initCloneObject.js'
    import isBuffer from '../isBuffer.js'
    import isObject from '../isObject.js'
    import isTypedArray from '../isTypedArray.js'
    import keys from '../keys.js'
    import keysIn from '../keysIn.js'
    
    /** Used to compose bitmasks for cloning. */
    const CLONE_DEEP_FLAG = 1
    const CLONE_FLAT_FLAG = 2
    const CLONE_SYMBOLS_FLAG = 4
    
    /** `Object#toString` result references. */
    const argsTag = '[object Arguments]'
    const arrayTag = '[object Array]'
    const boolTag = '[object Boolean]'
    const dateTag = '[object Date]'
    const errorTag = '[object Error]'
    const mapTag = '[object Map]'
    const numberTag = '[object Number]'
    const objectTag = '[object Object]'
    const regexpTag = '[object RegExp]'
    const setTag = '[object Set]'
    const stringTag = '[object String]'
    const symbolTag = '[object Symbol]'
    const weakMapTag = '[object WeakMap]'
    
    const arrayBufferTag = '[object ArrayBuffer]'
    const dataViewTag = '[object DataView]'
    const float32Tag = '[object Float32Array]'
    const float64Tag = '[object Float64Array]'
    const int8Tag = '[object Int8Array]'
    const int16Tag = '[object Int16Array]'
    const int32Tag = '[object Int32Array]'
    const uint8Tag = '[object Uint8Array]'
    const uint8ClampedTag = '[object Uint8ClampedArray]'
    const uint16Tag = '[object Uint16Array]'
    const uint32Tag = '[object Uint32Array]'
    
    /** Used to identify `toStringTag` values supported by `clone`. */
    const cloneableTags = {}
    cloneableTags[argsTag] = cloneableTags[arrayTag] =
    cloneableTags[arrayBufferTag] = cloneableTags[dataViewTag] =
    cloneableTags[boolTag] = cloneableTags[dateTag] =
    cloneableTags[float32Tag] = cloneableTags[float64Tag] =
    cloneableTags[int8Tag] = cloneableTags[int16Tag] =
    cloneableTags[int32Tag] = cloneableTags[mapTag] =
    cloneableTags[numberTag] = cloneableTags[objectTag] =
    cloneableTags[regexpTag] = cloneableTags[setTag] =
    cloneableTags[stringTag] = cloneableTags[symbolTag] =
    cloneableTags[uint8Tag] = cloneableTags[uint8ClampedTag] =
    cloneableTags[uint16Tag] = cloneableTags[uint32Tag] = true
    cloneableTags[errorTag] = cloneableTags[weakMapTag] = false
    
    /** Used to check objects for own properties. */
    const hasOwnProperty = Object.prototype.hasOwnProperty
    
    /**
     * Initializes an object clone based on its `toStringTag`.
     *
     * **Note:** This function only supports cloning values with tags of
     * `Boolean`, `Date`, `Error`, `Map`, `Number`, `RegExp`, `Set`, or `String`.
     *
     * @private
     * @param {Object} object The object to clone.
     * @param {string} tag The `toStringTag` of the object to clone.
     * @param {boolean} [isDeep] Specify a deep clone.
     * @returns {Object} Returns the initialized clone.
     */
    function initCloneByTag(object, tag, isDeep) {
      const Ctor = object.constructor
      switch (tag) {
        case arrayBufferTag:
          return cloneArrayBuffer(object)
    
        case boolTag:
        case dateTag:
          return new Ctor(+object)
    
        case dataViewTag:
          return cloneDataView(object, isDeep)
    
        case float32Tag: case float64Tag:
        case int8Tag: case int16Tag: case int32Tag:
        case uint8Tag: case uint8ClampedTag: case uint16Tag: case uint32Tag:
          return cloneTypedArray(object, isDeep)
    
        case mapTag:
          return new Ctor
    
        case numberTag:
        case stringTag:
          return new Ctor(object)
    
        case regexpTag:
          return cloneRegExp(object)
    
        case setTag:
          return new Ctor
    
        case symbolTag:
          return cloneSymbol(object)
      }
    }
    
    /**
     * Initializes an array clone.
     *
     * @private
     * @param {Array} array The array to clone.
     * @returns {Array} Returns the initialized clone.
     */
    function initCloneArray(array) {
      const { length } = array
      const result = new array.constructor(length)
    
      // Add properties assigned by `RegExp#exec`.
      if (length && typeof array[0] === 'string' && hasOwnProperty.call(array, 'index')) {
        result.index = array.index
        result.input = array.input
      }
      return result
    }
    
    /**
     * The base implementation of `clone` and `cloneDeep` which tracks
     * traversed objects.
     *
     * @private
     * @param {*} value The value to clone.
     * @param {number} bitmask The bitmask flags.
     *  1 - Deep clone
     *  2 - Flatten inherited properties
     *  4 - Clone symbols
     * @param {Function} [customizer] The function to customize cloning.
     * @param {string} [key] The key of `value`.
     * @param {Object} [object] The parent object of `value`.
     * @param {Object} [stack] Tracks traversed objects and their clone counterparts.
     * @returns {*} Returns the cloned value.
     */
    function baseClone(value, bitmask, customizer, key, object, stack) {
      let result
      const isDeep = bitmask & CLONE_DEEP_FLAG
      const isFlat = bitmask & CLONE_FLAT_FLAG
      const isFull = bitmask & CLONE_SYMBOLS_FLAG
    
      if (customizer) {
        result = object ? customizer(value, key, object, stack) : customizer(value)
      }
      if (result !== undefined) {
        return result
      }
      if (!isObject(value)) {
        return value
      }
      const isArr = Array.isArray(value)
      const tag = getTag(value)
      if (isArr) {
        result = initCloneArray(value)
        if (!isDeep) {
          return copyArray(value, result)
        }
      } else {
        const isFunc = typeof value === 'function'
    
        if (isBuffer(value)) {
          return cloneBuffer(value, isDeep)
        }
        if (tag === objectTag || tag === argsTag || (isFunc && !object)) {
          result = (isFlat || isFunc) ? {} : initCloneObject(value)
          if (!isDeep) {
            return isFlat
              ? copySymbolsIn(value, copyObject(value, keysIn(value), result))
              : copySymbols(value, Object.assign(result, value))
          }
        } else {
          if (isFunc || !cloneableTags[tag]) {
            return object ? value : {}
          }
          result = initCloneByTag(value, tag, isDeep)
        }
      }
      // Check for circular references and return its corresponding clone.
      stack || (stack = new Stack)
      const stacked = stack.get(value)
      if (stacked) {
        return stacked
      }
      stack.set(value, result)
    
      if (tag === mapTag) {
        value.forEach((subValue, key) => {
          result.set(key, baseClone(subValue, bitmask, customizer, key, value, stack))
        })
        return result
      }
    
      if (tag === setTag) {
        value.forEach((subValue) => {
          result.add(baseClone(subValue, bitmask, customizer, subValue, value, stack))
        })
        return result
      }
    
      if (isTypedArray(value)) {
        return result
      }
    
      const keysFunc = isFull
        ? (isFlat ? getAllKeysIn : getAllKeys)
        : (isFlat ? keysIn : keys)
    
      const props = isArr ? undefined : keysFunc(value)
      arrayEach(props || value, (subValue, key) => {
        if (props) {
          key = subValue
          subValue = value[key]
        }
        // Recursively populate clone (susceptible to call stack limits).
        assignValue(result, key, baseClone(subValue, bitmask, customizer, key, value, stack))
      })
      return result
    }
    
    export default baseClone
</pre>
</details>


<details>
<summary>
 toss에서 개발한 <a href="https://github.com/toss/es-toolkit">es-toolkit</a>
</summary>

<a href="https://github.com/toss/es-toolkit/blob/main/src/object/cloneDeep.ts">cloneDeep.ts</a>
<pre>
    function cloneDeepImpl<T>(obj: T, stack = new Map<any, any>()): T {
      if (isPrimitive(obj)) {
        return obj as T;
      }
    
      if (stack.has(obj)) {
        return stack.get(obj) as T;
      }
    
      if (Array.isArray(obj)) {
        const result: any = new Array(obj.length);
        stack.set(obj, result);
    
        for (let i = 0; i < obj.length; i++) {
          result[i] = cloneDeepImpl(obj[i], stack);
        }
    
        // For RegExpArrays
        if (Object.prototype.hasOwnProperty.call(obj, 'index')) {
          // eslint-disable-next-line
          // @ts-ignore
          result.index = obj.index;
        }
        if (Object.prototype.hasOwnProperty.call(obj, 'input')) {
          // eslint-disable-next-line
          // @ts-ignore
          result.input = obj.input;
        }
    
        return result as T;
      }
    
      if (obj instanceof Date) {
        return new Date(obj.getTime()) as T;
      }
    
      if (obj instanceof RegExp) {
        const result = new RegExp(obj.source, obj.flags);
    
        result.lastIndex = obj.lastIndex;
    
        return result as T;
      }
    
      if (obj instanceof Map) {
        const result = new Map();
        stack.set(obj, result);
    
        for (const [key, value] of obj.entries()) {
          result.set(key, cloneDeepImpl(value, stack));
        }
    
        return result as T;
      }
    
      if (obj instanceof Set) {
        const result = new Set();
        stack.set(obj, result);
    
        for (const value of obj.values()) {
          result.add(cloneDeepImpl(value, stack));
        }
    
        return result as T;
      }
    
      // eslint-disable-next-line @typescript-eslint/ban-ts-comment
      // @ts-ignore
      if (typeof Buffer !== 'undefined' && Buffer.isBuffer(obj)) {
        // eslint-disable-next-line @typescript-eslint/ban-ts-comment
        // @ts-ignore
        return obj.subarray() as T;
      }
    
      if (isTypedArray(obj)) {
        const result = new (Object.getPrototypeOf(obj).constructor)(obj.length);
        stack.set(obj, result);
    
        for (let i = 0; i < obj.length; i++) {
          result[i] = cloneDeepImpl(obj[i], stack);
        }
    
        return result as T;
      }
    
      if (obj instanceof ArrayBuffer || (typeof SharedArrayBuffer !== 'undefined' && obj instanceof SharedArrayBuffer)) {
        return obj.slice(0) as T;
      }
    
      if (obj instanceof DataView) {
        const result = new DataView(obj.buffer.slice(0));
        stack.set(obj, result);
    
        copyProperties(result, obj, stack);
    
        return result as T;
      }
    
      // For legacy NodeJS support
      if (typeof File !== 'undefined' && obj instanceof File) {
        const result = new File([obj], obj.name, { type: obj.type });
        stack.set(obj, result);
    
        copyProperties(result, obj, stack);
    
        return result as T;
      }
    
      if (obj instanceof Blob) {
        const result = new Blob([obj], { type: obj.type });
        stack.set(obj, result);
    
        copyProperties(result, obj, stack);
    
        return result as T;
      }
    
      if (obj instanceof Error) {
        const result = new (obj.constructor as { new (): Error })();
        stack.set(obj, result);
    
        result.message = obj.message;
        result.name = obj.name;
        result.stack = obj.stack;
        result.cause = obj.cause;
    
        copyProperties(result, obj, stack);
    
        return result as T;
      }
    
      if (typeof obj === 'object' && obj !== null) {
        const result = {};
        stack.set(obj, result);
    
        copyProperties(result, obj, stack);
    
        return result as T;
      }
    
      return obj;
    }
</pre>
</details>


    
