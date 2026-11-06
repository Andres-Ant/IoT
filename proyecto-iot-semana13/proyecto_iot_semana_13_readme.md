# Proyecto IoT Semana 13: Visualización histórica y automatización con IFTTT

## Descripción del Proyecto
Este proyecto consiste en la construcción de un flujo completo de IoT para monitorear la luminosidad de un aula de clases de las Unidades Tecnológicas de Santander (UTS). El sistema captura los datos mediante un microcontrolador ESP32 y los envía a la nube. El objetivo principal es alertar al coordinador académico mediante una notificación push en su celular cuando ocurra una anomalía en la iluminación, específicamente si la luminosidad desciende por debajo de 300 lux aproximados. Esto automatiza la supervisión sin requerir que el coordinador vigile el panel de control constantemente.

## Integrantes
* Luis Felipe Parra Salamanca
* Andres Felipe Antivar Mora

## Diagrama de Conexión del Sensor al ESP32
* **Sensor LDR:** Conectado al pin GPIO 35 (ADC1) del ESP32.
* **Circuito:** Se configuró un divisor de tensión utilizando el LDR y una resistencia de 10 kΩ.
* *Nota:* El circuito incluye además un sensor DHT11 en el pin 4, un HC-SR04 (Trigger pin 13, Echo pin 12), LEDs indicadores y un relé en el pin 26.

## Explicación del Flujo de Datos
El flujo del sistema opera bajo el siguiente esquema automatizado:
1. **Sensor (LDR):** Forma un divisor de tensión donde el ESP32 lee el valor analógico (0-4095) y realiza un mapeo invertido para convertirlo a un valor aproximado en lux (0-1000).
2. **ESP32:** Procesa la lectura y la publica mediante el protocolo MQTT cada 15 segundos hacia el tópico `Andres_Ant/feeds/luminosidad`.
3. **Adafruit IO:** Actúa como el broker MQTT en la nube, recibiendo el dato, guardándolo en el feed `luminosidad` y graficándolo en un *Line Chart* para análisis histórico.
4. **IFTTT (Applet):** Utiliza un Trigger que monitorea constantemente el feed de Adafruit IO evaluando la condición `luminosidad less than 300`.
5. **Notificación Push:** Cuando se cumple la condición, la acción (That) envía una notificación de alta prioridad desde la app de IFTTT al celular del usuario con el mensaje que incluye el valor actual de luminosidad.

## Evidencias (Capturas de Pantalla)
A continuación, se referencian las capturas solicitadas ubicadas en la carpeta `evidencias/`:
* **Dashboard Adafruit IO:** `![Dashboard en Adafruit IO](evidencias/dashboard_adafruit.png)`
* **Gráfico Histórico:** `![Gráfico Line Chart](evidencias/grafico_historico.png)`
* **Applet IFTTT Activo:** `![Applet configurado](evidencias/applet_ifttt.png)`
* **Notificación Push en Celular:** `![Notificación recibida](evidencias/notificacion_push.png)`

## Código Fuente Comentado
El código completo se encuentra en `codigo/esp32_luminosidad.ino`. A continuación, se destaca la sección fundamental de lectura, mapeo invertido y publicación MQTT:

```cpp
// Lectura cruda y escala aproximada 0-1000 para IFTTT
luminosidad = analogRead(LDR_PIN);
// Mapeo invertido: tapado = valor bajo, destapado = valor alto
luxAprox = constrain(map(luminosidad, 0, 4095, 1000, 0), 0, 1000);

// Umbral de alerta compartido con el Applet de IFTTT
const int LUX_ALERTA = 300;

// Función de publicación MQTT cada 15 segundos
void publicarDatos() {
  if (!client.connected() || !datosValidos) return;
  char buf[12];
  snprintf(buf, sizeof(buf), "%d", luxAprox);
  client.publish(TOPIC_LUZ, buf);  // Publicación que lee IFTTT
  Serial.print("[MQTT] Publicado en Adafruit IO | luminosidad = ");
  Serial.println(luxAprox);
}
```

## Conclusiones sobre la Utilidad de la Automatización en IoT
La integración de IFTTT transforma el proyecto de un entorno meramente observable a un sistema verdaderamente automatizado. Al delegar la lógica de alerta a la nube, el microcontrolador ESP32 optimiza sus recursos (limitándose a medir y publicar), mientras el sistema notifica fallas críticas por sí solo. Esto elimina la necesidad de intervención humana constante frente al dashboard, logrando un monitoreo inteligente y eficiente.