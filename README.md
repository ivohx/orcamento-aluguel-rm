import csv

# Valores base
VALORES = {
    "apartamento": 700.00,
    "casa": 900.00,
    "estudio": 1200.00
}

# Adicionais
ADICIONAIS = {
    ("apartamento", 2): 200.00,
    ("casa", 2): 250.00
}

# Valores fixos
VALOR_CONTRATO = 2000.00
PARCELAS_CONTRATO = 5

# Função de leitura de opção
def leia_opcao(mensagem, opcoes):
    while True:
        resposta = input(f"{mensagem} ({'/'.join(opcoes)}): ").strip().lower()
        if resposta in opcoes:
            return resposta
        else:
            print("Opção inválida, tente novamente.")

print("=== SISTEMA DE ORÇAMENTO DE ALUGUEL - IMOBILIÁRIA R.M ===")

# Escolha do tipo de imóvel
tipo = leia_opcao("Tipo de imóvel", ["apartamento", "casa", "estudio"])

# Número de quartos
if tipo in ["apartamento", "casa"]:
    quartos = int(input("Número de quartos (1 ou 2): "))
    if quartos not in [1, 2]:
        quartos = 1
else:
    quartos = 1

# Cálculo inicial
mensal = VALORES[tipo]

# Adicionais por quarto
if (tipo, quartos) in ADICIONAIS:
    mensal += ADICIONAIS[(tipo, quartos)]

# Garagem ou estacionamento
if tipo in ["apartamento", "casa"]:
    garagem = leia_opcao("Deseja incluir garagem?", ["sim", "nao"])
    if garagem == "sim":
        mensal += 300.00

elif tipo == "estudio":
    vagas = leia_opcao("Deseja adicionar vagas de estacionamento?", ["sim", "nao"])
    if vagas == "sim":
        mensal += 250.00  # 2 vagas inclusas
        adicionais = input("Deseja adicionar mais vagas? Informe a quantidade (0 para nenhuma): ")
        try:
            adicionais = int(adicionais)
            if adicionais > 0:
                mensal += adicionais * 60.00
        except ValueError:
            print("Valor inválido. Nenhuma vaga extra adicionada.")

# Desconto para apartamento sem crianças
if tipo == "apartamento":
    criancas = leia_opcao("Possui crianças?", ["sim", "nao"])
    if criancas == "nao":
        mensal -= mensal * 0.05  # 5% de desconto

# Contrato
valor_contrato = VALOR_CONTRATO
parcela_contrato = valor_contrato / PARCELAS_CONTRATO

# Exibir resumo
print("\n=== ORÇAMENTO GERADO ===")
print(f"Tipo de imóvel: {tipo.capitalize()}")
print(f"Quartos: {quartos}")
print(f"Valor mensal: R$ {mensal:.2f}")
print(f"Valor do contrato: R$ {valor_contrato:.2f} (em até {PARCELAS_CONTRATO}x de R$ {parcela_contrato:.2f})")

# Cálculo das 12 parcelas
meses = []
for mes in range(1, 13):
    total_mes = mensal + (parcela_contrato if mes <= PARCELAS_CONTRATO else 0)
    meses.append({
        "mes": mes,
        "mensalidade": mensal,
        "parcela_contrato": parcela_contrato if mes <= PARCELAS_CONTRATO else 0,
        "total": total_mes
    })

print("\n=== PARCELAS MENSAL + CONTRATO ===")
for item in meses:
    print(f"Mês {item['mes']:2d}: R$ {item['total']:.2f}")

# Gerar CSV
gerar_csv = leia_opcao("\nDeseja gerar arquivo CSV com as 12 parcelas?", ["sim", "nao"])
if gerar_csv == "sim":
    nome_csv = input("Nome do arquivo (padrão: orcamento_12_parcelas.csv): ").strip()
    if not nome_csv:
        nome_csv = "orcamento_12_parcelas.csv"
    if not nome_csv.lower().endswith(".csv"):
        nome_csv += ".csv"

    with open(nome_csv, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["mes", "mensalidade", "parcela_contrato", "total"])
        for it in meses:
            writer.writerow([
                it["mes"],
                f"{it['mensalidade']:.2f}",
                f"{it['parcela_contrato']:.2f}",
                f"{it['total']:.2f}"
            ])

    print(f"\nArquivo CSV gerado: {nome_csv}")

print("\nOrçamento gerado com sucesso. Obrigado por utilizar a Imobiliária R.M!")
