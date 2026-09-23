# Trabajo Práctico - Criptografía y Seguridad (Cryptohack)

Documento de resolución de desafíos de la categoría **Block Ciphers** (Sección **How AES works**).  
Para cada ejercicio se incluye la **flag** encontrada y una **breve explicación** didáctica paso a paso para que cualquier persona pueda comprenderlo y reproducirlo sin conocimientos previos.

---

# Sección: How AES works

## 1. Keyed Permutations

### Flag
`crypto{bijection}`

### Explicación
Los cifradores de bloque (*block ciphers*), como AES (Advanced Encryption Standard), operan dividiendo la información en bloques de longitud fija (en AES, bloques de 128 bits o 16 bytes) y aplicando sobre ellos una transformación dependiente de una clave secreta.

A esta transformación se la denomina **permutación con clave** (*keyed permutation*), lo que significa que para una clave determinada:
1. Cada posible bloque de entrada de texto plano se mapea a un único bloque de salida de texto cifrado.
2. Usando la misma clave, el proceso se puede invertir exactamente en reversa, mapeando el bloque de salida de vuelta a su bloque original de entrada.

Para que el descifrado sea siempre unívoco y no ambiguo, debe existir una correspondencia uno a uno (*one-to-one correspondence*) estricta entre todos los bloques del dominio de entrada y los del codominio de salida:
- **Inyectiva:** No existen dos bloques de entrada distintos que generen el mismo bloque de salida cifrado.
- **Sobreyectiva:** Cada posible bloque cifrado proviene de algún bloque de entrada.

En matemáticas, una función que cumple simultáneamente con ser inyectiva y sobreyectiva (es decir, una correspondencia biunívoca o uno a uno) recibe el nombre de **biyección** (**`bijection`**).

Siguiendo el formato solicitado por el reto (`crypto{term}`), la flag es: `crypto{bijection}`.

---

## 2. Resisting Bruteforce

### Flag
`crypto{biclique}`

### Explicación
Un cifrador de bloques se considera seguro si un atacante no puede distinguir su salida de una permutación puramente aleatoria de bits, y si no existe ningún método para revertir la permutación que sea más rápido que probar exhaustivamente todas las claves posibles (fuerza bruta).

En el ámbito académico y criptográfico, un algoritmo se califica formalmente como "roto" (*broken*) si se descubre un ataque que requiera menos operaciones computacionales que la fuerza bruta, aun si dicho ataque sigue siendo completamente inviable en la práctica.

Para AES-128:
- El espacio de claves es de $2^{128}$ posibilidades (un número tan astronómico que toda la red de minería de Bitcoin tardaría más de cien veces la edad del universo en recorrerlo por fuerza bruta).
- En 2011, los investigadores Andrey Bogdanov, Dmitry Khovratovich y Christian Rechberger publicaron el ataque **Biclique** (*biclique attack*), una variante sofisticada de los ataques *Meet-in-the-Middle* aplicada a todas las rondas de AES.
- Este ataque reduce la complejidad computacional teórica de romper AES-128 de $2^{128}$ a aproximadamente $2^{126.1}$ operaciones (un factor de mejora de apenas $\approx 3.79$).
- Aunque representa el **mejor ataque de clave única conocido contra AES** (*best single-key attack against AES*), la reducción es tan marginal que AES continúa siendo completamente seguro e inquebrantable en la práctica.

Por lo tanto, el término buscado es **`biclique`**, y la flag solicitada es: `crypto{biclique}`.

---

## 3. Structure of AES

### Flag
`crypto{inmatrix}`

### Explicación
En AES-128, los datos no se procesan como un flujo lineal de 16 bytes, sino que se organizan internamente en una **matriz de estado** (*state matrix*) de 4×4 bytes. Sobre esta matriz se aplican 10 rondas de transformaciones algebraicas sucesivas:
1. **KeyExpansion:** Se derivan 11 subclaves de ronda (*round keys*) de 128 bits cada una a partir de la clave original.
2. **Initial AddRoundKey:** Se realiza un XOR bit a bit entre los bytes del bloque de texto plano y la primera subclave.
3. **9 Rondas intermedias:** Cada una compuesta por:
   - **`SubBytes`**: Sustitución no lineal de cada byte mediante una tabla fija (*S-box*).
   - **`ShiftRows`**: Desplazamiento circular cíclico hacia la izquierda de las últimas tres filas de la matriz.
   - **`MixColumns`**: Multiplicación matricial sobre el campo de Galois $\text{GF}(2^8)$ para difundir los bytes de cada columna.
   - **`AddRoundKey`**: XOR con la subclave de la ronda actual.
