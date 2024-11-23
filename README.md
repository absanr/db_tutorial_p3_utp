
# Guía Completa: Análisis de Reseñas y Restaurantes con SQL Server

## Introducción

¡Hola a todos y bienvenidos a este tutorial! Hoy vamos a sumergirnos en un análisis práctico de datos de restaurantes y sus reseñas utilizando SQL Server. A lo largo de este video, integraremos conceptos clave como diseño de bases de datos, normalización, consultas avanzadas, funciones y triggers. Este ejemplo representa un caso de uso real que podrías encontrar en un entorno profesional.

## Objetivos

Nuestros objetivos para este tutorial son:

- **Comprender los datos disponibles**: Identificar las relaciones entre las tablas `restaurants` y `reviews`.
- **Aplicar consultas avanzadas**: Utilizar funciones nativas de SQL Server para responder preguntas clave de negocio.
- **Evaluar datos geoespaciales y sentimentales**: Analizar ubicaciones y sentimientos para obtener insights valiosos.
- **Automatización**: Implementar funciones y triggers para optimizar procesos repetitivos.

## Descripción de los Datos

### 1. Tabla `restaurants`

Esta tabla contiene información básica sobre los restaurantes:

- `id`: Identificador único del restaurante.
- `name`: Nombre del restaurante.
- `tag`: Categorías del restaurante (tipo de comida y características).
- `x`, `y`: Coordenadas geográficas (latitud y longitud).
- `district` y `IDDIST`: Distrito y su identificador asociado.
- `direction`: Dirección completa del restaurante.
- `stars`: Calificación promedio del restaurante en estrellas.
- `n_reviews`: Total de reseñas del restaurante.
- `min_price`, `max_price`: Rango de precios del menú.
- `platform`: Plataforma de origen de los datos.

### 2. Tabla `reviews`

Esta tabla contiene información detallada de las reseñas:

- `id_review`: Identificador único de la reseña.
- `review`: Texto completo de la reseña.
- `title`: Resumen o título de la reseña.
- `score`: Calificación otorgada (1-5).
- `likes`: Número de "me gusta" que recibió la reseña.
- `id_nick`: Identificador del usuario que realizó la reseña.
- `service`: Identificador del restaurante al que se refiere la reseña.
- `date`: Fecha de la reseña.
- `platform`: Plataforma de origen de la reseña.

### Estructura Relacional

La relación entre las tablas es fundamental para nuestro análisis:

- **Relación 1:N**: Un restaurante puede tener muchas reseñas.
- Las reseñas están vinculadas a los restaurantes a través de `id` en `restaurants` y `service` en `reviews`.

## Consultas y Análisis

### 1. Exploración Inicial de Datos

#### 1.1. Verificar los Restaurantes Disponibles

Para comenzar, vamos a visualizar todos los restaurantes disponibles:

```sql
SELECT * FROM restaurants;
```

Esto nos dará una visión general de los datos que tenemos en la tabla `restaurants`.

#### 1.2. Verificar las Primeras Reseñas

Ahora, veamos las primeras 10 reseñas para entender mejor su estructura:

```sql
SELECT TOP 10 * FROM reviews;
```

### 2. Métricas Clave

#### 2.1. Número de Reseñas por Restaurante

Queremos saber qué restaurantes tienen más reseñas:

```sql
SELECT r.name AS Restaurante, COUNT(re.id_review) AS TotalReseñas
FROM restaurants r
LEFT JOIN reviews re ON r.id = re.service
GROUP BY r.name
ORDER BY TotalReseñas DESC;
```

**Explicación**:

- Usamos un `LEFT JOIN` para incluir todos los restaurantes, incluso aquellos sin reseñas.
- Contamos el número de reseñas por restaurante.
- Ordenamos de mayor a menor para identificar los más comentados.

#### 2.2. Calificación Promedio por Restaurante

Veamos ahora cuáles son los restaurantes mejor calificados:

```sql
SELECT r.name AS Restaurante, AVG(re.score) AS CalificacionPromedio
FROM restaurants r
JOIN reviews re ON r.id = re.service
GROUP BY r.name
ORDER BY CalificacionPromedio DESC;
```

**Explicación**:

- Usamos `AVG(re.score)` para calcular la calificación promedio.
- Solo incluimos restaurantes que tienen reseñas (`JOIN` en lugar de `LEFT JOIN`).
- Ordenamos para destacar los restaurantes con mejores calificaciones.

