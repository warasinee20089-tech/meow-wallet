import streamlit as st
import pandas as pd
import sqlite3
import plotly.express as px
from datetime import datetime

# --- 1. การตั้งค่าระบบ (System Config) ---
# ต้องอยู่บรรทัดแรกสุดของโค้ด ห้ามย้าย!
st.set_page_config(
    page_title="Meow Wallet Stable",
    layout="wide",
    page_icon="🐾"
)

# --- 2. ฟังก์ชันจัดการฐานข้อมูลแบบปลอดภัย (Safe Database Handling) ---
DB_NAME = 'meow_wallet_stable.db'

def run_query(query, params=(), fetch=False):
    """
    ฟังก์ชันกลางสำหรับสั่งงาน Database
    ช่วยป้องกัน Error แบบ Database Locked ที่มักเกิดใน Streamlit
    """
    try:
        conn = sqlite3.connect(DB_NAME, check_same_thread=False)
        c = conn.cursor()
        c.execute(query, params)
        
        if fetch:
            data = c.fetchall()
            columns = [description[0] for description in c.description]
            result = pd.DataFrame(data, columns=columns)
        else:
            conn.commit()
            result = None
            
        conn.close()
        return result
    except Exception as e:
        st.error(f"เกิดข้อผิดพลาดกับฐานข้อมูล: {e}")
        return pd.DataFrame() if fetch else None

def init_db():
    """สร้างตารางเก็บข้อมูลถ้ายังไม่มี"""
    create_table_sql = """
    CREATE TABLE IF NOT EXISTS transactions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        date TEXT,
        type TEXT,
        category TEXT,
        amount REAL,
        note TEXT
    )
    """
    run_query(create_table_sql)

# เริ่มต้นระบบฐานข้อมูลทันที
init_db()

