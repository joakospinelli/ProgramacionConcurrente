### 1) Se requiere modelar un puente de un solo sentido, el puente solo soporta el peso de 5 unidades de peso. Cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones).
#### a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.
#### b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.

### 1.a)
```ada
TASK TYPE auto;
TASK TYPE camioneta;
TASK TYPE camion;

autos = array[0..A-1] of auto;
camionetas = array[0..B-1] of camioneta;
camiones = array[0..C-1] of camion;

TASK puente IS
    ENTRY entraAuto();
    ENTRY entraCamioneta();
    ENTRY entraCamion();
    ENTRY saleAuto();
    ENTRY saleCamioneta();
    ENTRY saleCamion();
END puente;

TASK BODY auto IS
    puente.entraAuto();
    puente.saleAuto();
END auto;

TASK BODY camioneta IS
    puente.entraCamioneta();
    puente.saleCamioneta();
END camioneta;

TASK BODY camion IS
    puente.entraCamion();
    puente.saleCamion();
END camion;

TASK BODY puente IS
VAR
    peso: integer;
BEGIN
    peso := 0;
    WHILE (true) LOOP
        SELECT
            WHEN (peso + 1 <= 5) -> ACCEPT entraAuto(); peso += 1; END entraAuto();
            OR WHEN (peso + 2 <= 5) -> ACCEPT entraCamioneta(); peso += 2; END entraCamioneta();
            OR WHEN (peso + 3 <= 5) -> ACCEPT entraCamion(); peso += 3; END entraCamion();
            OR ACCEPT saleAuto(); peso -= 1; END saleAuto();
            OR ACCEPT saleCamioneta(); peso -= 2; END saleCamioneta();
            OR ACCEPT saleCamion(); peso -= 3; END saleCamion();
        END SELECT;
    END LOOP;
END puente;
```

### 1.b)

```ada
TASK TYPE auto;
TASK TYPE camioneta;
TASK TYPE camion;

autos = array[0..A-1] of auto;
camionetas = array[0..B-1] of camioneta;
camiones = array[0..C-1] of camion;

TASK puente IS
    ENTRY entraAuto();
    ENTRY entraCamioneta();
    ENTRY entraCamion();
    ENTRY saleAuto();
    ENTRY saleCamioneta();
    ENTRY saleCamion();
END puente;

TASK BODY auto IS
    puente.entraAuto();
    puente.saleAuto();
END auto;

TASK BODY camioneta IS
    puente.entraCamioneta();
    puente.saleCamioneta();
END camioneta;

TASK BODY camion IS
    puente.entraCamion();
    puente.saleCamion();
END camion;

TASK BODY puente IS
VAR
    peso: integer;
BEGIN
    peso := 0;
    WHILE (true) LOOP
        SELECT
            WHEN (peso + 1 <= 5) AND (entraCamion()'count == 0 OR (entraCamion()'count > 0 AND peso + 3 + 1 > 5)) -> ACCEPT entraAuto(); peso += 1; END entraAuto();
            OR WHEN (peso + 2 <= 5) AND (entraCamion()'count == 0 OR ( entraCamion()'count > 0 AND  peso + 3 + 2 > 5)) -> ACCEPT entraCamioneta(); peso += 2; END entraCamioneta();
            OR WHEN (peso + 3 <= 5) -> ACCEPT entraCamion(); peso += 3; END entraCamion();
            OR ACCEPT saleAuto(); peso -= 1; END saleAuto();
            OR ACCEPT saleCamioneta(); peso -= 2; END saleCamioneta();
            OR ACCEPT saleCamion(); peso -= 3; END saleCamion();
        END SELECT;
    END LOOP;
END puente;
```

### 2) Se quiere modelar la cola de un banco que atiende un solo empleado, los clientes llegan y si esperan más de 10 minutos se retiran.
```ada
TASK empleado IS

END empleado;

TASK TYPE cliente;

clientes = array[0..C-1] of clientes;


TASK BODY cliente IS
    SELECT empleado.atender();
    OR DELAY(600);
        NULL;
END cliente;

TASK BODY empleado IS
    WHILE (true) LOOP
        ACCEPT atender();
            // Atiende al cliente
        END atender();
    END LOOP;
END empleado;
```

