## Ejercicio 1
### Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades  y  cada  camión  3  unidades.  Suponga  que  hay  una  cantidad  innumerable  de vehículos  (A  autos,  B  camionetas  y  C  camiones).  Analice  el  problema  y  defina  qué  tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 
#### a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad. 

```ada
PROCEDURE Puente IS;

    TASK TYPE Auto;
    TASK BODY Auto IS;
        peso:int;
    BEGIN
        peso = 1;
        Pasar.acceso_autos();
        Pasar.salir(peso);
    END Auto;

    TASK TYPE Camioneta;
    TASK BODY Camioneta IS;
        peso:int;
    BEGIN
        peso = 2;
        Pasar.acceso_camioneta();
        Pasar.salir(peso)
    END Camioneta;

    TASK TYPE Camion;
    TASK BODY Camion IS;
        peso:int;
    BEGIN
        peso = 3;
        Pasar.acceso_camion();
        Pasar.salir(peso);
    END Camion;

    v_autos = array (1..A) of Auto;
    v_camionetas = array (1..C) of Camioneta
    v_camiones = array (1..P) of Camion;

    TASK Pasar IS
        ENTRY acceso_autos;
        ENTRY acceso_camionetas;
        ENTRY acceso_camiones;
    END Pasar;

    TASK BODY Pasar IS;
        peso_total:int
        liberar_peso:int;
    BEGIN
        peso_total = 0;
        LOOP
            SELECT

                when (peso_total <= 2) => ACCEPT acceso_camiones() DO
                    peso_total += 3;
                END acceso_camiones;
            OR
                when (peso_total <= 3) => ACCEPT acceso_camionetas() DO
                    peso_total += 2;
                END acceso_camionetas;
            OR
                when (peso_total <= 4) => ACCEPT acceso_autos() DO
                    peso_total += 1
                END acceso_autos;
            OR
                ACCEPT salir(liberar_peso:int IN) DO
                    peso_total -= liberar_peso
                END salir;


            END SELECT;
        END LOOP;
    END Pasar;

BEGIN
    null
END Puente;
```

#### b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos. 

```ada
PROCEDURE Puente IS;

    TASK TYPE Auto;
    TASK BODY Auto IS;
        peso:int;
    BEGIN
        peso = 1;
        Pasar.acceso_autos();
        Pasar.liberar();
    END Auto;

    TASK TYPE Camioneta;
    TASK BODY Camioneta IS;
        peso:int;
    BEGIN
        peso = 2;
        Pasar.acceso_camioneta();
        Pasar.liberar()
    END Camioneta;

    TASK TYPE Camion;
    TASK BODY Camion IS;
        peso:int;
    BEGIN
        peso = 3;
        Pasar.acceso_camion();
        Pasar.liberar();
    END Camion;

    v_autos = array (1..A) of Auto;
    v_camionetas = array (1..C) of Camioneta
    v_camiones = array (1..P) of Camion;

    TASK Pasar IS
        ENTRY acceso_autos;
        ENTRY acceso_camionetas;
        ENTRY acceso_camiones;
    END Pasar;

    TASK BODY Pasar IS;
        peso_total:int
        liberar_peso:int;
    BEGIN
        peso_total = 0;
        LOOP
            SELECT

                when (peso_total <= 2) => ACCEPT acceso_camiones() DO
                    peso_total += 3;
                END acceso_camiones;
            OR
                when (acceso_camiones'COUNT == 0 OR peso_total >= 3) AND (peso_total <= 3) => ACCEPT acceso_camionetas() DO
                    peso_total += 2;
                END acceso_camionetas;
            OR
                when (acceso_camiones'COUNT == 0 OR peso_total >= 3) AND (peso_total <= 4) => ACCEPT acceso_autos() DO
                    peso_total += 1;
                END acceso_autos;
            OR
                ACCEPT salir(liberar_peso:int IN) DO
                    peso_total -= liberar_peso;
                END salir;


            END SELECT;
        END LOOP;
    END Pasar;

BEGIN
    null
END Puente;
```
 
## Ejercicio 2
### Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada.  

#### a. Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atendidos. 

```ada
PROCEDURE Banco IS;

TASK TYPE Cliente;
TASK BODY Cliente;
    pago:int;
    comprobante:Comprobante;
BEGIN
    Empleado.pedir_atencion(pago, comprobante)
END Cliente;

v_clientes = array (1..C) of Cliente;

TASK Empleado IS;
    ENTRY pedir_atencion(pago:int IN, comprobante:Comprobante OUT)
END Empleado;
TASK BODY Empleado;
    pago:int;
BEGIN
    LOOP
        ACCEPT pedir_atencion(pago:int IN, comprobante:Comprobante OUT) DO
            comprobante = atender(pago)
        END pedir_atencion;
    END LOOP;
END Empleado;

BEGIN
    null
END Banco;
```
#### b. Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago. 

