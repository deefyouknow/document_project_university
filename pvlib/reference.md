# **เอกสารบอกการทำงานของ Algorithm ของ Library Pvlib**

> **จะอธิบายการทำงานของ Algorithm ที่ใช้ในการคำนวณพลังงานแสงอาทิตย์**

## หัวข้อที่ 1

### ส่วนที่ใช้ในการทำดาต้าจำลองที่ 1 

> ### **1. Solar Position Calculation**
- Function: location.get_solarposition() → **เรียก** pvlib.solarposition.get_solarposition() 
- **Algo**rithm **หลัก** SPA (Solar Position Algorithm, NREL 2008):  
คำนวณ zenith, azimuth, elevation **โดยใช้** เวลา, พิกัด, ความสูงของพื้นผิว,  
ΔT (difference between UT and TT)  
SolarPosition reference[{1}](https://pvlib-python.readthedocs.io/en/stable/reference/solarposition.html) [{2}](https://assessingsolar.org/notebooks/solar_position.html)
- ผลลัพธ์:  
    - Zenith angle (θz)
    - Azimuth angle (φ)
    - Elevation angle (α = 90° − θz)
    
> ### **2. Clear-Sky Irradiance Models**
- Function: **location.get_clearsky(times, model='ineichen')**
- Algorithm หลัก:
    - Ineichen & Perez model (2002): 
        - ใช้ **Linke turbidity factor** + **extraterrestrial irradiance** เพื่อประมาณค่า GHI, DNI, DHI ภายใต้ท้องฟ้าใส .ref[{3}](https://pvlib-python.readthedocs.io/en/stable/user_guide/modeling_topics/clearsky.html)
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
- สูตรทั่วไป: $\text{I}_{\text{POA}} = \text{I}_{\text{beam}} + \text{I}_{\text{sky\ diffuse}} + \text{I}_{\text{ground\ reflected}}$
