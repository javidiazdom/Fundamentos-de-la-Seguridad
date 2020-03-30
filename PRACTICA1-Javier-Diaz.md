# Practica 1 Fundamentos de la Seguridad

#### Javier Díaz Domínguez

#### 2019-2020

## Cifrado simétrico de documentos.

En este apartado de la práctica se realizarán cifrados de documentos de texto empleando diferentes algoritmos de cifrado simétrico. Para ello se empleará la herramienta openssl, concretamente para este apartado el comando `enc` con sus diferentes variantes de modo de operación.

### Comando `enc`

Este comando sirve para hacer cifrados simétricos. Para ello utiliza diferentes algoritmos de cifrado, tanto de bloque como de flujo, utilizando contraseñas proporcionadas por el usuario como claves.

Entre sus opciones más importantes podemos encontrar las opciones de gestión de contraseña (`-k`,`-kfile`,`-pass`), los de selección de modo de operación (`-e`, `-d`) para seleccionar entre cifrado y descifrado del archivo de entrada, y finalmente la opción de selección de modo de operación, con la que podremos seleccionar el modo de cifrado. De estos, los más comunes son:

* __ECB (Electronic CodeBook)__: El más simple, simplemente se divide el mensaje en bloques y se cifran por separado.
* __CBC (Cipher Block Chaining)__: También trocea el mensaje en bloques, pero además utiliza XOR para combinar el cifrado del bloque previo con el texto plano del bloque siguiente. Para cifrar el primer bloque, se emplea un vector de inicialización que cambia aleatoriamente cada vez que se inicia un cifrado.
* __CFB (CipherFeedBack)__: Este modo de cifrado convierte un cifrado de bloque en un cifrado de flujo: para ello hace que el cifrado en bloque opere como una unidad de flujo de cifrado, generando bloques de flujo de claves que son operados con XOR y el texto plano para obtener el texto cifrado.
* __OFB (Output FeedBack)__: Convierte un cifrado de bloque en un __cifrado de flujo síncrono__. Genera bloques pseudoaleatorios que son operados a través de XOR con el texto plano para generar el texto cifrado. Requiere vector de inicialización.
* __CTR (Counter)__: Genera los bloques pseudoaleatorios utilzando un contador, que no es más que una función que produce una secuencia que se garantiza que no se repetirá en mucho tiempo.
* __DES__: Es el algoritmo prototipo del cifrado por bloques. Transforma un texto plano de una longitud determinada de bits en otro texto cifrado de la misma longitud. Utiliza cifrado en bloque, de 64 bits (56 efectivos, 8 últimos de paridad). La clave es de 64 bits. Este algoritmo está __roto__.
* __3DES (Triple DES)__: utiliza una clave de 168 bits, aunque la eficiciencia real es de 112 bits. Los bloques de cifrado siguen siendo de 64 bits. Este algoritmo no es tan vulnerable a los ataques por fuerza bruta como DES.
* __AES__: Está basado en una estructura de bloques, utilizando bloques de 128 bits y claves de 128, 192 y 256 bits. El cambio de un solo bit en la clave o en el bloque de texto a cifrar produce cifrados completamente distintos. Se caracteriza por su robustez y rapidez.
* __RC4__: Cifrado de flujo, hoy en día se considera débil. Utiliza dos algoritmos para el cifrado: KSA y PRGA.
* __Chacha20__: Sistema de cifrado de flujo que soporta claves de 128 y 256 bits. Está basado en una función pseudoaleatoria basada en operaciones add-rotate-xor.

### Creación del fichero de texto.

Para la demostración práctica del funcionamiento de estos algoritmos, se crea un fichero de texto con el siguiente contenido

```
Hola, esto es un fichero de prueba para el cifrado mediante 5 algoritmos
distintos. Utilizaremos los algoritmos AES y TDES obligatoriamente, y posteriormente un algoritmo de flujo. Los demás serán de libre elección. Para esta práctica concretamente se seleccionan los siguientes algoritmos: aes128, des3, rc4, cbc, chacha20.
```
### Cifrado y descifrado

Se procede al cifrado utilizando los 5 algoritmos simétricos:

- AES 128
```bash
openssl enc -aes128 -e -in small-text.txt -out aes128-small-text.cif
```

- Triple DES

```bash
openssl enc -des3 -e -in small-text.txt -out des3-small-text.cif
```

