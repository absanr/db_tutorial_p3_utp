
# Análisis de Reseñas de Restaurantes con SQL Server

## Introducción
¡Hola a todos! En este tutorial, exploraremos cómo utilizar SQL Server para analizar datos de restaurantes y sus reseñas. Este ejercicio integra conceptos clave como diseño de bases de datos, normalización, consultas avanzadas y funciones definidas por el usuario. Este caso práctico es relevante y representa un escenario que podrías encontrar en un entorno profesional.

## Objetivos
Nuestros objetivos para este tutorial son:

1. **Comprender los datos disponibles**: Identificar las relaciones entre las tablas `restaurants` y `reviews`.
2. **Aplicar consultas avanzadas**: Utilizar funciones y subconsultas para responder preguntas clave de negocio.
3. **Analizar datos para obtener insights valiosos**: Interpretar los resultados para tomar decisiones informadas.
4. **Entender cuándo y por qué utilizar ciertas características de SQL**: Explicar el uso apropiado de funciones y comandos.

## Descripción de los Datos

### Tabla `restaurants`
Contiene información básica sobre los restaurantes:

- `id`: Identificador único del restaurante.
- `name`: Nombre del restaurante.
- `tag`: Categorías del restaurante (tipo de comida y características).
- `x, y`: Coordenadas geográficas (latitud y longitud).
- `district`, `IDDIST`: Distrito y su identificador asociado.
- `direction`: Dirección completa del restaurante.
- `stars`: Calificación promedio del restaurante en estrellas.
- `n_reviews`: Total de reseñas del restaurante.
- `min_price`, `max_price`: Rango de precios del menú.
- `platform`: Plataforma de origen de los datos.

### Tabla `reviews`
Contiene información detallada de las reseñas:

- `id_review`: Identificador único de la reseña.
- `review`: Texto completo de la reseña.
- `title`: Resumen o título de la reseña.
- `score`: Calificación otorgada (1-5).
- `likes`: Número de "me gusta" que recibió la reseña.
- `id_nick`: Identificador del usuario que realizó la reseña.
- `service`: Identificador del restaurante al que se refiere la reseña.
- `date`: Fecha de la reseña.
- `platform`: Plataforma de origen de la reseña.

## Estructura Relacional
La relación entre las tablas es fundamental para nuestro análisis:

- **Relación 1:N**: Un restaurante puede tener muchas reseñas.
- Las reseñas están vinculadas a los restaurantes a través de `id` en `restaurants` y `service` en `reviews`.

## Consultas y Análisis

### 1. Exploración Inicial de Datos
#### 1.1. Visualizar Restaurantes Disponibles
```sql
SELECT * FROM restaurants;
```

#### 1.2. Ver las Primeras Reseñas
```sql
SELECT TOP 10 * FROM reviews;
```

### 2. Consultas Clave y Ejemplos Detallados

#### 2.1. Contar el Número de Reseñas por Restaurante
```sql
SELECT r.name AS Restaurante, COUNT(re.id_review) AS TotalReseñas
FROM restaurants r
LEFT JOIN reviews re ON r.id = re.service
GROUP BY r.name
ORDER BY TotalReseñas DESC;
```

#### 2.2. Calcular la Calificación Promedio por Restaurante
```sql
SELECT r.name AS Restaurante, AVG(re.score) AS CalificacionPromedio
FROM restaurants r
INNER JOIN reviews re ON r.id = re.service
GROUP BY r.name
HAVING COUNT(re.id_review) >= 5
ORDER BY CalificacionPromedio DESC;
```

#### 2.3. Identificar Restaurantes con Mayor Rango de Precios
```sql
SELECT name, min_price, max_price, (max_price - min_price) AS RangoPrecios
FROM restaurants
WHERE min_price IS NOT NULL AND max_price IS NOT NULL
ORDER BY RangoPrecios DESC;
```

#### 2.4. Encontrar Restaurantes en un Distrito Específico
```sql
SELECT name, direction, district
FROM restaurants
WHERE district = 'Centro';
```

### 3. Subconsultas y Funciones

#### 3.1. Obtener Restaurantes con Calificación por Encima del Promedio General
```sql
SELECT name, stars
FROM restaurants
WHERE stars > (SELECT AVG(stars) FROM restaurants);
```

#### 3.2. Función para Categorizar Restaurantes por Rango de Precios
```sql
CREATE FUNCTION dbo.CategoriaPrecio(@min_price DECIMAL(10,2), @max_price DECIMAL(10,2))
RETURNS NVARCHAR(20)
AS
BEGIN
    DECLARE @rango DECIMAL(10,2);
    SET @rango = @max_price - @min_price;
    RETURN CASE
        WHEN @rango >= 50 THEN 'Caros'
        WHEN @rango BETWEEN 20 AND 49.99 THEN 'Moderados'
        ELSE 'Económicos'
    END;
END;
```

Uso de la función:
```sql
SELECT name, dbo.CategoriaPrecio(min_price, max_price) AS CategoriaPrecio
FROM restaurants
WHERE min_price IS NOT NULL AND max_price IS NOT NULL;
```

#### 3.3. Contar Palabras en las Reseñas
```sql
SELECT id_review, 
       review,
       LEN(review) - LEN(REPLACE(review, ' ', '')) + 1 AS NumeroPalabras
FROM reviews;
```

### 4. Ejemplos Prácticos y Consideraciones

#### 4.1. Encontrar Reseñas Positivas y Negativas
```sql
SELECT id_review, score, review
FROM reviews
WHERE score = 5 OR score = 1;
```

#### 4.2. Uso de CASE sin Funciones Definidas por el Usuario
```sql
SELECT name,
       CASE
           WHEN n_reviews > 1000 THEN 'Muy Popular'
           WHEN n_reviews BETWEEN 500 AND 1000 THEN 'Popular'
           ELSE 'Menos Conocido'
       END AS Popularidad
FROM restaurants;
```

### 5. Diseño y Normalización

#### 5.1. Importancia de la Normalización
Ejemplo: Crear la tabla `districts` y modificar la tabla `restaurants`:
```sql
CREATE TABLE districts (
    IDDIST INT PRIMARY KEY,
    district_name NVARCHAR(100)
);

ALTER TABLE restaurants
DROP COLUMN district;

ALTER TABLE restaurants
ADD CONSTRAINT FK_Restaurants_Districts
FOREIGN KEY (IDDIST) REFERENCES districts(IDDIST);
```

#### 5.2. Desnormalización
Cuándo desnormalizar: Para optimizar consultas frecuentes que requieren uniones complejas.

## Interpretación de Resultados
- Restaurantes más comentados: Indican popularidad o alta afluencia.
- Mejores calificados: Reflejan satisfacción del cliente.
- Reseñas negativas: Señalan áreas de mejora.
- Reseñas positivas: Pueden destacarse en marketing.

## Conclusión
En este tutorial, aprendimos a utilizar SQL Server para realizar análisis avanzados de datos, aplicando:

- **Consultas avanzadas**: JOIN, GROUP BY, HAVING, subconsultas.
- **Funciones definidas por el usuario**: Para lógica modular.
- **Normalización**: Para garantizar la integridad de los datos.
