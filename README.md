# Redmine: Entrega Final

Este documento describe los pasos necesarios para configurar y levantar Redmine en un servidor, así como para mantenerlo funcionando como un servicio.


## Pasos de Configuración

### 1. Configuración de la Base de Datos

Primero, debes crear una base de datos y un usuario con permisos adecuados. Para hacerlo, ejecuta los siguientes comandos en MySQL:

```sh
sudo mysql
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'secretPassword';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
FLUSH PRIVILEGES;
```


### 2. Configuración del Archivo `database.yml`

Luego, necesitas adaptar el archivo `database.yml` para conectarte correctamente a la base de datos de Redmine.

Desde el directorio raíz del proyecto, ejecuta:

```sh
cp /opt/redmine-5.0.11/config/database.yml.example /opt/redmine-5.0.11/config/database.yml
vi /opt/redmine-5.0.11/config/database.yml
```

En la sección `production`, asegúrate de que los valores de `username`, `password` y `database` coincidan con los datos de la base de datos que acabas de crear. Ejemplo:

```yaml
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "secretPassword"
  encoding: utf8mb4
```

### 3. Instalación de las Dependencias de Ruby

Redmine utiliza varias gemas de Ruby, por lo que necesitas instalar Bundler y las gemas necesarias:

```sh
cd /opt/redmine-5.0.11
gem install bundler
bundle install
```

### 4. Configuración Inicial de la Base de Datos

Ejecuta los comandos para generar un token secreto y realizar las migraciones de la base de datos. También carga los datos por defecto en español:

```sh
cd /opt/redmine-5.0.11

bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production REDMINE_LANG=es bundle exec rake redmine:load_default_data
```

### 5. Levantar el Servidor de Aplicaciones

Con las dependencias instaladas y la base de datos configurada, puedes levantar el servidor Puma, que escuchará en el puerto 3000:

```sh
RAILS_ENV=production bundle exec rails server
```

---

## Configuración de Nginx

Redmine debe estar configurado detrás de Nginx para manejar las peticiones web. Sigue estos pasos para hacerlo:

1. Copia la configuración de Nginx proporcionada en el repositorio:

   ```sh
   sudo cp /vagrant/nginx/redmine-demo.conf /etc/nginx/sites-available/
   sudo ln -s /etc/nginx/sites-available/redmine-demo.conf /etc/nginx/sites-enabled/
   ```

2. Deshabilita la configuración predeterminada de Nginx para evitar conflictos:

   ```sh
   sudo rm /etc/nginx/sites-enabled/default
   ```

3. Recarga el servicio de Nginx para aplicar los cambios:

   ```sh
   sudo systemctl reload nginx.service
   ```

Ahora, puedes acceder a Redmine desde tu navegador usando la dirección **`nuestra-ip:80`**.

---

## Configuración como Servicio

Para ejecutar Redmine como un servicio y mantenerlo corriendo en segundo plano, sigue estos pasos:

1. Copia el archivo de configuración del servicio:

   ```sh
   sudo cp /vagrant/service/redmine.service /etc/systemd/system/
   ```

2. Recarga el daemon de systemd para aplicar la nueva configuración:

   ```sh
   sudo systemctl daemon-reload
   ```

3. Inicia y habilita el servicio de Redmine para que se ejecute automáticamente al iniciar el sistema:

   ```sh
   sudo systemctl start redmine
   sudo systemctl enable redmine.service
   ```

### Nota sobre la Configuración de `bundle`

Durante las pruebas, se encontró que usar el **path completo** para `bundle` causaba un error. Se resolvió este problema usando el comando `which bundle`, lo que permitió encontrar la ubicación correcta de `bundle`. Como resultado, se añadió una variable de entorno `PATH` personalizada para asegurar que el servicio use la ubicación adecuada de `bundle`.

---

## Conclusión

Con estos pasos, deberías tener Redmine corriendo en tu servidor, accesible a través de Nginx y ejecutándose como un servicio. Si encuentras algún problema durante la configuración, revisa los logs de `systemd` para más detalles.

---
