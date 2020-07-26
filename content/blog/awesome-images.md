---
path: redux
date: 2020-07-26T01:22:55.506Z
title: Redux Part 1
description: redux งงจังวุ้ย
---
วันนี้เราจะมาเรียน Redux กัน ไม่ต้องตกใจถ้ามันงง เพราะมันงงแน่ๆ เราต้องเข้าใจมันอย่างแท้จริงก่อน ไม่มีวิธีลัด แล้วหลังจากนี้จะแสดงวิธีที่ทำให้มันง่ายขึ้นโดยใช้ Redux Toolkit
โดย Part แรกนี้เราจะทำความเข้าใจแบบภาพรวมก่อนนะครับ

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
เป็นที่อยู่ของ state เราสร้าง store โดยการส่ง reducer ไปให้ store
store มี api
- `getState`
- `subscribe`
- `dispatch`

### Dispatch
หากต้องการจะ update state เราจะต้อง เรียกฟังชั่น `dispatch` โดยส่ง action เข้าไป

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
Reducer เป็น function ที่จะได้รับค่า state และ action และมีหน้าที่ๆจะตัดสินใจว่าจะทำอะไรกับ state โดยขึ้นอยู่กับ action

มีกฏบังคับอยู่
- ควรจะคำนวณ state ใหม่โดยขึ้นอยู่กับ state ปัจจุบันและ actions
- **ห้าม**ปรับ state ปัจจุบัน ต้อง update แบบ immutable โดยการ copy state เดิมและเปลี่ยนแปลงค่าในตัว copy แล้ว return ตัวใหม่คืน
- ห้ามทำ async หรือก่อ side effects

Logic ในการเขียน reducers
- สนใจ action ประเภทนี้ไหม
    - ถ้าใช่ copy state แล้ว update ค่าแล้ว return กลับไป
- ถ้าไม่สนใจ return state เดิม

### Redux Application Data Flow
- Setup เบื้องต้น
    - สร้าง Redux store โดยการส่ง root reducer function เข้าไป
    - Store เรียก rootReducer และรับ return value มาเก็บเป็น initial state
    - UI ไปเช็ค state ใน store แล้วดูว่าควร render อะไร แถมยัง subscribe ไว้กับ store เพื่่อจะได้รับรู้เวลา state มีการเปลี่ยนแปลง
- Updates
    - เกิดอะไรบางอย่าง เช่น User คลิกปุ่ม
    - ทำการ dispatch action ไปยัง store `dispatch({type: 'increment'})`
    - Store รัน reducer function เพื่อเช็คว่า action นั้นๆ ควรจะเปลี่ยน state ยังไง
    - Store ประกาศให้ ui รับรู้การเปลี่ยนแปลง
    - UI เช็คว่ามีการเปลี่ยนแปลงเกิดขึ้นไหม
    - UI rerender และแสดงการเปลี่ยนแปลง
    

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