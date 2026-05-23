# Taller AWS — Puntos 1 y 2

Repositorio para el taller de AWS de la materia Sistemas Operativos. Cubre la gestión de archivos en Amazon S3 y el despliegue de una aplicación FastAPI en Amazon EC2.

**Autor:** Samuel Giraldo Jimenez  
**Materia:** Sistemas Operativos

---

## 1. Gestión de archivos en Amazon S3

### a. Creación del bucket

Se creó un bucket en Amazon S3 con el nombre `user-1038868860-ueia-so` siguiendo el patrón solicitado.

### b. Operaciones usando Bash / AWS CLI

**Cargar un archivo:**

```bash
aws s3 cp archivo.txt s3://user-1038868860-ueia-so/
```

**Verificar la carga:**

```bash
aws s3 ls s3://user-1038868860-ueia-so/
```

**Descargar en otra carpeta:**

```bash
mkdir descarga
aws s3 cp s3://user-1038868860-ueia-so/archivo.txt descarga/
```

**Múltiples archivos:**

Cuando se trabaja con un solo archivo, `aws s3 cp` funciona indicando el archivo y el destino directamente. Para múltiples archivos se usa la bandera `--recursive`, que permite copiar todos los archivos de una carpeta de una sola vez. Sin esta bandera, habría que ejecutar el comando una vez por cada archivo.

```bash
# Subir carpeta completa
aws s3 cp archivos/ s3://user-1038868860-ueia-so/archivos/ --recursive

# Descargar carpeta completa
aws s3 cp s3://user-1038868860-ueia-so/archivos/ descarga_multi/ --recursive
```

### c. Operaciones usando boto3 (Python)

Script `s3_boto3.py` que realiza las operaciones con boto3:

```python
import boto3
import os

s3 = boto3.client('s3')
bucket = 'user-1038868860-ueia-so'

# Subir un archivo
s3.upload_file('archivo_boto3.txt', bucket, 'archivo_boto3.txt')

# Verificar la carga
response = s3.list_objects_v2(Bucket=bucket)
for obj in response['Contents']:
    print(obj['Key'])

# Descargar en otra carpeta
os.makedirs('descarga_boto3', exist_ok=True)
s3.download_file(bucket, 'archivo_boto3.txt', 'descarga_boto3/archivo_boto3.txt')

# Múltiples archivos (3 archivos de texto)
for i in range(1, 4):
    with open(f'file{i}.txt', 'w') as f:
        f.write(f'Contenido del archivo {i}')
    s3.upload_file(f'file{i}.txt', bucket, f'multiples/file{i}.txt')

for i in range(1, 4):
    s3.download_file(bucket, f'multiples/file{i}.txt', f'descarga_boto3/file{i}.txt')
```

**Diferencia con CLI:** Con boto3 no existe una función directa para subir o descargar múltiples archivos de una vez como `--recursive`. Se debe recorrer los archivos con un ciclo `for` y llamar a `upload_file` o `download_file` por cada uno. También se puede usar `list_objects_v2` para listar los archivos del bucket y descargarlos uno por uno.

---

## 2. Despliegue de aplicación FastAPI en Amazon EC2

### Aplicación

API básica en FastAPI con tres endpoints:

| Método | Ruta              | Descripción                    |
|--------|-------------------|--------------------------------|
| GET    | `/`               | Retorna mensaje de bienvenida  |
| GET    | `/items/{item_id}`| Consulta un item por ID        |
| POST   | `/items/`         | Crea un item                   |

### Pasos de despliegue

**1. Crear repositorio en GitHub**

```bash
git clone https://github.com/SamuelGhoop/test_docker_fastapi.git
```

**2. Crear instancia EC2**

- AMI: Ubuntu Server
- Tipo: t2.micro (Free Tier)
- Key pair: configurado para acceso SSH
- Security Group: puertos 22 (SSH) y 8000 (app) abiertos

**3. Conectarse a la instancia**

```bash
ssh -i tu-key.pem ubuntu@<IP-PUBLICA>
```

**4. Instalar dependencias dentro de EC2**

```bash
sudo apt update
sudo apt install -y python3-pip git docker.io
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

**5. Clonar el repo y construir la imagen Docker**

```bash
git clone https://github.com/SamuelGhoop/test_docker_fastapi.git
cd test_docker_fastapi
docker build -t fastapi-test .
docker run -d -p 8000:8000 fastapi-test
```

**6. Crear servicio systemd**

```bash
sudo nano /etc/systemd/system/fastapi_ec2.service
```

Contenido del archivo:

```ini
[Unit]
Description=FastAPI Docker Container
After=docker.service

[Service]
ExecStart=/usr/bin/docker run -p 8000:8000 fastapi-test
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Activar el servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable fastapi_ec2
sudo systemctl start fastapi_ec2
sudo systemctl status fastapi_ec2
```

**7. Configurar Security Group**

En la consola de AWS → EC2 → Security Groups → Edit inbound rules:

| Tipo        | Puerto | Source    |
|-------------|--------|-----------|
| SSH         | 22     | 0.0.0.0/0 |
| Custom TCP  | 8000   | 0.0.0.0/0 |

**8. Probar desde el navegador**

```
http://<IP-PUBLICA>:8000/docs
```

---

## Estructura del repositorio

```
test_docker_fastapi/
├── main.py              # Aplicación FastAPI
├── Dockerfile           # Imagen Docker
├── requirements.txt     # Dependencias
└── README.md            # Este archivo
```

---

## Requisitos

- Python 3.10+
- Docker
- AWS CLI configurado
- Cuenta AWS con acceso a S3 y EC2
