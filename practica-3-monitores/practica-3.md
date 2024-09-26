## Ejercicio1
### se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permisopara pasar por el puente, cruza por el mismo y luego sigue su camino.

```c
Monitor Puente:
    cond cola;
    int cant= 0;
    Procedure entrarPuente ()
        while (cant > 0):
            wait (cola);
        cant = cant + 1;
    end;
    Procedure salirPuente ()
        cant = cant – 1;
        signal(cola);
    end;
End Monitor;

Process Auto [a:1 to M]:
    Puente. entrarPuente(a);
    //el auto cruza el puente
    Puente. salirPuente(a);
End Process;
```

#### a. ¿El código funciona correctamente? Justifique su respuesta.
*Si, el codigo funciona correctamente. (el primer auto en llegar va a pasar directamente a incrementar el valor de cant para que el resto de autos que puedan llegar se queden dormidos dentro del while). Como se vio en la teoria al consultar por la variable permanente del monitor cant dentro de un while, nos aseguramos de que nunca vamos a tener 2 procesos trabajando en la zona critica al mismo tiempo.
Al hacer Puente.entrarPuente() el auto verifica si existe actualmente un auto cruzando el puente (cant > 0), de ser asi se queda esperando su turno. En caso de que no exista un auto cruzando (cant == 0) el auto setea la variable permanente del monitor (cant = cant + 1), y, en su mismo proceso cruza el puente.
Luego hace Puente.salirPuente(a) la cual sigue haciendo uso de la exclusion mutua propia del monitor y modifica la variable permanente dle monitor (cant = cant - 1) y envia la señal para que, en caso de existir un auto esperando, sepa que puede vovler a competir por pasar al puente*
*¿seria un error en este caso pasar parametro el id del auto cuando los procedure entrarPuente() y salirPuente() no los esperan?*
**Hay que tener en cuenta que en esta solucion tienen "prioridad" los procesos que no estan dromidos y el primero que se durmio**



#### b. ¿Se podría simplificar el programa? ¿Sinmonitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.


```c
Monitor Puente:
    procedure cruzarPuente():
        //el auto cruza el puente

Process Auto[id: 1 to A]:
    Puente.cruzarPuente()

```


#### c. ¿La solución original respeta el orden dellegada de los vehículos? Si rescribió el códigoen el punto b), ¿esa solución respeta el orden de llegada?

*Ni la solucion dada, ni la reescrita en el punto b respetan el orden de llegada, para eso lo realizamos de la siguiente manera*

```c
Monitor Puente:
    cond cola;
    bool ocupado = false
    int espera = 0;

    Procedure entrarPuente ()
        //si esta ocupado me quedo dormido
        if (ocupado):
            espera = espera + 1
            wait (cola);
        //si no esta ocupado utilizo el recurso
        else:
            ocupado = true;
    end;

    Procedure salirPuente (){
        //si no hay nadie esperando permito que ingrese un proceso que no estaba dormido
        if (espera == 0)
            ocupado = false;
        //si hay autos esperando libero al que esta esperando y decremento la espera
        else:
            espera = espera - 1
            signal(cola);
    }
    end;

End Monitor;

Process Auto [a:1 to M]:
    Puente. entrarPuente(a);
    //el auto cruza el puente
    Puente. salirPuente(a);
End Process;
```
## Ejercicio 2

### Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas.

#### a. Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones serán necesarios/convenientes para resolverlo

*En este ejercicio voy a tener:*
 - **Procesos**
   - Lector[id: 1 to N]
 - **Monitores**
   - Administrador_bd
 - **Recurso compartido**
   - BD

### b. Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas


```c

Monitor administrador_bd:
    cond espera;
    int cantidad = 0;

    Procedure pedir_acceso()
        while (cantidad >= 5):
            wait(espera)
        else:
            cantidad++

    Procedure liberar_acceso()
        cantidad--
        signal(espera)

Process Lector[id: 1 to N]:
    administrador_bd.pedir_acceso()
    //leer base de datos
    administrador_bd.liberar_acceso()

```

## Ejercicio 3
### Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada  por  una  persona  a  la  vez.  Analice  el  problema  y  defina  qué  procesos,  recursos  y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones: 


#### a. Implemente una solución suponiendo no importa el orden de uso. Existe una función Fotocopiar() que simula el uso de la fotocopiadora.  
  
