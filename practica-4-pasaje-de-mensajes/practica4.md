# PMA
## Ejercicio 1
### Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes situaciones: 

#### a. Existe un único empleado, el cual atiende por orden de llegada. 

```c
chan atencion(id);

Process Cliente[id: 1 to N]:
    send atencion(id)

Process Empleado:
    int proximo;
    while (true):
        receive atencion(proximo)
        atender(proximo)
```


#### b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior? 

```c
chan atencion(id);

Process Cliente[id: 1 to N]:
    send atencion(id)

Process Empleado[id: 1 to 2]:
    int proximo;
    while (true):
        receive atencion(proximo)
        atender(proximo)
```


#### c. Ídem b) pero  considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría? 

```c
chan atencion(int);
chan pedido(int);
chan siguiente[2](int)

Process Cliente[id: 1 to N]:
    send atencion(id)

Process Admin:
    int id_empleado;
    int id_cliente;
    while (true):
        receive pedido(id_empleado)
        if (empty(atencion)):
            id_cliente = -1
        else:
            receive atencion(id_cliente)
        send siguiente[id_empleado](id_cliente)

Process Empleado[id: 1 to 2]:
    int proximo;
    while (true):
        send pedido_cliente(id)
        receive siguiente(proximo)
        if (proximo == -1):
            delay(15)
        else:
            atender(proximo)
```


## Ejercicio 2
### Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia. 

```c
chan pedido_caja(int)
chan caja_indicada[P](int)
chan pedido[5](int, text)
chan avanzar()


Process Cliente[id: 1 to P]:
    int id_caja;
    text pago;
    text comprobante;

    send pedido_caja(id)                // pido una caja
    send avanzar()                      // aviso que hay pedido
    receive caja_indicada[id](id_caja)  // espero la recepcion del mismo
    send pedido[id_caja](id, pago)      // pido atencion en la caja indicada
    receive comprobante[id](comprobante)// espero el comprobante
    send liberar_caja(id_caja)          // aviso que libere la caja
    send avanzar()                      // aviso que hay pedido



Process Administrador:
    int cajas[5] = ([5] 0)
    int id_aux;
    while (true):

        receive avanzar()

        if (!empty(liberar_caja)):       //verifico si alguien libero una caja y decremento
            receive liberar_caja(id_aux)
            cajas[id_aux]--
        else:
            receive pedido_caja(id_aux)         // espero un pedido de caja
            minimo = buscar_minimo(cajas)       // obtengo la caja con menos espera
            cajas[minimo]++                     // aumento el contador correspondiente
            send caja_indicada[id_aux](minimo)  // envio la caja indicada al empleado que solicito

Process Caja[id: 1 to 5]:
    int id_empleado;
    text pago;
    text comprobante;
    while (true):
        receive pedido[id](id_empleado, pago)
        comprobante = procesar_pago(id_empleado, pago)
        send comprobante[id_empleado](comprobante)

```


## Ejercicio 3
### Se  debe  modelar  el  funcionamiento  de  una  casa  de  comida  rápida,  en  la  cual  trabajan  2 cocineros  y  3  vendedores,  y  que  debe  atender  a  C  clientes.  El  modelado  debe  considerar que: 
 - *Cada cliente realiza un pedido y luego espera a que se lo entreguen.*
 - *Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).*
 - *Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.*
Nota: maximizar la concurrencia.

```c
chan atencion(int, text);
chan pedido(int);
chan siguiente[3](int, text);
chan platos[C](text);
chan cocinar_plato(int, text);

Process Cliente[id: 1 to C]:
    text pedido;
    text plato;
    send atencion(id, pedido)
    receive platos[id](plato)

Process Coordinador:
    int id_cliente;
    int id_empleado;
    text pedido;
    while (true):
        receive pedido(id_empleado) // tomo el pedido de un cocinero
        if (!empty(atencion)):      // si aun no hay pedido del cliente genero datos
            id_cliente = -1
            pedido = "VACIO"
        else:
            receive atencion(id_cliente, pedido)        // recibo el pedido del cliente
        send siguiente[id_empleado](id_cliente, pedido) // envio el pedido y el cliente al cocinero    

Process Vendedor[id: 1 to 3]:
    int id_cliente;
    text pedido;
    while (true):
        send pedido(id)
        receive siguiente[id](id_cliente, pedido)   // recibe los datos del pedido
        if (pedido == "VACIO"):
        delay(180)                                  // si no habia pedido "real" repone gaseosas
        else:
            send cocinar_plato(id_cliente, pedido)  // si habia pedido "real" se lo envio a cualquier cocinero
        
Process Cocinero[id: 1 to 2]:
    int id_cliente;
    text pedido;
    text plato;
    while (true):
        receive cocinar_plato(id_cliente, pedido)   //recibo el id de cliente y el pedido
        plato = generar_plato(pedido)               // realizo el plato
        send platos[id_cliente](plato)              // se lo envio al cliente indicado
```