- RC4
```bash
openssl enc -rc4 -e -in small-text.txt -out rc4-small-text.cif
```

- DES CBC
```bash
openssl enc -des-cbc -e -in small-text.txt -out des-cbc-small-text.cif
```

- Chacha20
```bash
openssl enc -chacha20 -e -in small-text.txt -out chacha20-small-text.cif
```

Y posteriormente se procede al descifrado de los archivos cifrados

```bash
openssl enc -aes128 -d -in aes128-small-text.cif -out aes128-small-text.txt
openssl enc -des3 -d -in des3-small-text.cif -out des3-small-text.txt
openssl enc -rc4 -d -in rc4-small-text.cif -out rc4-small-text.txt
openssl enc -des-cbc -d -in des-cbc-small-text.cif -out des-cbc-small-text.txt
openssl enc -chacha20 -d -in chacha20-small-text.cif -out chacha20-small-text.txt
```

### Tamaño de los archivos cifrados

A continuación se vuelcan por pantalla los diferentes tamaños de los archivos cifrados con los distintos algoritmos.
``` bash
$ stat * -c "%N %s"
'aes128-small-text.cif' 352
'chacha20-small-text.cif' 345
'des3-small-text.cif' 352
'des-cbc-small-text.cif' 352
'rc4-small-text.cif' 345
'small-text.txt' 329
```

Como podemos observar, el tamaño original del archivo es de 329 bytes. Al utilizar los algoritmos de flujo `chacha20` y `rc4`, obtenemos un tamaño de fichero de 345 bytes. Esto se debe a la aparición de la sal al inicio del fichero, que es de 16 bytes.
Los tamaños de los ficheros comprimidos con los cifradores de bloque también tienen su explicación:
* Para `aes-128`, al fichero se le añaden 16 bytes de sal, $16 + 329 = 345$, y posteriormente se le añade el denominado *padding* para completar el tamaño del último bloque del cifrado hasta los 128 bits (16 bytes). Gracias al padding, el tamaño total del fichero es múltiplo de 16 bytes (128 bits).
* Para `des3` ocurre lo mismo que con `aes-128`, y el padding coincide ya que el número más cercano de 345 múltiplo de 8 (que es el tamaño de bloque de `des3` en bytes) coincide con el de 16.
* `des-cbc` comparte tamaño de bloque con `des3` por lo que la situación es la misma.

### Contenido de los ficheros encriptados

* Fichero original

```
Hola, esto es un fichero de prueba para el cifrado mediante 5 algoritmos
distintos. Utilizaremos los algoritmos AES y TDES obligatoriamente, y posteriormente un
algoritmo de flujo. Los demás serán de libre elección. Para esta práctica concretamente se seleccionan los siguientes algoritmos: aes128, des3, rc4, cbc, chacha20.
```

* `aes-128`
```
Salted__�����c�S��R��O��I�
���Ǖ/�'�-$ �$��Ɏ��l��f�ʻ�D��(�)��Z��	�I�o����gP��ϡQ���#�j��0��f�5�2�4x.��	C�O�mz`���(��u��˦�~����Kɸ�x���R�4"������$xS��R��x�v�3\�H���L�a�l�bHl�Z#�t� o���U�j1{�� �:]�������z3�w�J*3���f�V���՟�9�l
M��N��Xޫ��A�[2��,(�3���O�}�p�<�c:�7�%
```
* `des3`
```
6O�U�FD��5-8>�5��)wа��XZ�Ja��)����ѹ���'u��5f9i����'��Z�2�gj�W�^K���O=˚�ȂVp�����߷��)1ӯ7X���q|׵��%m�(���f9V�w*3��Od@~Ġ)5��_�@��0��;�q���8�����5����.��r�Cl����(M�4�RA�cB#R��*�4׊����������)ڤ
\7i�% 
```
* `des-cbc`
```
Salted__��f��?{[���oZ�߰j�1odр��H<�-B��7j��c*/�E������Ye�8V��]��ҟ�����A}���Qk��_�j��&;N�)����;
9��:9����n6�*����+����JK��)�0�7����z/wF~q�zE&�
                                              �z� �I=K�hn��r�B��%�[U�`]QX���p�֟�c�֨��6��
          ��<�g���P�E�%V~ǵ�j15�0��j[�V����K�5Ԟ �`�q�{�$��Ix�5�v%
```
* `chacha20`
```
Salted__N����G�F��d'������n�)��]U��WP�]�5��
                                           JEx�=4��z�zNAh�Qb��>z?K&�!}zl�~8���B��kF@����a���\��~�Ǽb�����/~ƫ��ĭ��j��~mH�@����za:J<�0J�e�V����K�A��$kp|O<H��ʩ���V�����;�;�
                                                                          �]xʼ��XeUZ���I/�u���1RX�k�nQ}OTȘw�E?�WsR��L:��e�;��vF!�cۜD@3����%�v%
