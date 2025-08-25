
## Fase de renconcimiento:

### nmap:

Primero hago un reconocimiento de los servicios que tiene el sistema:
![](Imagenes/imagen1.png)

Con gobuster realizo un escaneo en busqueda de rutas ocultas del servicio http

![](Imagenes/Pasted%20image%2020250825163043.png)

En el directorio secret hay tres imagenes que a priori no parecen ayudar mucho

![](Imagenes/Pasted%20image%2020250823194818.png)
En robots.txt aparece este mensaje refirendose a robin, podría ser un usuario tambien 

![](Imagenes/Pasted%20image%2020250825163142.png)
En el directorio robin hay este texto

![](Imagenes/Pasted%20image%2020250825163431.png)

## Fase de explotación

Puedo intentar acceder al servicio de ftp para comprobar si permite el inicio de session anonimo, puede que ahí haya algo de información

![](Imagenes/Pasted%20image%2020250825165104.png)

Descubro de que permite el acceso anonimo y hay un directorio llamado reminder, dentro hay un archivo de texto que dice lo siguiente:

![](Imagenes/Pasted%20image%2020250825165216.png)

En el texto dice que el usuario local esta en las coordenadas, de momento no se que coordenadas son. Lo unico que me queda mirar ahora es si en las imagenes que hay en secrets/ hay alguna informacion oculta.

Una de las imagenes contiene unas coordenadas que corresponden a Okuda, Okinawa, Prefectura de Okinawa, Japon. Voy a seguir analizando las imagenes con diferentes herramientas para ver si encuentro algo más.

![](Imagenes/Pasted%20image%2020250825165407.png)

Utilizando la herramienta de stegseek he podido hacer fuerza bruta en la imagen de most-wanted:

![](Imagenes/Pasted%20image%2020250825174931.png)

En las demás imagenes no he encontrado nada.

Si leeo lo que he extraido de la imagen me sale esta cadena:
![](Imagenes/Pasted%20image%2020250825175208.png)
Esta codificada en base64 así que simplemente la descodifico

![](Imagenes/Pasted%20image%2020250825175245.png)

Ahora que tengo la contraseña me falta encontrar un usuario, para ello voy a crear un diccionario con varios nombres que podrían ser possibles usuarios y usaré la herramienta de hydra para realizar el ataque.

![](Imagenes/Pasted%20image%2020250825180020.png)

encuentro el usuario: batman

Ya puedo acceder usando ssh
![](Imagenes/Pasted%20image%2020250825180158.png)

## Fase de escalada de privilegios

Una vez dentro voy a ver que privilegios tiene el usuario batman:

![](Imagenes/Pasted%20image%2020250825190531.png)

No puedo ejecutar sudo -l, hecho un vistazo para ver que hay en el sistema:

![](Imagenes/Pasted%20image%2020250825190622.png)

Leo la flag pero no me sirve de mucho

![](Imagenes/Pasted%20image%2020250825190646.png)

voy a descargarme la imagen para poder analizarla, la descargo con scp

![](Imagenes/Pasted%20image%2020250825181936.png)

Pruebo diferentes herramientas pero ninguna me funciona:

![](Imagenes/Pasted%20image%2020250825190823.png)

![](Imagenes/Pasted%20image%2020250825191002.png)

![](Imagenes/Pasted%20image%2020250825191023.png)

Hasta que he encontrado la herramienta stegoveritas, que permite hacer un analisis profundo de imágenes (cambiando contrastes, subiendo brillo, etc) y una de las imagenes he visto esto:


![](Imagenes/Pasted%20image%2020250825194154.png)

Parece otra contraseña así que voy a probar otra vez a hacer fuerza bruta.

![](Imagenes/Pasted%20image%2020250825194555.png)

Si accedo puedo buscar el directorio home de joker

![](Imagenes/Pasted%20image%2020250825195252.png)

![](Imagenes/Pasted%20image%2020250825195325.png)

Y hasta aquí la máquina!