# ArquiSelect â€” Entrega Final

Sistema experto de **caja blanca** (SWI-Prolog + Python/Tkinter con PySwip) que recomienda un estilo de arquitectura de software entre 11 estilos y explica por quÃ©. Este entregable agrega:

- **(A) Lenguaje natural con DCG:** el usuario describe su proyecto con sus palabras y Prolog lo traduce a requisitos.
- **(B) Modo Experto:** reglas lÃ³gicas, arquitecturas y vocabulario nuevos que se inyectan en caliente con `assertz` y quedan guardados en disco.
- **(C) Pruebas y evidencia:** `pruebas_entregable3.py` â†’ `evidencia_entregable3.txt`.

**Regla del proyecto:** toda la lÃ³gica vive en Prolog. Python solo envÃ­a datos, invoca predicados y muestra respuestas.

## EjecuciÃ³n

Requisitos: Python 3.9+, SWI-Prolog 9.x en el PATH (misma arquitectura de 64 bits) y `pip install pyswip`.

```bash
python main.py                    # aplicaciÃ³n
python pruebas_entregable3.py     # pruebas del entregable 3 (tambiÃ©n corre pruebas.py)
python pruebas.py                 # pruebas del entregable 2
python motor.py                   # versiÃ³n por consola del entregable 1
```

## Roles y PIN

En la pantalla de **Inicio** estÃ¡ el selector **ROL: ðŸ‘¤ Usuario / ðŸ›¡ Experto**. El rol actual aparece en el encabezado de todas las vistas.

- **Usuario** (por defecto): evaluaciÃ³n, historial y recarga de la base. No tiene acceso al Modo Experto.
- **Experto:** al elegirlo se pide un PIN. El PIN por defecto es **`experto2026`**. Se puede cambiar definiendo la variable de entorno `ARQUISELECT_PIN` antes de abrir la app, por ejemplo `set ARQUISELECT_PIN=otroPin` en Windows. Con el rol Experto aparece la tarjeta **Modo Experto** en Inicio, y el botÃ³n *Crear arquitectura nuevaâ€¦* de la evaluaciÃ³n abre esa vista.

> âš  **Es un control de acceso de demostraciÃ³n, no seguridad real.** El hash SHA-256 del PIN estÃ¡ en `main.py` y quien tenga acceso a los archivos puede cambiarlo o editar el `.pl` directamente.

## Archivos

| Archivo | Contenido |
|---|---|
| `main.py` | AplicaciÃ³n Tkinter: `VistaInicio` (rol), `VistaEvaluacion` (texto libre, casillas, preguntas guiadas, resumen y *Â¿Por quÃ©?*), `VistaModoExperto` (4 pestaÃ±as) y `VistaHistorial`. Incluye `PuenteProlog`, que hace todas las consultas. |
| `base_conocimiento.pl` | Hechos y reglas (secciones 1-8 de los entregables 1 y 2). La **secciÃ³n 9** contiene las reglas expertas (`ensenar_regla/8`). Carga `lenguaje_natural.pl`. |
| `lenguaje_natural.pl` | Normalizador, gramÃ¡tica DCG, lÃ©xico (`lexico/3`), umbrales numÃ©ricos y sinÃ³nimos en caliente. |
| `conocimiento_aprendido.pl` | Lo enseÃ±ado en el Modo Experto: estilos, reglas (clÃ¡usulas escritas con `portray_clause/2`) y sinÃ³nimos. Se crea al guardar lo primero. Cada escritura es atÃ³mica (`.tmp` + `rename`) y deja una copia `.bak`. Si estÃ¡ daÃ±ado, la app arranca igual y avisa. |
| `pruebas_entregable3.py` / `evidencia_entregable3.txt` | Pruebas del entregable 3 y su salida real. |
| `pruebas.py` / `evidencia_pruebas.txt` | Pruebas del entregable 2 (siguen pasando). |
| `pruebas_contradicciones.py` / `evidencia_contradicciones.txt` | Contradicciones duras/tensiones (secciÃ³n 10 de la base), entradas hostiles al DCG, lÃ­mites de longitud y tiempo, y prueba de la interfaz. |
| `GUIA_SUSTENTACION.md` | Cortes probados uno por uno, modificaciones en vivo y preguntas probables. |
| `arbol_logico.txt` | Ãrbol de llamadas generado del cÃ³digo real por `generar_arbol_logico.pl`. |
| `probar_cortes.py` + `bateria_cortes.pl` â†’ `evidencia_cortes.txt`, `tabla_cortes.md` | Quita cada `!` en una copia temporal y compara 100 consultas. |
| `motor.py` | Consola del entregable 1. |
| `historial_evaluaciones.json` | Evaluaciones guardadas. |

