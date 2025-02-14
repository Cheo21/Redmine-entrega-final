# Redmine: entrega final

Tener en cuenta para configurar:
* Configurar base de datos.
* Servidor
  * App server
    * levantar el servidor puma. 
  * Web Server
    * Configurar el nginx para servir staticos y pasar el resto de peticiones a Puma.
* Levantarlo como servicio

---

### Configurar Database

Crear una base de datos y un usuario con password para Redmine. Ejemplo: 

```sh
sudo mysql
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'secretPassword';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
FLUSH PRIVILEGES;
```
### Adaptar database.yml para conectarse correctamente a la db

Desde el root del proyecto:
```sh
cp /vagrant/redmine-5.0.11/config/database.yml.example /vagrant/redmine-5.0.11/config/database.yml
vi /vagrant/redmine-5.0.11/config/database.yml
```
En la seccion de production verificar que coincida con el usuario, password y base de datos
Ejemplo:
```sh
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "secretPassword" 
  # Use "utf8" instead of "utfmb4" for MySQL prior to 5.7.7
  encoding: utf8mb4
  ```


### Instalar las dependencias ruby
```sh
cd /vagrant/redmine-5.0.11
gem install bundler
bundle install
```

### Instalar las dependencias ruby
```sh
cd /vagrant/redmine-5.0.11

bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production REDMINE_LANG=es bundle exec rake redmine:load_default_data
```

### Levantar el app server:
Ya con eso podriamos levantar al app server que quedara escuchando en el puerto 3000
```sh
RAILS_ENV=production bundle exec rails server
```
---
### Añadir la configuracion de nginx que se encuentra en el repo
```sh
sudo cp /vagrant/nginx/redmine-demo.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/redmine-demo.conf /etc/nginx/sites-enabled/
## De momento para probar deshabilitaremos la configuracion por default
sudo rm /etc/nginx/sites-enabled/default

## Refrescamos el servicio de nginx para que tome las nuevas configuraciones.
sudo systemctl reload nginx.service
```
Ya con esto lo podriamos probar buscando en el navegador: **nuestra-ip:80** 


## Para mantenerlo como un servicio copiamos la configuracion dada en la carpeta /vagrant/service

```sh
sudo cp /vagrant/service/redmine.service /etc/systemd/system/
##Aplicamos la configuracion
sudo systemctl daemon-reload
sudo systemctl start redmine
sudo systemctl enable redmine.service

```
Nota: Se intento colocando el path completo donde se encuentra el comando bundle, sin embargo daba error. Se probo usando ***which bundle*** y se encontro otro lugar donde se ejecuta. Por ende se creo una variable de entorno PATH añadiendo varias opciones a donde buscar.
