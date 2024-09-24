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

#### a. Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones serán necesarios/convenientes para resolverlo. 

```c



```

#### b. Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.



## Ejercicio 3
###

####

```c



```

## Ejercicio 4
###

####

```c



```

## Ejercicio 5
###

####

```c



```

## Ejercicio 6
###

####

```c



```

## Ejercicio 7
###

####

```c



```

## Ejercicio 8
###

####

```c



```

## Ejercicio 9
###

####

```c



```

## Ejercicio 10
###

####

```c



```