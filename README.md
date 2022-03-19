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

9.- Si desea destruir los recursos creados:

      terraform destroy
