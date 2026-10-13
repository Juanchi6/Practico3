# Práctico 3 — Desarrollo Seguro de Software

Pruebas realizadas sobre OWASP Juice Shop y OWASP crAPI en un laboratorio local de Kali Linux, utilizando Firefox Network y `Edit and Resend`. Las capturas corresponden a Juice Shop en `http://localhost:3000` y crAPI en `http://localhost:8888`.

Se seleccionaron cinco desafíos de cada aplicación: J1, J2, J4, J6 y J7 de Juice Shop; C1, C3, C4, C5 y C6 de crAPI. Los identificadores J/C corresponden a la consigna. Cada caso incluye el defecto, su clasificación con la edición de OWASP utilizada, los pasos, la evidencia disponible y una mitigación.

Los identificadores de cuentas, carritos, vehículos y pedidos corresponden al estado del laboratorio y pueden variar en otra instalación.

## OWASP Juice Shop

### J1 — Zero Stars

**Vulnerabilidad:** validación insuficiente del valor de la calificación. El formulario limita los valores disponibles, pero el servidor aceptó una calificación de cero.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design: la regla que limita la calificación a 1–5 se aplica en el cliente, pero no se hace cumplir en el servidor.

**Prueba de concepto:**

1. Abrí `Customer Feedback` y envié un comentario con una estrella.
2. En las herramientas de Firefox (`Network`), identifiqué `POST /api/Feedbacks/`.
3. Utilicé `Edit and Resend` y cambié `rating: 1` por `rating: 0`, conservando los otros campos.
4. El servidor respondió **HTTP 201 Created**, aceptando la valoración de cero estrellas.

**Evidencia:**

![Solicitud con rating 0 y respuesta HTTP 201](evidencias/juice-shop/j1-zero-stars.png)

**Mitigación:** validar también del lado del servidor que `rating` sea un entero dentro del rango permitido (1–5) y rechazar los demás valores con un error 400.

### J2 — Login Admin

**Vulnerabilidad:** SQL Injection en el inicio de sesión: el correo introducido modifica la consulta SQL y permite omitir la comprobación de contraseña.

**Clasificación:** OWASP Top 10 2021 — A03: Injection.

**Prueba de concepto:**

1. Abrí el formulario `Login`.
2. Ingresé `admin@juice-sh.op'--` como correo y `prueba123` como contraseña.
3. El inicio de sesión devolvió **HTTP 200** y Juice Shop confirmó el desafío *Login Admin*.

**Evidencias:** solicitud de inicio de sesión con la inyección y respuesta HTTP 200; confirmación visual de Juice Shop de que se resolvió *Login Admin*.

![Solicitud SQL Injection y HTTP 200](evidencias/juice-shop/j2-login-admin-request.png)

![Confirmación del desafío Login Admin resuelto](evidencias/juice-shop/j2-login-admin-resultado.png)

**Mitigación:** utilizar consultas SQL parametrizadas y validar los datos de entrada; nunca concatenar valores del usuario directamente en una consulta de autenticación.

### J4 — Admin Registration

**Vulnerabilidad:** asignación indebida de privilegios mediante manipulación del registro de usuarios (Mass Assignment).

**Clasificación:** OWASP Top 10 2021 — A01: Broken Access Control.

**Prueba de concepto:**

1. Registré una cuenta normal y localicé la solicitud `POST /api/Users/` en Firefox Network.
2. Con `Edit and Resend`, usé otro correo y añadí `"role":"admin"` al JSON.
3. El servidor respondió **HTTP 201 Created** y Juice Shop confirmó *Admin Registration*.

**Evidencia:**

![Registro con rol admin, HTTP 201 y desafío completado](evidencias/juice-shop/j4-admin-registration.png)

**Mitigación:** asignar roles exclusivamente del lado del servidor, usando una lista permitida de campos de registro y controles de autorización.

### J6 — View Basket

**Vulnerabilidad:** acceso directo a objetos sin verificar autorización (IDOR), permitiendo consultar el carrito de otra cuenta.

**Clasificación:** OWASP Top 10 2021 — A01: Broken Access Control.

**Prueba de concepto:**

1. Con una cuenta normal, abrí `Your Basket` y localicé `GET /rest/basket/6` en Firefox Network.
2. Mediante `Edit and Resend`, cambié el identificador a `GET /rest/basket/1` sin modificar la sesión.
3. El servidor respondió **HTTP 200 OK** y Juice Shop confirmó el desafío *View Basket*.

**Evidencia:**

![Solicitud a carrito ajeno, HTTP 200 y desafío completado](evidencias/juice-shop/j6-view-basket.png)

**Mitigación:** validar en el servidor que el usuario autenticado sea propietario del carrito solicitado, rechazando el acceso a carritos ajenos.

### J7 — Christmas Special

**Vulnerabilidad:** la búsqueda permite una SQL Injection que revela productos retirados, y el servidor no comprueba que el producto siga disponible al agregarlo al carrito y comprarlo.

