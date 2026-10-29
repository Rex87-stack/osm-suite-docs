# OSM Suite

## Manual de uso y características

**Versión de referencia:** 2.3.4  
**Plataforma:** WordPress + WooCommerce + WhatsApp Business Platform de Meta

OSM Suite conecta WooCommerce con WhatsApp para detectar pedidos, enviar mensajes mediante plantillas de Meta, procesar respuestas de los clientes y gestionar una cola de envíos. La versión Pro añade automatizaciones avanzadas, facturas PDF y controles adicionales.

## 1. Qué hace el plugin

- Recibe mensajes de WhatsApp mediante un webhook de Meta.
- Detecta pedidos compartidos desde WooCommerce por texto.
- Guarda la relación entre el pedido y el número que inició la conversación por WhatsApp.
- Envía mensajes mediante la cola asíncrona para evitar bloqueos durante el pedido.
- Envía una bienvenida cuando llega un mensaje que no parece un pedido.
- Comparte un pedido desde WooCommerce mediante un enlace de WhatsApp.
- Envía plantillas aprobadas por Meta con variables dinámicas.
- Reacciona a estados de WooCommerce, como pendiente, procesando, completado, cancelado, reembolsado o fallido, según la configuración disponible.
- Genera facturas PDF con datos del pedido, impuestos, descuentos, envío, cliente y logo configurado.
- Permite enviar la factura desde el pedido individual o mediante acciones en lote.
- Registra actividad técnica en logs privados fuera del área pública del sitio.
- Incluye controles de cola, reintentos, diagnósticos y protección contra mensajes duplicados.

## 2. Antes de empezar

Necesitas:

- WordPress con WooCommerce activo.
- Una aplicación de Meta con WhatsApp Business configurado.
- Un número de teléfono de WhatsApp Business conectado a Meta.
- El identificador del número de teléfono de Meta.
- El identificador de la cuenta de WhatsApp Business.
- Un token de acceso válido.
- Una plantilla creada y aprobada en Meta.
- Acceso de administrador de WordPress.

### Importante sobre los números

Hay tres números que no deben confundirse:

- **Número del negocio en Meta:** es el número desde el que se envían los mensajes.
- **Número autorizado o de prueba:** es el teléfono que Meta permite utilizar durante las pruebas.
- **Número del cliente:** es el teléfono que escribe y abre la conversación.

El plugin debe responder al número real que aparece en el webhook de Meta, no al teléfono de facturación de WooCommerce. Para facturas manuales, OSM Suite intenta utilizar primero el número de WhatsApp vinculado al pedido. El teléfono de facturación queda como dato del cliente y no sustituye automáticamente al número de la conversación.

## 3. Instalación y actualización

1. En WordPress abre **Plugins > Añadir nuevo > Subir plugin**.
2. Selecciona el archivo `osm-suite-X.Y.Z-release.zip`.
3. Si WordPress indica que ya existe una versión, revisa que el número subido sea superior y selecciona **Reemplazar el actual con el subido**.
4. Activa el plugin si WordPress lo solicita.
5. Comprueba que aparece el menú **OSM Suite**.

La actualización no debe borrar pedidos, productos ni clientes de WooCommerce. Tampoco elimina la configuración del plugin si solo se reemplazan los archivos. Antes de actualizar una instalación en producción, realiza una copia de seguridad de la base de datos y los archivos.

## 4. Configuración inicial

### 4.1 Configuración API

Entra en **OSM Suite > Configuración API** y completa:

- Token de acceso de Meta.
- ID del número de teléfono.
- ID de la cuenta WhatsApp Business, si el panel lo solicita.
- Número autorizado para la interacción o para los enlaces de WhatsApp.
- Código de verificación del webhook.
- Datos básicos del negocio.

Guarda los cambios y utiliza **Comprobar conexión**. La prueba valida las credenciales sin enviar un mensaje a los clientes.

**Seguridad:** no compartas capturas ni logs que contengan tokens. Si un token aparece expuesto, revócalo y genera uno nuevo desde Meta.

### 4.2 Webhook de Meta

En la configuración de Webhooks de la aplicación de Meta utiliza:

```text
https://TU-DOMINIO/wp-json/osm-suite/v1/webhook
```

El identificador de verificación debe coincidir exactamente con el valor guardado en OSM Suite. Después de verificar la URL, suscribe el campo **messages**.

