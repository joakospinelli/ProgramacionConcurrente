### 1) Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.
#### a. Implemente una solución que modele el acceso de las personas a un detector (es decir si el detector está libre la persona lo puede utilizar caso contrario debe esperar).
#### b. Modifique su solución para el caso que haya tres detectores.

## 1.a)
```java
sem mutex = 1;

Process persona[id:0..N-1]{
    P(mutex);
    // Pasa por el detector de metales
    V(mutex);
}
```

## 1.b)
```java
sem mutex = 3;

Process persona[id:0..N-1]{
    P(mutex);
    // Pasa por el detector de metales
    V(mutex);
}
```
#
### 2) Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.
```java
sem mutex = 5;
sem accesoCola = 1;

queue recursos;
Process proceso[id:0..N]{
    Recurso miRecurso;
    P(mutex);
        P(accesoCola);
        miRecurso = recursos.pop();
        V(accesoCola);
        // Usa el recurso

        P(accesoCola);
        recursos.push(miRecurso);
        V(accesoCola);
    V(mutex);
}
```
#
### 3) Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:
### * no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
### • no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.
### Indique si la solución presentada es la más adecuada. Justifique la respuesta. 
```java
// Respuesta presentada
sem: semaphoro := 6;
alta: semaphoro := 4;
baja: semaphoro := 5;

Process usuarioAlta [id:1..L]{
    P(sem);
    P(alta);
    // Usa la BD
    V(sem);
    V(alta);
}

Process usuarioBaja [id:1..K]{
    P(sem);
    P(baja);
    // Usa la BD
    V(sem);
    V(baja);
}
/* El problema con esta solución es que al hacer P(sem) primero está bloqueando a las demás prioridades, por lo que aunque la BD tuviese espacio para usuarios de la otra prioridad, estos no podría pasar. */

// Mi solución
Process usuarioPrioridad [id:1..N]{
    P(alta/baja);
    P(sem);
    // Usa la BD
    V(sem);
    V(alta/baja);
}
/* De esta manera, primero comprueba que pueda entrar según su prioridad, por lo que, en el caso de que no pueda acceder, deja el espacio disponible para los usuarios de la otra prioridad. */
```
#
### 4) Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:
#### a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.
#### b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
#### c) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).
#### d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.
#### e) Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar.

## 4.a)
```java
sem impresora = 1;

Process persona[id:0..N-1]{
    text documento;

    P(impresora);
    Imprimir(documento);
    V(impresora);
}
```

## 4.b)
```java
sem accesoCola = 1;
sem personas[N] ([N] = 0);
cola llegada(int);
boolean libre = true;

Process persona[id:0..N-1]{

    P(accesoCola);
    if (not libre){
        llegada.push(id);
        V(accesoCola);
        P(personas[id]);
    } else {
        libre = false;
        V(accesoCola);
    }

    Imprimir(documento);

    P(accesoCola);
    if (empty(llegada)){
        libre = true;
    } else {
        V(personas[llegada.pop()]);
    }
    V(accesoCola);
}
```
## 4.c)
```java
// No sé si debería usar mutex cuando pregunto siguiente != id (PREGUNTAR)
// Está bien usar el if/else para V(mutex) o simplemente lo hago afuera del if? (PREGUNTAR)
sem mutex = 1;
sem personas[N] ([N] = 0);
int siguiente = 0;

Process persona[id:0..N-1]{

    if(id > 0)
        P(personas[id])
    
    //SC
    V(personas[id+1])
}




    P(mutex);
    if (siguiente != id){
        V(mutex);
        P(personas[id]);
    } else {
        V(mutex);
    }

    Imprimir(documento);

    P(mutex);
    siguiente++;
    V(mutex);

    V(personas[siguiente]);
}
```
## 4.d)
```java
sem accesoCola = 1;
sem personasEsperando = 0;
sem listo = 0;

sem[N] personas ([N] = 0);
queue llegada(int);

Process persona[id:0..N-1]{
    text documento;

    P(accesoCola);
    llegada.push(id);
    V(accesoCola);

    V(personasEsperando);

    P(personas[id]);
    Imprimir(documento);

    V(listo);

}

Process coordinador {
    int idAct;
    while (true){
        P(personasEsperando);

        P(accesoCola);
        idAct = llegada.pop();
        V(accesoCola);

        V(personas[idAct]);

        P(listo);
    }
}
```

## 4.e)
```java
// No sé si es necesario el semáforo de acceso a impresoras (PREGUNTAR)
sem accesoCola = 1;
sem accesoImpresoras = 1;
sem personasEsperando = 0;
sem impresoras = 5;

sem[N] personas ([N] = 0);

int[N] impresoraAsignada;

queue llegada(int);

Process persona[id:0..N-1]{
    text documento;
    int miImpresora;

    P(accesoCola);
    llegada.push(id);
    V(accesoCola);

    V(personasEsperando);

    P(personas[id]);
    miImpresora = impresoraAsignada[id];
    Imprimir(documento);

    P(accesoImpresoras);
    impresorasLibres[miImpresora] = true;
    V(accesoImpresora);

    V(impresoras);
}

Process coordinador {
// Sería global    boolean[5] impresorasLibres ([5] = true);
    int idAct, impresoraAct;

    while (true){
        P(personasEsperando);

        P(impresoras);

        P(accesoCola);
        idAct = llegada.pop();
        V(accesoCola);

        P(accesoImpresoras);
        impresoraAct = impresorasLibres.indexOf(true);
        impresoraAsignada[idAct] = impresoraAct;
        impresorasLibres[impresoraAct] = false;
        V(accesoImpresoras);

        V(personas[idAct]);
    }
}
```

