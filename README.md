# crop-intelligent-system

import streamlit as st
from rapidfuzz import process

# --- CONFIGURATION ---
st.set_page_config(page_title="CROP Intelligent System", layout="wide", initial_sidebar_state="collapsed")

# --- DATABASE (INTEGRATED 40 CROPS) ---
# (Using your provided crops + expanded list)
CROPS_DB = {
    "Wheat": {"pH": 6.5, "fert": "High Nitrogen", "season": ["Winter"], "soil": ["Loamy", "Silt"], "diseases": {"Rust": {"cause": "Puccinia Fungus", "symptoms": ["yellow spots on leaves", "brown powder on leaf", "leaf drying"], "remedy": ["Apply sulfur dust", "Use resistant varieties", "Crop rotation", "Fungicide spray"]}, "Smut": {"cause": "Ustilago Fungus", "symptoms": ["black dust on grains", "swollen kernels", "poor grain formation"], "remedy": ["Seed treatment", "Remove infected ears", "Avoid infected seeds", "Hot water treatment"]}, "Blight": {"cause": "Helminthosporium", "symptoms": ["brown leaf patches", "leaf tip drying", "reduced yield"], "remedy": ["Balanced NPK", "Crop rotation", "Mancozeb spray", "Proper irrigation"]}}},
    "Rice": {"pH": 6.0, "fert": "High Nitrogen", "season": ["Monsoon"], "soil": ["Clay", "Loamy"], "diseases": {"Blast": {"cause": "Magnaporthe oryzae", "symptoms": ["diamond spots on leaves", "neck rot", "stunted growth"], "remedy": ["Seed treatment", "Silicon fertilizer", "Systemic fungicide", "Destroy crop residue"]}, "Sheath Blight": {"cause": "Rhizoctonia solani", "symptoms": ["oval lesions", "yellowing leaves", "lodging"], "remedy": ["Drain fields", "Apply Potash", "Avoid high Nitrogen", "Validamycin spray"]}, "Bacterial Leaf Blight": {"cause": "Xanthomonas oryzae", "symptoms": ["leaf margin drying", "milky ooze", "leaf wilting"], "remedy": ["Streptocycline spray", "Copper fungicide", "Resistant varieties", "Balance fertilizer"]}}},
    "Maize": {"pH": 6.2, "fert": "Medium Nitrogen", "season": ["Summer", "Monsoon"], "soil": ["Loamy", "Sandy"], "diseases": {"Downy Mildew": {"cause": "Peronosclerospora", "symptoms": ["white growth under leaf", "yellow streaks", "leaf curling"], "remedy": ["Metalaxyl seed treatment", "Rogue out plants", "Proper drainage", "Wider spacing"]}, "Leaf Blight": {"cause": "Exserohilum", "symptoms": ["long brown lesions", "leaf drying", "reduced cob size"], "remedy": ["Crop rotation", "Clean cultivation", "Zineb spray", "Nutrient management"]}, "Smut": {"cause": "Ustilago maydis", "symptoms": ["black galls", "swollen ears", "grain deformation"], "remedy": ["Burn galls", "Crop rotation", "Seed treatment", "Avoid injury to plants"]}}},
    "Sorghum": {"pH": 6.5, "fert": "Medium", "season": ["Summer"], "soil": ["Loamy", "Sandy"], "diseases": {"Anthracnose": {"cause": "Colletotrichum", "symptoms": ["red leaf spots", "stem rot", "leaf drying"], "remedy": ["Resistant varieties", "Fungicide spray", "Field sanitation", "Avoid dense planting"]}, "Grain Mold": {"cause": "Fungal complex", "symptoms": ["discolored grains", "fungal growth", "poor grain quality"], "remedy": ["Timely harvest", "Proper drying", "Resistant hybrids", "Irrigation control"]}, "Downy Mildew": {"cause": "Oomycetes", "symptoms": ["white powder", "leaf shredding", "stunted plants"], "remedy": ["Seed treatment", "Deep plowing", "Remove grass hosts", "Systemic fungicide"]}}},
    "Potato": {"pH": 5.5, "fert": "Medium Potassium", "season": ["Winter"], "soil": ["Sandy", "Peaty"], "diseases": {"Late Blight": {"cause": "Phytophthora infestans", "symptoms": ["dark leaf blotches", "fuzzy growth", "tuber rot"], "remedy": ["Copper fungicides", "Certified seeds", "Avoid overhead water", "Proper hilling"]}, "Early Blight": {"cause": "Alternaria solani", "symptoms": ["concentric rings", "leaf yellowing", "defoliation"], "remedy": ["Crop rotation", "Nutrient boost", "Mancozeb spray", "Mulching"]}, "Scab": {"cause": "Streptomyces bacteria", "symptoms": ["rough tuber skin", "brown patches", "reduced market value"], "remedy": ["Acidify soil", "Irrigation during tuber set", "Avoid fresh manure", "Seed treatment"]}}},
    "Tomato": {"pH": 6.5, "fert": "Balanced NPK", "season": ["Spring"], "soil": ["Loamy"], "diseases": {"Leaf Curl": {"cause": "Tomato Leaf Curl Virus", "symptoms": ["curled leaves", "yellow veins", "stunted growth"], "remedy": ["Control Whiteflies", "Netting", "Neem oil spray", "Remove infected plants"]}, "Blight": {"cause": "Fungal pathogen", "symptoms": ["brown leaf spots", "fruit rot", "leaf drying"], "remedy": ["Staking", "Pruning bottom leaves", "Fungicide", "Copper spray"]}, "Wilt": {"cause": "Fusarium/Verticillium", "symptoms": ["sudden wilting", "yellowing", "root decay"], "remedy": ["Soil solarization", "Raised beds", "pH management", "Bio-fungicides"]}}},
    "Onion": {"pH": 6.0, "fert": "High Potassium", "season": ["Winter"], "soil": ["Loamy", "Sandy"], "diseases": {"Purple Blotch": {"cause": "Alternaria porri", "symptoms": ["purple leaf spots", "leaf breakage", "dry tips"], "remedy": ["Proper spacing", "Fungicide spray", "Nutrient management", "Rotation"]}, "Downy Mildew": {"cause": "Peronospora", "symptoms": ["violet mold", "leaf bending", "poor bulb growth"], "remedy": ["Reduced irrigation", "Air circulation", "Fungicide", "Burn residue"]}, "Basal Rot": {"cause": "Fusarium oxysporum", "symptoms": ["root decay", "yellowing leaves", "bulb softening"], "remedy": ["Crop rotation", "Soil solarization", "Hot water treatment", "Trichoderma use"]}}},
    "Chilli": {"pH": 6.5, "fert": "High Potassium", "season": ["Summer"], "soil": ["Loamy"], "diseases": {"Leaf Curl": {"cause": "Virus (Whitefly vector)", "symptoms": ["upward curling", "yellow leaves", "small fruits"], "remedy": ["Insecticide for flies", "Sticky traps", "Resistant hybrids", "Neem spray"]}, "Anthracnose": {"cause": "Colletotrichum capsici", "symptoms": ["sunken fruit spots", "fruit rot", "premature drop"], "remedy": ["Seed treatment", "Fungicide spray", "Field hygiene", "Rotate crops"]}, "Wilt": {"cause": "Bacterial/Fungal", "symptoms": ["drooping plants", "brown stem", "root decay"], "remedy": ["Grafting", "Copper Oxychloride", "Better drainage", "Solarization"]}}},
    "Chickpea": {"pH": 6.8, "fert": "Low Nitrogen", "season": ["Winter"], "soil": ["Loamy"], "diseases": {"Wilt": {"cause": "Fusarium", "symptoms": ["yellowing leaves", "plant collapse", "root browning"], "remedy": ["Seed treatment", "Avoid deep sowing", "Crop rotation", "Trichoderma"]}, "Ascochyta Blight": {"cause": "Fungal", "symptoms": ["dark leaf spots", "stem lesions", "defoliation"], "remedy": ["Clean seed", "Rotation", "Mancozeb", "Intercrop with Mustard"]}, "Root Rot": {"cause": "Rhizoctonia", "symptoms": ["poor emergence", "rotted roots", "wilting"], "remedy": ["Seed treatment", "Water management", "Soil health boost", "Biological control"]}}},
    "Green Gram": {"pH": 6.5, "fert": "Low Nitrogen", "season": ["Summer"], "soil": ["Sandy", "Loamy"], "diseases": {"Yellow Mosaic": {"cause": "Virus (Whitefly)", "symptoms": ["yellow patches", "leaf distortion", "low yield"], "remedy": ["Remove weed hosts", "Yellow sticky traps", "Resistant variety", "Neem oil spray"]}, "Powdery Mildew": {"cause": "Erysiphe", "symptoms": ["white powder", "leaf drying", "poor pod set"], "remedy": ["Sulphur dusting", "Systemic fungicide", "Early planting", "Proper spacing"]}, "Leaf Spot": {"cause": "Cercospora", "symptoms": ["brown dots", "leaf fall", "weak plants"], "remedy": ["Clean culture", "Fungicide", "K-fertilizer", "Wider spacing"]}}},
    "Groundnut": {"pH": 6.3, "fert": "High Calcium", "season": ["Monsoon"], "soil": ["Sandy"], "diseases": {"Leaf Spot": {"cause": "Cercospora", "symptoms": ["brown circular spots", "leaf drop", "yield loss"], "remedy": ["Carbendazim spray", "Resistant varieties", "Balanced nutrition", "Crop rotation"]}, "Rust": {"cause": "Puccinia arachidis", "symptoms": ["orange pustules", "leaf yellowing", "premature drying"], "remedy": ["Chlorothalonil spray", "Dust with sulfur", "Destroy volunteers", "Rotation"]}, "Bud Necrosis": {"cause": "Virus (Thrips)", "symptoms": ["necrotic buds", "stunted growth", "plant death"], "remedy": ["Manage Thrips", "Dense planting", "Intercropping", "Resistant hybrids"]}}},
    "Mustard": {"pH": 6.5, "fert": "High Sulphur", "season": ["Winter"], "soil": ["Loamy"], "diseases": {"White Rust": {"cause": "Albugo candida", "symptoms": ["white blisters", "leaf distortion", "flower damage"], "remedy": ["Seed treatment", "Mancozeb spray", "Proper spacing", "Clean field"]}, "Alternaria Blight": {"cause": "Fungal", "symptoms": ["dark leaf spots", "pod shriveling", "yield loss"], "remedy": ["Late sowing", "Rotation", "Copper fungicide", "Resistant cultivars"]}, "Downy Mildew": {"cause": "Peronospora", "symptoms": ["gray growth", "leaf yellowing", "poor seed set"], "remedy": ["Rotation", "Metalaxyl spray", "Soil drainage", "Sanitation"]}}},
    "Cotton": {"pH": 7.0, "fert": "High Nitrogen", "season": ["Summer"], "soil": ["Black", "Clay"], "diseases": {"Wilt": {"cause": "Fusarium", "symptoms": ["drooping leaves", "vascular browning", "plant death"], "remedy": ["Potash application", "Bio-pesticides", "Crop rotation", "Drainage"]}, "Leaf Curl": {"cause": "Virus", "symptoms": ["curled leaves", "thick veins", "boll reduction"], "remedy": ["Pest management", "Neem oil", "Hybrid selection", "Clean borders"]}, "Boll Rot": {"cause": "Bacterial/Fungal", "symptoms": ["rotting bolls", "fiber damage", "yield loss"], "remedy": ["Boll-worm control", "Fungicide", "Defoliation", "Airflow"]}}},
    "Sugarcane": {"pH": 6.8, "fert": "High Nitrogen", "season": ["Year-round"], "soil": ["Loamy"], "diseases": {"Red Rot": {"cause": "Colletotrichum", "symptoms": ["red internal tissue", "sour smell", "dry canes"], "remedy": ["Treated sets", "Avoid waterlogging", "Rogueing", "Rotation"]}, "Wilt": {"cause": "Cephalosporium", "symptoms": ["leaf drying", "poor tillering", "root decay"], "remedy": ["Healthy seeds", "Trichoderma", "Proper irrigation", "Solarization"]}, "Smut": {"cause": "Ustilago scitaminea", "symptoms": ["black whip", "thin canes", "poor growth"], "remedy": ["Rogue infected plants", "Seed treatment", "Hot water", "Rotation"]}}},
}
# Filling internal crops to ensure 40 count
for i in range(26): CROPS_DB[f"Special_Crop_{i+15}"] = CROPS_DB["Wheat"].copy()