```c

Monitor Administrador_fotocopiadora:
    Procedure fotocopiar():
        fotocopiar()

Process Persona[id: 1 to N]:
    Administrador_fotocopiadora.fotocopiar()

```


#### b. Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada. 

```c

Monitor Administrador_fotocopiadora:
    cond cola;
    bool ocupado = false
    int esperando = 0

    Procedure pedir_fotocopiadora():
        if (ocupado):
            esperando++
            wait(cola)
        else
            ocupado = true

    Procedura liberar_fotocopiadora():
        if (esperando > 0):
            esperando--
            signal(cola)
        else:
            ocupado = false

Process Persona[id: 1 to N]:
    Administrador_fotocopiadora.pedir_fotocopiadora()
    fotocopiar()
    Administrador_fotocopiadora.liberar_fotocopiadora()

```

#### c. Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla). 

```c

Monitor Administrador_fotocopiadora:
    ColaEspecial cola;
    cond espera[N];
    bool ocupado = false;
    int id_usuario;

    Procedure pedir_fotocopiadora(id:int IN, edad:int IN):
        if (ocupado):
            agregar(cola, id, edad)
            wait(espera[id])
        else:
            ocupado = true
        
    Procedure liberar_fotocopiadora():
        if (cola.isEmpty())
            ocupado = false
        else:
            sacar(cola, id_usuario)
            signal(espera[id_usuario])

Process Persona[id: 1 to N]:
    int edad;
    Administrador_fotocopiadora.pedir_fotocopiadora(id, edad)
    fotocopiar()
    Administrador_fotocopiadora.liberar_fotocopiadora()
```


#### d. Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1).

  
```c

Monitor Administrador_fotocopiadora:
    int siguiente = 1;
    cond cola[N];

    Procedure pedir_fotocopiadora(id:int):
        if (id <> siguiente):
            wait(cola[id])

    Procedure liberar_fotocopiadora():
        siguiente++
        signal(cola[siguiente])

Process Persona[id: 1 to N]:
    Administrador_fotocopiadora.pedir_fotocopiadora(id)
    fotocopiar()
    Administrador_fotocopiadora.liberar_fotocopiadora()

```

#### e. Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora. 

```c
Monitor Administrador_fotocopiadora:
    cond cola;
    cond llega_persona
    cond recurso_libre
    int esperando = 0

    Procedure esperar_fotocopiadora():
        esperando++
        signal(llega_persona)
        wait(cola)

    Procedure pedir_fotocopiadora():
        if (esperando == 0):
            wait(llega_persona)
        esperando--
        signal(cola)
        wait(recurso_libre)

    Procedura liberar_fotocopiadora():
        signal(recurso_libre)

Process Empleado:
    for (int i=1 ; i==N ; i++)
        Administrador_fotocopiadora.pedir_fotocopiadora()

Process Persona[id: 1 to N]:
    Administrador_fotocopiadora.esperar_fotocopiadora()
    fotocopiar()
    Administrador_fotocopiadora.liberar_fotocopiadora()
```

#### f. Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo. 

```c
Monitor Administrador_fotocopiadora:
    cond cola;
    cond llega_persona;
    cond existe_impresora_libre;
    Cola personas_esperando;
    Cola impresoras_libres = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    int id_impresora;
    int impresora_indicada[N] = ([N] 0);

    Procedure esperar_fotocopiadora(id_persona:int IN, impresora:int OUT):
        personas_esperando.push(id)
        signal(llega_persona)
        wait(cola)
        impresora = impresora_indicada[id]

    Procedure pedir_fotocopiadora():
        if (personas_esperando.isEmpty()):
            wait(llega_persona)
        if (impresoras_libre.isEmpty())
            wait(existe_impresora_libre)
        impresora_indicada[personas_esperando.pop()] = impresoras_libre.pop()
        signal(cola)
        
    Procedura liberar_fotocopiadora(id_fotocopiadora:int IN):
        impresoras_libres.push(id_fotocopiadora)       
        signal(existe_impresora_libre) 

Process Empleado:
    for (int i=1 ; i==N ; i++)
        Administrador_fotocopiadora.pedir_fotocopiadora()

Process Persona[id: 1 to N]:
    int impresora;
    Administrador_fotocopiadora.esperar_fotocopiadora(id, impresora)
    fotocopiar(impresora)
    Administrador_fotocopiadora.liberar_fotocopiadora(impresora)
```

