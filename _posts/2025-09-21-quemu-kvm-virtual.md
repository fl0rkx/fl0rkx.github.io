---
layout: single
title: Instalación de QEMU+KVM como virtualizador de máquinas principal. Alternativa a Virtualbox y VMWare en Linux. Parte I de II.
excerpt: "Siempre que hemos pensado en un virtualizador, dependiendo del SO donde estemos trabajando, nos viene a la mente Hype-V, Virtualbox o Vmware. Pero existe una alternativa bastante potente en Linux, que nos hará tener el rendimiento de una máquina en 'bare metal'"
date: 2025-09-21
classes: wide
header:
  teaser: /assets/images/quemu/QEMU-Logo.png
  teaser_home_page: true
categories:
  - Research
tags:
  - Virtualización
  - QEMU
  - Linux
---
> **⚠️ Aviso Legal Importante**  
>  
> La información contenida en este blog tiene fines exclusivamente informativos y de divulgación técnica.  
>  
> Todos los contenidos, procedimientos y referencias a terceros están debidamente citados cuando corresponda.  
>  
> No asumo responsabilidad alguna por daños, pérdidas, fallos o inconvenientes que puedan derivarse directa o indirectamente del uso de esta información, así como por la instalación o utilización de las herramientas y paquetes mencionados.  
>  
> El uso de dicha información y herramientas es bajo la completa y exclusiva responsabilidad del lector. Se recomienda realizar respaldos y tomar precauciones antes de aplicar cualquier procedimiento.  
>  
> Este aviso no sustituye asesoría profesional y no debe ser interpretado como garantía o recomendación específica.



Antes de hablar y empezar este post dejo una serie de enlaces que complementa la informacióin de la publicación y de las dudas que puedan surgir.

