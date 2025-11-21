# เอกสารประกอบข้อมูลการทำงานของโปรเจคโซล่าเซลปี 4

# รายละเอียดการทำงานเกือบทั้งหมดเกี่ยวกับโปรเจค

### [**Pvlib**](./pvlib/pvlib.md) คลิกพื่อไปหน้าเอกสาร

> เอาไว้ใช้ในการคำนวณตำแหน่งของพระอาทิตย์ด้วยสูตรที่ถูกต้องและแม่นยำที่มีหลักการตามเอกสาร

- pvlib คำนวณจาก สมการจริง (deterministic astronomical equations)
- ฟังก์ชันหลักของ pvlib ที่ใช้คำนวณตำแหน่งดวงอาทิตย์คือ
  - pvlib.solarposition.get_solarposition
  - ซึ่งเรียก NREL SPA algorithm (Solar Position Algorithm)
  - สามารถใช้ได้แม้มีอินพุตแค่ latitude + longitude + elevation + time

> pvlib ใช้อ้างอิงเอกสารทางการ:
>
> [**NREL – Solar Position Algorithm for Solar Radiation Applications**](https://docs.nrel.gov/docs/fy08osti/34302.pdf)
> *   ผู้เขียน: Reda, I., & Andreas, A. (2008)
> *   NREL/TP-560-34302
> เอกสารฉบับเต็ม (PDF):  *Solar Position Algorithm for Solar Radiation Applications* (สามารถค้นหาได้, เป็นไฟล์ 55 หน้า)
