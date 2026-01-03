# Extrutura básica de um servidor WEB.

O modelo a ser implementado é meramente um teste mas com um potencial de ser usado em produção visando a configuração segura, código limpo, e boas práticas.

## Requisitos:

- VirtualBox;
- Máquina Virtual (Ubuntu 24.04);
- Par de Chaves (PÚBLICA/PRIVADA);
- Servidor WEB (APACHE);
- Servidor de banco de dados (MYSQL);
- php 8.3;
- Servidor SSH (openSSH-server);
- Interface grafica para gerenciamento da base de dados (phpMyAdmin ou mysql workbench);


As primeiras configurações devem ser feitas no host criando um par de chaves. Abra o terminal e cole:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

A saída deve conter algo parecido com:

> Generating public/private ALGORITHM key pair.

Pressione Enter para aceitar o local padrão do arquivo. Se você criou chaves SSH anteriormente, ssh-keygen pode lhe perguntar se você quer subescrever a chave antiga. Você pode escolher substituir ou renomear a chave anterior. 

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
> Agora será necessário adcionar sua chave pública à sua conta do github, para que possa ser validada e utilizada mais facilmente no momento em que estiver instalando sua MÁQUINA VIRTUAL(VM), pois é nesse momento que a instalação pergunta se quer adicionar sua chave publica a o servidor, diminuindo a complexidade de instalação das chaves. Para ver e copiar a chave cole:

```
cat ~/.ssh/id_ed25519.pub
```

## Acessando a VM

Partindo do ponto onde você já está com sua VM instalada e com a chave publica devidamente configurada, devemos acessa-la utilizando SSh. Cole o comando:
```
ssh nome_de_usuario@endereco_do_servidor
```
Você deverá seguir as instruções da sua distribuição. 

Pronto, já podemos usar o terminal do host para digitar os comandos. 

É importante lembrar que, antes de executar qualquer tarefa no terminal linux, devemos realizar as atualizações básicas, cole: 

```
sudo apt update
sudo apt upgrade -y

```
## Instalar as ferramentas

Com isso já podemos instalar o servidor WEB Apache2:
```
sudo apt install apache2 -y
```

Você pode verificar se o Apache está em execução acessando o endereço IP do seu servidor em um navegador web. Você deverá ver a página de boas-vindas padrão do Apache.

A seguir instalamos o servidor de banco de dados MySql, também o PHP e dependências:
```
sudo apt install php libapache2-mod-php php-mysql -y
```
Após instalar o PHP e seu módulo Apache, reinicie o serviço Apache para que as alterações entrem em vigor:
```
sudo systemctl restart apache2
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

Feito isso podemos instalr a interface gráfica para gerenciar o banco de dados:
```
sudo apt install phpmyadmin php-mbstring -y
```

Podemos resumir os passos anteriores, você pode simplesmente rodar o comando:
```
sudo apt install apache2 php php-mbstring libapache2-mod-php php-mysql mysql-server phpmyadmin -y
```

Após a conclusão devemos fazer a ligação entre o apache e o phpmyadmin para que possa funcionar:
```
sudo nano /etc/apache2/apache2.conf
```

e nesse arquivo devemos incluir ( de preferência na última linha):
```
include /etc/phpmyadmin/apache.conf
```

## Criar um novo usuário MySQL (Recomendado)

Feito isso devemos criar um usuario no mysql e lhe fornecer as permissões necessárias:
```
sudo mysql -u root

CREATE USER 'nome_usuario'@'%' IDENTIFIED BY 'senha'; 

GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'%';

FLUSH PRIVILEGES;
```

## Testando a conclusão da instalação

Se os passos acima foram seguidos corretamente, você podera acessar o phpMyAdmin no seu navegador. Use o endereço http://meu_site/phpmyadmin, substituindo "meu_site" pelo seu nome de domínio ou endereço IP. Faça login com o usuário e senha que você criou no MySQL. 

Normalmente o método de autenticação padrão para o root usuário costuma ser o auth_socket plugin, que depende das credenciais do sistema operacional em vez de uma senha padrão para conexões locais. Se você precisar usar o usuário root para seu aplicativo, poderá alterar seu método de autenticação para mysql_native_password( ou caching_sha2_password dependendo da versão), o que permite conexões baseadas em senha. Se for esse o caso (Somente para testes em ambiente estritamente local)deveremos em primairo lugar acessar a VM via ssh e digitar o comando:
``` 
sudo mysql -u root
```

Altere o método de autenticação do usuário root e defina a senha para vazia:
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_new_password';
FLUSH PRIVILEGES;
EXIT;
```

Por padrão o phpMyAdmin bloqueia o acesso com senha vazia, então vamos mudar isso também:
```
sudo nano /etc/phpmyadmin/config.inc.php
```

Encontre a linha:
 > //$cfg['Servers'][$i]['AllowNoPassword'] = TRUE;

Retire as barras de comentário

Reinicie o serviço MySQL para que as alterações entrem em vigor:
```
sudo systemctl restart mysql 
```

