# ANALYSIS.md
## การวิเคราะห์เปรียบเทียบ: Monolithic vs Layered Architecture
### ENGSE207 Software Architecture | สัปดาห์ที่ 4

---

## คำถามที่ 1: การเปรียบเทียบโครงสร้าง (5 คะแนน)

### ตารางเปรียบเทียบ

| ข้อมูล | Monolithic (Week 3) | Layered (Week 4) |
|--------|---------------------|------------------|
| จำนวนไฟล์ JS หลัก | 1 ไฟล์ (server.js) | 7 ไฟล์ (controller, service, repository, model, connection, errorHandler, logger) |
| จำนวนบรรทัดทั้งหมด | ~150 บรรทัด | ~600 บรรทัด |
| จำนวน layers | 1 (ทุกอย่างอยู่ใน server.js) | 3 (Presentation, Business Logic, Data Access) |
| ความซับซ้อนโดยรวม | ต่ำ — โครงสร้างเรียบง่าย | สูงกว่า — มีโครงสร้างหลายชั้น |

Layered Architecture มีจำนวนไฟล์และบรรทัดโค้ดมากกว่า Monolithic อย่างเห็นได้ชัด จากไฟล์เดียว 150 บรรทัด กลายเป็น 7 ไฟล์รวมกันประมาณ 600 บรรทัด เหตุผลที่เพิ่มขึ้นมากขนาดนี้เพราะการแยก layer บังคับให้ต้องเขียนโครงสร้างเพิ่มในแต่ละไฟล์ เช่น class definition, constructor, JSDoc comments และ module.exports รวมถึงต้องสร้างไฟล์ utility เพิ่มอย่าง logger.js และ errorHandler.js ที่ใน Monolithic ฝังอยู่ใน server.js โดยตรง

ความซับซ้อนที่เพิ่มขึ้นนั้นคุ้มค่าในระยะยาว แต่ต้องพิจารณาตาม context ของโปรเจกต์ด้วย สำหรับโปรเจกต์ขนาดเล็กที่พัฒนาคนเดียวและไม่มีแผนขยายระบบ ความซับซ้อนที่เพิ่มขึ้นอาจไม่คุ้มค่าในช่วงแรก แต่สำหรับโปรเจกต์ที่มีทีมหรือต้องการ maintainability สูง ความซับซ้อนนี้คุ้มค่ามากเพราะเมื่อต้องแก้บั๊กหรือเพิ่มฟีเจอร์ใหม่ ผู้พัฒนารู้ทันทีว่าต้องไปแก้ที่ไฟล์ไหน ไม่ต้องไล่อ่าน server.js ทั้งไฟล์

---

## คำถามที่ 2: จุดแข็ง-จุดอ่อน (10 คะแนน)

### ตารางวิเคราะห์ Quality Attributes

| Quality Attribute | Monolithic | Layered | คะแนน Layered (1-5) | อธิบายเหตุผล |
|-------------------|------------|---------|---------------------|-------------|
| Maintainability | 2 | 5 | 5 | Layered แยกหน้าที่ชัดเจน เมื่อต้องการแก้ business rule เช่น เพิ่มเงื่อนไข HIGH priority แก้เฉพาะ taskService.js ได้เลย ไม่กระทบ controller หรือ repository |
| Testability | 2 | 5 | 5 | สามารถ unit test แต่ละ layer แยกกันได้ เช่น test taskService.js โดย mock taskRepository ทำให้รู้ว่า bug อยู่ที่ layer ไหน ใน Monolithic ทดสอบได้แค่ integration test เท่านั้น |
| Modifiability | 2 | 4 | 4 | การเปลี่ยน database จาก SQLite เป็น PostgreSQL ใน Layered แก้แค่ taskRepository.js และ connection.js ส่วน Monolithic ต้องแก้ทุกที่ที่มี SQL query ซึ่งกระจายอยู่ใน server.js |
| Reusability | 1 | 4 | 4 | taskService.js และ taskRepository.js สามารถนำกลับมาใช้ใหม่ได้ใน context อื่น เช่น CLI tool หรือ background job โดยไม่ต้องแตะ HTTP layer เลย ใน Monolithic logic ปนกับ Express จนแยกไม่ออก |
| Team Collaboration | 1 | 4 | 4 | ทีมสามารถแบ่งงานได้ชัดเจน คนหนึ่งทำ controller อีกคนทำ service โดยไม่ conflict กัน ใน Monolithic ทุกคนต้องแก้ไฟล์เดียวกันทำให้เกิด merge conflict บ่อย |
| Performance | 4 | 3 | 3 | Monolithic มี overhead น้อยกว่าเพราะไม่มีการเรียกผ่านหลาย function layers แต่ใน Layered มี function call overhead เพิ่มขึ้นเล็กน้อยจากการส่งข้อมูลระหว่าง Controller → Service → Repository อย่างไรก็ตามความต่างนี้แทบไม่มีผลในแอปพลิเคชันทั่วไป |
| Simplicity | 5 | 2 | 2 | Monolithic เข้าใจง่ายกว่ามากสำหรับผู้เริ่มต้น เพราะทุกอย่างอยู่ในที่เดียว ไม่ต้องกระโดดอ่านหลายไฟล์ Layered ต้องใช้เวลาทำความเข้าใจโครงสร้างก่อน |

