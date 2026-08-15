# Sesión 2: Complementos, enteros con signo y aritmética binaria

## Objetivos de aprendizaje

Al finalizar esta sesión serás capaz de:

- Calcular el complemento a la base y a la base disminuida de un número en cualquier base.
- Representar enteros con signo en magnitud-signo, complemento a 1 y complemento a 2.
- Sumar y restar enteros con signo en binario, detectando desbordamiento (overflow).
- Representar y sumar números en BCD (Decimal Codificado en Binario).

## 1. Complemento a la base y a la base disminuida

Dado un número *N* de *n* dígitos en base *b*:

- **Complemento a la base disminuida** (para binario: complemento a 1): se calcula como `(b^n − 1) − N`. En binario equivale a **invertir todos los bits**.
- **Complemento a la base** (para binario: complemento a 2): se calcula como `b^n − N`, o de forma equivalente, **complemento a la base disminuida + 1**.

### ¿Para qué sirven?

Los complementos existen principalmente para poder **restar usando solo el circuito de sumar**, y para **representar números negativos** sin necesitar un circuito aparte que interprete el signo. En hardware es mucho más simple diseñar un sumador que un sumador-y-restador; si se convierte la resta `A − B` en una suma `A + complemento(B)`, el mismo circuito sirve para ambas operaciones.

- El **complemento a la base disminuida** (complemento a 1) fue el primer intento histórico: es fácil de calcular (solo invertir bits), pero tiene dos defectos que lo hicieron poco práctico: existen dos representaciones del cero (+0 y −0), y al sumar hay que aplicar una corrección extra (el llamado *acarreo circular* o *end-around carry*) para que el resultado salga correcto.
- El **complemento a la base** (complemento a 2) resuelve ambos problemas: solo hay un cero, y la suma funciona de forma directa sin ningún ajuste posterior. Por eso es el esquema que usa prácticamente todo el hardware moderno para representar enteros con signo (ver sección 2).

En resumen, no es solo un procedimiento matemático: es la razón por la que una ALU puede sumar y restar con el mismo circuito, y por la que los números negativos se representan como se representan en cualquier procesador actual.

**Ejemplo (8 bits):** N = 00010110₂ (22 decimal)

```
Complemento a 1 (invertir bits):     11101001
Complemento a 2 (complemento a 1 +1): 11101010
```

## 2. Representación de enteros con signo

Con *n* bits existen tres formas clásicas de representar enteros con signo:

| Esquema | Bit de signo | Rango (n=8) | Ceros |
|---|---|---|---|
| Magnitud y signo | 1 = negativo | −127 a +127 | +0 y −0 |
| Complemento a 1 | 1 = negativo | −127 a +127 | +0 y −0 |
| Complemento a 2 | 1 = negativo | −128 a +127 | un solo 0 |

El **complemento a 2** es el esquema usado prácticamente en todo hardware moderno porque tiene un único cero y permite sumar y restar con el mismo circuito.

### ¿Cómo representa un número negativo?

En binario sin signo, cada posición tiene un peso positivo (128, 64, 32, 16, 8, 4, 2, 1 en 8 bits). En complemento a 2, el bit más significativo (el bit de signo) cambia de signo: su peso se vuelve **negativo**, y el resto de los bits conserva su peso normal.

```
peso:  -128  64  32  16  8  4  2  1
```

Para decodificar cualquier patrón, basta con sumar los pesos de los bits encendidos, tratando el primero como negativo. Con el ejemplo de arriba, −22 se representa como `11101010`:

```
-128 + 64 + 32 + 0 + 8 + 0 + 2 + 0 = -22
```

Esta vista explica de un solo golpe por qué el rango es asimétrico (−128 a +127: solo existe un patrón con el bit de signo en 1 y el resto en 0, que vale exactamente −128, pero no hay un +128 equivalente), y por qué basta con mirar ese primer bit para saber si el número es negativo.

**Obtención del negativo en complemento a 2:** se invierten todos los bits y se suma 1.