> [!NOTE]
> LEMBRE-SE DE READAPTAR TODAS AS CONFIGURAÇÕES QUANDO FOR ENVIAR ESTE PROJETO PARA PRODUÇÃO POIS TUDO ISSO PÕE EM RISCO A SEGURANÇA DO PROJETO


Podemos então testar o acesso do usuário root ao phpḾyAdmin (com a senha vazia)

Uma alernativa viável ao phpmyadmin é o mysql-workbench, que também é uma interfacee gráfica amigável, que facilita o acesso ao banco de dados. No seu host cole:
```
sudo apt install mysql-workbench-community
```
Caso não funcione você pode tentar:
```
sudo snap install mysql-workbench-community
```


# Conversando Sobre Segurança


## Passo 1: Localizar ou Criar o Arquivo do Virtual Host

Os arquivos de sites ficam em /etc/apache2/sites-available/.

Você deverá criar um arquivo com o nome do seu site contendo o seguinte conteúdo (pode excuir os comentários):
```
# --------------------------------------------------
## CONFIGURAÇÃO HTTP (Redireciona tudo para HTTPS)
# --------------------------------------------------


<VirtualHost *:80>
    ServerName seusite.com.br
    ServerAlias www.seusite.com.br
    DocumentRoot /var/www/seusite

    ServerAdmin webmaster@seusite.com.br

    # Redirecionamento permanente para HTTPS (Melhor para SEO)
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]

    # Logs customizados para porta 80
    ErrorLog ${APACHE_LOG_DIR}/seusite_error.log
    CustomLog ${APACHE_LOG_DIR}/seusite_access.log combined
</VirtualHost>

# --------------------------------------------------
# CONFIGURAÇÃO HTTPS (Principal)
# --------------------------------------------------


<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName seusite.com.br
    ServerAlias www.seusite.com.br
    DocumentRoot /var/www/seusite

    ServerAdmin webmaster@seusite.com.br

    # Configurações de SSL (ASSUMindo que você já tem certificados)
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/seusite.com.br/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/seusite.com.br/privkey.pem
    
    # Logs customizados para porta 443
    ErrorLog ${APACHE_LOG_DIR}/seusite_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/seusite_ssl_access.log combined

    # Bloco Directory: Onde suas regras do .htaccess vão
    <Directory /var/www/html/seusite>
        Options FollowSymLinks Indexes
        # .htaccess desativado
        AllowOverride None
        Require all granted
        
        # Regras de Reescrita (Exemplo Laravel/WordPress)
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^(.*$) index.php?url=$1 [QSA L]
        
        # Configurações extras de segurança HTTP Headers (Opcional, mas avançado)
        Header set X-Content-Type-Options "nosniff"
        Header set X-Frame-Options "SAMEORIGIN"
        Header set X-XSS-Protection "1; mode=block"
    </Directory>

</VirtualHost>
</IfModule>

```



