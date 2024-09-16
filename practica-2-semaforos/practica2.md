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
sem acceso_a_cola = 1;
sem existe_peticion = 0;
sem recurso_libre = 0;
sem esperando_recurso[P] = ([P] 0)

Process Persona[id: 1 to P]:
    //espero que este libre la cola para encolarme
    P(acceso_a_cola)
    cola.push(id)
    V(acceso_a_cola)
    //aviso que estoy esperando
    V(existe_peticion)
    //espero que me avisen que puedo pasar
    P(esperando_recurso[id])
    Imprimir(documento)
    //aviso que termine de usar el recurso
    V(recurso_libre)

Process Coordinador:
    //para todas las personas (1..P)
    for (int i=1 ; i=P ; i++)
        //espero que haya alguien esperando
        P(existe_peticion)
        //indico que puede usar la impresora
        P(acceso_a_cola)
        V(esperando_recurso[cola.pop()])
        V(acceso_a_cola)
        //espero que dejen de usar la impresora
        P(recurso_libre)
```

#### e. Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar. 

```c
Cola cola;
Cola cola_impresora_libre[5];
int impresora_indicada[P] = ([P] 0)
sem esperando_recurso[P] = ([P] 0)
sem recurso_libre = 5
sem acceso_a_cola_persona = 1
sem acceso_a_cola_impresora = 1
sem existe_peticion = 0

Process Persona[id: 1 to P]:
    P(acceso_a_cola_persona)
    cola.push(id)
    V(acceso_a_cola_persona)

    V(existe_peticion)

    P(esperando_recurso[id])

    Imprimir(documento)

    P(acceso_a_cola_impresora)
    cola_impresora_libre.push(impresora_indicada[id])
    V(acceso_a_cola_impresora)

    V(recurso_libre)


Process Coordinador:
    int siguiente = 0
    for (int i=1 ; i==P ; i++)
        P(existe_peticion)

        P(acceso_a_cola_persona)
        siguiente = cola.pop()
        V(acceso_a_cola_persona)

        P(acceso_a_cola_impresora)
        impresora_indicada[siguiente] = cola_impresora_libre.pop()
        V(acceso_a_cola_impresora)

        V(esperando_recurso[siguiente])
        
        P(recurso_libre)
```


## Ejercicio 7
### Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea, comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se queda esperando el puntaje del grupo (depende de todos aquellos que comparten el mismo enunciado). Cuando un grupo terminó, el profesor les otorga un puntaje que representa el orden en que se terminó esa tarea de las 10 posibles.
*Nota: Para elegir la tarea suponga que existe una función elegir que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).*

```c
sem esperando_barrera1[50] = ([50] 0);
int barrera1 = 50;
sem barrera1_libre = 1;
Cola cola[50];
sem cola_libre = 1;
sem alumno_finalizo = 0;
int puntaje[10] = ([10] 0)
sem finalizaron[10] = ([10] 0)

Process Alumno[id 1 to 50]:
    //tomar una tarea
    int tarea = asignar_tarea()
    //espero a que los 50 alumnos tengan su tarea (barrera1)
    P(barrera1_libre)
    barrera1 = barrera1 - 1
    if (barrera1 > 0):
        V(barrera1_libre)
    else:
        //si soy el ultimo en tomar una tarea despierto al resto
        V(barrera_libre)
        for (int i=1 ; i==50 ;i++):
            V(esperando_barrera1[j])
    P(esperando_barrera1[id])
    //realizo mi tarea
    realizar_tarea(tarea)
    //encolo la tarea que finalice
    P(cola_libre)
    cola.push(tarea)
    V(cola_libre)
    //avisar al profesor que termine tarea
    V(alumno_finalizo)
    P(finalizaron[tarea])

Process Profesor:
    //para cada uno de los 50 alumnos
    int tarea = -1
    int nota = 1
    int tareas_finalizadas[10] = ([10] 0);
    for (int i=1 ; 1==50 ; i++):
        P(alumno_finalizo)
        P(cola_libre)
        tarea = cola.pop()
        V(cola_libre)
        tareas_finalizadas[tarea] = tareas_finalizadas[tarea] + 1
        //verifico si el grupo ya termino
        if (tareas_finalizadas[tarea] == 5):
            puntaje[tarea] = nota
            nota = nota + 1
            //libero a cada uno de los integrantes del grupo qeu ya finalizo
            for (int i=1 ; i==5 ; i++):
                V(finalizaron[tarea])
