# 📈 Lucro de Preço Alvo (B3)

Simulador interativo e minimalista feito em Python para rodar direto no **Google Colab**, criado para calcular quanto seu investimento na bolsa (B3) vai render se uma ação ou BDR atingir o preço-alvo que você estipular.

## O que ele faz
- **Cotação em tempo real:** Busca automaticamente o preço atual dos ativos da B3 usando a API da **Brapi** (com backup do **Yahoo Finance**).
- **Cálculo de projeção:** Você informa quanto investiu e qual é a sua meta de preço; o painel calcula na hora o valor futuro da posição e o lucro (ou prejuízo) bruto.
- **Gestão em abas:** Permite simular até **2 carteiras separadas** (Carteira A e Carteira B) com até 10 ativos em cada uma.
- **Interface limpa:** Feito com `ipywidgets` e tema escuro, com formatação automática em Real (`R$`) enquanto você digita.

## Como rodar
Basta colar o código em uma célula do **Google Colab** e executar. As dependências (`ipywidgets` e `requests`) são instaladas automaticamente.
