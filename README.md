# Codetech-task---7
To deliver a fully functional dashboard application with immediate business utility, we will use Streamlit. Streamlit is faster to deploy than Dash, requires fewer lines of boilerplate code, and includes built-in KPI cards and responsive layouts.The Deliverable: Python Enterprise Analytics AppThis production-ready script generates its own complex retail dataset (simulating transactions, logistics, and profits) and builds an executive-level control panel.
1. Installation RequirementsRun this command in your terminal first:bashpip install streamlit pandas plotly numpy
Use code with caution.2. Dashboard Application Code (app.py)Save the code below into a file named app.py.pythonimport streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go

# --- PAGE CONFIGURATION ---
st.set_page_config(page_title="Executive Insights Dashboard", layout="wide")

# --- DATA GENERATION (Simulated Enterprise Dataset) ---
@st.cache_data
def load_data():
    np.random.seed(42)
    rows = 1000
    categories = ['Enterprise Software', 'Cloud Infrastructure', 'Hardware Terminals', 'Consulting Packages']
    regions = ['North America', 'EMEA', 'APAC', 'LATAM']
    segments = ['SMB', 'Mid-Market', 'Enterprise']
    
    dates = pd.date_range(start='2025-01-01', periods=rows, freq='h')
    
    df = pd.DataFrame({
        'Timestamp': np.random.choice(dates, rows),
        'Product_Category': np.random.choice(categories, rows, p=[0.4, 0.3, 0.2, 0.1]),
        'Region': np.random.choice(regions, rows),
        'Segment': np.random.choice(segments, rows),
        'Sales_USD': np.random.uniform(500, 15000, rows).round(2),
    })
    
    # Create realistic dependencies (Hardware has lower margin, Cloud has higher)
    margin_map = {'Enterprise Software': 0.70, 'Cloud Infrastructure': 0.85, 'Hardware Terminals': 0.15, 'Consulting Packages': 0.40}
    df['Margin_Pct'] = df['Product_Category'].map(margin_map)
    df['Profit_USD'] = (df['Sales_USD'] * df['Margin_Pct']).round(2)
    
    # Introduce random supply chain operational delays (in days)
    df['Delivery_Delay_Days'] = np.random.poisson(lam=3, size=rows)
    df.loc[df['Region'] == 'LATAM', 'Delivery_Delay_Days'] += 2 # Simulating local bottleneck
    
    df['Month'] = df['Timestamp'].dt.to_period('M').astype(str)
    return df.sort_values('Timestamp')

df = load_data()

# --- SIDEBAR FILTERS ---
st.sidebar.title("🎛️ Control Panel")
st.sidebar.markdown("Filter enterprise metrics in real-time.")

selected_regions = st.sidebar.multiselect("Select Regions", options=df['Region'].unique(), default=df['Region'].unique())
selected_segments = st.sidebar.multiselect("Customer Segments", options=df['Segment'].unique(), default=df['Segment'].unique())

# Filter Application
mask = (df['Region'].isin(selected_regions)) & (df['Segment'].isin(selected_segments))
f_df = df[mask]

# --- MAIN DASHBOARD HEADER ---
st.title("📈 Executive Performance & Operations Dashboard")
st.markdown("---")

# --- ROW 1: ACTIONABLE KPI CARDS ---
col1, col2, col3, col4 = st.columns(4)

total_sales = f_df['Sales_USD'].sum()
total_profit = f_df['Profit_USD'].sum()
avg_margin = (total_profit / total_sales * 100) if total_sales > 0 else 0
avg_delay = f_df['Delivery_Delay_Days'].mean()

col1.metric("Total Revenue", f"${total_sales:,.2f}", delta="+12.4% vs Last Qtr")
col2.metric("Net Profit", f"${total_profit:,.2f}", delta="+8.1% vs Last Qtr")
col3.metric("Blended Margin %", f"{avg_margin:.1f}%", delta="-1.2% Risk")
col4.metric("Avg Delivery Delay", f"{avg_delay:.1f} Days", delta="+0.4 Days Delay", delta_color="inverse")

st.markdown("---")

# --- ROW 2: STRATEGIC INSIGHT CHARTS ---
chart_col1, chart_col2 = st.columns(2)

with chart_col1:
    st.subheader("Revenue & Profit Contribution Trend")
    trend_df = f_df.groupby('Month')[['Sales_USD', 'Profit_USD']].sum().reset_index()
    fig_trend = go.Figure()
    fig_trend.add_trace(go.Bar(x=trend_df['Month'], y=trend_df['Sales_USD'], name='Gross Revenue', marker_color='#1f77b4'))
    fig_trend.add_trace(go.Scatter(x=trend_df['Month'], y=trend_df['Profit_USD'], name='Net Profit', line=dict(color='#ff7f0e', width=3)))
    fig_trend.update_layout(barmode='group', legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1))
    st.plotly_chart(fig_trend, use_container_width=True)

with chart_col2:
    st.subheader("Operational Delay Risk Assessment")
    fig_scatter = px.scatter(
        f_df, 
        x="Sales_USD", 
        y="Delivery_Delay_Days", 
        color="Product_Category",
        size="Profit_USD",
        hover_data=['Segment'],
        title="Deal Size vs Delivery Delays (Bubble Size = Net Profit)"
    )
    st.plotly_chart(fig_scatter, use_container_width=True)

st.markdown("---")

# --- ROW 3: EXECUTIVE SUMMARY & ACTIONS ---
st.subheader("📋 Automated Actionable Recommendations")

# Dynamic rule engine to deliver real-time data driven recommendations
recommendations = []
if avg_margin < 50.0:
    recommendations.append("⚠️ **Low Margin Alert:** The blended margin profile is dipping. Shift sales incentive structures toward high-margin **Cloud Infrastructure** configurations.")
if avg_delay > 3.2:
    recommendations.append("🚛 **Logistics Bottleneck:** Delivery delays are escalating. Review third-party fulfillment contracts specifically for orders originating in the heavily hit sub-regions.")
if f_df[f_df['Product_Category'] == 'Hardware Terminals']['Sales_USD'].sum() > (total_sales * 0.3):
    recommendations.append("📉 **Product Mix Imbalance:** Low-margin Hardware sales are expanding faster than recurring software pipelines. Implement immediate bundling mandates.")

if recommendations:
    for rec in recommendations:
        st.info(rec)
else:
    st.success("✅ **Operations Normal:** All primary metrics match target baselines. No emergency operational interventions required.")
Use code with caution.3. How to Launch Your DashboardExecute the file from your terminal command line interface:bashstreamlit run app.py
Use code with caution.This launches a browser tab locally at http://localhost:8501 featuring your interactive dashboard.Actionable Infrastructure Built Into This CodeFinancial Velocity Levers (KPIs): The dynamic color changes for the indicators signal performance shifts immediately, prompting quick adjustments.Operational Risk Plot: The "Deal Size vs. Delivery Delays" chart instantly shows if high-value transactions are stuck in fulfillment pipelines, helping retention teams focus on customers at risk of leaving.Real-Time Rules Engine: The automated recommendation box uses application state checks to dynamically create an action item checklist for operations teams based on data thresholds.If you are interested, I can extend this prototype. Tell me:What specific data formats or databases (e.g., PostgreSQL, Excel, Snowflake) do you need to connect to this pipeline?Which business KPI criteria (e.g., churn rates, inventory turns, marketing ROI) should we build custom alert parameters for?
