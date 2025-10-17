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

Em primeiro lugar convém criar o par de chaves, abra o terminal e cole:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

A saída deve conter algo assim:

> Generating public/private ALGORITHM key pair.

Pressione Enter para aceitar o local padrão do arquivo. Se você criou chaves SSH anteriormente, ssh-keygen pode pedir que você subescreva outra chave. Você pode escolher substituir ou renomear a chave anterior. 

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
> [!NOTE]
> Agora será necessário adcionar sua chave pública à sua conta do github, para que possa ser validada e utilizada mais facilmente no momento em que estiver instalando sua MÁQUINA VIRTUAL(VM), pois é nesse momento que a instalação pergunta se quer adicionar sua chave publica a o servidor, diminuindo a complexidade de instalação das chaves.

## Acessando a VM

Partindo do ponto onde você já está com sua VM instalada, nesse caso utilizando UBUNTU 24.04, temos que acessa-la utilizando SSh, cole o comando:
```
ssh nome_de_usuario@endereco_do_servidor
```
Você deverá seguir as instruções da sua distribuição. 

Pronto, ja podemos usar o terminal do sistema hospedeiro para digitar os comandos. 

É importante lembrar que, antes de executar qualquer tarefa no terminal linux, devemos realizar  as atualizações básicas, cole: 

```
sudo apt update
sudo apt upgrade -y
```
Com isso já podemos instalar o servidor WEB Apache2:
```
sudo apt install apache2 -y
```
Você pode verificar se o Apache está em execução acessando o endereço IP do seu servidor em um navegador da web. Você deverá ver a página de boas-vindas padrão do Apache.

A seguir instalamos o servidor de banco de dados MySql, também o PHP e dependências:
```
sudo apt install php libapache2-mod-php php-mysql -y
```
Após instalar o PHP e seu módulo Apache, reinicie o serviço Apache para que as alterações entrem em vigor:
```
sudo systemctl restart apache2 -y
```

Teste o PHP (opcional, mas recomendado):
Crie um arquivo PHP simples para verificar se o PHP está funcionando corretamente com o Apache:
```
sudo nano /var/www/html/info.php
```

Adicione o seguinte conteúdo ao "info.php" arquivo:
```
<?php
phpinfo();
?>
```

Salve e feche o arquivo. Em seguida, abra seu navegador e navegue até http://your_server_ip/info.php(substitua your_server_ippelo endereço IP real do seu servidor ou, localhost se estiver instalando localmente). Você deverá ver uma página detalhada exibindo sua configuração de PHP.
```
sudo apt install phpmyadmin php-mbstring -y
```

Alternativamente você pode simplesmente rodar o comando:
```
sudo apt install apache2 php php-mbstring libapache2-mod-php php-mysql mysql-server phpmyadmin -y
```

Após a conclusão devemos fazer a ligação entre o apache e o phpmyadmin para que possa funcionar:
```
sudo nano apache2.conf
```

e nesse arquivo devemos incluir o endereço:
```
include /etc/phpmyadmin/apache.conf
```

Feito isso devemos criar um usuario no mysql:
```
CREATE USER 'nome_usuario'@'%' IDENTIFIED BY 'senha';
```
Lembrando que o símbolo "&" serve como um coringa, pois pode assumir endereçõs_ip variados

GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'%';

Após a conclusão da instalação, você pode acessar o phpMyAdmin no seu navegador. Use o endereço http://seu_servidor/phpmyadmin, substituindo "seu_servidor" pelo nome do seu servidor ou endereço IP.
Faça login com o usuário root do MySQL e a senha que você configurou durante a instalação. 


Uma alernativa viável ao phpmyadmin é o mysql-workbench, que também é uma interfacee gráfica amigável, que facilita o acesso ao banco de dados, no seu host cole:
```
sudo apt install mysql-workbench-community
```
Caso não funcione você pode tentar:
```
sudo snap install mysql-workbench-community
```