4. **Ronda final (Round 10):** Idéntica a las rondas intermedias pero omitiendo la fase `MixColumns`.

Para resolver este desafío:
1. El script provisto incluye la función `bytes2matrix(text)`, que divide una secuencia de 16 bytes en filas de 4 elementos para formar la matriz 4×4.
2. Se nos entrega una matriz de estado numérica de 4×4:
   ```python
   matrix = [
       [99, 114, 121, 112],
       [116, 111, 123, 105],
       [110, 109, 97, 116],
       [114, 105, 120, 125],
   ]
   ```
3. Implementamos la función inversa `matrix2bytes(matrix)` recorriendo las filas de la matriz y aplanando los valores en un único array de bytes o string:
   ```python
   def matrix2bytes(matrix):
       return bytes([b for row in matrix for b in row])
   ```
4. Al convertir los 16 enteros ordinales a caracteres ASCII:
   - Fila 0: `[99, 114, 121, 112]` → `'c'`, `'r'`, `'y'`, `'p'`
   - Fila 1: `[116, 111, 123, 105]` → `'t'`, `'o'`, `'{'`, `'i'`
   - Fila 2: `[110, 109, 97, 116]` → `'n'`, `'m'`, `'a'`, `'t'`
   - Fila 3: `[114, 105, 120, 125]` → `'r'`, `'i'`, `'x'`, `'}'`
5. Concatenando todos los bytes se reconstruye el texto plano original: `crypto{inmatrix}`.

---

## 4. Round Keys

### Flag
`crypto{r0undk3y}`

### Explicación
El paso **`AddRoundKey`** es el único punto de todo el algoritmo AES donde la clave secreta se mezcla directamente con el estado (*state*). Sin este paso, las transformaciones de AES serían simplemente una permutación fija sin secreto alguno; `AddRoundKey` es lo que convierte a AES en una verdadera "permutación con clave" (*keyed permutation*).

Matemáticamente, la operación es muy directa:
- Se toma la matriz de estado actual $S$ de 4×4 bytes.
- Se toma la subclave de ronda $K$ de 4×4 bytes (derivada mediante el *Key Schedule* de AES).
- Se aplica la operación **XOR bit a bit** celda por celda entre ambos:
  $$S'_{i,j} = S_{i,j} \oplus K_{i,j} \quad \text{para } i,j \in \{0, 1, 2, 3\}$$

Para resolver este desafío:
1. Nos entregan dos matrices 4×4 de bytes: la matriz de estado `state` y la matriz de clave de ronda `round_key`.
2. Completamos la función `add_round_key(s, k)` realizando la operación XOR celda por celda:
   ```python
   def add_round_key(s, k):
       return [[s[i][j] ^ k[i][j] for j in range(4)] for i in range(4)]
   ```
3. Calculamos la matriz resultante celda a celda:
   - Fila 0: `[206^173, 243^129, 61^68, 34^82]` = `[99, 114, 121, 112]` → `"cryp"`
   - Fila 1: `[171^223, 11^100, 93^38, 31^109]` = `[116, 111, 123, 114]` → `"to{r"`
   - Fila 2: `[16^32, 200^189, 91^53, 108^8]` = `[48, 117, 110, 100]` → `"0und"`
   - Fila 3: `[150^253, 3^48, 194^187, 51^78]` = `[107, 51, 121, 125]` → `"k3y}"`
4. Convertimos la matriz resultante a una secuencia de bytes con la función `matrix2bytes()` desarrollada en el ejercicio anterior.
5. El texto resultante revela la flag: `crypto{r0undk3y}`.

---

## 5. Confusion through Substitution

### Flag
`crypto{l1n34rly}`

### Explicación
Claude Shannon identificó dos pilares fundamentales para el diseño de cualquier cifrador seguro: **confusión** y **difusión**. La **confusión** busca hacer que la relación entre la clave secreta y el texto cifrado sea lo más compleja y no lineal posible.