### 3) Se dispone de un sistema compuesto por 1 central y 2 procesos. Los procesos envían señales a la central. La central comienza su ejecución tomando una señal del proceso 1, luego toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.
#### El proceso 1 envía una señal que es considerada vieja (se deshecha) si en 2 minutos no fue recibida.
#### El proceso 2 envía una señal, si no es recibida en ese instante espera 1 minuto y vuelve a mandarla (no se deshecha).

```ada

```

### 4) En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos.
#### Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica.
#### Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones.
#### El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras.
```ada
TASK TYPE paciente;

TASK TYPE enfermera;

TASK medico IS
    ENTRY atenderEnfermo(estado: OUT boolean);
    ENTRY atenderEnfermo(estado: OUT boolean);
    ENTRY atenderEnfermera();
END medico;

TASK anotador IS
    ENTRY verNota(nota: OUT text);
    ENTRY dejarNota(nota: IN text);
END

enfermeras = array[0..E-1] of enfermera;
pacientes = array[0..P-1] of paciente;

TASK BODY medico IS
VAR
    estadoAux: boolean;
    notaAux: text;
BEGIN
    WHILE (true) LOOP
        SELECT ACCEPT atenderEnfermo(estadoAux); estadoAux := true; END atenderEnfermo(estadoAux);
            OR WHEN (atenderEnfermo'count = 0) -> ACCEPT atenderNormal(estadoAux); estadoAux := true; END atenderNormal(estadoAux);
            OR WHEN (atenderEnfermo'count = 0 AND atenderNormal'count = 0) -> ACCEPT atenderEnfermera(); END atenderEnfermera();
            OR WHEN (atenderEnfermo'count = 0 AND atenderNormal'count = 0 AND atenderEnfermera'count = 0) -> ACCEPT enviarNota(notaAux); DELAY(x); END enviarNota;
        END SELECT;
    END LOOP;
END

TASK BODY secretaria IS
VAR
    text nota;
BEGIN
    WHILE (true) LOOP
        anotador.verNota(notaAux: OUT text);
        medico.enviarnota(nota);
    END LOOP;
END;

TASK BODY anotador IS
VAR
    queue notas;
BEGIN
    WHILE (true) LOOP
        SELECT ACCEPT dejarNota(nota); notas.push(nota); END dejarNota(nota);
        OR
            WHEN (notas.size > 0) -> ACCEPT verNota(notas.pop()); END verNota(nota);
    END LOOP;
END;

TASK BODY enfermera IS
VAR
    nota: text;
BEGIN
    WHILE (true) LOOP
        SELECT medico.atenderEnfermera();
        ELSE
            // Escribe nota
            anotador.dejarNota(nota: IN text);
        END SELECT;
    END LOOP;

TASK BODY paciente IS
VAR
    estaEnferma: boolean;
    atendido: boolean;
    intentos: integer;
BEGIN
    estaEnferma := ...;
    atendido := false; intentos := 0;

    WHILE (NOT atendido AND intentos < 3) LOOP
        intentos += 1;
        if (estaEnferma) then begin
            SELECT medico.atenderEnfermo(atendido: OUT boolean);
            OR DELAY(300)
                DELAY(600);
            END SELECT;
        end
        else begin
            SELECT medico.atenderNormal(atendido: OUT boolean);
            OR DELAY(300)
                DELAY(600);
            END SELECT;
        end; 
    END LOOP;
END paciente;
```

### 5) En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos. Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento).

