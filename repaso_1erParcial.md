# Variables compartidas

### Se tiene un salón con cuatro puertas por donde entran los alumnos a un examen. Cada puerta lleva la cuenta de los que entraron por ella y a su vez se lleva la cuenta del total de personas en el salón.
```java
int total = 0;

Process puerta[id:0..3]{
    int personas = 0;

    while (true){
        <await (llegada);>
        personas++;
        <total++;>
    }
}
```

### Hay un docente que les debe tomar examen oral a 30 alumnos (de a uno a la vez) de acuerdo al orden dado por el Identificador del proceso

```java
int siguiente = -1;
boolean llegada = false;
boolean termino = false;

Process alumno[id:0..29]{
    <await (siguiente == id);>
    llegada = true;
    // Rinde el examen
    <await (termino == true);>
    termino = false;
}

Process profesor {
    for (int i=0; i<30; i++){
        siguiente = i;
        <await (llegada);>
        llegada = false;
        // Toma examen
        termino = true;
        <await (termino == false);>
    }
}
```

### Un cajero automático debe ser usado por N personas de a uno a la vez y según el orden de llegada al mismo. En caso de que llegue una persona anciana, la deben dejar ubicarse al principio de la cola.
```java
boolean libre = true;
int siguiente = -1;
queue personas(int);

Process persona[id:0..N-1]{
    boolean esAnciana = ...;
    <if (libre){
        libre = false;
        siguiente = id;
    } else {
        if (esAnciana) { personas.agregarPrimero(id); }
        else {
            personas.push(id);
        }
    }>

    <await (siguiente == id);>
    // Usa cajero
    <if (empty(personas)){
        libre = true;
    } else { siguiente = personas.pop(); }>
}
```