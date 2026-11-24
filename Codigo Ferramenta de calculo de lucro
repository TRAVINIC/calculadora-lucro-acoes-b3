# ==============================================================================

# LUCRO DE PREÇO ALVO - APLICAÇÃO MINIMALISTA (GOOGLE COLAB)

# ==============================================================================



import subprocess

import sys



def instalar_dependencias():

    pacotes = ["ipywidgets", "requests"]

    for pacote in pacotes:

        subprocess.check_call([sys.executable, "-m", "pip", "install", pacote, "-q"])



instalar_dependencias()



import re

import requests

import ipywidgets as widgets

from IPython.display import display, HTML



# ------------------------------------------------------------------------------

# Lista padrão de ativos da B3 (CORRIGIDO: "M2ST34")

# ------------------------------------------------------------------------------

ATIVOS_B3_POPULARES = [

    "CXSE3", "M2ST34", "PETR4", "VALE3", "ITUB4", "BBDC4", "BBAS3",

    "ABEV3", "WEGE3", "RENT3", "MGLU3", "LREN3", "B3SA3", "SUZB3",

    "PRIO3", "GGBR4", "CSNA3", "ELET3", "VBBR3", "RAIZ4", "EMBR3", "AZUL4", "NVDC34"

]



# ------------------------------------------------------------------------------

# 1. API DE COTAÇÃO DA B3 (BRAPI + YAHOO FINANCE)

# ------------------------------------------------------------------------------

def buscar_preco_mercado(ticker):

    """Consulta o preço atual de mercado do ativo via API pública (Brapi / Yahoo)."""

    ticker_clean = ticker.strip().upper()

    if not ticker_clean:

        return 0.0



    # 1. Busca via Brapi API (ideal para B3 e BDRs)

    try:

        url = f"https://brapi.dev/api/quote/{ticker_clean}"

        res = requests.get(url, timeout=5).json()

        results = res.get("results", [])

        if results and len(results) > 0:

            preco = results[0].get("regularMarketPrice")

            if preco is not None and preco > 0:

                return float(preco)

    except Exception:

        pass



    # 2. Busca secundária via Yahoo Finance

    symbol = ticker_clean if "." in ticker_clean else f"{ticker_clean}.SA"

    try:

        url = f"https://query1.finance.yahoo.com/v8/finance/chart/{symbol}"

        headers = {'User-Agent': 'Mozilla/5.0'}

        res = requests.get(url, headers=headers, timeout=5).json()

        result = res.get("chart", {}).get("result")

        if result and len(result) > 0:

            preco = result[0].get("meta", {}).get("regularMarketPrice")

            if preco is not None and preco > 0:

                return float(preco)

    except Exception:

        pass



    # 3. Fallback estático

    mock_prices = {

        "CXSE3": 20.42,

        "M2ST34": 7.11,

        "PETR4": 38.50,

        "VALE3": 61.20,

        "ITUB4": 34.10,

        "BBAS3": 27.80,

        "WEGE3": 42.15

    }

    return mock_prices.get(ticker_clean, 0.0)



# ------------------------------------------------------------------------------

# 2. FORMATADORES DE MOEDA (MÁSCARA VIVA EM REAL - R$)

# ------------------------------------------------------------------------------

def formatar_brl(valor):

    """Converte um float para a string R$ X.XXX,XX."""

    sinal = "-" if valor < 0 else ""

    val_abs = abs(valor)

    return f"{sinal}R$ {val_abs:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")



def converter_para_float(texto):

    """Converte entrada formatada em BRL de volta para float puro."""

    if not texto:

        return 0.0

    apenas_numeros = re.sub(r'\D', '', str(texto))

    if not apenas_numeros:

        return 0.0

    return float(apenas_numeros) / 100.0



# ------------------------------------------------------------------------------

# 3. COMPONENTE DA LINHA DO ATIVO

# ------------------------------------------------------------------------------

