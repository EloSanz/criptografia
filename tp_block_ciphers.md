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
