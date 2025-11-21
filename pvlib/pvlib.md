# **Pvlib**

[**HOME**](../README.md) คลิกกลับเอกสารหน้าแรก

### **เอกสารประกอบการเรียกใช้งานของ library pvlib**

> [**NREL – Solar Position Algorithm for Solar Radiation Applications**](https://docs.nrel.gov/docs/fy08osti/34302.pdf)

# **input ที่ใช้ในการคำนวณ**

- **ข้อมูลพิกัดตำแหน่ง  
พิกัดทางภูมิศาสตร์ที่ใช้อธิบายตำแหน่งบนพื้นผิวโลก**

  - **ละติจูด (Latitude)**
    - ละติจูดของสถานที่
    - ตำแหน่งที่อยู่เหนือหรือใต้ของเส้นศูนย์สูตร
  - **ลองจิจูด (Longitude)**
    - ลองติจูดของสถานที่
    - ตำแหน่งที่อยู่ตะวันออกหรือตะวันตกของเส้นเมริเดียนหลัก (ผ่านกรีนิช) 
  - **ความสูง (Elevation)**
    - คือความสูงเหนือหรือต่ำกว่าระดับน้ำทะเล 

# **Output ค่าที่ได้รับกลับมา**

- **ชุดข้อมูลผลลัพจะมี**
  - **Azimuth**
    - มุมทิศทางของดวงอาทิตย์บนระนาบแนวนอน
    - วัดจาก ทิศเหนือ = 0° หมุนตามเข็มนาฬิกา → ใช้เข็มทิศ
  - **Elevation (หรือ Apparent Elevation)**
    - มุมเงยของดวงอาทิตย์จาก ขอบฟ้า (horizon) ขึ้นไปบนท้องฟ้า
    - 0° = อยู่ที่ขอบฟ้า, 90° = อยู่ตรงเหนือหัว
    - ใช้ inclinometer/ระดับน้ำ เงยขึ้นตามค่าที่ได้
  - **Zenith**
    - มุมจากจุดเหนือศีรษะ (zenith point) ลงมาถึงดวงอาทิตย์
    - มุมกลับด้านจาก elevation (ใช้ในคำนวณ ไม่ได้วัดตรง ๆ)
    - ความสัมพันธ์คือ: Zenith = 90° − Elevation
    - ใช้ในสูตรคำนวณรังสี (เช่น GHI = DNI × cos(zenith) + DHI)

# รายละเอียดการทำงาน

1) **Pvlib คำนวณจากสมการจริง:** (deterministic astronomical equations)

   - Function หลัก: `pvlib.solarposition.get_solarposition`
   - ใช้ [NREL SPA algorithm](https://www.nrel.gov/solar/solar-position.html) (Solar Position Algorithm)
   - Input ขั้นต่ำ: `latitude + longitude + elevation + time`
   - ค่า Default: `pressure = 101325 Pa, temperature = 12°C`

2) **เอกสารอ้างอิงของ pvlib:**

   - [NREL – Solar Position Algorithm for Solar Radiation Applications](https://www.nrel.gov/docs/fy08osti/34302.pdf)
   - ชื่อเอกสาร: Reda, I., & Andreas, A. (2008), NREL/TP-560-34302
   - โค้ดที่อ้างอิง:
     ```bash
     pvlib/solarposition.py
     pvlib/spa.py
     ```

3) **จุดอ้างอิงในโค้ด:** (pvlib 0.10.x–0.11.x)

   - ไฟล์: `spa.py`
     ```bash
     SPA algorithm based on NREL report:
     Reda, I. and Andreas, A. (2008),
     Solar Position Algorithm for Solar Radiation Applications,
     NREL/TP-560-34302
     ```

   - สมการตามเอกสาร:

     | หมวด                          | ตำแหน่งใน pvlib                         | หน้าใน NREL SPA |
     | ----------------------------- | -------------------------------------- | --------------- |
     | การคำนวณ Julian Day           | บรรทัด ~150–200 (`julian_day`)         | หน้า 10–13      |
     | Earth heliocentric coords     | บรรทัด ~260–420 (`earth_heliocentric_longitude`) | หน้า 14–19      |
     | Nutation / Obliquity          | บรรทัด ~450–620                        | หน้า 20–25      |
     | Apparent Sun Longitude        | บรรทัด ~650–720                        | หน้า 26–29      |
     | Right Ascension / Declination | บรรทัด ~730–830                        | หน้า 30–33      |
     | Hour Angle                    | บรรทัด ~840–870                        | หน้า 34         |
     | Zenith / Elevation            | บรรทัด ~880–960                        | หน้า 35–38      |
     | Atmospheric refraction        | บรรทัด ~960–1020                       | หน้า 39–42      |

     (บรรทัดอาจต่างกันเล็กน้อย)

4) **Input: พิกัด (latitude, longitude) + elevation + time**

   - ตัวอย่าง:
     ```bash
     pos = pvlib.solarposition.get_solarposition(
         time=t,
         latitude=13.75,
         longitude=100.5,
         altitude=5
     )
     ```

   - Output: `zenith, elevation, azimuth, equation of time, hour angle, apparent elevation, apparent zenith`

5) **หน้า + สมการ:** (จากเอกสาร SPA)

   | หัวข้อ                            | หน้า | สมการ     |
   | --------------------------------- | ---- | --------- |
   | Julian day                        | 12   | Eq. 1–6   |
   | Geometric heliocentric longitude  | 15   | Eq. 10    |
   | Nutation in longitude & obliquity | 21   | Eq. 30–37 |
   | True obliquity                    | 24   | Eq. 38–40 |
   | Apparent longitude                | 27   | Eq. 47    |
   | Right ascension & declination     | 31   | Eq. 53–56 |
   | Local hour angle                  | 34   | Eq. 57    |
   | Topocentric zenith                | 37   | Eq. 60    |
   | Atmospheric refraction            | 41   | Eq. 64–65 |
