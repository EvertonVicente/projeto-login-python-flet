# Documentação do Projeto

## Introdução
Este projeto é uma aplicação de interface gráfica para login e cadastro, utilizando a biblioteca `flet` para a construção da interface. A aplicação inclui elementos como caixas giratórias personalizadas e botões para interação do usuário.

## Vídeo do Projeto

<video width="320" height="240" controls>
  <source src="Projeto-login.mp4" type="video/mp4">
  Seu navegador não suporta o elemento de vídeo.
</video>

## Estrutura do Código

### 1. Importação de Bibliotecas
- `flet`: Biblioteca utilizada para a construção da interface gráfica.
- `time`, `pi`, `re`, `bcrypt`, `csv`, `os`: Bibliotecas padrão do Python.

### 2. Carregamento e Salvamento de Dados de Usuários
Funções `carregar_usuarios()` e `salvar_usuarios()` são responsáveis por carregar dados de usuários de um arquivo CSV e salvá-los.

### 3. Classe `SpinningBox`
Classe que define uma caixa giratória personalizada para a interface.

### 4. Função `spin_boxes(page)`
Função para animar as caixas giratórias na interface.

### 5. Função `main(page)`
Função principal que cria a interface de login e cadastro.

## Elementos da Interface

### 1. Login
- **Campos:**
  - E-mail
  - Senha
- **Botões:**
  - Login
  - Recuperar Senha
  - Mostrar/Ocultar Senha

### 2. Cadastro
- **Campos:**
  - Nome
  - E-mail
  - Senha
- **Botões:**
  - Cadastrar
  - Mostrar/Ocultar Senha

### 3. Caixas Giratórias
Três caixas giratórias personalizadas com diferentes ângulos e direções de rotação.

### 4. Feedbacks
Mensagens de erro para validações de entrada. Mensagens de boas-vindas após login ou cadastro.

## Funcionalidades

### 1. Login
   - Validação de e-mail.
   - Verificação de existência do usuário.
   - Verificação de senha utilizando bcrypt.

### 2. Cadastro
   - Validação de e-mail único.
   - Validação de preenchimento de nome e senha.
   - Hashing da senha antes de salvar.

### 3. Recuperação de Senha
   - Simples mensagem indicando que o usuário receberá um e-mail de recuperação.

### 4. Mostrar/Ocultar Senha
   - Funcionalidade para exibir ou ocultar a senha nos campos de entrada.

### 5. Caixas Giratórias
   - Animação contínua das caixas giratórias na interface.

## Utilização
Para executar o projeto, basta rodar o script. A interface será aberta em um navegador web.

