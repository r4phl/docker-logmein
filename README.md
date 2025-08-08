Hola. =)

Para la instalacion de este programa, que solo esta para windows les dare un scritp de docker que ejecuta un wine
Tienen que saber que primero deben descargar el programa logmein de windows en un un OS de windows y luegooooo, sacar el programa de las rutas de donde lo guarda windows.

Se que esta parte se puedes por el mismo wine, pero es mucho complique que nos saltamos si solo sacamos estos archivos de la instalacion de windows. Total, solo necesitamos copiar los archivos .exe
Que serian estos archivos:


    ├── LogMeIn Rescue AVI Codec
    │   ├── racode64.ax
    │   └── racodec.ax
    └── LogMeIn-Rescue-Technician-Console
        ├── LogMeInRescueTechnicianConsole_x64
        │   ├── CustomSRUploader.cmd
        │   ├── LMIGuardianDll.dll
        │   ├── LMIGuardianEvt.dll
        │   ├── LMIGuardianSvc.exe
        │   ├── LMIProxyHelper.exe
        │   ├── LMIRSrv.dll
        │   ├── LMIRTechConsole.exe
        │   ├── MediaClientLib.dll
        │   ├── racodec64.ax
        │   ├── racodec86.ax
        │   ├── ractrlkeyhook.dll
        │   ├── rahook.dll
        │   └── zip.exe
        └── x86
            ├── LMIGuardianDll.dll
            ├── LMIGuardianEvt.dll
            └── LMIGuardianSvc.exe

Una vez los tengan copiados, les muestro cual es el scritp de docker para que funcione el programa.
Por favor, dejen recomendaciones o sus comentarios.

        ##########################################################################################################################
        FROM  debian:latest
        
        ENV DEBIAN_FRONTEND noninteractive
        
        RUN apt-get update -y && \
            apt-get install -y --no-install-recommends \
                software-properties-common \
                ca-certificates \
                locales \
                locales-all \
                wget
        
        # Install Wine
        RUN dpkg --add-architecture i386
        RUN mkdir -pm755 /etc/apt/keyrings
        RUN wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
        RUN wget -nc -P /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/debian/dists/bookworm/winehq-bookworm.sources
        RUN apt-get update
            # Wine 7.0 stable has some issues with some games I tested
            # Use Wine 7.11 staging instead
        RUN apt-get install -y --install-recommends winehq-staging
        
        # GStreamer plugins
        RUN apt-get update -y && \
            apt-get install -y --install-recommends \
                gstreamer1.0-libav:i386 \
                gstreamer1.0-plugins-bad:i386 \
                gstreamer1.0-plugins-base:i386 \
                gstreamer1.0-plugins-good:i386 \
                gstreamer1.0-plugins-ugly:i386 \
                gstreamer1.0-pulseaudio:i386
        
        # Install dependencies for display scaling
        RUN apt-get update -y && \
            apt-get install -y --install-recommends \
                build-essential \
                bc \
                git \
                xpra \
                xvfb \
                python3 \
                python3-pip
        
        # Install driver for Intel HD graphics
        RUN apt-get -y install libgl1-mesa-glx libgl1-mesa-dri
        
        ENV HOME /home/user
        RUN useradd --create-home --home-dir $HOME user \
        	&& chown -R user:user $HOME
        
        USER user
        
        ENV LC_ALL en_US.UTF-8
        ENV LANG en_US.UTF-8
        RUN export LANG=en_US.UTF-8
        
        # Make sure the terminal is still English
        ENV LANGUAGE en_US.UTF-8
        
        # Configuracion de Audio en docker
        #RUN winetricks sound=alsa
        
        RUN WINEARCH=win64 
        
        RUN WINEPREFIX=~/.wine64 winecfg
        
        ENV WINE_DISABLE_SHM=1
        ENV WINEDLLOVERRIDES="mscoree=d;mshtml=d"
        ENV LIBGL_ALWAYS_SOFTWARE=1
        ENV GALLIUM_DRIVER=llvmpipe
        
        # Ejecucion de logmein
        CMD wine /media/___USUARIO___/___CARPETA___/Logmein/7.50/LogMeIn-Rescue-Technician-Console/LogMeInRescueTechnicianConsole_x64/LMIRTechConsole.exe


IMPORTANTE:

Tienes que revisar las rutas de donde guardaste las carpetas de instalacion de donde copiaste el logmein del windows, con eso solo tienes que hacer dos comandos adicionales:

Construir el docker
    
    docker build . -t logmein

Ejecuar el docker con permisos para que suenen las alertas y tengans acceso a los archivos internos por si quieres compartir archivos:

    docker run -it --device=/dev/snd  --device=/dev/dri -e DISPLAY=$DISPLAY  -v /tmp/.X11-unix:/tmp/.X11-unix:ro --ipc=host -v /home/:/media  -v /etc/alsa:/etc/alsa -v "/usr/share/alsa:/usr/share/alsa" -v "/home/usuario/.config/pulse:/home/user/.config/pulse" -v "/run/user/$UID/pulse/native:/run/user/$UID/pulse/native" --env "PULSE_SERVER=unix:/run/user/$UID/pulse/native" --user "$(id -u)" --name logmein -h logmein  logmein

Aqui tambien tienes que revisar las rutas de tu programa, en mi caso yo uso debian y alsa, entonces tienes que ver tus controladores de audio para conectarlos con el docker.
Bueno, espero que les guste... :)
