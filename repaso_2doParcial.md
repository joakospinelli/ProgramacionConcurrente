# PMA

### Resolver con PMA (Pasaje de Mensajes ASINCRÓNICOS) el siguiente problema. En una oficina hay 3 empleados y P personas que van para ser atendidas para realizar un trámite. Cuando una persona llega espera hasta ser atendido por cualquiera de los empleados, le indica el trámite a realizar y espera a que le den el resultado. Los empleados atienden las solicitudes en orden de llegada; si no hay personas esperando, durante 5 minutos resuelven trámites pendientes (simular el proceso de resolver trámites pendientes por medio de un delay).
#### Nota: no generar demora innecesaria; cada persona hace sólo un pedido y termina; los empleados no necesitan terminar su ejecución.

```java
chan llegada(int id);
chan[P] empleado(int idEmpleado);
chan[P] resultado(text resultado);
chan[3] tramite(text tramite);
chan estoyLibre(int idEmpleado);
chan[3] empleadoSiguiente(int idCliente);

Process persona[id:0..P-1]{
    int idEmpleado;
    text miTramite;
    text miResultado;

    send llegada(id);
    receive empleado[id](miEmpleado);
    send tramite[miEmpleado](miTramite);
    receive resultado[id](miResultado);
}

Process empleado[id:0..2]{
    int idAux;
    text tramiteAux;
    while (true){
        send estoyLibre(id);
        receive empleadoSiguiente[id](idAux);
        if (idAux != -1){
            send empleado[idAux](id);
            receive tramite[id](tramiteAux);
            // Hace el trámite
            send resultado[idAux](tramiteAux);
        } else {
            delay(300);
        }
    }
}

Process coordinador {
    int idPersona;
    int idEmpleado;
    while (true){
        receive estoyLibre(idEmpleado);
        if (not empty(llegada)){
            receive llegada(idPersona);
        } else {
            idPersona := -1;
        }
        send empleadoSiguiente[idEmpleado](idPersona);
    }
}

```

### Resolver con PMA (Pasaje de Mensajes ASINCRÓNICOS) el siguiente problema. En un negocio hay 5 empleados que atienden a N personas que van a pedir un presupuesto, de acuerdo al orden de llegada. Cuando el cliente sabe que empleado lo va a atender le entrega el listado de productos que necesita, y luego el empleado le entrega el presupuesto del mismo.
#### Nota: maximizar la concurrencia. Existe una función HacerPresupuesto(lista) que simula la elaboración del presupuesto por parte de los empleados

```java
chan llegada(int id);
chan[N] empleado(int idEmpleado);
chan[N] presupuesto(double presupuesto);
chan[5] lista(text listaProductos);

Process empleado[id:0..4]{
    int idAux;
    text listaProductos;
    double presupuesto;

    while (true){
        receive llegada(idAux);
        send empleado[idAux](id);
        receive lista[id](listaProductos);
        presupuesto = hacerPresupuesto(lista);
        send presupuesto[idAux](presupuesto);
    }
}

Process persona[id:0..N-1]{
    text lista;
    double presupuesto;
    int idEmpleado;

    send llegada(id);
    receive empleado[id](idEmpleado);
    send lista[idEmpleado](lista);
    receive presupuesto[id](presupuesto);
}

```

### Resolver con PASAJE DE MENSAJES ASINCRÓNICOS (PMA) el siguiente problema. Se debe simular la atención en un peaje con 7 cabinas para atender a N vehículos (algunos de ellos son ambulancias). Cuando el vehículo llega al peaje se dirige a la cabina con menos vehículos esperando y se queda ahí hasta que lo terminan de atender y le dan el ticket de pago. Las cabinas atienden a los vehículos que van a ella de acuerdo al orden de llegada pero dando prioridad a las ambulancias; cuando terminan de atender a un vehículo le dan el ticket de pago. Nota: maximizar la concurrencia.

```java

chan llegada(id);
chan salida(int idCabina);
chan avisoCoordinador();

chan[7] avisoCabina();
chan[7] liberarCabina();
chan[7] llegadaVehiculo(int id);
chan[7] llegadaAmbulancia(int id);

chan[N] tickets(text ticket);
chan[N] miCabina(int idCabina);

Process cabina[id:0..6]{
    int idAux;
    text ticket;
    while (true){
        receive avisoCabina[id]();
        if (not empty llegadaAmbulancia[id]){
            receive llegadaAmbulancia[id](idAux);
        else {
            receive llegadaVehiculo[id](idAux);
        }
        ticket = generarTicket();
        send tickets[idAux](ticket);
        receive liberarCabina[id]();
        }
    }
}

Process vehiculo[id:0..N-1]{
    boolean soyAmbulancia = ...;
    text miTicket;
    int idCabina;

    send llegada(id);
    send avisoLlegada();
    receive miCabina[id](idCabina);
    if (soyAmbulancia){
        send llegadaAmbulancia[idCabina](id);
    } else {
        send llegadaVehiculo[idCabina](id);
    }
    send avisoCabina[idCabina]();
    receive tickets[id](miTicket);
    send liberarCabina[idCabina]();
    send salida(idCabina);
    send avisoCoordinador();
}

Process coordinador {
    int[7] cabinas ([7] = 0);
    int idAux, cabinaAux;

    while (true){
        receive avisoCoordinador();
        if (empty salida && not empty llegada){
            receive llegada(idAux);
            cabinaAux = cabinas.indexOf(cabinas.min());
            send miCabina[idAux](cabinaAux);
            cabinas[cabinaAux]++;
        }
        * (not empty salida) -> {
            receive salida(cabinaAux);
            cabinas[cabinaAux]--;
        }
    }
}
```

# PMS

### Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. En una carrera hay C corredores y 3 Coordinadores. Al llegar los corredores deben dirigirse a los coordinadores para que cualquiera de ellos le dé el número de "chaleco" con el que van a correr y luego se va. Los coordinadores atienden a los corredores de acuerdo al orden de llegada (cuando un coordinador está libre atiende al primer corredor que está esperando).
#### Nota: maximizar la concurrencia.

```java
Process corredor[id:0..C-1]{
    int miNumero;

    administrador!llegada(id);
    coordinador[*]?chaleco(miNumero);
}

Process coordinador[id:0..2]{
    int siguiente, nroChaleco;
    while (true){
        administrador!pedirNumero(id);
        administrador?atender(siguiente);
        nroChaleco = Random();
        corredor[siguiente]!chaleco(nroChaleco);
    }
}

Process administrador {
    int idCorredor, idCoordinador;
    queue corredores;
    while (true){
        do corredor[*]?llegada(idCorredor) -> corredores.push(idCorredor);
        * (not empty(corredores)); coordinador[*]?pedirNumero(idCoordinador) -> coordinador[idCoordinador]!atender(corredores.pop());
        od
    }
}
```

### Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. Simular la atención de una estación de servicio con un único surtidor que tiene un empleado que atiende a los N clientes de acuerdo al orden de llegada. Cada cliente espera hasta que el empleado lo atienda y le indica qué y cuánto cargar; espera hasta que termina de cargarle combustible y se retira.
#### Nota: cada cliente carga combustible sólo una vez; todos los procesos deben terminar.

```java
Process cliente[id:0..N-1]{
    String combustible; int cantidad;

    coordinador!llegada(id);
    empleado?consultar();
    empleado!pedir(combustible,cantidad);
    empleado?terminar();
}

Process empleado {
    boolean fin = false;
    String combustible;
    int idAux, cantidad;

    while (!fin){
        coordinador!siguiente();
        coordinador?atender(idAux,fin);
        cliente[idAux]!consultar();
        cliente[idAux]?pedir(combustible,cantidad);
        // Carga combustible
        cliente[idAux]!terminar();
    }
}

Process coordinador {
    queue clientes;
    int idAux;
    int cant = 0;
    boolean fin = false;

    while (!fin){
        * cliente[*]?llegada(idAux) -> clientes.push(idAux);
        * (not empty(clientes)) -> empleado?siguiente() ->  {
            cant++;
            if (cant == N) fin = true;
            empleado!atender(clientes.pop(),fin);
        }
    }
}
```

### Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar el uso del mismo. A su vez hay P personas que van a la exposición y solicitan usar el simulador, cada una de ellas espera a que el empleado lo deje acceder, lo usa por un rato y se retira para que el empleado deje pasar a otra persona. El empleado deja usar el simulador a las personas respetando el orden en que hicieron la solicitud.
#### Nota: cada persona usa sólo una vez el simulador.

```java

Process persona[id:0..P-1]{
    empleado!llegada(id);
    empleado?usar();
    // Usa el simulador
    empleado!salida();
}

Process empleado{
    boolean libre = true;
    int idAux;
    queue personas;

    // Podría hacerse todo adentro del mismo DO en vez de tener los IF ?
    do persona[*]?llegada(idAux) -> {
        if !(libre) personas.push(idAux);
        else {
            libre = false;
            persona[idAux]!usar();
        }
    }
    * persona[*]?salida() -> {
        if (personas.empty()) libre = true;
        else persona[personas.pop()]!usar();
    }
}

```

# ADA

### Resolver el siguiente problema ADA. Simular el con funcionamiento de un Entrenamiento de Básquet donde hay 20 jugadores y UN entrenador. El entrenador debe distribuir a los jugadores en 2 canchas. Cuando un jugador llega el entrenador le indica la cancha a la cual debe ir para que se dirija a ella y espere hasta que lleguen los 10; en ese momento comienzan a jugar el partido que dura 40 minutos. Cuando ambos partidos han terminado, el entrenador les da a los 20 jugadores una charla de 10 minutos y luego todos se retiran. El entrenador asigna el número de cancha en forma cíclica 1, 2, 1, 2 y así sucesivamente.

```ada

TASK TYPE jugador;

TASK entrenador IS
    ENTRY llegada(cancha: OUT integer);
    ENTRY darCharla;
    ENTRY salir;
END entrenador;

TASK TYPE cancha IS
    ENTRY llegada;
    ENTRY iniciarPartido;
    ENTRY terminarPartido;
END cancha;

jugadores = array[0..19] of jugador;
canchas = array[0..1] of cancha;

TASK BODY entrenador IS
BEGIN
    for i in 0..19 LOOP
        ACCEPT llegada(cancha: OUT integer) do
            cancha := (i MOD 2) + 1;
        END llegada;
    END LOOP;

    FOR i in 0..1 LOOP
        ACCEPT darCharla;
    END LOOP;
    delay(10);

    for i in 0..19 LOOP
        ACCEPT salir;
    END LOOP;
END entrenador;

TASK BODY cancha IS
BEGIN
    for i in 0..9 LOOP
        ACCEPT llegada;
    END LOOP;

    for i in 0..9 LOOP
        ACCEPT iniciarPartido;
    END LOOP;

    delay(40);
    for i in 0..9 LOOP
        ACCEPT terminarPartido;
    END LOOP;

    entrenador.darCharla;
END cancha;

TASK BODY jugador IS
VAR
    cancha: integer;
BEGIN
    entrenador.llegada(cancha);
    canchas[cancha].llegada;
    canchas[cancha].iniciarPartido;
    canchar[cancha].terminarPartido;
    entrenador.salir;
END jugador;
```

