# Intro

Esse é um guia compilado por mim para que futuramente eu tenha ideia do que realizei para a configuração do meu servidor pessoal.

Este guia contará com uma lista de passos que pode te ajudar (e me ajudar) a ter seu próprio servidor rodando em casa para desfrutar de NextCloud, piHole e outras ferramentas que podem ajudar a ter um melhor conhecimento de redes e também se livrar de serviços como Google Cloud e OneDrive.

---

## Setup

Para esse guia eu, pessoalmente estarei utilizando um Raspberry Pi 5 4GB como maquina principal. Caso queira é possível utilizar qualquer maquina que seja x86 ou ARM64 (aarch64), tendo em mente que essa maquina deve contar com um suporte no minimo OK.

### Instalação do OS (Sistema Operacional)

Como utilizarei um Raspberry Pi, irei utilizar um método mais simples e convencional para realizar a instalação do sistema operacional. Para isso utilizarei o [RaspberryPi Imager](https://www.raspberrypi.com/software/). Como estou utilizando Linux para fazer esse processo recomendo baixar a [AppImage](https://downloads.raspberrypi.com/imager/imager_latest_amd64.AppImage) para te salvar em dor de cabeça e perca de tempo com versões desatualizados que são disponibilizadas por cada Distro.

Apos baixar o arquivo AppImage entre na pasta onde o arquivo foi salvo e execute-o com o comando: `sudo ./imager_2.0.4_amd64.AppImage`

**Passo a passo**:
1. Selecione o dispositivo que irá utilizar: !(exemplo)[img/1_screen.png]
2. Selecione seu sistema operacional de preferencia, no meu caso utilizarei outros sistemas operacionais da própria Raspberry, como o Rasp_OS - Lite (64 bit): [[2_screen.png]]
3. Insira as informações pertinentes de sua preferência conforme o que for pedido.
4. Após configurar todas as informações pertinentes nas telas anteriores, você terá uma tela de acesso remoto que dependendo da sua intenção pode ser ou não interessante. Caso você queria acessar essa maquina remotamente para configuração via sua rede local (LAN) habilite a função SSH. Você tem duas opções para login via SSH:
	- Senha: com essa opção você poderá escolher uma senha para realizar o login via sua rede, porém essa opção é insegura caso quiser deixar essa maquina exposta na rede publica.
	- Chave SSH: uma das opções mais seguras caso queira deixar a maquina exposta na rede pública ou local.
	- Passo a passo: [[Configuração SSH]].
5. Caso quiser pode ativar a opção de RaspberryPi Connect, no meu caso deixarei desativado.
6. Continue para a escrita das informações no cartão SD (**Atenção todos os dados presentes nele serão apagados**).

### Login via SSH:

Com sua maquina rodando e conectada a internet você deverá descobrir o IP dessa maquina na sua rede local para realizar a conexão via SSH. Após descobrir o IP da sua maqui, geralmente algo parecido com 192.168.0.XXX, você será capaz de fazer login:

Para isso utilize o comando abaixo para adicionar sua chave customizada:
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

Com tudo funcionando corretamente mova as chaves para a pasta onde deveriam estar:
```bash
mv server-key-file-name server-key-file-name.pub ~/.ssh/
```

### Instalação interface web:

Nessa parte será instalado uma interface para gerência do seu servidor, são vários os tipos de software que pode ser instalado para ter controle do seu servidor sem a necessidade de todas as vezes realizar acesso via SSH. Alguns exemplos são: [Webmin](https://webmin.com), [Ajenti](https://github.com/ajenti/ajenti), [Cockpit](https://cockpit-project.org/) ou [1Panel](https://1panel.pro/).

Para esse tutorial estarei utilizando o Webmin já que imagino que servirá para a maioria das pessoas. Para realizar a instalação basta seguir os seguintes passos que são uma cópia direta do site da plataforma, caso precise de instruções mais detalhadas recomendo uma leitura da pagina de [download](https://webmin.com/download/) deles.

1. Setup:
```bash
	curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
sudo sh webmin-setup-repo.sh

```
2. Instalação:
	Para sistemas Fedora/Redhat/RHEL: 
	```bash
		sudo dnf install webmin
	```
	Para sistemas Debian/Ubunto/Mint:
	```bash
		apt-get install --install-recommends webmin usermin
	```

Após completar a instalação, você vai poder acessar o seu servidor via sua rede local no navegador com o endereço [https://192.168.0.xxx:10000](https://192.168.0.xxx:10000).
Nesta interface você poderá realizar instalação de novos aplicativos, atualizações de sistema e aplicativos bem como controlar e configurar todas as coisas que achar necessário.