```ada
PROCEDURE BANCO IS:

    TASK TYPE Cliente;
    v_cliente = array (1..C) of Cliente;
    TASK BODY CLiente;
        pago:int;
        comprobante:Comprobante;
    BEGIN
        SELECT
            Empleado.pedir_atencion(pago,comprobante);
        OR DELAY 600
            null;
        END SELECT;
    END Cliente;

    TASK Empleado IS;
        ENTRY pedir_atencion(pago:int IN, comprobante:Comprobante OUT)
    END Empleado;
    TASK BODY Empleado;
        pago:int;
    BEGIN
        LOOP
            ACCEPT pedir_atencion(pago:int IN, comprobante:Comprobante OUT) DO
                comprobante = atender(pago)
            END pedir_atencion;
        END LOOP;
    END Empleado;
BEGIN
    null
END Banco;
```
#### c. Implemente una solución donde los clientes se retiran si no son atendidos inmediatamente. 

```ada
PROCEDURE BANCO IS:

    TASK TYPE Cliente;
    v_cliente = array (1..C) of Cliente;
    TASK BODY CLiente;
        pago:int;
        comprobante:Comprobante;
    BEGIN
        SELECT
            Empleado.pedir_atencion(pago,comprobante);
        ELSE
            null;
        END SELECT;
    END Cliente;

    TASK Empleado IS;
        ENTRY pedir_atencion(pago:int IN, comprobante:Comprobante OUT)
    END Empleado;
    TASK BODY Empleado;
        pago:int;
    BEGIN
        LOOP
            ACCEPT pedir_atencion(pago:int IN, comprobante:Comprobante OUT) DO
                comprobante = atender(pago)
            END pedir_atencion;
        END LOOP;
    END Empleado;
BEGIN
    null
END Banco;
```
#### d.  Implemente  una  solución  donde  los  clientes  esperan  a  lo  sumo  10  minutos  para  ser atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más y se retiran si no son atendidos inmediatamente. 

```ada
PROCEDURE BANCO IS:

    TASK TYPE Cliente;
    v_cliente = array (1..C) of Cliente;
    TASK BODY CLiente;
        pago:int;
        comprobante:Comprobante;
    BEGIN
        SELECT
            Empleado.pedir_atencion(pago,comprobante);
        OR DELAY 600
            SELECT
                Empleado.pedir_atencion(pago, comprobante);
            ELSE
                null
        END SELECT;
    END Cliente;

    TASK Empleado IS;
        ENTRY pedir_atencion(pago:int IN, comprobante:Comprobante OUT)
    END Empleado;
    TASK BODY Empleado;
        pago:int;
    BEGIN
        LOOP
            ACCEPT pedir_atencion(pago:int IN, comprobante:Comprobante OUT) DO
                comprobante = atender(pago)
            END pedir_atencion;
        END LOOP;
    END Empleado;
BEGIN
    null;
END Banco;
```
 
## Ejercicio 3
### Se  dispone  de  un  sistema  compuesto  por  1  central  y  2  procesos  periféricos,  que  se comunican continuamente. Se requiere modelar su funcionamiento considerando las siguientes condiciones: 
 - *La  central  siempre  comienza  su  ejecución  tomando  una  señal  del  proceso  1;  luego toma  aleatoriamente  señales  de  cualquiera  de  los  dos  indefinidamente.  Al  recibir  una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.* 
 - *Los  procesos  periféricos  envían  señales  continuamente  a  la  central.  La  señal  del proceso  1  será  considerada  vieja  (se deshecha)  si  en  2  minutos  no  fue  recibida.  Si  la señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y vuelve a mandarla (no se deshecha).*


```ada
PROCEDURE Sistema IS;

    TASK Periferico1;
    TASK BODY Periferico1;
        señal:int;
    BEGIN
        señal = tomar_señal()
        Central.pedido_1(señal)
        LOOP
            señal = tomar_señal()
            SELECT
                Central.pedido_1()
            OR DELAY 120
                null;
        END LOOP;
    END Periferico1;

    TASK Periferico2;
    TASK BODY Periferico2;
        señal:int
    BEGIN
        señal = tomar_señal()
        LOOP
            SELECT
                Central.pedido_2(señal)
                señal = tomar_señal() 
            ELSE
                DELAY 60
        END LOOP;
    END Periferico2;

    TASK Timer IS;
        ENTRY empezar()
    TASK BODY Timer;
    BEGIN
        ACCEPT empezar()
        DELAY 180
        Central.paso_tiempo()
    END Timer;

    TASK Central IS;
        ENTRY pedido_1(dato:int IN);
        ENTRY pedido_2(DATO:int IN);
        ENTRY paso_tiempo();
    TASK BODY Central;
        dato:int;
        seguir:boolean;
    BEGIN
        ACCEPT pedido_1(dato:int IN)
        LOOP
            SELECT
                ACCEPT pedido_1(dato:int IN)
            OR
                ACCEPT pedido_2(dato:int IN);
                seguir = true;
                Timer.empezar();
                LOOP (seguir)
                    SELECT
                        when (paso_tiempo'COUNT == 0) => ACCEPT pedido_2(dato);
                    OR
                        ACCEPT paso_tiempo();
                        seguir = false;
                END LOOP;
            END SELECT;
        END LOOP;
    END Central;

BEGIN
    null;
END Sistema;
```