Meta debe poder acceder a la URL por HTTPS y WordPress debe responder correctamente a la verificación GET. En producción, la aplicación de Meta debe estar publicada y el número debe pertenecer a la cuenta de negocio correspondiente.

### 4.3 Prueba recomendada

Haz la prueba en este orden:

1. Envía un mensaje desde el panel de Meta al número autorizado.
2. Comprueba que WhatsApp recibe el mensaje.
3. Responde desde el teléfono autorizado.
4. Revisa **OSM Suite > Estado del Sistema** y **Cola de Mensajes**.
5. Confirma que aparece un registro `incoming_message` y después una respuesta saliente.

Mientras la aplicación de Meta esté en desarrollo, Meta puede limitar los webhooks y destinatarios a números de prueba autorizados. Esto no significa necesariamente que el plugin esté fallando.

## 5. Plantillas de Meta

Entra en **OSM Suite > Plantillas**.

Para cada plantilla registra:

- Evento o vínculo interno.
- Nombre exacto de la plantilla en Meta.
- Número de variables.
- Idioma exacto.
- Si tiene botones interactivos.
- Mapeo de cada variable, por ejemplo `{{1}}` y `{{2}}`.

El nombre, idioma, cantidad de variables y orden de botones deben coincidir con la plantilla aprobada en Meta. Si no coinciden, Meta puede rechazar el mensaje.

### Plantillas principales

- **Bienvenida automática:** responde a un mensaje que no parece un pedido.
- **Pedido detectado por texto:** confirma un pedido detectado en el mensaje recibido.
- **Factura PDF:** activa el envío de una factura mediante un botón o una acción manual. Requiere Pro.
- **Estados de WooCommerce:** permiten notificar cambios del pedido según los eventos configurados.
- **Acciones personalizadas:** permiten crear flujos adicionales con payloads y botones. Requieren Pro.

### Botones interactivos

Cada botón debe tener:

- Índice único.
- Texto visible.
- Payload exacto.
- Tipo compatible con la plantilla aprobada en Meta.

Por ejemplo, si el botón usa el payload `factura`, debe existir una plantilla vinculada con la acción interna de factura. El texto visible puede ser `Descargar Factura` o `Solicitar factura`; lo importante para la lógica es el payload.

### Sincronización con Meta

La sincronización manual consulta el estado real de una plantilla. La sincronización automática depende de la capacidad disponible en la licencia. Una plantilla debe estar aprobada antes de que el plugin la utilice en un envío normal.

## 6. Variables disponibles

El mapeo de variables permite llenar el contenido con información del pedido.

### Datos del pedido

- Nombre del cliente.
- Número de pedido.
- Total del pedido.
- Método de envío.
- Método de pago.
- Dirección de envío.
- Teléfono de facturación.
- Correo electrónico.

### Datos del reembolso

- Monto de este reembolso.
- Total reembolsado acumulado.
- Fecha y hora del reembolso.
- Productos reembolsados.
- Detalle del reembolso.

Si se reembolsa un solo artículo de un pedido con varios artículos, el monto de este reembolso corresponde a esa operación concreta. El total reembolsado acumulado incluye los reembolsos anteriores registrados en WooCommerce.

### Datos de entrega

- Fecha de entrega.
- Hora de entrega.
- Ventana de entrega.
- Transportista.
- Número de seguimiento.
- Enlace de seguimiento.

Los campos de entrega solo se rellenan si WooCommerce o un plugin de entregas guarda esos datos en el pedido. Como cada plugin utiliza claves diferentes, OSM Suite revisa varias claves habituales y permite configurar claves personalizadas cuando corresponde. Si el proveedor no guarda el dato en el pedido, el campo no puede inventarse y quedará vacío o mostrará el valor alternativo definido.

## 7. Compartir pedidos por WhatsApp

El plugin puede mostrar un enlace para compartir el pedido desde la página de finalización de WooCommerce o mediante el shortcode disponible.

El enlace abre WhatsApp con el número de interacción configurado y un texto prellenado. Este enlace no es el mismo mecanismo que el webhook de Meta:

- El enlace abre una conversación en WhatsApp.
- El webhook recibe el mensaje cuando el cliente lo envía.
- La API de Meta responde desde el número empresarial configurado.

Para probarlo, verifica que el número del enlace sea el número correcto de WhatsApp Business o de prueba. No uses el teléfono de facturación del cliente como número de destino del enlace.

## 8. Facturas PDF