class LinhaAtivo:

    def __init__(self, callback_remover, index):

        self.callback_remover = callback_remover

        self.index = index

        self.preco_atual_val = 0.0



        # Lista Suspensa com Busca (Combobox)

        self.input_ticker = widgets.Combobox(

            placeholder="Ativo (ex: CXSE3)",

            options=ATIVOS_B3_POPULARES,

            ensure_option=False,

            layout=widgets.Layout(width='120px', margin='2px')

        )



        # Botão de Lupa

        self.btn_buscar = widgets.Button(

            description="🔍",

            layout=widgets.Layout(width='32px', height='28px', margin='2px', padding='0px')

        )



        # Preço Atual com destaque azul fixo (#38bdf8)

        self.lbl_preco_atual = widgets.HTML(

            value="<span style='color: #38bdf8; font-size: 13px; font-weight: 600;'>R$ 0,00</span>",

            layout=widgets.Layout(width='120px', margin='2px 0px 2px 10px', display='flex', align_items='center')

        )



        self.input_valor_aplicado = widgets.Text(

            value="R$ 0,00",

            placeholder="R$ 0,00",

            layout=widgets.Layout(width='135px', margin='2px 5px')

        )



        self.input_meta_preco = widgets.Text(

            value="R$ 0,00",

            placeholder="R$ 0,00",

            layout=widgets.Layout(width='135px', margin='2px 5px')

        )



        self.lbl_lucro_futuro = widgets.HTML(

            value="<span style='color: #3fb950; font-weight: 600; font-size: 13px;'>R$ 0,00</span>",

            layout=widgets.Layout(width='130px', margin='0px', display='flex', align_items='center')

        )



        self.btn_remover = widgets.Button(

            description="🗑️",

            button_style="",

            layout=widgets.Layout(width='36px', height='28px', margin='0px')

        )



        # Eventos

        self.input_ticker.observe(self._on_ticker_change, names='value')

        self.btn_buscar.on_click(lambda b: self.executar_busca())

        self.input_valor_aplicado.observe(self._on_change_aplicado, names='value')

        self.input_meta_preco.observe(self._on_change_meta, names='value')

        self.btn_remover.on_click(lambda b: self.callback_remover(self))



        self.bloqueado_formatacao = False



    def _on_ticker_change(self, change):

        if change.get('name') == 'value' and change.get('new'):

            self.executar_busca()



    def executar_busca(self):

        ticker = self.input_ticker.value.strip().upper()

        if not ticker:

            return



        preco = buscar_preco_mercado(ticker)

        if preco > 0:

            self.preco_atual_val = preco

            self.lbl_preco_atual.value = f"<span style='color: #38bdf8; font-size: 13px; font-weight: 600;'>{formatar_brl(self.preco_atual_val)}</span>"

        else:

            self.lbl_preco_atual.value = "<span style='color: #f85149; font-size: 11px;'>Não encontrado</span>"



        self.recalcular_formula()



    def _on_change_aplicado(self, change):

        if self.bloqueado_formatacao:

            return

        self.bloqueado_formatacao = True



        num_float = converter_para_float(change['new'])

        self.input_valor_aplicado.value = formatar_brl(num_float)



        self.bloqueado_formatacao = False

        self.recalcular_formula()



    def _on_change_meta(self, change):

        if self.bloqueado_formatacao:

            return

        self.bloqueado_formatacao = True



        num_float = converter_para_float(change['new'])

        self.input_meta_preco.value = formatar_brl(num_float)



        self.bloqueado_formatacao = False

        self.recalcular_formula()



    def recalcular_formula(self):

        """Fórmula da Planilha: Valor Futuro = Valor Aplicado * (Meta Preço / Preço Atual)"""

        val_aplicado = converter_para_float(self.input_valor_aplicado.value)

        meta_preco = converter_para_float(self.input_meta_preco.value)

        preco_atual = self.preco_atual_val



        if preco_atual > 0 and val_aplicado > 0 and meta_preco > 0:

            valor_futuro = val_aplicado * (meta_preco / preco_atual)

            lucro_bruto = valor_futuro - val_aplicado



            # Verde se o Valor Futuro for maior/igual ao Aplicado, Vermelho se menor

            cor = "#3fb950" if valor_futuro >= val_aplicado else "#f85149"



            self.lbl_lucro_futuro.value = (

                f"<div style='line-height: 1.2;'>"

                f"<span style='color: {cor}; font-weight: 600; font-size: 13px;'>{formatar_brl(valor_futuro)}</span><br>"

                f"<span style='color: #8b949e; font-size: 10px;'>Lucro: {formatar_brl(lucro_bruto)}</span>"

                f"</div>"

            )

        else:

            self.lbl_lucro_futuro.value = "<span style='color: #8b949e; font-size: 13px;'>R$ 0,00</span>"



    def render(self):

        box_ticker = widgets.HBox([self.input_ticker, self.btn_buscar], layout=widgets.Layout(margin='0px', padding='0px', align_items='center'))



        # Delineado azul englobando o valor e a lixeira

        box_lucro_delineado = widgets.HBox([

            self.lbl_lucro_futuro,

            self.btn_remover

        ], layout=widgets.Layout(

            border='1px solid #1d4ed8',

            border_radius='6px',

            padding='4px 8px',

            background_color='#0f172a',

            align_items='center',

            width='190px',

            margin='2px 0px 2px 10px'

        ))



        return widgets.HBox([

            box_ticker,

            self.lbl_preco_atual,

            self.input_valor_aplicado,

            self.input_meta_preco,

            box_lucro_delineado

        ], layout=widgets.Layout(

            align_items='center',

            padding='6px 0px',

            border_bottom='1px solid #21262d'

        ))



