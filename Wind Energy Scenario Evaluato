import streamlit as st
from fpdf import FPDF
from io import BytesIO
import re

# --- Emoji-safe helper ---
def remove_emojis(text):
    return re.sub(r'[^\x00-\xFF]', '', text)

st.set_page_config(page_title="🌿 Wind Energy Scenario Evaluator", layout="wide")

# --- HEADER & INSTRUCTIONS ---
st.title("🏡 Wind Energy Scenario Evaluator")

st.markdown("""
Welcome to the **Wind Energy Scenario Evaluator**! 🌬️

You're tasked with helping a small village choose the most cost-effective and environmentally friendly wind farm plan.

🔧 **Instructions**:
1. Choose how many scenarios you want to compare.
2. Enter the details for each plan.
3. The app will calculate installation costs, maintenance, income, and payback period.
4. You'll get a clear recommendation and a downloadable PDF report!

Let's build a greener future together. 🌍
""")

# --- CURRENCY & SCENARIO COUNT ---
currency = st.selectbox("💱 Choose your currency:", ["USD", "EUR", "EGP", "GBP", "JPY", "INR"])
currency_symbols = {"USD": "$", "EUR": "€", "EGP": "E£", "GBP": "£", "JPY": "¥", "INR": "₹"}
symbol = currency_symbols[currency]

n_scenarios = st.slider("🔢 Number of scenarios to compare", min_value=2, max_value=5, value=2)

# --- SCENARIO INPUTS ---
st.header("📊 Enter Details for Each Plan")

scenarios = []
cols = st.columns(n_scenarios)

for i in range(n_scenarios):
    with cols[i]:
        st.subheader(f"Scenario {i + 1}")
        s = {}
        s['name'] = st.text_input("Plan name", value=f"Plan {i + 1}", key=f"name_{i}")
        s['n_turbines'] = st.number_input("Number of turbines", min_value=1, value=5, key=f"turbines_{i}")
        s['turbine_cost'] = st.number_input(f"Cost per turbine ({currency})", value=100000, step=10000, key=f"cost_{i}")
        s['annual_maintenance'] = st.number_input(f"Annual maintenance per turbine ({currency})", value=3000, key=f"maint_{i}")
        s['annual_income'] = st.number_input(f"Annual income per turbine ({currency})", value=10000, key=f"income_{i}")
        s['battery'] = st.checkbox("Include battery storage?", key=f"battery_{i}")
        if s['battery']:
            s['battery_cost'] = st.number_input(f"Battery cost ({currency})", value=50000, step=5000, key=f"battery_cost_{i}")
            s['battery_savings'] = st.number_input(f"Extra annual savings from battery ({currency})", value=2000, key=f"battery_savings_{i}")
        else:
            s['battery_cost'] = 0
            s['battery_savings'] = 0
        scenarios.append(s)

# --- CALCULATIONS ---
for s in scenarios:
    s['install_cost'] = s['n_turbines'] * s['turbine_cost'] + s['battery_cost']
    s['total_income'] = s['n_turbines'] * s['annual_income'] + s['battery_savings']
    s['total_maintenance'] = s['n_turbines'] * s['annual_maintenance']
    s['net_savings'] = s['total_income'] - s['total_maintenance']
    s['payback'] = s['install_cost'] / s['net_savings'] if s['net_savings'] > 0 else float('inf')

# --- DISPLAY COMPARISON ---
st.header("📈 Financial Comparison")

compare_cols = st.columns(n_scenarios)
for i in range(n_scenarios):
    s = scenarios[i]
    with compare_cols[i]:
        st.subheader(s['name'])
        st.write(f"**Total Installation Cost:** {symbol}{s['install_cost']:,.2f}")
        st.write(f"**Annual Income (with battery):** {symbol}{s['total_income']:,.2f}")
        st.write(f"**Annual Maintenance:** {symbol}{s['total_maintenance']:,.2f}")
        st.write(f"**Net Annual Savings:** {symbol}{s['net_savings']:,.2f}")
        if s['payback'] == float('inf'):
            st.write("**Payback Period:** Not achievable (no savings)")
        else:
            st.write(f"**Payback Period:** {s['payback']:.2f} years")

# --- BEST PLAN LOGIC ---
st.header("🤖 Recommendation: Best Plan")

valid = [(i, s['payback']) for i, s in enumerate(scenarios) if s['payback'] != float('inf')]

winner = None
reason = ""

if not valid:
    st.warning("No plan can recover its cost. Please revise your inputs.")
else:
    best_index = min(valid, key=lambda x: x[1])[0]
    winner = scenarios[best_index]
    reason = f"✅ **{winner['name']}** has the **shortest payback period**: {winner['payback']:.2f} years."
    st.success(f"🏆 **Recommended Plan: {winner['name']}**\n\n{reason}")

# --- PDF GENERATION ---
def generate_pdf(scenarios, winner, reason, currency, symbol):
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", 'B', 14)
    pdf.cell(200, 10, "Wind Energy Scenario Report", ln=True, align="C")
    pdf.ln(5)

    for i, s in enumerate(scenarios):
        pdf.set_font("Arial", 'B', 12)
        pdf.cell(200, 10, f"{s['name']} (Scenario {i + 1})", ln=True)
        pdf.set_font("Arial", '', 12)
        pdf.cell(200, 8, f"Number of Turbines: {s['n_turbines']}", ln=True)
        pdf.cell(200, 8, f"Turbine Cost: {symbol}{s['turbine_cost']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Annual Maintenance: {symbol}{s['annual_maintenance']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Annual Income per Turbine: {symbol}{s['annual_income']:,.2f}", ln=True)
        if s['battery']:
            pdf.cell(200, 8, f"Battery Cost: {symbol}{s['battery_cost']:,.2f}", ln=True)
            pdf.cell(200, 8, f"Battery Extra Annual Savings: {symbol}{s['battery_savings']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Total Installation Cost: {symbol}{s['install_cost']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Total Income: {symbol}{s['total_income']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Total Maintenance: {symbol}{s['total_maintenance']:,.2f}", ln=True)
        pdf.cell(200, 8, f"Net Annual Savings: {symbol}{s['net_savings']:,.2f}", ln=True)
        if s['payback'] != float('inf'):
            pdf.cell(200, 8, f"Payback Period: {s['payback']:.2f} years", ln=True)
        else:
            pdf.cell(200, 8, "Payback Period: Not achievable", ln=True)
        pdf.ln(5)

    if winner:
        pdf.set_font("Arial", 'B', 12)
        pdf.cell(200, 10, f"Recommended Plan: {winner['name']}", ln=True)
        pdf.set_font("Arial", '', 12)
        pdf.multi_cell(0, 8, remove_emojis(reason))

    return pdf

# --- PDF DOWNLOAD ---
st.header("📄 Export Your Report")
if st.button("Generate PDF Report"):
    pdf = generate_pdf(scenarios, winner, reason, currency, symbol)
    pdf_bytes = pdf.output(dest='S').encode('latin-1')
    buffer = BytesIO(pdf_bytes)

    st.download_button(
        label="📥 Download PDF",
        data=buffer,
        file_name="wind_energy_report.pdf",
        mime="application/pdf"
    )
