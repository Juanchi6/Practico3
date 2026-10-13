# Práctico 3 — Desarrollo Seguro de Software

Pruebas realizadas sobre OWASP Juice Shop y OWASP crAPI en un laboratorio local de Kali Linux.

## Avance

| Aplicación | Resueltos | Requeridos |
| --- | ---: | ---: |
| Juice Shop | 1 | 5 |
| crAPI | 0 | 5 |

## OWASP Juice Shop

### J1 — Zero Stars

**Vulnerabilidad:** validación insuficiente del valor de la calificación. El formulario limita los valores disponibles, pero el servidor aceptó una calificación de cero.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design (validación de reglas de negocio insuficiente).

**Prueba de concepto:**
1. Abrí `Customer Feedback` y envié un comentario con una estrella.
2. En las herramientas de Firefox (`Network`), identifiqué `POST /api/Feedbacks/`.
3. Utilicé `Edit and Resend` y cambié `rating: 1` por `rating: 0`, conservando los otros campos.
4. El servidor respondió **HTTP 201 Created**, aceptando la valoración de cero estrellas.

**Evidencia:**

![Solicitud con rating 0 y respuesta HTTP 201](evidencias/juice-shop/j1-zero-stars.png)

**Mitigación:** validar también del lado del servidor que `rating` sea un entero dentro del rango permitido (1–5) y rechazar los demás valores con un error 400.

## OWASP crAPI

_Pendiente._

## Entrega

Un único ZIP con este `README.md` y las evidencias enlazadas.
