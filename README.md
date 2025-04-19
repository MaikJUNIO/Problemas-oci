<h2>❗ Problema: Instalación con <code>yum</code> termina con "Killed"</h2>

<p>Durante la instalación del servidor web Apache (<code>httpd</code>) en una instancia de Oracle Cloud Infrastructure (OCI), al ejecutar el siguiente comando:</p>

<pre><code>sudo yum install -y httpd</code></pre>

<p>Se observa que el proceso comienza normalmente descargando los paquetes:</p>

<pre><code>
Ksplice for Oracle Linux 8 (x86_64)     21 MB/s |  19 MB     00:00
MySQL 8.4 Server Community              6.1 MB/s | 1.0 MB     00:00
MySQL 8.4 Tools Community               3.0 MB/s | 420 kB     00:00
MySQL Connectors Community              391 kB/s |  44 kB     00:00
Oracle Software for OCI users          36 MB/s  | 200 MB     00:05
Killed
</code></pre>

<p>El proceso es abruptamente finalizado con el mensaje:</p>
<pre><code>Killed</code></pre>

![image](https://github.com/user-attachments/assets/fbf8989e-9aec-4808-a58f-4e51f0fdbc36)

<h3>🔍 Diagnóstico: El sistema se quedó sin memoria (OOM)</h3>

<pre><code>dmesg | tail -n 20</code></pre>

<p>Al revisar el log del sistema con <code>dmesg</code>, se encuentra la siguiente línea:</p>

<pre><code>
[ 1437.269406] oom-kill:constraint=CONSTRAINT_NONE,...,task=yum,pid=4181,uid=0
[ 1437.359407] Out of memory: Killed process 4181 (yum) ...
</code></pre>

![image](https://github.com/user-attachments/assets/84e2ecf6-7c08-44b4-aa78-1d423ebaa26e)


<p>Esto confirma que el sistema ejecutó el <strong>OOM Killer</strong> ("Out Of Memory Killer"), un mecanismo del kernel de Linux que finaliza procesos cuando la memoria RAM se agota por completo. En este caso, el proceso <code>yum</code> fue eliminado para liberar memoria.</p>

<p>Este comportamiento es común en las instancias gratuitas de OCI como <strong>VM.Standard.E2.1.Micro</strong>, que tienen solo <strong>1 GB de RAM</strong> y <strong>no tienen memoria swap configurada por defecto</strong>.</p>

<h3>✅ Solución: Configurar memoria swap manualmente</h3>

<p>Para evitar que el sistema se quede sin memoria en instalaciones futuras, puedes crear un archivo de <code>swap</code> de 1 GB:</p>

<pre><code># Crear archivo de 1GB para swap
sudo fallocate -l 1G /swapfile

# Asignar permisos adecuados
sudo chmod 600 /swapfile

# Formatear como swap
sudo mkswap /swapfile

# Activar swap inmediatamente
sudo swapon /swapfile

# Verificar que esté activo
swapon --show
</code></pre>

<h2>Configurar el swap permanente en Linux</h2>

<p>Abre el archivo <code>/etc/fstab</code> con un editor de texto como <code>nano</code> o <code>vim</code>:</p>

<pre><code>sudo nano /etc/fstab</code></pre>

<p>Agrega la siguiente línea al final del archivo para que el archivo swap se monte automáticamente:</p>

<pre><code>/swapfile swap swap defaults 0 0</code></pre>

<p>Guarda el archivo (en <code>nano</code>, presiona <kbd>Ctrl</kbd> + <kbd>O</kbd> para guardar y <kbd>Ctrl</kbd> + <kbd>X</kbd> para salir).</p>

<p>Luego, puedes activar el swap con el siguiente comando:</p>

<pre><code>sudo swapon -a</code></pre>

![image](https://github.com/user-attachments/assets/0093e41e-248f-4d56-b80b-9a2e4d01513c)

<p>Con esto, tu archivo swap se montará automáticamente al reiniciar el sistema.</p>

<h2>Actualizar el sistema</h2>

<p>Es recomendable actualizar el sistema antes de instalar nuevos paquetes. Este paso puede tardar entre <strong>10 a 20 minutos</strong> dependiendo de tu conexión y recursos disponibles.</p>

<pre><code>sudo yum update -y</code></pre>

![image](https://github.com/user-attachments/assets/3b6182dd-fd62-4df3-abd1-1b44deaa9c6d)


<h2>Instalar Apache (httpd)</h2>

![image](https://github.com/user-attachments/assets/0e9d1a4c-0781-45f0-98e5-9476bfd51ebc)


<p>Luego de actualizar, instala el servidor web Apache con el siguiente comando. Esta instalación puede tardar algunos minutos.</p>

<pre><code>sudo yum install httpd -y</code></pre>

<p>Una vez finalizada la instalación, puedes iniciar y habilitar el servicio con los siguientes comandos:</p>

<pre><code>sudo systemctl start httpd
sudo systemctl enable httpd</code></pre>


<h3>📝 Recomendación</h3>

<p>Es recomendable realizar este procedimiento <strong>antes de usar yum o dnf para instalar paquetes grandes</strong> en entornos con recursos limitados.</p>
<p>También se puede considerar cerrar procesos innecesarios o ampliar la instancia si el uso es constante y elevado.</p>


<h2>Configurando la instancia 2</h2>

![image](https://github.com/user-attachments/assets/7f9cd42b-4cba-4bb3-bf68-5aac4011cf25)

![image](https://github.com/user-attachments/assets/97548229-62fd-4429-ac0b-216877d5f267)

![image](https://github.com/user-attachments/assets/43cb2cba-8491-4bc2-86a7-a37479213222)


















