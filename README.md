# Image Builder Tutorial

Tutorial básico para generar una imágen de redhat usando Image Builder en un servidor Red Hat 9.

La imágen se genera a partir de un blueprint en TOML, incluido en el repositorio.

## Instalación

Se debe habilitar cockpit que provee la interfaz web para el image-builder
```
# sudo systemctl enable --now cockpit.socket
```
Instalar los paquetes necesarios y habilitar el socket de composer para interactuar por CLI
```
# sudo dnf install osbuild-composer \
                   composer-cli \
                   cockpit-composer \ 
                   bash-completion

# sudo systemctl enable --now osbuild-composer.socket
```
Para que los usuarios puedan subir blueprints y crear imágenes, agregarlos al grupo weldr

```
# sudo vim /etc/group
...
weldr:x:986:admin  #ejemplo en mi laboratorio
...
```

Copiar la configuración de repositorios que vamos a usar, según la versión de Red Hat que queremos generar. Reiniciar el osbuild-composer.

```
# sudo cp /usr/share/osbuild-composer/repositories/rhel-9.* /etc/osbuild-composer/repositories/

# sudo systemctl restart osbuild-composer
```

## Uso

### Crear un blueprint 
Crear un blueprint en formato [TOML](https://toml.io/en/)

En el repositirio hay un blueprint básico, y la referencia está [acá](https://osbuild.org/docs/user-guide/blueprint-reference/)

Para generar contraseñas puede usar openssl 

```
$ openssl passwd -6 # Proveer la password por línea de comandos.
                  # -6 genera un hash SHA-512
```
o mkpasswd
```
$ mkpasswd -m sha-512
```
El hash generado se incluye en el blueprint. 

### Subir el blueprint al repositorio
```
$ composer-cli blueprints push basic_blueprint_rhel9.toml 
```
Para ver los blueprints que tiene subidos

```
$ composer-cli blueprints list

$ composer-cli blueprints show rhel9-basic
```
### Generar y descargar una imágen

Ver los tipos de imágen que se pueden generar
```
$ composer-cli compose types
```
Para generar una imagen se debe especificar el blueprint y el tipo de imágen. La imágen se genera en el servidor, no como archivo.

```
$ composer-cli compose start rhel9-basic vmdk
```
Para ver las imágenes generadas puede usar el subcomando status. 
```
$ composer-cli compose status
```
![Imágenes Generadas](/images/ImagenGenerada.png)

Para descargar la imágen generada
```
$ composer-cli compose image 92bf38ef-ced1-4355-a9cf-bdbc91cf0534 --filename /var/tmp/rhel9-basic.vmdk
```
Puede acceder a la WebUI con la url https://<hostname>:9990
![Image Builder WebUI](/images/ImageBuilderWebUI.png)

## Referencias
[Que es image builder](https://www.redhat.com/en/topics/linux/what-is-an-image-builder)

[Manual de Red Hat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/composing_a_customized_rhel_system_image/index)

## Licencia

[CC-BY-4.0](https://choosealicense.com/licenses/cc-by-4.0)