**Clasificación:** OWASP Top 10 2021 — A03: Injection (búsqueda) y A04: Insecure Design (validación de disponibilidad del producto).

**Prueba de concepto:**

1. Consulté `/rest/products/search?q='))--` y encontré *Christmas Super-Surprise-Box (2014 Edition)*, producto eliminado con `id: 10` y `deletedAt` informado.
2. Capturé `POST /api/BasketItems/` en Firefox Network y reenvié la solicitud con `ProductId: 10`, conservando el carrito de mi cuenta.
3. El producto apareció en el carrito y completé el checkout; Juice Shop confirmó el desafío *Christmas Special*.

**Evidencias:**

![Producto eliminado con ID 10 y fecha de eliminación](evidencias/juice-shop/j7-christmas-producto.png)

![Confirmación de la compra y desafío Christmas Special resuelto](evidencias/juice-shop/j7-christmas-resultado.png)

**Mitigación:** parametrizar las consultas SQL y verificar en el servidor que el producto esté activo tanto al añadirlo al carrito como al confirmar el pedido.

## OWASP crAPI

### C1 — Find Another User's Vehicle Location

**Vulnerabilidad:** BOLA: falta de autorización por objeto para consultar la ubicación de vehículos ajenos.

**Clasificación:** OWASP API Security Top 10 2023 — API1: Broken Object Level Authorization.

**Prueba de concepto:**

1. Con mi cuenta, identifiqué la solicitud `GET /identity/api/v2/vehicle/{id}/location` de mi vehículo.
2. En `Community`, encontré el identificador `author.vehicleId` de un vehículo de otro usuario.
3. Mediante `Edit and Resend`, sustituí el ID de la URL y conservé la autorización de mi sesión. La API respondió **HTTP 200** y devolvió las coordenadas y datos del otro propietario.

**Evidencia:** con la sesión de `Usuario Prueba DSS`, la respuesta muestra la ubicación y los datos del vehículo de `Robot`.

![Respuesta con ubicación y datos de Robot mientras la interfaz muestra Usuario Prueba DSS](evidencias/crapi/c1-vehicle-location.png)

**Mitigación:** comprobar en cada petición que el vehículo consultado pertenezca al usuario autenticado o que éste tenga autorización expresa.

### C3 — Get an Item for Free

**Vulnerabilidad:** validación insuficiente de la cantidad comprada. La API acepta valores negativos que invierten el importe de la operación.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design: falta la regla de negocio que exige cantidades positivas antes de calcular el importe de una compra.