---

## คำถามที่ 3: สถานการณ์จริง (5 คะแนน)

### สถานการณ์ที่ 1: เพิ่มฟีเจอร์ "Assign Task to User"

ใน Monolithic ต้องแก้ไข server.js ทั้งไฟล์ โดยเพิ่ม column `assigned_to` ใน SQL query ทุกจุดที่มี INSERT และ SELECT รวมถึงเพิ่ม validation logic และ HTTP handler ในไฟล์เดียวกัน ความเสี่ยงสูงเพราะการแก้ในจุดหนึ่งอาจกระทบส่วนอื่นโดยไม่ตั้งใจ

ใน Layered ทำงานเป็นขั้นตอนที่ชัดเจนโดยแก้ไขเพียง 3 จุดที่แยกจากกัน ได้แก่ เพิ่ม field `assigned_to` ใน Task.js model, เพิ่ม business rule ใน taskService.js เช่น ตรวจสอบว่า user ที่ assign มีอยู่จริง, และเพิ่ม SQL query ใน taskRepository.js แต่ละจุดแก้ไขได้อิสระโดยไม่กระทบกัน และสามารถ test แยกกันได้ด้วย

Layered ง่ายกว่าชัดเจน เพราะรู้ทันทีว่าแต่ละส่วนต้องแก้ที่ไหน ลด cognitive load และความเสี่ยงจากการแก้ไขโดยรวม

---

### สถานการณ์ที่ 2: มีบั๊กที่ Validation Logic (ตรวจสอบ title)

ใน Monolithic ต้องไล่อ่าน server.js ทั้งไฟล์เพื่อหาว่า validation อยู่ที่บรรทัดไหน และเมื่อพบแล้วก็ต้องระวังไม่ให้การแก้ไขกระทบ logic อื่นที่อยู่ใกล้กัน เช่น database query หรือ HTTP response ที่อยู่ในบรรทัดถัดไป

ใน Layered รู้ทันทีว่า title validation อยู่ใน Task.js ที่เมธอด `isValid()` และ business rule เพิ่มเติมอยู่ใน taskService.js เปิดไฟล์ถูกต้องตั้งแต่แรก แก้ไขได้เลยโดยไม่ต้องกังวลว่าจะกระทบ controller หรือ repository

Layered ง่ายกว่ามาก เพราะ validation logic อยู่ในที่เดียวที่คาดเดาได้ ลดเวลาในการหาบั๊กและลดความเสี่ยงจากการแก้ผิดไฟล์

---

### สถานการณ์ที่ 3: เปลี่ยนจาก SQLite เป็น PostgreSQL

ใน Monolithic ต้องแก้ไขหลายจุดทั่ว server.js เพราะ SQL queries และ database connection กระจายอยู่ปนกับ business logic และ HTTP handling ต้องแก้ไขทุก query ทีละบรรทัด และเสี่ยงต่อการพลาด query บางจุด

ใน Layered แก้ไขเพียง 2 ไฟล์เท่านั้น คือ database/connection.js สำหรับเปลี่ยน driver จาก sqlite3 เป็น pg และ taskRepository.js สำหรับปรับ SQL syntax ที่ต่างกันระหว่างสองฐานข้อมูล เช่น placeholder จาก `?` เป็น `$1` ส่วน taskService.js และ taskController.js ไม่ต้องแตะเลย เพราะไม่รู้จักฐานข้อมูลอยู่แล้ว

Layered ง่ายกว่ามาก นี่คือตัวอย่างที่ชัดเจนที่สุดของประโยชน์จาก Data Access Layer ที่แยกออกมา การเปลี่ยน database ทั้งระบบใช้เวลาแค่ไม่กี่ชั่วโมง แทนที่จะใช้เป็นวัน

---

## คำถามที่ 4: Trade-offs (5 คะแนน)

### 4.1 Complexity vs Maintainability

Trade-off นี้คุ้มค่าเมื่อโปรเจกต์มีขนาดกลางขึ้นไปหรือมีทีมพัฒนามากกว่า 1 คน ความซับซ้อนที่เพิ่มขึ้นในช่วงแรกเป็นการลงทุนที่คืนทุนเร็วมากเมื่อเริ่มต้องแก้ไขหรือเพิ่มฟีเจอร์ เพราะเวลาที่ประหยัดได้จากการหาบั๊กและแก้โค้ดได้ถูกที่นั้นมากกว่าเวลาที่เสียไปกับการตั้งค่าโครงสร้างในตอนแรกอย่างมาก

กรณีที่คุ้มค่า ได้แก่ โปรเจกต์ที่คาดว่าจะต้องดูแลระยะยาวเกิน 3 เดือน, ทีมที่มีนักพัฒนามากกว่า 1 คน, ระบบที่มี business logic ซับซ้อนและเปลี่ยนแปลงบ่อย และโปรเจกต์ที่ต้องการ unit testing อย่างจริงจัง

