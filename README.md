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


Debemos clonar el repo: 