## QuÃ© entiende la caja "Describe tu proyecto"

Escriba en la tarjeta **DESCRIBE TU PROYECTO** y pulse **ðŸ”Ž Interpretar**. Pasa lo siguiente:

1. Se marcan las casillas de los requisitos entendidos.
2. Lo negado pasa a "No aplica" y arrancan las preguntas guiadas.
3. Aparecen las lÃ­neas **âœ” EntendÃ­** y **âš  No entendÃ­**.
4. **Ver anÃ¡lisis sintÃ¡ctico** muestra el Ã¡rbol de la DCG.

Interpretar de nuevo reemplaza lo detectado antes.

La gramÃ¡tica divide el texto en *sintagmas*. Cada sintagma tiene conectores opcionales (`y, e, o, ademÃ¡s, tambiÃ©n, con, pero, ","`), una negaciÃ³n opcional (`no, sin, ni, tampoco, nunca`), rellenos que se omiten (artÃ­culos, verbos comunes y sustantivos genÃ©ricos como *el equipo* o *la aplicaciÃ³n*) y un nÃºcleo. MayÃºsculas, tildes y signos no importan.

| Requisito | Ejemplos de formulaciones (hay mÃ¡s en `lenguaje_natural.pl`) |
|---|---|
| equipo_pequeno | equipo pequeÃ±o/reducido/chico, pocos desarrolladores, **equipo de 3 personas**, **somos 4** (< 5) |
| equipo_grande | equipo grande/numeroso, varios equipos, **equipo de 20 personas** (â‰¥ 10) |
| sin_experiencia_devops | sin experiencia en devops, no tiene/tenemos experiencia en devops, no sabemos devops |
| presupuesto_bajo | poco presupuesto, presupuesto limitado/ajustado/bajo, bajo presupuesto, pocos recursos, poco dinero |
| entrega_rapida | entregar rÃ¡pido/pronto, entrega rÃ¡pida, poco tiempo, plazo corto, cuanto antes, **entregar en 3 semanas** (â‰¤ 6) |
| muchos_usuarios | muchos usuarios (al mismo tiempo), miles de usuarios, alto trÃ¡fico, **mÃ¡s de 1000 usuarios**, **10 mil usuarios** |
| carga_variable | carga variable, picos de demanda/trÃ¡fico, demanda impredecible |
| procesos_largos | procesos largos, tareas largas/pesadas, procesamiento por lotes |
| latencia_critica | baja latencia, tiempo real, respuesta inmediata, **responder en menos de 100 ms** |
| disponibilidad_alta | alta disponibilidad, disponible las 24 horas, 24/7, sin caÃ­das, tolerante a fallos |
| despliegue_independiente | despliegue independiente, desplegar (partes) por separado |
| dominio_complejo | dominio complejo, muchas reglas de negocio, lÃ³gica de negocio compleja |
| integracion_asincrona | comunicaciÃ³n asÃ­ncrona, colas de mensajes, mensajerÃ­a, eventos |
| consistencia_estricta | consistencia estricta/fuerte, transacciones, integridad de datos |
| mantenibilidad_largo_plazo | largo plazo, fÃ¡cil de mantener, muchos aÃ±os, mantenibilidad |
| testabilidad_alta | pruebas automÃ¡ticas/unitarias, fÃ¡cil de probar, testabilidad |
| independencia_tecnologica | cambiar la base de datos, cambiar el framework, sin reescribir todo |
| integracion_sistemas_heredados | sistemas heredados/legados/legacy/existentes |
| separar_interfaz_logica | separar interfaz y lÃ³gica, separaciÃ³n de responsabilidades, modelo vista controlador |
| datos_centralizados | datos centralizados, base de datos central, servidor central, varios clientes |