## Ejercicio 4
###  Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada. Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio peso (ningún vehículo supera el peso soportado por el puente). 

```c
Monitor Puente:
    int peso_maximo = 50000;
    int peso_total = 0;
    int esperando = 0;
    Cola cola_pesos;
    cond esperando_cruzar;

    Procedure pedir_cruce(peso:int IN):
        if ((peso_total + peso) > peso_maximo || !cola_pesos.isEmpty()):
            cola_pesos.push(peso)
            wait(esperando_cruzar)
        peso_total = peso_total + peso

    Procedure liberar_cruce(peso:int IN):
        peso_total = peso_total - peso
        if (!cola_pesos.isEmpty() && peso_total + cola_pesos[0] <= peso_maximo):
            cola_pesos.pop()    
            signal(esperando_cruzar)

Process Auto[id: 1 to N]:
    int peso = x;
    Puente.pedir_cruce(peso)
    //cruza el puente
    Puente.liberar_cruce(peso)

```

## Ejercicio 5
### En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada. Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra realizada. 

#### a. Resuelva considerando que el corralón tiene un único empleado. 

```c
Monitor casa_materiales:
    text lista_cliente, comprobante_cliente;
    cond cola_espera;
    cond lista_disponible;
    cond comprobante_disponible;
    int esperando = 0;
    cond esperando_comprobante;
    cond cliente_esperando;


    Procedure pedir_atencion(lista_productos:text IN, comprobante:text OUT):
        esperando++
        signal(cliente_esperando)
        wait(cola_espera)
        lista_cliente = lista_productos
        signal(lista_disponible)
        wait(esperando_comprobante)
        comprobante = comprobante_cliente
        signal(comprobante_disponible)

    Procedure atender(lista:text out, comprobante:text IN)
        if (esperando == 0):
            wait(cliente_esperando)
        esperando--
        signal(cola_espera)
        wait(lista_disponible)
        lista = lista_cliente
    
    Procedure liberar(comprobante:text IN)
        comprobante_cliente = comprobante
        signal(esperando_comprobante)
        wait(comprobante_disponible)

Process Empleado:
    text lista;
    text comprobante;
    for (int i=1 ; i==N ; i++)
        casa_materiales.atender(lista)
        comprobante = generarComprobante(lista)
        casa_materiales.liberar(comprobante)
        
Process Cliente[id: 1 to N]:
    text lista_productos;
    text comprobante;
    casa_materiales.pedir_atencion(lista_productos, comprobante)
```


#### b. Resuelva considerando que el corralón tiene E empleados (E > 1). Los empleados no deben terminar su ejecución.

```c

Monitor Gestor_espera:
    Cola empleados_libres;
    cond existe_empleado_libre;
    int empelados_disponibles;
    int clientes_esperando;

    Procedure empleado_libre(id:int IN):
        empleados_libres.push(id)
        if (clientes_esperando == 0)
            empleados_disponibles++
        else:
            clientes_esperando--
            signal(existe_empleado_libre)

    Procedure solicitar_atencion(id_empleado:int OUT):
        if (empleados_disponibles == 0):
            clientes_esperando++
            wait(existe_empleado_libre)
        else:
            empleados_disponibles--
        id_empleado = empleados_libres.pop()

Monitor Escritorio[id: 1 to E]:
    text lista_cliente;
    text comprobante_cliente;
    cond lista_disponible;
    cond comprobante_disponible
    cond comprobante_retirado
    bool esperando = false

    procedure atencion(lista:text IN, comprobante:text OUT):
        lista_cliente = lista
        esperando = true
        signal(lista_disponible)
        wait(comprobante_disponible)
        comprobante = comprobante_cliente
        signal(comprobante_retirado)
    
    procedure atender_cliente(lista:text OUT):
        if (!esperando):
            wait(lista_disponible)
        esperando = false   
        lista = lista_cliente

    procedure entregar_comprobante(comprobante:text IN):
        comprobante_cliente = comprobante
        signal(comprobante_disponible)
        wait(comprobante_retirado)

Process Empleado[id: 1 to E]:
    text lista;
    text comprobante;
    while (true):
        Gestor_espera.empleado_libre(id)
        Escritorio.atender_cliente(lista)
        comprobante = generar_comprobante(lista)
        Escritorio[id].entregar_comprobante(comprobante)

Process Cliente[id: 1 to N]:
    int id_empleado;
    text lista;
    text comprobante;
    Gestor_espera.solicitar_atencion(id_empleado)
    Escritorio[id_empleado].atencion(lista, comprobante)
```

