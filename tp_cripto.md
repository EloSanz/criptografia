# Trabajo Práctico - Criptografía y Seguridad (Cryptohack)

Documento de resolución de desafíos de la categoría **General**.  
Para cada ejercicio se incluye la **flag** obtenida y una **breve explicación** clara paso a paso para que cualquier persona pueda reproducirlo sin necesidad de conocimientos previos.

---

# Categoría: Encoding

## 2. ASCII

### Flag
`crypto{ASCII_pr1nt4bl3}`

### Explicación
ASCII es un estándar de codificación donde a cada carácter imprimible o de control le corresponde un número entero entre 0 y 127.

Para resolver este desafío:
1. Nos entregan una lista de números enteros:
   `[99, 114, 121, 112, 116, 111, 123, 65, 83, 67, 73, 73, 95, 112, 114, 49, 110, 116, 52, 98, 108, 51, 125]`
2. Cada número representa el código ordinal en ASCII de una letra o símbolo. Por ejemplo, `99` corresponde a `'c'`, `114` a `'r'`, `121` a `'y'`, etc.
3. Para obtener el texto original, convertimos cada entero a su carácter correspondiente (usando una tabla ASCII o la función `chr()` en Python) y los concatenamos en orden, obteniendo la flag final: `crypto{ASCII_pr1nt4bl3}`.

---

## 3. Hex

### Flag
`crypto{You_will_be_working_with_hex_strings_a_lot}`

### Explicación
La codificación hexadecimal (base 16) utiliza los dígitos `0-9` y las letras `a-f`. Cada byte (8 bits, con valores de 0 a 255) se representa exactamente con dos dígitos hexadecimales.

Para resolver este desafío:
1. Nos entregan una cadena de caracteres en formato hexadecimal:
   `63727970746f7b596f755f77696c6c5f62655f776f726b696e675f776974685f6865785f737472696e67735f615f6c6f747d`
2. Se agrupan los caracteres de dos en dos, donde cada par representa el valor numérico en base 16 de un byte:
   - `63` en hex = `99` en decimal → `'c'` en ASCII
   - `72` en hex = `114` en decimal → `'r'` en ASCII
   - `79` en hex = `121` en decimal → `'y'` en ASCII
   - `70` en hex = `112` en decimal → `'p'` en ASCII
   - ... y así sucesivamente.
3. Al convertir cada par a su byte/carácter ASCII correspondiente (por ejemplo con `bytes.fromhex(...)` en Python o cualquier decodificador hex online), se reconstruye el texto original, revelando la flag: `crypto{You_will_be_working_with_hex_strings_a_lot}`.

---

## 4. Base64

### Flag
`crypto/Base+64+Encoding+is+Web+Safe/`

### Explicación
Base64 es un sistema de codificación que permite representar datos binarios mediante un conjunto de 64 caracteres ASCII imprimibles (`A-Z`, `a-z`, `0-9`, `+`, `/`). Mientras que un byte son 8 bits, cada carácter en Base64 representa 6 bits de información; por lo tanto, cada bloque de 3 bytes (24 bits) se traduce directamente en 4 caracteres Base64 (4 × 6 = 24 bits).

Para resolver este desafío:
1. Nos entregan una cadena en formato hexadecimal:  
   `72bca9b68fc16ac7beeb8f849dca1d8a783e8acf9679bf9269f7bf`
2. **Paso 1 (Hex a Bytes):** Decodificamos la cadena hexadecimal a su secuencia de bytes binarios puros (por ejemplo con `bytes.fromhex(...)` en Python). Esto produce 27 bytes en bruto.
3. **Paso 2 (Bytes a Base64):** Tomamos esos bytes binarios y los codificamos bajo el estándar Base64 (utilizando `base64.b64encode(...)` en Python o herramientas como CyberChef).
4. El resultado final en formato Base64 produce la flag solicitada: `crypto/Base+64+Encoding+is+Web+Safe/`.

---

## 5. Bytes and Big Integers

### Flag
`crypto{3nc0d1n6_4ll_7h3_w4y_d0wn}`

### Explicación
Muchos criptosistemas matemáticos (como RSA) operan exclusivamente sobre números enteros en lugar de cadenas de texto. Para operar con texto, una cadena se convierte primero a su representación de bytes (en ASCII/UTF-8), luego esos bytes se interpretan como un único número en base 16 (hexadecimal) y finalmente se representan como un número entero en base 10 (un "Big Integer").

