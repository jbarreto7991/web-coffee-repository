# Coffee Website

## Install Web (Ubuntu 18.04)

**1. Instalación de aplicaciones**

 - sudo su
 - sudo apt-get update
 - sudo apt-get install php libapache2-mod-php -y
 - sudo apt-get install php-mysqli -y

**2. Descarga de código fuente**

 - cd /home/ubuntu
 - git clone https://github.com/jbarreto7991/web-coffee-repository.git
 - cd /home/ubuntu/web-coffee-repository

**3. Desplegando código en apache**

 - cp -r * /var/www/html
 - sudo /etc/init.d/apache2 restart

**4. Configuración apache**

 - chmod 777 /etc/php/7.2/apache2/php.ini
 - sed 's+;extension=mysqli+extension=mysqli+g' /etc/php/7.2/apache2/php.ini >> /etc/php/7.2/apache2/bk_php.ini
 - rm /etc/php/7.2/apache2/php.ini
 - mv /etc/php/7.2/apache2/bk_php.ini /etc/php/7.2/apache2/php.ini
 - sudo /etc/init.d/apache2 restart

**5. Validación index.php**

 - Validar la carga de la página index.php desde una instancia ubicada dentro de la red
 - curl http://PRIVATE_IP/index.php

**6. Validación Coffee.php**

 -  - Validar la carga de la página Coffee.php desde una instancia ubicada dentro de la red 
 - curl http://PRIVATE_IP/Coffee.php
 - Esta página se visualizará cuando la integración entre EC2 y RDS esté realizado



## MySQL Client

**1. Install MySQL**

 - sudo apt-get update
 - sudo apt-get install mysql-server -y
 - sudo service mysql status

    > Recordar "usuario" de RDS: $USUARIO_RDS
    > Recordar "contraseña" de RDS: $CONTRASEÑA_RDS
**2. Conexión a RDS**

 - mysql -u $USUARIO_RDS -h database-1.cmgkpsyjvnsf.us-east-2.rds.amazonaws.com -p

    > Ingresar $CONTRASEÑA_RDS
**3. Creación de base de datos "coffee"**

 - CREATE DATABASE coffee;

 - USE coffee;

  - CREATE TABLE IF NOT EXISTS `coffee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `type` varchar(255) DEFAULT NULL,
  `price` double DEFAULT NULL,
  `roast` varchar(255) DEFAULT NULL,
  `country` varchar(255) DEFAULT NULL,
  `image` varchar(255) DEFAULT NULL,
  `review` text,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=5 ;

 - SHOW TABLES;

 - INSERT INTO `coffee` (`id`, `name`, `type`, `price`, `roast`, `country`, `image`, `review`) VALUES
(1, 'Cafe au Lait', 'Classic', 2.25, 'Medium', 'France', 'Images/Coffee/Cafe-Au-Lait.jpg', 'A coffee beverage consisting strong or bold coffee (sometimes espresso) mixed with scalded milk in approximately a 1:1 ratio.'')'),
(2, 'Caffe Americano', 'Espresso', 3.25, 'Medium', 'Italy', 'Images/coffee/caffe_americano.jpg', 'Similar in strength and taste to American-style brewed coffee, there are subtle differences achieved by pulling a fresh shot of espresso for the beverage base.'),
(3, 'Peppermint White Chocolate Mocha', 'Espresso', 3.25, 'Medium', 'Italy', 'Images/coffee/white-chocolate-peppermint-mocha.jpg', 'Espresso with white chocolate and peppermint flavored syrups and steamed milk. Topped with sweetened whipped cream and dark chocolate curls.'),
(4, 'Galao', 'Latte', 4.2, 'Light', 'Portugal', 'Images/Coffee/galao_kaffee_portugal.jpg', 'Galao is a hot drink from Portugal made of espresso and foamed milk');

**4. Validar registros ingresados**

 - select * from coffee;


## Integración EC2 - RDS

**1. Configurar archivos de integración EC2 - RDS**

 - nano /var/www/html/Model/Credentials.php

**2. Modificar los siguientes valores:**

 - $host, con la ip privada del RDS
 - $user, con el usuario registrado en el RDS al momento de la creación
 - $passwd, con la contraseña registrada en el RDS al momento de la creación
 - $database, el valor debe ser "coffee". Según el database creado en pasos anteriores

**3. Reiniciar el servicio apache**

 - sudo /etc/init.d/apache2 restart


## Integración ALB - EC2

**1. Validación index.php**

 - Validar la carga de la página index.php 
 - http://DNS_ALB/index.php

**2. Validación Coffee.php**

 - Validar la carga de la página Coffee.php 
 - http://DNS_ALB/Coffee.php
 - Esta página se visualizará cuando la integración entre EC2 y RDS esté realizado

