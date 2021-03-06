Argos
=====

Argos - framework for massive security assessment of worldwide public ip cams

INTRODUCTION
============

this project started as a simple testing script for the security assesment of trednet ip cams, using shodan as a source for potential targets. 
but the project grew and now argos is a flexible framework that can be used to detect, security assess and statistically control ip cams from varius manufacturers around the globe.

argos relies on shodan to detect ip cams of a specific model/manufacturer, using a pretty simple plugin system based on bash code files with variables that allow the framework to fingerprint the cams. then argos calls thc hydra internally and parses the response log to test default passwords and finally tests known exploits, if any. 

results of a scanning execution can be sent through email automatically as well as seen on screen.


COMO FUNCIONA
=============

el framework revisa periodicamente la base de datos de shodan (http://www.shodanhq.com) en busca de dispositivos que cumplan con los patrones que nosotros hemos configurado previamente para cada caso.
esta entrega de "la máquina" viene con tres escanners configurados:

* camaras trendnet y derivadas (carpeta trendnet)
* camaras netwave y derivadas (carpeta netwave)
* camaras dlink y derivadas (carpeta dlink)

cuando se utiliza shodan con la API de una cuenta gratuita (como seguramente será vuestro caso), siempre devuelve 100 resultados, no más.
estos resultados pertenecen a los 100 últimos resultados mas recientes que encajen con los patrones de busqueda indicados.
esas 100 IPs son las que trabajará la máquina, buscando vulnerabilidades (como en el caso de las camaras trendnet),
y buscando passwords por defecto (d-link, netwave, trendnet, etc). 
en funcion de los resultados, el número de "victimas" de cada barrido podria oscilar entre 0 y 100, segun casos.
una vez obtiene los resultados de vulnerabilidades y passwords por defecto para ese conjunto de IP's, "la maquina" realiza una pausa configurable 
para luego volver realizar un nuevo barrido y una nueva consulta a la BDD de shodan, en busca de IP's nuevas que no estuvieran en la base de datos en anteriores barridos.

un simple CONTROL-C en el terminal finalizará el proceso de forma limpia y ordenada.

REQUISITOS
==========

* python 2.6 
  lo espera encontrar en /usr/bin/python26
  para indicar otra ruta, modificar la 1a linea del fichero utils/shodanquery.py

* una API key de shodan que sea válida
  1 - registrarse en ShodanHQ.com y visitar la zona DEVELOPERS para solicitar una API key
  2 - añadir la IP de vuestra conexión a la CONFIG de la WHITE LIST de dicha API KEY (si no no se puede usar)
  3 - colocar dicha API KEY en la linea correspondiente del fichero utils/shodanquery.py y guardar los cambios

* modulo shodan de python
  para instalarlo en sistemas debian, podemos utilizar el sistema easy_install de python:
  # apt-get install python-setuptools
  y ahora que disponemos de easy install, nos basta con hacer
  # easy_install-2.6 shodan
  para que el modulo shodan se instale automáticamente

* hydra de THC
  version estable o experimental
  compilados y presentes en el PATH

* un servidor de correo local válido compatible con el comando "mail" 
  probado con Exim4, el que viene por defecto en debian
  no hace falta que esté configurado para enviar emails al exterior si no quieres
  pero como minimo debe estar operativa la llamada "entrega de correo local, sin red".
  en sistemas debian puedes ejecutar el comando 
  dpkg-reconfigure exim4-config
  para revisar y alterar la configuracion de tu servidor de correo exim si fuera necesario
  
* configurar correctamente el framework para que entregue los resultados por email
  deberás indicar tu correo en el parámetro EMAILREPORTS que encontrarás en el fichero "config" de esta misma carpeta
  por norma general, si tu nombre de usuario en tu sistema linux es "harolfinch", aquí tendrás que poner "haroldfinch@localhost" 
  de este modo podras recoger los resultados de los escanners en tu correo local usando un cliente de correo como evolution.
  OPCIONAL: si tienes tu Exim4 correctamente configurado para que haga "re-entrega" a un servidor de correo externo, quizas te interese poner algun email de internet,
  sin embargo, si no quieres complicarte y no tienes conocimientos minimos de exim, la entrega local es la opción mas fácil

* un cliente de correo configurado correctamente
  si has decidido utilizar la entrega local de correo (tunombredeusuario@localhost), el cliente de correo deberá ser compatible con la recogida de correo local
  el ejemplo más común es evolution. 
  puedes añadir tu correo local en evolution a través de la opcion "directorio de correo en formato maildir" que encontrarás en el apartado "recepcion" del asistente para nuevas cuentas.

MODO DE USO
===========

como deciamos, esta entrega se presenta con tres escanners configurados e independientes.
cada uno de ellos se puede lanzar llamando al script llamados  "launch" seguido del nombre del "plugin" que queremos usar:

* dlink
* trendnet
* netwave

no hace falta ejecutarlos con privilegios de root.

a propósito de configuraciones básicas comunes, podrás encontrar un pequeño conjunto de variables de configuración en el fichero "config" que reside en esta misma carpeta:

* sendemail (TRUE por defecto)
  cuando implemente otras formas de hacer llegar los resultados al usuario, podrá desactivarse esta opcion para no tener que utilizar el correo electronico.
  sin embargo en estos momentos es la única opcion para la entrega de informes, y comentar o borrar esta linea supondria la pérdida total en el éter de los resultados.
  simplemente no toques este parámetro.

* emailreports (dirección de correo local o externa)
  ya se ha comentado previamente, pero repetimos
  por norma general, si tu nombre de usuario en tu sistema linux es "harolfinch", aquí tendrás que poner "haroldfinch@localhost" 
  de este modo podras recoger los resultados de los escanners en tu correo local usando un cliente de correo como evolution.
  OPCIONAL: si tienes tu Exim4 correctamente configurado para que haga "re-entrega" a un servidor de correo externo, quizas te interese poner algun email de internet,
  sin embargo, si no quieres complicarte y no tienes conocimientos minimos de exim, la entrega local es la opción mas fácil

COMPRENDIENDO LOS RESULTADOS
============================

Argos nos enviará un correo en cuyo titulo aparece el nombre del escaner que está generando los resultados, asi como la fecha y la hora del barrido.
En el cuerpo del mensaje encontraremos entre una y varias lineas similares con este formato:

http://Usuario:Contraseña@DireccionIP:Puerto

Al hacer click en estos enlaces, o al pegarlos tal cual en un navegador como firefox, iceweasel, chrome o chromium, se abrirá la interfaz web del dispositivo vulnerable.
en el 99% de ocasiones, no se nos pedirá usuario ni passwords porque van incorporados en el enlace, y el navegador comprende que debe usarlos en lugar de pedirlos al usuario,
asi que entraremos directamente al sistema vulnerado ;)