```ada
TASK TYPE usuario;

TASK servidor IS
    ENTRY enviarDocumento(doc: IN text, listo: OUT boolean);
END servidor;

usuarios = array[0..U-1] of usuario;

TASK BODY usuario IS
VAR
    doc: text; listo: boolean;
BEGIN
    listo := false;
    while (NOT listo) LOOP
        write(doc);
        SELECT servidor.enviarDocumento(doc,listo);
        OR DELAY(120);
            DELAY(60);
        END SELECT;
    END LOOP;
END usuario;

TASK BODY servidor IS
VAR
    doc: text; estado: boolean;
BEGIN
    WHILE (true) LOOP
        ACCEPT enviarDocumento(doc: IN text, listo: OUT boolean);
            listo = validarDocumento(doc);
        END enviarDocumento;
    END LOOP;
END servidor;
```

### 6) En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto.
#### Nota: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada
```ada
TASK TYPE persona IS
    ENTRY enviarId(miId: IN integer);
    ENTRY empezar();
    ENTRY ganador(idGanador: OUT integer);
END persona;

TASK TYPE equipo IS
    ENTRY enviarId(miId: IN integer);
    ENTRY llegada(id: IN integer);
    ENTRY terminar(valorMonedas: IN integer);
END equipo;

TASK coordinador IS
    ENTRY verResultados(puntos: IN integer,idEquipo: IN integer);
END coordinador;

personas = array[0..19] of persona;
equipos = array[0..4] of equipo;

TASK BODY equipo IS
VAR
    miembros, puntos, i: integer;
    idMiembros: array[0..3] of integer;
BEGIN
    ACCEPT enviarId(miId: IN integer);
        id := miId;
    END enviarId;
    
    FOR i in 0..3 LOOP
        ACCEPT llegada(id: IN int); idMiembros[miembros] := id; END llegada;
    END LOOP;

    for i in 0..3 LOOP
        personas[idMiembros[i]].empezar();
    END LOOP;

    WHILE (miembros < 4) LOOP
        ACCEPT terminar(valorMonedas: IN integer); puntos += valorMonedas; miembros += 1; END terminar;
    END LOOP;

    coordinador.verResultados(puntos,id);
END equipo;

TASK BODY coordinador IS
VAR
    i, mejorEquipo, mejorPuntaje: integer; equipos: array[0..4] of int;
BEGIN
    mejorPuntaje := -1;
    for i in 0..4 LOOP
        ACCEPT verResultados(puntos: IN integer;id: IN integer) do
            equipos[id] := puntos;
            if (puntos > mejorPuntaje) THEN BEGIN
                mejorPuntaje := puntos;
                mejorEquipo := id;
            END;
        END verResultados;
    END LOOP;


    for i in 0..19 LOOP
        personas[id].ganador(mejorEquipo);
    END LOOP;
END coordinador;

TASK BODY persona IS
VAR
    equipo, monedas, idGanador, valorMonedas, id: integer;
BEGIN
    ACCEPT enviarId(miId: IN integer);
        id := miId;
    END enviarId;

    equipo := ...;
    valorMonedas := 0;
    monedas := 0;
    idGanador := -1;
    equipos[equipo].llegada(id);
    ACCEPT empezar(); END empezar();
    WHILE (monedas < 15) LOOP
        valorMonedas += Moneda();
        monedas++;   
    END LOOP;
    equipos[equipo].terminar(valorMonedas);
    ACCEPT ganador(idGanador: OUT int); END ganador;
END persona;

VAR
    i: integer;
BEGIN
    for i in 0..19 LOOP
        personas[i].enviarId(i);
    END LOOP;

    for i in 0..4 LOOP
        grupos[i].enviarId(i);
    END LOOP;
END body;
```

### 7) Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia. A su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo.
#### Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria.