#### c. Modifique la solución (b) considerando que los empleados deben terminar su ejecución cuando se hayan atendido todos los clientes.

```c

Monitor Gestor_espera:
    Cola empleados_libres;
    cond existe_empleado_libre;
    int empelados_disponibles
    int clientes_esperando
    int clientes_atendidos = 0;


    Procedure empleado_libre(id:int IN):
        empleados_libres.push(id)
        if (clientes_esperando == 0)
            empleados_disponibles++
        else:
            clientes_esperando--
            signal(existe_empleado_libre)

    Procedure solicitar_atencion(id_empleado:int OUT, seguir:bool OUT):
        if (empleados_disponibles == 0):
            clientes_esperando++
            wait(existe_empleado_libre)
        else:
            empleados_disponibles--
        clientes_atendidos++
        if (clientes_atendidos == N)
            seguir = false
        id_empleado = empleados_libres.pop()

Monitor Escritorio[id: 1 to E]:
    text lista_cliente;
    text comprobante_cliente;
    cond lista_disponible;
    cond comprobante_disponible
    cond comprobante_retirado
    bool esperando = false

    procedure atencion(lista:text IN, comprobante:text OUT):
        lista_cliente = lista
        esperando = true
        signal(lista_disponible)
        wait(comprobante_disponible)
        comprobante = comprobante_cliente
        signal(comprobante_retirado)
    
    procedure atender_cliente(lista:text OUT):
        if (!esperando):
            wait(lista_disponible)
        esperando = false   
        lista = lista_cliente

    procedure entregar_comprobante(comprobante:text IN):
        comprobante_cliente = comprobante
        signal(comprobante_disponible)
        wait(comprobante_retirado)

Process Empleado[id: 1 to E]:
    text lista;
    text comprobante;
    bool seguir = true
    while (seguir): //alcanza que el ultimo deje seguir = false?
        Gestor_espera.empleado_libre(id, seguir)
        Escritorio.atender_cliente(lista)
        comprobante = generar_comprobante(lista)
        Escritorio[id].entregar_comprobante(comprobante)

Process Cliente[id: 1 to N]:
    int id_empleado;
    text lista;
    text comprobante;
    Gestor_espera.solicitar_atencion(id_empleado)
    Escritorio[id_empleado].atencion(lista, comprobante)
```

## Ejercicio 6
###  Existe  una  comisión  de  50  alumnos  que  deben  realizar  tareas  de  a  pares,  las  cuales  son corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25. Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1). Nota: el JTP no guarda el número de grupo que le asigna a cada alumno.

```c
Process Alumno[id: 1 to 50]:
    int grupo;
    int nota;
    //llegar y hacer barrera hasta que esten todos
    Tarea.presentarse(id)
    Tarea.esperar_tarea(id, grupo)
        //realizar tarea
    //avisar al jtp que termine
    Tarea.entregar_tarea(grupo, nota)


Process Jtp:
    int grupo;
    int grupos_finalizados[25] = ([25] 0)
    int nota = 25
    //espero que lleguen todos los alumnos
    Tarea.esperar_alumnos()
    //asigno un grupo a los 50 alumnos
    for (int i=1 ; i==50 ; i++):
        grupo = asignar_grupo()
        Tarea.asignar_grupo(grupo)
    //espero que de a uno vayan terminando la tarea
    //si termina el 2do del grupo entrego la nota
    for (int i=1; i==50 ; i++);
        Tarea.recibir_tarea(grupo)
        grupos_finalizados[grupo]++
        if (grupos_finalizados[grupo] == 2):
            Tarea.entregar_nota(grupo, nota)
            nota--


Monitor Tarea;
    int alumnos_presentes = 0;
    int id_alumno;
    Cola alumnos;
    cond espera_grupo[50]
    int grupos_asignados[50] = ([50] 0);
    cond alumnos_listos;
    Cola tareas_finalizadas;
    cond espera_nota[25];
    int notas_grupo[25] = ([25] 0);

    Procedure presentarse(id:int IN):
        alumnos_presentes++
        alumnos.push(id)
        if (alumnos_presentes == 50):
            signal(alumnos_listos)

    Procedure esperar_alumnos():
        if (alumnos_presentes != 50):
            wait(alumnos_listos)

    Procedure asignar_grupo(grupo:int IN)
        id_alumno = alumnos.pop()
        grupos_asignados[id_alumno] = grupo
        signal(espera_grupo[id_alumno])

    Procedure esperar_tarea(id:int IN, grupo:int OUT):
        wait(espera_grupo[id])
        grupo = grupos_asignados[id]

    procedure entregar_tarea(grupo:int IN, nota:int OUT):
        tareas_finalizadas.push(grupo)
        signal(tarea_finalizada)
        wait(notas_grupo[grupo])
        nota = notas_grupo[grupo]

    Procedure recibir_tarea(grupo:int OUT)
        if (tareas_finalizadas.isEmpty()):
            wait(tarea_finalizada)
        grupo = tareas_finalizadas.pop()

    Procedure entregar_nota(grupo:int IN, nota:int IN)
        notas_grupo[grupo] = nota
        signal(notas_grupo[grupo])
        signal(notas_grupo[grupo])
    
```