Para resolver el proceso inverso:
1. Nos entregan un número entero decimal gigante:  
   `11515195063862318899931685488813747395775516287289682636499965282714637259206269`
2. **Paso 1 (Decimal a Hexadecimal / Bytes):** Convertimos ese entero en base 10 a sus bytes correspondientes en formato *Big-Endian* (el byte más significativo primero).
   - Esto se puede hacer en Python nativo con `n.to_bytes((n.bit_length() + 7) // 8, byteorder='big')`, o utilizando la biblioteca criptográfica `Crypto.Util.number.long_to_bytes(n)`.
   - Si se hace a mano: convertimos el número a hexadecimal con `hex(n)`, lo que da `0x63727970746f...`.
3. **Paso 2 (Bytes a Texto):** Decodificamos esos bytes directamente a caracteres ASCII legibles.
4. El resultado descodificado revela la flag: `crypto{3nc0d1n6_4ll_7h3_w4y_d0wn}`.

---

## 6. Encoding Challenge

### Flag
`crypto{3nc0d3_d3c0d3_3nc0d3}`

### Explicación
Este desafío evalúa la capacidad de automatizar la decodificación de mensajes en tiempo real comunicándose a través de un socket de red (`socket.cryptohack.org:13377`). El servidor envía un objeto JSON con el tipo de codificación (`type`) y el valor codificado (`encoded`), exigiendo responder con el texto descodificado en menos de un tiempo límite, a lo largo de 100 niveles consecutivos.

Para resolver este desafío:
1. **Conexión al socket:** Establecemos una conexión TCP hacia el host `socket.cryptohack.org` en el puerto `13377` (por ejemplo con sockets nativos de Python o la librería `pwntools`).
2. **Ciclo de recepción y respuesta (100 niveles):** En cada ronda se recibe una línea en formato JSON con la estructura: `{"type": "<tipo>", "encoded": <valor>}`.
3. **Decodificación condicional según el tipo recibido:**
   - **`base64`**: Se decodifica usando `base64.b64decode(val).decode('utf-8')`.
   - **`hex`**: Se decodifica convirtiendo la cadena hexadecimal a texto con `bytes.fromhex(val).decode('utf-8')`.
   - **`rot13`**: Se aplica el cifrado César con desplazamiento 13 con `codecs.decode(val, 'rot_13')`.
   - **`bigint`**: Se convierte el entero a su representación en bytes en orden Big-Endian y luego a texto con `n.to_bytes(...)`.
   - **`utf-8`**: Se recibe una lista de ordinales enteros y se convierte cada uno a su carácter correspondiente con `chr()`.
4. **Envío:** Se responde al socket con un JSON de la forma `{"decoded": "<texto_plano>"}`.
5. Al completar con éxito las 100 respuestas consecutivas, el servidor entrega un JSON final con la flag: `crypto{3nc0d3_d3c0d3_3nc0d3}`.

---

# Categoría: XOR

## 7. XOR Starter

### Flag
`crypto{aloha}`

### Explicación
La operación XOR (*Exclusive OR*, denotada como `⊕` o con el operador `^` en programación) es una operación lógica bit a bit que devuelve `1` cuando los bits son distintos y `0` cuando son iguales. Para aplicar XOR sobre una cadena de texto, cada carácter se transforma en su valor numérico ASCII/Unicode, se opera bit a bit contra la clave numérica, y el número resultante se vuelve a convertir en carácter.

Para resolver este desafío:
1. Nos dan la cadena `"label"` y nos piden realizar la operación XOR de cada carácter con el número entero `13`.
2. Convertimos cada carácter a su valor ASCII con `ord()`, le aplicamos `^ 13`, y convertimos el valor resultante de nuevo a carácter con `chr()`:
   - `'l'` (ASCII 108): `108 ^ 13 = 97` → `'a'`
   - `'a'` (ASCII 97): `97 ^ 13 = 108` → `'l'`
   - `'b'` (ASCII 98): `98 ^ 13 = 111` → `'o'`
   - `'e'` (ASCII 101): `101 ^ 13 = 104` → `'h'`
   - `'l'` (ASCII 108): `108 ^ 13 = 97` → `'a'`
3. Uniendo los caracteres obtenidos resulta la palabra `"aloha"`.
4. El formato solicitado es `crypto{new_string}`, por lo que la flag final es: `crypto{aloha}`.

---

## 8. XOR Properties

### Flag
`crypto{x0r_i5_ass0c1at1v3}`