```ada
TASK TYPE servidor IS
    ENTRY enviarMuestra(test: IN text);
END servidor;

TASK especialista IS
    ENTR enviarResultados(codigo: IN integer,similitud: IN integer);

servidores = array[0..7] of servidor;

TASK BODY servidor IS
VAR
    muestra: text;
    codigo, similitud: integer;
BEGIN

    ACCEPT enviarMuestra(test: IN text); muestra := test; END enviarMuestra;
    Buscar(test,codigo,similitud);
    especialista.enviarResultados(codigo,similitud);
END servidor;

TASK BODY persona IS
VAR
    i, codigoMayor: integer;
    huella: text;
    codigos = array[0..7] of integer;
    similitudes = array[0..7] of integer;
BEGIN
    text huella;
    for i:=0 to 7 LOOP
        servidor[i].enviarMuestra(huella);
    END LOOP;

    for i:=0 to 7 LOOP
        ACCEPT enviarResultados(codigo: IN integer,similitud: IN integer) do
            codigos[i] := codigo; similitudes[i] := similitud;
        END enviarResultados;
    END LOOP;

    codigoMayor := codigos[similitudes.indexOf(similitudes.max())];
END persona;
```

### 8) Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3 camiones. Hay P personas que hacen continuos reclamos hasta que uno de los camiones pase por su casa. Cada persona hace un reclamo, espera a lo sumo 15 minutos a que llegue un camión y si no vuelve a hacer el reclamo y a esperar a lo sumo 15 minutos a que llegue un camión y así sucesivamente hasta que el camión llegue y recolecte los residuos; en ese momento deja de hacer reclamos y se va. Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos ha hecho sin ser atendido.
#### Nota: maximizar la concurrencia.


```ada
TASK TYPE camion IS
    ENTRY enviarId(miId: IN integer);
END camion;

TASK TYPE persona IS
    ENTRY enviarId(miId: IN integer);
END persona;

TASK empresa;

camiones = array[0..2] of camion;

personas = array[0..P-1] of persona;

TASK BODY camion IS
VAR
    clienteAtender, id: integer;
BEGIN
    ACCEPT enviarId(miId: IN integer);
        id := miId;
    END enviarId;
    WHILE (true) LOOP
        empresa.estoyLibre(id);
        ACCEPT recibirReclamo(cliente: IN int); clienteAtender := cliente; END recibirReclamo;
        // Va y recolecta residuos
        empresa.terminarReclamo(clienteAtender,id);
    END LOOP;

TASK BODY empresa IS
VAR
    estadoCamiones = array[0..2] of boolean;
    camionesLibres,camionAct: integer;
    atenderUsuarios = queue of integer;
BEGIN
    camionesLibres := 3;
    WHILE (true) LOOP
        SELECT // Debería ser un array en vez de una cola
            ACCEPT enviarReclamo(id: IN integer) DO
                if (camionesLibres > 0){
                    camionAct := estadoCamiones.indexOf(true);
                    camiones[camionAct].recibirReclamo(id);
                    estadoCamiones[camionAct] := false;
                    camionesLibres -= 1;
                } else {
                    if NOT (atenderUsuarios.contains(id))
                        atenderUsuarios.push(id);
                }
            END enviarReclamo;
        OR
            ACCEPT terminarReclamo(cliente: IN integer,camion: IN integer) DO
                personas[cliente].reclamoAtendido();
                if (not empty(atenderUsuarios)){
                    camiones[camion].recibirReclamo(atenderUsuarios.pop());
                } else {
                    camionesLibres++;
                    estadoCamiones[camion] := true;
                }
            END terminarReclamo;
        END SELECT;
    END LOOP;
END empresa;


END;

TASK BODY persona IS
VAR
    llego: boolean;
    id: integer;
BEGIN
    llego := false;
    ACCEPT enviarId(miId: IN integer);
        id := miId;
    END enviarId;
    WHILE (NOT llego) LOOP
        empresa.enviarReclamo(id)
        SELECT
            ACCEPT empresa.reclamoAtendido() DO
                llego := true;
            END reclamoAtendido;
        ELSE DELAY (900);
        END SELECT;
    END LOOP;
END;

BEGIN
    for i in 0..P-1 LOOP
        personas[i].enviarId(i);
    END LOOP;

    for i in 0..2 LOOP
        camiones[i].enviarId(i);
    END LOOP;
END;
```