## Ejercicio 4
### Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación. 
Nota:  maximizar  la  concurrencia;  suponga  que  hay  una  función  Cobrar()  llamada  por  el empleado que simula que el empleado le cobra al cliente. 

#### a. Implemente una solución para el problema descrito.

```c
chan pedido(int, int);
chan espera_cabina[N](int)
chan pago(int, int)
chan ticket[N](text);

Process Cliente[id: 1 to N]:
    int cabina;
    text ticket;
    send pedido(id)
    receive espera_cabina[id](cabina)
    usar_cabina(cabina)
    send pago(id, cabina)
    receive ticket[id](ticket)

Process Empleado:
    bool cabinas[10] = ([10] 0)
    int id_cliente;
    int id_cabina;
    text ticket;

    while (true):
        receive pedido(id_cliente)
        if (!empty(pago)):
            receive pago(id_cliente, id_cabina)
            cabinas[id_cabina] = false
            ticket = cobrar(id_cabina)
            send ticket[id_cliente](ticket)
        else:
            if (!empty(pedido) && existe_cabina_libre(cabinas)):
                receive pedido(id_cliente)
                id_cabina = buscar_cabina_libre(cabinas)
                cabinas[id_cabina] = true
                send espera_cabina[id_cliente](id_cabina)
```

#### b. Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.

```c


```


## Ejercicio 5
### Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a  imprimir.  Cada  impresora,  cuando  está  libre,  toma  un  documento  y  lo  imprime,  de acuerdo con el orden de llegada.
Nota: ni los administrativos ni el director deben esperar a que se imprima el documento. 

#### a. Implemente una solución para el problema descrito. 

```c
chan atencion(text)

Process Administrativo[id: 1 to N]:
    text documento;
    while (true):
        send atencion(documento)

Process Impresora[id: 1 to 3]:
    int id_cliente;
    while (true):
        receive atencion(documento)
        imprimir(documento)
```

#### b. Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos. 

```c
chan atencion_no_prioritaria(text)
chan atencion_prioritaria(text)
chan impresora_libre(int)
chan hay_pedido()

Process Administrativo[id: 1 to N]:
    text documento;
    while (true):
        send atencion_no_prioritaria(documento)
        send hay_pedido()

Process Director:
    text documento;
    while (true):
        send atencion_prioritaria(documento)
        send hay_pedido()

Process Coordinador:
    text documento;
    int id_impresora;
    while (true):
        receive impresora_libre(id_impresora)
        receive hay_pedido()
        if (!empty(atencion_prioritaria))
            receive atencion_prioritaria(documento)
        else:
            receive atencion_no_prioritaria(documento)
        send impresion[id_impresora](documento)

Process Impresora:
    text documento;
    while (true):
        send impresora_libre(id)
        receive impresion[id](documento)
        imprimir(documento)
```

#### c. Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución. 

```c
chan atencion(text)
chan impresora_libre(int)
chan impresion[3](text)

Process Administrativo[id: 1 to N]:
    text documento;
    for (int i=1 ; i==10 ; i++):
        send atencion(documento)

Process Coordinador:
    text documento;
    int id_impresora;
    for (int i=1 ; i==10*N ; i++):
        receive impresora_libre(id_impresora)
        receive atencion(documento)
        send impresion[id_impresora](documento)
    documento = "VACIO"
    for (int i=1 ; i==3 ; i++):
        send impresion[i](documento)

Process Impresora[id: 1 to 3]:
    text documento;
    bool seguir = true
    while (seguir):
        send impresora_libre(id)
        receive impresion[id](documento)
        if (documento == "VACIO"):
            seguir = false
        else
            imprimir(documento)
```

#### d. Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.

```c
chan atencion_no_prioritaria(text)
chan atencion_prioritaria(text)
chan hay_pedido()
chan impresion[3](text)
chan impresora_libre(int)

Process Administrativo[id: 1 to N]:
    text documento;
    for (int i=1; i==10 ; i++):
        send atencion_no_prioritaria(documento)
        send hay_pedido()

Process Director:
    text documento;
    for (int i=1; i==10 ; i++):
        send atencion_prioritaria(documento)
        send hay_pedido()

Process Coordinador:
    text documento;
    int id_impresora;
    for (int i=1 ; i==(10*N)+10 ; i++):
        receive impresora_libre(id_impresora)
        receive hay_pedido()
        if(!empty(atencion_prioritaria)):
            receive atencion_prioritaria(documento)
        else:
            receive atencion_no_prioritaria(documento)
        send impresion[id_impresora](documento)
    documento = "VACIO"
    for (int i=1 ; i==3 ; i++):
        send impresion[i](documento)

Process Impresora[id: 1 to 3]:
    text documento;
    bool seguir = true;
    while (seguir):
        send impresora_libre(id)
        receive impresion[id](documento)
        if (documento == "VACIO")
            seguir = false;
        else:
            imprimir(documento)
```