Las frases en **negrita** son **frases numÃ©ricas**: la regla aritmÃ©tica se evalÃºa dentro de la DCG y los umbrales estÃ¡n en `umbral_numerico/3`. Se aceptan nÃºmeros escritos con letras (*tres*, *veinte*, *diez mil*).

**Negaciones:**

- *"No necesitamos alta disponibilidad"* â†’ `disponibilidad_alta` va a **Negados**.
- *"Sin experiencia en DevOps"* sÃ­ es un requisito, porque el lÃ©xico se prueba antes que la negaciÃ³n.
- Algunas frases niegan por sÃ­ mismas (polaridad `niega`): *carga estable*, *pocos usuarios*, *presupuesto amplio*, *sin prisa*.

## Modo Experto (pestaÃ±as)

1. **Regla nueva:** id, explicaciÃ³n (obligatoria), condiciones (que deben cumplirse), exclusiones (traducidas a `\+`), efecto (*recomendar* con +1 a +5 o *descartar*) y estilo afectado. Python envÃ­a solo esos datos. Prolog valida (id repetido, estilo existente, requisitos existentes, al menos una condiciÃ³n, peso entre 1 y 5, sin solapes) y construye la clÃ¡usula `regla_experta/5`, que se inserta con `assertz` y se anexa al archivo. Nunca se ejecuta texto Prolog escrito por el usuario.
2. **Arquitectura nueva:** el formulario de estilos nuevos del entregable 2.
3. **Vocabulario:** frase â†’ requisito (afirma/niega). La caja de texto la entiende de inmediato, sin reiniciar.
4. **Reglas y conocimiento aprendido:** reglas, estilos y sinÃ³nimos aprendidos, cada uno con ðŸ—‘ y confirmaciÃ³n.

Las reglas disparadas suman su bonus en `puntaje/3` o descartan el estilo en `motivo_descarte/3`. Ranking, candidatos, descartes, *Â¿QuÃ© falta?* y la secciÃ³n **Â¿Por quÃ©? (auditorÃ­a)** las muestran sin reiniciar.

## Guion de demostraciÃ³n

1. **Rol Usuario.** Escriba *"somos un equipo de tres personas con poco presupuesto y necesitamos entregar rapido"* y pulse **Interpretar**. Se marcan 3 casillas y arrancan las preguntas guiadas. Abra **Ver anÃ¡lisis sintÃ¡ctico**.
2. **Rol Experto (PIN `experto2026`).** En **Modo Experto â†’ Regla nueva** cree `mvc_para_equipos_pequenos`:
   - condiciones: *Equipo reducido* y *Separar interfaz de usuario y lÃ³gica*
   - exclusiÃ³n: *Dominio complejo*
   - efecto: recomendar +3, estilo MVC

   Con esas dos casillas, MVC pasa de 5 a 8 puntos y queda primero. En **Resumen â†’ Â¿Por quÃ©? (auditorÃ­a)** aparece la regla.
3. Cierre la app y vuelva a abrirla: la regla sigue cargada (`conocimiento_aprendido.pl`).
4. Elimine la regla desde **Reglas y conocimiento aprendido**.
#   A R Q U I S E L E C T 
 
 