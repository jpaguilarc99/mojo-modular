### **Funciones y structures en Mojo**

Mojo es un lenguaje compilado y muchas de sus características de rendimiento y seguridad de memoria se derivan de este hecho. El código de Mojo puede compilarse antes de tiempo (AOT) o justo a tiempo (JIT). Mojo también admite entornos REPL como el que ejecuta el código en este cuaderno Jupyter y REPL de línea de comandos.

Al igual que otros lenguajes compilados, Mojo requiere una función main() como punto de entrada a un programa. Por ejemplo:


```mojo
fn main():
    var x: Int = 1
    x += 1
    print(x)
```

Analogo a Python, se espera def main() en lugar de fn main(). Ambas alternativas funcionan en Mojo, pero el uso de fn se comporta de forma diferente (es una función más estricta).

La función main() no es necesaria en un entorno REPL, como en este entorno de notebook. Pero la función main() es necesaria cuando se quiera escribir scripts .mojo.

#### **Sintaxis y semántica**

Mojo utiliza la misma sintaxis y semántica que Python, ya que originalmente fue pensado para ser un plus a este lenguaje.

Por ejemplo, al igual que Python, Mojo utiliza saltos de línea e indentación para definir bloques de código (no llaves), y Mojo soporta toda la sintaxis de flujo de control de Python, como las condiciones if y los bucles for.

Sin embargo, Mojo es todavía está en desarrollo temprano, por lo que hay algunas cosas de Python que no están implementadas en Mojo todavía. Todas las características de Python que faltan llegarán paso a paso, pero Mojo ya incluye muchas características y capacidades más allá de lo que está disponible en Python.

Por esto, la idea es centrarse más en algunas de las características del lenguaje que son exclusivas de Mojo (en comparación con Python).

#### **Funciones**

Las funciones Mojo pueden declararse con fn (como vimos arriba) o def (como en Python). La declaración fn impone comportamientos fuertemente tipados y seguros en memoria, mientras que def proporciona comportamientos dinámicos al estilo Python.

Tanto las funciones fn como def tienen su valor, y es importante que utilizar ambas. Sin embargo, para los propósitos de esta exploración, vamos a centrarnos sólo en las funciones fn. Para más detalles sobre ambas, consulta el manual de programación de Mojo.

Iremos viendo cómo las funciones fn imponen comportamientos fuertemente tipados y seguros para la memoria de los Scripts.

#### **Variables y Scope**

Se pueden declarar variables mutables, como x en la función main() anterior, con var para crear un valor mutable o con let para crear un valor inmutable. Veamos un ejemplo intentando modificar una declaración con *var* y una declaración con *let*


```mojo
# Declaración con var
var value1: Int = 5
value1 += 10
print(value1)
```

    15



