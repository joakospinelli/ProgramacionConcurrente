# PMA

### 1) Suponga que N personas llegan a la cola de un banco. Para atender a las personas existen 2 empleados que van atendiendo de a una y por orden de llegada a las personas. 

```java
chan personas(int id);

Process empleado[id:0..1]{
    int idAux;
    while (true){
        receive personas(idAux);
    }
}

Process persona[id:0..N-1] {
    send personas(id)
}
```
#

### 2) Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada. Luego del pago, se les entrega un comprobante.


```java
chan buscarCaja(int id);
chan[P] miCaja(int caja);
chan[5] esperaCaja(int id);
chan[P] comprobantes(int id);
chan atendido(int caja);

Process caja[id:0..4]{
    int cliente;
    while (true){
        receive esperaCaja[id](cliente);
        send comprobante[cliente](generarComprobante());
    }
}

Process cliente[id:0..P-1]{
    int caja;
    text comprobante;

    send buscarCaja(id);
    receive miCaja[id](caja);
    send esperaCaja[caja](id);
    receive comprobante[id](comprobante);
    send atendido(caja);
}

Process coordinador{
    int[5] cajas ([5] = 0);
    int idAux;
    int cajaMin;

    while (true){
        if (not empty(buscarCaja) and empty(atendido))
            receive buscarCaja(idAux);
            cajaMin = cajas.min();
            send miCaja[idAux](cajaMin);
            caja[idAux]++;

        * not empty (atendido)
            receive atendido(idAux);
            cajas[idAux]--;
    }
}
```
#

### 3) Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).

```java
chan pedidos(int id,String pedido);
chan queHago(int id);
chan[2] siguiente(int id,String pedido);
chan cocinar(int id,String pedido);
chan[C] comida(object comida); // No se me ocurría de qué tipo hacer la comida

Process cocinero[id:0..1]{
    int idAux;
    String pedido;
    Object comida;

    while (true){
        receive cocinar(idAux,pedido);
        comida = cocinarPedido(pedido);
        send comida[idAux](comida);
    }
}

Process vendedor[id:0..2]{
    int idAux;
    String pedido;
    while (true){
        send queHago(id);
        receive siguiente[id](idAux,pedido);
        if (idAux == -1){
            delay(random(1,3)); // Repone heladera
        } else {
            send cocinar(id,pedido);
        }
    }
}

Process coordinador {
    int vendedor;
    String pedidioAux;
    while (true){
        receive queHago(vendedor);
        if (empty pedidos) {
            idAux = -1; pedidoAux = "VACIO";
        } else {
            receive pedidos(idAux,pedidoAux);
        }
        send siguiente[vendedor](idAux,pedidoAux);
    }
}

Process cliente[id:0..C-1]{
    String pedido;
    Object comida;

    send pedidos(id,pedido);
    receive comida[id](comida);
}
```
#

### 4) Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. A cada cliente se le entrega un ticket factura. 
#### Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

```java
chan llegada(int id);
chan[N] cabinas(int cabina);
chan[N] tickets(text ticket);
chan terminar(int id,int cabina);

Process empleado {
    int miCabina;
    text miTicket;

    send llegada(id);
    receive cabinas(miCabina);
    send terminar(id,miCabina);
    receive tickets(miTicket);
}

Process cliente[id:0..N-1]{
    boolean[10] cabinasLibres;
    int idAux, cabinaAux;
    text ticket;

    while (true){
        if (not empty(llegada) && cabinasLibres.indexOf(true) != -1 && empty(terminar)){
            receive llegada(idAux);
            cabinaAux = cabinasLibres.indexOf(true);
            send cabinas[idAux] (cabinaAux);
            cabinasLibres[cabinaAux] = false;
        } * (not empty(terminar)) {
            receive terminar(idAux,cabinaAux);
            Cobrar(idAux);
            ticket = generarTicket();
            send tickets[idaux](ticket);
        }
    }
}
```
#

### 5) Resolver la administración de las impresoras de una oficina. Hay 3 impresoras, N usuarios y 1 director. Los usuarios y el director están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada, pero siempre dando prioridad a los pedidos del director.
#### Nota: los usuarios y el director no deben esperar a que se imprima el documento.
```java
chan canalDirector(text doc);
chan canalUsuarios(text doc);
chan[3] siguiente(text doc);
chan estoyLibre(int id);

Process usuario[id:0..N-1]{
    text doc;
    while (true){
        doc = generarDocumento();
        send canalUsuarios(doc);
    }
}

Process director {
    text doc;
    while (true){
        doc = generarDocumento();
        send canalDirector(doc);
    }
}

Process impresora[id:0..2]{
    text doc;

    while (true){
        send estoyLibre(id);
        receive siguiente[id](doc);
        Imprimir(doc);
    }
}

Process coordinador{
    text doc;
    int idImpresora;

    while (true){
        receive estoyLibre(idImpresora);
        if !empty(canalDirector){
            receive canalDirector(doc);
        }
        * empty(canalDirector) && !empty(canalUsuarios){
            receive canalUsuarios(doc);
        }
        send siguiente[idImpresora](doc);
    }
}
```
#

# PMS

### 1) Suponga que existe un antivirus distribuido en él hay R procesos robots que continuamente están buscando posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y continúan buscando. Hay un proceso analizador que se encargue de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

```java
Process robot[id:0..R-1]{
    String direccion;
    while (true){
        direccion = buscarVirus();
        analizador!enviar(direccion);
    }
}

Process analizador {
    String direccion;
    while (true){
        robot[*]?enviar(direccion);
        hacerPrueba(direccion);
    }
}
```
#

