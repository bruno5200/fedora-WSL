# Instalar Fedora en WSL para versiones de Windows 10 y Windows 11

para seguir esta guía te recomiendo hacer click en el archivo Readme.MD para que lo puedas visualizar de mejor manera si estás en tu navergador o clonar el repositorio para trancodificarlo a HTML y verlo en un navegador desde donde puedas copiar los comandos a la terminal y powershell.

## 1) Activa WSL

para empezar presiona la tecla WIN + X y abre PowerShell como administrador

habilitamos el uso de WSL para nuestras máquinas virtuales en caso de que aún no esté activado usando cualquiera de los 2 comandos a continuación:

```ps1
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux,VirtualMachinePlatform -All

```

```ps1
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 2) Actualizar el Kernel de Linux para WSL

Primero es recomendable actualizar el kernel de Linux antes de instalar cualquier distribución

usando el siguiente enlace tenemos acceso a el link del kernel más reciente para Windows:

[Actualización del Kernel Linux para Windows](https://docs.microsoft.com/es-es/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)


## 3) Habilitar WSL 2

Una vez instalado usar el comando:

```ps1
wsl --set-default-version 2
```

## 4) Descarga Fedora 37

Procedemos a descargar un contenedor base de fedora desde la web oficial y descargamos el que lleva x86_64 en formato xz

[Fedora Proyect "build 37"](https://koji.fedoraproject.org/koji/buildinfo?buildID=2091819)

o elegir las versiones compartidas en su servidor de archivos

[Package Fedora Proyect](https://kojipkgs.fedoraproject.org/packages/Fedora-Container-Base/)

o también tienes la opción de hacerlo desde docker así:

```dockerfile
docker run --name fedora37 fedora:37
```

```dockerfile
docker export -o fedora-37-rootfs.tar fedora37
```

## 5) Creación del directorio de alojamiento

Mientras descarga podemos crear el directorio en el que se alojará nuestro fedora usando este comando

```ps1
mkdir $HOME\wsl\fedora
```

## 6) Extacción de los ficheros

una vez completada la descarga procedemos a extraer usando Z-Zip o WinRAR (la que prefieras), entramos al folder con un nombre en hexadecimal muy estenso y ahí verás un archivo nombrado layer.tar, lo renombramos como "`fedora-<version>-rootfs.tar`" en mi caso es fedora 35 asi que mi archivo queda de la siguiente manera: "fedora-37-rootfs.tar" puedes usar cualquier versión de tu preferencia
ya sea por estabilidad u otra naturaleza.

## 7) Instalación de fedora

ejecutamos el siguiente comando para realizar la instalación (sólo necesitas cambiar el nombre si tu archivo es de otra versión de fedora)

```ps1
	wsl --import fedora $HOME\wsl\fedora $HOME\Downloads\fedora-37-rootfs.tar
