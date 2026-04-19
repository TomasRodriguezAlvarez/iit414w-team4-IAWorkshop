Business Question

¿Es posible predecir cuántos puntos de campeonato aportará un piloto en un Gran Premio específico basándonos en su posición de salida (grid) y el rendimiento histórico del equipo? 
Esta predicción es vital para que el Director de Estrategia pueda calcular el retorno de inversión de las actualizaciones del monoplaza y priorizar la defensa de posiciones críticas en carrera que aseguren el flujo de caja por premios de la FIA.

Target Variable
La variable objetivo será points (puntos obtenidos al final de la carrera). Se define como una variable numérica continua que va desde 0 hasta 25 (o 26 si se incluye la vuelta rápida), representando el resultado tangible del piloto para el Campeonato de Constructores.

Metric
Utilizaré MAE (Mean Absolute Error). Esta métrica es la más apropiada para este problema de negocio porque mide el error en las mismas unidades que el objetivo (puntos). Para un jefe de equipo, es mucho más intuitivo entender que el modelo "se equivoca por un promedio de 2.5 puntos" que interpretar métricas cuadráticas como el RMSE, las cuales penalizarían desproporcionadamente los errores grandes.

Rejected Alternative
Consideré utilizar un enfoque de Clasificación Binaria (Top-10 Finish: Sí/No), pero lo rechacé por ser insuficiente para la toma de decisiones estratégicas. En la Fórmula 1, la diferencia financiera y competitiva entre un P1 (25 pts) y un P10 (1 pt) es abismal. Un modelo binario trataría ambos resultados como un "éxito" por igual, ocultando el riesgo de perder 24 puntos críticos. Al elegir regresión, mantenemos la granularidad necesaria para distinguir entre un podio y un simple ingreso en la zona de puntos.