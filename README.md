# Open-redirect

<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  burpsuit
</div>

En este primer post de la sección de **Bug bounty** se va a llevar a cabo la explicación de qué es y de cómo funciona un *open redirect*, un tipo de vulnerabilidad que es relativamente fácil de encontrar.

Un *open redirect* es una vulnerabilidad que permite, dentro de una aplicación web, redirigir a los usuarios a otra URL proporcionada por la falta de una validación adecuada de un parámetro. Lo que hacen los ciberdelincuentes es generar una dirección, aparentemente legítima, para que el usuario se confíe entrando en ella y sufran un ataque de *phishing*.

Para llevar a cabo este ejercicio he utilizado la plataforma [Docker labs](https://dockerlabs.es/), donde se encuentra la máquina virtual que utilizaremos 'Hojaytinta'. Se descarga el *zip* en la propia *Kali* y extraemos el fichero *autodeploy.sh* y el archivo *tar* que será la máquina con la que practicar. PAra desplegar el laboratorio y obtener la direccion IP víctima sobre la que trabajar, se escribe lo siguiente en la terminal:

<code>sudo bash autodeploy.sh hojaytinta.tar</code>

<img width="522" height="303" alt="image" src="https://github.com/user-attachments/assets/68cb0668-57f9-4eda-a6d4-615a1eb358aa" />

Una vez realizadas estas acciones, se procede a explotar la vulnerabilidad.

Al acceder a la dirección *web* damos con una página de venta de libros, para encontrar el primer *open redirect* hay que cerrar sesión. Antes, como es obvio, es necesario iniciar sesión con los datos ya facilitados en la página. Para leer la petición que se produzca con el cierre de la sesión se utiliza **Burp Suite** y desde la la pestaña Proxy>HTTPhistory analizamos la petición GET del cierre de sesión.

<img width="743" height="712" alt="Captura de pantalla 2026-10-07 004311" src="https://github.com/user-attachments/assets/baf7aa29-e1a7-4572-b34c-f00a3abfc0ad" />

<code>/account/logout/callback?client_id=hoja-y-tinta-web&state=%7B%22redirectUri%22%3A%22%2F%22%7D</code>

Como se puede ver en la URL, esta redirigiendo el tráfico a otro sitio. Para saber a dónde, hay que decodificar con el decoder de *Burp Suite* pues está 'URLcodeado'.

<code>{"redirectUri":"/"}</code>

En este caso, no parece redirigir a ninguna *web*, pero añadiendo detrás una dirección podemos dirigirnos a ella y aquí se produce el *open redirect*. Para ello probamos con la dirección ya visitada *dockerlabs.es', pero además se debe añadir una segunda barra '/' para poder 'bypasear' la restricción y obtener una dirección maliciosa.

<code>http://hojaytinta.lab/account/logout/callback?client_id=hoja-y-tinta-web&state={"redirectUri":"//dockerlabs.es"}</code>

<img width="1216" height="853" alt="image" src="https://github.com/user-attachments/assets/9526ae16-bbdb-4a05-8641-13007c7d3a8b" />
