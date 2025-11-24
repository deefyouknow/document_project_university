# **เอกสารบอกการทำงานของ Algorithm ที่ใช้ในการจำลองข้อมูลเพื่อเป็น Feature**

> **จะอธิบายการทำงานของ Algorithm ที่ใช้ในการคำนวณพลังงานแสงอาทิตย์**

## หัวข้อที่ 1

### ส่วนที่ใช้ในการทำดาต้าจำลองที่ 1 

> ### **1. Solar Position Calculation**
- Function: location.get_solarposition() → **เรียก** pvlib.solarposition.get_solarposition() 
- **Algo**rithm **หลัก** SPA (Solar Position Algorithm, NREL 2008):  
คำนวณ zenith, azimuth, elevation **โดยใช้** เวลา, พิกัด, ความสูงของพื้นผิว,  
ΔT (difference between UT and TT). SPA Algorithm ถูกพัฒนาโดย NREL [1].
SolarPosition reference[{1}](https://pvlib-python.readthedocs.io/en/stable/reference/solarposition.html) [{2}](https://assessingsolar.org/notebooks/solar_position.html)
- ผลลัพธ์:  
    - Zenith angle (θz)
    - Azimuth angle (φ)
    - Elevation angle (α = 90° − θz)

> ### **2. Clear-Sky Irradiance Models**
- Function: **location.get_clearsky(times, model='ineichen')**
- Algorithm หลัก:
    - Ineichen & Perez model (2002): 
        - ใช้ **Linke turbidity factor** + **extraterrestrial irradiance** เพื่อประมาณค่า GHI, DNI, DHI ภายใต้ท้องฟ้าใส [2].ref[{3}](https://pvlib-python.readthedocs.io/en/stable/user_guide/modeling_topics/clearsky.html)
    - ตัวเลือกอื่น: 
        - Simplified Solis, Haenel, Bird model ไม่ได้ใช้
- ผลลัพธ์:
    - GHI (Global Horizontal Irradiance) คือ รวมพลังงานแสงอาทิตย์ที่สะท้อนและสะท้อนกลับไปในท้องฟ้า  
    สูตร [GHI = GHI0 * exp(-k * d)](https://en.wikipedia.org/wiki/Global_horizontal_irradiance)
    - DNI (Direct Normal Irradiance) คือ พลังงานแสงอาทิตย์ที่สะท้อนตรงไปที่พื้นผิว  
    สูตร [DNI = DNI0 * exp(-k * d)](https://en.wikipedia.org/wiki/Direct_normal_irradiance)
    - DHI (Diffuse Horizontal Irradiance) คือ พลังงานแสงอาทิตย์ที่สะท้อนและสะท้อนกลับไปในท้องฟ้า  
    สูตร [DHI = DHI0 * exp(-k * d)](https://en.wikipedia.org/wiki/Diffuse_horizontal_irradiance)

> ### **3. Irradiance Transposition (POA Irradiance)**
- **Function:** pvlib.irradiance.get_total_irradiance()
- สูตรทั่วไป: ถูกต้อง
    -  I_{POA} = I_{beam} + I_{sky\ diffuse} + I_{ground\ reflected}
    $I_{beam}$: รังสีตรง (Beam Irradiance)
    $I_{sky\ diffuse}$: รังสีกระจายจากท้องฟ้า (Sky Diffuse Irradiance)
    $I_{ground\ reflected}$: รังสีสะท้อนจากพื้นดิน (Ground Reflected Irradiance)

- โมเดล diffuse sky ที่รองรับ: [pvlib.irradiance.get_total_irradiance](https://pvlib-python.readthedocs.io/en/stable/reference/generated/pvlib.irradiance.get_total_irradiance.html)
    - **Isotropic** (ค่าเฉลี่ยกระจายเท่ากันทุกทิศ)
    - **Hay-Davies:** คำนึงถึง anisotropy ของท้องฟ้า โดยใช้ดัชนีความสว่าง (Anisotropy Index) เพื่อปรับปรุงการประเมินรังสีกระจายจากท้องฟ้า
    - **Perez**: เป็นโมเดลที่ซับซ้อนมากขึ้น โดยพิจารณาถึงองค์ประกอบต่างๆ เช่น circumsolar diffuse และ horizon brightening เพื่อให้การประเมินรังสีกระจายจากท้องฟ้าแม่นยำยิ่งขึ้น [3].
    - **Klucher, Reindl, King:** เป็นโมเดลอื่นๆ ที่ใช้ในการประเมินรังสีกระจายจากท้องฟ้า โดยแต่ละโมเดลมีสมมติฐานและวิธีการคำนวณที่แตกต่างกันไป
- ผลลัพธ์:
    - POA Global Irradiance (ใช้คูณกับ luminous efficacy → Lux)

### **4. Conversion to Lux**
คุณใช้ค่า Luminous Efficacy ≈ 115 lm/W

สูตร: Lux = Irradiance_{POA} × 115

- **สรุปสำหรับงานวิจัย:**
  - **Input:** timestamp, latitude, longitude, elevation
  - **Algorithm chain:**
    - Solar Position Algorithm (SPA, NREL) → Zenith, Azimuth, Elevation
    - Clear-Sky Model (Ineichen & Perez) → GHI, DNI, DHI
    - Irradiance Transposition (Isotropic/Perez) → POA irradiance
    - Lux Conversion (Luminous efficacy factor) → Lux (สำหรับเปรียบเทียบกับ BH1750)


**บรรณานุกรม**
[1]Reda & Andreas (2008), Solar Position Algorithm for Solar Radiation Applications (NREL Report)

[2]Ineichen & Perez (2002), A new airmass independent formulation for the Linke turbidity coefficient

[3]Perez et al. (1990), A new simplified version of the Perez diffuse irradiance model
