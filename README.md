# HTB Write-ups · Javier Jiménez Pérez

Documentación de máquinas de [Hack The Box](https://www.hackthebox.com/) resueltas como parte de mi portfolio de ciberseguridad. Cada write-up sigue el mismo esquema: **resumen para no técnicos → enumeración → explotación → impacto y riesgo → mitigación (antes/después)**.

Vengo de diez años de hardware y firmware, y me oriento a respuesta a incidentes y seguridad de dispositivos embebidos. Aquí practico la parte ofensiva para entender mejor cómo se rompe un sistema y cómo se arregla.

## Write-ups

| Máquina | Plataforma | Dificultad | Fallo | Write-up |
|---|---|---|---|---|
| Meow | Starting Point (Tier 0) | Very Easy | Telnet con `root` sin contraseña | [writeups/Meow.md](writeups/Meow.md) |
| Fawn | Starting Point (Tier 0) | Very Easy | FTP con acceso anónimo | [writeups/Fawn.md](writeups/Fawn.md) |

## Evidencia

- [Registro de comandos de Meow y Fawn](evidence/comandos_log_Meow_Fawn.md): comandos y salidas reales de mi terminal, con flags e IPs censuradas.

## Política de publicación

- Solo se publican soluciones de contenido **retirado** o de **Starting Point** de Hack The Box, tal como permiten [sus condiciones](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines). Nunca de máquinas activas.
- Sin flags completas, contraseñas, ficheros de VPN ni datos personales.
- Las IPs de laboratorio se sustituyen por `<IP_OBJETIVO>`.

## Aviso

Todo se ha realizado en el laboratorio autorizado de Hack The Box. Las técnicas descritas solo deben usarse sobre sistemas propios o con autorización por escrito del propietario.

## Uso de IA

La redacción se apoya en un asistente de IA a partir de mis propios registros de terminal. Los comandos y las salidas proceden de mi sesión real y los reviso yo antes de publicar.