>[¿Qué es la virtualización?](https://www.ibm.com/think/topics/virtualization)

>[QEMU](https://www.qemu.org/)

>[KVM : ¿qué es y para qué sirve?](https://www.redhat.com/es/topics/virtualization/what-is-KVM)



# ¿Qué es QEMU y cuál es su función?

QEMU (Quick Emulator) es un emulador y virtualizador de hardware de código abierto que permite ejecutar sistemas operativos y aplicaciones de manera virtualizada. Puede emular distintas arquitecturas de hardware, lo que permite probar software sin necesidad de contar con el mismo hardware específico.

Tiene distintas funciones como:
- Emular arquitecturas ARM.
- Virtualizador de sistemas.
- Desarrollo cruzado.
Entre otras funciones.

De forma muy resumida es un virtualizador con esteroides.

# ¿Qué es KVM y cuál es su función?

KVM (siglas de Kernel-based Virtual Machine) es una tecnología de virtualización incluida en el núcleo de Linux. Permite convertir un sistema Linux en un hipervisor, es decir, en una plataforma capaz de crear y ejecutar máquinas virtuales (VMs) de manera eficiente y con alto rendimiento.

Al igual que QEMU, KVM tiene distintas funciones:
- Aprovecha la virtualización del hardware.
- Gestionar recursos del sistema.
- Mantener el aislamiento y la seguridad.

# ¿Por qué se utiliza QEMU + KVM?

KVM se usa normalmente junto con QEMU, que se encarga de emular el hardware virtual de las máquinas (CPU, disco, red, etc.). Mientras tanto, KVM se encarga de ejecutar esas máquinas usando el hardware real, lo que mejora el rendimiento enormemente.


| Ventajas                          | Descripción                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|
| **Alto rendimiento**             | Muy cerca del rendimiento nativo gracias al uso del hardware del procesador. |
| **Código abierto y gratuito**    | Totalmente libre, sin costos de licencia, y con una gran comunidad.        |
| **Multiplataforma**              | Soporta Linux, Windows, BSD, y otros sistemas operativos.                  |
| **Escalable**                    | Ideal para servidores, entornos en la nube y centros de datos.             |
| **Seguro y estable**             | Integrado en el kernel de Linux, con actualizaciones cons


# Desventajas de funcionalidad respecto al SO
## Windows

Obviamente QEMU se puede instalar tanto en Windows como en Linux, pero no es posible usarlo junto con KVM hasta el dái que estoy escribiendo esto, por el simple motivo que estamos usando una tecnología exclusiva de Linux. Esto es porque Windows usa su propia tecnología de virtualización "Hyper-V".

## Linux

La serie de desventajas por usarlo en Linux, no se encuentran tanto en la funcionalidad, si no en la curva de aprendizaje. Estamos hablando que es un virtualiazador algo más avanzado y podría ser algo más difícil para los usuarios que no tienen experencia.


Con la puesta de conocimiento previa, pasamos a la parte más entretenida. La instalación y configuración de ciertos parámetros.

# Instalación 

Abrimos nuestra terminal y ponemos los siguiente comandos:

```bash
sudo apt update && sudo apt upgrade
sudo apt install qemu-system libvirt-daemon-system virt-manager mesa-utils mesa-utils-extra libgl1-mesa-dri nvidia-driver-libs nvidia-kernel-dkms
```
Con esta serie de comandos instalaremos los paquetetes correspondientes para el uso de QEMU+KVM y un visualizador gráfico para aquellos que no se sientan comodos en la terminal de momento.

Añado de que en mi configuración de hardware tengo una gráfica NVIDIA de la que me dispongo usar para mejorar el rendimiento más aún de la virtualización.

Para aquellas personas que no quieran utiliar su gráfica o no dispongan pueden pasar directamente al siguiente paso correspondiente. -->

Dicho esto, miramos si tenemos los drivers de nuestra gráfica:

```bash
glxinfo | grep "OpenGL"
OpenGL vendor string: Intel OpenGL renderer string: Mesa Intel(R) UHD Graphics 630 (CFL GT2) OpenGL core profile version string: 4.6 (Core Profile) Mesa 22.3.6 OpenGL core profile shading language version string: 4.60 OpenGL core profile context flags: (none) OpenGL core profile profile mask: core profile OpenGL core profile extensions: OpenGL version string: 4.6 (Compatibility Profile) Mesa 22.3.6 OpenGL shading language version string: 4.60 OpenGL context flags: (none) OpenGL profile mask: compatibility profile OpenGL extensions: OpenGL ES profile version string: OpenGL ES 3.2 Mesa 22.3.6 OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20 OpenGL ES profile extensions
```
Cómo podemos ver no tenemos los drivers de NVIDIA, por lo que debemos de instalarlos.

Añadimos los respositorios correspondiente en nuestro archivo `/etc/apt/sources.list`. En mi caso estoy usando Debian 12+1.

```shell
sudo nano /etc/apt/sources.list

deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```
Sabemos los paquetes en cuanto a drivers que tenemos que instalar, pero podemos usar una herramienta que nos facilite un poco más para saber cual debemos de instalar de NVIDIA:
```shell
sudo apt install nvidia-detect
```

Ahora lo siguiente es poner:

```shell
sudo apt update
sudo apt install nvidia-driver firmware-misc-nonfree
sudo apt install nvidia-smi
```

Cuando lo tengamos instalado, debemos de reiniciar no sin antes hacer lo siguiente:

```shell
sudo apt update
sudo apt install nvidia-driver firmware-misc-nonfree
sudo apt install nvidia-smi
```
REINICIO

Una vez reiniciado debemos de poner el siguiente comando para comprobar que tenemos los drivers instalados:

```shell
lspci -k | grep -EA3 'VGA|3D|Display'
00:02.0 VGA compatible controller: Intel Corporation CoffeeLake-H GT2 [UHD Graphics 630]
	DeviceName: Onboard - Video
	Subsystem: Micro-Star International Co., Ltd. [MSI] Device 127d
	Kernel driver in use: i915
--
01:00.0 3D controller: NVIDIA Corporation GP107M [GeForce GTX 1050 Mobile] (rev a1)
	Subsystem: Micro-Star International Co., Ltd. [MSI] Device 127d
	Kernel driver in use: nvidia
	Kernel modules: nvidia
```
Por el tipo de gráfica que tengo uso de forma prioritaria Intel para ahorrar energía y uso NVIDIA para tareas más pesadas. Para hacer que funcione la gráfica en las tareas que quiera debemos de instalar los siguientes paquetes:

```shell
sudo apt install nvidia-prime
sudo apt update
sudo apt install bumblebee-nvidia primus
sudo usermod -aG bumblebee $USER

primusrun glxinfo | grep "OpenGL renderer"

```  

Lo más probable es que nos salga un error como este -->

```log
primusrun glxinfo | grep "OpenGL renderer" primus: fatal: Bumblebee daemon reported: error: [XORG] (EE) Unable to locate/open config directory: "/etc/bumblebee/xorg.conf.d"
```
Debemos de poner crear un archivo de configuración:

```shell

sudo mkdir -p /etc/bumblebee/xorg.conf.d

sudo nano /etc/bumblebee/xorg.conf.d/10-nvidia.conf

Section "ServerLayout"
    Identifier "Layout0"
    Option "AutoAddDevices" "false"
EndSection

Section "Device"
    Identifier  "DiscreteNvidia"
    Driver      "nvidia"
    VendorName  "NVIDIA Corporation"
    BusID       "PCI:1:0:0"
    Option      "NoLogo" "true"
EndSection

⚠️ **Ojo con el BusID**:  
Tú lo viste en tu `lspci`: Está representado con xx:xx.x 3D Controller...
``` 
```bash
sudo systemctl restart bumblebeed
```

Antes de volver a reiniciar debemos de indicar al kernel que vamos a cambiar:

```shell
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf

sudo update-initramfs -u
```

**Reiniciamos**

Después de reinicio debemos de poner en la terminal -->

```bash
lspci -k | grep -EA3 'VGA|3D|Display'
```
Debe de salir algo como `Kernel driver in use: nvidia`

y con `Primusrun` --> 

```bash
primusrun glxinfo | grep "OpenGL renderer"
```
Debe salir algo como `OpenGL renderer string: NVIDIA GeForce GTX 1050/PCIe/SSE2`

## Configurar acceso directo para iniciar `virt-manager` con `primusrun`

```bash
nano ~/.local/share/applications/virt-manager-nvidia.desktop

[Desktop Entry]
Name=Virt-Manager (NVIDIA)
Comment=Virtual Machine Manager con primusrun
Exec=primusrun virt-manager
Icon=virt-manager
Terminal=false
Type=Application
Categories=System;Emulator;X-Red-Hat-Base;
```
Cuando iniciemos el programa con nuestra máquina virtual podemos usar --> `watch -n 1 nvidia-smi` para comprobar que tenemos un proceso que está usando la GPU.

Para forzar más aún la comprobación dentro de nuestra máquina virtual Linux, debemos de poner `glxgears` y en nuestro host el comando anterior para comprobar que efectivamente está en uso.

Es posible que con el comando anterior no muestre nada, pero si usamos la aceleración de gráficos es la única manera de que funcione.


En la siguiente parte hablaremos de una configuración estandar para iniciar una máquina desde una ISO o desde un archivo `.qcow2`, además de solucionar los posibles errores que puedan presentarse.










