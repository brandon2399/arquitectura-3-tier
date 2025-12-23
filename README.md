# DOCUMENTO DE DISEÑO DE ARQUITECTURA 3-TIER

**Fecha:** 23 de diciembre de 2025  
**Autor:** Brandon Concha  
**Tipo:** Prueba Técnica  
**Repositorio GitHub:**  
👉 https://github.com/brandon2399/arquitectura-3-tier  

---

## Parte 1: Preguntas Teóricas

### 1. Diferencia entre nube pública, privada e híbrida

**Nube Pública**  
Los servicios de computación y almacenamiento son propiedad de un proveedor externo (AWS, Azure, GCP) y se comparten con otras organizaciones a través de internet.  
Es altamente escalable y se paga únicamente por el uso.

**Nube Privada**  
Recursos informáticos utilizados exclusivamente por una sola organización.  
Puede estar ubicada en el centro de datos de la empresa o ser gestionada por un tercero.  
Ofrece mayor control y seguridad personalizada.

**Nube Híbrida**  
Combinación de nube pública y privada que permite compartir datos y aplicaciones entre ambas.  
Es ideal para organizaciones que desean mantener datos sensibles en infraestructura privada y aprovechar la escalabilidad de la nube pública para cargas variables.

---

### 2. Tres prácticas de seguridad en la nube

**Principio de Menor Privilegio (Least Privilege)**  
Configurar políticas de IAM para que usuarios y servicios tengan únicamente los permisos estrictamente necesarios.

**Cifrado de Datos**  
Implementar cifrado:
- En reposo (discos, bases de datos)
- En tránsito (certificados SSL/TLS para HTTPS)

**Seguridad Perimetral y de Red**  
Uso de firewalls gestionados (Security Groups, WAF) y segmentación de red mediante subredes públicas y privadas para aislar componentes críticos.

---

### 3. ¿Qué es IaC y cuáles son sus beneficios?

La **Infraestructura como Código (IaC)** es la gestión y aprovisionamiento de infraestructura mediante archivos de configuración legibles por máquina, en lugar de procesos manuales.

**Beneficios:**
- Repetibilidad (evita errores humanos)
- Velocidad de despliegue
- Consistencia entre entornos (Dev, Test, Prod)
- Control de versiones (Git)

**Herramientas:**

- **Terraform**
  - Agnóstico al proveedor
  - Usa lenguaje HCL
  - Mantiene un archivo de estado de la infraestructura

- **AWS CloudFormation**
  - Nativo de AWS
  - Usa JSON o YAML
  - Gestión integrada de recursos AWS

---

### 4. Métricas esenciales para el monitoreo

**Rendimiento de Recursos**
- Uso de CPU
- Memoria RAM
- Latencia de red

**Disponibilidad**
- Uptime
- Tasas de error (ej. errores HTTP 5xx)

**Almacenamiento**
- Espacio en disco disponible
- IOPS

**Costos**
- Monitoreo de presupuesto diario para evitar picos inesperados

---

### 5. ¿Qué es Docker y sus componentes?

Docker es una plataforma de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores, permitiendo que se ejecuten de forma consistente en cualquier entorno.

**Componentes principales:**

- **Dockerfile**  
  Archivo de texto con instrucciones para construir una imagen.

- **Imagen**  
  Plantilla de solo lectura que contiene el código, librerías y dependencias.

- **Contenedor**  
  Instancia ejecutable y ligera de una imagen.

- **Docker Hub / Registry**  
  Repositorio donde se almacenan y comparten imágenes.

---

## Parte 2: Caso Práctico  
## Diseño de Arquitectura 3-TIER – Justificación Técnica

### 1. Resumen

Este documento describe la arquitectura técnica para una aplicación web moderna desplegada en AWS.  
El diseño prioriza:
- Seguridad por aislamiento
- Alta disponibilidad
- Bajo costo operativo  
mediante el uso de servicios gestionados.

---

### 2. Diagrama Conceptual



### 3. Decisiones de Diseño y Justificación

#### Cómputo: AWS Fargate (Contenedores Serverless)

**Decisión**  
Utilizar Amazon ECS con capacidad Fargate en lugar de instancias EC2.

**Justificación**
- **Menor carga operativa:** No hay servidores que parchar ni sistemas operativos que asegurar.
- **Escalabilidad:** Escalado horizontal nativo según la demanda.
- **Eficiencia de costos:** Pago únicamente por CPU y RAM consumidos mientras el contenedor está activo.

---

#### Red: VPC con Aislamiento de 3 Capas

**Decisión**  
Segmentación de recursos en:
- Subredes públicas
- Subredes privadas de aplicación
- Subredes privadas de datos  
en dos Zonas de Disponibilidad.

**Justificación**
- **Seguridad (Defensa en Profundidad):** Backend y base de datos protegidos por múltiples capas.
- **Alta disponibilidad:** Diseño Multi-AZ que garantiza continuidad ante la falla de una zona.

---

#### Almacenamiento: Amazon S3 y Amazon RDS

**Decisión**  
Separar el almacenamiento de objetos (S3) de la base de datos relacional (RDS).

**Justificación**
- **Persistencia:** Los contenedores son efímeros; los datos deben residir en servicios con alta durabilidad (99.99%).
- **Rendimiento:** RDS ofrece backups automáticos y optimizaciones que serían costosas de mantener manualmente.

---

#### Seguridad: Principio de Menor Privilegio

**Decisión**  
Implementar Security Groups específicos y Roles de IAM.

**Justificación**
- **Control granular:** Cada componente solo se comunica con los recursos estrictamente necesarios.  
  Ejemplo: la base de datos solo acepta conexiones desde el backend.

---

### 4. Flujo de Operación

- **Entrada:** El tráfico es recibido por el Application Load Balancer (ALB).
- **Procesamiento:** Amazon ECS Fargate ejecuta la lógica de negocio en subredes privadas.
- **Almacenamiento:**  
  - Activos estáticos en Amazon S3  
  - Datos transaccionales en Amazon RDS
- **Salida:** Logs y métricas enviados a Amazon CloudWatch para monitoreo en tiempo real.

---

## Conclusión

Esta arquitectura no solo cumple con los requisitos del demo, sino que establece una base sólida para una aplicación de nivel productivo.  
Es:
- Segura
- Fácil de auditar
- Altamente resiliente

Es importante destacar que no existe una única solución perfecta.  
Siempre hay oportunidades de mejora y optimización, pero esta es la arquitectura que yo implementaría dadas las condiciones del ejercicio.

---

## Reflexión Personal
Todas las decisiones de arquitectura son una negociación entre  **beneficios** y **costos**.
cada decisión trae riesgos, los riesgos se mitigan, se trasladan o se asumen, en el mundo ideal basado en la probabilidad de ocurrencia, en el mundo real basado tal vez en cuánto me cuesta los trade-offs que llaman.

Cada decisión introduce riesgos que pueden:
- Mitigarse
- Trasladarse
- Asumirse

---

🔗 Repositorio del proyecto:  
https://github.com/brandon2399/arquitectura-3-tier


