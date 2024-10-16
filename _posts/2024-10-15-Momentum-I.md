---
title: Momentum I
layout: post
image: 
    path: /assets/covers/vulnhub.png
autor: AreiaNight
tags: [Momemtum, writeup, vulnhub]
category: Pentesting
---

Regresamos con las de Vulnhub después de algunas semana sin actividad gracias a dos cosas: Tengo trabajo. Me metí (de nuevo) de escritora de fanfics. Pero al fin regresé, así que vamos a ello. El día de hoy les traigo una máquina llamada *momentum*, es easy/medium, así que en teoría esta sería mi primera máquina medium. 

Comenzamos con lo de siempre, reconocimiento web. Ahora, nota importante, denle chance a la máquina de iniciarse porque si la activan y en corto tratan de analizar la web, no les aparecerá. Esperen de entre 10 a 15 minutos para que arranque como debe ser. Una vez todo listo, mandamos el siguiente comando: 

```
netdiscover -r IP/24
```

Al entrar a la página web mediante la ip, no hay mucho que ver, solo una pantalla de inicio con algunas imágenes, de las cuales reconocí la escultura de Lucifer (que tiene una historia muy chistosa de fondo, la cual contaré al final para dejar en evidencia mi background como artista). Mientras exploro la página, dejaré corriendo el nmap con mis especificaciones.

```
nmap -p- --open -sS -sC -sV --min-rate 5000 -v -Pn -n
```

Vale, al parecer solo hay dos puertos abiertos, el 80 y el 22, así que puedo suponer que esto será hacking web... Que divertido... wii... Pero bueno, es lo que hay, por lo que ahora queda hacer la enumaración de directorios para ver si podemos acceder a algo entretenido y con mucha, MUCHA, suerte, no tendré que usar burpsuit. 

![Pasted image 20241015161721.png](/assets/post/MomemtumI/Pasted%20image%2020241015161721.png)

Después de correr el feroxbuster, no me dio mucha información realmente, solo algunos directorios de manuales y demás. Pero lo que llamó mi atención fue que la darle click a las imágenes de la página principal cuando la tenemos en pantalla completa, nos manda a una especie de pequeña página informativa. Cuando esto pasa, tiendo a ver la composición del enlace y lo que vi me dio algunas pistas de lo que podríamos hacer. Por ejemplo, cada imagen tiene un *id distinto*, y lo que recuerdo es que a veces puedes usar esto para correr comandos por consola. Así que sí... tendré que abrir el burpsuit...

![Pasted image 20241015162021.png](/assets/post/MomemtumI/Pasted%20image%2020241015162021.png)

Después de varios intentos, no parecía ser vulnerable, así que volví a regresar a los directorios y los exploré de uno en uno. Todo estaba medio meh hasta que entré al de java y encontré algo de lo más curioso, una sección de código comentado que me hizo alzar una ceja. Se estaba usando una especie de encriptado mediante JavaScript con una frase clave, en este caso una función llamada CryptoJS AES.

```
/*
var CryptoJS = require("crypto-js");
var decrypted = CryptoJS.AES.decrypt(encrypted, "SecretPassphraseMomentum");
console.log(decrypted.toString(CryptoJS.enc.Utf8));
*/
U2FsdGVkX193yTOKOucUbHeDp1Wxd5r7YkoM8daRtj0rjABqGuQ6Mx28N1VbBSZt
```

Lo primero que pensé fue que quizá las imágenes tuviesen algo de información, ya que cada una tenía tres secciones: un ID, un nombre y un lugar. Así que bajé todas y fui corriendo tanto extftool como steghide, pero no hubo resultado alguno, así que me tocó investigar posibles vulnerabilidades y lo que encontré me indicó que estaba por buen camino. Sí, la información estaba encriptada, pero no en las imágenes... pero en los cookies. 

---
## Aprendiendo cosas con Areia pt 2 