```

## Ejercicio 8 
### Una fábrica de piezas metálicas debe producir T piezas por día. Para eso, cuenta con E empleados que se ocupan de producir las piezas de a una por vez. La fábrica empieza a producir una vez que todos los empleados llegaron. Mientras haya piezas por fabricar, los empleados tomarán una y la realizarán. Cada empleado puede tardar distinto tiempo en fabricar una pieza. Al finalizar el día, se debe conocer cual es el empleado que más piezas fabricó. 

#### a. Implemente una solución asumiendo que T > E. 

```c
Cola piezas_a_trabajar[T];
sem cola_piezas_libre = 1;
int barrera_llegada = 0;
sem barrera_llegada_libre = 1;
sem esperando_empleados[E] = ([E] 0)
int piezas_trabajadas[E] = ([E] 0)

Process Empleado[id: 1 to E]:
    //barrera para esperar que lleguen todos los trabajadores
    P(barrera_empleados_libre)
    barrera_llegada++
    if (barrera_llegada < E):
        V(barrera_empleados_libre)
    else:
        V(barrera_empleados_libre)
        for (int i=1 ; i==E ; i++):
            V(esperando_empleados[i])
    //me quedo esperando a que lleguen todos los trabajadores
    P(esperando_empleados[id])
    //necesito tomar piezas mientras existan (son T piezas en total)
    P(cola_piezas_libre)
    while (!piezas_a_trabajar.isEmpty()):
        //tomo una pieza para trabajarla
        Pieza pieza = piezas_a_trabajar.pop()
        V(cola_piezas_libre)
        pieza.fabricar()
        piezas_trabajadas[id]++;
        P(cola_piezas_libre)
    V(cola_piezas_libre)
    //hago otra barrera para asegurarme de que todos terminen y asi poder calcular un maximo asegurandome que todos terminaron
    P(barrera_empelados_libre)
    barrera_llegada--
    if (barrera_llegada == 0):
        for (int i=1 ; i==E ; i++):
            V(esperando_empleado[i])
    V(barrera_empleados_libre)
    P(esperando_empleado[id])
    maximo(piezas_trabajadas) //retorna el id del empleado que mas piezas realizo
```


#### b. Implemente una solución que contemple cualquier valor de T y E.

*La solucion del ejemplo a es valida para cualquier T y E*


## Ejercicio 9
### Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:
 - Los carpinteros continuamente hacen marcos (cada marco es armando por un único carpintero) y los deja en un depósito con capacidad de    almacenar 30 marcos.
 - El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios. 
 - Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador).

```c
//variables para el manejo del deposito de marcos
Cola deposito_marcos[30];
sem deposito_marcos_libre = 1;
sem total_marcos = 30;
sem existen_marcos = 0;
//variables para el manejo del deposito de vidrios
Cola deposito_vidrios[50];
sem deposito_vidrios_libre = 1;
sem total_vidrios = 50;
sem existen_vidrios = 0;
//variables para el manejo del deposito de ventanas (sin limite?)
Cola deposito_ventanas;
sem deposito_ventanas_libre = 1;

Process Carpintero[id: 1 to 4]:
    Marco marco
    while (true):
        P(total_marcos)  //mientras tenga espacio para depositar marcos, lo creo y lo deposito
        marco = realizar_marco()
        P(deposito_marcos_libre)
        deposito_marcos.push(marco)
        V(deposito_marcos_libre)
        V(existen_marcos)  //aviso que existe al menos un marco en el deposito

Process Vidriero:
    Vidrio vidrio
    while (true):
        P(total_vidrios)  //mientras tenga espacio para depositar vidrios, lo creo y lo deposito
        vidrio = realizar_vidrio()
        P(deposito_vidrios_libre)
        deposito_vidrios.push(vidrio)
        V(deposito_vidrios_libre)
        V(existen_vidrios)  //aviso que existe al menos un vidrio en el deposito

