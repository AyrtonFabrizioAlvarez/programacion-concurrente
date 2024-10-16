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
chan pedido_empleado(int);
chan siguiente_cliente(int)

Process Cliente[id: 1 to N]:
    send atencion(id)

Process Admin:
    int id_empleado;
    int id_cliente;
    while (true):
        receive pedido_empleado(id_empleado)
        if (empty(atencion)):
            id_cliente = -1
        else:
            receive atencion(id_cliente)
            send siguiente_cliente[id_empleado](id_cliente)

Process Empleado[id: 1 to 2]:
    int proximo;
    while (true):
        send pedido_cliente(id)
        receive siguiente_cliente(proximo)
        if (proximo == -1):
            delay(15)
        else:
            atender(proximo)
```


## Ejercicio 2
### Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia. 

```c



```


## Ejercicio 3
### Se  debe  modelar  el  funcionamiento  de  una  casa  de  comida  rápida,  en  la  cual  trabajan  2 cocineros  y  3  vendedores,  y  que  debe  atender  a  C  clientes.  El  modelado  debe  considerar que: 
 - Cada cliente realiza un pedido y luego espera a que se lo entreguen. 
 - Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto). 
 - Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente. 
Nota: maximizar la concurrencia.

```c



```


## Ejercicio 4
### 


## Ejercicio 5
### 