```
* `rc4`
```
Salted__qh��ٗ
            ru��W?8�4�4
9%�S�0�����3�5:G��XO
�f�����q�>�j��W�s�@�#����s���P!�
6:�U/p�Jt���q^��H�X;��6��7]�Ӭy�i\e��{��.~�0�&�Z�O�΋W�t!��}�)��w}n��� '��o~l�>����ko@�@���	�����n�pr�AT��c���d��&	қ��Ч�}��*FC��e��#�#̛�r�)��&K.�,���~Sq
6�.f%@&�{�����@k����MŞ
```

### Gestión de contraseñas

PKCS (Public Key Cryptography Standard) es un grupo de **estándares** de utilización de técnicas criptográficas en la gestión de claves públicas en algoritmos criptográficos. Concretamente openssl nos permite utilizar `pbkdf1` y `pbkdf2`, que son implementaciones de PKCS. La principal diferencia entre ambos es que `pbkdf1` no es capaz de generar claves de más de 160 bits, mientras que `pbkdf2` lo hace de 128, 256 y 512 bits.

El algoritmo funciona aplicando una función pseudoaleatoria (HMAC con una función hash aprobada) a la contraseña de entrada junto con un valor de sal, repitiendo este proceso muchas veces (mínimo 1000 iteraciones) produciendo una clave derivada. Ésta se puede utilizar como una clave criptográfica en operaciones posteriores. También se puede utilizar la salida de este algoritmo para rellenar el vector de inicialización. El uso de la sal en la gestión de la contraseña permite que para una misma contraseña se generen diferentes claves criptográficas.

### Descifrado de un fichero con clave, vector y sal.

Se propone la demostración de que un fichero puede ser descifrado con la clave criptográfica, el vector de inicialización y la sal, sin el conocimiento de la contraseña.

Para proceder con la demostración en primer lugar se cifra un fichero. Se utiliza como contraseña prueba, y la opción `-p` para mostrar el vector de inicialización, la sal y la clave.

```bash
$ openssl enc -des-cfb -in small-text.txt -out cfb-small-text.cif -k prueba -p > cvs.txt
```

comprobamos el contenido del fichero generado

```bash
$ cat cvs.txt 
salt=99C6D2581D8BF4E0
key=952CEA08115F4846
iv =FA67BC49486D61CB
```

preparamos el fichero eliminando los primeros 16 bytes de la sal del inicio del fichero
```
$ cat cfb-small-text-1.cif | dd ibs=16 obs=16 skip=1 > cfb-small-text1.cif
20+1 registros leídos
20+1 registros escritos
329 bytes copied, 0,000149952 s, 2,2 MB/s
```

y procedemos al descifrado utilizando estos datos

```bash
$ openssl enc -des-cfb -d -in cfb-small-text1.cif -K 952cea08115f4846 -iv fa67bc49486d61cb
Hola, esto es un fichero de prueba para el cifrado mediante 5 algoritmos
distintos. Utilizaremos los algoritmos AES y TDES obligatoriamente, y posteriormente un
algoritmo de flujo. Los demás serán de libre elección. Para esta práctica concretamente se seleccionan los siguientes algoritmos: aes128, des3, rc4, cbc, chacha20.
```

### Demostración de la peligrosidad del modo de operación ecb

Para ello utilizaremos la siguiente imagen

![ulpgc.png](ulpgc-pgm.png)

Se procede eliminando las cabeceras para la encriptación

```
$ head -n 3 ulpgc.pgm > header.txt
```

se extrae el cuerpo de la imagen
```
$ tail -n +4 ulpgc.pgm > body.bin
```

se cifra el cuerpo de la imagen
```
$ openssl enc -aes-128-ecb -nosalt -k prueba -in body.bin -out body.cif
```

por último, se juntan las cabeceras y el cuerpo cifrado
```
$ cat header.txt body.cif > ulpgc-ecb.pgm
```

y se obtiene la siguiente imagen

![ulpgc-ecb.png](ulpgc-ecb.png)

que efectivamente prueba la peligrosidad del modo de operación ecb, ya que deja ver patrones similares a los de la foto original. En contrapunto, si utilizaramos otro modo de encriptación como cbc, el resultado sería el siguiente

![ulpgc-cbc.png](ulpgc-cbc.png)

## Cifrado y comprobación de resúmenes: Generación de claves asimétricas (pública-privada) y firmado de resúmenes

Openssl ofrece la posibilidad de generar resúmenes de archivos con diferentes algoritmos de resumen mediante la opción `dgst`. Haciendo uso de esta herramienta, se generan varios resúmenes utilizando los siguientes algoritmos

1. SHA-1
```bash
$ openssl dgst -sha1 small-text.txt
SHA1(small-text.txt)= 0df857cb098430eeb259bf9e0c830889d3174f47
```
3. SHA224
```bash
$ openssl dgst -sha224 small-text.txt
SHA224(small-text.txt)= 64010395cc20c147c7a7566a54a6ac5be73cb04a30e1389548c2183a
```
3. SHA384
```bash
$ openssl dgst -sha384 small-text.txt
SHA384(small-text.txt)= 54ffce1aa48d5e4ac279b5841759c5580d6e70f09f558cc3a3f6cfbde21760e499815dd214be19297f84671dc048cf9b
```

Se realiza un pequeño cambio en el contenido del fichero: se sustituyen dos caracteres, y se comprueba que, en efecto, los resúmenes del archivo cambian considerablemente

1. SHA-1
```bash
$ openssl dgst -sha1 small-text.txt  
SHA1(small-text.txt)= 1cae6758d51562e16a6c99f9fa55fb6e852527c8
```
3. SHA224
```bash
$ openssl dgst -sha224 small-text.txt
SHA224(small-text.txt)= d6da3cb542cee30107019fb4b3a616c4dbcebdb92d308b02b8acc912
```
3. SHA384
```bash
$ openssl dgst -sha384 small-text.txt
SHA384(small-text.txt)= a7e82cb8640ccd70e455d5285fa89ea67be185f5a7f1111f59d1be99d817927551208fa548ad8e1e6bcd7a7efd926ae9
```

### Generación de un par de claves asimétricas RSA de 2048 bits.

Se genera la clave privada

```bash
$ openssl genpkey -algorithm RSA -aes256 -out privateRSA.pem -pkeyopt rsa_keygen_bits:2048
```

y se introduce como contraseña `prueba`. A continuación se muestra el contenido del fichero que contiene la clava privada

```
$ cat privateRSA.pem 
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFLTBXBgkqhkiG9w0BBQ0wSjApBgkqhkiG9w0BBQwwHAQI4V+LRBBOFzoCAggA
MAwGCCqGSIb3DQIJBQAwHQYJYIZIAWUDBAEqBBCtVxlBvJVXvGRlpEilcQQUBIIE
0JOTlbyl+NBrBwu01bbZvIOiDesAiXGKBnn36plukfFkEkf/cMNR7vyDqtqe0/Sz
H5gDLWKnS8BM8gnnywMaeG+jJ9ILvKIEOw5zzHT2Ji3hHze4CeEHBwRSMQR3B01Y
Kdy62I61HoBZomkR+BLN6uWVakL6kom6jVIyUIMZTm00sCSf+5gTYdTMxWNyV/s0
RkUihENPpI/dvwlTaWjjeD25oEtXPLhat6BO+9RpywcXn/PDAXdqpqYdXZ/sBdMa
SaMfb5FbhNcGtV96iw3lzsg7Tzn17AFv1nKHmlzl+cbNBcUSxsMFNBoQ8yrDq3ze
X2BYJcEk7mzt/jB+SaT9GkDTR9QHy0tv1qv/Pv/vutF+B+IwkDymC0vBiCpjrnxw
H1CizDr4nVMjnye+UISLuJqULazblNuykgIezm28VY23SwDpmglynjIvCO9ueaLF
uB4kaVC7FGBK2XVSLMtX1JMsvsMbdL+nNNvdnhVwkB0sBwplXicmlwwkfRAwgCmT
f6AKUou+/1IMogXgo14C3HpL3uObKM7dPrDxKnpSMGaMGLi/tp4SL8e3TGL/2Gs8
e0Unxo8CJTBAXhL4NMdUvw0CCqXA7i97WwxeVx9COQaDAaVTSxQsxHvFBnNJGMGS
2xvX05nR6iWeofVpme5xdHQPNiE/DnJS3E8MdPwGp3lX+7ADCv2YAnYFEw5SFXSE
LktoyAfg7OJPBfx0vee7AlMzHKMkokjNQ2E8QftlOuwg+xGex4BwDrTuM+wpjpZe
e3Uj0r/Z/6zhKUBnHln9ixb61hOGt3e1RMUyVhGpicWgAOeq6PXv4qgnOS4SzaGj
jPtZokGG5WDLw4TiZBQbuXjMGjP0gd73TYiEzspNUWea3TwDUDDcb4wMdBOJcy0Z
w6NxX56j7HjnEfmULjAXdiasubQ4f3wL3T98+hu2EGoiKCOMx1Sa91JQ4JIg5UOk
H3kuoLrunvaSvBQGDHiq3vmKWXdvBVrONcm2gevFrAcPqukvCRF6qlnLTqjVAhxJ
vYgXw+KsHAY/fD50m2r501ToLG0ORSRsz7DPYCXLH9jbaLDnhY2JXn9C3LgRwyHf
i+BOpNexmXPlG4OuNESVb5VWUQrhTPG1w12E3sCe/1WlVPGa+ujErQtReJTrZrgi
vW6lvxHr0iUVnKK2VAZ63K01Bbu4OBfGNEDDAPKhU2phM4/souKxs/jRK1QXPck9
msqWWLC6ZTaog2z6O5e8lLhpngwh/wNkhHsNBRedP3RRhpEdbWa0rfNE0vbL2skW
6NeLdqIiVMiJf3DlD9hmU3bPnnLcDgdip7fesVt70WUoKCuPNIJ6ea9e3UjRiWTl
fWfULmWLj3uAM61FJEo9fNb8I55y3qmzCnxTobQaPQa4fKqvVkspcilvWUXL6QFr
o6acJ9YUcHWFT9+ODjYd8JdYQ0XPMGaUzLtYl20DBf0mPzBFZ0A1853QabFViyeR
l1olOYJZSiJjo2GnIgl9ot+0ZmOkAMTn2eXZmL7xu2/ZVZeh9lw4tWo2xDduImlj
/9rNHEAND/nZFHjTAonURE1VsB3WqdAufsUbMadvrgIfaMhYLnHeIFN0BaPPb1Do
kh7IfaETgDge1SLSZy5k1VW/K1qVfxwRV+nYxGekFZCz
-----END ENCRYPTED PRIVATE KEY-----
```

Para extraer la clave pública del fichero utilizamos la siguiente orden
```bash
openssl pkey -in privateRSA.pem -pubout -out publicRSA.pem
```
y obtenemos la siguiente clave pública

```bash
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArw4swEraW1U47cSipXfG
0F9O6mGj/Q8wVdn4fW1cTJR5DZJdygh3IrGRlQETTRbzoqN8UiuOQTEcgLTef+vY
x7Meiuui2oSwhLlPMFEaZ5RAYXSB4V8zVtTY4QHNmEUMd5hY2PLtmDmzh/NAbGgL
6pZJRcRiaSuuK8vQCL/xHfO4fWPHs4dtlvbUplicCuFV+N0pU5FrdzPKo8DEx2Mq
xigg+3copXL1tp6FsX4vazFbGmiy9wIDoWXXUSbEuPuw50UbyqOC3FU/V6fh7BiW
F1ucrIKs8czJ8jdmFEdyf36AVwp6PgQ+ZvYS202wlT5T2RdlXbhGDphE1YJm9G1a
9wIDAQAB
-----END PUBLIC KEY-----
```

### Exportación del par de claves en formatos DER Y PEM

Para seleccionar el formato de exportación de las claves se utiliza la opción `-outform DER | PEM` del comando `openssl genpkey`. Los comandos de generación de claves quedarían así para el formato DER

```
$ openssl genpkey -algorithm RSA -aes256 -out privateRSA.der -outform DER -pkeyopt rsa_keygen_bits:2048
```
y para el formato PEM
```
$ openssl genpkey -algorithm RSA -aes256 -out privateRSA.pem -outform PEM -pkeyopt rsa_keygen_bits:2048
```

Para la conversión de las claves entre ambos formatos podemos utilizar el comando `pkey`, de la siguiente manera

```
$ openssl pkey -in privateRSA.pem -inform PEM -outform DER -out privateRSA.der
```
y viceversa

```
$ openssl pkey -in privateRSA.der -inform DER -outform PEM -out privateRSA.pem
```

Lo mismo se aplica para la conversión de la clave pública.

### Firma y comprobación de la firma

En primer lugar se crea el fichero `mensaje.txt` que contiene `Esto es un mensaje`, y se crea el resumen firmado con la clave privada que contiene el fichero `privateRSA_1.pem` (que es el fichero creado anteriormente).

```bash
$ openssl dgst -sha256 -sign privateRSA_1.pem -out resumen_firmado.sha mensaje.txt
```

y se verifica que el resumen es correcto con la siguiente instrucción

```bash
$ openssl dgst -sha256 -verify publicRSA_1.pem -signature resumen_firmado.sha mensaje.txt
```

### Creación dos claves DH y demostración de la equivalencia entre combinaciones.

Se generan los pares de claves

```
$ openssl genpkey -algorithm X25519 -out privateDH_1.pem
$ openssl pkey -in privateDH_1.pem -pubout -out publicDH_1.pem
$ openssl genpkey -algorithm X25519 -out privateDH_2.pem      
$ openssl pkey -in privateDH_2.pem -pubout -out publicDH_2.pem
```
y se deriva un valor utilizando la combinación `privateDH_1.pem`, `publicDH_2.pem`.
```bash
$ openssl pkeyutl -derive -inkey privateDH_1.pem -peerkey publicDH_2.pem -out secret1
```
y la combinación `privateDH_2.pem`, `publicDH_1.pem`.
```bash
$ openssl pkeyutl -derive -inkey privateDH_2.pem -peerkey publicDH_1.pem -out secret2
```
Se comprueba que, efectivamente, los valores generados son equivalentes
```bash
$ cmp secret1 secret2
```

## Cifrado asimétrico de documentos

En esta parte de la práctica se pide cifrar un documento de texto y enviarlo a un compañero junto con la clave cifrada con su clave pública RSA. 
Para cumplir la especificación, se crea el fichero de texto `mensaje.txt` y se cifra utilizando `aes-256`.

```bash
$ openssl enc -aes-256-cfb -in mensaje.txt -out mensaje.aes-256-cfb
```

El siguiente paso es cifrar la contraseña con la clave pública de mi compañero (Néstor Pérez Haro)

```
$ openssl pkeyutl -pubin -encrypt -in contraseña -out contraseña.cif -inkey clavepublicanes.pem
```

y por último firmar el resumen del archivo de texto con mi clave privada
```bash
$ openssl dgst -sha256 -sign myprivatekey.pem -out resumen-mensaje-firmado.sha256 mensaje.txt
```

Con estos tres archivos se puede enviar un mensaje cifrado que cumple con los principios de la seguridad en la comunicación: confidencialidad, integridad, autenticación y no repudio.

Así mismo, los archivos que he obtenido del intercambio de mensajes son los siguientes:

- `archivoaes-nes`, cifrado mediante aes-256-cbc

- `contra-nes.cif`, que es la contraseña del cifrado simétrico del arhivo anteriorç
  
- `firmanes.rsa`, que es el resumen firmado del documento cifrado.

Con estos datos es posible lo siguiente

  * Descifrar la contraseña simétrica contenida en `contra-nes.cif` utilizando mi clave privada

    ```bash
    $ openssl pkeyutl -decrypt -in contra-nes.cif -out contra-nes.txt -inkey myprivatekey.pem
    ```
  * Luego de esto, se utiliza la contraseña obtenida para descifrar el archivo `archivoaes`
    ```bash
    $ openssl enc -aes-256-cbc -d -in archivoaes-nes -out mensaje-nestor.txt -kfile contra-nes.txt
    ```
  * Por último debemos verificar la autenticidad del emisor con su firma
    ```
    $ openssl dgst -md5 -verify clavepublicanes.pem -signature firmanes.rsa mensaje.txt
    ```