```
+22 = 00010110
-22 = 11101010   (invertir bits de 00010110 y sumar 1)
```

### Atajo rápido (sin sumar 1 aparte)

Existe un método más rápido para obtener el complemento a 2 en binario, que evita invertir todos los bits y luego sumar 1 por separado (con su posible acarreo en cadena):

Recorre el número de derecha a izquierda (del bit menos significativo al más significativo). Copia los bits tal cual hasta encontrar el **primer 1** (inclúyelo tal cual). A partir de ahí, invierte todos los bits restantes.

```
+22 = 0001 0 1 1 0
              ↑↑
       copia hasta el primer 1 (incluido): ...10
       invierte el resto:                  111...

-22 = 1110 1 0 1 0
```

Este atajo da exactamente el mismo resultado que invertir todo y sumar 1, porque es matemáticamente equivalente: al sumar 1 al complemento a 1, todos los ceros a la derecha se vuelven unos y el acarreo se propaga hasta el primer 1 (que se vuelve 0). El truco calcula ese mismo patrón directamente en un solo paso, sin necesidad de hacer la suma aparte. Es útil para hacerlo mentalmente y rápido.

## 3. Aritmética con enteros con signo (complemento a 2)

La suma en complemento a 2 se realiza igual que la suma binaria normal, ignorando el acarreo que sale del bit más significativo. El **desbordamiento (overflow)** ocurre cuando el resultado no cabe en el rango representable con *n* bits.

**Regla práctica de overflow:** hay desbordamiento si el acarreo que entra al bit de signo es distinto del acarreo que sale de él.

**Ejemplo (8 bits): 100 + 50**

```
  01100100   (+100)
+ 00110010   (+50)
-----------
  10010110   → bit de signo = 1 (negativo), pero 100+50=150 > 127
```

Aquí hay overflow: el resultado matemático no cabe en 8 bits con signo.

**Ejemplo (8 bits): 20 + (−5), sin overflow**

Este ejemplo muestra cómo una resta (`20 − 5`) se convierte en una suma usando el complemento a 2 del número que se resta:

```
  00010100   (+20)
+ 11111011   (−5, complemento a 2 de 00000101)
-----------
1 00001111   → se descarta el acarreo de salida (el 1 que "sale" por la izquierda)
```

Resultado: `00001111` = **+15**, que coincide con 20 − 5. Aunque hubo un acarreo de salida del bit más significativo, **no hay overflow**: el acarreo que entra al bit de signo (1) es igual al que sale (1), así que el resultado es correcto. Este caso ilustra la regla de overflow: no basta con que haya acarreo de salida, lo que importa es si ese acarreo coincide con el que entra al bit de signo.

**Aplicación cotidiana: el problema del año 2038**

Los sistemas Unix/Linux representan el tiempo como un contador de segundos desde el 1 de enero de 1970, guardado tradicionalmente en un entero **con signo de 32 bits** (el mismo esquema de complemento a 2 que se acaba de ver, solo que con *n* = 32 en vez de 8). El 19 de enero de 2038 a las 03:14:07 UTC, ese contador llega a su valor máximo positivo representable (`01111111...1`) y, al sumarle 1 más, se desborda exactamente como el ejemplo de 100 + 50: el bit de signo pasa a 1 y el número se interpreta como negativo. El reloj del sistema "saltaría" de golpe al 13 de diciembre de 1901. Es el mismo problema conceptual que el error Y2K, pero causado por overflow de enteros con signo en vez de por un formato de fecha de dos dígitos. Es una de las razones por las que muchos sistemas modernos ya migraron `time_t` a 64 bits.

## 4. BCD (Decimal Codificado en Binario)

En BCD, cada dígito decimal (0–9) se representa con 4 bits (0000–1001). Los patrones 1010–1111 no son válidos. Al sumar dos números BCD, si un grupo de 4 bits resulta mayor a 9 o genera acarreo, se debe **corregir sumando 6 (0110)** a ese grupo.

