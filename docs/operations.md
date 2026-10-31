# Operación diaria

Lo que se aprendió usando el sistema todos los días. Son prácticas del autor, no un producto: adaptalas a tu caso.

## 1. Que arranque solo
Cada asistente corre en su propia sesión `tmux` administrada por un servicio de usuario de systemd
([`ai-consoles.service`](../examples/systemd/ai-consoles.service)). Con `loginctl enable-linger` las sesiones vuelven solas
después de un corte de luz o un reinicio. Los scripts auxiliares de la sesión son sólo atajos para entrar a mirar.

## 2. Supervisar los procesos, no sólo la sesión
**Que `systemd` vigile `tmux` no alcanza.** Si la sesión sigue viva pero se cayó un proceso auxiliar (por ejemplo un conector
que usa el asistente), todo parece andar y no responde. La solución fue un **chequeo de salud con un temporizador (cada 2 minutos)**
que comprueba la sesión y esos procesos auxiliares y, si falta alguno, reinicia el servicio y deja constancia.

## 3. Reinicio limpio una vez por día
Los asistentes retoman la conversación anterior al reiniciar (`--continue`), pero con el tiempo el contexto crece y la
sesión se vuelve lenta y cara. Una vez por día, de madrugada, se hace un **cierre limpio**: la sesión arranca de cero y lo
importante queda en archivos de memoria y de pendientes. El resto de los reinicios sí conservan la conversación.

## 4. Cuidar el cupo
- El modelo intermedio es el predeterminado; el más potente se usa sólo para decisiones con dinero o diseño complejo
  (ver [`models.md`](models.md)).
- Las consignas largas se derivan según el riesgo del dato ([`architecture.md`](architecture.md)).
- Umbrales semanales de consumo como política operativa de la persona: 60 % ahorro, 85 % crítico, 90 % reserva.
- **Los cambios de modelo los hace sólo la persona.** Una consola que muestre un aviso de límite con opción de cambiar de modelo
  se deja como está hasta que ella decida.

## 5. Mirar varios equipos
Además de las consolas ([`monitoring.md`](monitoring.md)), una tarea programada cada minuto junta el uso de CPU, memoria y disco de
cada equipo y lo publica en la misma página protegida. Un archivo `touch` sirve como **pausa** de la publicación.

## 6. Qué conviene tener a mano
- Una copia cifrada de la configuración y una forma de restaurarla en una placa nueva.
- La lista de qué servicio depende de cuál, para saber qué reiniciar primero.
- Un lugar donde queden anotados los pendientes, porque la conversación se reinicia.