#### e. Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.
 
  
# PMS

## Ejercicio 1
### Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados. 

#### a. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo. 


#### b. Implemente una solución con PMS sin tener en cuenta el orden de los pedidos. 

```c
//TENER EN CUENTA QUE ESTA SOLUCION REALIZA DEMORA INNECESARIA Y NO MAXIMIZA LA CONCURRENCIA
Process Examinador[id: 1 to R]:
    text reporte;
    while (true):
        reporte = generar_reporte()
        Analizador ! problema(reporte)

Process Analizador:
    text reporte;
    while (true):
        Examinador[*] ? problema(reporte)
        realizar_pruebas(reporte)
```

#### c. Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron.

```c
Process Examinador[id: 1 to R]:
    text reporte;
    while (true):
        reporte = generar_reporte()
        Intermedio ! problema(reporte)

Process Intermedio:
    Cola cola;
    text reporte;
    while (true):
        do
            Examinador[*] ? problema(reporte) -> cola.push(reporte)
            !cola.isEmpty() ; Analizador ? listo() -> Analizador ! problema(cola.pop())
        od

Process Analizador:
    text reporte;
    while (true):
        Intermedio ! listo()
        Intermedio ? problema(reporte)
        realizar_pruebas(reporte)
```


## Ejercicio 2
### En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado  y  vuelve  a  su  trabajo.  El  segundo  empleado  toma  cada  muestra  de  ADN preparada,  arma  el  set  de  análisis  que  se  deben  realizar  con  ella  y  espera  el  resultado  para archivarlo.  Por  último,  el  tercer  empleado  se  encarga  de  realizar  el  análisis  y  devolverle  el resultado al segundo empleado.

```c

Process Empleado_1:
    text muestra;
    while (true):
        muestra = preparar_muestra()
        Coordinador ! toma(muestra)

Process Coordinador:
    Cola cola;
    text muestra;
    while (true):
        do
            Empleado_1 ? toma(muestra) -> cola.push(muestra)
            !cola.isEmpty() -> Empleado_2 ? listo() -> Empleado_2 ! muestra(cola.pop())
        od

Process Empleado_2:
    text muestra;
    text set_analisis;
    text res;
    while (true):
        Coordinador ! listo()
        Coordinador ? muestra(muestra)
        set_analisis = preparar_set(muestra)
        Empleado_3 ! set(set_analisis)
        Empleado_3 ? resultado(res)
        archivar(res)

Process Empleado_3:
    text set_analisis;
    text res;
    while (true):
        Empleado_2 ? set(set_analisis)
        res = generar_resultado(set_analisis)
        Empleado_2 ! resultado(res)

```

## Ejercicio 3
### En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.
Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución 

#### a. Considerando que P=1. 

```c
Process Alumno[id: 1 to N]:
    text examen;
    int nota;
    examen = rendir_examen()
    Coordinador ! termino_examen(id, examen)
    Profesor ? espera_nota(nota)

Process Coordinador:
    Cola cola;
    int rindieron = 0;
    int id_alumno;
    text examen;
    do
        (rindieron < N) ; Alumno[*] ? termino_examen(id_alumno, examen) -> cola.push(id_alumno, examen); rindieron++;
        (!cola.isEmpty()) ; Profesor ? listo() -> Profesor ! para_corregir(cola.pop());
    od

Process Profesor:
    int id_alumno;
    text examen;
    int nota;
    for (int i=1 ; i==N ; i++):
        Coordinador ! listo()
        Coordinador ? para_corregir(id_alumno, examen)
        nota = corregir_examen(examen)
        Alumno[id_alumno] ! espera_nota(nota)

```

#### b. Considerando que P>1. 

```c

Process Alumno[id: 1 to N]:
    int nota, id_profesor;
    text examen = rendir_examen()
    Coordinador ! termino_examen(id, examen)
    Profesor[*] ? espera_nota(nota)

Process Coordinador:
    Cola cola;
    int id_profesor, id_alumno;
    int rindieron, corregidos = 0;
    text examen;
    do
        (rindieron < N) ; Alumno[*] ? termino_examen(id_alumno, examen) -> cola.push(id_alumno, examen); rindieron++;
        (!cola.isEmpty()) ; Profesor[*] ? listo(id_profesor)-> Profesor[id_profesor] ? para_corregir(cola.pop())
    od
    for (int i=1 ; i==P ; i++):
        Profesor[*] ? listo(id_profesor)
        Profesor[id_profesor] ! para_corregir(-1, "VACIO")

Process Profesor[id: 1 to P]:
    bool seguir = true;
    int id_alumno;
    text examen;
    while (seguir):
        Coordinador ! listo(id):
        Coordinador ? para_coregir(id_alumno, examen)
        if (id_alumno != -1):
            nota = corregir_examen(examen)
            Alumno[id_alumno] ! espera_nota(id_alumno)
        else:
            seguir = false
```

