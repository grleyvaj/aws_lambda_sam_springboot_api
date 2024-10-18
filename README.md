# Proyecto Spring Boot API Rest en AWS (Lambda + Api Gateway)

## DESCRIPCIÓN DEL PROYECTO
En este proyecto se desarrolla una API Rest / Microservicio de Spring Boot y se despliega en infraestructura de la nube de AWS como una lambda expuesta con api gateway, empleando SAM

## ESTRUCTURA
- Crear API Rest con  Spring Boot
- Crear StreamLambdaHandler
  1. Esta clase sirve como punto de entrada para que AWS Lambda enrute las solicitudes a una aplicación Spring Boot utilizando la integración de AWS Lambda Proxy.
  2. Inicializa un contexto de aplicación Spring Boot dentro de un entorno Lambda, permitiendo que la aplicación Spring Boot gestione solicitudes HTTP a través de eventos Lambda (como AwsProxyRequest y AwsProxyResponse).
- Crear `template.yaml` de SAM para crear infra en AWS
  ```bash
  sam build
  ```

## REQUISITOS

### Instalación
1. **AWS CLI**  
   [Guía de instalación de AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
    - Configurar credenciales de AWS con el comando:
      ```bash
      aws configure
      ```
    - Probar con:
      ```bash
      aws --version
      ```
2. **SAM CLI**  
   [Guía de instalación de Terraform](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
    - Probar con:
      ```bash
      sam --version
      ```

### Arranque
Desplegar la api como lambda usando sam. 

- **Crear el jar que se hace upload al bucket de S3**
    ```bash
    mvn clean package
    ```

- **Comandos SAM para validar la template**
    ```bash
    sam build
    sam validate
    ```

- **Crear el bucket donde SAM subira el JAR**
    ```bash
    aws s3 mb s3://my-springboot-aws-sam-bucket
    ```

- **Empaquetar la aplicación y subirá los archivos al bucket antes creado**
    ```bash
    sam package --output-template-file packaged.yaml --s3-bucket <nombre-del-bucket>
    ```
  Ejemplo
    ```bash
    sam package --template-file template.yml --output-template-file packaged.yml --s3-bucket my-springboot-aws-sam-bucket
    ```

- **Desplegar la aplicación**
    ```bash
    sam deploy --template-file packaged.yml --stack-name <nombre-del-stack> --capabilities CAPABILITY_IAM --region <region>
    ```
  Ejemplo
    ```bash
    sam deploy --template-file packaged.yml --stack-name FinancialInstitutionApi --capabilities CAPABILITY_IAM --region us-east-2
  ```

- **Obtener la URL de la API**
    ```bash
    aws cloudformation describe-stacks --stack-name SpringBootLambdaApi --query "Stacks[0].Outputs[?OutputKey=='SpringBootLambdaApi'].OutputValue" --output text
    ```
 
    Ejemplo
    ```bash
    aws cloudformation describe-stacks --stack-name FinancialInstitutionApi --query "Stacks[0].Outputs[?OutputKey=='FinancialInstitutionApi'].OutputValue" --output text
    ```
  
### POLITIAS IAM
Adicionalmente, para poder ejecutar los comandos anteriores, para que el usuario tenga los permisos debe asegurarse que tenga asignado las siguientes políticas
![iam-user.png](imgs%2Fiam-user.png)

Cree la politica `MyIAMPolicy` con los siguientes:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"iam:CreateRole",
				"iam:AttachRolePolicy",
				"iam:PassRole",
				"tag:GetResources",
				"tag:TagResources",
				"tag:UntagResources",
				"iam:TagRole",
				"iam:PutRolePolicy",
				"iam:DeleteRole",
				"iam:DetachRolePolicy",
				"iam:CreateInstanceProfile",
				"iam:GetInstanceProfile",
				"iam:DeleteInstanceProfile",
				"iam:AddRoleToInstanceProfile",
				"iam:CreatePolicy",
				"iam:AttachRolePolicy",
				"iam:PassRole",
				"iam:TagPolicy"
			],
			"Resource": "*"
		}
	]
}
```

- Se creo un archivo `event.json` que contiene la estructura de un evento para probar la invocacion a la lambda
```bash
sam local generate-event apigateway aws-proxy > event.json
```

- Probar invocandolo desde la terminal con el comando
```bash
aws lambda invoke --function-name financial-institution-lambda --payload event.json response.json
```

### RECURSOS CREADOS EN AWS
- LAMBDA
![lambda.png](imgs%2Flambda.png)
- API GATEWAY
![api-gateway.png](imgs%2Fapi-gateway.png)