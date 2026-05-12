# pyrefly: ignore [missing-import]
import json
from datetime import datetime
from pathlib import Path
from tkinter import messagebox

import customtkinter as ctk


# ========== CONFIGURACAO VISUAL ==========
ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")

CORES = {
    "fundo": "#0a0510",
    "painel": "#160d21",
    "card": "#1c1229",
    "card_hover": "#2d1b46",
    "primaria": "#9333ea",
    "primaria_hover": "#a855f7",
    "sucesso": "#22c55e",
    "sucesso_hover": "#15803d",
    "perigo": "#f43f5e",
    "perigo_hover": "#b91c1c",
    "neutra": "#6b21a8",
    "neutra_hover": "#7e22ce",
    "texto": "#f5f3ff",
    "texto_fraco": "#a78bfa",
}

FONTE_TITULO = ("Segoe UI", 24, "bold")
FONTE_SUBTITULO = ("Segoe UI", 13)
FONTE_SECAO = ("Segoe UI", 16, "bold")
FONTE_TEXTO = ("Segoe UI", 12)
FONTE_BOTAO = ("Segoe UI", 12, "bold")


# ========== DADOS DA LOJA ==========
PRODUTOS = [
    {"id": 1, "nome": "Camiseta", "preco": 79.99, "categoria": "Camisetas"},
    {"id": 2, "nome": "Calça Jeans", "preco": 199.99, "categoria": "Calças"},
    {"id": 3, "nome": "Tênis", "preco": 399.99, "categoria": "Calçados"},
    {"id": 4, "nome": "Jaqueta", "preco": 250.00, "categoria": "Casacos"},
    {"id": 5, "nome": "Regata", "preco": 70.00, "categoria": "Camisetas"},
    {"id": 6, "nome": "Calça Baggy", "preco": 299.99, "categoria": "Calças"},
    {"id": 7, "nome": "Calça Flare", "preco": 199.99, "categoria": "Calças"},
    {"id": 8, "nome": "Calça Reta", "preco": 199.99, "categoria": "Calças"},
    {"id": 9, "nome": "Camisa Social", "preco": 100.00, "categoria": "Camisas"},
]

REGIOES_FORTALEZA = [
    {"bairro": "Aldeota", "frete": 15.00},
    {"bairro": "Centro", "frete": 10.00},
    {"bairro": "Messejana", "frete": 20.00},
    {"bairro": "Benfica", "frete": 12.00},
]

FORMAS_PAGAMENTO = ["Pix", "Cartão de crédito", "Cartão de débito", "Dinheiro"]


# ========== VARIAVEIS GLOBAIS ==========
PASTA_SAIDA = Path(__file__).resolve().parent
carrinho = {}

app = None
status_itens_label = None
status_total_label = None
conteudo = None
coluna_esquerda = None
busca_var = None
catalogo_lista = None
chat_textbox = None
carrinho_lista = None
subtotal_label = None
total_label = None


# ========== FUNCOES UTILITARIAS ==========
def formatar_moeda(valor):
    texto = f"R$ {valor:,.2f}"
    return texto.replace(",", "X").replace(".", ",").replace("X", ".")


def buscar_produto(produto_id):
    return next((produto for produto in PRODUTOS if produto["id"] == produto_id), None)


def obter_regiao(bairro):
    return next(regiao for regiao in REGIOES_FORTALEZA if regiao["bairro"] == bairro)


def calcular_subtotal():
    return sum(
        item["produto"]["preco"] * item["quantidade"]
        for item in carrinho.values()
    )


def contar_itens():
    return sum(item["quantidade"] for item in carrinho.values())


# ========== MONTAGEM DA INTERFACE ==========
def criar_layout():
    global conteudo, coluna_esquerda

    app.grid_columnconfigure(0, weight=1)
    app.grid_rowconfigure(1, weight=1)

    criar_cabecalho()

    conteudo = ctk.CTkFrame(app, fg_color="transparent")
    conteudo.grid(row=1, column=0, sticky="nsew", padx=18, pady=(0, 18))
    conteudo.grid_columnconfigure(0, weight=1)
    conteudo.grid_columnconfigure(1, minsize=340)
    conteudo.grid_rowconfigure(0, weight=1)

    coluna_esquerda = ctk.CTkFrame(conteudo, fg_color="transparent")
    coluna_esquerda.grid(row=0, column=0, sticky="nsew", padx=(0, 12))
    coluna_esquerda.grid_columnconfigure(0, weight=1)
    coluna_esquerda.grid_rowconfigure(0, weight=3)
    coluna_esquerda.grid_rowconfigure(1, weight=2)

    criar_area_catalogo()
    criar_area_chat()
    criar_area_carrinho()