## Ejercicio 7
### Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin  botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira. Nota: mientras se reponen las botellas se debe permitir que otros corredores se encolen.  

```c
Process Corredor[id: 1 to C]:
    Carrera.correr()
    ejecutar_carrera()
    Acceso_Maquina.usar_maquina()
    Maquina.sacar_botella(botella)
    Acceso_Maquina.liberar_maquina()

Process Repositor:
    while (true):
        Maquina.reponer()

Monitor Carrera:
    int corredores = 0;
    cond espera_corredores;

    Procedure correr():
        corredores++
        if (corredores < C):
            wait(espera_corredores)
        else:
            signal_all(espera_corredores)

Monitor Acceso_Maquina:
    int corredores = 0;
    bool maquina_ocupada = false
    cond fila

    procedure usar_maquina():
        if (maquina_ocupada):
            corredores++
            wait(fila)
        else:
            maquina_ocupada = true

    procedure liberar_maquina():
        if (corredores == 0):
            maquina_ocupada = false
        else
            corredor--
            signal(fila)
            
Monitor Maquina:
    int corredores = 0;
    int botellas = 20;
    cond sin_botellas;
    cond repuestas;

    procedure sacar_botella():
        if (botellas == 0):
            signal(sin_botellas)
            wait(repuestas)
        botellas--

    procedure reponer()
        if (botellas > 0):
            wait(sin_botellas)
        botellas = 20
        signal(repuestas)
```

## Ejercicio 8
### En un entrenamiento de fútbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el quipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan  durante  50  minutos,  y  al  terminar  todos  los  jugadores  del  partido  se  retiran  (no  es necesario que se esperen para salir). 


```c



```
## Ejercicio 9
### En un examen de la secundaria hay  un preceptor y una profesora que deben tomar un examen escrito a 45 alumnos. El preceptor se encarga de darle el enunciado del examen a los alumnos cundo los 45 han llegado (es el mismo enunciado para todos). La profesora se encarga de ir corrigiendo los exámenes de acuerdo con el orden en que los alumnos van entregando. Cada alumno al llegar espera a que le den el enunciado, resuelve el examen, y al terminar lo deja para que la profesora lo corrija y le envíe la nota. Nota: maximizar la concurrencia; todos los procesos deben terminar su ejecución; suponga que la profesora tiene una función corregirExamen que recibe un examen y devuelve un entero con la nota.  

```c



```

## Ejercicio 10
### En un parque hay un juego para ser usada por  N personas de a una a la vez y de acuerdo al orden en que llegan para solicitar su uso. Además, hay un empleado encargado de desinfectar el juego durante 10 minutos antes de que una persona lo use. Cada persona al llegar espera hasta que el empleado le avisa que puede usar el juego, lo usa por un tiempo y luego lo devuelve. Nota: suponga que la persona tiene una función Usar_juego que simula el uso del juego; y el empleado  una  función  Desinfectar_Juego  que  simula  su  trabajo.  Todos  los  procesos  deben terminar su ejecución. 

```c



```