import streamlit as st
import pandas as pd

# Konfigurasi halaman
st.set_page_config(
    page_title="Dashboard Statistik Pemain Sepak Bola",
    page_icon="⚽",
    layout="wide"
)

# Membaca data
@st.cache_data
def load_data():
    return pd.read_csv("players.csv")

df = load_data()

# Judul
st.title("⚽ Dashboard Statistik Pemain Sepak Bola")

# Sidebar
st.sidebar.header("Filter Data")

# Filter negara
negara_list = sorted(df["team_country"].dropna().unique())
negara = st.sidebar.selectbox(
    "Pilih Negara",
    ["Semua"] + list(negara_list)
)

# Filter posisi
posisi_list = sorted(df["position"].dropna().unique())
posisi = st.sidebar.selectbox(
    "Pilih Posisi",
    ["Semua"] + list(posisi_list)
)

# Pencarian pemain
nama_pemain = st.sidebar.text_input("Cari Nama Pemain")

# Filter dataframe
filtered_df = df.copy()

if negara != "Semua":
    filtered_df = filtered_df[
        filtered_df["team_country"] == negara
    ]

if posisi != "Semua":
    filtered_df = filtered_df[
        filtered_df["position"] == posisi
    ]

if nama_pemain:
    filtered_df = filtered_df[
        filtered_df["player"].str.contains(
            nama_pemain,
            case=False,
            na=False
        )
    ]

# Statistik utama
col1, col2, col3 = st.columns(3)

with col1:
    st.metric(
        "Jumlah Pemain",
        len(filtered_df)
    )

with col2:
    st.metric(
        "Jumlah Negara",
        filtered_df["team_country"].nunique()
    )

with col3:
    st.metric(
        "Jumlah Klub",
        filtered_df["club"].nunique()
    )

st.divider()

# Data pemain
st.subheader("Data Pemain")
st.dataframe(filtered_df, use_container_width=True)

st.divider()

# Top Scorer
st.subheader("🏆 Top 10 Pencetak Gol")

top_goals = (
    filtered_df[["player", "goals"]]
    .sort_values(by="goals", ascending=False)
    .head(10)
)

st.dataframe(top_goals, use_container_width=True)

st.bar_chart(
    top_goals.set_index("player")
)

st.divider()

# Top Assist
st.subheader("🎯 Top 10 Assist")

top_assists = (
    filtered_df[["player", "assists"]]
    .sort_values(by="assists", ascending=False)
    .head(10)
)

st.dataframe(top_assists, use_container_width=True)

st.bar_chart(
    top_assists.set_index("player")
)

st.divider()

# Statistik per negara
st.subheader("🌍 Jumlah Pemain per Negara")

country_count = (
    filtered_df["team_country"]
    .value_counts()
)

st.bar_chart(country_count)

st.divider()

# Detail pemain
st.subheader("🔍 Detail Pemain")

pilihan_pemain = st.selectbox(
    "Pilih Pemain",
    filtered_df["player"].sort_values().unique()
)

detail = filtered_df[
    filtered_df["player"] == pilihan_pemain
]

st.dataframe(detail.T, use_container_width=True)