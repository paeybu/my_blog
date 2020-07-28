---
path: redux-toolkit
date: 2020-07-26T11:17:12.565Z
title: ทำชีวิตให้สบายขึ้นโดยการใช้ Redux Toolkit
description: เมื่อ redux มันน่าปวดหัว ต้องหาทางแก้
---
# ทำให้ชีวิตสบายขึ้นด้วย Redux Toolkit

### มีไว้เพื่ออะไร?

ไว้ใช้เป็นมาตรฐานในการเขียน logic ของ redux 

ถูกสร้างมาเพื่อแก่ไขข้อกังวลใน Redux 3 ข้อ

* ตั้ง redux store วุ่นวายจัง
* ต้องเพิ่ม package เยอะแยะกว่าจะทำไรได้
* มี boilerplate code เยอะเกิน

### มีอะไรใน redux toolkit?

Redux toolkit มี API ให้ใช้ตามนี้ เราจะได้เห็นการใช้งานของ API พวกนี้ในไม่ช้า

* `configureStore()` เป็น wrapper ของ createStore ทำให้ชีวิตง่ายขึ้น สามารถรวบรวม slice reducers และเพิ่ม middleware ได้อย่างง่ายดาย แถมยังรวม redux-thunk ไว้แล้ว และยัง config redux devtools ไว้เรียบร้อย
* `createReducer()` สร้าง reducer functions ด้วย lookup table แทนที่จะต้องเขียน switch statement แถมยังใช้ library `immer` เพื่อให้ update ได้ง่ายๆโดยไม่ต้องทำ immutable update
* `createAction()` ใช้เพื่อสร้าง action creator ตาม action type string ที่ใส่เข้าไป แถมยังมี `.toString()` เพื่อให้เป็นค่า constants ได้
* `createSlice()` รับ object ที่เป็น reducer functions ชื่อ slice และ initial state เพื่อสร้าง slice reducer ที่มี action creators และ action types
* `createAsyncThunk()` รับ action type string และ function ที่ return เป็น promise และสร้าง thunk ที่ dispatch pending/fullfilled/rejected ตามสถานะของ promise
* `createEntityAdapter` สร้าง reducers ที่สามารถใช้ซ้ำได้ และ selectors เพื่อให้จัดการ normalized data ใน store
* `createSelector` จาก library `Reselect`

### การติดตั้ง
หากอยากสร้าง project โดยการใช้ redux toolkit สามารถใช้ template official ของ redux js ได้เลย

`npx create-react-app my-app --template redux`

เพิ่มใน app ที่สร้างมาแล้ว

`npm install @reduxjs/toolkit`

### จากการเขียน Redux แบบเดิม สู่การเขียนด้วย redux toolkit

เราจะลองดูตัวอย่างการเขียน redux แบบเดิมกัน

```javascript
// สร้าง constants action type
const INCREMENT = 'INCREMENT'
const DECREMENT = 'DECREMENT'

// action creators
function increment() {
  return { type: INCREMENT }
}

function decrement() {
  return { type: DECREMENT }
}

// reducer
function counter(state = 0, action) {
  switch (action.type) {
    case INCREMENT:
      return state + 1
    case DECREMENT:
      return state - 1
    default:
      return state
  }
}

// สร้าง store
const store = Redux.createStore(counter)

document.getElementById('increment').addEventListener('click', () => {
  store.dispatch(increment())
})
```
### `configureStore`

มาๆ เราจะเริ่มการ refactor กันทีละส่วน เราจะมาเริ่มกันที่ `configureStore` 

โดยปกติแล้วเราจะใช้ `createStore` ในการสร้าง store โดยการส่ง root reducer function ไปเป็น argument 

Redux toolkit มี configure store ที่ wrap `createStore()` ทำหน้าที่เดียวกันแต่ยัง setup redux devtools ไว้แล้วโดยไม่ต้องเอามาใส่

เพราะฉนั้นเราแทนที่ createStore ด้วย configureStore ได้ โดยที่ `createStore` รับ object ที่มี fields เป็นชื่อของ reducer

```javascript
// Before:
const store = createStore(counter)

// After:
const store = configureStore({
  reducer: counter
})
```

จะเห็นได้ว่าไม่ค่อยมีไรเปลี่ยนแปลงเท่าไหร่ แต่ตอนนี้เราก็จะใช้ redux devtools ได้ทันที

### `createAction`
ง่ายๆเลย รับ action type เป็น string เป็น argument แล้ว return มาเป็น action creator function 
จะเห็นได้ว่าชื่อจริงๆมันผิดนะ เราไม่ได้สร้าง action แต่สร้าง action creator แต่ยังไงก็เถอะชื่อนั้นมันยาวเกินเขาเลยใช้แค่นี้

