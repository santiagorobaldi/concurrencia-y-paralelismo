# Práctica 1 — Concurrencia y Paralelismo

## Ejercicio 1

Para el siguiente programa concurrente, todas las variables están inicializadas en 0. Indicar cuál/es de las siguientes opciones son verdaderas:

- a) En algún caso el valor de x al terminar el programa es 56.
- b) En algún caso el valor de x al terminar el programa es 22.
- c) En algún caso el valor de x al terminar el programa es 23.

```
P1::
  if (x == 0) then
    y := 4 * 2
    x := y + 2

P2::
  if (x > 0) then
    x := x + 1

P3::
  x := (x * 3) + (x * 2) + 1
```

### Solución

Acciones de grano fino relevantes:
- **P1**: lee x (if), escribe x (x := y+2 = 10 siempre)
- **P2**: lee x (if), lee x, escribe x (x := x+1)
- **P3**: lee x (para x\*3), lee x (para x\*2), escribe x

Valores alcanzables para x antes de que P3 lea: **0, 10, 11**

---

#### a) x = 56 ✓

P3 necesita leer x=11 dos veces: `3(11) + 2(11) + 1 = 56`

| Paso | Proceso | Acción | x |
|------|---------|--------|---|
| 1 | P1 | if(x==0) → true | 0 |
| 2 | P1 | x := 10 | 10 |
| 3 | P2 | if(x>0) → true | 10 |
| 4 | P2 | lee x → 10 | 10 |
| 5 | P2 | x := 11 | 11 |
| 6 | P3 | lee x → 11 (para x\*3) | 11 |
| 7 | P3 | lee x → 11 (para x\*2) | 11 |
| 8 | P3 | x := 33+22+1 = **56** | **56** |

---

#### b) x = 22 ✓

P3 lee x=0 y x=10 → `3(0) + 2(10) + 1 = 21`, luego P2 hace x+1=22

| Paso | Proceso | Acción | x |
|------|---------|--------|---|
| 1 | P1 | if(x==0) → true | 0 |
| 2 | P3 | lee x → 0 (para x\*3) | 0 |
| 3 | P1 | x := 10 | 10 |
| 4 | P3 | lee x → 10 (para x\*2) | 10 |
| 5 | P3 | x := 0+20+1 = 21 | 21 |
| 6 | P2 | if(x>0) → true | 21 |
| 7 | P2 | lee x → 21 | 21 |
| 8 | P2 | x := **22** | **22** |

---

#### c) x = 23 ✓

P3 lee x=0 y x=11 → `3(0) + 2(11) + 1 = 23`

| Paso | Proceso | Acción | x |
|------|---------|--------|---|
| 1 | P1 | if(x==0) → true | 0 |
| 2 | P3 | lee x → 0 (para x\*3) | 0 |
| 3 | P1 | x := 10 | 10 |
| 4 | P2 | if(x>0) → true | 10 |
| 5 | P2 | lee x → 10 | 10 |
| 6 | P2 | x := 11 | 11 |
| 7 | P3 | lee x → 11 (para x\*2) | 11 |
| 8 | P3 | x := 0+22+1 = **23** | **23** |

**Las tres opciones a), b) y c) son verdaderas.**

---

## Ejercicio 2

Dado un número N, verificar cuántas veces aparece en un arreglo de longitud M. Solución concurrente de grano grueso.

### Solución

```
int cont = 0;

Process Buscar[i = 0 to M-1]::
  if (A[i] == N) then
    <cont := cont + 1>
```

El `<>` protege el incremento porque es lectura + suma + escritura sobre una variable compartida.

---

## Ejercicio 3

Un SO mantiene 5 instancias de un recurso en una cola. Un proceso saca una instancia, la usa y la devuelve.

### Solución

```
int instancias = 5;

Process Usuario[i = 0 to N-1]::
  <await instancias > 0; instancias := instancias - 1>
  // usar el recurso
  <instancias := instancias + 1>
```

---

## Ejercicio 4

N personas deben imprimir un trabajo cada una.

### a) Una impresora, sin orden

```
bool impresora = true;

Process Personas[id = 0 to N-1]::
  <await impresora; impresora := false>
  imprimir(documento)
  <impresora := true>
```

---

### b) Respetando orden de llegada (con turnos)

```
int turno_actual = 1;
int siguiente = 1;

Process Personas[id = 0 to N-1]::
  int mi_turno;
  <mi_turno := siguiente; siguiente := siguiente + 1>
  <await turno_actual == mi_turno>
  imprimir(documento)
  <turno_actual := turno_actual + 1>
```

---

### c) Prioridad por edad

