### ¿Qué es MITRE ATT&CK?

Es una **matriz/base de conocimiento pública** mantenida por MITRE que cataloga las **tácticas, técnicas y procedimientos (TTPs)** que usan los atacantes reales, basada en observaciones de campo (incidentes reales, threat intel, red teams).

La idea central: en vez de pensar "¿qué vulnerabilidad tiene este sistema?", ATT&CK te hace pensar **"¿qué hace un atacante paso a paso desde que entra hasta que logra su objetivo?"**

**Estructura de 3 niveles:**

1. **Tácticas** = el "por qué" — el objetivo táctico del atacante en cada fase (ej: obtener acceso inicial, escalar privilegios, moverse lateralmente). Son las columnas de la matriz.
2. **Técnicas** = el "cómo" — la forma concreta de lograr esa táctica (ej: phishing, explotación de una app pública, dumping de credenciales).
3. **Sub-técnicas** = variantes más específicas de una técnica (ej: dentro de "Phishing", está "Spearphishing Link" o "Spearphishing Attachment").

Cada técnica tiene un ID (ej. `T1190` = Exploit Public-Facing Application), lo cual sirve como lenguaje común entre analistas, red teams y blue teams.

**Las 14 tácticas del framework Enterprise** (en orden aproximado del ciclo de ataque):

Reconnaissance → Resource Development → Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → Command and Control → Exfiltration → Impact

**Para qué se usa en la práctica:**

- Blue team: mapear qué técnicas detecta o no su SOC/SIEM (ATT&CK Navigator)
- Red team / pentester: estructurar un test siguiendo TTPs realistas en vez de solo buscar CVEs sueltos
- Threat intel: describir el comportamiento de grupos APT con un vocabulario estandarizado
- Reportes: en vez de decir "encontramos XSS", podés mapearlo a la técnica ATT&CK correspondiente y dar contexto de qué haría un atacante con eso

Como estás con Caido en pentesting web, ATT&CK te da más peso en tácticas como **Initial Access**, **Execution**, **Credential Access** y **Exfiltration**, que son las que más aparecen en compromisos vía aplicaciones web.

**Procedimiento** = la implementación específica y real que usa un atacante (o grupo) concreto para ejecutar una técnica. Es el nivel más detallado y contextual de los tres.

Para verlo en capas:

|Nivel|Pregunta que responde|Ejemplo|
|---|---|---|
|Táctica|¿Por qué?|Acceso Inicial|
|Técnica|¿Cómo, en general?|Exploit Public-Facing Application (T1190)|
|**Procedimiento**|¿Cómo, exactamente, en este caso real?|El grupo APT29 explota una vulnerabilidad de deserialización específica en Microsoft Exchange, con este payload puntual y esta secuencia de comandos, para lograr RCE|

O sea: la técnica es genérica y reutilizable ("explotar una app pública"), el procedimiento es la receta concreta que un actor de amenazas específico ejecutó — con su payload, sus herramientas, su orden de comandos. Por eso en las páginas de ATT&CK vas a ver, debajo de cada técnica, una sección de **"Procedures"** donde listan ejemplos reales de grupos APT usando esa técnica de formas distintas.

En tus reportes de pentesting esto se traduce así: la técnica te dice qué categoría de ataque mapear (T1190), y el procedimiento sos vos describiendo exactamente qué endpoint, qué payload y qué pasos usaste con Caido para explotarlo — esa es tu evidencia/pasos de reproducción, que ya tenías en la plantilla del informe.