```bash
import flet
from flet import *
import time
from math import pi
import re
import bcrypt
import csv
import os

# Carregar dados de usuários do arquivo CSV
usuarios = []

def carregar_usuarios():
    global usuarios
    try:
        with open('usuarios.csv', mode='r') as arquivo_csv:
            leitor = csv.DictReader(arquivo_csv)
            for linha in leitor:
                usuarios.append(linha)
    except FileNotFoundError:
        # Se o arquivo não existir, comece com uma lista vazia
        usuarios = []

# Salvar dados de usuários no arquivo CSV
def salvar_usuarios():
    with open('usuarios.csv', mode='w', newline='') as arquivo_csv:
        campos = ['nome', 'email', 'senha']
        escritor = csv.DictWriter(arquivo_csv, fieldnames=campos)
        escritor.writeheader()
        for usuario in usuarios:
            escritor.writerow(usuario)

# Classe para criar uma caixa giratória personalizada
class SpinningBox(UserControl):
    def __init__(self, border_color, bg_color, rotate_angle, direction):
        self.border_color = border_color
        self.bg_color = bg_color
        self.rotate_angle = rotate_angle
        self.direction = direction
        self.rotation_angle = 0
        super().__init__()

    def build(self):
        return Container(
            width=48,
            height=48,
            border=border.all(2.5, self.border_color),
            bgcolor=self.bg_color,
            border_radius=2,
            rotate=transform.Rotate(self.rotation_angle, alignment.center),
            animate_rotation=animation.Animation(700, "easeInOut"),
        )

# Função para animar as caixas giratórias
def spin_boxes(page):
    spinning_boxes = page.controls[0].content.content.controls[1].controls

    while True:
        for box in spinning_boxes:
            try:
                box.rotation_angle += (pi / 180) * box.direction
                box.rotate = transform.Rotate(box.rotation_angle, alignment.center)
                box.update()
            except Exception as e:
                pass
        time.sleep(0.01)

# Função principal que cria a interface de login e cadastro
def main(page: Page):
    page.horizontal_alignment = "center"
    page.vertical_alignment = "center"
    page.bgcolor = "#1f262f"

    mostrar_senha_login = False
    mostrar_senha_cadastro = False

    def login(e):
        email = entrada_email.value
        senha = entrada_senha.value

        # Validação do e-mail
        if not re.match(r"[^@]+@[^@]+\.[^@]+", email):
            entrada_email.error_text = "Por favor, insira um endereço de e-mail válido."
            entrada_email.style = "color: red"
            page.update()
        else:
            entrada_email.error_text = None  # Limpe qualquer mensagem de erro anterior
            entrada_senha.error_text = None

            # Verificar se o usuário existe
            if any(usuario['email'] == email for usuario in usuarios):
                usuario = next(usuario for usuario in usuarios if usuario['email'] == email)
                senha_armazenada = usuario['senha']

                # Verificar se a senha está correta
                if bcrypt.checkpw(senha.encode('utf-8'), senha_armazenada.encode('utf-8')):
                    page.clean()
                    page.add(Text(f"Olá, {email}\nBem-vindo à nossa aplicação"))
                else:
                    entrada_senha.value = ''  # Limpe a senha
                    entrada_senha.error_text = "Senha incorreta."
                    entrada_senha.style = "color: red"  # Defina a cor do texto como vermelho
                    page.update()
            else:
                entrada_senha.value = ''
                entrada_senha.error_text = "Usuário não encontrado."
                entrada_senha.style = "color: red"
                page.update()
                page.add(Text("Usuário não encontrado. Por favor, faça o cadastro."))

    def cadastrar_usuario(e):
        nome = entrada_nome.value
        email = entrada_email_cadastro.value
        senha = entrada_senha_cadastro.value

        # Validação do e-mail
        if not re.match(r"[^@]+@[^@]+\.[^@]+", email):
            entrada_email_cadastro.error_text = "Por favor, insira um endereço de e-mail válido."
            entrada_email_cadastro.style = "color: red"
            page.update()

        else:
            entrada_email_cadastro.error_text = None  # Limpe qualquer mensagem de erro anterior
            entrada_senha_cadastro.error_text = None

            # Verificar se o e-mail já está em uso
            if any(usuario['email'] == email for usuario in usuarios):
                entrada_email_cadastro.error_text = "Este e-mail já está sendo usado."
                page.update()

            elif not nome or not nome:
                entrada_nome.error_text = "Por favor, preencha o nome."
                page.update()

            elif not senha or not senha:
                entrada_senha_cadastro.error_text = "Por favor, preencha a senha."
                page.update()

            else:
                # Criar novo usuário
                salt = bcrypt.gensalt()
                hashed_senha = bcrypt.hashpw(senha.encode('utf-8'), salt)
                novo_usuario = {'nome': nome, 'email': email, 'senha': hashed_senha.decode('utf-8')}
                usuarios.append(novo_usuario)
                salvar_usuarios()

                page.clean()
                page.add(Text(f"Usuário cadastrado com sucesso. Bem-vindo, {nome}!"))

    def recuperar_senha(e):
        page.clean()
        page.add(Text("Você receberá um e-mail de recuperação de senha em breve."))

    def toggle_mostrar_senha_login(e):
        nonlocal mostrar_senha_login
        mostrar_senha_login = not mostrar_senha_login
        entrada_senha.password = not mostrar_senha_login
        page.update()

    def toggle_mostrar_senha_cadastro(e):
        nonlocal mostrar_senha_cadastro
        mostrar_senha_cadastro = not mostrar_senha_cadastro
        entrada_senha_cadastro.password = not mostrar_senha_cadastro
        page.update()

    # Definir elementos da interface
    entrada_email = TextField(label="Digite o seu e-mail")
    entrada_senha = TextField(label="Digite a sua senha", password=True)
    entrada_email_cadastro = TextField(label="Digite o seu e-mail para cadastro")
    entrada_nome = TextField(label="Digite o seu nome")
    entrada_senha_cadastro = TextField(label="Digite a sua senha para cadastro", password=True)

    mostrar_senha_login_button = Row([
        ElevatedButton("Mostrar senha", on_click=toggle_mostrar_senha_login)],
        spacing=12, alignment=MainAxisAlignment.END)

    mostrar_senha_cadastro_button = Row([
        ElevatedButton("Mostrar senha", on_click=toggle_mostrar_senha_cadastro)],
        spacing=12, alignment=MainAxisAlignment.END)

    recuperar_senha_button = Row([
        ElevatedButton("Recuperar Senha", on_click=recuperar_senha),
    ], spacing=12, alignment=MainAxisAlignment.CENTER)

    botoes_login = Row([
        ElevatedButton("Login", on_click=login),
        recuperar_senha_button,
        mostrar_senha_login_button
    ], spacing=12, alignment=MainAxisAlignment.START)

    botoes_cadastro = Row([
        ElevatedButton("Cadastrar", on_click=cadastrar_usuario),
        mostrar_senha_cadastro_button
    ], spacing=12, alignment=MainAxisAlignment.START)

    cadastro_layout = Column([
        Text(" "),
        Text("Não tem uma conta? Faça o cadastro abaixo."),
        entrada_nome,
        entrada_email_cadastro,
        entrada_senha_cadastro,
        botoes_cadastro
    ], spacing=12)

    login_layout = Column([
        Row([Text(value="Seja Bem-Vindo!", style="headlineMedium")], alignment=MainAxisAlignment.CENTER),
        Text(" "),
        Text("Acesse sua Conta"),
        entrada_email,
        entrada_senha,
        botoes_login
    ], spacing=12)

    # Criar a interface gráfica
    page.add(
        Card(
            width=408,
            height=800,
            elevation=15,
            content=Container(
                bgcolor="#23262a",
                border_radius=6,
                content=Column(
                    horizontal_alignment=CrossAxisAlignment.CENTER,
                    controls=[
                        Divider(height=40, color='transparent'),
                        Stack(
                            controls=[
                                SpinningBox("#FFFFFF", None, 0, 1),
                                SpinningBox("#FFFFFF", None, pi, -1),
                                SpinningBox("#FFFFFF", None, pi, -2),
                            ]
                        ),
                        Divider(height=20, color="transparent"),
                        Column(
                            alignment=MainAxisAlignment.CENTER,
                            spacing=5,
                            controls=[
                                login_layout,
                                cadastro_layout
                            ],
                        ),
                    ],
                ),
            ),
        )
    )

    page.update()
    spin_boxes(page)

if __name__ == "__main__":
    carregar_usuarios()
    flet.app(target=main, view=flet.WEB_BROWSER)
