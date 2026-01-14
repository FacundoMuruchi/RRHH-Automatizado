# Automatización de gestor de RRHH

Este documento describe **cómo ejecutar pruebas reproducibles** del workflow de automatización de gestión de candidatos desarrollado con **n8n**.

El objetivo es permitir que cualquier persona (reclutador técnico, docente o reviewer) pueda **levantar el entorno, ejecutar el flujo y validar su funcionamiento end-to-end**.

---

## 1. Requisitos previos

Antes de comenzar, asegurate de contar con lo siguiente:

* **Docker** y **Docker Compose** instalados
* **Cuenta de n8n** (local, self-hosted)
* **Cuenta de Google** con acceso a:

  * Google Sheets
  * Gmail API habilitada
* **Cuenta de AWS** con:

  * Bucket S3 creado
  * Acceso público de lectura a los assets de imágenes
* **Cuenta de Google Gemini** (API Key)

---

## 2. Levantar n8n en local

Ejecutar n8n usando Docker:

```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Acceder desde el navegador:

```
http://localhost:5678
```

### 2.1 Exponer n8n con ngrok

Ejecutar ngrok apuntando al puerto de n8n:
```bash
ngrok http 5678
```
ngrok generará una URL pública similar a:
```bash
https://abcd-123-45-67.ngrok-free.app
```
Esta URL debe configurarse en un archivo .yaml para comunicar Typeform con n8n local expuesto con ngrok.

```yaml
version: "3.8"

services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    ports:
      - "5678:5678"
    environment:
      - WEBHOOK_URL=URL_NGROK
    volumes:
      - NOMBRE_VOLUMEN:/home/node/.n8n

volumes:
  n8n_data:
    external: true
```

---

## 3. Importar el workflow

1. Abrir n8n
2. Ir a **Import workflow**
3. Importar el archivo:

```
RRHH_automatizado.json
```

4. Verificar que todos los nodos estén correctamente conectados

---

## 4. Configuración de credenciales

### 4.1 Google Sheets

* Crear una credencial de **Google Sheets OAuth2**
* Compartir la planilla con el mail del service account

La planilla debe contener al menos dos hojas:

* `Aprobados`
* `Rechazados`

---

### 4.2 Gmail

* Configurar **Gmail OAuth2**
* Autorizar el envío de correos

---

### 4.3 AWS S3

* Configurar credenciales AWS en n8n
* El bucket debe contener imágenes como:

```
autoinc_logo_recibido.png
autoinc_logo_felicidades.png
autoinc_logo_rechazo.png
```

Los objetos deben ser accesibles públicamente por URL.

---

### 4.4 Google Gemini

* Crear credencial con **API Key**
* Limitar el modelo a respuestas determinísticas

---

## 5. Datos de prueba

Para ejecutar pruebas reproducibles, usar siempre valores controlados.

### Ejemplo de candidato (aprobado)

```
Nombre: Juan
Apellido: Pérez
Edad: 25
Email: juan.perez@test.com
Experiencia: Sí
Motivación: "Tengo experiencia en automatización y muchas ganas de crecer"
```

### Ejemplo de candidato (rechazado)

```
Nombre: Ana
Apellido: Gómez
Edad: 19
Email: ana.gomez@test.com
Experiencia: No
Motivación: "Quiero plata"
```

---

## 6. Ejecución de la prueba

1. Disparar el workflow desde el **Typeform Trigger** (modo test)

2. Verificar:

   * Score de motivación generado
   * Selección correcta de imagen desde S3
   * Envío de email de recepción

3. En caso de candidato válido:

   * Se envía mail de aprobación a RRHH
   * RRHH responde **Approve / Reject**

4. Verificar resultado:

   * Registro agregado en Google Sheets
   * Email final enviado al candidato

---

## 7. Validaciones esperadas

| Escenario          | Resultado esperado      |
| ------------------ | ----------------------- |
| Edad < 18          | Revisión                |
| Sin experiencia    | Revisión                |
| Motivación < 80    | Revisión                |
| Aprobado por RRHH  | Email + hoja Aprobados  |
| Rechazado por RRHH | Email + hoja Rechazados |

---

## 8. Notas finales

Este workflow está diseñado como **prototipo funcional realista**, aplicando buenas prácticas de:

* Automatización
* Integración con IA
* Human-in-the-loop
* Separación de responsabilidades

---

## 9. Explicación a detalle del workflow (Inglés)

https://www.notion.so/HR-Applicant-Management-Automation-with-n8n-2e78d5dc55c2801dbda7eba57348a2a4?source=copy_link