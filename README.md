# Open-redirect

<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
</div>

En este primer post de la sección de **Bug bounty** se va a llevar a cabo la explicación de qué es y de cómo funciona un *open redirect*, un tipo de vulnerabilidad recurrente en las plataformas de recompensa por fallos.

Un *open redirect* es una vulnerabilidad que permite, dentro de una aplicación web, redirigir a los usuarios a otra URL proporcionada por la falta de una validación adecuada de un parámetro. Lo que hacen los ciberdelincuentes es generar una dirección, aparentemente legítima, para que el usuario se confíe entrando en ella y sufran un ataque de *phishing*.

Para llevar a cabo este ejercicio he utilizado la plataforma [Docker labs](www.dockerlabs.es)