### Explicación
La operación XOR posee cuatro propiedades fundamentales en el álgebra de Boole:
1. **Conmutativa:** $A \oplus B = B \oplus A$ (el orden de los factores no altera el resultado).
2. **Asociativa:** $A \oplus (B \oplus C) = (A \oplus B) \oplus C$ (se pueden agrupar en cualquier orden sin paréntesis).
3. **Elemento neutro (Identidad):** $A \oplus 0 = A$ (hacer XOR con 0 deja el valor intacto).
4. **Auto-inversa:** $A \oplus A = 0$ (cualquier valor operado contra sí mismo se anula a cero).

Para resolver este desafío nos dan los siguientes valores en hexadecimal:
- $K_1$
- $K_2 \oplus K_1$
- $K_2 \oplus K_3$
- $\text{CIPHERTEXT} = \text{FLAG} \oplus K_1 \oplus K_3 \oplus K_2$

Aplicando las propiedades anteriores:
1. Debido a la conmutatividad y asociatividad, el cifrado equivale a:
   $$\text{CIPHERTEXT} = \text{FLAG} \oplus (K_1 \oplus K_2 \oplus K_3)$$
2. Notamos que tenemos directamente $K_1$ y además la combinación $(K_2 \oplus K_3)$. Por lo tanto, podemos calcular la clave total combinada simplemente haciendo:
   $$\text{CLAVE\_TOTAL} = K_1 \oplus (K_2 \oplus K_3) = K_1 \oplus K_2 \oplus K_3$$
3. Finalmente, aprovechando la propiedad de auto-inversa ($X \oplus X = 0$), para despejar $\text{FLAG}$ basta con aplicar XOR entre el $\text{CIPHERTEXT}$ y la $\text{CLAVE\_TOTAL}$:
   $$\text{FLAG} = \text{CIPHERTEXT} \oplus \text{CLAVE\_TOTAL}$$
4. Decodificando las cadenas de hex a bytes, operando byte a byte y convirtiendo a texto ASCII, se obtiene la flag: `crypto{x0r_i5_ass0c1at1v3}`.

---

## 9. Favourite byte

### Flag
`crypto{0x10_15_my_f4v0ur173_by7e}`

### Explicación
En este reto, el texto original fue cifrado aplicando XOR a cada byte contra una única clave secreta de 1 byte (un valor entre 0 y 255). 

Dado que el espacio de búsqueda para 1 byte es sumamente reducido (solo $2^8 = 256$ posibles valores), este tipo de cifrado es vulnerable a un ataque de fuerza bruta instantáneo o a un análisis de texto conocido (*Known-Plaintext Attack*): sabemos que la flag siempre comienza con el prefijo `"crypto{"`.

Para resolver este desafío:
1. Nos entregan la cadena cifrada en formato hexadecimal:  
   `73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d`
2. **Método 1 (Fuerza Bruta):**
   - Decodificamos la cadena hexadecimal a sus bytes correspondientes.
   - Iteramos por cada posible valor de clave entre `0` y `255`.
   - Aplicamos XOR entre cada byte cifrado y la clave candidata.
   - Al probar la clave `16` (en hexadecimal `0x10`), el texto resultante se vuelve legible y contiene el formato de la flag.
3. **Método 2 (Texto conocido / Deducir la clave directamente):**
   - El primer byte del mensaje cifrado es `0x73`.
   - Sabemos que el primer carácter de la flag debe ser `'c'`, cuyo valor ASCII es `99` (`0x63`).
   - Por la propiedad auto-inversa de XOR: $\text{CLAVE} = \text{Cifrado}[0] \oplus \text{'c'} = 0x73 \oplus 0x63 = 0x10$ (es decir, el entero 16).
   - Al aplicar `0x10` al resto de los bytes, se descifra inmediatamente toda la flag.
4. El mensaje descifrado es: `crypto{0x10_15_my_f4v0ur173_by7e}`.

---

## 10. You either know, XOR you don't

### Flag
`crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`

### Explicación
Este desafío utiliza un cifrado XOR con clave repetida (*Repeating-key XOR* o cifrado de Vigenère aplicado a XOR). Cada byte del mensaje original se opera mediante XOR con el byte correspondiente de la clave cíclica:
$$C[i] = P[i] \oplus K[i \pmod L]$$
donde $L$ es la longitud de la clave.

A diferencia del desafío anterior, la clave no es de 1 solo byte, pero podemos atacarlo mediante un **Known-Plaintext Attack** (Ataque de texto plano conocido) gracias a que conocemos el formato inicial de la flag: `"crypto{"` (7 bytes) y su carácter de cierre `"}"`.