**Ejemplo:** 8 + 7 en BCD

```
  1000  (8)
+ 0111  (7)
-------
  1111  → mayor a 9, se corrige sumando 0110
+ 0110
-------
1 0101  → dígito 5, acarreo 1 → resultado: 15 ✔
```

**Ejemplo (dos dígitos):** 48 + 37 en BCD

Aquí la corrección de un dígito genera un acarreo que afecta al siguiente, mostrando que la corrección se hace dígito por dígito, de derecha a izquierda:

```
Unidades:  1000  (8)
         + 0111  (7)
         -------
           1111  → mayor a 9, se corrige sumando 0110
         + 0110
         -------
         1 0101  → dígito 5, acarreo 1 hacia las decenas

Decenas:   0100  (4)
         + 0011  (3)
         + 0001  (acarreo de las unidades)
         -------
           1000  → dígito 8, ≤ 9, no necesita corrección
```

Resultado: dígito de decenas `8`, dígito de unidades `5` → **85**, que coincide con 48 + 37.

**Aplicación cotidiana: por qué existe el BCD si el binario puro es más eficiente**

El binario puro representa más valores con menos bits, pero convertir un binario puro a decimal (para mostrarlo en una pantalla) es un cálculo adicional que puede introducir errores de redondeo, sobre todo con cifras monetarias. El BCD sacrifica eficiencia de almacenamiento a cambio de que **cada dígito decimal se mantenga exacto**, sin conversión. Por eso se usa (o se usó históricamente) en calculadoras, relojes y hornos de microondas con display de 7 segmentos, en cajeros automáticos y sistemas bancarios, y en el tipo de dato `DECIMAL`/`NUMERIC` de muchas bases de datos, precisamente para evitar el tipo de error que produce el punto flotante binario al representar cantidades de dinero (se verá con más detalle en la Sesión 3).

## 5. Ejercicios manuales

1. Obtén el complemento a 1 y a 2 de 01011010₂ (8 bits).
2. Representa −45 en complemento a 2 usando 8 bits.
3. Suma 90 + 45 en complemento a 2 (8 bits) e indica si hay overflow.
4. Resta 84 − 37 en complemento a 2 (8 bits), convirtiéndola en una suma (84 + complemento a 2 de 37). Indica si hay overflow.
5. Suma 275 + 148 en BCD, mostrando el proceso de corrección dígito por dígito.
6. Suma 56 + 27 en BCD, mostrando el proceso de corrección dígito por dígito.

## 6. Práctica en Python (GitHub Codespaces)

Crea `sesion02.py` en tu Codespace:

```python
def complemento2(n: int, bits: int) -> str:
    """Regresa la representación en complemento a 2 de un entero (positivo o negativo) en 'bits' bits."""
    # TODO
    pass

def suma_complemento2(a: int, b: int, bits: int):
    """Suma a + b representados en complemento a 2 con 'bits' bits.
    Regresa (resultado_decimal, hubo_overflow: bool)."""
    # TODO
    pass

if __name__ == "__main__":
    print(complemento2(-22, 8))
    print(suma_complemento2(100, 50, 8))   # Se espera overflow=True
    print(suma_complemento2(20, -5, 8))    # Se espera overflow=False
```

### Ejercicios en Python

7. Completa `complemento2(n, bits)` y `suma_complemento2(a, b, bits)` en `sesion02.py`.
8. Escribe al menos 5 casos de prueba (`assert`) que cubran: un resultado positivo sin overflow, un resultado negativo sin overflow, y al menos dos casos con overflow (uno al sumar dos positivos y otro al sumar dos negativos).

## Recursos

- Tanenbaum, A. S., *Structured computer organization*, 6ª ed., Apéndice B.
- Stallings, W., *Computer organization and architecture*, 6ª ed., Capítulo de aritmética de computadoras.

## Próxima sesión

Sesión 3: punto flotante IEEE754, códigos ASCII/EBCDIC y arquitectura de Von Neumann.