en algunos otros casos será necesario hacer click en algun enlace dentro de la interfaz web del dispositivo antes de acceder a la zona protegida, pero aun asi el navegador no nos pedirá user/pass una vez entremos en dicha zona progegida. 
Un ejemplo de esto son las camaras de la familia netwave, en las cuales, al entrar en ellas, aún tendremos que hacer click en el enlace "modo server push" o "modo 2" (segun fabricantes), antes de poder entrar a la zona protegida donde se puede ver la señal de video de la webcam. 

En muy pocos casos (la interfaz web que ha puesto algun revendedor de camaras netwave en sus dispositivos) quizas tengamos que introducir el usuario y la contraseña a mano
a pesar de estar documentados en el enlace. Pero aún así lograremos entrar correctamente en el dispositivo.
Todo dependerá de pequeños cambios en el código html que haya decidido implementar el revendedor del dispositivo, pero la mecánica interna no cambia en absoluto.

NOTA: este formato de enlace (con usuario:contraseña delante de la ip) no funciona en microchof internet exploiter. ni falta que hace.

COMO DESARROLLAR ESCANNERS NUEVOS BASADOS EN ESTE FRAMEWORK
===========================================================

Cada escanner es un modulo separado, un programa independiente, cuyo código parte del contenido de la plantilla (carpeta template)
luego dicha plantilla de escanner debe ser "configurada" para detectar y crackear un determinado tipo de dispositivo.

