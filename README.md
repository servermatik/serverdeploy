# SERVERDEPLOY SUBNORMAL


Ejemplo de configuración de docker-compose para Tomcat 8.5 con Keycloak 12.0

Docker docker-compose

Configuración de DNS. Este ejemplo usa [LE] [Let's Encrypt] para certificados SSL.

Crear nombres host dominio: servermatik, mysql, tomcat, keycloak, wordpress

Acceso a Internet entrante para el acceso Let's Encrypt. Edite .env y cambie la configuración


Tutorial:

Cree entradas de hosts como se indica arriba.


Acceso a Internet entrante para el acceso Let's Encrypt. Edite .env y cambie la configuración





Cree entradas de hosts como se indica arriba.


     docker volume create --name=mysql-data
     docker volume create --name=wordpress-data




Inicie el servidor Keycloak. Debería tardar entre 20 y 30 segundos.


  
    docker-compose up -d keycloak



Espera un poco. Verifique sus registros y espere a que Keycloak se conecte.

Inicie sesión en el master realm en Keycloak. Esto guardará las credenciales en ~ / .keycloak / kcadm.config dentro del contenedor.



    docker-compose exec keycloak kcadm.sh config credentials \
    --server http://localhost:8080/auth --realm master \
    --user keycloak --password keycloak
    
    
    
Crear demo realm

    docker-compose exec keycloak kcadm.sh create realms \
    -s realm=tomcat-keycloak -s enabled=true -o
    
    
    
Crear  demo client (para Tomcat)

    docker-compose exec keycloak kcadm.sh create clients \
    -r tomcat-keycloak \
    -s clientId=tomcat-client \
    -s publicClient=true \
    -s directAccessGrantsEnabled=true \
    -s 'rootUrl=http://tomcat.sofn.aptplatforms.com/tomcat-client' \
    -s 'redirectUris=[ "/roles/*", "/index.html", "/" ]' \
    -i
    
    
    
Cree el usuario de demostración y establezca una contraseña temporal

      docker-compose exec keycloak kcadm.sh create users \
    -r tomcat-keycloak -s username=tester -s enabled=true

     docker-compose exec keycloak kcadm.sh set-password \
    -r tomcat-keycloak --username=tester --new-password tester --temporary
    
    
Cree los roles de demostración y asígnelos al usuario. Tenga en cuenta que solo asignamos role0 y role1. role2 se utiliza para mostrar fallos de autorización intencionados.


      docker-compose exec keycloak kcadm.sh create roles -r tomcat-keycloak -s name=role0

      docker-compose exec keycloak kcadm.sh create roles -r tomcat-keycloak -s name=role1

      docker-compose exec keycloak kcadm.sh create roles -r tomcat-keycloak -s name=role2

      docker-compose exec keycloak kcadm.sh add-roles -r tomcat-keycloak --uusername=tester --rolename role0

      docker-compose exec keycloak kcadm.sh add-roles -r tomcat-keycloak --uusername=tester --rolename role1
  
  
  
Inicie el servidor Tomcat. Debería tardar entre 20 y 30 segundos.


    docker-compose up -d tomcat wordpress
  
  
  
    
Nota: Hay un script que corre todos estos comandos, pero me gusta detallar las cosas :)

Servermatik.es