### 2) En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado.

```java
Process preparador {
    text muestra;
    while (true){
        muestra = prepararMuestra();
        coordinador!enviarMuestra(muestra);
    }
}

Process coordinador {
    queue muestras;
    text muestraAux;
    while (true){
        if preparador?enviarMuestra(muestraAux); muestras.push(muestraAux);
        * (!empty(muestras)) armador?estoyLibre(); armador!enviarMuestra(muestra.pop());
        fi
    }
}

Process armador {
    text muestraAux, setAnalisis, archivo, resultAux;

    while (true){
        coordinador!estoyLibre();
        armador?enviarMuestra(muestraAux);
        
        setAnalisis = armarSet(muestraAux);
        analizador!enviarSet(setAnalisis);
        analizador?enviarResultados(resultAux);
        archivarResultados(archivo,resultAux);
    }
}

Process analizador {
    text setAnalisis;
    text resultados;

    while (true){
        armador?enviarSet(setAnalisis);
        resultados = analizar(setAnalisis);
        armador!enviarResultados(resultados);
    }
}
```
#

### 3) En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respectando el orden en que los alumnos van entregando.
### a) Considerando que P=1.
### b) Considerando que P>1.
### c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.
#### Nota: maximizar la concurrencia y no generar demora innecesaria.

#
### 3.a)
```java
Process alumno[id:0..N-1]{
    text examen;

    examen = hacerExamen();
    coordinador!entregar(examen,id);
    profesor?recibirNota(nota);
}

Process coordinador {
    cola examenes(text examen,int id);
    text examenAux; int id;

    while (true){
        if (!examenes.empty()); profesor?estoyLibre() -> profesor!enviarExamen(examenes.pop());
        * alumno[*]?entregar(examen,id) -> examenes.push(examenAux,id);
        fi
    }
}

Process profesor{
    text examenAux;
    int idAlumno, nota;

    while (true){
        coordinador!estoyLibre();
        coordinador?enviarExamen(examenAux,idAlumno);
        nota = Corregir(examenAux);
        alumno[idAlumno]!recibirNota(nota);
    }
}
```

### 3.b)

```java
Process alumno[id:0..N-1]{
    text examen;

    examen = hacerExamen();
    coordinador!entregar(examen,id);
    profesor[*]?recibirNota(nota);
}

Process coordinador {
    cola examenes(text examen, int id);
    text examenAux; int id, idProfesor;

    do (!examenes.empty()); profesor[*]?estoyLibre(idProfesor) -> profesor[idProfesor]!enviarExamen(examenes.pop());
        * alumno[*]?entregar(examen,id) -> examenes.push(examenAux,id);
    od

}

Process profesor[id:0..P-1]{
    text examenAux;
    int idAlumno, nota;

    while (true){
        coordinador!estoyLibre(id);
        coordinador?enviarExamen(examenAux,idAlumno);
        nota = Corregir(examenAux);
        alumno[idAlumno]!recibirNota(nota);
    }
}
```

### 3.c)

```java
Process alumno[id:0..N-1]{
    text examen;

    coordinador!llegue();
    coordinador?empezar();
    examen = hacerExamen();
    coordinador!entregar(examen,id);
    profesor[*]?recibirNota(nota);
}

Process coordinador {
    cola examenes(text examen, int id);
    text examenAux; int id, idProfesor;
    int cant=0;
    
    do alumno[*]?llegue() -> {
        cant++; if (cant == N){
            for (int i=0;i<N;i++)
                alumno[i]!empezar();
        }
    }
    * (!examenes.empty()); profesor[*]?estoyLibre(idProfesor) -> profesor[idProfesor]!enviarExamen(examenes.pop());
    * alumno[*]?entregar(examen,id) -> examenes.push(examenAux,id);
    od
}

Process profesor[id:0..P-1]{
    text examenAux;
    int idAlumno, nota;

    while (true){
        coordinador!estoyLibre(id);
        coordinador?enviarExamen(examenAux,idAlumno);
        nota = Corregir(examenAux);
        alumno[idAlumno]!recibirNota(nota);
    }
}
```

### 4)  En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar el uso del mismo. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. El empleado deja usar el simulador a las personas respetando el orden de llegada.
#### Nota: cada persona usa sólo una vez el simulador.

```java
Process persona[id:0..P-1]{
    empleado!llegada(id);
    empleado?pasar();
    empleado!terminar();
}

Process empleado{
    cola personas(int);
    int idAux;
    boolean libre = true;

    do !(libre); personas[*]?llegada(idAux) -> personas.push(idAux);
    * (libre) && (personas.empty()); personas[*]?llegada(idAux) -> libre = false; persona[idAux]!pasar();
    * !(personas.empty()); personas[*]?terminar() -> persona[personas.pop()]!pasar();
    * (personas.empty()); personas[*]?terminar() -> libre = true;
}
```

### 5)  En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo al orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente.
#### Nota: cada Espectador una sólo una vez la máquina.

```java
Process espectador[id:0..E-1]{
    maquina!llegada(id);
    maquina?pasar();
    empleado!terminar();
}

Process maquina {
    cola espectadores(int);
    int idAux;
    boolean libre = true;

    do !(libre); espectadores[*]?llegada(idAux) -> espectadores.push(idAux);
    * (libre) && (espectadores.empty()); espectadores[*]?llegada(idAux) -> libre = false; espectadores[idAux]!pasar();
    * !(espectadores.empty()); espectadores[*]?terminar() -> espectadores[espectadores.pop()]!pasar();
    * (espectadores.empty()); espectadores[*]?terminar() -> libre = true;
}
```