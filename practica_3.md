### 1) Implementar el acceso a una base de datos de solo lectura que puede atender a lo sumo 5 consultas simultáneas.
```java
// El disponibles-- del entrar debería ir en un else? (PREGUNTAR)
Monitor baseDatos {
    int disponibles = 5;
    cond esperar;

    Procedure entrar(){
        if (disponibles == 0)
            dormidos++
            wait(esperar);
        else
            disponibles--;
    }

    Procedure liberar(){
        if(dormidos > 0)
            signal(esperar);
            dormidos--
        else
            disponibles++;
    }
}

Process persona[id:0..N-1]{
    baseDatos.entrar();
    // Usa BD
    baseDatos.liberar();
}
```

#
### 2) Existen N personas que deben fotocopiar un documento cada una. Resolver cada ítem usando monitores:
#### a) Implemente una solución suponiendo que existe una única fotocopiadora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Fotocopiar() que simula el uso de la fotocopiadora. Sólo se deben usar los procesos que representan a las Personas (y los monitores que sean necesarios).
#### b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
#### c) Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo a la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla).
#### d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1).
#### e) Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora.
#### f) Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuando puede usar una fotocopiadora, y cual debe usar

## 2.a)
```java
// No sé si fotocopiar() lo tiene que hacer la persona o la fotocopiadora (PREGUNTAR)
// Era de la persona 
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.usar(documento);
}

Monitor fotocopiadora {
    Procedure usar(text documento){
        Fotocopiar(documento);
    }
}
```
## 2.b)
```java
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.llegada();
    fotocopiadora.usar(documento);
    fotocopiadora.liberar();
}

Monitor fotocopiadora {
    boolean libre = true;
    int esperando = 0;
    cond espera;

    Procedure llegada(){
        if (not libre){
            esperando++; wait(espera);
        } else{
            libre = false;
        }
    }

    Procedure usar(text documento){
        Fotocopiar(documento);
    }

    Procedure liberar(){
        if (esperando > 0){
            esperando--; signal(espera);
        } else libre = true;
    }
}
```
## 2.c)
```java
Process persona[id:0..N-1]{
    text documento;
    int edad = ...;

    fotocopiadora.llegada(id,edad);
    fotocopiadora.usar(documento);
    fotocopiadora.liberar();
}

Monitor fotocopiadora {
    boolean libre = true;
    queue colaEspera;
    cond espera[N];

    Procedure llegada(int id,int edad){
        if (not libre){
            colaEspera.agregarOrdenado(id,edad);
            wait(espera[id]);
        } else{
            libre = false;
        }
    }

    Procedure usar(text documento){
        Fotocopiar(documento);
    }

    Procedure liberar(){
        int siguiente;

        if (not empty(colaEspera)){
            siguiente = colaEspera.pop();
            signal(espera[siguiente]);
        } else {
            libre = true;
        }
    }
}
```
## 2.d)
```java
// No sé si la persona tiene que esperar a que llegue X-1 o si cuando X-1 no está la puede usar (PREGUNTAR)
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.llegada(id);
    fotocopiadora.usar(documento);
    fotocopiadora.liberar();
}

Monitor fotocopiadora {
    int siguiente = 0;
    cond[N] espera;
    
    Procedure llegada(int id){
        if (id != siguiente){
            wait espera[id];
        }
    }

    Procedure usar(text documento){
        Fotocopiar(documento);
    }

    Procedure liberar(){
        siguiente++;
        signal espera[siguiente];
    }
}
```
## 2.e)
```java
// Este puede estar todo mal (PREGUNTAR)
// No sé si es necesario usar el if (esperando == 0) en la llegada (PREGUNTAR) No iba
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.llegada();
    fotocopiadora.usar(documento);
    fotocopiadora.liberar();
}

Process empleado {
    while (true){
        fotocopiadora.asignar();
    }
}

Monitor fotocopiadora {
    cond empleado;
    cond espera;
    cond terminar;
    int esperando = 0;

    Procedure llegada(){
        //if (esperando == 0){
            signal(empleado);
        //}
        esperando++;
        wait(espera);
    }

    Procedure usar(documento: IN text){
        Fotocopiar(documento);
    }

    Procedure liberar(){
        signal(terminar);
    }

    Procedure asignar(){
        if (esperando == 0) wait(empleado);
        signal(espera);
        esperando--;
        wait(terminar);
    }
}
```
## 2.f)
```java
Process persona[id:0..N-1]{
    text documento;
    int miImpresora;

    fotocopiadora.llegada(id,miImpresora);
    fotocopiadora.usar(documento);
    fotocopiadora.liberar(miImpresora);
}

Process empleado {
    while (true){
        fotocopiadora.asignar();
    }
}

Monitor fotocopiadora {
    cond empleado;
    cond espera;
    cond terminar;
    queue cola;
    boolean[10] estadoImpresoras ([10] = true);
    int[N] impresoraAsignada;
    int siguiente, impresoraAux;
    int ocupadas = 0;

    Procedure llegada(id: IN int; impresora: OUT int){
        //if (esperando == 0){
            signal(empleado);
        //}
        cola.push(id);
        wait(espera);
        impresora = impresoraAsignada[id];
    }

    Procedure usar(documento: IN text){
        Fotocopiar(documento);
    }

    Procedure liberar(impresora: IN int){
        estadoImpresoras[impresora] = true;
        signal(terminar);
        ocupadas--;
    }

    Procedure asignar(){
        if (empty(cola)) wait(empleado);
        if (ocupadas == 10) wait(terminar);

        siguiente = cola.pop();
        impresoraAux = estadoImpresoras.indexOf(true);
        impresoraAsignada[siguiente] = impresoraAux;
        estadoImpresoras[impresoraAux] = false;
        signal(espera);
        ocupadas++;
    }
}
```

