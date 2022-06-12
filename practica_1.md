### 4) Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.

```java
queue[5] instancias;
int cant = 5;

Process proceso[id:0..4]{
    Object instancia;
    <await (cant > 0); cant--; instancia = instancias.pop(); >
    // Usa instancia
    <instancias.push(instancia);>
    <cant++;>
}
```
#
### 5) En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.
#### a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.
#### b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
#### c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).
#### d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).
#### e) Modifique la solución de (c) para el caso en que además hay un proceso Coordinador que le indica a cada persona cuando puede usar la impresora.

## 5.a)
```java
boolean libre = true;

Process persona[id:0..N-1]{
    text documento;
    <await (libre); libre = false>
    Imprimir(documento);
    <libre = true;>
}
```

## 5.b)
```java
cola llegada(int);
int siguiente = -1;
boolean libre = true;

Process persona[id:0..N-1]{
    text documento;

    <if (libre){
        siguiente = id;
        libre = false;
    } else {
        llegada.push(id);
    }>

    <await (siguiente == id);>
    Imprimir(documento);

    <if (not empty(llegada)){
        siguiente = llegada.pop();
    } else { libre = true; }>
}
```

## 5.c)
```java
// Quedó muy parecido al anterior (PREGUNTAR)
colaOrdenada llegada(int);
int siguiente = -1;
boolean libre = true;

Process persona[id:0..N-1]{
    text documento;

    <if (libre){
        siguiente = id;
        libre = false;
    } else {
        llegada.agregarOrdenado(id);
    }>

    <await (siguiente == id);>
    Imprimir(documento);

    <if (not empty(llegada)){
        siguiente = llegada.pop();
    } else { libre = true; }>
}
```

## 5.d)
```java
// Es necesario proteger siguiente++ ? (PREGUNTAR)
int siguiente = 0;

Process persona[id:0..N-1]{
    text documento;

    <await (siguiente == id);>
    Imprimir(documento);
    <siguiente++;>
}
```

## 5.e)
```java
// Habría que proteger el agregarOrdenado? Como agrega ordenado no sé si habría interferencia (PREGUNTAR)
colaOrdenada llegada(int);
int siguiente = -1;
boolean libre = true;

Process persona[id:0..N-1]{
    text documento;

    <llegada.agregarOrdenado(id);>

    <await (siguiente == id); libre = false;>
    Imprimir(documento);

    <libre = true;>
}

Process coordinador {
    while(true){
        <await (libre && not empty(llegada)); siguiente = llegada.pop();>
    }
}
```
#

### 6) Resolver con SENTENCIAS AWAIT (<> y/o <await B; S>) el siguiente problema. En un examen final hay P alumnos y 3 profesores. Cuando todos los alumnos han llegado comienza el examen. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respectando el orden en que los alumnos van entregando.

```java
int cantAlumnos = 0;
int corregidos = 0;
cola examanes(text,int);
int[P] notas ([P] = -1);

Process alumno[id:0..P-1]{
    int nota;
    text examen;

    <cant++;>
    <await (cant == P);>
    // Resuelve el examen
    <examenes.push(examen,id);>

    <await (notas[id] != -1);>
    nota = notas[id];
}

Process profesor[id:0..2]{
    int alumnoAct; text examenAct;
    while (corregidos < P){
        <await (not empty(examenes)); corregidos++;
        examenes.pop(alumnoAct,examenAct);>

        notas[alumnoAct] = corregir(examenAct);
    }
}
```
#

### 8) Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 2 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador.
#### Nota: puede basarse en la solución para implementar barreras con Flags y Coordinador vista en la teoría 3.
```java
cola llegada(int);
int siguiente = -1;
boolean libre = true;

Process procesoSC[id:0..i-1]{
    llegada.push(id);
    while (siguiente != id) skip;
    SCEnter
        // Usa la sección crítica
        libre = true;
    SCEnd
}

Process coordinador{
    while (true){
        while (empty(llegada)) skip;
        siguiente = llegada.pop();
        
        while (libre) skip;
        libre = false;
    }
}
```