Process Armador[id: 1 to 2]:
    Marco marco;
    Vidrio vidrio;
    Ventana ventana;
    while (true):
        P(existen_marcos)  //si existen marcos bloqueo, tomo uno y libero
        P(deposito_marcos_libre)
        marco = deposito_marcos.pop()
        V(deposito_marcos_libre)
        V(total_marcos)  // aviso que libere un lugar en el deposito de marcos
        P(existen_vidrios)  //si existen vidrios bloqueo, tomo uno y libero
        P(deposito_vidrios_libre)
        vidrio = deposito_vidrios
        V(deposito_vidrios_libre)
        V(total_vidrios)  //aviso que libere un lugar en el deposito de vidrios
        ventana = armar_ventana(marco, vidrio)
        P(deposito_ventanas_libres)
        deposito_ventanas.push(ventana)
        V(depositos_ventanas_libres)

```


## Ejercicio 10
### A una cerealera van T camiones a descargarse trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo tipo de cereal.  


#### a. Implemente una solución que use un proceso extra que actúe como coordinador entre los camiones. El coordinador debe retirarse cuando todos los camiones han descargado. 


```c

Process Camion_Trigo[id: 1 to T]:

Process Camion_Maiz[id: 1 to M]:

Process Coordinador:

```


#### b. Implemente una solución que no use procesos adicionales (sólo camiones).


```c
sem total_camiones = 7;
sem total_trigo = 5;
sem total_maiz = 5;

Process Camion_Trigo[id: 1 to T]:
    P(total_trigo)
    P(total_camiones)
    //deposito el trigo
    V(total_camiones)
    V(total_trigo)

Process Camion_Maiz[id: 1 to M]:
    P(total_maiz)
    P(total_camiones)
    //deposito el maiz
    V(total_camiones)
    V(total_maiz)
```

## Ejercicio 11
### En un vacunatorio hay un empleado de salud para vacunar a 50 personas. El empleado de salud atiende a las personas de acuerdo con el orden de llegada y de a 5 personas a la vez. Es decir, que cuando está libre debe esperar a que haya al menos 5 personas esperando, luego vacuna a las 5 primeras personas, y al terminar las deja ir para esperar por otras 5. Cuando ha atendido a las 50 personas el empleado de salud se retira. Nota: todos los procesos deben terminar su ejecución; suponga que el empleado tienen una función VacunarPersona() que simula que el empleado está vacunando a UNA persona.

```c
Cola cola
sem cola_libre = 1;
int personas_en_espera = 0;
sem despertar_empleado = 0;
sem espera_para_irse[5] = ([5] 0)

Process Persona[id: 1 to 50]:
    P(cola_libre)
    personas_en_espera++;
    cola.push(id)
    if ((personas_en_espera mod 5) == 0):
        V(cola_libre)
        V(despertar_empleado)
    else:
        V(cola_libre)
    //esta espera es para asegurarse de que terminen de vacunarse el grupo de 5 personas
    P(espera_para_irse[id])

Process Empleado:
    Persona persona;
    Cola vacunados
    for (int i=1 ; i==10 ; i++):
        P(despertar_empleado)
        for (int j=1 ; j==5 ; j++):
            P(cola_libre)
            persona = cola.pop()
            V(cola_libre)
            vacunar_persona(persona)
            vacunados.push(persona.id)
        //luego de vacunarse el grupo de 5 personas, los libero
        for (int k=1 ; k==5 ; k++);
            V(espera_para_irse[vacunados.pop()])
        
```


## Ejercicio 12
### Simular la atención en una Terminal de Micros que posee 3 puestos para hisopar a 150 pasajeros. En cada puesto hay una Enfermera que atiende a los pasajeros de acuerdo con el orden de llegada al mismo. Cuando llega un pasajero se dirige al Recepcionista, quien le indica qué puesto es el que tiene menos gente esperando. Luego se dirige al puesto y espera a que la enfermera correspondiente lo llame para hisoparlo. Finalmente, se retira.

#### a. Implemente una solución considerando los procesos Pasajeros, Enfermera y Recepcionista. 

```c

Process Pasajeros[id: 1 to P]:

Process Enfermera[id: 1 to 3]:

Process Recepcionista:

```

#### b. Modifique la solución anterior para que sólo haya procesos Pasajeros y Enfermera, siendo los pasajeros quienes determinan por su cuenta qué puesto tiene menos personas esperando. 

*Nota: suponga que existe una función Hisopar() que simula la atención del pasajero por parte de la enfermera correspondiente.*