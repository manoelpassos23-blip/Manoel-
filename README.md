meu app trader
app.py
import pandas as pd
import plotly.express as px
import streamlit as st

# Configuração da página do aplicativo
st.set_page_config(page_title="Painel de Assertividade Trader", layout="wide")

st.title("📊 Painel de Validação e Assertividade Estatística")
st.subheader("Descubra onde estão seus melhores padrões operacionais")

# --- SIMULAÇÃO DE BANCO DE DADOS ---
# Em um app real, esses dados viriam de um formulário ou arquivo CSV.
if "dados_operacoes" not in st.session_state:
    st.session_state.dados_operacoes = pd.DataFrame(
        [
            {
                "Ativo": "EUR/USD",
                "Estratégia": "M5 - Suporte/Resistência",
                "Horário": "Manhã (08h-12h)",
                "Resultado": "Win",
            },
            {
                "Ativo": "GBP/USD",
                "Estratégia": "M1 - Fluxo de Vela",
                "Horário": "Tarde (12h-18h)",
                "Resultado": "Loss",
            },
            {
                "Ativo": "EUR/USD",
                "Estratégia": "M5 - Suporte/Resistência",
                "Horário": "Manhã (08h-12h)",
                "Resultado": "Win",
            },
            {
                "Ativo": "BTC/USD",
                "Estratégia": "M1 - Pullback",
                "Horário": "Noite (18h-00h)",
                "Resultado": "Win",
            },
            {
                "Ativo": "EUR/USD",
                "Estratégia": "M1 - Fluxo de Vela",
                "Horário": "Manhã (08h-12h)",
                "Resultado": "Loss",
            },
            {
                "Ativo": "GBP/USD",
                "Estratégia": "M5 - Suporte/Resistência",
                "Horário": "Manhã (08h-12h)",
                "Resultado": "Win",
            },
        ]
    )

df = st.session_state.dados_operacoes

# --- PAINEL LATERAL PARA ADICIONAR NOVAS ANÁLISES ---
st.sidebar.header("📝 Cadastrar Nova Operação")
with st.sidebar.form("formulario_operacao"):
    novo_ativo = st.selectbox(
        "Par de Moedas / Ativo", ["EUR/USD", "GBP/USD", "USD/JPY", "BTC/USD"]
    )
    nova_estrategia = st.selectbox(
        "Estratégia Analisada",
        ["M5 - Suporte/Resistência", "M1 - Pullback", "M1 - Fluxo de Vela"],
    )
    novo_horario = st.selectbox(
        "Horário da Operação",
        ["Madrugada (00h-08h)", "Manhã (08h-12h)", "Tarde (12h-18h)", "Noite (18h-00h)"],
    )
    novo_resultado = st.radio("Resultado da Análise", ["Win", "Loss"])

    botao_salvar = st.form_submit_button("Salvar Operação")

    if botao_salvar:
        nova_linha = {
            "Ativo": novo_ativo,
            "Estratégia": nova_estrategia,
            "Horário": novo_horario,
            "Resultado": novo_resultado,
        }
        st.session_state.dados_operacoes = pd.concat(
            [df, pd.DataFrame([nova_linha])], ignore_index=True
        )
        st.rerun()

# --- MÉTRICAS GERAIS ---
total_ops = len(df)
wins = len(df[df["Resultado"] == "Win"])
assertividade_geral = (wins / total_ops * 100) if total_ops > 0 else 0

col1, col2, col3 = st.columns(3)
col1.metric(label="Total de Análises Cadastradas", value=total_ops)
col2.metric(label="Total de Acertos (Wins)", value=wins)
col3.metric(
    label="Assertividade Geral", value=f"{assertividade_geral:.1f}%", delta=None
)

st.markdown("---")

# --- ANÁLISE DE ASSERTIVIDADE POR CATEGORIA ---
st.header("🎯 Onde está sua maior assertividade?")

col_graf1, col_graf2 = st.columns(2)

with col_graf1:
    st.subheader("Assertividade por Estratégia")
    # Calcula a taxa de acerto por estratégia
    df_estrategia = (
        df.groupby(["Estratégia", "Resultado"]).size().unstack(fill_value=0)
    )
    if "Win" not in df_estrategia.columns:
        df_estrategia["Win"] = 0
    if "Loss" not in df_estrategia.columns:
        df_estrategia["Loss"] = 0
    df_estrategia["Taxa de Acerto (%)"] = (
        df_estrategia["Win"] / (df_estrategia["Win"] + df_estrategia["Loss"]) * 100
    )
    df_estrategia = df_estrategia.reset_index()

    fig1 = px.bar(
        df_estrategia,
        x="Estratégia",
        y="Taxa de Acerto (%)",
        text=df_estrategia["Taxa de Acerto (%)"].map("{:.1f}%".format),
        color="Estratégia",
        range_y=[0, 100],
    )
    st.plotly_chart(fig1, use_container_width=True)

with col_graf2:
    st.subheader("Assertividade por Horário")
    # Calcula a taxa de acerto por horário
    df_horario = df.groupby(["Horário", "Resultado"]).size().unstack(fill_value=0)
    if "Win" not in df_horario.columns:
        df_horario["Win"] = 0
    if "Loss" not in df_horario.columns:
        df_horario["Loss"] = 0
    df_horario["Taxa de Acerto (%)"] = (
        df_horario["Win"] / (df_horario["Win"] + df_horario["Loss"]) * 100
    )
    df_horario = df_horario.reset_index()

    fig2 = px.bar(
        df_horario,
        x="Horário",
        y="Taxa de Acerto (%)",
        text=df_horario["Taxa de Acerto (%)"].map("{:.1f}%".format),
        color="Horário",
        range_y=[0, 100],
    )
    st.plotly_chart(fig2, use_container_width=True)

# --- TABELA DE DADOS ---
st.markdown("---")
st.subheader("📋 Histórico de Operações Registradas")
st.dataframe(df, use_container_width=True)