# ------------------------------------------------------------------------------

# 4. GESTOR DA CARTEIRA (ATÉ 10 ATIVOS)

# ------------------------------------------------------------------------------

class CarteiraManager:

    def __init__(self, nome_carteira):

        self.nome_carteira = nome_carteira

        self.linhas = []



        self.container_linhas = widgets.VBox([])



        self.btn_adicionar = widgets.Button(

            description="+ Adicionar Ativo",

            button_style="",

            layout=widgets.Layout(margin='12px 0px 0px 0px', width='140px')

        )

        self.btn_adicionar.on_click(self.adicionar_ativo)



        self.lbl_status_limite = widgets.HTML(

            value="",

            layout=widgets.Layout(margin='15px 0px 0px 10px')

        )



        header_html = widgets.HTML("""

        <div style="display: flex; background-color: #161b22; padding: 10px 0px; border-radius: 6px 6px 0px 0px; border-bottom: 1px solid #30363d; font-family: 'Segoe UI', sans-serif; font-size: 11px; font-weight: 600; color: #8b949e; text-transform: uppercase; letter-spacing: 0.5px; align-items: center;">

            <div style="width: 156px; padding-left: 8px;">ATIVO</div>

            <div style="width: 130px; color: #38bdf8;">PREÇO ATUAL</div>

            <div style="width: 145px;">VALOR APLICADO</div>

            <div style="width: 145px;">META PREÇO</div>

            <div style="width: 190px; color: #38bdf8; padding-left: 10px;">LUCRO PREVISTO</div>

        </div>

        """)



        self.layout_final = widgets.VBox([

            header_html,

            self.container_linhas,

            widgets.HBox([self.btn_adicionar, self.lbl_status_limite])

        ], layout=widgets.Layout(padding='10px 0px'))



    def adicionar_ativo(self, b=None):

        if len(self.linhas) >= 10:

            self.lbl_status_limite.value = "<span style='color: #f85149; font-size: 12px;'>Limite máximo de 10 ativos atingido.</span>"

            return



        self.lbl_status_limite.value = ""

        nova_linha = LinhaAtivo(self.remover_ativo, len(self.linhas))

        self.linhas.append(nova_linha)

        self.atualizar_render()



    def remover_ativo(self, linha_obj):

        if linha_obj in self.linhas:

            self.linhas.remove(linha_obj)

            self.lbl_status_limite.value = ""

            self.atualizar_render()



    def atualizar_render(self):

        self.container_linhas.children = [l.render() for l in self.linhas]



