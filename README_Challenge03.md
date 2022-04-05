# Coffee Website

## Install Web (Ubuntu 18.04)

**1. Instalación de aplicaciones (APP A y APP B)**

 - sudo su
 - sudo apt-get update
 - sudo apt-get install php libapache2-mod-php -y
 - sudo apt-get install php-mysqli -y

**2. Descarga de código fuente (APP A y APP B)**

 - cd /home/ubuntu
 - git clone https://github.com/jbarreto7991/web-coffee-repository.git
 - cd /home/ubuntu/web-coffee-repository

**3. Personalización de proyecto**

*3.1. APP A*
 - mkdir /var/www/html/appa
 - cp -r * /var/www/html/appa
 - sudo /etc/init.d/apache2 restart

*3.2. APP B*
 - mkdir /var/www/html/appb
 - cp -r * /var/www/html/appb
 - sudo /etc/init.d/apache2 restart


**4. Configuración apache (APP A y APP B)**

 - chmod 777 /etc/php/7.2/apache2/php.ini
 - sed 's+;extension=mysqli+extension=mysqli+g' /etc/php/7.2/apache2/php.ini >> /etc/php/7.2/apache2/bk_php.ini
 - rm /etc/php/7.2/apache2/php.ini
 - mv /etc/php/7.2/apache2/bk_php.ini /etc/php/7.2/apache2/php.ini
 - sudo /etc/init.d/apache2 restart

**5. Modificación de puerto apache2**

*5.1. APP A*
 - nano /etc/apache2/ports.conf 
 - #Modificar puerto a 8081
 - sudo /etc/init.d/apache2 restart

*5.2. APP B*
 - nano /etc/apache2/ports.conf 
 - #Modificar puerto a 8082
 - sudo /etc/init.d/apache2 restart


**6. Validación index.php (APP A y APP B)**

 - Validar la carga de la página index.php desde una instancia ubicada dentro de la red
 - curl http://PRIVATE_IP/appa/index.php (APP A)
 - curl http://PRIVATE_IP/appb/index.php (APP B)

**7. Validación Coffee.php (APP A y APP B)**

 - Validar la carga de la página Coffee.php desde una instancia ubicada dentro de la red 
 - curl http://PRIVATE_IP/appa/Coffee.php
 - curl http://PRIVATE_IP/appb/Coffee.php
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


## Load Testing EC2

 - apt-get install stress
 - stress --cpu 3


## Athena

 - Query creación de tabla Athena

 - CREATE EXTERNAL TABLE IF NOT EXISTS vpcflowlog (
version string, 
account_id string, 
interface_id string, 
srcaddr string, 
dstaddr string, 
srcport int,
dstport int,
protocol int,
packets int,
bytes int,
start_time string,
end_time string,
action string,
log_status string 
  ) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
'serialization.format' = ' ',
'field.delim' = ' '
)
LOCATION 's3://{bucket_name}/AWSLogs/';


 - Query Select de tabla Athena

 - select
interface_id,srcaddr,dstaddr,srcport,dstport,protocol,action,log_status
from vpcflowlog 
where version not in ('version') 
order by interface_id,action,log_status;
