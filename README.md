# Introdução

Esse é um guia compilado por mim para que futuramente eu tenha ideia do que realizei para a configuração do meu servidor pessoal.

Este guia contará com uma lista de passos que pode te ajudar (e me ajudar) a ter seu próprio servidor rodando em casa para desfrutar de NextCloud, piHole e outras ferramentas que podem ajudar a ter um melhor conhecimento de redes e também se livrar de serviços como Google Cloud e OneDrive.

---

## Setup

Para esse guia, será utilizado um Raspberry Pi 5 4 GB como máquina principal. Caso queira, é possível utilizar qualquer máquina que seja x86 ou ARM64 (aarch64), tendo em mente que essa máquina deve contar com um suporte no mínimo OK.

### Instalação do OS (Sistema Operacional)

Como utilizarei um Raspberry Pi, irei utilizar um método mais simples e convencional para realizar a instalação do sistema operacional. Para isso, utilizarei o [RaspberryPi Imager](https://www.raspberrypi.com/software/). Como estou utilizando Linux para fazer esse processo, recomendo baixar a [AppImage](https://downloads.raspberrypi.com/imager/imager_latest_amd64.AppImage) para te salvar de dor de cabeça e perda de tempo com versões desatualizadas disponibilizadas por cada Distro.

Após baixar o arquivo AppImage, entre na pasta onde o arquivo foi salvo e execute-o com o comando: `sudo ./imager_2.0.4_amd64.AppImage`

**Passo a passo**:

1. Selecione o dispositivo que irá utilizar:
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/1_screen.png)
2. Selecione seu sistema operacional de preferência, no meu caso utilizarei outros sistemas operacionais da própria Raspberry, como o Rasp OS Lite (64 bit):
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/2_screen.png)
3. Insira as informações pertinentes de sua preferência conforme o que for solicitado.
4. Após configurar todas as informações pertinentes nas telas anteriores, você terá uma tela de acesso remoto que, dependendo da sua intenção, pode ser ou não interessante. Caso você queira acessar essa máquina remotamente para configuração via sua rede local (LAN), habilite a função SSH. Você tem duas opções para login via SSH:
   - Senha: com essa opção, você poderá escolher uma senha para realizar o login via sua rede, porém essa opção é insegura caso queira deixar essa máquina exposta na rede pública.
   - Chave SSH: uma das opções mais seguras caso queira deixar a máquina exposta na rede pública ou local.
   - Passo a passo: ![Configuração SSH](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/Configura%C3%A7%C3%A3o%20SSH.md).
5. Caso queira, pode ativar a opção de Raspberry Pi Connect, no meu caso deixarei desativado.
6. Continue para a escrita das informações no cartão SD (**Atenção: todos os dados presentes nele serão apagados**).
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/5_screen.png)

### Login via SSH

Com sua máquina rodando e conectada à internet, você deverá descobrir o IP dessa máquina na sua rede local para realizar a conexão via SSH. Após descobrir o IP da sua máquina, geralmente algo parecido com 192.168.0.XXX, você poderá fazer login:

Para isso, utilize o comando abaixo para adicionar sua chave customizada:

```bash
ssh-add server-key-file-name
```

Após isso, tente realizar o login via comando no terminal:

```bash
ssh seu-usuario@ip-da-sua-maquina
```

Caso tenha problemas, tente usar a chave diretamente no comando:

```bash
ssh -i server-key-file-name seu-usuario@ip-da-sua-maquina
```

Com tudo funcionando corretamente, mova as chaves para a pasta onde deveriam estar:

```bash
mv server-key-file-name server-key-file-name.pub ~/.ssh/
```

### Instalação da interface web

