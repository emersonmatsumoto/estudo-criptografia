
# SSH com YubiKey via FIDO2 (ed25519-sk)

Este método utiliza o protocolo FIDO2 para transformar a tua YubiKey numa "Security Key" nativa do OpenSSH. A chave privada física nunca sai do dispositivo e exige um toque no botão para autorizar o acesso.

## 1. Pré-requisitos

* **Hardware:** YubiKey série 5 ou Security Key (NFC/Blue).
* **Software:** OpenSSH versão 8.2 ou superior (tanto no PC local como no servidor).

## 2. Gerar a Chave "Residente"

Usar uma chave **residente** é fundamental: permite que recuperes os ficheiros da chave em qualquer computador apenas ligando a YubiKey, sem precisar de backups manuais.

```bash
# O parâmetro -O resident é o que permite a recuperação posterior
ssh-keygen -t ed25519-sk -O resident -O application=ssh:yubikey-fido

```

* **PIN:** Se configuraste um PIN FIDO2 na YubiKey, ele será solicitado.
* **Touch:** Quando a luz da YubiKey piscar, toca no botão dourado.
* **Ficheiros:** Serão criados `id_ed25519_sk` e `id_ed25519_sk.pub` na pasta `~/.ssh/`.

---

## 3. Instalar a Chave no Servidor

Envia a tua chave pública para o servidor remoto:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub utilizador@servidor

```

---

## 4. Como Recuperar

Se você apagou o arquivo `id_ed25519_sk` ou reinstalou o sistema, pode recriar os arquivos físicos diretamente do hardware da YubiKey::

### Para recriar os arquivos no disco:

```bash
cd ~/.ssh
ssh-keygen -K

```

*Este comando vai "puxar" a chave da YubiKey e criar arquivos como `id_ed25519_sk_rk`.*

### Para carregar apenas na memória (sessão atual):

```bash
ssh-add -K

```

---

## 5. Como Usar no Dia a Dia

Para ligar ao servidor, basta ter a YubiKey inserida e correr:

```bash
ssh utilizador@servidor

```

O terminal dirá: `Confirm user presence for key ED25519-SK...`.
**Toque no botão da YubiKey para completar o login.**

---

## 6. Dicas de Configuração (~/.ssh/config)

Para não precisar especificar usuários ou caminhos toda vez que for conectar, edite o seu arquivo de configuração:

```text
Host meu-servidor
    HostName 1.2.3.4
    User o-teu-utilizador
    IdentityFile ~/.ssh/id_ed25519_sk

```

---

### Notas importantes:

* O sufixo `@openssh.com` na chave pública é normal e indica o uso do protocolo FIDO2.
* Se o comando `ssh-keygen -K` não encontrar nada, é porque a chave original não foi gerada com o parâmetro `-O resident`.
