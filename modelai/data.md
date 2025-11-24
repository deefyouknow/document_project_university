# หน้านี้เป็นข้อมูลที่ใช้ในการฝึกฝนโมเดล AI

ตารางด้านล่างแสดงตัวอย่างข้อมูลที่สร้างขึ้นจากการรันโค้ด ข้อมูลนี้สามารถนำไปใช้ในการฝึกฝนโมเดล AI เพื่อทำนายค่าต่างๆ เช่น ปริมาณแสง (Lux) จากข้อมูลสภาพอากาศและตำแหน่ง

| ts                  | apparent_zenith | apparent_elevation | azimuth | latitude | longitude | elevation | ghi | dni | dhi | weather_source | lux_one | lux_two | lux_three | lux_four |
|-----------------------|-----------------|--------------------|---------|----------|-----------|-----------|-----|-----|-----|----------------|---------|---------|-----------|----------|
| 2024-10-27 10:00:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |
| 2024-10-27 10:01:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |
| 2024-10-27 10:02:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |
| 2024-10-27 10:03:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |
| 2024-10-27 10:04:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |
| 2024-10-27 10:05:00+07:00 | 45              | 45                 | 180     | 13.75    | 100.5     | 50        | 500 | 800 | 200 | TMY            | 10000   | 12000   | 11000     | 9000     |