por "configurado" se entiende que nos hemos tomado la molestia de hacer un trabajo de investigacion para averiguar tres cosas:

 * alguna forma de identificar unívocamente o con muy bajo % de error a estos dispositivos 
   (lo que coloquialmente llamamos "obtener el fingerprinting")

 * ruta exacta a un archivo interno del dispositivo (normalmente un archivo CGI o una carpeta determinados) que esté protegido con autenticación HTTP estándar.
   en la mayoria de casos, la carpeta raíz "/" es la decisión correcta, pero en otros dispositivos no se pide usuario y contraseña nada mas entrar en ellos, sino que se primero se presenta ua pantalla, y el password se pide al pulsar un boton o acceder a determinada carpeta/archivo. 
   es en estos casos donde debemos hilar algo mas fino, revisar el codigo HTML en busca de respuestas y, cuando las encontramos, cambiar el "/" por cosas como "/videostream.cgi" (ejemplo de las camaras IP de la familia netwave).

 * los datos por defecto que trae el dispositivo: usuario y contraseña(s) por defecto.
   por ejemplo, en la familia de camaras de netwave, encontramos con el que el usuario por defecto es siempre admin, y segun modelos la contraseña por defecto es 
   * una cadena vacia ""
   * "admin"
   * "123456"
   por lo tanto en la configuración del escanner de las camaras netwave se ha colocado el usuario "admin" 
   y en el diccionario que utiliza se han insertado los tres passwords como opciones a probar. 

Cuando tenemos estos tres datos, abrimos el script "launch" y alteramos las variables que podemos encontrar en las primeras lineas del mismo, y que son autoexplicativas:

* scannername='Choeese a name for this scanner'
  nombre coloquial que le daremos a este escanner. el titulo de los emails y la emision de registros en pantalla utilizarán esta variable

* fingerprint='Shodan query that identifies device'
  aqui pondremos el fingerprint de dicho dispositivo, una vez hayamos encontrado un fingerprint válido y que no tenga errores o que tenga un % de error muy bajo.
  por ejemplo, el fingerprint de las camaras de trendnet es "netcam" porque utilizan dicha palabra en el mensaje de solicitud de contraseña (el auth realm)
  y al parecer no existe ningun otro modelo de dispositivo electronico en todo el planeta que utlice la misma expresion en ninguna parte de su contenido de la interfaz web
  por tanto, la expresión netcam solo aperece en camaras de trendnet y es una forma muy eficaz de realizar un fingerprinting de las mismas. 

* path_to_auth='/route/to/auth/protected/dir/or/file.cgi'
  ruta a la zona protegida del dispositivo.
  en la mayoria de casos con un simple '/' es suficiente, pero por ejemplo en las cámaras netwave esto nos daria falsos positivos de contraseña encontrada 
  porque la carpeta raiz '/' no esta protegida, sino que es publica. 
  siguiendo el ejemplo de las camaras netwave, una apuesta 100% segura es indicar la ruta al archivo que entrega el flujo de video '/videostream.cgi', 
  ya que lógicamente está protegido.
  tenemos que asegurarnos lo maximo posible de que el archivo o carpeta que indicamos aquí, estará siempre presente en todos los modelos/submodelos de los dispositivos
  que hemos "fingerprinteado". 
  Por ejemplo, los primeros dias de pruebas con las camaras de netwave traté de comprobar el acceso a las mismas probando contra el fichero '/snapshot.cgi'
  el fichero snapshot.cgi es un cgi que entrega una captura estática de video de lo que se ve en ese momento en la cámara.
  sin embargo no tardé en darme cuenta de que dicha función y dicho fichero no estan presentes en todas las camaras de dlink, por lo que obtenia gran cantidad de resultados falsos.
  analizando mejor el codigo html generado de la interfaz web, me di cuenta de que era mejor idea progar a acceder al fichero '/videostream.cgi', 
  ya que logicamente TODAS las camaras lo tienen :P  

* defaultlogin='admin'
  login por defecto del dispositivo

* wordlistfile='../dic/dictionary.dic'
  ruta a un fichero que contenga el/los posible(s) password(s) por defecto del modelo de dispositivo o la familia de dispositivos
  cada escanner tiene su propia carpeta "dic", y dentro de la misma hay un fichero "dictionary.dic" que es el que se utiliza por defecto.
  lo mas logico es editar a mano dicho fichero y asi no tienes que cambiar este parámetro nunca.

