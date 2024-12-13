---
title: Bug en página web
layout: post
image: 
    path: /assets/covers/bughunting.png
autor: AreiaNight
tags: [Bug, webpage]
category: Pentesting
---

Hace unos meses, mientras buscaba trabajo, me encontré con una página web local. Como persona curiosa que soy, decidí explorar un poco la página, sin malas intenciones, simplemente para ver si encontraba algún error o comportamiento inusual. Ya saben, como cuando introduces un símbolo donde deberían ir solo letras, esperando que algo interesante ocurra.

Fue en este proceso donde comencé a notar cosas extrañas, entre ellas la posibilidad de manipular la función de subida de archivos. Aquí les comparto un video donde documenté lo sucedido:

<iframe src="https://streamable.com/e/3fx16y" width="560" height="315" frameborder="0" allowfullscreen></iframe>

Como pueden observar, no me tomó mucho tiempo lograr subir un archivo. Esto podría derivar en una posible ejecución remota de comandos (si las circunstancias lo permitieran, aunque no lo intenté). También podría explotarse junto con otras vulnerabilidades para transferir archivos maliciosos. Además, el captcha no parece funcionar correctamente, ya que pude acceder a la siguiente página sin necesidad de verificar mi identidad.

![](/assets/post/BugBounty/1.png)

En la segunda parte del análisis, noté que la opción para enviar solicitudes de empleo no estaba habilitada, lo que implica que cualquier persona interesada en aplicar jamás lograría conectarse con la empresa. Esto me pareció sospechoso, ya que en teoría los puestos de trabajo están abiertos:

![](/assets/post/BugBounty/2.png)

Decidí no continuar investigando, ya que hacerlo requeriría técnicas más avanzadas y, por supuesto, el consentimiento explícito de la empresa. Me he puesto en contacto con ellos y espero que puedan corregir estos problemas para evitar riesgos mayores en el futuro.

Lo que me sorprendió de este caso es que no esperaba toparme con algo así fuera de los entornos controlados de laboratorios como TryHackMe o VulnHub. Las fallas identificadas son errores básicos de configuración web, pero aún así pueden representar un gran riesgo.

Por lo que puedo deducir, la página parece estar alojada en un servicio externo, lo que probablemente protege los servidores internos y la información sensible de la empresa. De todas formas, no realicé más pruebas por razones éticas, pero me pareció un caso curioso y quise compartirlo aquí