# ------------------------------------------------------------------------------

# 5. CONSTRUÇÃO DA INTERFACE PRINCIPAL

# ------------------------------------------------------------------------------

def criar_app_lucro_preco_alvo():

    css_custom = """

    <style>

        .widget-box { font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Roboto, sans-serif !important; }

        .widget-text input, .widget-combobox input {

            background-color: #0d1117 !important;

            color: #c9d1d9 !important;

            border: 1px solid #30363d !important;

            border-radius: 6px !important;

            padding: 4px 8px !important;

            font-size: 12px !important;

        }

        .widget-text input:focus, .widget-combobox input:focus {

            border-color: #38bdf8 !important;

            outline: none !important;

        }

        .widget-button {

            background-color: #21262d !important;

            color: #c9d1d9 !important;

            border: 1px solid #30363d !important;

            border-radius: 6px !important;

            font-size: 12px !important;

            transition: all 0.2s ease !important;

        }

        .widget-button:hover {

            background-color: #30363d !important;

            color: #ffffff !important;

            border-color: #8b949e !important;

        }



        .widget-tab > .p-TabBar .p-TabBar-tab {

            background-color: #161b22 !important;

            color: #8b949e !important;

            border: 1px solid #30363d !important;

            border-bottom: none !important;

            border-radius: 6px 6px 0px 0px !important;

            margin-right: 4px !important;

            padding: 8px 16px !important;

        }

        .widget-tab > .p-TabBar .p-TabBar-tab.p-mod-current {

            background-color: #0d1117 !important;

            color: #38bdf8 !important;

            border-color: #30363d !important;

            font-weight: 600 !important;

        }

    </style>

    """



    header_title = widgets.HTML("""

    <div style="background-color: #0d1117; padding: 18px 20px; border-bottom: 1px solid #21262d; margin-bottom: 15px; border-radius: 8px 8px 0px 0px;">

        <h2 style="margin: 0; color: #38bdf8; font-size: 20px; font-weight: 500; letter-spacing: -0.5px;">📈 Lucro de Preço Alvo</h2>

        <p style="margin: 4px 0 0 0; color: #8b949e; font-size: 12px;">Simulador minimalista de projeção de retorno por meta de cotação</p>

    </div>

    """)



    carteira_1 = CarteiraManager("Carteira A")

    carteira_2 = CarteiraManager("Carteira B")



    # Linha inicial na Carteira A com M2ST34

    carteira_1.adicionar_ativo()

    carteira_1.linhas[0].input_ticker.value = "M2ST34"

    carteira_1.linhas[0].executar_busca()

    carteira_1.linhas[0].input_valor_aplicado.value = "R$ 2.000,44"

    carteira_1.linhas[0].input_meta_preco.value = "R$ 15,00"



    carteira_2.adicionar_ativo()



    tabs = widgets.Tab()

    tabs.children = [carteira_1.layout_final, carteira_2.layout_final]

    tabs.set_title(0, '📋 Carteira A')

    tabs.set_title(1, '📋 Carteira B')



    app_container = widgets.VBox([

        widgets.HTML(css_custom),

        header_title,

        tabs

    ], layout=widgets.Layout(

        max_width='810px',

        margin='0 auto',

        padding='10px',

        background_color='#0d1117',

        border_radius='10px',

        border='1px solid #30363d'

    ))



    display(app_container)



criar_app_lucro_preco_alvo()


