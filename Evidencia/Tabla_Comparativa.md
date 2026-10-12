# Tabla comparativa de resultados OWASP ZAP

Proyecto analizado: FrontEmpresa (Blazor, .NET 10, InteractiveServer) en `https://localhost:7020/cliente`

| | Antes | Después |
|---|---|---|
| Total de alertas | 11 | 5 |
| Riesgo alto | 0 | 0 |
| Riesgo medio | 4 | 0 |
| Riesgo bajo | 3 | 1 |
| Informativas | 4 | 4 |

Evidencia: `Informe_ZAP_Antes.html` e `Informe_ZAP_Despues.html`.

## Comparativa por alerta

| Alerta encontrada | Riesgo | Corrección realizada | Resultado final |
|---|---|---|---|
| CSP: Directiva Wildcard | Medio | CSP propia sin comodines; las fuentes se limitan a `'self'` | Corregida |
| CSP: Failure to Define Directive with No Fallback | Medio | Se definieron `default-src`, `form-action`, `base-uri`, `object-src` y `frame-ancestors` | Corregida |
| CSP: script-src unsafe-inline | Medio | `script-src 'self'` con nonce por petición para el script del import map | Corregida |
| CSP: style-src unsafe-inline | Medio | `style-src 'self'` | Corregida |
| Cookie Sin Flag de Seguridad | Bajo | Cookie antiforgery con `HttpOnly`, `Secure` (Always) y `SameSite=Strict` | Corregida |
| Falta encabezado X-Content-Type-Options | Bajo | Cabecera `X-Content-Type-Options: nosniff` | Corregida |
| Strict-Transport-Security Header No Establecido | Bajo | `AddHsts` (365 días, includeSubDomains) y `UseHsts` fuera de Development | Mitigada |
| Aplicación Web Moderna | Informativo | No requiere modificación | Informativa |
| Divulgación de información - Comentarios sospechosos | Informativo | No requiere modificación | Falso positivo |
| Reexaminar las Directivas de Control de Caché (4) | Informativo | `Cache-Control: no-store` por defecto en respuestas dinámicas | Informativa |
| Respuesta de Gestión de Sesión Identificada (2) | Informativo | No requiere modificación | Informativa |

## Justificación de las alertas que permanecen

- **Strict-Transport-Security (mitigada):** ASP.NET Core excluye `localhost` de HSTS a propósito y solo envía el header fuera del entorno Development. El código deja HSTS configurado para producción, por eso en el análisis local la alerta persiste. Forzarlo en localhost haría que el navegador recuerde que `localhost` solo funciona por HTTPS y afectaría a otros proyectos locales.
- **Aplicación Web Moderna (informativa):** ZAP solo avisa de que la página usa JavaScript moderno (Blazor). No es una vulnerabilidad.
- **Comentarios sospechosos (falso positivo):** el hallazgo proviene de archivos JavaScript del framework (Blazor y Bootstrap), no de código propio.
- **Directivas de Control de Caché (informativa):** corresponde a archivos estáticos del framework, cuyo caché es intencional.
- **Gestión de Sesión Identificada (informativa):** ZAP detecta la cookie antiforgery de Blazor. La aplicación no tiene login ni sesiones de usuario.

## Cambios realizados en el código

- `Program.cs`: middleware con las cabeceras de seguridad (CSP con nonce, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `Cache-Control`), configuración de la cookie antiforgery y de HSTS.
- `Components/App.razor`: se agregó el nonce al componente `ImportMap` para que el script en línea cumpla con la CSP.