¡Hola a todos! Bienvenides a su sección favorita, aprendiendo cosas con Areia. ¡Hoy, hablaremos sobre los cookies! *soniditos de yay de fondo*


> **¿Qué son las cookies?**
>
> Las **cookies** son pequeños archivos de texto que los sitios web almacenan en el navegador de los usuarios cuando visitan una página. Estos archivos contienen **información sobre la actividad y las preferencias** del usuario en el sitio, y permiten que los sitios web recuerden esa información para futuras visitas o durante la misma sesión.

Sinceramente, las cookies son un tema que aún me cuesta algo de entender, pero trateré de explicarlo lo mejor posible, al menos las cookies de este caso en particular. Así que ahí vamos. 

Como ya se dijo, las cookies son archivos que almacenan datos, ya sea del sistema, de las preferencias del usuario, historial del navegador, etc. Además de que existen varios tipos de cookies: temporales, persistentes, de tercer, etc. Dependiendo de la función de las cookies, será la información que se almacene. 

Al momento de entrar al inspector de página web, nos topamos con la siguiente información de la cookie: 

![Pasted image 20241015165409.png](/assets/post/MomemtumI/Pasted%20image%2020241015165409.png)

Para este punto tuve que investigar el cómo sé que que está trabajando esta cookie y, sobre todo, qué tipo de cookie era, cosa que no es fácil si no sabes mucho de su comportamiento. Vamos paso por paso para poder entender qué está pasando aquí, porque yo estaba igual de perdida. 

El *experience/max-age* es eso, la expiración de la cookie, que en este caso es cuando la sesión se cierre. 

El *size* es el tamaño de la cookie, o sea, el tamaño de los datos que tiene almacenado. 

El *httpOnly* en false nos indica que la cookie **no** es `HttpOnly`, por lo tanto, es accesible desde el código *JavaScript* en la página web. Cuando supe esto, mi cabeza conectó esta información con el fragmento de código que teníamos anteriormente. Pero por ahora vamos a seguir explicando los otros aspectos.

*Secure* significa que se puede transmitir a través de conexiones HTTP no cifradas (es decir, no solo a través de HTTPS). Esto podría representar un riesgo de seguridad si se envía a través de conexiones no seguras.

La *SameSite* significa que la cookie se enviará en solicitudes de otros sitios (cross-site requests), lo que podría tener implicaciones en cuanto a la protección frente a ataques CSRF (Cross-Site Request Forgery). También, para ser enviada con esta configuración, la cookie debería tener el atributo `Secure` activado, lo cual no parece ser el caso aquí.

Y por último, *Last Accessed* que indica la última vez que esta cookie fue accedida por el navegador.

----

Continuando con la máquina, ahora que sabemos que está cookie puede ser accesible mediante javascrip, quizá puede estar cifrando algo. Así que ahora toca desencriptar la cookie y para eso no, no me voy a meter a hacer un script para eso, voy a usar esta página web para hacerlo. Está bien que me guste hacer scripts, pero tampoco soy masoquista como para hacerme uno.

![Pasted image 20241015162211.png](/assets/post/MomemtumI/Pasted%20image%2020241015162211.png)

Ahora tenemos algo de información importante: `auxerre-alienum##`, que puede ser una serie de usuario y contraseña. Si recordaba bien, el port 22 estaba abierto, por lo que podemos establecer una conexión mediante ssh. Así que después de instalar las cosas necesarias (porque en este equipo no tenía nada por haber formateado mi host pc), llegó la hora de intentarlo. Así que usando medusa (una alternativa a hydra que prefiero usar), me hice dos files: username y psswd que serían los diccionarios a intentar. Con esto hecho, simplemente lancé el comando necesario: 

```
medusa -h 192.168.3.58 -U username.txt -P psswd.txt -M ssh
```

![Pasted image 20241015162435.png](/assets/post/MomemtumI/Pasted%20image%2020241015162435.png)
<center><s> (Medusa supremacy) </s></center><br>