def criar_cabecalho():
    global status_itens_label, status_total_label

    cabecalho = ctk.CTkFrame(app, height=86, corner_radius=0, fg_color=CORES["painel"])
    cabecalho.grid(row=0, column=0, sticky="ew", padx=0, pady=(0, 18))
    cabecalho.grid_columnconfigure(0, weight=1)

    bloco_titulo = ctk.CTkFrame(cabecalho, fg_color="transparent")
    bloco_titulo.grid(row=0, column=0, sticky="w", padx=24, pady=16)

    ctk.CTkLabel(
        bloco_titulo,
        text="Loja C & IA",
        font=FONTE_TITULO,
        text_color=CORES["texto"],
    ).pack(anchor="w")

    ctk.CTkLabel(
        bloco_titulo,
        text="Moda, atendimento rápido e compra organizada",
        font=FONTE_SUBTITULO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", pady=(2, 0))

    bloco_status = ctk.CTkFrame(cabecalho, fg_color="transparent")
    bloco_status.grid(row=0, column=1, sticky="e", padx=24, pady=16)

    status_itens_label = ctk.CTkLabel(
        bloco_status,
        text="0 itens",
        font=FONTE_BOTAO,
        text_color=CORES["texto"],
        fg_color=CORES["card"],
        corner_radius=8,
        width=90,
        height=32,
    )
    status_itens_label.pack(side="left", padx=(0, 8))

    status_total_label = ctk.CTkLabel(
        bloco_status,
        text="Total: R$ 0,00",
        font=FONTE_BOTAO,
        text_color=CORES["texto"],
        fg_color=CORES["primaria"],
        corner_radius=8,
        width=150,
        height=32,
    )
    status_total_label.pack(side="left")


def criar_area_catalogo():
    global busca_var, catalogo_lista

    painel = ctk.CTkFrame(coluna_esquerda, corner_radius=12, fg_color=CORES["painel"])
    painel.grid(row=0, column=0, sticky="nsew", pady=(0, 12))
    painel.grid_columnconfigure(0, weight=1)
    painel.grid_rowconfigure(1, weight=1)

    topo = ctk.CTkFrame(painel, fg_color="transparent")
    topo.grid(row=0, column=0, sticky="ew", padx=16, pady=(14, 8))
    topo.grid_columnconfigure(1, weight=1)

    ctk.CTkLabel(
        topo,
        text="Catálogo",
        font=FONTE_SECAO,
        text_color=CORES["texto"],
    ).grid(row=0, column=0, sticky="w")

    busca_var = ctk.StringVar()

    busca_entry = ctk.CTkEntry(
        topo,
        textvariable=busca_var,
        placeholder_text="Buscar produto ou categoria",
        height=36,
        border_width=1,
        fg_color=CORES["fundo"],
    )
    busca_entry.grid(row=0, column=1, sticky="ew", padx=(16, 0))

    catalogo_lista = ctk.CTkScrollableFrame(
        painel,
        fg_color=CORES["fundo"],
        corner_radius=10,
    )
    catalogo_lista.grid(row=1, column=0, sticky="nsew", padx=16, pady=(0, 16))
    catalogo_lista.grid_columnconfigure(0, weight=1)

    busca_var.trace_add("write", lambda *_: mostrar_catalogo())


def criar_area_chat():
    global chat_textbox

    painel = ctk.CTkFrame(coluna_esquerda, corner_radius=12, fg_color=CORES["painel"])
    painel.grid(row=1, column=0, sticky="nsew")
    painel.grid_columnconfigure(0, weight=1)
    painel.grid_rowconfigure(1, weight=1)

    topo = ctk.CTkFrame(painel, fg_color="transparent")
    topo.grid(row=0, column=0, sticky="ew", padx=16, pady=(14, 8))
    topo.grid_columnconfigure(0, weight=1)

    ctk.CTkLabel(
        topo,
        text="Conversa",
        font=FONTE_SECAO,
        text_color=CORES["texto"],
    ).grid(row=0, column=0, sticky="w")

    acoes = ctk.CTkFrame(topo, fg_color="transparent")
    acoes.grid(row=0, column=1, sticky="e")

    ctk.CTkButton(
        acoes,
        text="Ver carrinho",
        width=110,
        height=34,
        font=FONTE_BOTAO,
        fg_color=CORES["neutra"],
        hover_color=CORES["neutra_hover"],
        command=ver_carrinho_no_chat,
    ).pack(side="left", padx=(0, 8))

    ctk.CTkButton(
        acoes,
        text="Suporte",
        width=90,
        height=34,
        font=FONTE_BOTAO,
        fg_color=CORES["neutra"],
        hover_color=CORES["neutra_hover"],
        command=abrir_suporte,
    ).pack(side="left", padx=(0, 8))

    ctk.CTkButton(
        acoes,
        text="Limpar",
        width=90,
        height=34,
        font=FONTE_BOTAO,
        fg_color=CORES["perigo"],
        hover_color=CORES["perigo_hover"],
        command=limpar_chat,
    ).pack(side="left")

    chat_textbox = ctk.CTkTextbox(
        painel,
        font=FONTE_TEXTO,
        wrap="word",
        corner_radius=10,
        fg_color=CORES["fundo"],
        text_color=CORES["texto"],
    )
    chat_textbox.grid(row=1, column=0, sticky="nsew", padx=16, pady=(0, 16))
    chat_textbox.configure(state="disabled")


def criar_area_carrinho():
    global carrinho_lista, subtotal_label, total_label

    carrinho_painel = ctk.CTkFrame(
        conteudo,
        width=340,
        corner_radius=12,
        fg_color=CORES["painel"],
    )
    carrinho_painel.grid(row=0, column=1, sticky="nsew")
    carrinho_painel.grid_propagate(False)
    carrinho_painel.grid_columnconfigure(0, weight=1)
    carrinho_painel.grid_rowconfigure(1, weight=1)

    ctk.CTkLabel(
        carrinho_painel,
        text="Seu carrinho",
        font=FONTE_SECAO,
        text_color=CORES["texto"],
    ).grid(row=0, column=0, sticky="w", padx=16, pady=(14, 8))

    carrinho_lista = ctk.CTkScrollableFrame(
        carrinho_painel,
        fg_color=CORES["fundo"],
        corner_radius=10,
    )
    carrinho_lista.grid(row=1, column=0, sticky="nsew", padx=16, pady=(0, 12))
    carrinho_lista.grid_columnconfigure(0, weight=1)

    resumo = ctk.CTkFrame(carrinho_painel, fg_color="transparent")
    resumo.grid(row=2, column=0, sticky="ew", padx=16, pady=(0, 14))
    resumo.grid_columnconfigure(0, weight=1)

    subtotal_label = ctk.CTkLabel(
        resumo,
        text="Subtotal: R$ 0,00",
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    )
    subtotal_label.grid(row=0, column=0, sticky="w")

    total_label = ctk.CTkLabel(
        resumo,
        text="Total: R$ 0,00",
        font=("Segoe UI", 18, "bold"),
        text_color=CORES["texto"],
    )
    total_label.grid(row=1, column=0, sticky="w", pady=(4, 12))

    ctk.CTkButton(
        resumo,
        text="Finalizar compra",
        height=44,
        font=FONTE_BOTAO,
        fg_color=CORES["sucesso"],
        hover_color=CORES["sucesso_hover"],
        command=abrir_checkout,
    ).grid(row=2, column=0, sticky="ew")


# ========== CATALOGO ==========
def mostrar_catalogo():
    for widget in catalogo_lista.winfo_children():
        widget.destroy()

    termo = busca_var.get().strip().lower()

    produtos_filtrados = [
        produto
        for produto in PRODUTOS
        if termo in produto["nome"].lower() or termo in produto["categoria"].lower()
    ]

    if not produtos_filtrados:
        ctk.CTkLabel(
            catalogo_lista,
            text="Nenhum produto encontrado.",
            font=FONTE_TEXTO,
            text_color=CORES["texto_fraco"],
        ).grid(row=0, column=0, padx=12, pady=20)
        return

    for linha, produto in enumerate(produtos_filtrados):
        criar_card_produto(linha, produto)


def criar_card_produto(linha, produto):
    card = ctk.CTkFrame(catalogo_lista, corner_radius=8, fg_color=CORES["card"])
    card.grid(row=linha, column=0, sticky="ew", padx=8, pady=6)
    card.grid_columnconfigure(0, weight=1)

    info = ctk.CTkFrame(card, fg_color="transparent")
    info.grid(row=0, column=0, sticky="ew", padx=14, pady=12)

    ctk.CTkLabel(
        info,
        text=produto["nome"],
        font=("Segoe UI", 14, "bold"),
        text_color=CORES["texto"],
    ).pack(anchor="w")

    ctk.CTkLabel(
        info,
        text=f'{produto["categoria"]}  |  ID {produto["id"]}',
        font=("Segoe UI", 11),
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", pady=(2, 0))

    ctk.CTkLabel(
        card,
        text=formatar_moeda(produto["preco"]),
        font=("Segoe UI", 14, "bold"),
        text_color=CORES["texto"],
    ).grid(row=0, column=1, padx=12, pady=12)

    ctk.CTkButton(
        card,
        text="Adicionar",
        width=108,
        height=36,
        font=FONTE_BOTAO,
        fg_color=CORES["primaria"],
        hover_color=CORES["primaria_hover"],
        command=lambda produto_id=produto["id"]: adicionar_produto(produto_id),
    ).grid(row=0, column=2, padx=(0, 14), pady=12)


# ========== CHAT ==========
def escrever_chat(remetente, mensagem):
    hora = datetime.now().strftime("%H:%M")
    chat_textbox.configure(state="normal")
    chat_textbox.insert("end", f"[{hora}] {remetente}: {mensagem}\n\n")
    chat_textbox.see("end")
    chat_textbox.configure(state="disabled")


def bot_fala(mensagem):
    escrever_chat("Assistente virtual", mensagem)


def usuario_fala(mensagem):
    escrever_chat("Você", mensagem)


def limpar_chat():
    chat_textbox.configure(state="normal")
    chat_textbox.delete("1.0", "end")
    chat_textbox.configure(state="disabled")
    bot_fala("Chat limpo. Como posso ajudar agora?")


def ver_carrinho_no_chat():
    usuario_fala("Ver carrinho")

    if not carrinho:
        bot_fala("Seu carrinho está vazio.")
        return

    linhas = ["Resumo do carrinho:"]

    for item in carrinho.values():
        produto = item["produto"]
        quantidade = item["quantidade"]
        total_item = produto["preco"] * quantidade
        linhas.append(f"- {quantidade}x {produto['nome']} = {formatar_moeda(total_item)}")

    linhas.append(f"Subtotal: {formatar_moeda(calcular_subtotal())}")
    bot_fala("\n".join(linhas))


# ========== CARRINHO ==========
def adicionar_produto(produto_id):
    produto = buscar_produto(produto_id)

    if produto is None:
        bot_fala("Produto não encontrado.")
        return

    if produto_id not in carrinho:
        carrinho[produto_id] = {"produto": produto, "quantidade": 0}

    carrinho[produto_id]["quantidade"] += 1
    usuario_fala(f"Adicionar {produto['nome']}")
    bot_fala(f"{produto['nome']} foi adicionado ao carrinho.")
    atualizar_resumo_carrinho()


def alterar_quantidade(produto_id, diferenca):
    if produto_id not in carrinho:
        return

    carrinho[produto_id]["quantidade"] += diferenca

    if carrinho[produto_id]["quantidade"] <= 0:
        nome = carrinho[produto_id]["produto"]["nome"]
        del carrinho[produto_id]
        bot_fala(f"{nome} foi removido do carrinho.")
    else:
        produto = carrinho[produto_id]["produto"]
        quantidade = carrinho[produto_id]["quantidade"]
        bot_fala(f"{produto['nome']} agora tem quantidade {quantidade}.")

    atualizar_resumo_carrinho()


def remover_produto(produto_id):
    if produto_id not in carrinho:
        return

    nome = carrinho[produto_id]["produto"]["nome"]
    del carrinho[produto_id]
    bot_fala(f"{nome} foi removido do carrinho.")
    atualizar_resumo_carrinho()


def atualizar_resumo_carrinho():
    for widget in carrinho_lista.winfo_children():
        widget.destroy()

    if not carrinho:
        ctk.CTkLabel(
            carrinho_lista,
            text="Carrinho vazio",
            font=FONTE_TEXTO,
            text_color=CORES["texto_fraco"],
        ).grid(row=0, column=0, padx=12, pady=20)
    else:
        for linha, produto_id in enumerate(carrinho):
            criar_card_carrinho(linha, produto_id)

    subtotal = calcular_subtotal()
    quantidade = contar_itens()
    palavra_item = "item" if quantidade == 1 else "itens"

    subtotal_label.configure(text=f"Subtotal: {formatar_moeda(subtotal)}")
    total_label.configure(text=f"Total: {formatar_moeda(subtotal)}")
    status_itens_label.configure(text=f"{quantidade} {palavra_item}")
    status_total_label.configure(text=f"Total: {formatar_moeda(subtotal)}")


def criar_card_carrinho(linha, produto_id):
    item = carrinho[produto_id]
    produto = item["produto"]
    quantidade = item["quantidade"]
    total_item = produto["preco"] * quantidade

    card = ctk.CTkFrame(carrinho_lista, corner_radius=8, fg_color=CORES["card"])
    card.grid(row=linha, column=0, sticky="ew", padx=8, pady=6)
    card.grid_columnconfigure(0, weight=1)

    ctk.CTkLabel(
        card,
        text=produto["nome"],
        font=("Segoe UI", 13, "bold"),
        text_color=CORES["texto"],
    ).grid(row=0, column=0, sticky="w", padx=12, pady=(10, 0))

    ctk.CTkLabel(
        card,
        text=f"{quantidade}x  |  {formatar_moeda(total_item)}",
        font=("Segoe UI", 11),
        text_color=CORES["texto_fraco"],
    ).grid(row=1, column=0, sticky="w", padx=12, pady=(2, 10))

    controles = ctk.CTkFrame(card, fg_color="transparent")
    controles.grid(row=0, column=1, rowspan=2, sticky="e", padx=10, pady=10)

    ctk.CTkButton(
        controles,
        text="-",
        width=30,
        height=30,
        font=FONTE_BOTAO,
        fg_color=CORES["neutra"],
        hover_color=CORES["neutra_hover"],
        command=lambda pid=produto_id: alterar_quantidade(pid, -1),
    ).pack(side="left", padx=(0, 4))

    ctk.CTkButton(
        controles,
        text="+",
        width=30,
        height=30,
        font=FONTE_BOTAO,
        fg_color=CORES["primaria"],
        hover_color=CORES["primaria_hover"],
        command=lambda pid=produto_id: alterar_quantidade(pid, 1),
    ).pack(side="left", padx=(0, 4))

    ctk.CTkButton(
        controles,
        text="X",
        width=30,
        height=30,
        font=FONTE_BOTAO,
        fg_color=CORES["perigo"],
        hover_color=CORES["perigo_hover"],
        command=lambda pid=produto_id: remover_produto(pid),
    ).pack(side="left")


# ========== SUPORTE ==========
def abrir_suporte():
    suporte_win = ctk.CTkToplevel(app)
    suporte_win.title("Suporte ao cliente")
    suporte_win.geometry("470x420")
    suporte_win.transient(app)
    suporte_win.grab_set()
    suporte_win.configure(fg_color=CORES["fundo"])

    ctk.CTkLabel(
        suporte_win,
        text="Suporte ao cliente",
        font=FONTE_SECAO,
        text_color=CORES["texto"],
    ).pack(anchor="w", padx=22, pady=(20, 8))

    ctk.CTkLabel(
        suporte_win,
        text="Mensagem",
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", padx=22)

    mensagem_entry = ctk.CTkTextbox(suporte_win, height=130, corner_radius=8)
    mensagem_entry.pack(fill="x", padx=22, pady=(4, 12))

    ctk.CTkLabel(
        suporte_win,
        text="Contato (e-mail ou WhatsApp)",
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", padx=22)

    contato_entry = ctk.CTkEntry(suporte_win, placeholder_text="Opcional", height=36)
    contato_entry.pack(fill="x", padx=22, pady=(4, 18))

    def enviar_suporte():
        mensagem = mensagem_entry.get("1.0", "end").strip()
        contato = contato_entry.get().strip()

        if not mensagem:
            messagebox.showwarning("Aviso", "Digite uma mensagem antes de enviar.")
            return

        arquivo = PASTA_SAIDA / "suporte_contatos.txt"

        with arquivo.open("a", encoding="utf-8") as f:
            f.write(
                f"{datetime.now().strftime('%d/%m/%Y %H:%M:%S')} | "
                f"Mensagem: {mensagem} | Contato: {contato or 'Não informado'}\n"
            )

        usuario_fala(f"Enviar suporte: {mensagem[:50]}")
        bot_fala(f"Sua mensagem foi registrada em {arquivo.name}.")
        suporte_win.destroy()

    ctk.CTkButton(
        suporte_win,
        text="Enviar mensagem",
        height=42,
        font=FONTE_BOTAO,
        fg_color=CORES["sucesso"],
        hover_color=CORES["sucesso_hover"],
        command=enviar_suporte,
    ).pack(fill="x", padx=22)


# ========== CHECKOUT ==========
def criar_campo(janela, label, placeholder):
    ctk.CTkLabel(
        janela,
        text=label,
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", padx=22)

    entrada = ctk.CTkEntry(janela, placeholder_text=placeholder, height=36)
    entrada.pack(fill="x", padx=22, pady=(4, 12))
    return entrada


def texto_resumo_checkout(bairro):
    subtotal = calcular_subtotal()
    frete = obter_regiao(bairro)["frete"]
    total = subtotal + frete

    return (
        f"Subtotal: {formatar_moeda(subtotal)}\n"
        f"Frete para {bairro}: {formatar_moeda(frete)}\n"
        f"Total final: {formatar_moeda(total)}"
    )


def criar_nota_fiscal(nome, telefone, endereco, regiao, pagamento):
    subtotal = calcular_subtotal()
    frete = obter_regiao(regiao)["frete"]
    total_final = subtotal + frete
    itens = []

    for item in carrinho.values():
        produto = item["produto"]
        quantidade = item["quantidade"]
        itens.append(
            {
                "id": produto["id"],
                "produto": produto["nome"],
                "categoria": produto["categoria"],
                "preco_unitario": produto["preco"],
                "quantidade": quantidade,
                "total_item": produto["preco"] * quantidade,
            }
        )

    return {
        "loja": "Loja C & IA",
        "data_compra": datetime.now().strftime("%d/%m/%Y %H:%M:%S"),
        "cliente": {
            "nome": nome,
            "telefone": telefone,
            "endereco": endereco,
            "regiao": regiao,
        },
        "pagamento": pagamento,
        "itens_comprados": itens,
        "subtotal": subtotal,
        "frete": frete,
        "total_pago": total_final,
    }


def salvar_nota_fiscal(nota):
    pasta_notas = PASTA_SAIDA / "notas_fiscais"
    pasta_notas.mkdir(exist_ok=True)

    data_arquivo = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
    arquivo = pasta_notas / f"nota_fiscal_{data_arquivo}.json"

    with arquivo.open("w", encoding="utf-8") as f:
        json.dump(nota, f, ensure_ascii=False, indent=4)

    return arquivo


def abrir_checkout():
    if not carrinho:
        bot_fala("Você não pode finalizar com o carrinho vazio.")
        return

    checkout_win = ctk.CTkToplevel(app)
    checkout_win.title("Finalizar compra")
    checkout_win.geometry("520x610")
    checkout_win.transient(app)
    checkout_win.grab_set()
    checkout_win.configure(fg_color=CORES["fundo"])

    ctk.CTkLabel(
        checkout_win,
        text="Finalizar compra",
        font=FONTE_SECAO,
        text_color=CORES["texto"],
    ).pack(anchor="w", padx=22, pady=(20, 12))

    nome_entry = criar_campo(checkout_win, "Nome do cliente", "Ex: Ana Souza")
    telefone_entry = criar_campo(checkout_win, "Telefone", "Ex: (85) 99999-9999")
    endereco_entry = criar_campo(checkout_win, "Endereço de entrega", "Rua, número e complemento")

    ctk.CTkLabel(
        checkout_win,
        text="Região de entrega",
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", padx=22)

    regiao_var = ctk.StringVar(value=REGIOES_FORTALEZA[0]["bairro"])
    regiao_menu = ctk.CTkOptionMenu(
        checkout_win,
        values=[regiao["bairro"] for regiao in REGIOES_FORTALEZA],
        variable=regiao_var,
        height=36,
        fg_color=CORES["card"],
        button_color=CORES["primaria"],
        button_hover_color=CORES["primaria_hover"],
    )
    regiao_menu.pack(fill="x", padx=22, pady=(4, 12))

    ctk.CTkLabel(
        checkout_win,
        text="Forma de pagamento",
        font=FONTE_TEXTO,
        text_color=CORES["texto_fraco"],
    ).pack(anchor="w", padx=22)

    pagamento_var = ctk.StringVar(value=FORMAS_PAGAMENTO[0])
    pagamento_menu = ctk.CTkOptionMenu(
        checkout_win,
        values=FORMAS_PAGAMENTO,
        variable=pagamento_var,
        height=36,
        fg_color=CORES["card"],
        button_color=CORES["primaria"],
        button_hover_color=CORES["primaria_hover"],
    )
    pagamento_menu.pack(fill="x", padx=22, pady=(4, 14))

    resumo_label = ctk.CTkLabel(
        checkout_win,
        text=texto_resumo_checkout(regiao_var.get()),
        justify="left",
        font=FONTE_TEXTO,
        text_color=CORES["texto"],
        fg_color=CORES["painel"],
        corner_radius=8,
        padx=14,
        pady=12,
    )
    resumo_label.pack(fill="x", padx=22, pady=(0, 16))

    def atualizar_resumo_por_regiao(*_):
        resumo_label.configure(text=texto_resumo_checkout(regiao_var.get()))

    def confirmar_compra():
        nome = nome_entry.get().strip()
        telefone = telefone_entry.get().strip()
        endereco = endereco_entry.get().strip()

        if not nome or not telefone or not endereco:
            messagebox.showwarning(
                "Dados incompletos",
                "Preencha nome, telefone e endereço antes de finalizar.",
            )
            return

        nota = criar_nota_fiscal(
            nome=nome,
            telefone=telefone,
            endereco=endereco,
            regiao=regiao_var.get(),
            pagamento=pagamento_var.get(),
        )
        arquivo = salvar_nota_fiscal(nota)

        usuario_fala(f"Finalizar compra para {regiao_var.get()}")
        bot_fala(
            f"Compra finalizada com sucesso. Total pago: "
            f"{formatar_moeda(nota['total_pago'])}. Nota salva em {arquivo.name}."
        )

        carrinho.clear()
        atualizar_resumo_carrinho()
        checkout_win.destroy()
        messagebox.showinfo("Compra finalizada", f"Nota fiscal salva em:\n{arquivo}")

    regiao_var.trace_add("write", atualizar_resumo_por_regiao)

    ctk.CTkButton(
        checkout_win,
        text="Confirmar compra",
        height=44,
        font=FONTE_BOTAO,
        fg_color=CORES["sucesso"],
        hover_color=CORES["sucesso_hover"],
        command=confirmar_compra,
    ).pack(fill="x", padx=22)


# ========== INICIALIZACAO DO PROGRAMA ==========
def iniciar_app():
    global app

    app = ctk.CTk()
    app.title("Loja C & IA - Assistente de vendas")
    app.geometry("1120x720")
    app.minsize(1000, 650)
    app.configure(fg_color=CORES["fundo"])

    criar_layout()
    mostrar_catalogo()
    atualizar_resumo_carrinho()
    bot_fala(
        "Olá! Seja bem-vindo à Loja C & IA. Escolha os produtos no catálogo, "
        "acompanhe o carrinho ao lado e finalize quando estiver pronto."
    )

    app.mainloop()


if __name__ == "__main__":
    iniciar_app()
