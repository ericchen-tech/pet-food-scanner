import streamlit as st
import google.generativeai as genai
import easyocr
import re
import json
import numpy as np
from PIL import Image

st.set_page_config(page_title="寵物營養掃描儀", page_icon="🐾")

st.title("🐾 寵物食品成分掃描儀")
st.caption("拍照成分表，自動計算乾物比與餵食量")

# 側邊欄設定
with st.sidebar:
    st.header("⚙️ 設定")
    api_key = st.text_input("輸入 Gemini API Key", type="password")
    weight = st.number_input("寵物體重 (kg)", value=5.0, step=0.5)
    factor = st.select_slider("寵物需求因子", 
                             options=[0.6, 0.8, 1.0, 1.2, 1.4, 1.6], 
                             value=1.2,
                             help="0.8: 減肥中, 1.0: 低活動量, 1.2: 一般結紮成年, 1.6: 高活動量")

# 拍照功能
img_file = st.camera_input("請對準包裝上的成分表拍照")

if img_file and not api_key:
    st.error("請在側邊欄輸入 API Key 才能進行 AI 分析喔！")

if img_file and api_key:
    # 初始化 AI
    genai.configure(api_key=api_key)
    model = genai.GenerativeModel('gemini-1.5-flash')
    
    with st.spinner("AI 正在努力閱讀中..."):
        # 影像處理
        image = Image.open(img_file)
        img_np = np.array(image)
        
        # OCR 辨識
        reader = easyocr.Reader(['ch_tra', 'en'])
        result = reader.readtext(img_np, detail=0)
        raw_text = " ".join(result)
        
        # 請求 Gemini
        prompt = f"Extract pet food nutrition (protein, fat, fiber, moisture, ash) as JSON from: {raw_text}"
        try:
            response = model.generate_content(prompt)
            match = re.search(r'\{.*\}', response.text, re.DOTALL)
            data = json.loads(match.group())
            
            # --- 你的核心計算公式 ---
            p, f, m, a = data.get('protein', 0), data.get('fat', 0), data.get('moisture', 10), data.get('ash', 8)
            fb = data.get('fiber', 0)
            
            dm_p = p / (1 - m/100)
            nfe = 100 - (p + f + fb + m + a)
            me = 10 * ((3.5 * p) + (8.5 * f) + (3.5 * nfe))
            rer = 70 * (weight ** 0.75)
            daily_g = (rer * factor / me) * 1000
            
            # 顯示漂亮結果
            st.divider()
            col1, col2 = st.columns(2)
            col1.metric("蛋白質乾物比 (DM)", f"{round(dm_p, 1)}%")
            col2.metric("代謝能 (ME)", f"{round(me)} kcal")
            
            st.success(f"✅ 建議餵食量：每日 **{round(daily_g, 1)}** 公克")
            
        except Exception as e:
            st.error(f"分析失敗，請確認照片清晰度。錯誤：{e}")