**Relación con la consigna:** el objetivo de C3 es obtener un artículo gratis. Aunque el enunciado menciona una devolución, la prueba usa una cantidad negativa en una compra: se registra un artículo y el importe resulta negativo en lugar de cobrarse. Esta variante aparece en la [solución oficial del desafío 8 de crAPI](https://github.com/OWASP/crAPI/blob/develop/docs/challengeSolutions.md#challenge-8---get-an-item-for-free).

**Prueba de concepto:**

1. Compré un Seat de $10 y capturé `POST /workshop/api/shop/orders` en Firefox Network.
2. Reenvié la solicitud con `{"product_id":1,"quantity":-1}`; el servidor respondió **HTTP 200**.
3. En `Past Orders` apareció un pedido de Seat con importe **−$10**. La evidencia prueba que la aplicación aceptó un pedido con importe negativo.

**Evidencias:**

![Solicitud con cantidad negativa](evidencias/crapi/c3-quantity-negativa.png)

![Pedido registrado con importe negativo](evidencias/crapi/c3-pedido-negativo.png)

**Mitigación:** validar en el servidor que `quantity` sea un entero positivo y calcular importes exclusivamente con valores permitidos.

### C4 — Increase Balance by $1,000 or More

**Vulnerabilidad:** el defecto de validación de C3 permite introducir cantidades negativas de mayor magnitud. Al calcular el importe y descontarlo del saldo sin comprobar su signo, el movimiento esperado se convierte en un crédito.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design: el cálculo del movimiento de saldo carece de una validación de las reglas de compra.

**Relación con la consigna:** el objetivo de C4 es aumentar el saldo al menos $1.000. Se aprovechó la misma falla de C3, usando una cantidad negativa mayor, como propone la [solución oficial del desafío 9 de crAPI](https://github.com/OWASP/crAPI/blob/develop/docs/challengeSolutions.md#challenge-9---increase-your-balance-by-1000-or-more). No se utilizó el flujo de devolución mencionado en el enunciado.

**Prueba de concepto:**

1. Capturé una compra normal de Seat (`product_id: 1`, precio $10).
2. Reenvié `POST /workshop/api/shop/orders` con `{"product_id":1,"quantity":-101}`.
3. Tras la prueba, la aplicación mostró un saldo de **$1.090**. Una cantidad de `-101` a $10 genera un importe de **−$1.010**, que se suma al saldo al descontarlo.

**Evidencia y alcance:** la captura muestra `quantity: -101` en el editor y un saldo final de **$1.090**. El cálculo explica el crédito de $1.010, pero la imagen no muestra el saldo previo ni la respuesta de la solicitud. Por eso, el incremento exacto se explica con el procedimiento y el cálculo, no con una comparación visual de dos saldos.

![Saldo de $1090 y cuerpo con cantidad -101 en el editor de solicitudes](evidencias/crapi/c4-aumento-saldo.png)

**Mitigación:** rechazar cantidades no positivas y comprobar en el servidor los importes y movimientos de saldo antes de procesar cualquier compra.

### C5 — Server-Side Request Forgery (SSRF)

**Vulnerabilidad:** la función `Contact Mechanic` permite indicar desde el cliente la URL de destino de una solicitud realizada por el servidor.

**Clasificación:** OWASP API Security Top 10 2023 — API7: Server Side Request Forgery.

**Prueba de concepto:**

1. En `Dashboard → Contact Mechanic`, envié un reporte de prueba y localicé la solicitud `POST /workshop/api/merchant/contact_mechanic` en Firefox Network.
2. Con `Edit and Resend`, modifiqué únicamente `mechanic_api` para que apuntara a `https://www.google.com`, conservando los demás parámetros y la sesión.
3. La respuesta incluyó `status: 200` y contenido HTML de Google en `response_from_mechanic_api`. El HTML devuelto —no la etiqueta `<!doctype html>` por sí sola— indica que crAPI realizó una solicitud al destino indicado por el cliente.

**Evidencia y alcance:** se ve la respuesta con HTML de Google y estado 200. La captura no muestra el valor enviado en `mechanic_api`, por lo que ese cambio queda descrito en los pasos reproducibles.

![Respuesta de contact_mechanic con status 200 y HTML de Google](evidencias/crapi/c5-ssrf-google.png)

**Mitigación:** limitar los destinos a servicios de mecánicos autorizados mediante una lista permitida; validar esquema, host, resolución DNS y redirecciones, y bloquear acceso a redes internas y servicios de metadatos.

### C6 — NoSQL Injection: Find Coupons Without Knowing the Code

**Vulnerabilidad:** inyección NoSQL en la validación de cupones. La API acepta un objeto con un operador de consulta en el campo `coupon_code` en lugar de exigir un código de texto.

**Clasificación:** OWASP API Security Top 10 2019 — API8: Injection. Se utiliza esta edición porque incluye explícitamente la inyección NoSQL: datos controlados por el cliente se interpretan como parte de una consulta.

**Prueba de concepto:**

1. Desde `Shop → Add Coupons`, envié el código inválido `PRUEBA123` e identifiqué la solicitud `POST /community/api/v2/coupon/validate-coupon` en Firefox Network.
2. Con `Edit and Resend`, sustituí el cuerpo por `{"coupon_code":{"$ne":null}}`, manteniendo la autenticación de mi sesión.
3. El servidor respondió **HTTP 200** y devolvió un cupón previamente desconocido: `TRAC075`, con `amount: "75"`.

**Evidencia y alcance:** la respuesta a `{"coupon_code":{"$ne":null}}` devuelve `TRAC075` sin que se hubiera enviado ese código. Esto demuestra la obtención del cupón. Su aplicación posterior se comprobó durante la práctica, pero no quedó fotografiada en la evidencia entregada.

![Operador NoSQL, HTTP 200 y cupón obtenido](evidencias/crapi/c6-nosql-cupon.png)

**Mitigación:** verificar del lado del servidor que `coupon_code` sea una cadena de texto con el formato permitido; rechazar objetos y operadores de consulta, y construir las consultas con filtros controlados por el servidor.

## Referencias

- [Guía oficial de soluciones de OWASP Juice Shop](https://pwning.owasp-juice.shop/companion-guide/latest/appendix/solutions.html): Zero Stars, Login Admin, Admin Registration, View Basket y Christmas Special.
- [Desafíos oficiales de crAPI](https://github.com/OWASP/crAPI/blob/develop/docs/challenges.md): los casos C1, C3, C4, C5 y C6 de la consigna corresponden a los desafíos oficiales 1, 8, 9, 11 y 12, respectivamente.
- [OWASP Top 10 2021 — A04: Insecure Design](https://top10.owasp.org/2021/A04_2021-Insecure_Design/): referencia para los defectos de reglas de negocio de J1, J7, C3 y C4.
- [OWASP API Security Top 10 2019 — API8: Injection](https://api-security.owasp.org/editions/2019/en/0xa8-injection/): referencia para la clasificación de C6.

## Entrega

Entregar un único archivo ZIP o 7ZIP que contenga este `README.md` y la carpeta `evidencias/`, conservando las rutas relativas para visualizar las trece capturas. También puede entregarse un único informe que incorpore las imágenes, de acuerdo con la consigna.