### O que foi adicionado?

    Redirecionamento HTTP para HTTPS (Porta 80 para 443): 
        Essencial para segurança e SEO. Todo tráfego não seguro é automaticamente movido para a versão segura do site.


    Configuração de SSL/TLS:
        Inclui diretivas SSLEngine, SSLCertificateFile e SSLCertificateKeyFile, que são necessárias para carregar seus certificados (geralmente gerados via Certbot/Let's Encrypt).


    Módulos Condicionais (<IfModule mod_ssl.c>):
        Garante que a configuração SSL só seja carregada se o módulo SSL do Apache estiver habilitado.


    ServerAlias:
        Inclui a versão www. do seu domínio, garantindo que ambas as versões funcionem.


    Arquivos de Log Específicos: 
        Em vez de usar os logs padrão do sistema, criamos arquivos de log exclusivos para este site (seusite_access.log, seusite_error.log, etc.).


    HTTP Security Headers (Opcional): 
        Adiciona cabeçalhos de segurança que ajudam a proteger contra ataques comuns (XSS, Clickjacking).



O principal desafio em um ambiente local é que você provavelmente não tem um domínio real (seusite.com.br) ou certificados SSL válidos emitidos por uma autoridade como o Let's Encrypt.



## Ajustes Necessários para Ambiente Local

Aqui está o que você precisa adaptar:

    1. Configure seu arquivo hosts

        Se você quiser usar um nome de domínio "falso" localmente (como meusite.local ou www.seusite.local), você precisa mapeá-lo para o seu endereço IP local (127.0.0.1) no arquivo hosts do seu sistema operacional.
        Linux: Edite /etc/hosts
        macOS: Edite /etc/hosts
        Windows: Edite C:\Windows\System32\drivers\etc\hosts
        Adicione a seguinte linha:

        127.0.0.1 meusite.local
        127.0.0.1 www.meusite.local



    2. Simplifique a configuração HTTP (Porta 80)

        Se você não quiser se preocupar com HTTPS localmente (o que é comum), você pode simplificar o Virtual Host e remover a parte de redirecionamento SSL.

        
### Configuração Simplificada para Uso Local (Apenas HTTP):

```

<VirtualHost *:80>
    ServerName meusite.local
    ServerAlias www.meusite.local
    DocumentRoot /var/www/html/seusite 
    # Ou o caminho exato para onde estão seus arquivos locais (ex: /var/www/html/projeto)

    <Directory /var/www/html/seusite>
        Options FollowSymLinks Indexes
        AllowOverride None
        Require all granted
        
        # Regras de Reescrita (Exemplo Laravel/WordPress)
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.php [L]
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/meusite_error.log
    CustomLog ${APACHE_LOG_DIR}/meusite_access.log combined
</VirtualHost>

```

    3. Habilite Módulos Necessários
        Certifique-se de que os módulos Apache necessários estejam habilitados no seu ambiente local (principalmente mod_rewrite):

```
sudo a2enmod rewrite
# Se for usar SSL localmente, habilite mod_ssl também:
# sudo a2enmod ssl
sudo systemctl restart apache2
```

Se você precisar testar a configuração HTTPS completa localmente, você não pode usar os certificados do Let's Encrypt.

Você precisará gerar certificados autoassinados (self-signed certificates) ou usar ferramentas como mkcert para criar uma autoridade de certificação local confiável. 

Seu navegador exibirá um aviso de segurança, mas a conexão HTTPS funcionará para fins de teste.


Em resumo: A estrutura do Virtual Host é a mesma, mas você precisa ajustar os ServerName, DocumentRoot e, geralmente, ignorar a parte do SSL ao rodar em localhost ou em um domínio local.


Gerar certificados autoassinados (self-signed certificates) é um processo comum para ambientes de desenvolvimento local. 

Eles permitem que você teste a configuração HTTPS sem precisar comprar ou emitir certificados reais, embora seu navegador exiba um aviso de segurança (dizendo que a conexão não é privada) porque o certificado não é assinado por uma autoridade reconhecida publicamente.

Você pode usar a ferramenta de linha de comando openssl, que geralmente já está instalada na maioria dos sistemas Linux e macOS.
Aqui está o passo a passo:

### Pré-requisitos
Certifique-se de que o openssl esteja instalado. Você pode verificar executando:
```
openssl version
```

### Passo 1: Criar um diretório para os certificados

É uma boa prática manter seus certificados organizados em um local seguro.
```
sudo mkdir -p /etc/apache2/ssl/local
cd /etc/apache2/ssl/local
```

### Passo 2: Gerar a Chave Privada e o Certificado
Você pode gerar ambos com um único comando openssl.
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout apache.key -out apache.crt
```

    Explicação dos parâmetros:
        req -x509: Cria uma solicitação de assinatura de certificado X.509.

        -nodes: "No DES" (Não criptografa a chave privada com uma senha, o que é útil para inicialização automática do servidor).

        -days 365: Define a validade do certificado para um ano.

        -newkey rsa:2048: Gera uma nova chave privada RSA de 2048 bits.

        -keyout apache.key: Nome do arquivo da chave privada.

        -out apache.crt: Nome do arquivo do certificado.


### Passo 3: Preencher as informações do Certificado
    O comando acima solicitará que você insira algumas informações. Para um certificado local, a maioria pode ser preenchida com valores genéricos, exceto pelo Common Name (CN).
    IMPORTANTE:
     O Common Name deve corresponder exatamente ao ServerName que você configurou no seu Virtual Host (por exemplo, meusite.local ou localhost).


    Exemplo de como preencher:

    Country Name (2 letter code) [AU]:BR
    State or Province Name (full name) [Some-State]:Sao Paulo
    Locality Name (eg, city) [Sydney]:Sao Paulo
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Minha Empresa Local
    Organizational Unit Name (eg, section) []:Desenvolvimento
    Common Name (e.g. server FQDN or YOUR name) []:meusite.local 
    Email Address []:admin@meusite.local



### Passo 4: Atualizar a Configuração do Virtual Host
    Agora, edite o arquivo de Virtual Host que configuramos anteriormente (/etc/apache2/sites-available/meusite.conf) e aponte as diretivas SSL para os novos arquivos gerados:

```

    # ... (Seu bloco HTTP:80) ...

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName meusite.local
    DocumentRoot /var/www/html/seusite 

    SSLEngine on
    # Aponte para os arquivos que você acabou de criar:
    SSLCertificateFile /etc/apache2/ssl/local/apache.crt
    SSLCertificateKeyFile /etc/apache2/ssl/local/apache.key
    
    # ... (Seu bloco Directory e outras configurações) ...

</VirtualHost>
</IfModule>
    

```

### Passo 5: Habilitar SSL e Reiniciar Apache


```
#Habilite o módulo SSL (se ainda não estiver ativado):

sudo a2enmod ssl

#Habilite o site (se ainda não estiver ativado):
sudo a2ensite meusite.conf

#Reinicie o Apache para aplicar todas as mudanças:
sudo systemctl restart apache2

```
> [NOTE]
>Ao acessar https://meusite.local no seu navegador, você verá o aviso de segurança. Você precisará clicar em "Avançado" ou "Aceitar o risco e continuar" para prosseguir para o seu site local.
