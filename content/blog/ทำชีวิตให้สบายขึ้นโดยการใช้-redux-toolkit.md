---
path: redux-toolkit
date: 2020-07-26T11:17:12.565Z
title: ทำชีวิตให้สบายขึ้นโดยการใช้ Redux Toolkit
description: เมื่อ redux มันน่าปวดหัว ต้องหาทางแก้
---

### มีไว้เพื่ออะไร?

ไว้ใช้เป็นมาตรฐานในการเขียน logic ของ redux 

ถูกสร้างมาเพื่อแก่ไขข้อกังวลใน Redux 3 ข้อ

* ตั้ง redux store วุ่นวายจัง
* ต้องเพิ่ม package เยอะแยะกว่าจะทำไรได้
* มี boilerplate code เยอะเกิน

### มีอะไรมั่งใน redux toolkit?

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

*เดี๋ยวมาต่อ ขี้เกียจอยู่*