#### 2.3. Restaurantes con Mayor Rango de Precios

Identifiquemos los restaurantes con la mayor diferencia de precios en su menú:

```sql
SELECT name, min_price, max_price, max_price - min_price AS RangoPrecios
FROM restaurants
ORDER BY RangoPrecios DESC;
```

**Explicación**:

- Calculamos el rango de precios restando `min_price` de `max_price`.
- Ordenamos para ver los restaurantes con la mayor variedad de precios.

### 3. Funciones Definidas por el Usuario

Las funciones nos permiten reutilizar código y simplificar nuestras consultas.

#### 3.1. Función para Determinar la Longitud de las Reseñas

Creemos una función que nos diga cuántos caracteres tiene una reseña:

```sql
CREATE FUNCTION LongitudResena(@review NVARCHAR(MAX))
RETURNS INT
AS
BEGIN
    RETURN LEN(@review);
END;
```

**Uso de la función**:

```sql
SELECT id_review, dbo.LongitudResena(review) AS Longitud
FROM reviews;
```

**Explicación**:

- La función `LongitudResena` recibe una reseña y devuelve su longitud.
- Aplicamos esta función a todas las reseñas para obtener sus longitudes.

#### 3.2. Función para Categorizar Restaurantes por Número de Reseñas

Clasifiquemos los restaurantes según su popularidad:

```sql
CREATE FUNCTION CategoriaRestaurantes(@n_reviews INT)
RETURNS NVARCHAR(50)
AS
BEGIN
    RETURN CASE
        WHEN @n_reviews > 1000 THEN 'Muy Popular'
        WHEN @n_reviews BETWEEN 500 AND 1000 THEN 'Popular'
        ELSE 'Menos Conocido'
    END;
END;
```

**Uso de la función**:

```sql
SELECT name, dbo.CategoriaRestaurantes(n_reviews) AS Categoria
FROM restaurants;
```

**Explicación**:

- La función `CategoriaRestaurantes` clasifica los restaurantes en "Muy Popular", "Popular" o "Menos Conocido" según el número de reseñas.
- Aplicamos esta función a la columna `n_reviews` de la tabla `restaurants`.

### 4. Automatización con Triggers

Los triggers nos ayudan a automatizar tareas que deben ejecutarse después de ciertas acciones.

#### 4.1. Trigger para Actualizar la Calificación Promedio

Asegurémonos de que la calificación promedio de cada restaurante esté siempre actualizada:

```sql
CREATE TRIGGER ActualizarCalificacionPromedio
ON reviews
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    UPDATE restaurants
    SET stars = (
        SELECT AVG(score)
        FROM reviews
        WHERE service = restaurants.id
    )
    WHERE id IN (SELECT DISTINCT service FROM inserted);
END;
```

**Explicación**:

- Este trigger se ejecuta después de insertar, actualizar o eliminar reseñas.
- Recalcula la calificación promedio (`stars`) para los restaurantes afectados.
- Usa `INSERTED` para identificar los restaurantes que necesitan actualización.

## Interpretación de Resultados

### Popularidad y Calificación

- **Restaurantes más comentados**: Aquellos con más reseñas son generalmente más populares.
- **Mejores calificados**: Las calificaciones promedio nos indican el nivel de satisfacción de los clientes.

### Análisis de Texto

- **Longitud de reseñas**: Las reseñas más largas pueden contener información más detallada y útil.
- **Palabras clave**: Analizando las reseñas, podemos identificar tendencias o aspectos comunes que gustan o disgustan a los clientes.

### Automatización

- **Actualización automática**: Con el trigger, aseguramos que las calificaciones promedio reflejen siempre los datos más recientes.
- **Eficiencia**: Automatizar procesos reduce errores y ahorra tiempo.

## Conclusión

En este tutorial, hemos explorado cómo utilizar SQL Server para realizar un análisis completo de datos de restaurantes y sus reseñas. Al aplicar consultas avanzadas, funciones y triggers, podemos extraer insights valiosos y mantener nuestros datos actualizados y consistentes.

¿Listo para llevar tus habilidades al siguiente nivel? ¡Manos a la obra! Pon en práctica lo aprendido y sigue explorando las capacidades de SQL Server para tus proyectos.
