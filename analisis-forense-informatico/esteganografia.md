# Esteganografía 

## ¿Qué es y para qué sirve?
**Esteganografía** es la técnica de ocultar mensajes o información embebiéndola dentro de un fichero, imagen u otro objeto contenedor, de forma que su propia existencia pase desapercibida.
* **Permite enviar información sensible sin levantar sospechas** a través de redes públicas o inseguras. 
*  **Marcas de agua digitales y DRM**: Se emplea para insertar información de copyright, autoría o seguimiento en archivos multimedia (como imágenes, audios o vídeos) sin alterar su calidad visual o sonora.   
* **Seguridad ofensiva y forense**: Se estudia en el ámbito de la ciberseguridad tanto para simular la exfiltración de datos por parte de atacantes como para ayudar a los analistas forenses a detectar artefactos u objetos ocultos.   

## Ejercicios prácticos (Steghide, Stegcracker, Stegseek)

### Ejercicio steghide 
>**Steghide**  Es la herramienta principal de esteganografía que se utiliza en Linux para ocultar y extraer datos de archivos multimedia (como imágenes o audios).

Esconder secreto y recuperarlo: Realice todos los pasos para insertar un texto secreto en una imagen. Puede seguir los mismos pasos que en el tutorial pero asegúrese de utilizar una imagen única y un texto único (es decir, imagen y texto que ninguno de sus compañeros se le ocurra usar para evitar posibles coincidencias. 


**Lo primero que realicé fue crear un archivo de texto secreto para embeberlo dentro de la imagen:**
 ![Texto alternativo](/img/secretotxt.png) 
**A continuación, mediante el comando steghide, incrusté y cifré el mensaje secreto dentro de la imagen contenedora:**
 ![Texto alternativo](/img/steghideembed2.png) 

**Mediante el comando steghide info yelyazaveta-img.jpeg, comprobé la correcta incrustación del secreto en la imagen contenedora. Al introducir el salvoconducto (contraseña), la herramienta confirmó la presencia del archivo secreto.txt (56 Bytes), cifrado con el algoritmo Rijndael-128 en modo CBC y comprimido:**
![Texto alternativo](/img/steginfo.png) 
 **Para recuperar y extraer la información oculta de la imagen, ejecuté el comando de extracción especificando el archivo contenedor mediante el parámetro -sf (Stego-File) y la contraseña correspondiente con -p:**
![Texto alternativo](/img/extractsteg.png)

### Ejercicio Stegcracker y Stegseek

>**Stegcracker**  es una herramienta de fuerza bruta basada en Python diseñada para romper contraseñas de archivos esteganografiados (especialmente los creados con steghide) utilizando un diccionario de palabras.


>**Stegseek**  Es una herramienta de alta velocidad escrita en C y C++ diseñada específicamente para realizar ataques de fuerza bruta en archivos ocultos con steghide, multiplicando masivamente el rendimiento de la búsqueda.

Craquear password: En el último paso se realiza un crackeo del password, debe de usted realizar 2 crakings y comparar cuál es el más rápido usando su ordenador, misma foto, mismo diccionario y mismo secreto.
 
Ejer 1 - Usando Stegcracker (calcular tiempo de craqueado)

Ejer 2 - Usando Stegseek: Debe de investigar cómo instalarlo, y cómo usarlo. Recuerde que tiene que calcular el tiempo de ejecución. Puede encontrar dicho programa en:   StegSeek 
 
**Paso previo: Crear un diccionario con varias contraseñas falsas y mi contraseña real**
![Texto alternativo](/img/pasopreviosteg.png)

**Paso 1: Instalación de las herramientas**

Para llevar a cabo el ejercicio de fuerza bruta, procedí a instalar la herramienta Stegcracker utilizando el gestor de paquetes de Python (pip3) en mi entorno de Ubuntu:
![Texto alternativo](/img/instalstegcracker.png)
A continuación, para instalar Stegseek descargué el paquete .deb de su repositorio oficial:
![Texto alternativo](/img/instalstegseek.png)
Finalimente instalé Stegseek en mi sistema:
![Texto alternativo](/img/instalstegseek2.png)


**Paso 2: Ejecución y cronometraje del ataque de fuerza bruta**

Tras ejecutar las pruebas de fuerza bruta, Stegseek demostró un rendimiento enormemente superior frente a Stegcracker. Mientras que Stegcracker (desarrollada en Python) requiere llamadas continuas al sistema que ralentizan el proceso (registrando un tiempo de 1,661 s para un diccionario pequeño), Stegseek (optimizada en C/C++) procesa miles de palabras por segundo. La diferencia de velocidad es abismal, posicionando a Stegseek como la herramienta indispensable en análisis forenses y auditorías de seguridad modernas.
![Texto alternativo](/img/timestegcracker.png)
![Texto alternativo](/img/timestegseek.png)