## Ejercicio 4
### En  una  clínica  existe  un  médico  de  guardia  que  recibe  continuamente  peticiones  de atención de las E  enfermeras que trabajan en su piso y de las  P  personas que llegan a la clínica ser atendidos. Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica.Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le  hace  una  nota  y  se  la  deja  en  el  consultorio  para  que  esta  resuelva  su  pedido  en  el momento  que  pueda  (el  pedido  puede  ser  que  el  médico  le  firme  algún  papel). Cuando  la petición  ha  sido  recibida  por  el  médico  o  la  nota  ha  sido  dejada  en  el  escritorio,  continúa trabajando y haciendo más peticiones. El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras.

```ada
PROCEDURE Clinica IS;

    TASK Medico IS;
        ENTRY atender_persona(pedido:Pedido IN)
        ENTRY atender_enfermera(pedido:Pedido IN)
    TASK BODY Medico;
        pedido_actual:Pedido;
        nota:Nota;
    BEGIN
        LOOP
            SELECT
                ACCEPT atender_persona(pedido:Pedido IN) DO
                    pedido_actual = pedido
                    atender_pedido(pedido_actual)
                END atender_persona;
            OR
                when (atender_persona'COUNT == 0) => ACCEPT atender_enfermera(pedido:Pedido IN) DO
                    pedido_Actual = pedido;
                END atender_enfermera;
                atender_pedido(pedido_actual)

            ELSE
                SELECT
                    Escritorio.retirar_nota(nota)
                    procesar_nota(nota)
                ELSE
                    null;
            END SELECT;
        END LOOP;
    END Medico;

    TASK TYPE Enfermera IS;
    TASK BODY Enfermera;
        pedido:Pedido;
        nota:Nota;
    BEGIN
        LOOP
            pedido = tomar_pedido()
            SELECT 
                Medico.atender_enfermera(pedido)
            ELSE
                nota = generar_nota(pedido)
                Escritorio.recibir_pedido(nota)
        END LOOP;
    END Enfermera;



    TASK Escritorio IS;
        ENTRY recibir_nota()
        ENTRY retirar_nota()
    TASK BODY Escritorio;
        Cola notas;
    BEGIN
        SELECT
            when (!notas.isEmpty()) => ACCEPT retirar_nota(nota:Nota OUT) DO
                nota = notas.pop()
            END retirar_nota;
        OR
            ACCEPT recibir_nota(nota:Nota IN) DO
                notas.push(nota)
            END recibir_nota;        
        END SELECT;
    END Escritorio;


    TASK TYPE Persona IS;
    TASK BODY Persona;
        pedido:Pedido;
        atendido:boolean;
        intentos:int;
    BEGIN
        intentos = 0;
        atendido = false;
        pedido = generar_pedido();
        LOOP (not atendido)
            SELECT
                Medico.atender_persona(pedido);
                atendido = true;
            OR DELAY(300)
                DELAY(600);
                intentos++;
                IF (intentos == 3) THEN
                    atendido = true;
                END IF;
        END LOOP;
    END Persona;

    v_enfermeras = array (1..E) of Enfermera;
    v_personas = array (1..P) of Persona;

BEGIN
    NULL;
END Clinica;
```

## Ejercicio 5
###   En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una  conoce  previamente  a  que  equipo  pertenece).  Cuando  las  personas  van  llegando esperan  con  los  de  su  equipo  hasta  que  el  mismo  esté  completo  (hayan  llegado  los  4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de  1,  2  o  5  pesos)  y  se  suman  los  montos  de  las  60  monedas  conseguidas  en  el  grupo.  Al finalizar  cada  persona  debe  conocer  el  grupo  que  más  dinero  junto.  Nota:  maximizar  la concurrencia.  Suponga  que  para  simular  la  búsqueda  de  una  moneda  por  parte  de  una persona existe una función Moneda() que retorna el valor de la moneda encontrada. 
 
