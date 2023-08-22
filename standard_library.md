## **Assert**

Verifica una condición que debe ser verdadera en el contexto de parámetros y tiempos de compilación:


```mojo
from Assert import assert_param
```

Hagamos un assert de que dos números son positivos:


```mojo
fn add_positives[x: Int, y: Int]() -> UInt8:
    assert_param[x > 0]()
    assert_param[y > 0]()
    return x + y
```


```mojo
let res = add_positives[2, 4]()
print(res)
```

    6


Ejecuta correctamente la función debido a que se cumplen los asserts para x, y. Veamos ahora qué pasa si no se cumplen los asserts:


```mojo
add_positives[-2, 4]()
```

    error: [0;1;31m[1mExpression [7]:7:1: [0m[1mno viable expansions found
    [0mfn __lldb_expr__7(inout __mojo_repl_arg: __mojo_repl_context__):
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [7]:9:28: [0m[1m  call expansion failed - no concrete specializations
    [0m    __mojo_repl_expr_impl__(__mojo_repl_arg, __get_address_as_lvalue(__mojo_repl_arg.`res`.load().address), __get_address_as_lvalue(__mojo_repl_arg.`___lldb_expr_failed`.load().address))
    [0;1;32m                           ^
    [0m[0m
    [0;1;30m[1mExpression [7]:13:1: [0m[1m    no viable expansions found
    [0mdef __mojo_repl_expr_impl__(inout __mojo_repl_arg: __mojo_repl_context__, inout `res`: __mlir_type.`!kgen.declref<@"$SIMD"::@SIMD<_63x13_type: @"$DType"::@DType = #lit.struct<{value: dtype = ui8}>, _63x26_size: @"$Builtin"::@"$Int"::@Int = #lit.struct<{value = 1}>>>`, inout `___lldb_expr_failed`: __mlir_type.`!kgen.declref<@"$Builtin"::@"$Bool"::@Bool>`) -> None:
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [7]:21:26: [0m[1m      call expansion failed - no concrete specializations
    [0m  __mojo_repl_expr_body__()
    [0;1;32m                         ^
    [0m[0m
    [0;1;30m[1mExpression [7]:15:3: [0m[1m        no viable expansions found
    [0m  def __mojo_repl_expr_body__() -> None:
    [0;1;32m  ^
    [0m[0m
    [0;1;30m[1mExpression [7]:18:25: [0m[1m          call expansion failed - no concrete specializations
    [0m    add_positives[-2, 4]()
    [0;1;32m                        ^
    [0m[0m
    [0;1;30m[1mExpression [5]:6:1: [0m[1m            no viable expansions found
    [0mfn add_positives[x: Int, y: Int]() -> UInt8:
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [5]:7:24: [0m[1m              constraint failed: param assertion failed
    [0m    assert_param[x > 0]()
    [0;1;32m                       ^
    [0m[0m
    expression failed to parse (no further compiler diagnostics)

También podemos agregar un mesage custom para cambiar el error del output del compilador:


```mojo
fn add_positives[x: Int, y: Int]() -> UInt8:
    assert_param[x > 0, "x no es positivo"]()
    assert_param[y > 0, "y no es positivo"]()
    return x + y

let res = add_positives[-2, -4]()
print(res)
```

    error: [0;1;31m[1mExpression [8]:12:1: [0m[1mno viable expansions found
    [0mfn __lldb_expr__8(inout __mojo_repl_arg: __mojo_repl_context__):
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [8]:14:28: [0m[1m  call expansion failed - no concrete specializations
    [0m    __mojo_repl_expr_impl__(__mojo_repl_arg, __get_address_as_lvalue(__mojo_repl_arg.`res`.load().address), __get_address_as_lvalue(__mojo_repl_arg.`___lldb_expr_failed`.load().address))
    [0;1;32m                           ^
    [0m[0m
    [0;1;30m[1mExpression [8]:18:1: [0m[1m    no viable expansions found
    [0mdef __mojo_repl_expr_impl__(inout __mojo_repl_arg: __mojo_repl_context__, inout `res`: __mlir_type.`!kgen.declref<@"$SIMD"::@SIMD<_63x13_type: @"$DType"::@DType = #lit.struct<{value: dtype = ui8}>, _63x26_size: @"$Builtin"::@"$Int"::@Int = #lit.struct<{value = 1}>>>`, inout `___lldb_expr_failed`: __mlir_type.`!kgen.declref<@"$Builtin"::@"$Bool"::@Bool>`) -> None:
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [8]:27:26: [0m[1m      call expansion failed - no concrete specializations
    [0m  __mojo_repl_expr_body__()
    [0;1;32m                         ^
    [0m[0m
    [0;1;30m[1mExpression [8]:20:3: [0m[1m        no viable expansions found
    [0m  def __mojo_repl_expr_body__() -> None:
    [0;1;32m  ^
    [0m[0m
    [0;1;30m[1mExpression [8]:23:36: [0m[1m          call expansion failed - no concrete specializations
    [0m    let res = add_positives[-2, -4]()
    [0;1;32m                                   ^
    [0m[0m
    [0;1;30m[1mExpression [8]:6:1: [0m[1m            no viable expansions found
    [0mfn add_positives[x: Int, y: Int]() -> UInt8:
    [0;1;32m^
    [0m[0m
    [0;1;30m[1mExpression [8]:7:44: [0m[1m              constraint failed: x no es positivo
    [0m    assert_param[x > 0, "x no es positivo"]()
    [0;1;32m                                           ^
    [0m[0m
    expression failed to parse (no further compiler diagnostics)

#### **debug_assert**

Verifica que la condición es verdadera en una compilación de debug


```mojo
from Assert import debug_assert

fn test_debug_assert[x: Int](y: Int):
    debug_assert(x == 42, "x no es igual a 42")
    debug_assert(y == 42, "y no es igual a 42")
test_debug_assert[1](2)
```

## **Benchmark**

Sirve para calcular benchmarks de rendimiento en algoritmos.


```mojo
from Benchmark import Benchmark
```


```mojo
alias n = 35
```


```mojo
fn fib(n: Int) -> Int:
    if n <= 1:
        return n
    else:
        return fib(n-1) + fib(n-2)
```

Para aplicar el Benchmark a la función, se crea una fn anidada que no toma argumentos y no retorna valores:


```mojo
fn bench():
    fn closure():
        for i in range(n):
            _ = fib(i)
    let nanoseconds = Benchmark().run[closure]()
    print("Nanoseconds:", nanoseconds)
    print("Seconds:", Float64(nanoseconds) / 1e9)
bench()
```

Comparamos la recursividad con las iteraciones:


```mojo
fn fib_iterative(n: Int) -> Int:
    var count = 0
    var n1 = 0
    var n2 = 1

    while count < n:
       let nth = n1 + n2
       n1 = n2
       n2 = nth
       count += 1
    return n1

fn bench_iterative():
    fn iterative_closure():
        for i in range(n):
            _ = fib_iterative(i)

    let iterative = Benchmark().run[iterative_closure]()
    print("Nanoseconds iterative:", iterative)

bench_iterative()
```

    Nanoseconds iterative: 0


## **Buffer**


```mojo
from Buffer import Buffer
from DType import DType
from Pointer import DTypePointer
```

El bufer no implica directamente la memoria, es más bien una vista o lectura de datos que son propiedades de otros objetos.

Asignamos un ocho uint8 y pasamos el puntero al bufer:


```mojo
let p = DTypePointer[DType.uint8].alloc(8)
let x = Buffer[8, DType.uint8](p)
```

Hacemos ceros los valores para asegurarnos que no se utilice información basura:



```mojo
x.zero()
print(x.simd_load[8](0))
```

    [0, 0, 0, 0, 0, 0, 0, 0]


**Get Item y Set Item**

Iteramos a través del SIMD y asignamos elementos:


```mojo
for i in range(len(x)):
    x[i] = i
print(x.simd_load[8](0))
```

    [0, 1, 2, 3, 4, 5, 6, 7]


**Copy Init**

Copiamos el bufer x a y, cambiamos el tamaño dinámico a 4 y multiplicamos todos los valores por 10.

Vemos que el pointer modificó dinamicamente los primeros 4 items del SIMD.


```mojo
var y = x
y.dynamic_size = 4

for i in range(y.dynamic_size):
    y[i] *= 10
print(x.simd_load[8](0))
```

    [0, 10, 20, 30, 4, 5, 6, 7]


## **Pointer**

Almacena un address al darle un DType, permitiendo indexar, cargar y modificar datos con acceso conveniente a operaciones SIMD.


```mojo
from Pointer import DTypePointer

from DType import DType
from Random import rand
from Memory import memset_zero
```

**Inicialización**

Creamos dos variables para almacenar un nuevo address en la memoria heap y guardamos 8 bytes


```mojo
var p1 = DTypePointer[DType.uint8].alloc(8)
var p2 = DTypePointer[DType.uint8].alloc(8)
```

**Operadores**


```mojo
if p1:
    print("p1 is not null")
print("p1 is at a lower address than p2:", p1 < p2)
print("p1 and p2 are equal:", p1 == p2)
print("p1 and p2 are not equal:", p1 != p2)
```

    p1 is not null
    p1 is at a lower address than p2: False
    p1 and p2 are equal: False
    p1 and p2 are not equal: True


**Almacenando y cargando datos SIMD**


```mojo
memset_zero(p1, 8)
```


```mojo
var all_data = p1.simd_load[8](0)
print(all_data)
```

    [0, 0, 0, 0, 0, 0, 0, 0]


Guardemos valores aleatorios en 4 bits solamente:


```mojo
rand(p1, 4)
print(all_data)
```

    [0, 0, 0, 0, 0, 0, 0, 0]


Se debe volver a cargar los datos para ver los cambios:


```mojo
all_data = p1.simd_load[8](0)
print(all_data)
```

    [0, 33, 193, 117, 0, 0, 0, 0]



```mojo
var half = p1.simd_load[4](0)
half = half + 1
p1.simd_store[4](4, half)
```


```mojo
all_data = p1.simd_load[8](0)
print(all_data)
```

    [0, 33, 193, 117, 1, 34, 194, 118]


**Pointer Arithmetic**


```mojo
p1 += 1
all_data = p1.simd_load[8](0)
print(all_data)
```

    [33, 193, 117, 1, 34, 194, 118, 0]



```mojo
p1 -= 1
all_data = p1.simd_load[8](0)
print(all_data)
```

    [0, 33, 193, 117, 1, 34, 194, 118]


**Liberando memoria**


```mojo
p1.free()
```


```mojo
all_data = p1.simd_load[8](0)
print(all_data)
```

    [0, 61, 132, 204, 57, 86, 0, 0]


### **Construimos nuestra propia Struct Matrix**

Jugar con Pointers es delicado. Construyamos una abstracción de struct segura que interactúe con el pointer:


```mojo
struct Matrix:
    var data: DTypePointer[DType.uint8]
    
    fn __init__(inout self):
        "Inicialziamos la struct y le asignamos zeros"
        self.data = DTypePointer[DType.uint8].alloc(64)
        memset_zero(self.data, 64)
        
    fn __del__(owned self):
        return self.data.free()
    
    fn __getitem__(self, row: Int) -> SIMD[DType.uint8, 8]:
        "Nos permite usar let x = obj[1]"
        return self.data.simd_load[8](row * 8)
    
    fn __setitem__(self, row: Int, data: SIMD[DType.uint8, 8]):
        "Nos permite usar obj[1] = SIMD[DType.uint8]()"
        return self.data.simd_store[8](row * 8, data)
    
    fn print_all(self):
        print("--------matrix--------")
        for i in range(8):
            print(self[i])        
```


```mojo
let matrix = Matrix()
matrix.print_all()
```

    --------matrix--------
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0, 0, 0, 0]



```mojo
for i in range(8):
    matrix[i] = i
matrix.print_all()
```

    --------matrix--------
    [0, 0, 0, 0, 0, 0, 0, 0]
    [1, 1, 1, 1, 1, 1, 1, 1]
    [2, 2, 2, 2, 2, 2, 2, 2]
    [3, 3, 3, 3, 3, 3, 3, 3]
    [4, 4, 4, 4, 4, 4, 4, 4]
    [5, 5, 5, 5, 5, 5, 5, 5]
    [6, 6, 6, 6, 6, 6, 6, 6]
    [7, 7, 7, 7, 7, 7, 7, 7]



```mojo
for i in range(8):
    matrix[i][0] = 9
    matrix[i][7] = 9
matrix.print_all()
```

    --------matrix--------
    [9, 0, 0, 0, 0, 0, 0, 9]
    [9, 1, 1, 1, 1, 1, 1, 9]
    [9, 2, 2, 2, 2, 2, 2, 9]
    [9, 3, 3, 3, 3, 3, 3, 9]
    [9, 4, 4, 4, 4, 4, 4, 9]
    [9, 5, 5, 5, 5, 5, 5, 9]
    [9, 6, 6, 6, 6, 6, 6, 9]
    [9, 7, 7, 7, 7, 7, 7, 9]


## **DynamicVector**


```mojo
from Vector import DynamicVector
```

**init**

Podemos reservar memoria para agregar elementos sin el costo de copiar los elementos a medida que crece el vector.


```mojo
var vec = DynamicVector[Int](8)
```

**push_back**

Utilizamos push back para agregar elementos al vector


```mojo
vec.push_back(10)
vec.push_back(20)

print(len(vec))
```

    2


**variables**


```mojo
print(vec.capacity)
print(vec.data[0])
print(vec.size)
```

    8
    10
    2


**pop_back**

Se accede al último elemento del vector, se quita la reserva en memoria, y se reduce el tamaño de elementos por 1:


```mojo
print(vec.pop_back())
```

    20