Para resolver este desafío:
1. Nos entregan el texto cifrado en formato hexadecimal:  
   `0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104`
2. **Paso 1 (Obtener la clave parcial):**  
   Sabemos que los primeros 7 bytes del texto plano ($P[0..6]$) corresponden a los caracteres `"crypto{"`. Aplicamos XOR entre los primeros 7 bytes cifrados y dicho prefijo:
   $$K[0..6] = C[0..6] \oplus \text{"crypto{"}$$
   Esto nos arroja exactamente: `b'myXORke'`.
3. **Paso 2 (Deducir el carácter restante):**  
   La secuencia `myXORke` claramente se perfila como la palabra en inglés `"myXORkey"` (de longitud 8 bytes).  
   *Comprobación adicional:* Además, el último byte cifrado `C[-1]` operado con el cierre `"}"` ($C[-1] \oplus \text{'}'}$) nos da exactamente `'y'`, confirmando que el byte final de la clave de 8 posiciones es `'y'`. Por ende, la clave completa es:
   $$K = \text{"myXORkey"}$$
4. **Paso 3 (Descifrar el mensaje completo):**  
   Aplicamos la operación XOR sobre todos los bytes del texto cifrado repitiendo la clave `"myXORkey"` periódicamente en ciclos de 8 bytes ($C[i] \oplus K[i \pmod 8]$).
5. El mensaje completamente descifrado revela la flag: `crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`.

---

## 11. Lemur XOR

### Flag
`crypto{X0Rly_n0t!}`

### Explicación
Este desafío demuestra una vulnerabilidad crítica cuando se reutiliza una misma clave secreta de flujo (*One-Time Pad* o *Stream Cipher Key*) para cifrar dos mensajes distintos (en este caso, dos imágenes: `lemur.png` y `flag.png`).

Sean:
- $I_1 = \text{Imagen del Lemur}$
- $I_2 = \text{Imagen con la Flag}$
- $K = \text{Clave secreta en bytes (del mismo tamaño que las imágenes)}$

Ambas imágenes fueron cifradas con la misma clave mediante XOR:
$$C_1 = I_1 \oplus K$$
$$C_2 = I_2 \oplus K$$

Por las propiedades conmutativa, asociativa y auto-inversa del XOR:
$$C_1 \oplus C_2 = (I_1 \oplus K) \oplus (I_2 \oplus K) = I_1 \oplus I_2 \oplus (K \oplus K) = I_1 \oplus I_2 \oplus 0 = I_1 \oplus I_2$$

La clave secreta $K$ se anula por completo, resultando en el XOR visual directo entre los píxeles de ambas imágenes ($I_1 \oplus I_2$). Como el fondo de la imagen de la flag es uniforme y el texto contrasta fuertemente, los caracteres de la flag quedan expuestos a simple vista sobre el lemur.

Para resolver este desafío:
1. Tomamos las dos imágenes provistas: `lemur_ed66878c338e662d3473f0d98eedbd0d.png` y `flag_7ae18c704272532658c10b5faad06d74.png`.
2. Extraemos los valores de color RGB de cada píxel de ambas imágenes (teniendo en cuenta la decodificación de scanlines PNG o utilizando librerías de imagen como PIL/Pillow o herramientas gráficas como GIMP/CyberChef).
3. Aplicamos la operación XOR componente a componente para cada píxel:
   $$\text{Rojo}_{\text{resultado}} = \text{Rojo}_1 \oplus \text{Rojo}_2$$
   $$\text{Verde}_{\text{resultado}} = \text{Verde}_1 \oplus \text{Verde}_2$$
   $$\text{Azul}_{\text{resultado}} = \text{Azul}_1 \oplus \text{Azul}_2$$
4. Al generar y visualizar la imagen resultante, se distingue claramente sobre la silueta del lemur el texto de la flag: `crypto{X0Rly_n0t!}`.

---

# Categoría: Mathematics

## Greatest Common Divisor (GCD)

### Flag / Respuesta
`1512`

### Explicación
El Máximo Común Divisor (GCD o MCD) entre dos números enteros positivos $a$ y $b$ es el mayor número entero que divide exactamente a ambos sin dejar resto.

Para calcular eficientemente el GCD de números grandes se utiliza el **Algoritmo de Euclides**, el cual se fundamenta en la propiedad:
$$\gcd(a, b) = \gcd(b, a \pmod b)$$
repitiendo divisiones sucesivas hasta que el resto sea igual a 0, momento en el cual el último divisor no nulo es el GCD.

