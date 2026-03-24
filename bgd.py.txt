import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# Page setup
st.set_page_config(page_title="Kenya Population Dashboard", layout="wide")

# Title
st.title("Kenya Population Dashboard (2019 Census)")

# Load dataset
df = pd.read_csv("kenya-population-distibution-2019-census.csv")

# Clean column names
df.columns = df.columns.str.strip()

# Fix numeric columns
for col in ["Total", "Male", "Female"]:
    df[col] = df[col].astype(str).str.replace(",", "", regex=False)
    df[col] = pd.to_numeric(df[col], errors="coerce")

# Drop missing values
df = df.dropna(subset=["Total"])

# Columns
county_col = "County"
pop_col = "Total"

# Sidebar
st.sidebar.header("Controls")
top_n = st.sidebar.slider("Select Top N Counties", 5, 20, 10)

# Sort
top_data = df.sort_values(by=pop_col, ascending=False).head(top_n)

# Table
st.subheader("Top Counties by Population")
st.dataframe(top_data, use_container_width=True)

# Layout
col1, col2 = st.columns(2)

# Bar chart
with col1:
    st.subheader("Bar Chart")
    fig, ax = plt.subplots()
    ax.bar(top_data[county_col], top_data[pop_col])
    plt.xticks(rotation=45)
    ax.set_xlabel("County")
    ax.set_ylabel("Population")
    st.pyplot(fig)

# Pie chart
with col2:
    st.subheader("Population Share")
    fig2, ax2 = plt.subplots()
    ax2.pie(top_data[pop_col], labels=top_data[county_col], autopct='%1.1f%%')
    st.pyplot(fig2)

# KPIs
st.subheader("Key Metrics")

total = df[pop_col].sum()
avg = df[pop_col].mean()
max_val = df[pop_col].max()

k1, k2, k3 = st.columns(3)
k1.metric("Total Population", f"{total:,.0f}")
k2.metric("Average Population", f"{avg:,.0f}")
k3.metric("Highest County Population", f"{max_val:,.0f}")

# Insight
st.subheader("Top County")
st.success(f"The county with the highest population is {top_data.iloc[0][county_col]}")