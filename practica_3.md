### 1) Implementar el acceso a una base de datos de solo lectura que puede atender a lo sumo 5 consultas simultáneas.
```java
Monitor baseDatos {
    int disponibles = 5;
    cond esperar;
    int dormidos = 0;

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
            dormidos--;
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
    Fotocopiar(documento);
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
    Fotocopiar(documento);
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
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.llegada(id);
    Fotocopiar(documento);
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

    Procedure liberar(){
        siguiente++;
        signal espera[siguiente];
    }
}
```
## 2.e)
```java
Process persona[id:0..N-1]{
    text documento;
    fotocopiadora.llegada();
    Fotocopiar(documento);
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
        signal(empleado);
        esperando++;
        wait(espera);
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
    Fotocopiar(documento);
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
        signal(empleado);
        cola.push(id);
        wait(espera);
        impresora = impresoraAsignada[id];
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
    Corralon.llegada(lista,comp);
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

    Procedure llegada(lista: IN text,comp: OUT text){
        if (not libre){
            esperando++; wait(espera);
        } else {
            libre = false;
        }
        listaAct = lista;
        hayLista = true;
        signal(empleado);
        wait(esperarComprobante);
        comp = compAct;
        signal(recibiComp);
        if (esperando > 0) {
            signal espera; esperando--;
        } else libre = true;
    }

    Procedure esperarCliente(lista: OUT text){
        if (not hayLista) wait(empleado);
        lista = listaAct;
        hayLista = false;
    }

    Procedure entregarComprobante(comp: IN text){
        compAct = comp; signal(esperarComprobante);
        wait(recibiComp);
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

    Aula.llegada(id,miGrupo);
    // Hace práctica
    Aula.entregar(miGrupo);
    Aula.verNota(miGrupo,miNota);
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

    Procedure entregar(grupo: IN int){
        entregas.push(grupo);
        signal(hayEntrega);
        wait(correccion[grupo]);
    }

    Procedure verNota(grupo: IN int; miNota: OUT int){
        miNota = notas[grupo];
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
        entregasGrupo[grupoAct]++;
        if (entregasGrupo[grupoAct] == 2){
            notas[grupoAct] = i;
            signal_all(correccion[grupoAct]);
        }
    }
}
```
#
### 5) En un entrenamiento de futbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).
```java
Process jugador[id:0..19]{
    int equipo = darEquipo();
    
    entrenamiento[equipo].llegada(cancha);
    cancha[cancha].jugarPartido();
    delay(50);
}

Monitor entrenamiento[id:0..3]{

    int miembros = 0;
    int cancha;
    cond espera;

    Procedure llegada(c: OUT int){
        miembros++;
        if (miembros == 5){
            administrador.equipoCompleto(id,cancha);
        } else {
            wait(espera);
        }
        signal_all(espera);
        c = cancha;
    }
}

Monitor administrador {

    int equiposCompletos;
    cond equipos;
    boolean[2] canchasLibres ([2] = true);
    int[2] equiposCancha ([2] = 0);

    Procedure equipoCompleto(id: IN int, cancha: OUT int){
        equiposCompletos++;
        if (equiposCompletos < 2){
            cancha = 1;
        } else {
            cancha = 2;
        }
    }
}

// No sé si está bien implementado el jugarPartido (PREGUNTAR)
Monitor cancha[id:0..1]{

    int equipos = 0;
    cond jugar;

    Procedure jugarPartido(){
        jugadores++;
        if (jugadores < 10){
            wait(jugar);
        } else {
            delay(50);
            signal_all(jugar);
        }
    }

}
```
#
### 6) En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el monto total juntado por su grupo.
#### Nota: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.
```java
Process persona[id:0..19]{
    int miEquipo = ...;
    int puntos = 0; int puntosEquipo;

    equipo[miEquipo].llegada();
    for (int i=0;i<15;i++){
        puntos += Moneda();
    }

    equipo[miEquipo].sumarPuntos(puntos,puntosEquipo);
}

Monitor equipo[id:0..4]{
    
    cond miembros;
    int puntosTotales = 0;
    int llegaron = 0;
    int terminaron = 0;

    Procedure llegada(){
        llegaron++;
        if (llegaron == 4){
            signal_all(miembros);
        } else {
            wait(miembros)
        }
    }

    Procedure sumarPuntos(puntos: IN int, final: OUT int){
        puntosTotales += puntos;
        terminaron++;
        if (terminaron == 4){
            signal_all(miembros);
        } else {
            wait(miembros);
        }
        final = puntosTotales;
    }
}
```
#
### 7) Se debe simular una maratón con C corredores donde en la llegada hay UNA máquinas expendedoras de agua con capacidad para 20 botellas. Además existe un repositor encargado de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira.
#### Nota: maximizar la concurrencia; mientras se reponen las botellas se debe permitir que otros corredores se encolen. 
```java
// No sé como hacer para que los corredores se encolen mientras se reponen botellas; quizás con un monitor aparte (PREGUNTAR)
Process corredor[id:0..C-1]{
    maquina.usarMaquina();
}

Process repositor {
    while (true){
        maquina.esperarReponer();
    }
}

Monitor maquina {
    int botellas = 20; int idAux;
    boolean libre = true;
    int esperando = 0;
    
    cond personas;
    cond reponer;
    cond esperandoBotellas;

    Procedure usarMaquina(){
        if (not libre){
            esperando++;
            wait(personas);
        }
        else {
            libre = false;
        }

        if (botellas == 0){
            signal(reponer);
            wait(esperandoBotellas);
        }
        botellas--;

        if (esperando > 0){
            esperando--;
            signal(personas);
        } else {
            libre = true;
        }
    }

    Procedure esperarReponer(){
        if (botellas != 0) wait(reponer);
        botellas = 20;
        signal(esperandoBotellas);
    }
}
```

```java
// Solución usando un monitor aparte (PREGUNTAR)
Process corredor[id:0..C-1]{
    cola.llegada();
    maquina.usar();
    cola.salida();
}

Process repositor {
    while (true){
        maquina.reponer();
    }
}

Monitor cola {
    boolean libre = true;
    int esperando = 0;

    cond espera;

    Procedure llegada(){
        if (not libre) {
            esperando++;
            wait(espera);
        } else {
            libre = false;
        }
    }

    Procedure salida(){
        if (esperando > 0){
            signal(espera); esperando--;
        } else {
            libre = true;
        }
    }
}

Monitor maquina {
    int botellas = 20;
    cond reponer;

    Procedure usar(){
        if (botellas == 0){
            signal(reponer);
        }
        botellas--;
    }

    Procedure reponer(){
        if (botellas > 0) wait(reponer);
        botellas = 20;
    }
}
```