กรณีที่ไม่คุ้มค่า ได้แก่ prototype หรือ proof of concept ที่ทำเพื่อทดสอบไอเดีย, โปรเจกต์เล็กที่มี logic ไม่ซับซ้อนและพัฒนาคนเดียว, และงานที่มี deadline กระชั้นชิดและไม่มีแผน maintain ในระยะยาว

### 4.2 Performance Overhead

Performance overhead จากการเรียกผ่าน layers นั้นมีอยู่จริงแต่น้อยมากในทางปฏิบัติ การเรียก function ใน JavaScript แต่ละครั้งใช้เวลาในระดับ microseconds ซึ่งเมื่อเทียบกับเวลาที่ใช้ในการ query database หรือ network latency แล้วแทบไม่มีนัยสำคัญเลย ในการทดสอบจริงความต่างระหว่าง Monolithic และ Layered ในแอปพลิเคชัน CRUD ทั่วไปมักต่ำกว่า 1 millisecond ต่อ request

แอปพลิเคชันที่ performance overhead นี้อาจมีความสำคัญ ได้แก่ ระบบ real-time ที่ต้องการ latency ต่ำมากในระดับ microseconds เช่น high-frequency trading, game server ที่ต้องประมวลผล thousands of operations ต่อวินาที และ embedded systems ที่มีทรัพยากรจำกัดมาก ซึ่งสำหรับแอปพลิเคชัน web ทั่วไปอย่าง Task Board นี้ overhead ดังกล่าวไม่มีผลกระทบใดๆ เลย

---

## คำถามที่ 5: การตัดสินใจเลือกใช้ (5 คะแนน)

### Decision Tree
```
เริ่มต้นโปรเจกต์
│
├─ ขนาดทีม?
│  ├─ 1-2 คน → Monolithic ก็พอ ถ้าโปรเจกต์เล็ก
│  │            แต่ถ้าโปรเจกต์ขนาดกลางขึ้นไป → Layered
│  └─ 3+ คน → Layered เสมอ เพื่อลด merge conflict
│
├─ ขนาดโปรเจกต์?
│  ├─ เล็ก (< 1,000 บรรทัด) → Monolithic ก็เพียงพอ
│  ├─ กลาง (1,000-10,000 บรรทัด) → Layered แนะนำ
│  └─ ใหญ่ (> 10,000 บรรทัด) → Layered จำเป็น
│
├─ ระยะเวลาพัฒนา?
│  ├─ ต้องการเร็ว (< 1 เดือน) → Monolithic ถ้าเป็น prototype
│  │                              Layered ถ้าต้อง maintain ต่อ
│  └─ มีเวลา (> 1 เดือน) → Layered เสมอ
│
└─ ต้องการ maintainability สูง?
   ├─ ใช่ → Layered เสมอ
   └─ ไม่ → Monolithic ก็เพียงพอ
```

### เหตุผลของการตัดสินใจแต่ละข้อ

**ขนาดทีม** เป็นปัจจัยสำคัญที่สุด เพราะปัญหาหลักของ Monolithic ในทีมคือทุกคนต้องแก้ไขไฟล์เดียวกัน ทำให้เกิด merge conflict บ่อย และยากต่อการ review code เพราะไม่รู้ว่าการแก้ของคนหนึ่งกระทบส่วนของอีกคนหรือไม่ Layered แก้ปัญหานี้ได้โดยตรงเพราะแต่ละคนรับผิดชอบ layer ที่ต่างกัน

**ขนาดโปรเจกต์** สัมพันธ์กับจำนวน logic ที่ต้องจัดการ เมื่อโค้ดเกิน 1,000 บรรทัดใน Monolithic จะเริ่มหา logic ที่ต้องการแก้ไขได้ยากขึ้นเรื่อยๆ และความเสี่ยงที่การแก้ไขจะกระทบส่วนอื่นก็สูงขึ้นตาม Layered ช่วยควบคุมความซับซ้อนนี้ได้เพราะแต่ละไฟล์มีหน้าที่ชัดเจน

**ระยะเวลาพัฒนา** บอกถึงความตั้งใจของโปรเจกต์ ถ้าต้องการเร็วและเป็นแค่ prototype ที่จะทิ้งทีหลัง Monolithic เหมาะกว่า แต่ถ้ามีแผนพัฒนาต่อเนื่องควรลงทุนกับ Layered ตั้งแต่แรก เพราะการ refactor จาก Monolithic เป็น Layered ทีหลังนั้นเสียเวลามากกว่าการออกแบบให้ถูกตั้งแต่ต้น

**ความต้องการ maintainability** เป็นตัวสรุปสุดท้าย ถ้าระบบต้องดูแลยาวนานและมีการเปลี่ยนแปลง business rule บ่อย Layered เป็นคำตอบที่ชัดเจน แต่ถ้าเป็นระบบที่สร้างแล้วแทบไม่แตะอีก เช่น internal tool ที่ใช้ครั้งเดียว Monolithic ก็เพียงพอ

---