```mojo
# Declaración con let
let value2: Int = 10

# Generemos un error intentando reasignar la variable
value2 += 20
print(value2)
```

    error: [0;1;31m[1mExpression [10]:21:5: [0m[1mexpression must be mutable for in-place operator destination
    [0m    value2 += 20
    [0;1;32m    ^~~~~~
    [0m[0m
    expression failed to parse (no further compiler diagnostics)

- La declaración *let* vuelve la variable inmutable, por lo que no se puede incrementar su valor.
- La declaración *var* vuelve la variable mutable y nos permite incrementar el valor.

La forma *fn* de declarar funciones en Mojo, difere a *def* en que para *fn* hay que definir el scope de las variables definidas internamente *var* o *let*. Intentemos declarar una *fn* con y sin tipado y scope para generar un error y luego de manera correcta:


```mojo
fn do_maths() -> Int:
    x = 2
    let y = 3
    return x * y    
```

    error: [0;1;31m[1mExpression [17]:8:5: [0m[1muse of unknown declaration 'x', 'fn' declarations require explicit variable declarations
    [0m    x = 2
    [0;1;32m    ^
    [0m[0m
    error: [0;1;31m[1mExpression [17]:10:12: [0m[1muse of unknown declaration 'x', 'fn' declarations require explicit variable declarations
    [0m    return x * y    
    [0;1;32m           ^
    [0m[0m
    expression failed to parse (no further compiler diagnostics)

Vemos que nos genera un error pidiendonos que pongamos explicitamente la declaración de la variable y su tipo. Definamos la función correctamente: 


```mojo
fn do_maths() -> Int:
    let x: Int = 2
    let y: Int = 2
    return x * y
```

**Argumentos de Funciones y valores de retorno**

A diferencia de las variables locales, todos los argumentos de una function deben especificar un tipo.
Para poder retornar un valor en una función se debe especificar el valor de retorno en la declaración de la function (-> Int)


```mojo
fn add(x: Int, y: Int) -> Int:
    return x + y
z = add(1, 2)
print(z)
```

    3


**Mutabilidad y Propiedad de Argumentos**

Exploremos cómo se comparten los valores de los argumentos con una función.

Veamos que, arriba, add() no modifica x o y, sólo lee los valores. De hecho, tal y como está escrita, la función no puede modificarlos porque los argumentos fn son referencias inmutables, por defecto.

En términos de convenciones de argumentos, esto se llama "tomar prestado", y aunque es el valor por defecto para las funciones fn, puedes hacerlo explícito con la declaración "borrowing" de esta manera (esto se comporta exactamente igual que la add() de arriba):


```mojo
fn add(borrowed x: Int, borrowed y: Int) -> Int:
    return x + y
```

Si se quiere que los argumentos sean mutables, se debe declarar que la convención de argumentos es "inout". Esto significa que los cambios realizados en los argumentos dentro de la función son visibles fuera de la función.

Por ejemplo, esta función puede modificar las variables originales:


```mojo
fn add_inout(inout x: Int, inout y: Int) -> Int:
    x += 1
    y += 1
    return x + y

var a = 1
var b = 2
c = add_inout(a, b)
print(a) # a + 1
print(b) # b + 1
print(c) # a: 2 + b: 3
```

    2
    3
    5


Otra opción es declarar el argumento como "owned", lo que proporciona a la función plena propiedad del valor (es mutable y único garantizado). De esta forma, la función puede modificar el valor internamente y no preocuparse de afectar a variables externas a la función. Por ejemplo:


```mojo
from String import String

fn set_fire(owned text: String) -> String:
    text += "🔥"
    return text

fn mojo():
    let a: String = "Mojo"
    let b = set_fire(a)
    print(a)
    print(b)
mojo()
```

    Mojo
    Mojo🔥


En este caso, Mojo creó una copia de a y la pasó como argumento a la función, sin modificar en memoria originalmente la variable.

Sin embargo, si quiere dar a la función la propiedad del valor y no quiere hacer una copia (que puede ser una operación costosa para algunos tipos), entonces puede añadir el operador ^ "transferir" cuando pase a a la función. El operador de transferencia destruye efectivamente el nombre de la variable local: cualquier intento de invocarla provoca un error de compilación. Ejemplo:


```mojo
let b = set_fire(a^) # La expresión ^a hace que se transfiera la propiedad de la variable
```

### **Structures**

Mojo permite crear abstracciones de alto nivel de objetos en una estructura o "struct". Una "struct" en Python es el análogo a una clase. Ambos soportan métodos, campos, decoradores, entre otros. Sin embargo, las structs de Mojo son completamente estáticas, están hechas en tiempo de compilación, por lo que no soportan dinamismo o cambios en tiempo de ejecución a la estructura. 

Veamos la estructura general de una "struct":


```mojo
struct MyPair:
    var first: Int
    var second: Int
    
    # El inicializador funciona como constructor
    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second
        
    fn dump(inout self):
        print(self.first)
        print(self.second)
```

Utilicemos la struct MyPair:


```mojo
def pair_test() -> Bool:
    let p = MyPair(1, 2)
    return True
pair_test()
```

#### **Integración de Python en Mojo**


```mojo
from PythonInterface import Python

let np = Python.import_module("numpy")

ar = np.arange(15).reshape(3, 5)
print(ar)
print(ar.shape)
```

    [[ 0  1  2  3  4]
     [ 5  6  7  8  9]
     [10 11 12 13 14]]
    (3, 5)

