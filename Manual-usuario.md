# Manual de Usuario

## Introducción a la aplicación

# ReportGenerator.py
La aplicación desarrollada tiene como propósito generar reportes en formato PDF relacionados con la gestión de equipos deportivos, jugadores, partidos y estadísticas individuales.
Utiliza Flask como framework backend en Python, integrando funcionalidades de generación de documentos con la librería ReportLab, y almacenamiento de los reportes en una base de datos MongoDB.

A través de distintos módulos, el sistema permite generar reportes personalizados como:

♥ Listado de equipos registrados.
♥ Roster de jugadores por equipo o por partido.
♥ Historial de partidos jugados.

Estadísticas individuales de los jugadores.

Cada reporte se genera dinámicamente a partir de los datos enviados desde el frontend o servicios externos, 
y se guarda en la base de datos para futuras consultas o auditorías.

## Requisitos del Sistema
# Requisitos de software:
  ♥ Python 3.10 o superior
  ♥ Flask
  ♥ ReportLab
  ♥ Requests
  ♥ PyMongo
  ♥ Biblioteca bson para manejo de binarios
  ♥ MongoDB en ejecución local o remota

# Requisitos de hardware (mínimos):
  ♥ Procesador Dual Core 2.0 GHz
  ♥ 4 GB de memoria RAM
  ♥ Espacio libre en disco: 500 MB
  
## Uso de los Módulos Principales
# Módulo de Marcador

Permite registrar y visualizar los resultados de los partidos, incluyendo equipos locales, visitantes, fechas y puntajes.
Los datos se utilizan para generar reportes del historial de partidos o estadísticas individuales.

Funciones relacionadas:
  ♥ Generar_Historial_Partidos(datos)
Crea un PDF con el registro de todos los partidos jugados, mostrando los equipos, la fecha y el marcador. 

# Módulo de Reportes

Responsable de crear y almacenar los reportes PDF en MongoDB.
Cada reporte incluye encabezados personalizados, tablas con datos organizados y, en algunos casos, logotipos o imágenes descargadas desde URLs externas.

# Funciones principales:
  ♥ Generar_Equipos(datos) → Genera un listado con equipos registrados (nombre, localidad y logo).
  ♥ Generar_Jugadores(datos, Equipo) → Crea el roster de jugadores de un equipo determinado.
  ♥ Generar_Roster_Partido(datos) → Muestra los jugadores de ambos equipos en un partido específico.
  ♥ Generar_Roster_Partido_delado(token, id_partido) → Genera el roster comparativo de ambos equipos en formato lado a lado.
  ♥ Generar_Reporte_Estadisticas_Jugador(datos) → Presenta un reporte con las estadísticas individuales de un jugador, incluyendo anotaciones y faltas totales.

Todos los reportes son exportados en formato PDF, guardados en memoria binaria (Binary) y almacenados en la colección Reporteria dentro de MongoDB.

## app.py
Este código define una API REST con Flask, usada para:

  ♥ Autenticar un usuario y obtener un token.
  ♥ Generar diferentes tipos de reportes (equipos, jugadores, partidos, estadísticas).
  ♥ Acceder a usuarios almacenados en una base de datos MongoDB.
  ♥ Consumir información de una fuente externa mediante un token.
  
# Estructura general
  ♥ requests: para hacer peticiones HTTP a servicios externos.
  ♥ database: módulo local que contiene la conexión MongoDB (variable db).
  ♥ data: módulo que parece manejar autenticación y obtención de datos externos.
  ♥ ReportGenerator: módulo que genera los reportes en distintos formatos.

# Estructura y Flujo General
  ♥ El servidor Flask inicia y espera peticiones.
  ♥ El usuario o sistema externo hace una solicitud a una ruta específica (por ejemplo: /Reporte/Equipos).
  ♥ El sistema recibe los datos en formato JSON, los procesa, y llama a una función del módulo ReportGenerator.
  ♥ Se genera un PDF con la información correspondiente, se almacena en la base de datos, y se devuelve al usuario como descarga o respuesta.
  ♥ Algunas rutas también interactúan con la base de datos para mostrar usuarios o consumir datos externos.

## Principales Endpoints (Rutas)
  ♥ /
Método: GET
Función: Verifica que la API esté activa.
  ♥ /auth