Nessa parte, será instalada uma interface para gerência do seu servidor. São vários os tipos de software que podem ser instalados para ter controle do seu servidor sem a necessidade de, todas às vezes, realizar acesso via SSH. Alguns exemplos são: [Webmin](https://webmin.com), [Ajenti](https://github.com/ajenti/ajenti), [Cockpit](https://cockpit-project.org/) ou [1Panel](https://1panel.pro/).

Para esse tutorial, estarei utilizando o Webmin, já que imagino que servirá para a maioria das pessoas. Para realizar a instalação, basta seguir os seguintes passos, os quais são uma cópia direta do site da plataforma. Caso precise de instruções mais detalhadas, recomendo uma leitura da página de [download](https://webmin.com/download/) deles.

1. Setup:

```bash
curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
sudo sh webmin-setup-repo.sh
```

1. Instalação:
Para sistemas Fedora/Redhat/RHEL:

```bash
sudo dnf install webmin
```

Para sistemas Debian/Ubuntu/Mint:

```bash
sudo apt-get install --install-recommends webmin usermin
```

Após completar a instalação, você vai poder acessar o seu servidor via sua rede local no navegador com o endereço [https://192.168.0.xxx:10000](https://192.168.0.xxx:10000).
Nesta interface, você poderá realizar instalação de novos aplicativos, atualizações de sistema e aplicativos, bem como controlar e configurar todas as coisas que achar necessário.

## Instalação do Docker

Esta seção será destinada à instalação do Docker para realizar a instalação e configuração do Nextcloud All-in-One.

Para instalar o docker em um computador linux, utilize os seguintes comandos ou, caso queira, verifique diretamente o guia oficial em [Docker download.](https://docs.docker.com/engine/install/).

- **Adicione o repositório oficial do Docker e instale-o:**

<details>
<summary>Debian</summary>

Primeiramente adicione a chave:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Instale todos os requerimentos:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>Fedora</summary>

Primeiramente adicione a chave:

```bash
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

Instale todos os requerimentos:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>RHEL</summary>

Primeiramente adicione a chave:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

Instale todos os requerimentos:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>Ubuntu</summary>

Primeiramente adicione a chave:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Instale todos os requerimentos:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

## Instalação do Tailscale

Essa seção será destinada à instalação de uma VPN que permite que você acesse seu servidor de forma privada e segura sem a necessidade de liberar portas e lidar com problemas de firewall ou ainda utilizar um túnel Cloudflare.

Para isso, será utilizada a ferramenta [Tailscale](https://tailscale.com), ela é compatível com a maioria dos dispositivos móveis Android/Apple e computadores Linux/Windows.

Para realizar o download da ferramenta, utilize o seguinte comando:
Recomendo utilizar esse script fornecido pela própria plataforma, porém, caso você tenha problemas com esse método, sinta-se à vontade para realizar o download conforme o tutorial no [site deles](https://tailscale.com/download/linux).

   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```

Após o script finalizar, o serviço do Tailscale deverá estar executando normalmente. Para continuar, você deverá fazer login com sua conta da plataforma para adicioná-lo à sua rede pessoal.

Em seu servidor, acesse o terminal e digite o seguinte comando:

```bash
sudo tailscale up
```

Esse comando será executado e fornecerá um URL, acesse o endereço e realize login.
   **Não utilize [Ctrl + c] para copiar o endereço, pois isso pode resultar em um interrompimento do processo.**

- Exemplo do comando que aparecerá para realizar login:

    ```bash
       To authenticate, visit:
       https://login.tailscale.com/a/xxxxxxxxxxxx
    ```

Após acessar e realizar o login, o seu dispositivo deve aparecer em sua tailnet.
![imagem](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/6_screen.png)

## Instalação do Nextcloud

Nesta seção serão dispostos todos os comandos para a instalação do Nextcloud All-in-One (versão docker). Todos os comandos foram baseados no [GitHub oficial](github.com/nextcloud/all-in-one) do projeto.

- Instalação do Nextcloud

Para realizar a instalação via Docker com Tailscale, será necessário modificar o arquivo docker-compose conforme indicado nesta [discussão do git](https://github.com/nextcloud/all-in-one/discussions/6817#discussion-8836814).

Para isso, deve ser inserida uma nova seção de DNS e modificar duas configurações de ambiente.

1. No arquivo do docker-compose, adicione a seguinte configuração de DNS:

```bash
  services:
   nextcloud-aio-mastercontainer:
+    dns:
+      - 100.100.100.100  # Tailscale Magic DNS
+      - xxx.x.x.xx       # O resolverdor de dns do seu servidor no tailscale
+      - 1.1.1.1          # Fallback DNS
```

O resolvedor de DNS é o que está indicado no exemplo abaixo:
![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/7_screen.png)

2. Seguindo, retire a marcação de comentário "#" dos seguintes itens, como no exemplo:

```bash
     environment:
       APACHE_PORT: 11000
       APACHE_IP_BINDING: 127.0.0.1
```

Por fim, seu arquivo docker compose deve se assimilar ao exemplo abaixo (lembrando que o exemplo xxx.xxx.xx.xx deve ser substituído pelo endereço de seu servidor):
![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/8_screen.png)

- **Atenção**

<details>
<summary>Caso queira usar um armazenamento externo</summary>

Caso você for usar um dispositivo de armazenamento externo, ou seja, utilizar um HD diferente para armazenar os dados do Nextcloud, siga os seguintes passos para realizar esse processo antes de realizar a instalação do container.

Monte o novo dispositivo de armazenamento através da interface gráfica, lembre-se de que na maioria das vezes os diretórios são montados na pasta /mnt por padrão. Ou os comandos após esses seguintes passos:
![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/9_screen.png)

   1. Para realizar o processo pela interface gráfica, coloque um nome para o volume a ser montado, no caso pode ser algo como: /mnt/ncdata (como está descrito no exemplo do docker compose providenciado pelo Nextcloud);
   2. Selecione a partição do disco na opção ao lado: novo sistema de arquivos nativo do Linux (New Linux Native Filesystem);
   3. Marque as opções Save and Mount at Boot e Check filesystem at boot como check first.

- Comandos para terminal:

    Descubra qual é seu dispositivo para montar com:

    ```bash
    sudo fdisk -l
    ```

    Navegue para a pasta /mnt do sistema:

    ```bash
    cd /mnt/
    ```

    Crie um ponto para montar o dispositivo:

    ```bash
    mkdir /ncdata
    ```

    Monte o dispositivo:

    ```bash
    mount /dev/nome-do-seu-dispositivo /mnt/ncdata
    ```

Por fim, retime o marcador de comentário “#” no seguinte item do arquivo docker-compose:

```bash
    environment:
      NEXTCLOUD_DATADIR: /mnt/ncdata
```

![exemplo](https://github.com/ChancellorMarko/manual_hospedagem/blob/main/img/10_screen.png)

Apos isso, continue com o processo normalmente.

[Caso tenha mais dúvidas](https://github.com/nextcloud/all-in-one#how-to-store-the-filesinstallation-on-a-separate-drive)
</details>

Por fim, inicie o container docker do Nextcloud AIO para começarmos a servilo via sua rede privada Tailscale.

No mesmo diretório do arquivo docker-compose:

```bash
docker compose up -d
```