#
### 3) En un corralón de materiales hay un empleado que debe atender a N clientes de acuerdo al orden de llegada. Cuando un cliente es llamado por el empleado para ser atendido, le da una lista con los productos que comprará, y espera a que el empleado le entregue el comprobante de la compra realizada.
```java
Process cliente[id:0..C-1]{
    text lista, comp;
    Corralon.llegada();
.....


    Corralon.pasar(lista,comp);
}

Process empleado{
    text lista, comp;
    while(true){
        Corralon.esperarCliente(lista);
        comp = generarComprobante(lista);
        Corralon.entregarComprobante(comp);
    }
}

Monitor Corralon {
    cond empleado;
    cond espera;
    cond esperarComprobante;

    int esperando = 0;
    boolean libre = true;
    text listaAct, compAct;

    Procedure llegada(){
        if (not libre){
            esperando++; wait(espera);
        } else {
            libre = false;
        }
    }

    Procedure pasar(lista: IN text;comp: OUT text){
        listaAct = lista;
        hayLista = true;
        signal(empleado);
        wait(esperarComprobante);
        comp = compAct;
        signal(recibiComp)
        if (esperando > 0) 
            signal(espera);
            esperando--;
        else 
            libre = true;
    }

    Procedure esperarCliente(lista: OUT text){
        if (not hayLista) wait(empleado);
        lista = listaAct;
    }

    Procedure entregarComprobante(comp: IN text){
        compAct = comp; signal(esperarComprobante);
        wait(recibiComp);
        hayLista = false;
    }
}
```
#
### 4) Suponga una comisión con 50 alumnos. Cuando los alumnos llegan forman una fila, una vez que están los 50 en la fila el jefe de trabajos prácticos les entrega el número de grupo (número aleatorio del 1 al 25) de tal manera que dos alumnos tendrán el mismo número de grupo (suponga que el jefe posee una función DarNumero() que devuelve en forma aleatoria un número del 1 al 25, el jefe de trabajos prácticos no guarda el número que le asigna a cada alumno). Cuando un alumno ha recibido su número de grupo comienza a realizar la práctica. Al terminar de trabajar, el alumno le avisa al jefe de trabajos prácticos y espera la nota. El jefe de trabajos prácticos, cuando han llegado los dos alumnos de un grupo les devuelve a ambos la nota del GRUPO (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1).
```java
// Se me había ocurrido agregar un procedure recibirGrupo() que le da su grupo al alumno con un parámetro de salida (PREGUNTAR) HAY QUE HACERLO (Se puede combinar con la llegada)
Process alumno[id:0..49]{
    int miGrupo;
    int miNota;

    Aula.llegada(id);
    Aula.recibirGrupo(id,miGrupo);
    // Hace práctica
    Aula.entregar(id);
    Aula.verNota(id,miNota);
}

Process profesor{
    Aula.esperarAlumnos();
    Aula.darGrupos();

    for (int i=25;i<0;i++){
        Aula.esperarEntrega();
        Aula.corregir(i);
    }
}

Monitor Aula {
    cond JTP;
    cond hayEntrega;

    cond[50] espera;
    cond[25] correccion;

    int[50] grupoAsignado;
    int[25] entregasGrupo ([25] = 0);
    int[25] notas;
    int cant = 0; int contNota = 25;
    int grupoAct;

    Procedure llegada(id: IN int,grupo: OUT int){
        cant++;
        if (cant == 50) signal(JTP);
        wait(espera[id]);
        grupo = grupoAct;
    }

    Procedure entregar(id: IN int){
        entregasGrupo[grupoAsignado[id]]++;
        if (entregasGrupo[grupoAsignado[id]] == 2){
            entregas.push(grupoAsignado[id]);
            signal(hayEntrega);
        }
        wait(correccion[grupoAsignado[id]]);
    }

    Procedure verNota(id: IN int; miNota: OUT int){
        miNota = notas[grupoAsignado[id]];
    }

    Procedure esperarAlumnos(){
        if (cant < 50) wait(JTP);
    }

    Procedure darGrupos(){
        for (int i=0;i<50;i++){
            grupoAsignado[i] = DarNumero();
            signal(espera[i]);
        }
    }

    Procedure esperarEntrega(){
        if (entregas.size == 0) wait(hayEntrega);
    }

    Procedure corregir(i: IN int){
        grupoAct = entregas.pop();
        notas[grupoAct] = i;
        contNota--;

        signal_all(correccion[grupoAct]);
    }
}
```
#
### 5) En un entrenamiento de futbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).
```java

// El jugador debería jugar el partido, el delay lo hace un proceso aparte
Process jugador[id:0..19]{
    int equipo = darEquipo();
    
    entrenamiento[equipo].llegada(cancha);
    // camina hacia 
    cancha[cancha].jugarPartido();
}

Monitor entrenamiento[id:0..3]{

    int miembros = 0;
    int cancha;
    cond espera;

    Procedure llegada(c OUT int){
        miembros++;
        if (miembros == 5){
            administrador.equipoCompleto(id,cancha);
        } else {
            wait(espera);
        }
        signal_all(espera);
        c = cancha
   
   
    }
}

Monitor administrador {

    int equiposCompletos;
    cond equipos;
    boolean[2] canchasLibres ([2] = true);
    int[2] equiposCancha ([2] = 0);

    Procedure equipoCompleto(id: IN int, cancha: OUT int){
        equiposCompletos++;
        if (equiposCompletos < 2){ // Si falta un equipo me duermo
            cancha = 1;
        } else {
            cancha = 2; // Si hay al menos un equipo para jugar despierto al primero
        }
    }
}

Monitor cancha[id:0..1]{

    int equipos = 0;
    cond jugadoresEsperando;

    Procedure jugarPartido(){
        
        jugadores++;
        if (jugadores == 10){
            signal(empezar)
        }
        wait(terminoPartido);
    }

    Procedure iniciarPartido(){
        if(jugadores < 10)
            wait(empezar)
        delay(50);
        signalall(terminoPartido)
    }
}
```