La facturación está disponible en **OSM Suite > Facturación** y requiere Pro.

### Configuración

Puedes definir:

- Nombre legal o razón social.
- Identificación fiscal.
- Dirección fiscal.
- Ciudad y país.
- Teléfono y correo.
- Moneda mostrada.
- Logo manual por URL o archivo disponible en WordPress.
- Pie de factura.

El PDF toma del pedido de WooCommerce los productos, cantidades, precios, descuentos, envío, impuestos y total. El importe de impuestos se basa en los datos guardados en el pedido, no en una tasa inventada por el plugin.

### Logo

El logo manual tiene prioridad cuando está configurado correctamente. El favicon no debe considerarse el logo comercial del documento. Si el tema no tiene logo de identidad del sitio, selecciona un logo manual o utiliza una URL pública que devuelva directamente una imagen compatible.

La URL debe:

- Empezar por HTTPS.
- Responder sin autenticación.
- Entregar una imagen real, no una página HTML.
- Ser accesible por el servidor donde corre WordPress.
- Tener una extensión o tipo compatible, como PNG, JPG o JPEG.

### Envío automático por botón

1. Configura una plantilla de Meta para factura.
2. Añade un botón con payload `factura` en la plantilla que se envía al completar el pedido.
3. Vincula ese payload con la plantilla de factura.
4. Guarda y sincroniza la configuración.
5. El cliente pulsa el botón.
6. OSM Suite identifica el pedido activo vinculado a la conversación, genera el PDF, lo sube a Meta y envía el documento.

### Envío manual desde un pedido

1. Abre **WooCommerce > Pedidos**.
2. Entra en el pedido.
3. En **Acciones del pedido**, selecciona **Enviar factura por WhatsApp**.
4. Guarda o ejecuta la acción.
5. Revisa la nota del pedido y **OSM Suite > Cola de Mensajes**.

También puede utilizarse la acción en lote **Enviar factura por WhatsApp**. Los pedidos sin número de WhatsApp vinculado se omiten para evitar enviar la factura a un teléfono incorrecto.

## 9. Cola de mensajes

OSM Suite no depende de que la petición web permanezca abierta: agrega los envíos a una cola y los procesa mediante tareas programadas.

En **OSM Suite > Cola de Mensajes** puedes revisar:

- Mensajes pendientes.
- Mensajes enviados.
- Mensajes fallidos.
- Tipo de mensaje.
- Teléfono de destino.
- Número de intentos.
- Último error.
- Fecha del próximo intento.

Los errores temporales se reintentan con una espera progresiva. Un error definitivo de Meta, como una plantilla inexistente, un destinatario no autorizado o un archivo inválido, debe corregirse antes de volver a enviar.

No vacíes toda la cola si todavía necesitas investigar un error. Primero copia el ID del mensaje y el texto del último error.

## 10. Estados y reembolsos de WooCommerce

Cuando se configura una plantilla para un estado, el cambio de estado del pedido puede crear un mensaje saliente. Los estados y acciones disponibles dependen de la licencia y de las plantillas guardadas.

Para un reembolso parcial:

1. Abre el pedido en WooCommerce.
2. Reembolsa las líneas o el importe correspondiente.
3. Guarda el reembolso en WooCommerce.
4. El evento de reembolso podrá usar `monto_reembolso`, `productos_reembolsados` y `detalle_reembolso`.

La fuente de verdad es WooCommerce. OSM Suite no calcula un reembolso independiente del pedido.

## 11. Gratis y Pro

### Incluido en Gratis

- Configuración básica de Meta.
- Comprobación de conexión.
- Webhook y recepción básica de mensajes.
- Bienvenida automática.
- Detección básica de pedidos por texto.
- Compartir pedido por WhatsApp.
- Cola básica de mensajes.
- Datos básicos del negocio.
- Diagnóstico principal.
- Sincronización manual de plantillas.

### Disponible en Pro

- Estados completos de WooCommerce.
- Reintentos y controles avanzados de cola.
- Pausar, reanudar y procesar cola.
- Acciones masivas de cola.
- Exportación CSV.
- Diagnóstico completo.
- Gestor avanzado de plantillas.
- Sincronización automática con Meta.
- Acciones personalizadas y constructor completo de mensajes.
- Facturas PDF y envío por WhatsApp.
- Envíos fuera de la ventana de 24 horas, sujeto a las reglas de Meta.
- Order via WhatsApp.
- Cuentas bancarias y enlaces de pago.

