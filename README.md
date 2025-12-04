# 📄 Automatización de RRHH

### **Caso elegido**

Automatización del proceso de recepción, evaluación y respuesta a
postulaciones laborales.

------------------------------------------------------------------------

### **Workflow (trigger)**

El workflow se activa mediante el nodo **Webhook -- "obtener datos de
typeform"**, que recibe las respuestas enviadas desde un formulario
Typeform.

------------------------------------------------------------------------

### **Descripción de cada nodo**

-   **obtener datos de typeform (Webhook):** recibe los datos del
    formulario (nombre, apellido, edad, email, experiencia). Es el punto
    de inicio del flujo.

-   **verificar edad y experiencia (IF):** evalúa si el postulante marcó
    que tiene experiencia (boolean) y si su edad es mayor a 18.

-   **Enviar mensaje de rechazo:** si el IF falla, se envía un email
    automático informando que no fue seleccionado.

-   **enviar recibido a aplicante:** si el IF pasa, se envía un correo
    confirmando que su aplicación fue recibida correctamente.

-   **obtener fecha y hora (HTTP Request):** consulta la WorldTimeAPI
    para obtener la fecha y hora actual de Buenos Aires. Incluye
    reintentos en caso de fallo.

-   **formatear fecha (Date & Time):** convierte la fecha recibida a
    formato `dd/MM/yyyy` para usarla en la notificación interna.

-   **Merge:** combina los datos del formulario con la fecha formateada
    para generar un único objeto.

-   **enviar info a RRHH:** envía un correo interno con los datos del
    postulante (nombre, edad, email, experiencia y fecha de envío).

------------------------------------------------------------------------

### **Qué evalúa el if**

El nodo **IF** verifica: 1. **Experiencia = true** 2. **Edad \> 18**

Esto asegura que solo candidatos que cumplen requisitos mínimos reciban
confirmación y sean reportados a RRHH, mientras que quienes no califican
reciben una notificación de rechazo.

------------------------------------------------------------------------

### **Notificación**

-   Los correos se envían mediante nodos **Gmail**.
-   El email al postulante cambia según el resultado del IF
    (confirmación o rechazo).
-   La notificación interna combina datos del formulario con la fecha
    formateada.