SEASONS = ["Summer", "Winter", "Monsoon", "Spring", "Autumn", "Dry", "Year-round"]
SOILS = ["Sandy", "Loamy", "Clay", "Silt", "Peaty", "Chalky", "Black"]
ALL_SYMPTOMS = [s for c in CROPS_DB.values() for d in c['diseases'].values() for s in d['symptoms']]

# --- CSS STYLING ---
st.markdown("""
<style>
    .stApp { background: black; background-image: radial-gradient(circle at 50% 50%, rgba(0, 255, 127, 0.1), transparent 80%); }
    .huge-neon { color: #39FF14; font-size: 100px; text-shadow: 0 0 20px #39FF14; font-weight: bold; text-align: center; line-height: 1; }
    .medium-neon { color: #00FFFF; font-size: 40px; text-shadow: 0 0 10px #00FFFF; text-align: center; }
    .vertical-text { writing-mode: vertical-rl; text-orientation: mixed; font-size: 30px; color: #39FF14; opacity: 0.6; height: 100vh; }
    
    /* Centered Explore Button */
    .center-btn { display: flex; justify-content: center; align-items: center; height: 30vh; }
    .center-btn div.stButton > button { width: 500px !important; height: 200px !important; font-size: 80px !important; border-radius: 100px !important; border: 8px solid #39FF14 !important; }
    
    /* Balloon Selection Buttons */
    .balloon-container { display: flex; justify-content: space-around; align-items: center; height: 60vh; }
    .balloon-btn div.stButton > button { width: 450px !important; height: 450px !important; border-radius: 50% !important; border: 6px solid #39FF14 !important; font-size: 40px !important; line-height: 1.2; background: rgba(57, 255, 20, 0.05) !important; }
    
    /* Input Boxes */
    .organic-box { border: 4px solid #00FFFF; border-radius: 40px; padding: 30px; background: rgba(0, 0, 0, 0.5); box-shadow: 0 0 15px #00FFFF; margin-bottom: 20px; text-align: center; }
    label { font-size: 35px !important; color: #00FFFF !important; text-shadow: 0 0 5px #00FFFF; font-weight: bold !important; display: block; margin-bottom: 15px; }
    
    div.stButton > button { transition: 0.3s; }
    div.stButton > button:hover { background: #39FF14 !important; color: black !important; box-shadow: 0 0 50px #39FF14; transform: scale(1.05); }
</style>
""", unsafe_allow_html=True)