Desactivar o caducar Pro no debe borrar la configuración. Las funciones Gratis deben continuar disponibles y las funciones Pro quedan bloqueadas hasta renovar la licencia.

## 12. Estado del sistema y seguridad

En **OSM Suite > Estado del Sistema** revisa:

- Estado de WordPress y WooCommerce.
- Cron de la cola.
- Configuración de Meta.
- Ruta de logs.
- Estado de almacenamiento privado.
- Diagnósticos de conexión.

Los logs y datos internos se guardan fuera del área pública cuando el servidor lo permite. No publiques el contenido de los logs ni tokens en capturas, tickets o foros.

Buenas prácticas:

- Utiliza HTTPS.
- Usa tokens de Meta con los permisos mínimos necesarios.
- Rota los tokens si se exponen.
- Mantén WordPress, WooCommerce, PHP y el plugin actualizados.
- Haz copias de seguridad antes de actualizar.
- No actives la eliminación de datos al desinstalar salvo que quieras borrar deliberadamente la configuración, tablas, logs y datos privados del plugin.

## 13. Solución de problemas

### La conexión aparece correcta, pero no llegan mensajes

Comprueba la URL del webhook, el código de verificación, la suscripción al campo `messages`, el modo de la aplicación de Meta y el cron de WordPress. Si Meta está en desarrollo, prueba únicamente con números autorizados.

### El número dice que no existe en WhatsApp

Confirma que el número está escrito con código de país y sin espacios. Distingue el número de prueba de Meta del número del cliente. Prueba el enlace en WhatsApp Web y en la aplicación móvil, porque el comportamiento de un número de prueba puede variar entre clientes.

### El pedido se detecta, pero no se responde

Revisa la cola. Si hay un registro `incoming_message` sin respuesta, comprueba el log privado, el cron y si existe una plantilla de bienvenida o pedido aprobada. Si el envío está en `failed`, corrige el error de Meta antes de reintentar.

### Meta rechaza una plantilla

Verifica nombre exacto, idioma, número de variables, orden de variables, botones y estado aprobado en Meta. No basta con que el texto sea parecido.

### La factura no se genera

Confirma que la licencia Pro está activa, que el pedido existe, que el pedido está vinculado a un número de WhatsApp y que la acción aparece en el pedido. Revisa el ID de cola y el último error.

### La factura llega dañada o Meta rechaza el archivo

Comprueba que el archivo tenga encabezado PDF válido, que el servidor pueda leerlo y que Meta reciba el tipo MIME `application/pdf`. Genera una factura nueva desde la cola después de corregir el problema; no reutilices un archivo temporal incompleto.

### El PDF no muestra el logo

Usa un logo manual por HTTPS y verifica que la URL entregue la imagen directamente. El servidor debe poder descargarla sin cookies ni autenticación. Si el logo está en la biblioteca de medios, comprueba que la URL pública siga activa y que el archivo sea legible por el usuario del servidor web.

### La factura usa el teléfono equivocado

El teléfono de facturación puede ser diferente al teléfono de WhatsApp. Para enviar manualmente, primero debe existir una conversación o vínculo de WhatsApp asociado al pedido. Si no existe, la acción debe omitirse o corregirse el vínculo antes de enviar.

## 14. Flujo recomendado para una tienda nueva

1. Instala y activa OSM Suite.
2. Configura Meta y guarda las credenciales.
3. Comprueba la conexión.
4. Configura y verifica el webhook.
5. Suscribe `messages` en Meta.
6. Crea y aprueba las plantillas.
7. Configura el mapeo de variables.
8. Prueba con el número autorizado de Meta.
9. Confirma que la cola procesa entradas y salidas.
10. Activa la facturación y prueba un PDF si tienes Pro.
11. Revisa Estado del Sistema y conserva una copia de seguridad.
12. Antes de producción, publica la aplicación de Meta y cambia las credenciales o números de prueba por los de producción.

## 15. Información para soporte

Cuando solicites ayuda, incluye:

- Versión de OSM Suite.
- Versión de WordPress, WooCommerce y PHP.
- Tipo de mensaje afectado.
- ID de pedido.
- ID de cola.
- Fecha y hora aproximada.
- Texto exacto del error de Meta.
- Captura de la configuración sin tokens ni datos personales.

No incluyas tokens, contraseñas, firmas, nombres completos, teléfonos de clientes ni el contenido completo de los logs si no es necesario.

