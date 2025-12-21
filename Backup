import streamlit as st
import pandas as pd
import plotly.graph_objects as go
#streamlit run streamlit_app.py

# Initialize session state
if "trades" not in st.session_state:
    st.session_state.trades = []
if "risk" not in st.session_state:
    st.session_state.risk = 100
if "account_size" not in st.session_state:
    st.session_state.account_size = 50000

st.set_page_config(page_title="Trade Dashboard", layout="wide")

st.title("📊 Trade Dashboard (Advanced)")

# Sidebar: Account + Risk + Save/Load
st.sidebar.header("⚙️ Settings")

account_choice = st.sidebar.selectbox(
    "Account Size",
    options=[50000, 100000, 150000, "Custom"],
    index=0
)
if account_choice == "Custom":
    st.session_state.account_size = st.sidebar.number_input("Enter Account Size", value=st.session_state.account_size, min_value=1000)
else:
    st.session_state.account_size = account_choice

st.session_state.risk = st.sidebar.number_input("Risk per trade", value=st.session_state.risk, min_value=1)

if st.sidebar.button("💾 Save Trades"):
    if st.session_state.trades:
        df = pd.DataFrame(st.session_state.trades)
        df.to_csv("trades.csv", index=False)
        st.sidebar.success("Trades saved!")

uploaded_file = st.sidebar.file_uploader("📂 Load Trades", type="csv")
if uploaded_file:
    st.session_state.trades = pd.read_csv(uploaded_file).to_dict("records")
    st.sidebar.success("Trades loaded!")

# --- Trade buttons ---
st.subheader("Record a Trade")
col1, col2, col3, col4, col5 = st.columns(5)
comment = st.text_input("Optional Comment")

with col1:
    if st.button("✅ +0.5R", use_container_width=True):
        st.session_state.trades.append({"R": 0.5, "Comment": comment})
with col2:
    if st.button("✅ +1R", use_container_width=True):
        st.session_state.trades.append({"R": 1, "Comment": comment})
with col3:
    if st.button("✅ +2R", use_container_width=True):
        st.session_state.trades.append({"R": 2, "Comment": comment})
with col4:
    if st.button("❌ -1R", use_container_width=True):
        st.session_state.trades.append({"R": -1, "Comment": comment})
with col5:
    if st.button("↩️ Undo Last", use_container_width=True):
        if st.session_state.trades:
            st.session_state.trades.pop()
            st.warning("Last trade entry removed!")


# Display analytics
if st.session_state.trades:
    df = pd.DataFrame(st.session_state.trades)
    df["PnL"] = df["R"] * st.session_state.risk
    df["Cumulative PnL"] = df["PnL"].cumsum()

    total_pnl = df["PnL"].sum()
    winners = df[df["PnL"] > 0]
    losers = df[df["PnL"] <= 0]
    total_profit = winners["PnL"].sum()
    total_loss = losers["PnL"].sum()
    avg_win = winners["PnL"].mean() if not winners.empty else 0
    avg_loss = losers["PnL"].mean() if not losers.empty else 0
    profit_factor = (total_profit / abs(total_loss)) if not losers.empty else float("inf")
    win_rate = len(winners) / len(df) * 100 if len(df) > 0 else 0
    expectancy = df["R"].mean() if not df.empty else 0

    # --- Consecutive Wins/Losses ---
    streaks = []
    current = 0
    for pnl in df["PnL"]:
        if pnl > 0:
            current = current + 1 if current >= 0 else 1
        else:
            current = current - 1 if current <= 0 else -1
        streaks.append(current)

    current_streak = streaks[-1]
    consecutive_wins = max([s for s in streaks if s > 0], default=0)
    consecutive_losses = min([s for s in streaks if s < 0], default=0)

    # --- Max Drawdown ---
    equity_curve = df["Cumulative PnL"]
    rolling_max = equity_curve.cummax()
    drawdown = equity_curve - rolling_max
    max_drawdown = drawdown.min()

    # --- Row 1: PnL Stats ---
    r1c1, r1c2, r1c3, r1c4 = st.columns(4)
    r1c1.metric("💰 Net PnL", f"{total_pnl:,.2f}")
    r1c2.metric("📈 Total Profit", f"{total_profit:,.2f}")
    r1c3.metric("📉 Total Loss", f"{total_loss:,.2f}")
    r1c4.metric("⚠️ Max Drawdown", f"{max_drawdown:,.2f}")

    # --- Row 2: Performance Stats ---
    r2c1, r2c2, r2c3 = st.columns(3)
    r2c1.metric("✅ Win Rate", f"{win_rate:.1f}%")
    r2c2.metric("📊 Profit Factor", f"{profit_factor:.2f}")
    r2c3.metric("📈 Expectancy", f"{expectancy:.2f} R")

    # --- Row 3: Streaks ---
    r3c1, r3c2, r3c3 = st.columns(3)
    r3c1.metric("🔥 Current Streak", f"{current_streak}")
    r3c2.metric("🏆 Max Wins", f"{consecutive_wins}")
    r3c3.metric("💀 Max Losses", f"{abs(consecutive_losses)}")

    # --- Trade History ---
    st.subheader("📜 Trade History")
    st.dataframe(df, use_container_width=True)

    # --- Equity Curve (starts from 0) ---
    st.subheader("📈 Equity Curve (PnL Based)")
    fig = go.Figure()
    fig.add_trace(go.Scatter(
        x=df.index + 1,
        y=df["Cumulative PnL"],
        mode="lines+markers",
        line=dict(color="green" if total_pnl >= 0 else "red"),
        fill="tozeroy",
        name="Equity Curve"
    ))
    fig.update_layout(
        xaxis_title="Trade #",
        yaxis_title="PnL",
        height=300
    )
    st.plotly_chart(fig, use_container_width=True)