## Ejercicio 1
### Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión. 

#### a. Analice el problema y defina qué procesos, recursos y semáforos/sincronizaciones serán necesarios/convenientes para resolverlo. 

*Para este problema necesitamos:*
 - *1 proceso Persona [1 to N]*
 - *1 semaforo (detector libre/ocupado)*


#### b. Implemente una solución que modele el acceso de las personas a un detector (es decir, si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar). 


```c
sem detector = 1

Process Persona [1 to N]:
    P(detector)
    //utilizo el detector
    V(detector)    
    //ingreso al avion
```

#### c. Modifique su solución para el caso que haya tres detectores.

```c
sem detector = 3 //existen 3 detectores

Process Persona [id: 1 to N]:
    P(detector)
    //utilizo 1 de los 3 detectores
    V(detector)
    //ingreso al avion
```

#### d. Modifique la solución anterior para el caso en que cada persona pueda pasar más de una vez, siendo aleatoria esa cantidad de veces.

```c
sem detector = 3

Process Persona [id: 1 to N]:
    int intentos = Random();
    for (int i = 0 ; i < intentos ; i++):
        P(detector)
        //utilizo 1 de los 3 detectores
        V(detector)
        //ingreso al avion pero soy caprichoso y salgo
```  
<br>

## Ejercicio 2
### Un sistema de control cuenta con 4 procesos que realizan chequeos en forma colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño  N).  De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes situaciones: 

#### a. Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).

```c
sem recurso_libre = 1; int historial[N]; int cant_verificados = 0;

Process Proceso[id: 1 to 4]:
    P(recurso_libre);
    while (cant_verificados < N):
        if (historial[cant_verificados].gravedad == 3):
            imprimir(historial[cant_impresos])
        cant_verificados = cant_verificados + 1
        V(recurso_libre)
        P(recurso_libre)
    V(recurso_libre)
```

#### b. Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global. 

```c
sem historial_libre = 1; sem nivel_libre[4] = ([4] 1);
int historial[N]; int cant_verificados = 0; int fallos_totales[4] = ([4] 0)

Process Proceso[id: 1 to 4]:
    P(historial_libre)  //espero que el historial este libre
    while (cant_verificados < N):
        int fallo_actual = historial[cant_verificados].gravedad()  //me quedo con el nivel del fallo
        cant_verificados = cant_verificados + 1
        V(historial_libre)  //libero el historial
        P(nivel_libre[fallo_actual])  //espero que el vector en la posicion del nivel este libre
        fallos_totales[fallo_actual] = fallos_totales[fallo_actual] + 1
        V(nivel_libre[fallo_actual])  //libero la posicion del vector
        P(historial_libre)  //espero que el historial vuelva a estar libre
    V(historial_libre)  //libero el historial luego de verigicar que ya no quedan mas posiciones para leer
```

#### c. Ídem b pero cada proceso debe ocuparse de contar los fallos de un nivel de gravedad determinado.

```c
sem historial_libre = 1; int historial[N];
int cant_verificados = 0; int fallos_totales[4] = ([4] 0);

Process Proceso[id: 0 to 3]:
    P(historial_libre)
    while (cant_verificados < N):
        if (id == historial[cant_verificados].gravedad())  //si el nivel de gravedad me corresponde verificarlo
            cant_verificados = cant_verificados + 1
            fallos_totales[id] = fallos_totales[id] + 1
        V(historial_libre)
        P(historial_libre)
    V(historial_libre)
```
<br>

## Ejercicio 3
### Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente para su reúso.

```c
sem hay_elementos = 5;  //se utiliza para saber que aun hay elementos en la cola
sem cola_libre = 1;  //se utiliza para acceder a la cola de a 1 proceso a la vez (sabiendo que existen elementos)
Cola cola[5];

Process Proceso[id: 1 to P]:
    while (true):
        P(hay_elementos)  //espero que existan elementos en la cola
        P(cola_libre)  //espero que nadie este accediendo a la cola
        recurso = cola.pop()
        V(cola_libre)  //libero la cola luego de usarla
        // utilizo el recurso
        P(cola_libre)  //espero que nadie este accediendo a la cola
        cola.push(recurso)
        V(cola_libre)  //libero la cola luego de usarla
        V(hay_elementos)  //aviso que existen nuevamente elementos en la cola
```
<br>

## Ejercicio 4
### Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción: 
-  *no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.*
-  *no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.*
### Indique si la solución presentada es la más adecuada. Justifique la respuesta.

```c
sem total = 6; sem alta = 4; sem baja = 5;

Process Usuario-Alta[I: 1 to L]:
    P(total)
    P(alta)
    //usa la BD
    V(total)
    V(alta)

Process Usuario-Baja[I: 1 to K]:
    P(total)
    P(baja)
    //usa la BD
    V(total)
    V(baja)
```
*Supongamos el siguiente caso:*
 - *6 usuario-baja pasaron el primer semaforo P(total)*
 - *5 de ellos pasaron el segundo semaforo (P(baja)), mientras, el restante espera luego de pasar el primer semaforo P(baja)*
 - *en este momento solo tenemos 5 usuarios utilizando la BD y no puede ingresar un 6to debido a la restriccion que nos dice que solo pueden 5 usuarios-baja utilizar la BD al mismo tiempo*
 - *En simultaneo los usuario-alta estan esperando a poder pasar el primer semaforo P(total), cuando tranquilamente podria 1 de ellos estar utilizando la BD*

**Para solucionar este ejercicio maximizando la concurrencia podemos realizarlo de esta forma**

```c
sem total = 6; sem alta = 4; sem baja = 5;

Process Usuario-Alta[I: 1 to L]:
    P(alta)
    P(total)
    //usa la BD
    V(total)
    V(alta)

Process Usuario-Baja[I: 1 to K]:
    P(baja)
    P(total)
    //usa la BD
    V(total)
    V(baja)
```