Para resolver este desafío:
1. Nos solicitan calcular el $\gcd(a, b)$ para:
   - $a = 66528$
   - $b = 52920$
2. **Aplicamos el Algoritmo de Euclides paso a paso:**
   - $66528 = 1 \times 52920 + 13608$ $\implies \gcd(66528, 52920) = \gcd(52920, 13608)$
   - $52920 = 3 \times 13608 + 12096$ $\implies \gcd(52920, 13608) = \gcd(13608, 12096)$
   - $13608 = 1 \times 12096 + 1512$ $\implies \gcd(13608, 12096) = \gcd(12096, 1512)$
   - $12096 = 8 \times 1512 + 0$ $\implies$ El resto es 0.
3. El último resto no nulo obtenido es **`1512`**.
4. La respuesta solicitada a ingresar en el campo es: `1512`.

---

## Extended GCD

### Flag / Respuesta
`-8404`

### Explicación
La Identidad de Bézout establece que para cualesquiera dos enteros $a$ y $b$ con máximo común divisor $\gcd(a, b)$, existen enteros $u$ y $v$ (llamados coeficientes de Bézout) tales que:
$$a \cdot u + b \cdot v = \gcd(a, b)$$

El **Algoritmo de Euclides Extendido** permite calcular no solo el $\gcd(a, b)$, sino también dichos coeficientes $u$ y $v$, lo cual es fundamental en criptografía para encontrar el inverso multiplicativo modular (por ejemplo, para descifrar en RSA).

Para resolver este desafío:
1. Nos dan dos números primos:
   - $p = 26513$
   - $q = 32321$
2. Como ambos son números primos distintos, son coprimos entre sí, por lo que:
   $$\gcd(p, q) = 1$$
3. Buscamos enteros $u$ y $v$ tales que:
   $$26513 \cdot u + 32321 \cdot v = 1$$
4. **Ejecución del Algoritmo de Euclides Extendido:**
   - Divisiones sucesivas:
     - $32321 = 1 \times 26513 + 5808$
     - $26513 = 4 \times 5808 + 3281$
     - $5808 = 1 \times 3281 + 2527$
     - $3281 = 1 \times 2527 + 754$
     - $2527 = 3 \times 754 + 265$
     - $754 = 2 \times 265 + 224$
     - $265 = 1 \times 224 + 41$
     - $224 = 5 \times 41 + 19$
     - $41 = 2 \times 19 + 3$
     - $19 = 6 \times 3 + 1$
   - Despejando hacia atrás los restos en combinación lineal de $p$ y $q$:
     - Se obtiene: $u = 10245$ y $v = -8404$.
   - **Comprobación:**  
     $$26513 \times 10245 + 32321 \times (-8404) = 271625685 - 271625684 = 1$$
5. La consigna pide ingresar como flag el menor entre $u$ y $v$ ($\min(u, v)$):
   $$\min(10245, -8404) = -8404$$
6. La respuesta a ingresar es: `-8404`.

---

## Modular Arithmetic 1

### Flag / Respuesta
`4`

### Explicación
La aritmética modular estudia las relaciones de congruencia entre números enteros. Decimos que dos enteros $a$ y $b$ son congruentes módulo $m$ ($a \equiv b \pmod m$) si su diferencia $(a - b)$ es un múltiplo exacto de $m$, lo cual equivale a decir que ambos dejan el mismo resto al dividirlos por $m$.

Para resolver este desafío debemos calcular los restos $x$ e $y$ en el rango canónico $[0, m-1]$:
1. **Calcular $x$:**
   $$11 \equiv x \pmod 6$$
   - Dividimos $11$ entre $6$: $11 = 1 \times 6 + 5$.
   - El resto de la división es $5$.
   - Por lo tanto, $x = 5$.

2. **Calcular $y$:**
   $$8146798528947 \equiv y \pmod{17}$$
   - Dividimos $8146798528947$ entre $17$:
     $$8146798528947 = 479223442879 \times 17 + 4$$
   - El resto de la división es $4$.
   - Por lo tanto, $y = 4$.

3. La consigna indica que la solución es el menor de los dos enteros obtenidos ($\min(x, y)$):
   $$\min(5, 4) = 4$$
4. La respuesta a ingresar en el campo es: `4`.

---

## Modular Arithmetic 2

### Flag / Respuesta
`1`

