# Open-redirect

En este primer post de la sección de **Bug bounty** se va a llevar a cabo la explicación de qué es y de cómo funciona un *open redirect*, un tipo de vulnerabilidad que es relativamente fácil de encontrar.

<div>
  <img src="https://img.shields.io/badge/-Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-Burp suite-FF6633?style=for-the-badge&logo=burpsuite&logoColor=white" />
  
</div>

## Introducción
Un *open redirect* es una vulnerabilidad que permite, dentro de una aplicación web, redirigir a los usuarios a otra URL proporcionada por la falta de una validación adecuada de un parámetro. Lo que hacen los ciberdelincuentes es generar una dirección, aparentemente legítima, para que el usuario se confíe entrando en ella y sufran un ataque de *phishing*.

Para llevar a cabo este ejercicio se ha utilizado la plataforma [Docker labs](https://dockerlabs.es/), donde se encuentra la máquina virtual que utilizaremos 'Hojaytinta'. Se descarga el *zip* en la propia *Kali* y extraemos el fichero *autodeploy.sh* y el archivo *tar*, que será la máquina con la que practicar. Para desplegar el laboratorio y obtener la direccion IP víctima sobre la que trabajar, se escribe el siguiente comando en la terminal:

<code>sudo bash autodeploy.sh hojaytinta.tar</code>

<img width="522" height="303" alt="image" src="https://github.com/user-attachments/assets/68cb0668-57f9-4eda-a6d4-615a1eb358aa" />

Como último paso se deben añadir la IP y la dirección en '/etc/hosts'.

<img width="549" height="198" alt="Captura de pantalla 2026-10-07 191432" src="https://github.com/user-attachments/assets/d6c03a25-ba0a-45e4-93f3-f65e71870d8a" />

Una vez realizadas estas acciones, se procede a explotar la vulnerabilidad.

## Explotación

Al acceder a la dirección *web* damos con una página de venta de libros, para encontrar el primer *open redirect* hay que cerrar sesión. Antes, como es obvio, es necesario iniciar sesión con los datos ya facilitados en la página. Para leer la petición que se produce con el cierre de la sesión se utiliza **Burp Suite** y desde la pestaña Proxy>HTTPhistory analizamos la petición GET del cierre de sesión.

<img width="643" height="612" alt="Captura de pantalla 2026-10-07 004311" src="https://github.com/user-attachments/assets/baf7aa29-e1a7-4572-b34c-f00a3abfc0ad" />

<code>/account/logout/callback?client_id=hoja-y-tinta-web&state=%7B%22redirectUri%22%3A%22%2F%22%7D</code>

Como se puede ver en la URL, está redirigiendo el tráfico a otro sitio. Para saber a dónde, hay que decodificar con el decoder de *Burp Suite*, pues la dirección está 'URLcodeada'.

<code>{"redirectUri":"/"}</code>

En este caso, no parece redirigir a ninguna *web*, pero añadiendo detrás una dirección podemos dirigirnos a ella y es aquí donde se produce el *open redirect*. Para ello probamos con la dirección ya visitada 'dockerlabs.es', a la cual además se debe añadir una segunda barra '/' para poder 'bypasear' la restricción y obtener una dirección maliciosa. De manera que queda así:

<code>http://hojaytinta.lab/account/logout/callback?client_id=hoja-y-tinta-web&state={"redirectUri":"//dockerlabs.es"}</code>

<img width="916" height="553" alt="image" src="https://github.com/user-attachments/assets/9526ae16-bbdb-4a05-8641-13007c7d3a8b" />

Una vez explotada la vulnerabilidad, vamos a por el segundo *open redirect*. Este se encuentra en un anuncio de la web, los cuales son propensos a tener este tipo de vulnerabilidades.

De nuevo, desde *Burp Suite* se puede ver la petición y como vuelve a haber una redirección. Además utilizando el *Decoder*, se descifra la *URL*.

<img width="935" height="378" alt="Captura de pantalla 2026-10-07 183207" src="https://github.com/user-attachments/assets/f90e25c8-a724-4b81-aef4-7d595c03bfa3" />

<code>http://hojaytinta.lab/ads/tr?ci=el-jardin-de-sal-9f21&e=i&tu=https%3A%2F%2Fads.hojaytinta-partners.com%2Fpixel%3Fcampaign%3Dtrailer%26book%3Del-jardin-de-sal</code>

<code>http://hojaytinta.lab/ads/tr?ci=el-jardin-de-sal-9f21&e=i&tu=https://ads.hojaytinta-partners.com/pixel?campaign=trailer&book=el-jardin-de-sa</code>

Cambiar simplemente la dirección *web* por a la que queremos dirigirnos no es suficiente y es necesario añadir un '@' Esta manera de redirecionar no es una vulnerabilidad, es como funciona internet, se puede probar poniendo en el navegador lo siguiente:

<code>google.com@dockerlabs.es</code>

<img width="1212" height="195" alt="Captura de pantalla 2026-10-07 183441" src="https://github.com/user-attachments/assets/af00d0f8-f3b1-44ef-b56e-43f25780e32e" />
<img width="1663" height="226" alt="Captura de pantalla 2026-10-07 183656" src="https://github.com/user-attachments/assets/d2630f1a-f1f8-4614-bd7c-b921ccfdcb0a" />

Sin embargo, esto no funciona pues es necesario tratar con las cabeceras de la petición web, en este caso con la cabecera 'Referer' para que reconozca el origen de la petición. Lo indispensable es añadir a la cabecera de la petición la dirección de la página principal para que la redirección se produzca desde el mismo dominio. El proceso para encontrar la petición es el mismo que en el primer *open redirect*.

<code>Referer: http://hojaytinta.lab/libro/el-jardin-de-sal</code>

<code>GET /ads/tr?ci=el-jardin-de-sal-9f21&e=i&tu=https://ads.hojaytinta-partners.com@www.google.es&book=el-jardin-de-sal HTTP/1.1</code>

<img width="1497" height="320" alt="Captura de pantalla 2026-10-07 185259" src="https://github.com/user-attachments/assets/ac99196d-666f-46d7-a9bd-08913ccccf43" />

Como se muestra en la imagen, la respuesta del *Repeater* de *Burp Suite* indica la redirección. Pero llegados a este punto, esto no es suficiente para considerarse crítico pues, como he mostrado, es necesario añadir manualmente la cabecera *Referer*. Si solamente se enviara a la víctima la *URL*, no redirigiría a esta, pues falta la cabecera que indica que la dirección viene del mismo dominio. Esto se soluciona utilizando la sección de comentarios de la propia *web*, donde dejando un mensaje con el *link* ya funcionaría, puesto que el usuario al clicar estaría yendo desde la propia dirección *web* y estaría haciendo la función del *Referer*.

<code>Link para descuetno del 20%: http://hojaytinta.lab/ads/tr?ci=el-jardin-de-sal-9f21&e=i&tu=https://ads.hojaytinta-partners.com@www.google.es&book=el-jardin-de-sal</code>

<img width="1353" height="792" alt="Captura de pantalla 2026-10-07 191012" src="https://github.com/user-attachments/assets/0ad06bde-c483-4719-bf63-03c4c019f7c5" />

Lo que haría el atacante sería crear una página *web* parecida a la original para redireccionar a la víctima a ella. Una vez en su página podría pedirle la información que desea robarle, como datos bancarios o contraseñas de inicio de sesión.

Con este post se pretende mostrar una vulnerabilidad que se podría encontrar en la vida real, más allá de los clásicos y educativos CTFs. Una vulnerabilidad donde un atacante podría hacer ir a la víctima a la página que él quisiera y cometer acciones maliciosas contra el atacado.