#
### 5) Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea, comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se queda esperando el puntaje del grupo, el cual está dado por todos aquellos que comparten el mismo enunciado. Cuando un grupo terminó, el profesor les otorga un puntaje que representa el orden en que se terminó esa tarea de las 10 posibles.
#### Nota: Para elegir la tarea suponga que existe una función elegir que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).

```java
// Acá hice lo mismo que en el 4.c con V(completar[tarea]) en el if/else (PREGUNTAR / CORREGIR)
int cant = 0;
int[10] entregaronTarea ([10] = 0);
int[10] notaTarea ([10] = 0);
queue tareasEntregadas(int);

sem elegir = 1;
sem accesoCola = 1;
sem hayTarea = 0;

sem[50] empezar ([50] = 0);
sem[10] completar ([10] = 1);

Process alumno[id:0..49]{
    int tarea = elegirTarea();

    P(elegir);
    cant++;
    if (cant == 50) {
        // Podría ser un sólo semaforo (no un arreglo) y hacer V 50 veces
        for (int i=0; i<50; i++){ V(empezar[i]); }
    }
    V(elegir);

    P(empezar[id]);

    // Hace tarea

    P(completar[tarea]);
    entregaronTarea[tarea]++;
    // No respeta el enunciado (el profesor debería verificar que sean 5)
    if (entregaronTarea[tarea] == 5){
        V(completar[tarea]);

        P(accesoCola);
        tareasEntregadas.push(tarea);
        V(accesoCola);

        V(hayTarea);
    } else {
        V(completar[tarea]);
    }

    P(tareaCorregida[tarea]);
}

Process profesor {
    int tareaAct;

    for (int i=10;i>0;i++){
        P(hayTarea);

        P(accesoCola);
        tareaAct = tareasEntregadas.pop();
        V(accesoCola);

        notaTarea[tareaAct] = i;
        for (int i=0;i<5;i++){
            V(tareaCorregida[tareaAct]);
        }
    }
}
```
#
### 6) A una empresa llegan E empleados y por día hay T tareas para hacer (T>E), una vez que todos los empleados llegaron empezaran a trabajar. Mientras haya tareas para hacer los empleados tomarán una y la realizarán. Cada empleado puede tardar distinto tiempo en realizar cada tarea. Al finalizar el día se le da un premio al empleado que más tareas realizó.
```java
int llegaron = 0;
int terminaron = 0;
int cantTareas = T;
int premio;
int[E] tareasCompletadas ([E] = 0);
queue tareas;

sem accesoCola = 1;
sem accesoLlegaron = 1;
sem accesoTerminaron = 1;
sem empezar = 0;
sem darPremio = 0;

Process empleado[id:0..E-1]{
    int miTarea;

    P(accesoLlegaron);
    llegaron++;
    if (llegaron == E){
        for (int i=0;i<E;i++){ V(empezar); }
    }
    V(accesoLlegaron);

    P(empezar);

    P(accesoCola);
    while (cantTareas > T){
        miTarea = tareas.pop();
        cantTareas--;
        V(accesoCola);

        completarTarea(miTarea);
        tareasCompletadas[id]++;

        P(accesoCola);
    }
    V(accesoCola);

    P(accesoTerminaron);
    terminaron++;
    if (terminaron == E){
        V(darPremio);
    }
    V(accesoTerminaron);
}

Process empresa {
    P(darPremio);
    premio = tareasCompletadas.indexOf(tareasCompletadas.max());
}
```
#
### 7) Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:
#### • Los carpinteros continuamente hacen marcos (cada marco es armando por un único carpintero) y los deja en un depósito con capacidad de almacenar 30 marcos.
#### • El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios.
#### • Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador)
```java
// Agregué semáforos para asegurarme de que haya un marco/vidrios cuando hago un pop() en el armador (PREGUNTAR)
sem cantMarcos = 30;
sem accesoMarcos = 1;
sem hayMarcos = 0;
queue marcos;

sem cantVidrios = 50;
sem accesoVidrios = 1;
sem hayVidrios = 0;
queue vidrios;

Process carpintero[id:0..3]{
    Marco marco;
    while (true){
        marco = hacerMarco();
        P(cantMarcos);
        P(accesoMarcos);
        marcos.push(marco);
        V(accesoMarcos);
        V(hayMarcos);
    }
}

Process vidriero {
    Vidrio vidrio;
    while (true){
        vidrio = hacerVidrio();
        P(cantVidrios);
        P(accesoVidrios);
        vidrios.push(vidrio);
        V(accesoVidrios);
        V(hayVidrios);
    }
}

Process armador[id:0..1]{
    Marco marco;
    Vidrio vidrio;
    while (true){
        P(hayMarcos);
        P(accesoMarcos);
        marco = marcos.pop();
        V(accesoMarcos);
        V(cantMarcos);

        P(hayVidrios);
        P(accesoVidrios);
        vidrio = vidrios.pop();
        V(accesoVidrio);
        V(cantVidrios);

        hacerVentana(marco,vidrio);
    }
}
```