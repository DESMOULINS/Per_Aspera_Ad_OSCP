# Session Hijacking:
El proceso de ingreso o conexión es.

Inicio de conexión ->
<- Acepta conexión
Envia credenciales ->
<- Acepta credenciales
Configuración de la sesión ->
<- Configuración de la sesión (Asigna Sessión ID)
Pide información ->
<- Transmite comunicación
<- Transmite comunicación
<- Transmite comunicación

## Principales vulnerabilidades:
- Ausencia de bloqueo de cuentas por uso de sessions id invalidos
- Tiempo indefinido de vida
- Algoritmos debiles de creación de session ID
- Manejo inseguro de sessions id
- Las conexiones TCP/IP son las más vulnerables
- Falta de encriptación de la conexión

## Proceso normal de robo de sesión:
- Sniff: Ataques MITM 
- Monitor: Analizar el trafico
- Session Desynchronization: Romper la conectividad del cliente al servidor.
- Session ID Prediction: Toma control de la sesión.
- Command Injection: Comienza la inyección en base a la sesión.

- 




