En AES, la confusión se implementa en el paso **`SubBytes`**:
- Cada byte de la matriz de estado (un valor entre `0x00` y `0xFF`) se sustituye por otro byte según una tabla precalculada de 16×16 conocida como la **S-box** (*Substitution box*).
- La S-box de AES está rigurosamente diseñada mediante la inversión en el campo de Galois $\text{GF}(2^8)$ combinada con una transformación afín, lo que le otorga una alta resistencia contra el criptoanálisis lineal y diferencial.

Para resolver este desafío:
1. Nos entregan una matriz de estado 4×4 resultante de haberle aplicado la transformación `SubBytes`.
2. Para revertir esta sustitución y recuperar el estado original, debemos aplicar la **S-box inversa** (`inv_s_box`):
   ```python
   def sub_bytes(s, sbox=inv_s_box):
       return [[sbox[s[i][j]] for j in range(4)] for i in range(4)]
   ```
3. Sustituimos cada byte $s_{i,j}$ por su entrada correspondiente `inv_s_box[s[i][j]]`.
4. Convertimos la matriz resultante a bytes mediante `matrix2bytes()`.
5. El texto plano recuperado contiene la flag: `crypto{l1n34rly}`.

---

## 6. Diffusion through Permutation

### Flag
`crypto{d1ffUs3R}`

### Explicación
El segundo principio de Shannon es la **difusión**: si cambiamos un solo bit del texto plano o de la clave, aproximadamente la mitad de los bits del texto cifrado deberían cambiar de forma impredecible (*efecto avalancha*). Mientras que `SubBytes` introduce no-linealidad localmente a nivel de cada byte individual, no mezcla información entre distintas posiciones del bloque. La difusión en AES se logra combinando dos pasos:

1. **`ShiftRows` (Permutación a nivel de filas):**
   - La fila 0 no se desplaza.
   - La fila 1 se desplaza cíclicamente 1 posición a la izquierda.
   - La fila 2 se desplaza cíclicamente 2 posiciones a la izquierda.
   - La fila 3 se desplaza cíclicamente 3 posiciones a la izquierda.
   - Para revertirlo (**`inv_shift_rows`**), simplemente desplazamos las filas en sentido inverso (hacia la derecha).

2. **`MixColumns` (Transformación lineal a nivel de columnas):**
   - Trata cada columna de 4 bytes como un polinomio sobre $\text{GF}(2^8)$ y la multiplica por una matriz fija módulo $x^4 + 1$.
   - Para revertirlo (**`inv_mix_columns`**), se multiplica cada columna por la matriz inversa correspondiente.

Para resolver este desafío:
1. Nos entregan una matriz de estado tras la fase de difusión (`ShiftRows` seguido de `MixColumns`).
2. Para revertir el proceso, debemos aplicar las transformaciones inversas en el **orden estrictamente opuesto**:
   - Primero se aplica **`inv_mix_columns(matrix)`**.
   - Luego se aplica **`inv_shift_rows(matrix)`**.
3. Convertimos la matriz resultante a bytes con `matrix2bytes()`.
4. El mensaje descifrado revela la flag: `crypto{d1ffUs3R}`.

---

## 7. Bringing It All Together

### Flag
`crypto{MYAES128}`

### Explicación
Este desafío integra todos los componentes vistos a lo largo del módulo en una rutina completa de **descifrado AES-128**:
1. **Derivación de claves:** Se ejecuta el *Key Schedule* para generar las 11 subclaves de ronda ($K_0, K_1, \dots, K_{10}$) a partir de la clave maestra de 128 bits.
2. **Inversión del flujo de AES:** Como el descifrado debe desandar el camino exacto del cifrado en reversa, las rondas y operaciones se aplican en orden inverso:
   - **Ronda inicial de descifrado:** Se realiza `AddRoundKey` con la última subclave ($K_{10}$).
   - Luego se aplica `inv_shift_rows` e `inv_sub_bytes`.
   - **9 Rondas principales (de la ronda 9 a la 1):**
     1. `AddRoundKey` con la subclave $K_r$.
     2. `inv_mix_columns`.
     3. `inv_shift_rows`.
     4. `inv_sub_bytes`.
   - **Ronda final de descifrado:** Se realiza `AddRoundKey` con la primera subclave ($K_0$).
3. Al aplicar esta rutina completa sobre el bloque de texto cifrado (*ciphertext*) provisto en el reto, los 16 bytes resultantes se descifran en el texto plano original.
4. El texto descifrado revela la flag final de la sección: `crypto{MYAES128}`.