```
colaEspecial C;
int Siguiente = -1;

Process Personas[id = 0 to N-1]::
  int edad = ...;
  <if (Siguiente == -1) Siguiente := id
   else Agregar(C, edad, id)>
  <await (Siguiente == id)>
  imprimir(documento)
  <if (empty(C)) Siguiente := -1
   else Siguiente := Sacar(C)>
```

`Sacar(C)` devuelve el id de la persona de mayor edad esperando.

---

### d) Orden estricto por identificador

```
int Siguiente = 0;

Process Personas[id = 0 to N-1]::
  <await (Siguiente == id)>
  imprimir(documento)
  <Siguiente := Siguiente + 1>
```

---

### e) Con proceso Coordinador (basado en c)

```
colaEspecial C;
int Siguiente = -1;
bool permiso[N] = false;

Process Personas[id = 0 to N-1]::
  int edad = ...;
  <if (Siguiente == -1) Siguiente := id
   else Agregar(C, edad, id)>
  <await (permiso[id])>
  imprimir(documento)
  permiso[id] := false;
  <if (empty(C)) Siguiente := -1
   else Siguiente := Sacar(C)>

Process Coordinador::
  while(true)
    <await (Siguiente != -1)>
    permiso[Siguiente] := true
    <await (permiso[Siguiente] == false)>
```

---

## Ejercicio 5

P alumnos y 3 profesores. Cuando todos los alumnos llegaron comienza el examen. Los profesores corrigen en orden de entrega.

### Solución

```
int llegaron = 0;
int examenesCorregidos = 0;
queue colaExamenes;
int nota[P];
bool corregido[P] = false;

Process Alumno[id = 0 to P-1]::
  <llegaron := llegaron + 1>
  <await (llegaron == P)>
  // resuelve el examen
  <encolar(colaExamenes, id)>
  <await (corregido[id])>
  // lee nota[id]

Process Profesor[id = 0 to 2]::
  int alumno;
  while(examenesCorregidos < P)
    <await (not empty(colaExamenes)); alumno := desencolar(colaExamenes);
     examenesCorregidos := examenesCorregidos + 1>
    // corrige
    nota[alumno] := ...;
    corregido[alumno] := true;
```

---

## Ejercicio 6

Analizar si la siguiente solución cumple las 4 condiciones del problema de la Sección Crítica:

```
int turno = 1;

Process SC1::
  while (true)
    while (turno == 2) skip;
    SC;
    turno = 2;
    SNC;

Process SC2::
  while (true)
    while (turno == 1) skip;
    SC;
    turno = 1;
    SNC;
```

### Análisis

| Condición | ¿Cumple? | Justificación |
|-----------|----------|---------------|
| Exclusión mutua | ✓ | turno vale 1 o 2, nunca ambos acceden a SC simultáneamente |
| Ausencia de deadlock | ✓ | turno siempre vale 1 o 2, siempre uno puede avanzar |
| Ausencia de demora innecesaria | ✓ | el turno cambia antes de SNC, el otro puede entrar durante la SNC del primero |
| Entrada garantizada | ✓ | SNC es finita, el otro proceso siempre eventualmente cambia el turno |

**La solución cumple las 4 condiciones.**

---

## Ejercicio 7

Solución de grano fino con variables compartidas y proceso Coordinador para el problema de la Sección Crítica.

### Solución

Basada en el patrón Flags y Coordinador de la teoría.

```
int quiero[1:n] = ([n] 0);   // SC[i] avisa que quiere entrar
int permiso[1:n] = ([n] 0);  // Coordinador le da permiso a SC[i]

Process SC[i = 1 to n]::
  while (true)
    // protocolo de entrada
    quiero[i] = 1;                       // aviso que quiero entrar
    while (permiso[i] == 0) skip;        // espero permiso
    // sección crítica
    permiso[i] = 0;                      // bajo el permiso al salir
    quiero[i] = 0;                       // aviso que salí
    // sección no crítica

Process Coordinador::
  while (true)
    int i = 1;
    while (quiero[i] == 0)
      i = i mod n + 1;                   // busco cualquiera que quiera entrar
    permiso[i] = 1;                      // le doy permiso
    while (quiero[i] == 1) skip;         // espero que termine su SC
    permiso[i] = 0;
```

### Funcionamiento
- `quiero[i]` tiene dos roles: avisar que quiere entrar (=1) y avisar que terminó la SC (=0)
- `permiso[i]` es la señal del Coordinador hacia SC[i]
- El Coordinador recorre los procesos en round-robin buscando al primero que quiera entrar
- La exclusión mutua está garantizada porque el Coordinador atiende de a uno a la vez
