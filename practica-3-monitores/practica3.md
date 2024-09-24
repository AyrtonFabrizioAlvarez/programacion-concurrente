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
    int empelados_disponibles
    int clientes_esperando

    Procedure empleado_libre(id:int IN):
        empleados_libres.push(id)
        if (clientes_esperando == 0)
            empleados_disponibles++
        else:
            clientes_esperando--
            signal(existe_empleado_libre)

    Procedure solicitar_atencion(id_empleado:int out):
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
    bool esperando = false

    procedure atencion(id_empleado:int IN, lista:text IN, comprobante:text OUT):
        lista_cliente = lista
        esperando = true
        signal(lista_disponible)
        wait(comprobante_disponible)
        comprobante = comprobante_cliente
        signal(comprobante_disponible)
    
    procedure atender_cliente(lista:text OUT):
        if (!esperando):
            wait(lista_disponible)
        esperando = false
        lista = lista_cliente

    procedure entregar_comprobante(comprobante:text IN):
        comprobante_cliente = comprobante
        signal(comprobante_disponible)
        wait(comprobante_disponible)

Process Empleado[id: 1 to E]:
    text lista;
    text comprobante;
    Gestor_espera.empleado_libre(id)
    Escritorio.atender_cliente(lista)
    comprobante = generar_comprobante(lista)
    Escritorio.entregar_comprobante(comprobante)

Process Cliente[id: 1 to ]:
    int id_empleado;
    text lista;
    text comprobante;
    Gestor_espera.solicitar_atencion(id_empleado)
    Escritorio.atendencion(id_empleado, lista, comprobante)
```

#### c. Modifique la solución (b) considerando que los empleados deben terminar su ejecución cuando se hayan atendido todos los clientes.