# ROS-OVER-RPI-4
PROCESO DE INTALACION VALIDADO
Paso 1: instalar dependencias y descargar los paquetes

% sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

% sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

% sudo apt-get update
% sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential  cmake


inicializar rosdep y actualízar

% sudo rosdep init
% rosdep update
Paso 2: solucionar los problemas
Cuando haya terminado, creemos un espacio de trabajo dedicado a catkin para construir ROS:
% mkdir ~/ros_catkin_ws

% cd ~/ros_catkin_ws

Ahora tienes dos opciones:

ROS-Comm: instalación (Bare Bones): podría ser la opción preferida para Raspberry Pi, ya que probablemente lo ejecutará sin entorno grafico ,para el robot seria lo mejor pero es m as complejo. No incluye RVIZ, lo que hace que el proceso de instalación sea más corto y menos complicado.

Instalación de escritorio: incluye herramientas GUI, como rqt, rviz y bibliotecas genéricas de robots.
Como se esta en pruebas se hara la instalacion de escritorio:

% rosinstall_generator desktop --rosdistro melodic --deps --wet-only --tar > melodic-desktop-wet.rosinstall  

% wstool init -j8 src melodic-desktop-wet.rosinstall

El comando tardará unos minutos en descargar todos los paquetes ROS principales en la carpeta src. Si wstool init falla o se interrumpe, puede reanudar la descarga ejecutando:

% wstool update -j 4 -t src

% mkdir -p ~/ros_catkin_ws/external_src
% cd ~/ros_catkin_ws/external_src
% wget    http://sourceforge.net/projects/assimp/files/assimp-3.1/assimp-3.1.1_no_test_models.zip/download -O assimp-3.1.1_no_test_models.zip
% unzip assimp-3.1.1_no_test_models.zip
% cd assimp-3.1.1
% cmake .
% make
% sudo make install
% cd ..
% sudo apt-get install  libogre-1.9-dev

Finalmente, necesitaremos solucionar los problemas con libboost. Estoy usando la solución de esta publicación en stackoverflow:

"Los errores durante la compilación son causados ​​por la función‘ boost :: posix_time :: milliseconds ’que
en las versiones más recientes de boost, acepta solo un argumento entero, pero el paquete actionlib en ROS le da un flotante en varios lugares. Puede listar todos los archivos usando esa función:

% find -type f -print0 | xargs -0 grep 'boost :: posix_time :: milisegundos' | cut -d: -f1 | sort -u
Ábrelos en tu editor de texto y busca la llamada a la función "boost :: posix_time :: milisegundos".
y reemplace llamadas como esta:
boost :: posix_time :: milisegundos (loop_duration.toSec ​​() * 1000.0f));
con:
boost :: posix_time :: milisegundos (int (loop_duration.toSec ​​() * 1000.0f)));
y estos:
boost :: posix_time :: milisegundos (1000.0f)
con:
boost :: posix_time :: milisegundos (1000)

Te recomiendo que uses el editor de texto nano, que es más simple Ctrl + O está guardando, Ctrl + X está saliendo y Ctrl + W está buscando.
A continuación, usamos la herramienta rosdep para instalar el resto de las dependencias:

% rosdep install --from-paths src --ignore-src --rosdistro melodic -y

Paso 3: compile y obtenga la instalación
% sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic -j2

Si el proceso de compilación se congela (muy probablemente, si instala la versión de escritorio), se necesita aumentar el espacio de intercambio disponible. De forma predeterminada, es de 100 MB, intenta aumentarlo a 2048 MB.

Todo el proceso de compilación dura aproximadamente 1 hora.

Ahora ROS Melodic debería estar instalado en la Raspberry Pi 4. 
se tendra la nueva instalación con el siguiente comando:

% echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc

Paso 4: Instalar el RPLIDAR ROS

En la carpeta home se crea otra carpeta

% mkdir -p ~/catkin_ws/src
% cd ~/catkin_ws/
% catkin_make
% echo "source $HOME/catkin_ws/devel/setup.bash" >> ~/.bashrc
% cd src
% sudo git clone  https://github.com/Slamtec/rplidar_ros.git
% catkin_make

Se espera la terminacion y se prueba con: 
% roslaunch rplidar_ros rplidar.launch

Para esoejecutar ROS en varias maquinas:

Para esta parte,se  necesita pc con Ubuntu 18.04 (O en una maquina virtual) con ROS Melodic instalado. en ubuntu, ROS se puede instalar simplemente usando apt-get como se describe en este tutorial.

http://wiki.ros.org/melodic/Installation/Ubuntu

Ahora las instalaciones de ubuntu y raspbian deben quedar en la misma red:

Ejecute roscore en del pc y exporte ROS_MASTER_URI

% roscore
% export ROS_MASTER_URI=http://[IP-DEL-PC]:11311

En la raspberry se ejecuta:

% export ROS_MASTER_URI=http://[IP-DEL-PC]:11311
% export ROS_IP=[IP-DEL-PC]

y se lanza el lidar en la Rpi:
% roslaunch rplidar_ros rplidar.launch

Si se inicia con éxito, verifique los topicos presentes en el pc  con la lista rostopic

Si se pueden ver / escanear mensajes, todo funciona como se supone que debe funcionar. Luego,se da el comando RVIZ en el pc, y se agregan los mensajes de escaneo láser y elije / escanear tema. También deberá cambiar el marco fijo a / láser

























