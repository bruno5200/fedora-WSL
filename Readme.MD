# Instalar Fedora en WSL

### presiona la tecla WIN + X y abre PowerShell como administrador

## 1) habilitamos el uso de WSL para nuestras máquinas virtuales en caso de que aún no esté activado usando cualquiera de los 2 comandos a continuación:

	Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux,VirtualMachinePlatform -All

	dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

## 2) Primero es recomendable actualizar el kernel de Linux antes de instalar cualquier distribución

### usando el siguiente enlace tenemos acceso a el link del kernel más reciente para Windows:

	https://docs.microsoft.com/es-es/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package

## 3) Una vez instalado usar el comando:
	
	wsl --set-default-version 2

## 4) Procedemos a descargar un contenedor base de fedora desde la web oficial y descargamos el que lleva x86_64 en formato xz desde la web oficial de fedora

	https://koji.fedoraproject.org/koji/packageinfo?packageID=26387

## 5) Mientras descarga podemos crear el directorio en el que se alojará nuestro fedora usando este comando
	
	mkdir $HOME\wsl\fedora

## 6) una vez completada la descarga procedemos a extraer usando Z-Zip o WinRAR (la que prefieras), entramos al folder con un nombre en hexadecimal muy estenso y ahí verás un archivo nombrado layer.tar,
### lo renombramos como "fedora-<version>-rootfs.tar" en mi caso es fedora 35 asi que mi archivo queda de la siguiente manera: "fedora-35-rootfs.tar" puedes usar cualquier versión de tu preferencia
### ya sea por estabilidad u otra naturaleza.

## 7) ejecutamos el siguiente comando para realizar la instalación (sólo necesitas cambiar el nombre si tu archivo es de otra versión de fedora)
	
	wsl --import fedora $HOME\wsl\fedora $HOME\Downloads\fedora-35-rootfs.tar

## 8) los siguientes comandos son unicamente para PowerShell y te ayudarán a gestionar tus distribuciones

### para listar:

	wsl -l

### para asignar una distribución por defecto:
### wsl -s <distro name>
### en mi caso es:

	wsl -s fedora

### para lanzar fedora como root usamos el siguiente comando

	wsl -d fedora
### el siguiente comando termina la instancia de fedora

	wsl -t fedora

## 9) creamos un usuario que no tenga privilegios luego de iniciar fedora como usuario root:
	
### primero es necesario instalar passwd y lo podemos instalar con el siguiente comando

	dnf install -y passwd cracklib-dicts
	
### añadimos el nuevo usuario con:
	
	useradd -G wheel miusuario

### para crear el pasword del nuevo usuario

	passwd miusuario


### ahora salimos de WSL con "exit" y lo volvemos a iniciar con el siguiente comando cambiando el usuario en tu caso

	wsl -d fedora -u miusuario

### verificamos que nuestro inicio sea correcto

	whoami

### comprobamos que podemos usar la contraseña creada para acceder como administrador que nos mostrará una lista de los usuarios en la que el nuestro estará en la aprte inferior

	sudo cat /etc/shadow

### hacemos este usuario el predeterminado usando el siguiente comando sin olvidar cambiar el miusuario por tu usuario creado
	
	printf "\n[user]\ndefault = miusuario\n" | sudo tee -a /etc/wsl.conf

### salimos de wsl y terminamos la instancia de fedora con el comando anteriormente visto de PowerShell

	wsl -t fedora
	
### comprobamos que funciona correctamente al ejecutar WSL con este comando

	wsl

## 10) hacemos una personalización necesaria para poder trabajar sin inconvenientes que yo recomiendo,
### primero vamos a reinstalar shadow-utils
	
	sudo dnf reinstall -y shadow-utils

### otros comandos que puedes estar acostumbrado a usar que faltan en los rootfs y puedes requerir son (opcional):

	sudo dnf -y install iproute findutils ncurses

### Si bien el vi incluido es un editor de texto útil, es probable que desee uno que sea más robusto o tenga una interfaz de usuario que se adapte a sus necesidades. Buenas opciones incluyen vim, micro o nano. En mi caso instalaré nano
	
	sudo dnf install -y nano
	
### desde ahora ya puedes proceder a instalar paquetes y personalizar tu distribución como desees. 😉

## 11) *BONUS* (muy útil) si deseas hacer esta instalación en otra máquina y no quieres repetir todo el tuturial no es necesario realizar todos los pasos en cada instalación
### simplemente exporta todo tu conjunto en un tar, procederemos a limpiar los paquetes descargados con este comando:
	
	sudo dnf clean all

### salimos de WSL con exit y luego ejecutamos el comando que tambien sirve para hacer un respaldo:
	
	wsl --export fedora $HOME\Downloads\fedora-wsl.tar

### y listo sólo tienes que instalarlo en otra máquina siguiendo los pasos iniciales, pero sin tener que descargar el contenedor base, sólo usa el tuyo propio y luego actualizalo o exporta usando el mismo comando. 😀