```ada
PROCEDURE Playa IS;

    TASK Coordinador IS;
        ENTRY termino_equipo(id:int IN, monto:int IN)
        ENTRY quien_gana(id:int OUT);
    TASK BODY Coordinador;
        maximo:int
        ganador:int
    BEGIN
        maximo = -1;
        FOR i IN 1..5 LOOP;
            ACCEPT termino_equipo(id:int IN, monto:int IN) DO
                IF (monto > maximo) THEN
                    maximo = monto;
                    ganador = id
                END IF;
            END termino_equipo;
        END FOR;
        FOR i IN 1..20 LOOP;
            ACCEPT quien_gana(id:int OUT) DO
                id = ganador
            END quien_gana;
        END FOR;
    END Coordinador;

   TASK TYPE Equipo IS;
        ENTRY recibir_identificacion(numero:int IN)
        ENTRY llega_persona()
        ENTRY empezar()
        ENTRY sumar_total(monto:int IN)
    TASK BODY Equipo;
        id:int;
        total:int;
    BEGIN
        total = 0;
        ACCEPT recibir_identificacion(numero:int IN) DO
            id = numero;
        END recibir_identificacion;
        FOR i IN 1..4 LOOP
            ACCEPT llega_persona()
        END FOR;
        FOR i IN 1..4 LOOP
            ACCEPT empezar()
        END FOR;
        FOR i IN 1..4 LOOP
            ACCEPT sumar_total(monto:int IN) DO
                total += monto;
            END sumar_total;
        END FOR;
        Coordinador.termino_equipo(id, total)
    END Equipo;


    TASK TYPE Persona IS;
    TASK BODY Persona;
        total:int;
        equipo:int;
        ganador:int;
    BEGIN
        equipo = elegir_equipo()
        total = 0;
        v_equipos(equipo).llega_persona()
        v_equipos(equipo).empezar()
        FOR i IN 1..15 LOOP;
            total += Moneda()
        END FOR;
        v_equipos(equipo).sumar_total(total)
        Coordinador.quien_gana(ganador)
    END Persona;

    v_equipos = array (1..5) of Equipo;
    v_personas = array (1..20) of Persona;

BEGIN
    FOR i IN 1..5 LOOP;
        v_equipos(i).recibir_identificacion(i)
    END FOR;
END Playa;
```

## Ejercicio 5 (no numerado)
### En  un  sistema  para  acreditar  carreras  universitarias,  hay  UN  Servidor  que  atiende  pedidos de  U  Usuarios  de  a  uno  a  la  vez  y  de  acuerdo  con  el  orden  en  que  se  hacen  los  pedidos. Cada  usuario  trabaja  en  el  documento  a  presentar,  y  luego  lo  envía  al  servidor;  espera  la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento). 

```ada



```

## Ejercicio 6
### Se  debe  calcular  el  valor  promedio  de  un  vector  de  1  millón  de  números  enteros  que  se encuentra  distribuido  entre  10  procesos  Worker  (es  decir,  cada Worker  tiene  un  vector  de 100  mil  números).  Para  ello,  existe  un  Coordinador  que  determina  el  momento  en  que  se debe  realizar  el  cálculo  de  este  promedio  y  que,  además,  se  queda  con  el  resultado.
Nota: maximizar la concurrencia; este cálculo se hace una sola vez. 

```ada



```

## Ejercicio 7
### Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se  asemeja  a  TEST  en  su  BD;  al  final  del  procesamiento,  el  especialista  debe  conocer  el código  de  la  huella  con  mayor  valor  de  similitud  entre  las  devueltas  por  los  8  servidores. Cuando  ha  terminado  de  procesar  una  huella  comienza  nuevamente  todo  el  ciclo.  Nota: suponga  que  existe  una  función  Buscar(test,  código,  valor)  que  utiliza  cada  Servidor  donde recibe  como  parámetro  de  entrada  la  huella  test,  y  devuelve  como  parámetros  de  salida  el código  y  el  valor  de  similitud  de  la  huella  más  parecida  a  test  en  la  BD  correspondiente. Maximizar la concurrencia y no generar demora innecesaria. 

```ada



```

## Ejercicio 8
### Una  empresa  de  limpieza  se  encarga  de  recolectar  residuos  en  una  ciudad  por  medio  de  3 camiones.  Hay  P  personas  que  hacen  reclamos  continuamente  hasta  que  uno  de  los camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a que  llegue  un  camión;  si  no  pasa,  vuelve  a  hacer  el  reclamo  y  a  esperar  a  lo  sumo  15 minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte los  residuos.  Sólo  cuando  un  camión  llega,  es  cuando  deja  de  hacer  reclamos  y  se  retira. Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos ha hecho sin ser atendido. Nota: maximizar la concurrencia. 


```ada



```