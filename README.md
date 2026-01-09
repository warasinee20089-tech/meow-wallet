import streamlit as st
import pandas as pd
import sqlite3
import plotly.express as px
from datetime import datetime

# --- 1. ตั้งค่าหน้าเว็บ (Page Config) ---
st.set_page_config(
    page_title="Meow Wallet Pro",
    page_icon="🐾",
    layout="wide"  # ใช้โหมดหน้าจอกว้าง
)

# --- 2. CSS ตกแต่งธีมแมว (Custom CSS) ---
st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500&display=swap');
    
    html, body, [class*="css"] {
        font-family: 'Kanit', sans-serif;
    }
    .stApp {
        background-color: #FFF5F7; /* พื้นหลังสีชมพูอ่อน */
    }
    .metric-card {
        background-color: #FFFFFF;
        border-radius: 15px;
        padding: 20px;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        text-align: center;
    }
    h1, h2, h3 {
        color: #FF69B4; /* สีหัวข้อชมพูเข้ม */
    }
    </style>
""", unsafe_allow_html=True)

# --- 3. ฟังก์ชันจัดการ Database (ให้เสถียรขึ้น) ---
def get_connection():
    # check_same_thread=False ช่วยแก้ปัญหา Error ใน Streamlit Cloud
    conn = sqlite3.connect('meow_wallet_pro.db', check_same_thread=False)
    return conn

def init_db():
    conn = get_connection()
    c = conn.cursor()
    c.execute("""
        CREATE TABLE IF NOT EXISTS transactions (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            date TEXT,
            type TEXT,
            category TEXT,
            amount REAL,
            note TEXT
        )
    """)
    conn.commit()
    conn.close()

# เรียกใช้ฟังก์ชันสร้างตารางทันทีที่รัน
init_db()

# --- 4. ส่วนหัวของแอป ---
st.title("🐱 Meow Wallet: กระเป๋าตังค์แมวเหมียว")
st.markdown("ระบบบันทึกรายรับ-รายจ่ายที่น่ารักที่สุดในโลก!")

# --- 5. การคำนวณยอดเงิน (Metrics) ---
conn = get_connection()
df = pd.read_sql_query("SELECT * FROM transactions", conn)
conn.close()

if not df.empty:
    total_income = df[df['type'] == 'รายรับ']['amount'].sum()
    total_expense = df[df['type'] == 'รายจ่าย']['amount'].sum()
    balance = total_income - total_expense
else:
    total_income, total_expense, balance = 0, 0, 0

# แสดงผลแบบ 3 คอลัมน์ (การ์ดตัวเลข)
col1, col2, col3 = st.columns(3)
col1.metric("💰 รายรับทั้งหมด", f"{total_income:,.2f} บาท")
col2.metric("💸 รายจ่ายทั้งหมด", f"{total_expense:,.2f} บาท")
col3.metric("🐷 คงเหลือสุทธิ", f"{balance:,.2f} บาท", delta=f"{balance:,.2f} บาท")

st.markdown("---")

# --- 6. สร้าง Tabs เพื่อแบ่งหน้าจอ (สำคัญมากสำหรับ UI สมัยใหม่) ---
tab1, tab2, tab3 = st.tabs(["📝 บันทึกรายการ", "📊 วิเคราะห์กราฟ", "✏️ แก้ไข/ลบข้อมูล"])

# === TAB 1: บันทึกข้อมูล ===
with tab1:
    st.subheader("เพิ่มรายการใหม่")
    with st.form("transaction_form", clear_on_submit=True):
        col_date, col_type = st.columns(2)
        date = col_date.date_input("วันที่", datetime.now())
        tx_type = col_type.radio("ประเภท", ["รายจ่าย", "รายรับ"], horizontal=True)
        
        category = st.selectbox("หมวดหมู่", 
            ["อาหาร 🍜", "เดินทาง 🚗", "ช้อปปิ้ง 🛍️", "ค่าบ้าน/น้ำไฟ 🏠", "เงินเดือน 💵", "โบนัส 🎁", "อื่นๆ ✨"])
        
        amount = st.number_input("จำนวนเงิน", min_value=0.0, step=10.0)
        note = st.text_input("หมายเหตุ (เช่น ข้าวมันไก่)")
        
        submitted = st.form_submit_button("บันทึกข้อมูล ✅")
        
        if submitted:
            conn = get_connection()
            c = conn.cursor()
            c.execute("INSERT INTO transactions (date, type, category, amount, note) VALUES (?, ?, ?, ?, ?)",
                      (date, tx_type, category, amount, note))
            conn.commit()
            conn.close()
            st.success("บันทึกเรียบร้อยเมี๊ยว! 🐾")
            st.rerun() # รีเฟรชหน้าทันที

# === TAB 2: วิเคราะห์กราฟ ===
with tab2:
    st.subheader("ภาพรวมการเงิน")
    if not df.empty:
        col_chart1, col_chart2 = st.columns(2)
        
        # กราฟวงกลม (Donut Chart)
        with col_chart1:
            st.write("##### สัดส่วนรายจ่าย")
            exp_df = df[df['type'] == 'รายจ่าย']
            if not exp_df.empty:
                fig = px.pie(exp_df, values='amount', names='category', hole=0.4,
                             color_discrete_sequence=px.colors.qualitative.Pastel)
                st.plotly_chart(fig, use_container_width=True)
            else:
                st.info("ยังไม่มีรายจ่าย")
        
        # กราฟแท่งรายวัน
        with col_chart2:
            st.write("##### การใช้จ่ายรายวัน")
            daily_df = df.groupby('date')['amount'].sum().reset_index()
            if not daily_df.empty:
                fig2 = px.bar(daily_df, x='date', y='amount', color_discrete_sequence=['#FF69B4'])
                st.plotly_chart(fig2, use_container_width=True)
            else:
                st.info("ยังไม่มีข้อมูล")
    else:
        st.info("กรุณาบันทึกข้อมูลก่อนเพื่อดูการวิเคราะห์")

# === TAB 3: แก้ไขและลบข้อมูล (Feature ใหม่!) ===
with tab3:
    st.subheader("จัดการประวัติรายการ")
    if not df.empty:
        # แสดงตารางแบบโต้ตอบได้
        st.dataframe(df.sort_values(by="date", ascending=False), use_container_width=True)
        
        st.write("##### 🗑️ ลบรายการที่ผิดพลาด")
        # ดึงรายชื่อมาให้เลือก
        tx_options = df.apply(lambda x: f"ID: {x['id']} | {x['date']} | {x['category']} | {x['amount']} บาท", axis=1)
        selected_tx = st.selectbox("เลือกรายการที่จะลบ", tx_options)
        
        if st.button("ลบรายการที่เลือก ❌"):
            # ดึง ID จากข้อความที่เลือก
            tx_id = selected_tx.split("|")[0].replace("ID:", "").strip()
            
            conn = get_connection()
            c = conn.cursor()
            c.execute("DELETE FROM transactions WHERE id = ?", (tx_id,))
            conn.commit()
            conn.close()
            st.success("ลบข้อมูลเรียบร้อย!")
            st.rerun()
    else:
        st.info("ยังไม่มีรายการให้จัดการ")