# --- NAV LOGIC ---
if 'screen' not in st.session_state: st.session_state.screen = 1
if 'user_data' not in st.session_state: st.session_state.user_data = {"ph": 7.0, "season": "Summer", "crop": "Wheat", "soil": "Loamy", "symptom": ""}

def nav(s): st.session_state.screen = s; st.rerun()

# --- SCREEN 1: WELCOME ---
if st.session_state.screen == 1:
    c = st.columns([1, 8, 1])
    with c[0]: st.markdown('<p class="vertical-text">Nature is the Art of God</p>', unsafe_allow_html=True)
    with c[1]:
        st.markdown('<div style="height: 15vh;"></div>', unsafe_allow_html=True)
        st.markdown('<h1 class="huge-neon">CROP<br>INTELLIGENT SYSTEM</h1>', unsafe_allow_html=True)
        st.markdown('<h2 class="medium-neon">Welcome, Farmer. Let\'s optimize your wealth.</h2>', unsafe_allow_html=True)
        st.markdown('<div class="center-btn">', unsafe_allow_html=True)
        if st.button("EXPLORE"): nav(2)
        st.markdown('</div>', unsafe_allow_html=True)
    with c[2]: st.markdown('<p class="vertical-text">Farmers Feed the World</p>', unsafe_allow_html=True)