### Explicación
Cuando el módulo $p$ es un número primo, el conjunto de enteros $\{0, 1, \dots, p-1\}$ bajo la suma y la multiplicación modular forma un cuerpo finito (*finite field*), denotado como $\mathbb{F}_p$. En este espacio matemático aplica una de las propiedades más importantes de la teoría de números: el **Pequeño Teorema de Fermat** (*Fermat's Little Theorem*).

El teorema establece que si $p$ es un número primo y $a$ es un entero no divisible por $p$ ($\gcd(a, p) = 1$):
$$a^{p-1} \equiv 1 \pmod p$$
Y equivalentemente:
$$a^p \equiv a \pmod p$$

Para resolver este desafío:
1. Nos dan:
   - $p = 65537$ (un número primo muy conocido en criptografía, el cuarto primo de Fermat $F_4 = 2^{2^4} + 1$).
   - La base $a = 273246787654$, la cual no es múltiplo de $p$ ($\gcd(a, p) = 1$).
2. Nos piden calcular:
   $$273246787654^{65536} \pmod{65537}$$
3. Observamos la estructura de los exponentes:
   - El exponente es exactamente $65536 = 65537 - 1 = p - 1$.
4. Por aplicación directa del **Pequeño Teorema de Fermat**:
   $$a^{p-1} \equiv 1 \pmod p \implies 273246787654^{65537-1} \equiv 1 \pmod{65537}$$
5. Por lo tanto, no se requiere ninguna calculadora: el resultado es directamente **`1`**.

---

## Modular Inverting

### Flag / Respuesta
`9`

### Explicación
Dado un elemento $g$ en un cuerpo finito $\mathbb{F}_p$ (donde $p$ es primo), su **inverso multiplicativo modular** es el único elemento $d = g^{-1}$ tal que:
$$g \cdot d \equiv 1 \pmod p$$

Podemos calcularlo de dos maneras elegantes:

1. **Método 1 (Mediante el Pequeño Teorema de Fermat):**
   - Sabemos que $g^{p-1} \equiv 1 \pmod p$.
   - Reescribiendo la potencia:
     $$g \cdot g^{p-2} \equiv 1 \pmod p$$
   - Por definición, esto implica directamente que el inverso es:
     $$d \equiv g^{p-2} \pmod p$$
   - En este desafío tenemos $g = 3$ y $p = 13$:
     $$d \equiv 3^{13-2} \equiv 3^{11} \pmod{13}$$
     - $3^3 = 27 \equiv 1 \pmod{13}$
     - $3^{11} = (3^3)^3 \times 3^2 \equiv 1^3 \times 9 \equiv 9 \pmod{13}$
   - Obtenemos $d = 9$.

2. **Método 2 (Búsqueda directa / Aritmética modular básica):**
   - Buscamos un número $d$ tal que $3 \cdot d = 13k + 1$:
     - Si $k = 1$: $13(1) + 1 = 14$ (no es divisible por 3).
     - Si $k = 2$: $13(2) + 1 = 27$.
     - $3 \cdot d = 27 \implies d = 9$.
   - **Comprobación:**  
     $$3 \times 9 = 27 = 2 \times 13 + 1 \equiv 1 \pmod{13}$$

3. La respuesta a ingresar en el campo es: `9`.

---

## Privacy-Enhanced Mail (PEM)

### Flag / Respuesta
`15682700288056331364787171045819973654991149949197959929860861228180021707316851924456205543665565810892674190059831330231436970914474774562714945620519144389785158908994181951348846017432506464163564960993784254153395406799101314760033445065193429592512349952020982932218524462341002102063435489318813316464511621736943938440710470694912336237680219746204595128959161800595216366237538296447335375818871952520026993102148328897083547184286493241191505953601668858941129790966909236941127851370202421135897091086763569884760099112291072056970636380417349019579768748054760104838790424708988260443926906673795975104689`

### Explicación
**PEM (Privacy-Enhanced Mail)** es un formato estándar utilizado para almacenar y transmitir claves criptográficas y certificados en texto plano ASCII. Estructuralmente:
1. Las estructuras de datos criptográficas se definen mediante la sintaxis **ASN.1** (en el caso de una clave privada RSA, según el estándar PKCS#1: módulo $n$, exponente público $e$, exponente privado $d$, primos $p$ y $q$, etc.).
2. Esos datos se serializan en binario utilizando las reglas de codificación **DER** (*Distinguished Encoding Rules*).
3. Para facilitar su transmisión en entornos que sólo admiten texto (como correos o páginas web), los datos en DER se codifican en **Base64** y se envuelven entre encabezados y pies delimitadores:
   ```text
   -----BEGIN RSA PRIVATE KEY-----
   ... (datos en Base64) ...
   -----END RSA PRIVATE KEY-----
   ```

Para resolver este desafío:
1. Nos dan el archivo `privacy_enhanced_mail.pem` que contiene una clave privada RSA de 2048 bits.
2. La consigna solicita extraer el valor del **exponente privado $d$** como un número entero en base decimal.
3. Podemos extraerlo directamente utilizando la herramienta estándar `openssl`:
   ```bash
   openssl rsa -in privacy_enhanced_mail.pem -text -noout
   ```
   O mediante la librería `cryptography` / `pycryptodome` en Python:
   ```python
   from Crypto.PublicKey import RSA
   key = RSA.importKey(open('privacy_enhanced_mail.pem').read())
   print(key.d)
   ```
4. El valor decimal obtenido de $d$ es:
   `15682700288056331364787171045819973654991149949197959929860861228180021707316851924456205543665565810892674190059831330231436970914474774562714945620519144389785158908994181951348846017432506464163564960993784254153395406799101314760033445065193429592512349952020982932218524462341002102063435489318813316464511621736943938440710470694912336237680219746204595128959161800595216366237538296447335375818871952520026993102148328897083547184286493241191505953601668858941129790966909236941127851370202421135897091086763569884760099112291072056970636380417349019579768748054760104838790424708988260443926906673795975104689`

---

## CERTainly not

### Flag / Respuesta
`22825373692019530804306212864609512775374171823993708516509897631547513634635856375624003737068034549047677999310941837454378829351398302382629658264078775456838626207507725494030600516872852306191255492926495965536379271875310457319107936020730050476235278671528265817571433919561175665096171189758406136453987966255236963782666066962654678464950075923060327358691356632908606498231755963567382339010985222623205586923466405809217426670333410014429905146941652293366212903733630083016398810887356019977409467374742266276267137547021576874204809506045914964491063393800499167416471949021995447722415959979785959569497`

### Explicación
Un certificado digital SSL/TLS conforme al estándar **X.509** asocia una clave pública criptográfica con la identidad de una organización o dominio. A diferencia de los archivos PEM (que están codificados en Base64 con encabezados ASCII), el formato **DER** es la representación binaria pura de las estructuras de datos **ASN.1**.

Para resolver este desafío:
1. Nos entregan el archivo binario `2048b-rsa-example-cert.der`.
2. La consigna solicita extraer el **módulo $n$** de la clave pública RSA contenida en el certificado y presentarlo como un número entero en base decimal.
3. Podemos inspeccionar y extraer el módulo del certificado en formato DER directamente con OpenSSL:
   ```bash
   openssl x509 -inform der -in 2048b-rsa-example-cert.der -modulus -noout
   ```
   Esto nos devuelve el módulo en formato hexadecimal:
   `Modulus=B4CFD15E3329EC0BCFAE...`
4. Al convertir esa secuencia hexadecimal de 2048 bits a un número entero decimal (base 10), obtenemos la respuesta requerida:
   `22825373692019530804306212864609512775374171823993708516509897631547513634635856375624003737068034549047677999310941837454378829351398302382629658264078775456838626207507725494030600516872852306191255492926495965536379271875310457319107936020730050476235278671528265817571433919561175665096171189758406136453987966255236963782666066962654678464950075923060327358691356632908606498231755963567382339010985222623205586923466405809217426670333410014429905146941652293366212903733630083016398810887356019977409467374742266276267137547021576874204809506045914964491063393800499167416471949021995447722415959979785959569497`

---

## SSH Keys

### Flag / Respuesta
`3931406272922523448436194599820093016241472658151801552845094518579507815990600459669259603645261532927611152984942840889898756532060894857045175300145765800633499005451738872081381267004069865557395638550041114206143085403607234109293286336393552756893984605214352988705258638979454736514997314223669075900783806715398880310695945945147755132919037973889075191785977797861557228678159538882153544717797100401096435062359474129755625453831882490603560134477043235433202708948615234536984715872113343812760102812323180391544496030163653046931414723851374554873036582282389904838597668286543337426581680817796038711228401443244655162199302352017964997866677317161014083116730535875521286631858102768961098851209400973899393964931605067856005410998631842673030901078008408649613538143799959803685041566964514489809211962984534322348394428010908984318940411698961150731204316670646676976361958828528229837610795843145048243492909`

### Explicación
Las claves públicas de OpenSSH (las que habitualmente residen en `~/.ssh/authorized_keys` o con extensión `.pub`) utilizan un formato de línea simple compuesto por tres campos separados por espacios:
```text
<tipo_clave> <datos_codificados_en_base64> <comentario>
```
Por ejemplo: `ssh-rsa AAAAB3NzaC1yc2E... bschneier@facts`

El bloque central en Base64 almacena internamente una estructura binaria definida por el protocolo SSH (RFC 4251 y RFC 4253). En el caso de una clave RSA (`ssh-rsa`), los datos binarios están empaquetados en campos consecutivos con prefijo de longitud de 4 bytes en orden Big-Endian:
1. Longitud (4 bytes) + String del identificador del algoritmo (`"ssh-rsa"`).
2. Longitud (4 bytes) + Entero de precisión múltiple (*mpint*) del exponente público $e$ (comúnmente $65537$).
3. Longitud (4 bytes) + Entero de precisión múltiple (*mpint*) del **módulo $n$**.

Para resolver este desafío:
1. Tomamos el archivo `bruce_rsa.pub` descargado.
2. Extraemos la cadena Base64 central y la decodificamos a sus bytes binarios crudos.
3. Desempaquetamos la estructura leyendo secuencialmente los campos con prefijo de 4 bytes:
   - Primer campo: `"ssh-rsa"` (7 bytes).
   - Segundo campo: exponente $e = 65537$ (3 bytes: `0x010001`).
   - Tercer campo: el módulo $n$ (en bytes Big-Endian).
4. Convertimos los bytes del módulo $n$ a un número entero en base decimal (base 10) usando `int.from_bytes(n_bytes, byteorder='big')`.
5. El valor decimal obtenido de $n$ es la respuesta solicitada:
   `3931406272922523448436194599820093016241472658151801552845094518579507815990600459669259603645261532927611152984942840889898756532060894857045175300145765800633499005451738872081381267004069865557395638550041114206143085403607234109293286336393552756893984605214352988705258638979454736514997314223669075900783806715398880310695945945147755132919037973889075191785977797861557228678159538882153544717797100401096435062359474129755625453831882490603560134477043235433202708948615234536984715872113343812760102812323180391544496030163653046931414723851374554873036582282389904838597668286543337426581680817796038711228401443244655162199302352017964997866677317161014083116730535875521286631858102768961098851209400973899393964931605067856005410998631842673030901078008408649613538143799959803685041566964514489809211962984534322348394428010908984318940411698961150731204316670646676976361958828528229837610795843145048243492909`

---

## Transparency

### Flag
`crypto{thx_redpwn_for_inspiration}`

### Explicación
**Certificate Transparency (CT)** es un registro público, abierto y auditable que almacena todos los certificados TLS emitidos por las Autoridades Certificadoras (CAs). Fue creado para detectar certificados emitidos fraudulentamente o por error (por ejemplo, casos históricos donde CAs comprometidas emitieron certificados falsos para dominios como Google o Microsoft). Servicios como [crt.sh](https://crt.sh) permiten consultar libremente estos registros.

Para resolver este desafío:
1. Nos entregan el archivo `transparency.pem` que contiene una clave pública RSA.
2. La consigna nos pide encontrar qué subdominio de `cryptohack.org` utiliza los parámetros de dicha clave pública en su certificado TLS, y visitarlo para obtener la flag.
3. Para buscar todos los certificados emitidos públicamente para los subdominios de CryptoHack, accedemos a la base de datos de logs de Certificate Transparency en **[crt.sh](https://crt.sh)**:
   - En el buscador escribimos `%.cryptohack.org` (el símbolo `%` actúa como comodín para encontrar cualquier subdominio).
   - O consultamos directamente su API web: `https://crt.sh/?q=%.cryptohack.org&output=json`.
4. Al revisar los subdominios registrados, sobresale inmediatamente uno creado específicamente para este reto:  
   `thetransparencyflagishere.cryptohack.org`
5. *(Opcional / Verificación técnica)*: Si inspeccionamos el certificado que sirve dicho sitio web, podemos comprobar que su clave pública RSA coincide de forma exacta con la del archivo `transparency.pem`.
6. Finalmente, abrimos en el navegador o realizamos una petición con `curl`:
   ```bash
   curl https://thetransparencyflagishere.cryptohack.org
   ```
7. El servidor web responde directamente con la flag: `crypto{thx_redpwn_for_inspiration}`.

