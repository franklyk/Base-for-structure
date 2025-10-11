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

    ´´´
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ´´´

    A saída deve conter algo assim:

    > Generating public/private ALGORITHM key pair.



    Vamos partir do princípio de que já teremos uma VM criada, nesse caso utilizaremos a distribuição linux (UBUNTU 24.04) para exemplificar.