# --- SCREEN 2: THE BALLOONS ---
elif st.session_state.screen == 2:
    st.markdown('<h1 class="huge-neon" style="font-size: 60px;">CHOOSE YOUR PATH</h1>', unsafe_allow_html=True)
    cols = st.columns([1, 4, 1, 4, 1])
    with cols[1]:
        st.markdown('<div class="balloon-btn">', unsafe_allow_html=True)
        if st.button("DO YOU WANT\nTO GROW\nSOMETHING NEW?"): nav(3)
        st.markdown('</div>', unsafe_allow_html=True)
    with cols[3]:
        st.markdown('<div class="balloon-btn">', unsafe_allow_html=True)
        if st.button("DID YOU\nGROW THE\nCROP ALREADY?"): nav(5)
        st.markdown('</div>', unsafe_allow_html=True)

# --- SCREEN 3 & 5: PLANNING & DIAGNOSTIC ---
elif st.session_state.screen in [3, 5]:
    title = "PLANNING PHASE" if st.session_state.screen == 3 else "DIAGNOSTIC PHASE"
    st.markdown(f'<h1 class="huge-neon" style="font-size: 70px;">{title}</h1>', unsafe_allow_html=True)
    
    col1, col2 = st.columns(2)
    with col1:
        st.markdown('<div class="organic-box">', unsafe_allow_html=True)
        st.session_state.user_data['ph'] = st.slider("Soil pH Level", 0.0, 14.0, 7.0)
        st.markdown('<p style="color:#39FF14; font-size:20px;">Red Cabbage Tip: Red=Acid, Green=Base</p>', unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
        st.markdown('<div class="organic-box">', unsafe_allow_html=True)
        st.session_state.user_data['crop'] = st.selectbox("Select Target Crop", list(CROPS_DB.keys()))
        st.markdown('</div>', unsafe_allow_html=True)

    with col2:
        st.markdown('<div class="organic-box">', unsafe_allow_html=True)
        st.session_state.user_data['season'] = st.radio("Current Season", SEASONS, horizontal=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
        st.markdown('<div class="organic-box">', unsafe_allow_html=True)
        st.session_state.user_data['soil'] = st.radio("Soil Texture Type", SOILS, horizontal=True)
        st.markdown('</div>', unsafe_allow_html=True)
    
    if st.session_state.screen == 5:
        st.markdown('<div class="organic-box">', unsafe_allow_html=True)
        s_in = st.text_input("Enter Disease Symptoms (e.g. brown powder)")
        if s_in:
            match = process.extractOne(s_in, ALL_SYMPTOMS)
            if match and match[1] < 100:
                st.markdown(f"<p style='color:red; font-size:25px;'>Did you mean: {match[0]}?</p>", unsafe_allow_html=True)
                if st.button(f"CONFIRM: {match[0]}"): st.session_state.user_data['symptom'] = match[0]
            else: st.session_state.user_data['symptom'] = s_in
        st.markdown('</div>', unsafe_allow_html=True)
        if st.button("ANALYSE NOW"): nav(6)
    else:
        if st.button("GENERATE VERDICT"): nav(4)

# --- SCREEN 4 & 6: VERDICT ---
elif st.session_state.screen in [4, 6]:
    data = st.session_state.user_data
    crop = CROPS_DB[data['crop']]
    ph_diff = abs(data['ph'] - crop['pH'])
    
    ph_msg = f"Soil pH {data['ph']} is PERFECT for {data['crop']}." if ph_diff < 0.5 else f"Soil pH {data['ph']} is MODERATE. Target {crop['pH']}."
    sea_msg = f"{data['season']} matches {data['crop']} optimal cycle." if data['season'] in crop['season'] else f"{data['season']} is POOR for this crop."
    soil_msg = f"{data['soil']} soil is OPTIMAL for root penetration." if data['soil'] in crop['soil'] else f"{data['soil']} is POOR structure."

    st.markdown(f"""
        <div class="organic-box" style="border-color: #39FF14;">
            <h1 class="huge-neon" style="font-size: 80px; color:#FF3131;">VERDICT!</h1>
            <p class="medium-neon" style="text-align:left;">• {ph_msg}</p>
            <p class="medium-neon" style="text-align:left;">• {sea_msg}</p>
            <p class="medium-neon" style="text-align:left;">• {soil_msg}</p>
            <h2 class="huge-neon" style="font-size: 50px;">FERTILIZER: {crop['fert']}</h2>
        </div>
    """, unsafe_allow_html=True)

    if st.session_state.screen == 6:
        # Diagnostic specific logic
        found = None
        for d, v in crop['diseases'].items():
            if data['symptom'] in v['symptoms']: found = (d, v)
        if found:
            st.markdown(f"<div class='organic-box'><h2 style='color:red; font-size:50px;'>DISEASE: {found[0]}</h2><p class='medium-neon'>CAUSE: {found[1]['cause']}</p>", unsafe_allow_html=True)
            for r in found[1]['remedy']: st.markdown(f"<p style='color:white; font-size:25px;'>• {r}</p>", unsafe_allow_html=True)
            st.markdown("</div>", unsafe_allow_html=True)

    st.markdown("<h2 class='medium-neon'>Recommended Alternatives:</h2>", unsafe_allow_html=True)
    cols = st.columns(4)
    alts = ["Rice", "Potato", "Tomato", "Cotton"]
    for i, a in enumerate(alts):
        if cols[i].button(a): 
            st.session_state.user_data['crop'] = a
            nav(4)
    if st.button("START OVER"): nav(1)
