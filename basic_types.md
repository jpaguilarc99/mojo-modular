### **Tipos de datos básicos**

**Python Object**

Este tipo de objetos nos permiten ejecutar o evaluar código Python directamente en los interpretes de Mojo, debido a que Mojo fue pensado inicialmente para hacer parte de Python y luego se volvió un lenguaje por separado.


```mojo
x = Python.evaluate('5 + 10')
print(x)
```

    15


Utilizar el comando %%python en la parte superior de las celdas ejecutará código Python en lugar de Mojo.


```mojo
%%python
x = 5 + 10
print(x)
```

    15


x es actualmente un pointer al heap de memoria.

Los conceptos fundamentales de la "pila" y la memoria del montón "heap" son importantes de entender. "heap allocated memory" se refiere a la memoria asignada en la región de memoria conocida como el "heap". El heap es una estructura de datos utilizada por los programas para almacenar y gestionar dinámicamente datos durante la ejecución. A diferencia de la memoria estática, que se reserva en tiempo de compilación y se asigna a variables globales o locales estáticas, la memoria del heap se reserva y se libera en tiempo de ejecución según las necesidades del programa.

- stack: o memoria de "pila" es rápida pero pequeña. Sus valores de almacenamiento no pueden cambiar durante la ejecución.
- pointer: un puntero es una dirección para buscar el valor en algún lugar especifico de la memoria.
- heap: o memoria del montón, es una memoria amplia y su tamaño puede variar durante la ejecución, pero requiere de un puntero para acceder a los datos almacenados, lo cual es lento.

Una de las funcionalidades más llamativas de Mojo, es que se puede acceder a los paquetes o librerías desarrolladas en Python:


```mojo
let py = Python.import_module("builtins")
py.print("estamos usando el print nativo de python")
```

    estamos usando el print nativo de python



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



```mojo
py.print(py.type(ar))
```

    <class 'numpy.ndarray'>



```mojo
py.print(py.id(ar))
```

    139981712481328


Se puede ver que el ID está apuntando a un objeto en C desde Python, y Mojo se comporta de igual forma al utilizar un Python Object. Acceder al valor realmente utiliza la dirección para buscar los datos en el heap, lo que es costoso en rendimiento.

Veamos una representación de cómo un objeto de C está siendo apuntado como un dictionary a heap:


```mojo
%%python
heap = {
        139981712481328: {
                "type": "int",
                "ref_count": 1,
                "size": 1,
                "digit": 8,
                #...
            }
            #...
    }
```

Mientras que para la memoria de stack se representaría como:


```mojo
%%python
[
    { "frame": "main", "variables": { "x": 139981712481328 } }
]
```

Para el objeto de Python se puede cambiar fácilmente la representación:


```mojo
ar = "mojo"
```

Y el objeto de heap en C cambiaría de igual forma su representación:


```mojo
%%python
heap = {
    44601345678945 : {
        "type": "string",
        "ref_count": 1,
        "size": 4,
        "ascii": True,
        # utf-8 / ascii for "mojo"
        "value": [109, 111, 106, 111]
        # ...
    }
}
```

Mojo también nos permite hacer esto cuando el tipo es un PythonObject, funciona de la misma manera exacta que lo haría en un programa de Python.

Esto permite que el tiempo de ejecución realice cosas convenientes para nosotros.

Una vez que el ref_count llega a cero, se desasignará del montón (heap) durante la recolección de basura, para que el sistema operativo pueda usar esa memoria para otra cosa.
Un entero puede crecer más allá de los 64 bits aumentando su tamaño.
Podemos cambiar dinámicamente el tipo.
Los datos pueden ser grandes o pequeños, no tenemos que pensar en cuándo debemos asignar al montón.
Sin embargo, esto también conlleva una penalización, se utiliza mucha memoria adicional para los campos extras, y se requieren instrucciones de CPU para asignar los datos, recuperarlos, realizar la recolección de basura, etc.

En Mojo, podemos eliminar toda esa sobrecarga:

### **Mojo 🔥**


```mojo
var y = 5 + 10
print(y)
```

    15


- Ya no se requiere la costosa asignación, recolección de basura y desvío
- El compilador puede hacer enormes optimizaciones cuando conoce el tipo numérico
- El valor se puede pasar directamente a los registros para operaciones matemáticas
- No hay sobrecarga asociada con la compilación en bytecode y la ejecución a través de un intérprete
- Los datos ahora se pueden empacar en un vector para obtener enormes ganancias de rendimiento
- Esta última es muy importante en el mundo actual, veamos cómo Mojo nos brinda el poder para aprovechar el hardware moderno.

**SIMD**

SIMD significa Single Instruction, Multiple Data. El hardware actual contiene registros especiales que te permiten realizar la misma operación en un vector con una sola instrucción, mejorando el rendimiento:


```mojo
from DType import DType
y_1 = SIMD[DType.uint8, 4](1, 2, 3, 4)
print(y_1)
```

    [1, 2, 3, 4]


En la definición [DType.uint8, 4], se conocen como parámetros, lo que significa que deben ser conocidos en tiempo de compilación, mientras que (1, 2, 3, 4) son los argumentos que pueden ser conocidos en tiempo de compilación o en tiempo de ejecución.

Por ejemplo, la entrada del usuario o los datos obtenidos de una API son conocidos en tiempo de ejecución y, por lo tanto, no se pueden utilizar como parámetros durante el proceso de compilación.

En otros lenguajes, 'argumento' y 'parámetro' significan lo mismo, pero en Mojo es una distinción muy importante.

Ahora, esto es un vector de números de 8 bits empaquetados en 32 bits. Podemos realizar una instrucción única en todo el vector en lugar de 4 instrucciones separadas:


```mojo
y_1 *= 10
print(y_1)
```

    [10, 20, 30, 40]


La representación binaria es la forma en que se almacena la memoria, con cada bit representando un 0 o un 1. La memoria generalmente es direccionable por bytes, lo que significa que cada dirección de memoria única apunta a un byte, que consta de 8 bits.

Así es como se representan los primeros 4 dígitos en un uint8 en el hardware:

1 = 00000001
2 = 00000010
3 = 00000011
4 = 00000100
El 1 y el 0 binarios representan ENCENDIDO o APAGADO.

Estamos empaquetando los datos juntos con SIMD en la pila (o stack) para que puedan ser pasados a un registro SIMD de la siguiente manera:

00000001 00000010 00000011 00000100

El registro SIMD en las CPU modernas es grande; veamos qué tamaño tiene en el entorno de desarrollo Mojo:


```mojo
from TargetInfo import simdbitwidth
print(simdbitwidth())
```

    512


Esto significa que podríamos agrupar 64 números de 8 bits juntos y realizar un cálculo de todos ellos con una sola instrucción.

También podemos inicializar SIMD con un solo argumento, para generar por ejemplo un vector de ones:


```mojo
ones = SIMD[DType.uint8, 4](1)
print(ones)
```

    [1, 1, 1, 1]


#### **Scalars**

Son valores numéricos simples, o en resumidas cuentas un SIMD escalar:



```mojo
var x = Uint8(1)
x = "va a generar un error"
```

    error: [0;1;31m[1mExpression [50]:25:13: [0m[1muse of unknown declaration 'Uint8'
    [0m    var x = Uint8(1)
    [0;1;32m            ^~~~~
    [0m[0m
    expression failed to parse (no further compiler diagnostics)

UInt8 solo es una simplificación para SIMD[DType.uint8, 1]. Veamos todos los tipos de valores SIMD:

- Float16.
- Float32.
- Float64.
- Int8.
- Int16.
- Int32.
- Int64.
- UInt8.
- UInt16.
- UInt32.
- UInt64.

También vemos que al intentar modificar el tipo de la variable Uint8 se genera un error. Esto es debido a que Mojo es **fuertemente tipado**.

Si utilizamos por ejemplo paquetes de Python, esto nos entregará objetos con comportamiento de python, el cual es loosely typed o tiene un tipado de variables muy débil.


```mojo
np = Python.import_module("numpy")
arr = np.ndarray([5])
print(arr)
arr = "esto si funciona bien"
print(arr)
```

    [0.   0.25 0.5  0.75 1.  ]
    esto si funciona bien


#### **Strings**

En Mojo, los strings asignados al heap, no se importan por defecto. Hay que acceder a sus libs:


```mojo
from String import String
s = String("Mojo🔥")
print(s)
```

    Mojo🔥


Un String es un puntero a datos asignados al heap, lo que significa que podemos cargar muchos datos en el String y cambiar el tamaño de los datos dinamicamente durante la ejecución.

**DynamicVector** es similar a las listas de Python, por lo que podemos hacer slicing de los indices del String, que representan los chars:


```mojo
print(s[0])
```

    M



```mojo
print(s[1])
```

    o


Ahora la representación decimal ASCII de los valores:


```mojo
from String import ord
print(ord(s[0]))
```

    77


Podemos construir un String de esta forma, poniendo el 78 que es la N y el 79 que es la O :


```mojo
from Vector import DynamicVector
let vec = DynamicVector[Int8](2)
vec.push_back(78)
vec.push_back(79)
```

Podemos usar una StringRef para obtener un puntero a la misma ubicación en memoria, pero con los métodos necesarios para mostrar los números como texto:


```mojo
from Pointer import DTypePointer
from DType import DType
let vec_str_ref = StringRef(DTypePointer[DType.int8](vec.data).address, vec.size)
print(vec_str_ref)
```

    NO


Como siempre está apuntando al mismo espacio en memoria heap, cambiar el valor original del vector también va a cambiar el valor referenciado:


```mojo
vec[1] = 78
print(vec_str_ref)
```

    NN


Creamos una copia profunda del String y lo asignamos al heap:


```mojo
from String import String
let vec_str = String(vec_str_ref)
print(vec_str)
```

    NN


Ahora podemos modificar el vector original para que no cambie el valor de vec_str en la memoria heap:


```mojo
vec[0] = 65
vec[1] = 65
print(vec_str)
```

    NN


**StringLiteral**

Es escrito directamente, y se carga en la read-only memory, lo que significa que es constante y solo existe en la duración del programa:


```mojo
var lit = "Este es mi StringLiteral"
print(lit)
```

    Este es mi StringLiteral



```mojo
# Forcemos un error
lit = 20
```

    error: [0;1;31m[1mExpression [82]:33:11: [0m[1mcannot implicitly convert 'Int' value to 'StringLiteral' in assignment
    [0m    lit = 20
    [0;1;32m          ^~
    [0m[0m
    expression failed to parse (no further compiler diagnostics)


```mojo
emoji = String("🔥😀")
print("fire:", emoji[0:4])
print("smiley:", emoji[4:8])
```

    fire: 🔥
    smiley: 😀


#### **Bool**


```mojo
let bool: Bool = True
print(bool == False)
```

    False



```mojo
let yes: Bool = True
let no: Bool = False
print(yes != no)
```

    True


#### **Int**

El Int tiene el mismo tamaño que la arquitectura del PC. Ejemplo, si la máquina es de 64 bits, el int también lo es.


```mojo
let i: Int = 2
print(i)
```

    2


Puede ser usado como un index:


```mojo
var vec_2 = DynamicVector[Int]()
vec_2.push_back(2)
vec_2.push_back(4)
vec_2.push_back(6)

print(vec_2[i])
```

    6


#### **FloatLiteral**


```mojo
let float: FloatLiteral = 3.3
print(float)
```

    3.2999999999999998



```mojo
let f32 = Float32(float)
print(f32)
```

    3.2999999523162842


#### **ListLiteral**

Cuando inicializas la lista, los tipos pueden ser inferidos, sin embargo, al recuperar un elemento, necesitas proporcionar el tipo como un parámetro:


```mojo
let list: ListLiteral[Int, FloatLiteral, StringLiteral] = [1, 5.0, "Mojo"]
print(list.get[2, StringLiteral]())
```

    Mojo


#### **Tuplas**


```mojo
let tup = (1, "Mojo", 3)
print(tup.get[2, Int]())
```

    3


#### **Slicing**

Los slices en Mojo siguen la convención start:end:step


```mojo
let original = String("MojoDojo")
print(original[0:4])
```

    Mojo



```mojo
let slice_expression = slice(0, 4)
print(original[slice_expression])
```

    Mojo



```mojo
print(original[0:4:2])
```

    Mj



```mojo
let slice_expression = slice(0, 4, 2)
print(original[slice_expression])
```

    Mj


#### **Errores**


```mojo
def return_error():
    raise Error("Esto retorna un error de tipo")
return_error()
```

    Error: Esto retorna un error de tipo


**Ejemplos**

1. Usar el interprete de Python para calcular 2 a la potencia de 8 en un objeto de Python e imprimirlo:


```mojo
potencia = Python.evaluate("pow(2, 8)")
print(potencia)
```

    256


2. Usar el modulo math de Python y retornar el valor de pi a Mojo.


```mojo
let math = Python.import_module("math")
let pi = math.pi
print(pi)
```

    3.141592653589793


3. Inicializar dos floats de 64 bits utilizando SIMD


```mojo
ft1 = SIMD[DType.float64, 1](2.0)
ft2 = SIMD[DType.float64, 1](2.0)
multiply_floats = ft1 * ft2
print(multiply_floats)
```

    4.0


4. Crear un ciclo utilizando SIMD que imprima cuatro filas de datos tales que: 

[1,0,0,0]

[0,1,0,0]

[0,0,1,0]

[0,0,0,1]

En Mojo se crea de forma similar los ciclos que en Python:


```mojo
for i in range(4):
    print(i)
```

    0
    1
    2
    3



```mojo
from DType import DType
vector_ex = SIMD[DType.int8, 4](0)
```


```mojo
for i in range(4):
    vec_temp = SIMD[DType.int8, 4](0)
    vec_temp[i] = 1
    print(vec_temp)
```

    [1, 0, 0, 0]
    [0, 1, 0, 0]
    [0, 0, 1, 0]
    [0, 0, 0, 1]