Ahora solo nos queda conectarnos por ssh con las credenciales necesarias y listo, estamos dentro con el usuario auxerre. Ja, y yo creí que mi nombre Areia era demasiado, este bato me ganó.

![Pasted image 20241015162909.png](/assets/post/MomemtumI/Pasted%20image%2020241015162909.png)

Una vez adentró, solo bastó con lanzar un ls para obtener la primera flag, la de usuario. 

![Pasted image 20241015163249.png](/assets/post/MomemtumI/Pasted%20image%2020241015163249.png)

Ahora queda lo siguiente, subir de privilegios. Siempre que quiero subir de privilegios, trato primero de ver con que versión de Linux estoy trabajando, para eso usamos el `uname -a` el cual nos da la información necesaria. 

![Pasted image 20241015163305.png](/assets/post/MomemtumI/Pasted%20image%2020241015163305.png)

Hmn... No se ha actualizado desde el 2021 y estamos en el 2024, no creo encontrar muchos exploits para una máquina tan reciente... Así que ahora toca buscar otras formas de subir privilegios. Vamos primero a ver si podemos usar el sudo sin identificarnos como root con algún binario, a veces podemos subir privilegios por esa parte. 

![Pasted image 20241015163629.png](/assets/post/MomemtumI/Pasted%20image%2020241015163629.png)

¿Pero que...? Bueno, respiremos y pensemos. Por alguna extraña razón "sudo" no está disponible, así que tendremos que recurrir a otra alternativa. Para eso decidí usar un script de enumeración, en este caso el lse.sh para ver si podíamos encontrar algo comprometedor y encontré algo interesante, al parecer podíamos alterar ciertas tareas crone, he hecho esto antes, así que quizá podamos hacer algo con eso. 

![Pasted image 20241015164340.png](/assets/post/MomemtumI/Pasted%20image%2020241015164340.png)

Y no solo eso, también habían procesos corriendo de fondo como root, ahora la cuestión era si podíamos acceder a esos procesos con nuestros privilegios actuales para poder sacar la flag de root. Ya tenemos por ahora dos posibles rutas, quizá el proceso en segundo plano esté relacionado con las tareas crone, así que ahora era tiempo de averiguar cuales eran estos procesos corriendo en segundo plano. Sinceramente, esta es mi parte favorita, una vez adentro de la máquina. Y en efecto, ahí está. Ahora vamos a intentar abusar de los crones para ver si podemos sacarnos algo. 

![Pasted image 20241015165025.png](/assets/post/MomemtumI/Pasted%20image%2020241015165025.png)

Después de intentar muchas modificar el crone, no se pudo por cuestiones de permisos. Asi que tras volver a hacer una enumeración con este pequeño script, me di percaté de algo interesante que anteriormente que sí había notado, pero como ya era bastante tarde cuando lo estaba haciendo lo dije "luego lo veo" y jamás lo vi. Como sea, de lo que estoy hablando es que está corriendo un redis. Así que vamos a intentar ingresar a ver sí podemos sacar algo de ahí. 

![Pasted image 20241015165206.png](/assets/post/MomemtumI/Pasted%20image%2020241015165206.png)

Al parecer logré entrar con el comando básico y el puerto default, ahora solo queda ver que tenemos por acá. Esta es mi primera vez usando redis, así que tuve que hacer una pequeña investigación para ver con que me estaba enfrentado y, sobre todo, los comandos que debía utilizar para poder navegar por el servidor que estaba alojado en la máquina Momentum. Después de lanzar un INFO, decidí irme por el KEYS * y lo que me lanzó me hizo decir "ah, mira tú". Ahora, solo debemos lanzar un GET con el nombre de la key para ver lo que hay adentro. 

![Pasted image 20241015165329.png](/assets/post/MomemtumI/Pasted%20image%2020241015165329.png)

