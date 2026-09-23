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
