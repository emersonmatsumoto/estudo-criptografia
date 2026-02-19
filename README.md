# Configuração de SSH com Chave Protegida por TPM 2.0
Este guia descreve como gerar e utilizar chaves criptográficas armazenadas no hardware (TPM) para autenticação SSH, utilizando o padrão PKCS#11.

1. Instalação e Permissões
Instale as ferramentas necessárias e garanta que seu usuário tenha acesso ao hardware do TPM sem precisar de sudo.

# Instalação dos pacotes
```console
sudo apt update
sudo apt install tpm2-tools tpm2-pkcs11-tools libtpm2-pkcs11-1 openssh-client
```

# Adicionar usuário ao grupo tss para acessar o /dev/tpmrm0
```console
sudo usermod -aG tss $USER
Nota: É necessário reiniciar o computador ou fazer logout/login para que a permissão de grupo seja aplicada.
```

2. Configuração do Repositório (Database)

# Inicializar o repositório
```console
tpm2_ptool init
```

3. Criação do Token e da Chave
O Token funciona como um contêiner (cartão virtual) e a Chave é o par criptográfico gerado dentro do TPM.

# 1. Criar o Token
```console
tpm2_ptool addtoken --pid=1 --label=ssh_token --userpin=1234 --sopin=12345678
```

# 2. Gerar a chave RSA 2048 dentro do TPM
```console
tpm2_ptool addkey --label=ssh_token --userpin=1234 --algorithm=rsa2048
```

4. Integração com SSH
Para usar a chave, precisamos apontar para a biblioteca que faz a ponte entre o SSH e o TPM.

Visualizar a chave pública
```console
ssh-keygen -D /usr/lib/x86_64-linux-gnu/pkcs11/libtpm2_pkcs11.so
```

Copiar a chave para o servidor remoto
```console
ssh-copy-id -i <(ssh-keygen -D /usr/lib/x86_64-linux-gnu/pkcs11/libtpm2_pkcs11.so) usuario@servidor
```

Acessar o servidor
```console
ssh -I /usr/lib/x86_64-linux-gnu/pkcs11/libtpm2_pkcs11.so usuario@servidor
```

5. Automatização (Opcional)
Para não precisar digitar o caminho da biblioteca toda vez, edite seu arquivo ~/.ssh/config:

```
Host meu-servidor
    HostName 192.168.1.100
    User seu-usuario
    PKCS11Provider /usr/lib/x86_64-linux-gnu/pkcs11/libtpm2_pkcs11.so
```

Agora você pode conectar apenas com: ssh meu-servidor.

Dicas de Manutenção
Listar objetos: 
```console
tpm2_ptool listobj
```

Backup: Basta copiar a pasta `~/.tpm2-pkcs11/`. Lembre-se que o backup só funciona neste mesmo chip TPM.

Remover chave: Use o comando `tpm2_ptool objdel --id <ID>` (o ID você encontra no listobj).