```python
import pymysql
import pvlib
import pandas as pd
import time
from datetime import datetime
import numpy as np

# ค่าประมาณการแปลง W/m^2 เป็น Lux 
LUMINOUS_EFFICACY = 115.0 

# Database Configuration (แทนที่ด้วยข้อมูลจริงของคุณ)
DB_HOST = "your_db_host"
DB_USER = "your_db_user"
DB_PASSWORD = "your_db_password"
DB_NAME = "your_db_name"
DB_PORT = 3306

# -----------------------------------------------------------
# 1) MySQL connection
# -----------------------------------------------------------
def get_connection():
    return pymysql.connect(
        host=DB_HOST,
        user=DB_USER,
        password=DB_PASSWORD,
        database=DB_NAME,
        port=DB_PORT,
        cursorclass=pymysql.cursors.DictCursor,
        autocommit=True
    )

# -----------------------------------------------------------
# 2) อ่าน input
# -----------------------------------------------------------
def get_latest_input(conn):
    with conn.cursor() as cur:
        cur.execute("""
            SELECT id, latitude, longitude, elevation,
                   degree_angle_one, degree_angle_two, 
                   degree_angle_three, degree_angle_four
            FROM location
            ORDER BY id DESC
            LIMIT 1;
        """)
        return cur.fetchone()

# -----------------------------------------------------------
# 3) สร้างช่วงเวลา
# -----------------------------------------------------------
def generate_times_for_year(reference_date, tz="Asia/Bangkok"):
    start = reference_date - pd.Timedelta(days=365)
    end   = reference_date + pd.Timedelta(days=365)
    return pd.date_range(start=start, end=end - pd.Timedelta(minutes=1), freq="1min", tz=tz)

# -----------------------------------------------------------
# 4) ดึงข้อมูลสภาพอากาศ TMY (แก้ไข: รองรับ pvlib ทุกเวอร์ชัน)
# ------------------------------------------------ richer
def get_weather_data(lat, lon, target_times):
    print(f"Fetching TMY weather data for Lat: {lat}, Lon: {lon}...")
    try:
        # แก้ไข 1: รับค่าใส่ตัวแปรเดียว (result) แล้วดึงตัวแรก (data)
        # ป้องกัน error "expected 4, got 2"
        result = pvlib.iotools.get_pvgis_tmy(lat, lon, map_variables=True)
        tmy_data = result[0] # ตัวแรกคือ DataFrame ข้อมูลเสมอ

        # Resample & Interpolate
        tmy_minute = tmy_data.resample('1min').interpolate()
        tmy_minute['key'] = tmy_minute.index.strftime('%m-%d %H:%M')

        weather_df = pd.DataFrame(index=target_times)
        weather_df['key'] = weather_df.index.strftime('%m-%d %H:%M')

        tmy_lookup = tmy_minute.drop_duplicates(subset=['key'])

        merged = weather_df.merge(tmy_lookup[['key', 'ghi', 'dni', 'dhi']], 
                                  on='key', how='left').set_index(weather_df.index)

        merged = merged.fillna(method='ffill').fillna(method='bfill')
        return merged

    except Exception as e:
        print(f"Warning: Could not fetch weather data ({e}). Using Clear Sky.")
        return None

# -----------------------------------------------------------
# 5) PVLIB compute (แก้ไข: เปลี่ยน dn เป็น dni)
# -----------------------------------------------------------
def compute_solar_and_lux(times, lat, lon, elev, angles):
    site = pvlib.location.Location(lat, lon, tz=str(times.tz), altitude=elev)
    sp = site.get_solarposition(times)

    # ดึง weather
    weather = get_weather_data(lat, lon, times)

    results = sp[['apparent_zenith', 'apparent_elevation', 'azimuth']].copy()

    if weather is not None:
        dni = weather['dni']
        ghi = weather['ghi']
        dhi = weather['dhi']
        source = 'TMY'
        print("Using Historical Weather Data (TMY).")
    else:
        clearsky = site.get_clearsky(times, model='ineichen')
        dni = clearsky['dni']
        ghi = clearsky['ghi']
        dhi = clearsky['dhi']
        source = 'ClearSky'
        print("Using Clear Sky Model (Fallback).")

    # เก็บค่าสภาพอากาศ
    results['ghi'] = ghi.fillna(0).astype(int)
    results['dni'] = dni.fillna(0).astype(int)
    results['dhi'] = dhi.fillna(0).astype(int)
    results['weather_source'] = source

    surface_tilt = 90 
    angle_keys = ['degree_angle_one', 'degree_angle_two', 'degree_angle_three', 'degree_angle_four']
    lux_cols = ['lux_one', 'lux_two', 'lux_three', 'lux_four']

    for i, key in enumerate(angle_keys):
        surface_azimuth = angles.get(key)

        if surface_azimuth is None:
            results[lux_cols[i]] = 0
            continue

        # แก้ไข 2: เปลี่ยน dn=dni เป็น dni=dni
        poa_irrad = pvlib.irradiance.get_total_irradiance(
            surface_tilt=surface_tilt,
            surface_azimuth=surface_azimuth,
            dni=dni,   # <--- แก้ตรงนี้จาก dn เป็น dni
            ghi=ghi,
            dhi=dhi,
            solar_zenith=sp['apparent_zenith'],
            solar_azimuth=sp['azimuth'],
            model='isotropic'
        )

        lux_raw = poa_irrad['poa_global'] * LUMINOUS_EFFICACY
        results[lux_cols[i]] = lux_raw.fillna(0).astype(int)

    return results

# -----------------------------------------------------------
# 6) ล้าง output table
# -----------------------------------------------------------
def clear_output_table(conn):
    with conn.cursor() as cur:
        try:
            cur.execute("TRUNCATE sunposition_and_lux_data;")
        except:
            pass

# -----------------------------------------------------------
# 7) เขียนข้อมูลลง DB
# -----------------------------------------------------------
def write_output(conn, df, lat, lon, elev, batch_size=5000):
    sql = """
        INSERT INTO sunposition_and_lux_data
        (ts, apparent_zenith, apparent_elevation, azimuth,
         latitude, longitude, elevation,
         ghi, dni, dhi, weather_source,
         lux_one, lux_two, lux_three, lux_four)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s);
    """

    df = df.where(pd.notnull(df), None)

    data = []
    for ts, row in df.iterrows():
        data.append((
            ts.to_pydatetime(),
            row['apparent_zenith'],
            row['apparent_elevation'],
            row['azimuth'],
            lat, lon, elev,
            row['ghi'], 
            row['dni'], 
            row['dhi'], 
            row['weather_source'],
            row['lux_one'],
            row['lux_two'],
            row['lux_three'],
            row['lux_four']
        ))

    with conn.cursor() as cur:
        total = len(data)
        for i in range(0, total, batch_size):
            batch = data[i:i+batch_size]
            cur.executemany(sql, batch)
            print(f"[{datetime.now()}] Inserted rows {i+1} - {min(i+batch_size, total)}")

# -----------------------------------------------------------
# 8) Main loop
# -----------------------------------------------------------
def main_loop():
    last_location_id = None
    conn = get_connection()

    try:
        while True:
            row = get_latest_input(conn)

            if row is None:
                print("No input data found. Waiting...")
                time.sleep(10)
                continue

            current_id = row['id']

            if current_id != last_location_id:
                lat = row['latitude']
                lon = row['longitude']
                elev = row['elevation']

                angles = {
                    'degree_angle_one': row['degree_angle_one'],
                    'degree_angle_two': row['degree_angle_two'],
                    'degree_angle_three': row['degree_angle_three'],
                    'degree_angle_four': row['degree_angle_four']
                }

                print(f"[{datetime.now()}] New location ID {current_id} detected.")
                print("Regenerating data with WEATHER INFO...")

                clear_output_table(conn)

                reference_date = pd.Timestamp.now(tz="Asia/Bangkok")
                times = generate_times_for_year(reference_date)

                solar_data = compute_solar_and_lux(times, lat, lon, elev, angles)

                write_output(conn, solar_data, lat, lon, elev)

                last_location_id = current_id
                print(f"[{datetime.now()}] Finished.")

            time.sleep(10)

    except KeyboardInterrupt:
        print("Stopping...")
    except Exception as e:
        print(f"Error: {e}")
    finally:
        conn.close()

if __name__ == "__main__":
    main_loop()
```
