# 🔐 Laboratorio de Criptografía Aplicada: Arquitectura Cliente-Servidor

Guía práctica y pedagógica para la implementación, experimentación y análisis de primitivas criptográficas y protocolos de seguridad en un entorno cliente-servidor controlado.

---

## 📋 Tabla de Contenidos

1. [Alcance y Reglas de Seguridad](#-alcance-y-reglas-de-seguridad)
2. [Arquitectura y Topología de Red](#-arquitectura-y-topología-de-red)
3. [Requisitos y Software Necesario](#-requisitos-y-software-necesario)
4. [Instalación y Configuración del Entorno](#-instalación-y-configuración-del-entorno)
5. [Guía de Actividades Prácticas](#-guía-de-actividades-prácticas)
   - [Actividad A: Hash Criptográfico e Integridad (SHA-256)](#actividad-a-hash-criptográfico-e-integridad-sha-256)
   - [Actividad B: Cifrado Simétrico Autenticado (AES-256-GCM)](#actividad-b-cifrado-simétrico-autenticado-aes-256-gcm)
   - [Actividad C: Criptografía Asimétrica RSA (Claves, Cifrado y Firmas)](#actividad-c-criptografía-asimétrica-rsa-claves-cifrado-y-firmas)
   - [Actividad D: Acuerdo de Claves con Curvas Elípticas (X25519 / ECDH)](#actividad-d-acuerdo-de-claves-con-curvas-elípticas-x25519--ecdh)
   - [Actividad E: Almacenamiento Seguro de Contraseñas (Argon2id)](#actividad-e-almacenamiento-seguro-de-contraseñas-argon2id)
   - [Actividad F: Comunicación Segura HTTPS/TLS Cliente-Servidor](#actividad-f-comunicación-segura-httpstls-cliente-servidor)
   - [Actividad G: Auditoría y Captura de Tráfico de Red (Wireshark)](#actividad-g-auditoría-y-captura-de-tráfico-de-red-wireshark)
6. [Resumen de Primitivas Criptográficas](#-resumen-de-primitivas-criptográficas)
7. [Solución de Problemas Frecuentes (Troubleshooting)](#-solución-de-problemas-frecuentes-troubleshooting)
8. [Entregables y Criterios de Evaluación](#-entregables-y-criterios-de-evaluación)

---

## 🛡️ Alcance y Reglas de Seguridad

- **Ámbito estrictamente académico:** Este laboratorio se desarrolla en un entorno aislado (red interna virtual).
- **No usar secretos reales:** Jamás utilice contraseñas, claves privadas o datos personales reales.
- **Servidor didáctico:** El servidor HTTPS provisto en `server_https.py` está diseñado para pruebas locales; no debe exponerse a redes públicas ni a Internet.
- **Control de versiones:** Nunca suba claves privadas (`.key`, `.pem`) ni hashes de contraseñas de producción al repositorio Git.

---

## 🌐 Arquitectura y Topología de Red

El laboratorio está diseñado para dos Máquinas Virtuales (VMs) de Ubuntu interconectadas mediante una **Red Interna** de VirtualBox denominada `cripto-lan`:

```
+-----------------------------------+             +-----------------------------------+
|          VM 1: SERVIDOR           |             |           VM 2: CLIENTE           |
|                                   |             |                                   |
|  IP: 10.10.10.10/24               |             |  IP: 10.10.10.20/24               |
|                                   |             |                                   |
|  - server_https.py (Puerto 8443)  |  cripto-lan |  - curl / openssl s_client        |
|  - server.crt & server.key        |<===========>|  - Wireshark / tcpdump            |
|  - crypto_lab.py                  | (Aislada)   |  - Copia de server.crt            |
|                                   |             |  - crypto_lab.py                  |
+-----------------------------------+             +-----------------------------------+
```

> 💡 **Nota práctica:** Las **Actividades A a la E** se enfocan en algoritmos locales y pueden realizarse en cualquiera de las máquinas (o incluso en una sola máquina durante el desarrollo). Las **Actividades F y G** requieren explícitamente la interacción cliente-servidor en red.

---

## 📦 Requisitos y Software Necesario

Todas las herramientas utilizadas son libres y gratuitas:

| Componente | Herramienta | Propósito en el Laboratorio |
| :--- | :--- | :--- |
| **Virtualización** | Oracle VirtualBox | Plataforma para ejecutar las VMs aisladas (paquete base gratuito). |
| **Sistemas Operativos** | Ubuntu Server / Desktop | Entornos de ejecución para servidor y cliente. |
| **Herramientas Base** | OpenSSL, curl, tcpdump | Gestión de certificados, clientes HTTP y captura en consola. |
| **Análisis de Red** | Wireshark | Inspección visual de paquetes y análisis del handshake TLS. |
| **Lenguaje** | Python 3 + venv + pip | Ejecución de scripts del laboratorio. |
| **Librería Criptográfica** | `cryptography` (pyca/cryptography)| Implementación robusta de AES-GCM, RSA, OAEP, PSS y X25519. |
| **Librería de Contraseñas**| `argon2-cffi` | Implementación del algoritmo estándar Argon2id. |

---

## 🚀 Instalación y Configuración del Entorno

Ejecute estos pasos en **ambas máquinas virtuales** (Servidor y Cliente) para preparar el entorno:

### 1. Instalar paquetes del sistema
```bash
sudo apt update
sudo apt install -y openssl python3 python3-venv python3-pip curl wireshark tcpdump
```

### 2. Crear y activar el entorno virtual de Python
```bash
mkdir -p ~/lab-crypto && cd ~/lab-crypto
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Instalar librerías criptográficas de Python
```bash
pip install cryptography argon2-cffi
```

### 4. Distribución de archivos
- Copie `crypto_lab.py` en `~/lab-crypto/` de **ambas VMs**.
- Copie `server_https.py` en `~/lab-crypto/` del **Servidor**.

---

## 🧪 Guía de Actividades Prácticas

---

### Actividad A: Hash Criptográfico e Integridad (SHA-256)

#### 🎯 Objetivo
Comprender el concepto de **integridad de datos**, verificar cómo una función hash genera una "huella digital" única e irreversible, y observar el **efecto avalancha** (cómo un cambio mínimo en la entrada altera radicalmente la salida).

#### 💡 Fundamento Criptográfico
- **Unidireccional:** A partir del hash resultante no es posible recuperar el documento original.
- **Determinista:** La misma entrada siempre genera exactamente el mismo hash.
- **Efecto Avalancha:** Si se modifica tan solo un bit o un espacio del archivo, el hash cambiará en aproximadamente el 50% de sus bits de forma impredecible.
- **Importante:** Un hash garantiza *integridad*, pero **no** garantiza *confidencialidad* (el archivo no está cifrado) ni *autenticidad* por sí solo.

#### 🛠️ Paso a Paso

1. **Crear un documento original de prueba:**
   ```bash
   echo "Acta de matrícula de prueba" > acta.txt
   ```

2. **Calcular el hash SHA-256 con la utilidad del sistema y con el script:**
   ```bash
   sha256sum acta.txt
   python crypto_lab.py hash acta.txt
   ```
   *Ambos comandos deben entregar el mismo resultado en formato hexadecimal (64 caracteres).*

3. **Simular una alteración no autorizada (agregar texto al documento):**
   ```bash
   echo "Cambio no autorizado" >> acta.txt
   ```

4. **Recalcular el hash del documento modificado:**
   ```bash
   sha256sum acta.txt
   python crypto_lab.py hash acta.txt
   ```

#### 📋 Salida Esperada
```text
# Antes de la modificación:
SHA-256: 3c8e192a... (un valor hexadecimal de 64 caracteres)

# Después de la modificación:
SHA-256: b74d9f01... (un valor totalmente diferente al anterior)
```

#### 🔍 Análisis para el Reporte
- Compare visualmente ambos hashes. ¿Existe alguna similitud entre el hash previo y el posterior a pesar de que el archivo conserva el texto inicial?
- Explique por qué un hash no sirve para proteger la confidencialidad de los datos.

---

### Actividad B: Cifrado Simétrico Autenticado (AES-256-GCM)

#### 🎯 Objetivo
Aprender a proteger la **confidencialidad** y la **autenticidad/integridad** de un archivo utilizando cifrado simétrico moderno de tipo **AEAD** (Authenticated Encryption with Associated Data) con AES-256 en modo GCM.

#### 💡 Fundamento Criptográfico
- **Cifrado Simétrico:** Utiliza la **misma clave secreta** tanto para cifrar como para descifrar.
- **AES-GCM (Galois/Counter Mode):** Es un algoritmo AEAD. A diferencia de modos antiguos (como CBC o ECB), GCM no solo cifra el texto (confidencialidad), sino que genera una etiqueta de autenticación (*Authentication Tag*). Si alguien modifica un solo byte del archivo cifrado, el descifrado falla inmediatamente, detectando la manipulación.
- **Nonce (Number used Once):** Vector de inicialización de 96 bits (12 bytes). Debe ser único para cada operación con la misma clave. El nonce **no es secreto** y viaja como prefijo en el archivo cifrado.

#### 🛠️ Paso a Paso

1. **Cifrar el archivo `acta.txt`:**
   ```bash
   python crypto_lab.py encrypt acta.txt
   ```
   *El script genera una clave `aes.key` (si no existe) y el archivo cifrado `acta.txt.aesgcm`.*

2. **Verificar los archivos generados y sus permisos:**
   ```bash
   ls -l acta.txt* aes.key
   ```
   *Observe que `aes.key` se crea con permisos `600` (`-rw-------`) para protegerla de otros usuarios.*

3. **Descifrar el archivo cifrado:**
   ```bash
   python crypto_lab.py decrypt acta.txt.aesgcm
   ```
   *Se generará el archivo `acta.recovered`.*

4. **Verificar que el archivo recuperado es idéntico al original:**
   ```bash
   cmp acta.txt acta.recovered && echo "✅ Éxito: Los archivos son exactamente idénticos"
   ```

#### 🧪 Prueba de Alteración (Ataque de Manipulación)
¿Qué ocurre si un atacante altera el archivo cifrado en tránsito?

1. **Crear una copia y alterar un byte del archivo cifrado:**
   ```bash
   cp acta.txt.aesgcm acta_alterada.aesgcm
   # Modificamos un byte mediante Python:
   python3 -c "data = bytearray(open('acta_alterada.aesgcm','rb').read()); data[20] ^= 0xFF; open('acta_alterada.aesgcm','wb').write(data)"
   ```

2. **Intentar descifrar el archivo alterado:**
   ```bash
   python crypto_lab.py decrypt acta_alterada.aesgcm
   ```

#### 📋 Salida Esperada
- **En descifrado normal:**
  ```text
  Descifrado correcto: acta.recovered
  ```
- **En la prueba de alteración:**
  ```text
  cryptography.exceptions.InvalidTag
  ```
  *El sistema detecta inmediatamente que el mensaje fue manipulado y aborta el descifrado sin entregar datos corruptos.*

---

### Actividad C: Criptografía Asimétrica RSA (Claves, Cifrado y Firmas)

#### 🎯 Objetivo
Entender el funcionamiento de un par de claves asimétricas (pública y privada), el cifrado de datos pequeños mediante **RSA-OAEP** y la autenticación/no-repudio mediante **Firmas Digitales (RSA-PSS)**.

#### 💡 Fundamento Criptográfico
- **Par de Claves:**
  - **Clave Pública (`rsa_public.pem`):** Puede distribuirse a cualquier persona.
  - **Clave Privada (`rsa_private.pem`):** Debe mantenerse estrictamente en secreto.
- **Regla de Cifrado (Confidencialidad):**
  - Se cifra con la **Clave Pública** del receptor $\rightarrow$ Solo el receptor con su **Clave Privada** puede descifrar.
  - *Limitación:* RSA es computacionalmente costoso; no se usa para archivos grandes, sino para mensajes cortos o para intercambiar claves simétricas (cifrado híbrido). Se utiliza con relleno seguro **OAEP**.
- **Regla de Firma Digital (Autenticidad e Integridad):**
  - El emisor firma con su **Clave Privada** $\rightarrow$ Cualquier persona puede verificar la firma con la **Clave Pública** del emisor.
  - Si el documento cambia aunque sea una coma, la firma deja de ser válida. Se utiliza con relleno seguro **PSS**.

#### 🛠️ Paso a Paso

#### Parte 1: Generar el par de claves RSA (3072 bits)
```bash
python crypto_lab.py rsa-keys
ls -l rsa_private.pem rsa_public.pem
```

#### Parte 2: Cifrado y Descifrado Asimétrico (RSA-OAEP)
1. **Cifrar un mensaje corto con la clave pública:**
   ```bash
   python crypto_lab.py rsa-wrap "Clave de matrícula temporal: SEC-2026-X"
   ```
   *Genera el archivo cifrado `rsa_message.bin`.*

2. **Descifrar el mensaje con la clave privada:**
   ```bash
   python crypto_lab.py rsa-unlock rsa_message.bin
   ```

#### Parte 3: Firma Digital y Verificación de Integridad (RSA-PSS)
1. **Crear un documento formal:**
   ```bash
   echo "Resolución académica: Matrícula Aprobada 2026" > resolucion.txt
   ```

2. **Firmar digitalmente el documento con la clave privada:**
   ```bash
   python crypto_lab.py sign resolucion.txt
   ```
   *Genera el archivo de firma `resolucion.txt.sig`.*

3. **Verificar la firma con la clave pública (debe ser VÁLIDA):**
   ```bash
   python crypto_lab.py verify resolucion.txt resolucion.txt.sig
   ```

4. **Simular una falsificación o alteración del documento firmado:**
   ```bash
   echo "Adición fraudulenta: Beca 100%" >> resolucion.txt
   ```

5. **Reintentar la verificación (debe ser INVÁLIDA):**
   ```bash
   python crypto_lab.py verify resolucion.txt resolucion.txt.sig
   ```

#### 📋 Salida Esperada
- **Verificación original:**
  ```text
  FIRMA VÁLIDA: contenido coincide con la firma.
  ```
- **Verificación tras alterar el documento:**
  ```text
  FIRMA INVÁLIDA: el archivo cambió o la firma no corresponde.
  ```

---

### Actividad D: Acuerdo de Claves con Curvas Elípticas (X25519 / ECDH)

#### 🎯 Objetivo
Comprender cómo dos entidades distantes (Alice y Bob) pueden establecer una **clave secreta compartida idéntica** comunicándose a través de un canal público inseguro, sin que la clave secreta viaje jamás por la red.

#### 💡 Fundamento Criptográfico
- **Diffie-Hellman sobre Curvas Elípticas (ECDH):** Utiliza las propiedades matemáticas de la curva Curve25519 (X25519).
- **Mecanismo:**
  - Alice genera un par efímero $(a_{priv}, A_{pub})$ y envía $A_{pub}$ a Bob.
  - Bob genera un par efímero $(b_{priv}, B_{pub})$ y envía $B_{pub}$ a Alice.
  - Alice calcula: $S = a_{priv} \times B_{pub}$
  - Bob calcula: $S = b_{priv} \times A_{pub}$
  - Gracias a la conmutatividad matemática, **ambos obtienen exactamente el mismo valor secreto $S$**.
- **Ventaja de Curvas Elípticas frente a RSA:** Claves mucho más pequeñas (256 bits en X25519 ofrecen una seguridad equivalente a RSA 3072 bits) con un rendimiento computacional drásticamente superior.

#### 🛠️ Paso a Paso

1. **Ejecutar la demostración de acuerdo de claves:**
   ```bash
   python crypto_lab.py ecdh-demo
   ```

#### 📋 Salida Esperada
```text
Secreto Alice: 4f8b2c19a... (32 bytes en hex)
Secreto Bob:   4f8b2c19a... (32 bytes en hex)
¿Coinciden?: True
El secreto compartido no se transmitió directamente.
```

#### 🔍 Análisis para el Reporte
- Explique qué vería un espía pasivo que captura el tráfico en la red durante este intercambio.
- ¿Por qué ECDH puro requiere ser complementado con firmas o certificados en un protocolo como TLS? *(Pista: Ataque Man-in-the-Middle).*

---

### Actividad E: Almacenamiento Seguro de Contraseñas (Argon2id)

#### 🎯 Objetivo
Comprender por qué las contraseñas **nunca deben almacenarse en texto plano ni con funciones hash rápidas** (como MD5 o SHA-256), y experimentar con el estándar moderno **Argon2id**.

#### 💡 Fundamento Criptográfico
- **El problema de SHA-256 para contraseñas:** SHA-256 fue diseñado para ser ultra-rápido. Una tarjeta gráfica moderna (GPU) puede calcular miles de millones de hashes SHA-256 por segundo, facilitando ataques de fuerza bruta y diccionarios masivos.
- **Argon2id (Ganador de la Password Hashing Competition - RFC 9106):**
  - **Función lenta y costosa deliberadamente:** Diseñada para tardar decenas o cientos de milisegundos por cálculo.
  - **Memory-Hard:** Exige una cantidad significativa de memoria RAM ($m=65536$ KiB = 64 MB), lo que anula la ventaja de las GPUs y circuitos ASIC en ataques paralelos.
  - **Salt automático:** Genera un valor aleatorio único por cada contraseña para evitar ataques mediante tablas arcoíris (*Rainbow Tables*).

#### 🛠️ Paso a Paso

1. **Generar el hash seguro de una contraseña:**
   ```bash
   python crypto_lab.py password-hash
   ```
   *Ingrese una contraseña de prueba cuando se le solicite (por ejemplo: `SuperClaveSegura2026!`).*

2. **Inspeccionar el formato estructurado del hash generado:**
   ```bash
   cat password.hash
   ```

3. **Verificar con la contraseña CORRECTA:**
   ```bash
   python crypto_lab.py password-check "$(cat password.hash)"
   ```
   *Ingrese exactamente la misma contraseña.*

4. **Verificar con una contraseña INCORRECTA:**
   ```bash
   python crypto_lab.py password-check "$(cat password.hash)"
   ```
   *Ingrese una contraseña errónea para comprobar el rechazo.*

#### 📋 Salida Esperada y Anatomía del Hash
El contenido de `password.hash` tendrá una estructura similar a:
```text
$argon2id$v=19$m=65536,t=3,p=4$qN8k...$W8mR...
```

| Sección del Hash | Significado |
| :--- | :--- |
| `$argon2id` | Algoritmo y variante híbrida (resistente a ataques de canal lateral y GPU). |
| `$v=19` | Versión del algoritmo Argon2 (0x13). |
| `$m=65536` | Costo de memoria: utiliza 64 MiB de RAM por cálculo. |
| `$t=3` | Costo de tiempo: realiza 3 iteraciones sobre la memoria. |
| `$p=4` | Paralelismo: utiliza 4 hilos de ejecución concurrentes. |
| `$salt...` | Salt aleatorio generado automáticamente en Base64. |
| `$hash...` | Valor digest resultante de la función. |

---

### Actividad F: Comunicación Segura HTTPS/TLS Cliente-Servidor

#### 🎯 Objetivo
Integrar las primitivas criptográficas aprendidas (certificados X.509, RSA, ECDH, AES-GCM) en un canal de transporte seguro real mediante **HTTPS / TLS 1.3**.

#### 💡 Fundamento Criptográfico
- **TLS (Transport Layer Security):** Provee un canal seguro sobre TCP.
- **Certificado X.509:** Asocia la clave pública del servidor con su identidad (su IP `10.10.10.10` y dominio `lab.local`).
- **Autenticación del Servidor:** El cliente usa `server.crt` para validar que realmente está hablando con el servidor legítimo y no con un suplantador.

---

#### 🛠️ Paso a Paso

#### Paso 1: Configuración en el Servidor (`10.10.10.10`)

1. **Generar un certificado autofirmado con SAN (Subject Alternative Name):**
   ```bash
   cd ~/lab-crypto
   openssl req -x509 -newkey rsa:3072 -sha256 -days 7 -nodes \
     -keyout server.key -out server.crt \
     -subj "/CN=lab.local" \
     -addext "subjectAltName=DNS:lab.local,IP:10.10.10.10"
   ```

2. **Asegurar la clave privada del servidor:**
   ```bash
   chmod 600 server.key
   ```

3. **Iniciar el servicio HTTPS:**
   ```bash
   python server_https.py
   ```
   *Deje esta terminal abierta. El servidor quedará escuchando en el puerto 8443.*

4. **Copiar el certificado público `server.crt` hacia el Cliente:**
   *Copie el archivo `server.crt` a la máquina cliente mediante `scp` o carpeta compartida:*
   ```bash
   # Ejemplo desde el servidor hacia el cliente:
   scp server.crt usuario@10.10.10.20:~/lab-crypto/
   ```

---

#### Paso 2: Ejecución de peticiones desde el Cliente (`10.10.10.20`)

1. **Petición GET validando el certificado con `--cacert`:**
   ```bash
   cd ~/lab-crypto
   curl --cacert server.crt https://10.10.10.10:8443/
   ```

2. **Petición POST enviando datos dentro del túnel cifrado:**
   ```bash
   curl --cacert server.crt -X POST https://10.10.10.10:8443/ \
     -H "Content-Type: text/plain" \
     --data "matricula=ESTUDIANTE-SEGURIDAD-2026"
   ```

3. **Inspección profunda de la negociación TLS con OpenSSL:**
   ```bash
   openssl s_client -connect 10.10.10.10:8443 -CAfile server.crt -tls1_3
   ```

#### ⚠️ Advertencia Fundamental sobre el parámetro `-k` (`--insecure`)
> **¡NUNCA use `curl -k` en producción!**  
> El flag `-k` le dice al cliente: *"No verifiques el certificado, acepta a cualquiera"*. Si un atacante intercepta la red (Man-in-the-Middle) y presenta su propio certificado falso, `curl -k` lo aceptará sin advertencia, permitiendo al atacante leer y modificar todo el tráfico supuestamente "cifrado". En este laboratorio, confiamos en el certificado mediante `--cacert server.crt`.

#### 📋 Salida Esperada en el Cliente
- **Respuesta GET:**
  ```json
  {"servicio": "API de matrículas de laboratorio", "estado": "activo", "hora_servidor": "..."}
  ```
- **Respuesta POST:**
  ```json
  {"resultado": "solicitud recibida por HTTPS"}
  ```
- **En la terminal del Servidor:**
  ```text
  Solicitud recibida dentro de TLS: matricula=ESTUDIANTE-SEGURIDAD-2026
  ```

---

### Actividad G: Auditoría y Captura de Tráfico de Red (Wireshark)

#### 🎯 Objetivo
Inspeccionar los paquetes que circulan por la interfaz de red para comprobar empíricamente que la información viaja cifrada y analizar las fases del **Handshake TLS**.

#### 🛠️ Paso a Paso

1. **Iniciar Wireshark en la máquina Cliente:**
   - Abra Wireshark en Ubuntu Desktop (o use `sudo tcpdump` en Ubuntu Server).
   - Seleccione la interfaz de red asociada a la red interna `cripto-lan` (por ejemplo: `enp0s8` o `eth1`).

2. **Aplicar filtro de captura / visualización:**
   ```text
   tcp.port == 8443
   ```

3. **Generar tráfico desde el Cliente:**
   Abra una terminal y ejecute nuevamente la petición POST:
   ```bash
   curl --cacert server.crt -X POST https://10.10.10.10:8443/ \
     -H "Content-Type: text/plain" \
     --data "matricula=ESTUDIANTE-SEGURIDAD-2026"
   ```

4. **Detener la captura y analizar el flujo de paquetes:**
   Identifique en la lista de paquetes los siguientes hitos:
   - **TCP Three-Way Handshake:** `[SYN]`, `[SYN, ACK]`, `[ACK]` entre `10.10.10.20` y `10.10.10.10:8443`.
   - **Client Hello:** El cliente envía las suites de cifrado soportadas y su clave pública efímera ECDH.
   - **Server Hello:** El servidor selecciona la suite (por ejemplo `TLS_AES_256_GCM_SHA384`) y envía su clave efímera.
   - **Encrypted Extensions & Certificate:** El intercambio de certificados y validación de identidad.
   - **Application Data Protocol:** Los paquetes que contienen la petición POST y la respuesta.

5. **Inspeccionar el contenido de un paquete de `Application Data`:**
   - Seleccione un paquete de datos y revise la vista hexadecimal inferior.
   - Compruebe que la cadena `"matricula=ESTUDIANTE-SEGURIDAD-2026"` **NO aparece en ningún lado**. El texto está protegido con AES-GCM.

6. **Comparación contra tráfico en texto plano (HTTP sin TLS):**
   - Si se enviaran datos por HTTP tradicional (puerto 80 / 8080), cualquier sniffer podría leer el payload en texto claro con clic derecho $\rightarrow$ *Follow TCP Stream*. En HTTPS esto es imposible.

> 🔒 **Aclaración clave:** Wireshark muestra los metadatos de conexión TLS, pero **nunca** muestra la clave privada del servidor ni el texto plano descifrado de la sesión.

---

## 📊 Resumen de Primitivas Criptográficas

A continuación se detalla la matriz de algoritmos utilizados en este laboratorio:

| Actividad | Primitiva / Algoritmo | Longitud de Clave | Servicio de Seguridad | Estándar / Referencia |
| :---: | :--- | :---: | :--- | :--- |
| **A** | **SHA-256** | Salida 256 bits | **Integridad** de datos | FIPS PUB 180-4 |
| **B** | **AES-256-GCM** | 256 bits (Nonce 96b) | **Confidencialidad + Integridad** (AEAD) | NIST SP 800-38D |
| **C** | **RSA-OAEP** | 3072 bits | **Confidencialidad** (Mensajes cortos / Claves) | RFC 8017 (PKCS#1 v2.2) |
| **C** | **RSA-PSS** | 3072 bits | **Autenticidad, Integridad y No Repudio** | RFC 8017 (PKCS#1 v2.2) |
| **D** | **X25519 (ECDH)** | 256 bits (Curva 25519) | **Acuerdo de Secretos Compartidos** | RFC 7748 |
| **E** | **Argon2id** | Salida 256b, RAM 64MB | **Almacenamiento Seguro de Contraseñas** | RFC 9106 |
| **F / G** | **TLS 1.3 / HTTPS** | Híbrido (ECDH + AES) | **Seguridad en Capa de Transporte** | RFC 8446 |

---

## 🔧 Solución de Problemas Frecuentes (Troubleshooting)

### 1. `ModuleNotFoundError: No module named 'cryptography'`
- **Causa:** El script se ejecutó con el Python del sistema en lugar del entorno virtual.
- **Solución:** Active el entorno virtual antes de ejecutar:
  ```bash
  source ~/lab-crypto/.venv/bin/activate
  ```

### 2. `curl: (60) SSL certificate problem: self-signed certificate`
- **Causa:** curl no confía por defecto en certificados autofirmados.
- **Solución:** Especifique el certificado con `--cacert server.crt`. No use `-k`.

### 3. `OSError: [Errno 98] Address already in use` al iniciar `server_https.py`
- **Causa:** Quedó una instancia previa del servidor ejecutándose en segundo plano en el puerto 8443.
- **Solución:** Localice y finalice el proceso:
  ```bash
  sudo lsof -i :8443
  # O mate el proceso directamente:
  sudo fuser -k 8443/tcp
  ```

### 4. Permisos denegados en claves privadas
- **Causa:** Permisos de archivo inseguros.
- **Solución:** Asigne permisos exclusivos al propietario:
  ```bash
  chmod 600 server.key rsa_private.pem aes.key
  ```

---

## 📑 Entregables y Criterios de Evaluación

Para la sustentación e informe técnico del laboratorio, asegúrese de recopilar las siguientes evidencias:

1. **Diagrama de Red y Flujo Criptográfico:**
   - Esquema visual que represente la topología y el paso de datos entre las VMs.
2. **Registro de Evidencias (Capturas y Logs):**
   - **Actividad A:** Captura mostrando los hashes antes y después de la modificación.
   - **Actividad B:** Evidencia del descifrado exitoso (`cmp`) y del error `InvalidTag` al corromper el archivo.
   - **Actividad C:** Captura del mensaje descifrado con RSA, verificación de firma válida y de firma inválida tras alteración.
   - **Actividad D:** Demostración de coincidencia de secretos en ECDH.
   - **Actividad E:** Hash de Argon2id desglosado y prueba de contraseña correcta vs incorrecta.
   - **Actividad F:** Salida de `curl` con `--cacert` y negociación en `openssl s_client`.
   - **Actividad G:** Captura de Wireshark con el filtro `tcp.port == 8443` mostrando el Handshake y el contenido cifrado de `Application Data`.
3. **Análisis Crítico:**
   - Respuesta a las preguntas de reflexión de cada actividad.
   - Matriz de riesgos identificando por qué no deben usarse configuraciones inseguras (`curl -k`, SHA-256 para passwords, reutilización de nonce).
