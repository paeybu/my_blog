---
path: redux
date: 2020-07-26T01:22:55.506Z
title: พยายามอธิบาย Redux เป็นภาษาคน
description: redux งงจังวุ้ย
---
# Redux
วันนี้เราจะมาเรียน Redux กัน

### Redux คืออะไร
Redux เป็น pattern และ libary เพื่อให้จัดการและอัพเดท state ของ app ได้โดยใช้ event ที่เรียกว่า **actions**

มีหน้าที่เป็น store กลางไว้เก็บ state เวลาจะใช้ใน app เพื่อให้ state มีการเปลี่ยนแปลงอย่างคาดเดาได้

### ใช้ตอนไหน?
- เวลาต้องการใช้ state หลายๆที่ใน app
- state มีการเปลี่ยนแปลงบ่อย
- Logic ในการ update state ซับซ้อน

### Lib ที่ใช้ร่วมกับ redux
- React-Redux เวลาจะใช้กับ react
- Redux Toolkit เป็นวิธีการเขียน logic redux ที่ทาง official redux แนะนำ
- Redux devtools extensions ใช้ไว้ debug redux บน chrome

## Terminology

### Store
Store มีไว้เก็บ state ของ app ในที่ๆเดียว
การที่จะเปลี่ยน state ได้เราจะต้องแจกจ่าย action

### Action
เป็น POJO (Plain old javascript object / Javascript object ธรรมดาๆ) ที่มี field ชื่อว่า **type** คิดว่ามันเป็น event ที่บ่งบอกว่าเกิดอะไรขึ้นใน app เช่น login, logout, addTodo
```javascript
{
  type: 'INCREMENT'
}
```

### Action creators
เป็น function ที่ return action จะได้ไม่ต้องเขียน action object
```javascript
const addTodo = text => {
  return {
    type: 'todos/todoAdded',
    payload: text
  }
}
```

### Reducers
Actions จะเปลี่ยน state ได้อย่างไรขึ้นอยู่กับ **reducers**

### ตัวอย่าง

```javascript
// Function นี้มีเพื่อไว้สร้าง store
import { createStore } from 'redux'

// Reducers จะสังเกตได้ว่า reducers จะรับค่า initialState (state = 0) และเช็คว่า Action ที่ถูกส่งมามีค่า type เป็นอะไร และจะทำการปรับ state โดยมีข้อแม่ว่าต้องไม่ mutate state คือห้ามเปลี่ยน state โดยตรง ต้องส่ง object ใหม่ไปในกรณีที่ state เปลี่ยนแปลง
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}

//สร้าง store โดย store นี้จะมี api เป็น function
//{ subscribe, dispatch, getState }
let store = createStore(counter)

//subscribe เพื่อให้ UI รู้ถึงการเปลี่ยนแปลงของ store
//ในกรณีที่ใช้ react เราจะทำผ่าน react-redux ไม่ต้องใช้วิธีนี้
//บางทีเราอาจจะเก็บค่า state นี้ไว้ใน localStorage
store.subscribe(() => console.log(store.getState()))

// ส่ง action เวลาต้องการจะเปลี่ยนแปลง state
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```