```

## 8) Gestión de Distribuciones en Powershell

los siguientes comandos son unicamente para PowerShell y te ayudarán a gestionar tus distribuciones

para listar:

```ps1
wsl -l
```

para asignar una distribución por defecto `wsl -s <distro name>`, en mi caso es:

```ps1
wsl -s fedora
```

para lanzar fedora como root usamos el siguiente comando

```ps1
wsl -d fedora
```

el siguiente comando termina la instancia de fedora

```ps1
wsl -t fedora
```

## 9) Usuario no Root

crearemos un usuario que no tenga privilegios luego de iniciar fedora como usuario root
	
primero es necesario instalar passwd y lo podemos instalar con el siguiente comando:

```zsh
dnf install -y passwd cracklib-dicts
```

añadimos el nuevo usuario con:

```zsh
useradd -G wheel miusuario
```

y para crear el pasword del nuevo usuario:

```zsh
passwd miusuario
```

ahora salimos de WSL con "exit" y lo volvemos a iniciar con el siguiente comando cambiando el usuario en tu caso

	wsl -d fedora -u miusuario

verificamos que nuestro inicio sea correcto.

```
whoami
```

### comprobamos que podemos usar la contraseña creada para acceder como administrador que nos mostrará una lista de los usuarios en la que el nuestro estará en la parte inferior.

```
sudo cat /etc/shadow
```

hacemos este usuario el predeterminado usando el siguiente comando sin olvidar cambiar el miusuario por tu usuario creado

```
printf "\n[user]\ndefault = miusuario\n" | sudo tee -a /etc/wsl.conf
```

salimos de wsl y terminamos la instancia de fedora con el comando anteriormente visto de PowerShell.

```ps1
wsl -t fedora
```

comprobamos que funciona correctamente al ejecutar WSL con este comando:

```ps1
wsl
```
## 10) Configuraciones extras

hacemos una personalización necesaria para poder trabajar sin inconvenientes que yo recomiendo,

primero vamos a reinstalar shadow-utils

```zsh
sudo dnf reinstall -y shadow-utils
```
otros comandos que puedes estar acostumbrado a usar que faltan en los rootfs y puedes requerir son (opcional):

```
sudo dnf -y install iproute findutils ncurses
```

Si bien el vi incluido es un editor de texto útil, es probable que desee uno que sea más robusto o tenga una interfaz de usuario que se adapte a sus necesidades. Buenas opciones incluyen vim, micro o nano. En mi caso instalaré nano.

```zsh
sudo dnf install -y nano
```

desde ahora ya puedes proceder a instalar paquetes y personalizar tu distribución como desees. 😉

## 11) *BONUS* (muy útil) 

si deseas hacer esta instalación en otra máquina y no quieres repetir todo el tuturial no es necesario realizar todos los pasos en cada instalación

simplemente exporta todo tu conjunto en un tar, procederemos a limpiar los paquetes descargados con este comando:

```zsh	
sudo dnf clean all
```

salimos de WSL con `exit` y luego ejecutamos el comando que tambien sirve para hacer un respaldo:

```ps1	
wsl --export fedora $HOME\Downloads\fedora-wsl.tar
```
Y listo sólo tienes que instalarlo en otra máquina siguiendo los pasos iniciales, pero sin tener que descargar el contenedor base, sólo usa el tuyo propio y luego actualízalo o exporta usando el mismo comando. 😀


## Actualizr la versión de fedora

primero realiza un backup de lo que tienes hasta ahora, puedes especificar otra carpeta diferente a la de descargas sólo con cambiar la ruta.

```ps1
wsl --export fedora $HOME\Downloads\fedora-wsl.tar
```

Ejecutamos el siguiente comando dentro del Shell de fedora que quieras actualizar para actualizar todos los paquetes necesarios a la versión más reciente.

```zsh
sudo dnf --refresh upgrade
```

Ahora debemos instalar el plugin para actualizar el sistema y si ya está presente comprobar que no esté desactualizado por algún motivo.

```zsh
sudo dnf install dnf-plugin-system-upgrade
```

En mi caso pasaré a la versión 38 (puedes poner la versión correspondiente a la que vayas a hacer el upgrade) que ya es estable en la fecha que hago esta actualización del repositorio, nos pedirá confirmar que tenemos todos los paquetes actualizados a la versión más reciente para ello podemos presionar la tecla Y y luego Enter.

```zsh
sudo dnf system-upgrade download --releasever=38
```

De la misma manera al acabar este proceso nos preguntará si estamos de acuerdo en que se importe una nueva clave GPG a lo que deberemos de responder otra vez con Y y presionar Enter.

Luego tendremos que confirgurar una variable de entorno ya que nuestra distribución no está montada sobre un hardware para reiniciarse y tratará de hacerlo fallando en el proceso y generando un posible loop de reinicios, para lo que es necesario ejecutar el siguiente comando:

```zsh
export DNF_SYSTEM_UPGRADE_NO_REBOOT=1
```

Posteriormente corremos los comandos `sudo -E dnf system-upgrade reboot` y `sudo -E dnf system-upgrade upgrade` para que se aplique correctamente la actualización de todos los paquetes necesarios en nuestro fedora 37 WSL.

Finalmente necesitamos recargar la base de datos de los Paquetes de RPM

```zsh
sudo rpmdb --rebuilddb
```

Por ultimo si nos queda alguna duda de si tenemos todo actualizado podemos volver a refrescar las actualizaciones y verificarla así:

```zsh
sudo dnf upgrade --refresh
```
