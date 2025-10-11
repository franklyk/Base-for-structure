# Extrutura básica de um servidor WEB.

## Requisitos:

- VirtualBox;
- Máquina Virtual (VM);
- Par de Chaves (PÚBLICA/PRIVADA);
- Servidor WEB (APACHE);
- Servidor de banco de dados (MYSQL);
- php 8.3;
- Servidor SSH (openSSH-server);
- Interface grafica para gerenciamento da base de dados (phpMyAdmin);

Em primeiro lugar devemos criar o par de chaves abra o terminal e cole:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

A saída deve conter algo assim:

> Generating public/private ALGORITHM key pair.

Pressione Enter para aceitar o local padrão do arquivo. Se você criou chaves SSH anteriormente, ssh-keygen pode pedir que você subescreva outra chave. Você pode escolher substituir ou renomear a shave anterior. 

> Enter a file in which to save the key (/home/YOU/.ssh/id_ALGORITHM):[Press enter]

> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]

## Adicionar sua chave SSH ao ssh-agent

Antes de adicionar uma nova chave SSH ao agente para gerenciar suas chaves, você deve verificar as chaves SSH existentes e gerado uma nova chave SSH.

1- Inicie o ssh-agent em segundo plano.
```
eval "$(ssh-agent -s)"
```

Verifique a saida:
> Agent pid 59566


2- Adicione sua chave SSH privada ao ssh-agent.
```
ssh-add ~/.ssh/id_ed25519
```
    Agora será necessário adcionar sua chave pública à sua conta do github, para que possa ser validada, e, utilizada com maior facilidade no momento em que estiver instalando sua MÁQUINA VIRTUAL(VM); 
    
