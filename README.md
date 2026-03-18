# 🕵️‍♂️ Write-up: Estafa Laboral "Tarifa por Adelantado" (Calendly + Stripe)

Este repositorio documenta el análisis técnico de un intento de fraude financiero disfrazado de oportunidad laboral en el área tecnológica. La investigación demuestra cómo los atacantes abusan de la infraestructura de plataformas SaaS legítimas para evadir filtros de seguridad y ejecutar estafas de ingeniería social.

## 📝 Resumen Ejecutivo
Durante una búsqueda activa de oportunidades profesionales, recibí un correo no solicitado ofreciendo un rol de "comunicación en el entorno tecnológico". Al investigar el enlace proporcionado para agendar la entrevista, identifiqué la integración oculta de una pasarela de pagos (Stripe) dentro de una página de agendamiento legítima (Calendly). El objetivo del atacante: obligar al candidato a pagar una tarifa fraudulenta para poder "confirmar" su entrevista.

## 🎣 1. Vector de Entrada: Análisis del Correo
El ataque comienza con una campaña masiva de correos electrónicos. 


**Red Flags (Señales de alerta) identificadas:**
* **Dominio descartable:** El remitente utiliza `@mail2world.com`, un servicio de correo gratuito, en lugar de un dominio corporativo.
* **Inconsistencia de identidad:** El remitente se identifica como "Kevin Torres", pero la firma del correo pertenece a "Clover Craft".
* **Falta de especificidad:** El texto es deliberadamente vago, sin detalles técnicos reales sobre el cargo, apuntando a víctimas desesperadas por entrar al sector IT.

## 🛠️ 2. Análisis Técnico (Frontend & DevTools)
Para confirmar las sospechas sin comprometer la seguridad, se procedió a inspeccionar el DOM y el tráfico de red de la URL proporcionada (`https://calendly.com/...`) utilizando las herramientas de desarrollador del navegador.

### La inyección del Paywall (Stripe)
Al revisar la estructura de la página, se encontró evidencia concluyente de intenciones de cobro:

![Captura del inspector de elementos mostrando el iframe](ruta/a/tu/imagen_elementos.png)

* **Iframe de Pago:** El documento principal carga un `<iframe src="https://js.stripe.com/...">`. Calendly permite esta integración legítimamente, pero aquí se usa como un *paywall* fraudulento.
* **Permisos del navegador:** Al revisar la pestaña *Properties* del iframe, se verificó que cuenta con el atributo `allow="payment *"`, otorgándole acceso a las APIs de pago del navegador (como Google Pay o tarjetas de crédito guardadas).

### Telemetría y Consola
![Captura de la consola y red](ruta/a/tu/imagen_consola.png)

* Los scripts cargados (`out-4.5.45.js`) pertenecen a la telemetría antifraude de Stripe, confirmando que hay un motor de cobro real y activo operando detrás de la supuesta "entrevista de 30 minutos".
* Los errores `ERR_BLOCKED_BY_CLIENT` en la consola demuestran los intentos de la plataforma por cargar scripts de rastreo (`onetrust`) y geolocalización, bloqueados exitosamente por las políticas de privacidad del navegador local.

## ⚙️ 3. Kill Chain (Metodología del Atacante)
1. **Reconocimiento:** Extracción de correos electrónicos de portales de empleo o filtraciones.
2. **Armamento:** Creación de una cuenta gratuita en Calendly y vinculación con una cuenta de Stripe.
3. **Distribución:** Envío masivo del correo de *phishing* con el enlace.
4. **Explotación:** La víctima, creyendo que agenda una entrevista real, ingresa al enlace seguro.
5. **Acción sobre el objetivo:** La víctima se topa con un formulario de cobro obligatorio para reservar la hora, perdiendo su dinero (Advance-fee scam).

## 🛡️ 4. Mitigación y Reporte de Incidentes
Como estudiante de Informática, considero fundamental no solo identificar la amenaza, sino neutralizarla. Se tomaron las siguientes acciones de respuesta:

1. **Reporte a la plataforma (Calendly):** Se envió la evidencia técnica y el correo original al equipo de Trust & Safety (`trust@calendly.com`) para solicitar la baja inmediata de la cuenta y el bloqueo de la integración de Stripe asociada a ese perfil.
2. **Reporte Nacional (CSIRT Chile):** Se escaló el incidente al Equipo de Respuesta ante Incidentes de Seguridad Informática del Ministerio del Interior (`soc@interior.gob.cl`), enviando los Indicadores de Compromiso (IoC) y URLs para la emisión de alertas preventivas, protegiendo a otros ciudadanos en búsqueda de empleo.

---
*Investigación documentada para fines educativos y de concienciación en ciberseguridad.*
