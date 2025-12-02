# Windows 
- Tipo de instalação
	Selecione a instalação completa

- link para setar na instalacao em remote targets
	http://yourdomain.com.br/glpi/front/inventory.php

- link para forcar invetory
	localhost:62354

- export info manual
	## execute o cmd como administrador
	"C:\Program Files\GLPI-Agent\glpi-inventory.bat" > c:\inventory.xml
	"C:\Program Files\GLPI-Agent\glpi-inventory.bat" --json > c:\inventory.json


#Linux
	$ wget -c https://github.com/glpi-project/glpi-agent/releases/download/1.15/glpi-agent-1.15-linux-installer.pl
	$ sudo chmod +x glpi-agent-1.15-linux-installer.pl
	$ sudo chmod 750 glpi-agent-1.15-linux-installer.pl
	
	# install
	$ sudo ./glpi-agent-1.15-linux-installer.pl --install
	
	#link para setar na instalacao em remote 
	http://yourdomain.com.br/glpi/front/inventory.php
	
	#Pressionar enter 2x
	
	#force inventory
	$ sudo glpi-agent --force --no-ssl-check

	#inventory manual
	$ sudo glpi-inventory --json > /home/usuario/Documentos/inventory.json

## linux agent com appimage
	$ wget -c https://github.com/glpi-project/glpi-agent/releases/download/1.15/glpi-agent-1.15-x86_64.AppImage
	$ sudo ./glpi-agent-1.15-x86_64.AppImage --install --server "http://yourdomain.com.br/glpi/front/inventory.php"

	#inventory manual
	$ sudo glpi-inventory --json > /root/inventory.json