*De esta forma van a pasar el primer semaforo segun la restriccion establecida, pero se van a detener al querer ingresar a la BD siempre y cuando el cupo de 6 usuarios en total este completo:*

*Supongamos el siguiente caso:*
 - *En un momento determinado tenemos en la BD 4 usuario-alta (maximo permitido) y 2 usuario-baja*
 - *en ese caso no hay otro usuario-alta que haya pasado el primer semaforo P(alta), pero si existen usuario-baja que pasaron su primer semaforo P(baja) y que al liberarse un espacio del segundo semaforo P(total), entonces un usuario-baja puede entrar a la BD*


## Ejercicio 5
### En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones: 

#### a. La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores; un empelado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez. 

*asumo la existencia de 2 metodos (generar_paquete() y consumir_paquete())*

```c
int sala[N];
int ocupado = 0;  //representa la proxima posicion ocupada (para tomar paquete)
int libre = 0;    //representa la proxima posicion libre (para depositar paquete)
sem vacio = N;    //indica que la sala esta vacia (inicialmente lo esta)
sem lleno = 0;    //indica que la sala esta completa

Process Preparador:  //deposita los paquetes
    Paquete paquete = generar_paquete()
    P(vacio)
    sala[libre] = paquete;
    libre = (libre + 1) mod N
    V(lleno)

Process Entregador:  //toma los paquetes
    P(lleno)
    Paquete paquete = sala[ocupado]
    ocupado = (ocupado + 1) mod N
    V(vacio)
    consumir_paquete(paquete)
```


#### b. Modifique la solución a para el caso en que haya P empleados Preparadores. 

```c
int sala[N];
int ocupado = 0;
int libre = 0;
sem vacio = N;
sem lleno = 0;
sem preparador_libre = 1;    //controla si hay otro preparador trabajando en la sala

Process Preparador[id: 1 to P]:
    Paquete paquete = generar_paquete()
    P(vacio)
    P(preparador_libre)
    sala[libre] = paquete
    libre = (libre + 1) mod N
    V(preparador_libre)
    V(lleno)

Process Entregador:
    P(lleno)
    Paquete paquete = sala[ocupado]
    ocupado = (ocupado + 1) mod N
    V(vacio)
    consumir_paquete(paquete)
```

#### c. Modifique la solución a para el caso en que haya E empleados Entregadores. 

```c
int sala[N];
int ocupado = 0;
int libre = 0;
sem vacio = N;
sem lleno = 0;
sem entregador_libre = 1;

Process Preparador:
    Paquete paquete = generar_paquete()
    P(vacio)
    sala[libre] = paquete
    libre = (libre + 1) mod N
    V(lleno)

Process Entregador[id: 1 to E]:
    P(lleno)
    P(entregador_libre)
    Paquete paquete = sala[ocupado]
    ocupado = (ocupado + 1) mod N
    V(vacio)
    consumir_paquete(paquete)
```

#### d. Modifique la solución a para el caso en que haya P empleados Preparadores y E empleadores Entregadores. 

```c
int sala[N];
int ocupado = 0;
int vacio = 0;
sem vacio = N;
sem lleno = 0;
sem entregador_libre = 1;
sem preparador_libre = 1;

Process Preparador[id: 1 to P]:
    Paquete paquete = generar_paquete()
    P(vacio)
    P(preparador_libre)
    sala[ocupado] = paquete
    ocupado = (ocupado + 1) mod N
    V(preparador_libre)
    V(lleno)

Process Entregador[id: 1 to E]:
    P(lleno)
    P(entregador_libre)
    Paquete paquete = sala[libre]
    libre = (libre + 1) mod N
    V(entregador_libre)
    V(vacio)

```

## Ejercicio 6
### Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos: 

#### a. Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas. 

```c
sem impresora_libre = 1;

Process Persona[id: 1 to N]:
    P(impresora_libre)
    Imprimir(documento)
    V(impresora_libre)
```

#### b. Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada. 

```c
Cola cola;
sem esperando[N] = ([N] 0);
sem acceso_a_cola = 1;
bool existe_proceso_encolado = false;

Process Persona [id: 1 to N]{
    int proximo_id;
    P(acceso_a_cola);
    if (!existe_proceso_encolado){
        existe_proceso_encolado = true; //para avisar que tienen que empezar a encolarse
        V(acceso_a_cola);
    }
    else {
        cola.push(id);
        V(acceso_a_cola);
        P(espera[id]);
    }
    Imprimir(documento);
    P(acceso_a_cola);
    if (cola.isEmpty()){
        existe_proceso_encolado = false;
    }
    else {
        proximo_id = cola.pop();
        V(espera[proximo_id]);
    }
    V(acceso_a_cola);
}

```

#### c. Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1). 

```c
sem esperando[N] = ([N] 0);
int turno_siguiente = 1;

Process Persona[id: 1 to N]:
    if (id != turno_siguiente):
        P(esperando[id])
    Imprimir(documento)
    turno_siguiente = turno_siguiente + 1
    V(esperando[turno_siguiente])
```



#### d. Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora. 

```c
Cola cola;
int siguiente = 0;
sem existe_peticion = 0;
sem impresora_libre = 0;
sem cola_libre = 1;
sem esperando[P] = ([P] 0)

Process Persona[id: 1 to P]:
    P(cola_libre)
    cola.push(id)
    V(cola_libre)
    P(esperando[id])
    Imprimir(documento)
    V(impresora_libre)

Process Coordinador:
    P(cola_libre)
    siguiente = cola.pop()
    V(cola_libre)
    V(esperando[siguiente])
```

#### e. Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar. 
