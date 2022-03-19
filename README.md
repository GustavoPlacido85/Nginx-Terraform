# Nginx-Terraform
Instalar un servidor Nginx debajo de un balanceador con alta disponibilidad en Terraform 

En este repositorio se creará un servidor Nginx que será responsable de manejar todas las solicitudes entrantes a los pods de un clúster de kubernetes que crearemos usando terraform y EKS

Recursos que se crearan en la cuenta de AWS:

  EKS cluster

  VPC, Subnets, Nat gateway, etc

  IAM policy

  2 pods de kubernetes para pruebas

  1 AWS Load balancer

  Nginx


1.- Debemos clonar el repo: 


      git clone git@github.com:GustavoPlacido85/Nginx-Terraform.git
      cd Nginx-Terraform
      terraform init

2.- Validamos la salida y revisamos los recursos que se crearán, después de hacer el doublecheck aplicamos los cambios.


      terraform apply

3.- Creamos una variable de entorno para conservar los datos de kubeconfig:


      export KUBECONFIG=${PWD}/kubeconfig_my-cluster

4.- Creamos la entrada de nginx dentro del clúster:

      kubectl apply -f ingress-nginx.yaml

5.- Desplegamos dos contenedores para realizar pruebas:

      kubectl apply -f pod1.yaml

      kubectl apply -f pod2.yaml

6.- Implementar las rutas:

      kubectl apply -f routes.yaml

7.- Activamos el autoscaling:

      kubectl apply -f auto-scaler.yaml

8.- Despues de haber creado todos los recursos, hacemos una pequeña prueba:

Copiamos el nombre de DNS del balanceador https://console.aws.amazon.com/ec2/v2/home#LoadBalancers

      curl {AWS_LOAD_BALANCER_URL}/pod1

      curl {AWS_LOAD_BALANCER_URL}/pod2

9.- Ahora configuraremos un entorno ELK dentro del cluster de kubernetes, usaremos logstash que enviará los registros a kibana y s3.
necesitamos instalar elasticsearch en nuestro clúster.

      kubectl apply -f elasticsearch-ss.yaml
    
10.- Instalemos el logstash, el será responsable de enviar los registros del contenedor a elasticsearch y s3. En este archivo, tenemos el nombre de los índices de kubernetes, los depósitos y directorios en s3. Antes de ejecutar el comando, debe cambiar el nombre del depósito y el nombre de la región dentro del archivo.

       kubectl apply -f logstash-deployment.yaml
    
11.- Ahora Filebeat que es el complemento responsable de obtener los registros del contenedor. Logstash se conecta con filebeat.

       kubectl apply -f filebeat-ds.yaml
      
      
12.- Si se quisieran tener las métricas de kubernetes en kibana, se puede instalar el plugin metric beat. 

       kubectl apply -f metricbeat-ds.yaml 
       
13.- Instalación de Kibana

       kubectl apply -f kibana-deployment.yaml
       
14.- Instalamos un cron para eliminar registros antiguos en elasticsearch (El actual es de 30 días, puede editarse)    

        kubectl apply -f curator-cronjob.yaml
        
15.- Instalamos una herramienta para comprobar si todo se ha configurado correctamente.

        kubectl apply -f json-log-generator.yaml
        

*Si ejecutamos el comando "kubectl logs json-log-generator-app -n kube-system", verá que la aplicación usa el formato JSON para escribir los registros. Y tiene un campo llamado "aplicación". Este campo es muy importante. Porque logstash usa este campo para saber qué índice de búsqueda elástica se usará y también el directorio s3. Por ejemplo, si instala su aplicación dentro de este clúster configurado, usa el formato json para registrar los datos y el valor del campo de la aplicación en el json es "my-app" o "my-awesome-app", este índice será creado en elasticsearch y se creará una carpeta con ese nombre en s3.*


*Para verificar si los registros van a s3, puede verificar directamente en su depósito (el período de rotación configurado es de 10 minutos)

16.- Para validar si los registros van a Kibana:

Obtener el nombre del pod de kibana:

      kubectl get pods -n kube-system
 
17.- Abrir un túnel para acceder a Kibana en el puerto 5601:

      kubectl port-forward -n kube-system {KIBANA_LOGGING_POD_NAME_HERE} 5601:5601
      
      
18.- Acceder a la URL en la máquina "localhost:5601" Configuramos el nombre del índice en kibana como ej. "my-app*" Y podremos ver los registros allí.      

19.- Si la creación de estos recursos son solo para prubas no olvidemos destruirlos para no generar gastos:

      terraform destroy

       
       