Método: GET
Función: Autentica a un usuario.
  - Simula un login con credenciales (nombre: "Macario", contraseña: "apple123").
  - Llama a dt.obtener_token() para obtener un token de acceso.
  - Asigna el usuario actual con dt.set_user(usuario).
  - Uso real: serviría para autenticar usuarios antes de generar reportes.
  ♥ /Reporte/Equipos
Método: POST
Función: Genera un reporte PDF de equipos registrados.
Entrada: JSON con los datos de los equipos.
Procesa: RG.Generar_Equipos(data)
Salida: Archivo PDF (descargable o guardado en base de datos).
  ♥ /Reporte/Jugadores?equipo=nombre
Método: POST
Función: Genera un reporte PDF con los jugadores de un equipo específico.
Entrada: JSON con los datos de los jugadores.
Salida: PDF con lista de jugadores.
  ♥ /Reporte/Partidos
Método: POST
Función: Genera un reporte de historial de partidos con sus resultados.
Procesa: RG.Generar_Historial_Partidos(datos)
  ♥ /Reporte/Partido/Roster
Método: POST
Función: Crea un reporte de roster de un partido (alineaciones y jugadores).
Procesa: RG.Generar_Roster_Partido(datos)
  ♥ /Reporte/Estadistica/Jugador
Método: POST
Función: Genera un reporte individual de estadísticas por jugador.
Procesa: RG.Generar_Reporte_Estadisticas_Jugador(data)
  ♥ /usuarios
Método: GET
Función: Devuelve todos los usuarios registrados en la base de datos MongoDB.
Consulta: db.usuarios.find({}, {'_id': 0})
  ♥ /externa
Método: GET
Función: Consume datos desde una fuente externa usando el token de autenticación.
Procesa:
   - Obtiene el token con dt.obtener_token().
   - Recupera datos con dt.Obtener_Jugadores(token).

## data.py
Este módulo realiza peticiones REST al servidor .NET que maneja toda la lógica del sistema deportivo.
Sirve de puente entre Flask (tu API intermedia en Python) y ese backend.

#Funciones de consulta general
Todas las funciones siguen la misma estructura:
   - Arman el encabezado con el token.
   - Hacen la petición GET al endpoint.
   - Validan el status_code.
   - Devuelven los datos JSON o un mensaje de error.
     
## database.py
El archivo database.py es el módulo encargado de conectarse a la base de datos MongoDB y de gestionar la descarga de reportes PDF que han sido previamente almacenados por el sistema.

## Funciones principales

# Conexión a MongoDB
El módulo define la URI de conexión y establece el acceso a la base de datos Reporteria, donde se guardan los reportes generados por el sistema.
# get_mongo_connection()
Esta función devuelve una conexión nueva a MongoDB usando la variable de entorno MONGO_URI o un valor por defecto.
Propósito:
Permitir que otros módulos (como los generadores de PDF) puedan conectarse fácilmente a la base de datos sin repetir código.
# Descarga de reportes PDF desde MongoDB
Descargar_Reporte(nombre_reporte)

   - Busca un documento dentro de la colección Reporteria cuyo campo nombre_reporte coincida con el parámetro recibido.
   - Si lo encuentra, extrae el PDF almacenado en binario (archivo_pdf).
   - Usa send_file() para devolver el archivo al cliente como descarga directa.
     
## encabezado.py
El archivo define la función encabezado_pdf, utilizada para insertar un encabezado institucional o corporativo en los reportes PDF generados por el sistema.
Incluye un logo, un título personalizado y la fecha y hora de generación, con un diseño limpio y profesional.
# Función principal: encabezado_pdf()
¿Qué hace?
Inserta en el PDF un encabezado con:
   - Logo institucional
   - Título del reporte
   - Fecha y hora de generación
   - Agrega una línea decorativa debajo del encabezado.
   - Mantiene un diseño profesional y uniforme en todos los reportes.
# Estas librerías permiten:
   - reportlab → Crear documentos PDF dinámicos.
   - colors y TableStyle → Definir estilos y colores en tablas.
   - inch → Manejar medidas en pulgadas.
   - Image, Paragraph, Spacer, Table → Añadir elementos al documento PDF.
   - datetime → Insertar la fecha y hora actual.
   - os → Verificar la existencia del archivo del logo.
