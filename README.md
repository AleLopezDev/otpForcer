# otpForcer

Uso: python3 otpForcer.py <num_hilos> <url> [proxy]

Este script en Python realiza un ataque de fuerza bruta contra un endpoint que verifica códigos OTP (One-Time Passwords), sera funcional siempre y cuando el servidor NO bloquee las peticiones por parte del cliente una vez se haya alcanzado el número máximo posible. Utiliza múltiples hilos para probar todas las combinaciones posibles de un código OTP de 6 dígitos, permitiendo el uso opcional de un proxy para anonimizar las solicitudes. Además, incluye un manejo adecuado de excepciones para validar URLs y conexiones, asegurando un cierre correcto en caso de error.

No me hago responsable por el mal uso que se le pueda dar a esta herramienta.

https://github.com/AleLopezDev/otpForcer/assets/113992535/092e1178-5843-42ac-a0e5-1d9c551246d0