# --- 3. การตกแต่งหน้าตา (CSS Styling) ---
st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500&display=swap');
    
    html, body, [class*="css"], .stMarkdown {
        font-family: 'Kanit', sans-serif !important;
    }
    .stApp { background-color: #FFF0F5 !important; } /* สีชมพูพาสเทล */
    
    /* ปรับแต่งการ์ดตัวเลข */
    div[data-testid="stMetric"] {
        background-color: white;
        border-radius: 12px;
        padding: 10px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        border: 1px solid #FFB6C1;
    }
    
    .stButton>button {
        background-color: #FF69B4;
        color: white;
        border-radius: 20px;
        border: none;
        padding: 10px 20px;
        font-weight: bold;
    }
    .stButton>button:hover {
        background-color: #FF1493;
        color: white;
    }
    </style>
""", unsafe_allow_html=True)

# --- 4. ส่วน Sidebar (เมนูข้าง) ---
with st.sidebar:
    st.image("https://cdn-icons-png.flaticon.com/512/616/616430.png", width=80)
    st.title("⚙️ ตั้งค่า")
    budget = st.number_input("🎯 ตั้งงบรายจ่ายเดือนนี้", value=5000, step=500)
    st.info("💡 ทิป: ลองบันทึกรายรับ-รายจ่าย แล้วดูน้องแมวเปลี่ยนอารมณ์นะ!")

# --- 5. ดึงข้อมูลมาแสดงผล (Data Fetching) ---
try:
    df = run_query("SELECT * FROM transactions", fetch=True)
except Exception:
    df = pd.DataFrame() # กันเหนียวถ้าดึงข้อมูลพลาด

# คำนวณยอดเงิน
if not df.empty:
    total_inc = df[df['type'] == 'รายรับ']['amount'].sum()
    total_exp = df[df['type'] == 'รายจ่าย']['amount'].sum()
    balance = total_inc - total_exp
else:
    total_inc, total_exp, balance = 0, 0, 0

# --- 6. ส่วนแสดงผลหลัก (Dashboard) ---
st.title("🐾 Meow Wallet: ระบบบัญชีทาสแมว")

# Layout: การ์ดตัวเลข + น้องแมว
col_metrics, col_cat = st.columns([3, 1])

with col_metrics:
    c1, c2, c3 = st.columns(3)
    c1.metric("💰 รายรับรวม", f"{total_inc:,.2f} ฿")
    c2.metric("💸 รายจ่ายรวม", f"{total_exp:,.2f} ฿")
    c3.metric("🐷 ยอดคงเหลือ", f"{balance:,.2f} ฿")
    
    # Logic แจ้งเตือนงบประมาณ
    if total_exp > budget:
        st.error(f"⚠️ แย่แล้ว! ใช้เงินเกินงบไป {total_exp - budget:,.2f} บาท")
    elif total_exp > (budget * 0.8):
        st.warning(f"⚠️ ระวัง! ใช้ไป 80% ของงบแล้ว")
    else:
        st.success("✅ สุขภาพการเงินแข็งแรงดีเยี่ยม")

with col_cat:
    # Logic น้องแมวแสดงอารมณ์
    if total_exp > budget:
        st.image("https://cdn-icons-png.flaticon.com/512/1865/1865089.png", caption="ถังแตกแล้ว!")
    elif balance > 3000:
        st.image("https://cdn-icons-png.flaticon.com/512/616/616554.png", caption="รวยเวอร์!")
    else:
        st.image("https://cdn-icons-png.flaticon.com/512/616/616430.png", caption="สวัสดีเมี๊ยว")

st.markdown("---")

# --- 7. Tabs แบ่งการทำงาน ---
tab1, tab2, tab3 = st.tabs(["📝 บันทึกรายการ", "📊 วิเคราะห์กราฟ", "💾 จัดการข้อมูล"])

# === Tab 1: บันทึกข้อมูล ===
with tab1:
    with st.form("entry_form", clear_on_submit=True):
        col_d, col_t = st.columns(2)
        date = col_d.date_input("วันที่", datetime.now())
        tx_type = col_t.radio("ประเภท", ["รายจ่าย", "รายรับ"], horizontal=True)
        
        category = st.selectbox("หมวดหมู่", 
            ["อาหาร 🍜", "เดินทาง 🚗", "ของใช้ 🛍️", "ค่าหอ/น้ำไฟ 🏠", "เงินเดือน 💵", "เสี่ยงโชค 🎰", "อื่นๆ ✨"])
        amount = st.number_input("จำนวนเงิน (บาท)", min_value=0.0, step=10.0)
        note = st.text_input("บันทึกช่วยจำ")
        
        submitted = st.form_submit_button("บันทึกข้อมูล ✅")
        
        if submitted:
            if amount > 0:
                run_query(
                    "INSERT INTO transactions (date, type, category, amount, note) VALUES (?,?,?,?,?)",
                    (date, tx_type, category, amount, note)
                )
                st.success("บันทึกเรียบร้อยเมี๊ยว!")
                st.rerun()
            else:
                st.warning("⚠️ กรุณาระบุจำนวนเงินให้ถูกต้อง")

# === Tab 2: วิเคราะห์แนวโน้ม ===
with tab2:
    if not df.empty:
        c_chart1, c_chart2 = st.columns(2)
        
        with c_chart1:
            st.subheader("สัดส่วนรายจ่าย")
            exp_df = df[df['type'] == 'รายจ่าย']
            if not exp_df.empty:
                fig = px.pie(exp_df, values='amount', names='category', hole=0.4,
                             color_discrete_sequence=px.colors.qualitative.Pastel)
                st.plotly_chart(fig, use_container_width=True)
            else:
                st.info("ยังไม่มีข้อมูลรายจ่าย")
                
        with c_chart2:
            st.subheader("แนวโน้มการใช้เงิน (รายวัน)")
            df['date_obj'] = pd.to_datetime(df['date'])
            daily = df[df['type'] == 'รายจ่าย'].groupby('date_obj')['amount'].sum().reset_index()
            if not daily.empty:
                fig_line = px.line(daily, x='date_obj', y='amount', markers=True)
                fig_line.update_traces(line_color='#FF69B4')
                st.plotly_chart(fig_line, use_container_width=True)
            else:
                st.info("บันทึกข้อมูลหลายๆ วันเพื่อดูกราฟเส้น")
    else:
        st.info("เริ่มบันทึกข้อมูลที่ Tab แรกได้เลย")

# === Tab 3: จัดการข้อมูล & Excel ===
with tab3:
    if not df.empty:
        st.dataframe(df.sort_values("id", ascending=False), use_container_width=True)
        
        # ปุ่มโหลด Excel
        csv_data = df.to_csv(index=False).encode('utf-8')
        st.download_button(
            "📥 ดาวน์โหลดไฟล์ CSV",
            data=csv_data,
            file_name='meow_wallet_data.csv',
            mime='text/csv'
        )
        
        st.markdown("### ❌ ลบรายการ")
        del_list = df.apply(lambda x: f"ID:{x['id']} | {x['date']} | {x['category']} {x['amount']}บ.", axis=1)
        target = st.selectbox("เลือกรายการที่จะลบ", del_list)
        
        if st.button("ยืนยันการลบ"):
            if target:
                t_id = target.split("|")[0].replace("ID:", "").strip()
                run_query("DELETE FROM transactions WHERE id=?", (t_id,))
                st.success("ลบข้อมูลแล้ว")
                st.rerun()
    else:
        st.info("ยังไม่มีข้อมูลให้จัดการ")
