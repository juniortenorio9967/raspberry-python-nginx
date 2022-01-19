## Web Server em Raspberry Pi, com Python/Flask e Nginx

### Sequência de passos para implementar um servidor Web em uma placa Raspberry Pi, com Nginx Web Server Open Source e aplicação em Python/Flask.

Tecnologias utilizadas:

- Raspberry Pi 4 - 4GB RAM, Microprocessador ARM, Sistema Operacional Raspbian Linux-Debian based
    https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/
    
- Nginx Web Server - A empresa Nginx possui uma gama de produtos, entre eles o poderoso Nginx Web Server, portável para Raspberry Pi 4
    https://docs.nginx.com/nginx/admin-guide/web-server/app-gateway-uwsgi-django/
    
- Python/Flask - Flask é um micro framework de aplicações Web baseado na linguagem Python
    https://en.wikipedia.org/wiki/Flask_(web_framework)
    
- uWSGI - Serviço que permite a conexão entre a aplicação e o web server Nginx
    https://docs.nginx.com/nginx/admin-guide/web-server/app-gateway-uwsgi-django/

### Passo 1: Preparar a Raspberry, atualizando o Linux
	sudo apt-get update 
	sudo apt-get upgrade
  
### Passo 2: Instalar os software necessários
	sudo apt-get install nginx
	sudo apt-get install python3-pip
	sudo apt-get install build-essential python3-dev libssl-dev libffi-dev python3-setuptools
	sudo pip3 install flask uwsgi

### Passo 3: Criar o app Flask
	Criar uma pasta no diretório home:
	cd ~
	mkdir myserver
  
	Alterar a propriedade owner da pasta:
	sudo chown www-data:www-data /home/pi/myserver
  
	Entrar na pasta:
	cd myserver

	Criar arquivo rasp.py:
	sudo nano rasp.py

	Copiar o texto abaixo para o arquivo e salvar:
	from flask import Flask
 	app = Flask(__name__)

 	@app.route("/")
	def index():
	    return "<h3>Hello World</h3>"

	if __name__ == "__main__":
	    app.run(host='0.0.0.0')

  ### Passo 4: Testar o app
    Antes de executar o teste, verificar o ip da Raspberry, digitando o comando:
    ifconfig
    
    Digitar no navegador web o endereço ip identificado acima, por exemplo:
    http://192.168.101.112
    
    A página default do Nginx deverá aparecer.
    
    Executar um teste para verificar se o serviço uWSGI funciona corretamente:
      sudo uwsgi --socket 0.0.0.0.0:8000 --protocol=http -w rasp:app
    
    Voltar ao navegador e acrescentar a porta:
    http://192.168.101.112:8000
     
    Após o teste, retornar ao terminal e encerrar o serviço uWSGI digitando ctrl+c.
    
  ### Passo 5: Criar um arquivo de inicialização para o serviço uWSGI
      sudo nano uwsgi.ini
    
    Copiar o texto abaixo para o arquivo e salvar:
    	
	[uwsgi]
	module = rasp:app

	master = true
	processes = 1x
	threades = 2

	socket = myserver.sock
	chmod-socket = 664
	vacuum = true
	
	die-on-term = true


    
    
