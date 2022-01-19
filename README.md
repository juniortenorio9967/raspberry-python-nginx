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
    
Fontes pesquisadas:
    https://singleboardbytes.com
    https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04
    https://stackoverflow.com/questions/61354262/nginx-server-running-flask-on-raspberry-pi


### Passo 1: Preparar a Raspberry, atualizando o Linux
	sudo apt-get update 
	sudo apt-get upgrade
  
### Passo 2: Instalar os software necessários
	sudo apt-get install nginx
	sudo apt-get install python3-pip
	sudo apt-get install build-essential python3-dev libssl-dev libffi-dev python3-setuptools
	sudo pip3 install flask uwsgi wheel

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
    http://192.168.101.112 (Você deverá informar seu IP)
    
    A página default do Nginx deverá aparecer.
    
    Executar um teste para verificar se o serviço uWSGI funciona corretamente:
      sudo uwsgi --socket 0.0.0.0.0:8000 --protocol=http -w rasp:app
    
    Voltar ao navegador e acrescentar a porta:
    http://(seu IP):8000
     
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

  ### Passo 6: Testar o arquivo de inicialização uWSGI
  	sudo uwsgi --ini uwsgi.ini
	
	Abrir uma nova janela de terminal, digitar o comando abaixo para visualizar os arquivos
	da pasta myserver. Dentre eles deverá estar presente o arquivo myserver.sock. Este arquivo aparece somente
	quando o serviço uWSGI está rodando, daí a necessidade de abrir outro terminal, pois o primeiro estará rodando
	o serviço.
	
	cd ~/myserver
	ls
	
	Nesse momento, o uWSGI não está utilizando nenhuma porta, apenas o Nginx está respondendo na porta padrão.
	Para encerrar o serviço uWSGI, digitar ctrl+c.
	
  ### Passo 7: Configurar o Nginx para utilizar o serviço uWSGI
  	
	Agora o Nginx será configurado para direcionar todo o tráfego para o serviço uWSGI. Esse recurso do
	Nginx é conhecido como "Proxy Reverso".
	
	Primeiro, excluir o arquivo default do Nginx:
	sudo rm /etc/nginx/sites-enabled/default
	
	Em seguida, criar um novo arquivo:
	sudo nano /etc/nginx/sites-available/myserver_proxy
	
	Copiar o código abaixo para o arquivo e salvar:
	
	server {
	listen 80;
	server_name localhost;
	
	location / {
	include uwsgi_params;
	uwsgi_pass unix:/home/pi/myserver/myserver.sock;
	  }
	}
	
	Para finalizar, criar atalho (simlink) na pasta "sites-enable" para o arquivo criado:
	sudo ln -s /etc/nginx/sites-available/myserver_proxy /etc/nginx/sites-enabled


  ### Passo 8: Reiniciar o Nginx
  
  	sudo systemctl restart nginx
	
	Após reiniciar, o endereço http://(seu IP) irá responder com "502 Bad Gateway". O Nginx está tentando
	repassar as requisições do navegador para o uWSGI, que nesse momento não está rodando, gerando o erro.

  ### Passo 9: Rodar o uWSGI sempre que a Raspberry iniciar
  
  	Para possibilitar que o uWSGI inicie junto com Linux, utilizamos o serviço "systemd". Vamos entrar no
	diretório:
	cd /etc/systemd/system
	
	criar o arquivo de serviço para o uWSGI:
	sudo nano uwsgi.service
	
	Copiar o código abaixo para o arquivo criado e salvar:
	
	[Unit]
	Description=uWSGI Service
	After=network.target
	
	[Service]
	User=pi
	Group=www-data
	WorkingDirectory=/home/pi/myserver
	Environment="PATH=/home/pi/myserver/bin"
	ExecStart=/home/pi/myserver/bin/uwsgi --ini uwsgi.ini
	
	[Install]
	WantedBy=multi-user.target
    
    
	Para que o systemd através de seu daemon capture o novo arquivo e o execute, precisamos reinicializar o daemon:
	sudo systemctl daemon-reload
	
	Podemos startar o serviço manualmente agora...
	sudo systemctl start uwsgi.service
	
	...e para que da próxima vez que a Raspberry iniciar, o serviço uWSGI suba junto:
	sudo systemctl enable uwsgi.service
	
	Para finalizar, testamos o status do serviço:
	sudo systemctl status uwsgi.service
	
	O status será apresentado, onde a linha mais importante é a que informa "Active: active (running)", indicando que
	o serviço está rodando OK!
	
  ###  Passo 10: Reiniciar e testar
  
  	sudo reboot
	
	Após reiniciar a Raspberry, acessar o endereço http://(seu IP), e a página web "Hello World" deverá se apresentar.
	
