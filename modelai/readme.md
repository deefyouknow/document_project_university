# Model AI ที่คัดเลือกยังไม่ตัดสินใจ 

- Long short-term memory (LSTM): เหมาะกับข้อมูลลำดับที่ยาว แต่ซับซ้อนและใช้เวลาในการเทรนนาน
- GRU (Gated Recurrent Unit): คล้าย LSTM แต่โครงสร้างเรียบง่ายกว่า ทำให้เทรนเร็วกว่า แต่บางครั้งอาจให้ผลลัพธ์ด้อยกว่า LSTM ในข้อมูลที่ซับซ้อนมาก
- Random Forest Model: ใช้งานง่าย, ตีความได้ง่าย แต่ไม่เหมาะกับข้อมูลที่เป็นลำดับ หรือข้อมูลที่มีความสัมพันธ์เชิงเวลา
- XGBoost:
    - จุดเด่น: แม่นยำมากโดยเฉพาะกับข้อมูลที่มีปริมาณน้อย และมีประสิทธิภาพสูง
    - จุดด้อย: อาจเกิด overfitting หากปรับแต่งพารามิเตอร์ไม่ดี

# **!อันนี้คือสิ่งที่ต้องการในโปรเจคนี้**
**input: **
  **Sensor:**
  - LUXAGLE_1: INT
  - LUXAGLE_2: INT
  - LUXAGLE_3: INT
  - LUXAGLE_4: INT
  - SolarPower: INT
  - Data: DATETIME

**Output:** Sensor -> add to table sql

**Processing:**
  - AverageAllluxBestDegree: INT ! รับค่าเฉลี่ยของ LUXAGLE_1, LUXAGLE_2, LUXAGLE_3, LUXAGLE_4 ว่าระหว่างจุดไหนมีค่ามากสุด 
  - Activate: TINYINT ! เป็นการระบุว่าจะไปที่ตำแหน่ง AverageAllluxBestDegree หรือไม่การตัดสินใจนี้ได้มาจากการประเมินค่าความเชื่อมั่นของโมเดล

**Result:**
  {
    result: {
      PredictedValue: 159 degree,
      Activate: 1
    }
  }