#### c. Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.
 
```c

Process Alumno[id: 1 to N]:
    int nota, id_profesor;
    Coordinador ! llegue()
    Coordinador ? empezar()
    text examen = rendir_examen()
    Coordinador ! termino_examen(id, examen)
    Profesor[*] ? espera_nota(nota)

Process Coordinador:
    Cola cola;
    int id_profesor, id_alumno;
    int rindieron, corregidos = 0;
    text examen;
    for (int i=1 ; i==N ; i++):
        Alumno[*] ? llegue()
    for (int i=1 ; i==N ; i++):
        Alumno[i] ! empezar()
    
    do
        (rindieron < N) ; Alumno[*] ? termino_examen(id_alumno, examen) -> cola.push(id_alumno, examen); rindieron++;
        (!cola.isEmpty()) ; Profesor[*] ? listo(id_profesor)-> Profesor[id_profesor] ? para_corregir(cola.pop())
    od

    for (int i=1 ; i==P ; i++):
        Profesor[*] ? listo(id_profesor)
        Profesor[id_profesor] ! para_corregir(-1, "VACIO")

Process Profesor[id: 1 to P]:
    bool seguir = true;
    int id_alumno;
    text examen;
    while (seguir):
        Coordinador ! listo(id):
        Coordinador ? para_coregir(id_alumno, examen)
        if (id_alumno != -1):
            nota = corregir_examen(examen)
            Alumno[id_alumno] ! espera_nota(id_alumno)
        else:
            seguir = false
```

 
## Ejercicio 4
### En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. 
Nota: cada persona usa sólo una vez el simulador.

#### a. Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión mutua. 

```c

Process Persona[id: 1 to P]:
    Empleado ! solicitar(id)
    usar_simulador()
    Empleado ! liberar()

Process Empleado:
    int id_persona;
    Persona[*] ? solicitar(id_persona)
    Persona[id_persona] ? liberar()

```


#### b. Modifique la solución anterior para que el empleado considere el orden de llegada para dar acceso al simulador.

```c

Process Persona[id: 1 to P]:
    Empleado ! solicitar(id)
    Empleado ? pasar()
    usar_simulador()
    Empleado ! liberar()

Process Empleado:
    Cola cola;
    int id_persona;
    bool ocupado = false;
    do
        Persona[*] ? solicitar(id_persona)->   
                                                if (ocupado):
                                                    cola.push(id_persona)
                                                else:
                                                    Persona[id_persona] ! pasar()
                                                    ocupado = true

        Persona[*] ? liberar()->    
                                    if (cola.isEmpty()):
                                        ocupado = false
                                    else:
                                        Persona[cola.pop()] ! pasar()
    od

```

```c

Process Persona[id: 1 to N]:
    Coordinador ! solicitar(id)
    Empleado ? pasar()
    usar_simulador()
    Empleado ! liberar()

Process Coordinador:
    Cola cola;
    int personas = 0;
    int id_persona;
    do
        (personas < N) ; Persona[*] ? solicitar(id_persona) -> cola.push(id_persona); personas++
        (!cola.isEmpty()) ; Empleado ? siguiente() -> Empleado ! dar_acceso(cola.pop())
    od

Process Empleado:
    int id_persona;
    for (int i=1 ; i==P ; i++)
        Coordinador ! siguiente()
        Coordinador ? dar_acceso(id_persona)
        Persona[id_persona] ! pasar()
        Persona[id_persona] ? liberar()


```

## Ejercicio 5
### En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente.
Nota: cada Espectador una sólo una vez la máquina. 

```c

Process Espectador[id: 1 to E]:
    Maquina ! solicitar(id)
    Maquina ? pasar()
    usar_maquina()
    Maquina ! liberar()

Process Maquina:
    Cola cola;
    int id_espectador;
    bool ocupado = false;
    do
        Espectador[*] ? solicitar(id_espectador) -> if (ocupado):
                                                        cola.push(id_espectador)
                                                    else:
                                                        ocupado = true
                                                        Espectador[id_espectador] ! pasar()
        Espectador[*] ? liberar() -> if (cola.isEmpty())
                                        ocupado = false
                                    else:
                                        Espectador[cola.pop()] ! pasar()
    od

```