```javascript
//วิธีเดิม สร้าง action type เป็น constant และ action creator
const INCREMENT = 'INCREMENT'

function incrementOriginal() {
  return { type: INCREMENT }
}

console.log(incrementOriginal())
// {type: "INCREMENT"}

//วิธีใหม่ ใช้ createAction
const incrementNew = createAction('INCREMENT')

console.log(incrementNew())
// {type: "INCREMENT"}
```

ทีนี้ถ้าเราอยากได้ตัว string ของ type นี้กับมาเป็น reference ล่ะ? เพราะเราไม่ได้สร้าง const ชื่อนี้มา เราสามารถอ้างอิงตัว `incrementNew` นี้แล้วเรียก `.toString()` ได้ หรือจะใช้ `.type` ก็ได้

```javascript
const increment = createAction('INCREMENT')

console.log(increment.toString())
// "INCREMENT"

console.log(increment.type)
// "INCREMENT"
```

ที่นี้ลองมาดูตัวอย่างที่เขียน counter ก่อนหน้านี้แต่ใช้ `createAction` และ `configureStore`

```javascript
const increment = createAction('INCREMENT')
const decrement = createAction('DECREMENT')

function counter(state = 0, action) {
  switch (action.type) {
    case increment.type:
      return state + 1
    case decrement.type:
      return state - 1
    default:
      return state
  }
}

const store = configureStore({
  reducer: counter
})

document.getElementById('increment').addEventListener('click', () => {
  store.dispatch(increment())
})
```

### `createReducer`
คืออันนี้เด็ดจริง เพราะก่อนหน้านี้เราต้อง check ค่า action.type แล้วคิดว่าจะทำอะไร ก็จะต้องใช้ conditional logic เช่น `if/else` แต่ปกติจะใช้ `switch`

ทีนี้ถ้าเราใช้ `createReducer` เราจะสามารถสร้าง reducers function โดยใช้ lookup table object ได้ โดยที่ key ของ object ก็คือ action type string นี่แหละ ดูตัวอย่างแล้วจะเข้าใจเอง
เราจะใช้ ES6 computed property syntax เพื่อที่จะให้ key เป็นชื่อของ variables

```javascript
const increment = createAction('INCREMENT')
const decrement = createAction('DECREMENT')

// รับ initialState และ object ที่เป็น lookup table
const counter = createReducer(0, {
  //ให้ key เป็นชื่อของ action type
  [increment.type]: state => state + 1,
  [decrement.type]: state => state - 1
})
```
แต่เราสามารถย่อได้อีกหน่อย เพราะว่าเวลาใช้ computed property syntax มันจะเรียก `.toString()` บน variable นั้นๆ ถ้าจำได้จะเห็นว่า createAction มี .toString() ให้ใช้เช่นกัน เพราะฉนั้นเราตัด .type ออกได้เลย

```javascript
const counter = createReducer(0, {
  [increment]: state => state + 1,
  [decrement]: state => state - 1
})
```

คราวนี้ ตัวอย่าง counter หลังจากใช้ `createAction` และ `createReducer` ก็จะเป็นเช่นนี้

```javascript
const increment = createAction('INCREMENT')
const decrement = createAction('DECREMENT')

const counter = createReducer(0, {
  [increment]: state => state + 1,
  [decrement]: state => state - 1
})

const store = configureStore({
  reducer: counter
})
```

### `createSlice`
แต่ยังไม่หมด! ทำไมเราจะต้องสร้าง action creators พวกนี้แยกด้วยล่ะ ต้องมานั่งกรอก action type string เช่น INCREMENT, DECREMENT อีก ทั้งที่ส่วนสำคัญจริงๆคือตรง reducer functions

พระเอกของเรา `createSlice` จะมาแก้ปัญหานี้เอง เราสามารถส่ง object ที่มี reducer functions และมันจะสร้าง action type strings และ action creator functions โดยเอามาจากชื่อใน reducers ที่เราสร้าง!!

สรุปง่ายๆคือเราสามารถแทนที่ `createAction` และ `createReducer` ด้วย `createSlice` ตัวเดียว จบเลย

`createSlice` จะ return slice object ที่มี reducer function เป็น field ชื่อ `reducer` และ action creators ใน object ชื่อ `actions`

ลองมาดูตัวอย่างของ counter หลังจากใช้ `createSlice` กัน...

```javascript
// ส่ง Object ที่มีชื่อ, initialState, และ reducers
// โดยจะสร้าง action ตามชื่อของ key ใน reducers เลย
const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1
  }
})

const store = configureStore({
  reducer: counterSlice.reducer
})
```

ดึง reducer และ actions จาก slice ด้วย ES6 destructuring syntax
```javscript
const { actions, reducer } = counterSlice
const { increment, decrement } = actions
```

บทถัดไปเราจะมาดูตัวอย่างที่ยากขึ้นกันกัน แล้วจะเห็นว่า Redux Toolkit เจ๋งกว่านี้อีก เพราะมันใช้ immer ทำให้ update state ง่ายโคตรๆ
