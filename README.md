# diario-alimentare-
import streamlit as st

# Configurazione pagina stile App Mobile
st.set_page_config(page_title="Diario Alimentare", page_icon="🍏", layout="centered")

st.title("🍏 Diario Alimentare")
st.write("Inserisci i tuoi pasti e tieni traccia delle calorie giornaliere.")

# Inizializza il database e i pasti nella sessione se non esistono
if "database_cibi" not in st.session_state:
    st.session_state.database_cibi = {
        "riso": 130.0,
        "petto di pollo": 165.0,
        "olio d'oliva": 884.0,
        "mela": 52.0,
        "pane": 265.0,
        "uovo": 155.0,
        "salmone": 208.0,
        "avocado": 160.0
    }

if "pasti_giornata" not in st.session_state:
    st.session_state.pasti_giornata = []

# --- SEZIONE INSERIMENTO ---
st.subheader("✍️ Aggiungi un alimento")

# Selezione del cibo o aggiunta di uno nuovo
opzioni_cibo = sorted(list(st.session_state.database_cibi.keys())) + ["+ Aggiungi nuovo alimento..."]
cibo_scelto = st.selectbox("Scegli il cibo:", opzioni_cibo)

kcal_100g = 0.0
nome_nuovo_cibo = ""

if cibo_scelto == "+ Aggiungi nuovo alimento...":
    nome_nuovo_cibo = st.text_input("Nome del nuovo cibo:").strip().lower()
    kcal_100g = st.number_input("Calorie per 100g:", min_value=0.0, step=1.0, value=0.0)
else:
    kcal_100g = st.session_state.database_cibi[cibo_scelto]
    st.info(f"Valore energetico: {kcal_100g} kcal per 100g")

grammi = st.number_input("Grammi consumati (g):", min_value=1.0, step=1.0, value=100.0)

if st.button("Aggiungi al totale", type="primary"):
    cibo_effettivo = nome_nuovo_cibo if cibo_scelto == "+ Aggiungi nuovo alimento..." else cibo_scelto
    
    if cibo_effettivo == "":
        st.error("Inserisci un nome valido per il cibo!")
    else:
        if cibo_scelto == "+ Aggiungi nuovo alimento...":
            st.session_state.database_cibi[cibo_effettivo] = kcal_100g
        
        calorie_totali = (kcal_100g * grammi) / 100.0
        st.session_state.pasti_giornata.append({
            "cibo": cibo_effettivo,
            "grammi": grammi,
            "calorie": calorie_totali
        })
        st.success(f"Aggiunto: {grammi}g di {cibo_effettivo} ({calorie_totali:.1f} kcal)")

# --- SEZIONE RIEPILOGO ---
st.markdown("---")
st.subheader("📊 Riepilogo Giornaliero")

if not st.session_state.pasti_giornata:
    st.info("Non hai ancora inserito cibi oggi. Il tuo totale è 0 kcal.")
else:
    totale_calorie = 0.0
    for item in st.session_state.pasti_giornata:
        st.write(f"• **{item['grammi']}g** di *{item['cibo'].capitalize()}* ➔ **{item['calorie']:.1f} kcal**")
        totale_calorie += item['calorie']
    
    st.markdown("---")
    st.metric(label="CALORIE TOTALI CONSUMATE", value=f"{totale_calorie:.1f} kcal")

    if st.button("Azzera Giornata"):
        st.session_state.pasti_giornata = []
        st.rerun()