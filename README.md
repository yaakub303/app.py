import streamlit as st
import sqlite3
import pandas as pd
from datetime import datetime

# إعدادات الصفحة لتناسب شاشة التلفون الصغير واللابتوب الكبير
st.set_page_config(page_title="نظام التنمية - المنطقة 4", layout="centered")

# كود لتكبير الخط والخانات (CSS) لراحة العين في العمل الميداني
st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Arial&display=swap');
    html, body, [class*="css"] { font-family: 'Arial', sans-serif !important; direction: rtl; text-align: right; }
    .stTextInput input, .stSelectbox div, .stTextArea textarea { font-size: 1.2rem !important; padding: 10px !important; }
    label { font-size: 1.3rem !important; font-weight: bold !important; color: #1a237e !important; }
    .stButton>button { width: 100%; border-radius: 12px; height: 3.5em; background-color: #27ae60; color: white; font-size: 1.4rem !important; font-weight: bold; }
    </style>
    """, unsafe_allow_html=True)

# قاعدة البيانات (SQLite)
def get_db():
    conn = sqlite3.connect('social_data_v2.db', check_same_thread=False)
    return conn

conn = get_db()
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS households (
    id INTEGER PRIMARY KEY AUTOINCREMENT, date TEXT, town TEXT, 
    name TEXT, phone TEXT, income TEXT, health TEXT, needs TEXT)''')
conn.commit()

# واجهة التطبيق
st.title("📋 استمارة المنطقة الرابعة")
st.subheader("قسم التنمية الاجتماعية")

with st.form("mobile_form", clear_on_submit=True):
    town = st.selectbox("البلدة", ["تمنين الفوقا", "تمنين التحتا", "بدنايل", "قصرنبا", "النبي ايلا", "جلالا", "حوش الرافقة"])
    name = st.text_input("الاسم الثلاثي لصاحب الطلب")
    phone = st.text_input("رقم الهاتف (للتواصل)")
    income = st.text_input("الدخل الشهري التقريبي")
    health = st.multiselect("الأمراض المزمنة", ["سكري", "ضغط", "قلب", "ربو", "أعصاب", "كلى"])
    needs = st.text_area("أولويات الدعم (ماذا يحتاجون؟)")
    
    submitted = st.form_submit_button("💾 حفظ البيانات الآن")
    
    if submitted:
        if name and phone:
            c.execute("INSERT INTO households (date, town, name, phone, income, health, needs) VALUES (?,?,?,?,?,?,?)",
                      (datetime.now().strftime("%Y-%m-%d"), town, name, phone, income, str(health), needs))
            conn.commit()
            st.success(f"✅ تم الحفظ! الله يعطيك العافية.")
        else:
            st.error("⚠️ خيي أحمد، لازم تعبي الاسم والتلفون عالأقل.")

st.divider()
if st.checkbox("🔍 عرض البيانات المخزنة وتنزيل Excel"):
    df = pd.read_sql_query("SELECT * FROM households", conn)
    st.dataframe(df)
    csv = df.to_csv(index=False).encode('utf-8-sig')
    st.download_button("📥 تحميل ملف الإكسل", data=csv, file_name="report_area4.csv", mime="text/csv")
