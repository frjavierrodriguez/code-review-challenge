# Review

En este fichero se incluye los anotaciones de la code review realizada al código del repositorio.
autor: **frjavierrodriguez@gmail.com**

## 1.- Review general del código: 
El objeto de esta primera iteración es revisar el estado actual del código para comprobar 
si se está realizando un uso apropiado de tipos, convención de nombre, posibles NullPointerExceptions

### com.idealista.application

1. **AdsService**:
    * Convención de nombre. Usar AdService en lugar AdsService
    * Interfaz donde se definen las operaciones relacionadas con el negocio. No debería conocer de la existencia de las clases **PublicAd** & **QualityAd**.

2. **AdsServiceImpl**:
   * Convención de nombre. Usar AdServiceImpl en lugar AdsServiceImpl
   * AdsServiceImpl#calculateScore(Ad ad) -> Compejidad ciclomática elevada. Refactorizar y desglosar en operaciones más sencillas
   * AdsServiceImpl#calculateScore(Ad ad) -> Método demasiado largo. Refactorizar
   * AdsServiceImpl#calculateScore(Ad ad) -> Controlar que el parámetro de entrada es no nulo. En dicho caso, devolver 0.
   * AdsServiceImpl#calculateScore(Ad ad)::L30, L49 y L71  -> Posible NullPointerException, si el listado de pictures es nulo en las instancia de AD tratada.
   * AdsServiceImpl#calculateScore(Ad ad)::L31, L50 -> Posible NullPointerException, si el valor del atributo Typology es nulo en las instancia de AD tratada.
   * AdsServiceImpl#calculateScore(Ad ad)::L94 - 108 -> Usar switch or if ... else. En lugar de 2 estructuras If para determinar si estamos tratando un anuncio de un piso o chalet.
   * AdsServiceImpl#calculateScore(Ad ad)::L110 - 114 -> Usar constantes para los literales.
   * Se está usando Java8, por lo que se recomienda el uso de la nueva API Time en lugar de usar java.util.Date.
   * Se recomienda el uso de java.time.Instant para este caso.
   * AdsServiceImpl#findPublicAds() -> Se puede simplificar con el uso de Streams:
     ```ads.stream().sorted(Comparator.comparingInt(Ad::getScore)).map(ad -> {...}).collect(Collectors.toList())```
   * AdsServiceImpl#findQualityAds() -> Se puede simplificar con el uso de Streams:
     ```ads.stream().map(ad -> {...}).collect(Collectors.toList())```

### com.idealista.domain

1. Ad & Picture
  * Uso de clase Integer en lugar de primitivo: Generalmente, se recomienda el uso de tipos primitivos a menos que necesite un objeto por alguna razón (por ejemplo, para poner en una colección, genéricos).
  * Se está usando Java8, por lo que se recomienda el uso de la nueva API Time en lugar de usar java.util.Date. Se recomienda el uso de java.time.Instant para este caso.
  * Implementar patrón Builder como alternativa para constructor con demasiados parámetros. 

2. Constants
  * Añadir constructor privado a la clase para evitar que se puedan crear instancias

3. Quality & Typology
  * Convención de código. Usar ';' para finalizar la lista de valores del enumerado. En lugar de usar ','

4. AdRepository
  * Remover la cadena "Ad" del nombre de los métodos en la interfaz. Es redundante.
          List<Ad> findAll();
          void save(Ad ad);
          List<Ad> findRelevant();
          List<Ad> findIrrelevant();

  * Voy a suponer que el número de anuncios a manejar es elevado. Por este motivo, pienso que las operaciones findAllAds,
    findRelevantAds & findIrrelevantAds deberían de ofrecer mecanismos que nos permitan recuperar la información de forma
    fragmentada. Por ejemplo usando técnicas para paginar los resultados:

          Page<Ad> findAll(final PageRequest pageable)

              - PageRequest -> Clase que permite indicar el nº de página a recuperar así como el nº de elementos por páginas.
              - Page<T> -> Clase que contiene la lista de elementos que compone la página solicitadas e incluye informacioón como
                     Nº de página, tamaño de página, nº total de elementos.

### com.idealista.infrastructure.persistence

1. AdVO & PictureVO
  * Implementar métodos equals & hashCode.
  * Uso de clase Integer: Generalmente, debe usar tipos primitivos a menos que necesite un objeto por alguna razón (por ejemplo, para poner en una colección, genéricos).
  * Se está usando Java8, por lo que se recomienda el uso de la nueva API Time en lugar de usar java.util.Date.
    Se recomienda el uso de java.time.Instant para este caso.
  * Implementar patrón Builder como alternativa para consytructor con demasiados parámetros. 

2. InMemoryPersistence:
   - En lugar de usar java.util.List como colección de objetos. Se recomendaría el uso de java.util.Map, ya que mejora el rendimiento
     para acceder a objetos especificos por la clave de 0(n) a 0(1). Por lo que se mejora el rendimiento para la operación **save**.
     Así como las operaciones encargadas de cargar las imágenes.

## 2.- Testing
Incluir casos de pruebas que permitan dar cobertura a la mayor parte del código implementado:
Por ejemplo, para la implementación del servicio AdsServiceImpl:
- Casos de pruebas método findPublicAds:
  - testFindPublicAds_Empty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista vacía de anuncios relevantes.
  - testFindPublicAds_NoEmpty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista con N anuncios relevantes.
  - testFindPublicAds_MapRule_XX -> Caso de prueba que se encarga de comprobar la lógica definida para transformar de Ad a PublicAd.
    - ¿Contiene el nuevo objeto PublicAd los valores esperados para la instancia de Ad procesada?
    - ¿Como se comporta cuando la lista de imágenes es vacía, nula o tiene algún elemento?
  - testFindPublicAds_Sort_XX -> Tests que se encargan de comprobar el orden de los elementos en el listado.
- Casos de pruebas método findQualityAds:
  - testFindQualityAds_Empty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista vacía de anuncios irrelevantes.
  - testFindQualityAds_NoEmpty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista con N anuncios irrelevantes.
  - testFindQualityAds_MapRule_XX -> Caso de prueba que se encarga de comprobar la lógica definida para transformar de Ad a QualityAd.
    - ¿Contiene el nuevo objeto QualityAd los valores esperados para la instancia de Ad procesada?
    - ¿Como se comporta cuando la lista de imágenes es vacía, nula o tiene algún elemento?
- Casos de pruebas método calculateScores:
  - testCalculateScores_Empty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista vacía.
  - testCalculateScores_NoEmpty -> Caso de prueba que se encarga de comprobar la operación cuando el repositorio devuelve una lista con N elementos.
  - testCalculateScores_Errors -> Como se debe comportar el sistema en caso de error cuando no son procesados todos los elementos del listado (No es atómico).
  - testCalculateScores_onlyOne_XX -> Dar cobertura completa a la lógica encargada para el cálculo de puntos:
    - Anuncio 0 -> Probar las distintas rutas donde los puntos calculados puede ser 0.
      - Escenario 1: Anuncio sin fotos (-10), con descripción  menos de 20 palabras y no contiene palabras claves (+5) y no completo (0) = 0
      - Escenario 2: Anuncio sin fotos (-10), sin descripción (0) y no completo (0) = 0
      - Escenario 3: Anuncio sin fotos (-10), con descripción  menos de 20 palabras que contiene una de las palabras claves (+10) y no completo (0) = 0
    - Anuncio 10 -> Probar las distintas rutas donde los puntos calculados puede ser 0.
      - Escenario 3: Anuncio con 1 foto SD o calidad nula (+10), sin descripción (0) y no completo (0) = 10
      - Escenario 3: Anuncio sin fotos (-10), con descripción  menos de 20 palabras que contiene 4 de las palabras claves (+20) y no completo (0) = 10
      ...
- Además de debería dar cobertura a la implementación InMemoryPersistence.java así como a AdsController.java.

## Clean code 
En lineas generales se observa violación de algunos de los principios SOLID. A continuación se citan algunos ejemplos:
1. Responsabilidad Única (Single responsibility): "Una clase debe tener una única razón para cambiar."

- *com.idealista.infrastructure.persistence.InMemorypersistence*: Se puede observar en la implementación de esta clase que 
  existen varios agentes que pueden requerir modificaciones sobre la misma. Es responsable de la lógica para la persistencia
  de los anuncios, imágenes e incluye las reglas para el mapeo de VO's al modelo.
- *com.idealista.application.AdsService/AdsServiceImpl*: Servicio responsable de la gestón de la clase del modelo Ad y además
  se encarga de las puntuaciones e incluyen las reglas para el mapeo de las clases del model a DTO's.

El primer paso para resolver esta violación es identificar las responsabilidades y separarlas. Por ejemplo en el caso de **AdServiceImpl** se identifican 3:
- Búsqueda de anuncios irrelevantes -> Cambios en los criterios de búsqueda para identificar anuncios irrelevantes y en como presentarse, implicarán cambios sobre AdServiceImpl
- Busqueda de anuncios relevantes -> Cambios en los criterios de búsqueda para identificar anuncios relevantes y en como presentarse, implicarán cambios sobre AdServiceImpl
- Cálculo de puntuaciones -> Cambios en los criterios para aplicar la puntuación implicarán cambios sobre este servicio.

Una posible solución para este principio podría ser separar cada uno de los motivos de cambio en una clases especificas para dichas responsabilidades. 


2. Principio Abierto y cerrado: Una clase debe estar abierta a la extensión y cerrada a la modificación
- *com.idealista.application.AdsService*: Por ejemplo, estamos incumpliendo este principio, dado que cada nuevo filtro que tengamos que incluir a
  futuro para los anuncios, implicarán modificaciones sobre esta interfaz y su implementación. Por otra parte cambios en la lógica responsable de calcular la 
  puntuación implicará cambios sobre este servicio.
- *com.idealista.domain.AdRepository*: Estamos incumpliendo este principio, dado que cada nuevo filtro que tengamos que incluir a futuro para los anuncios, implicarán modificaciones sobre esta
  interfaz y sus implementaciones.

Para respetar este principio debemos centrarnos en la abstracción. Tomando como ejemplo las operaciones de filtrado para AdsServiceImpl, cada nueva operación de filtro o cambio en los criterios
requerirá de modificaciones en esta clase. Una posible solución podría ser la creación de un conjunto de clases que nos permitan definir filtros para los anuncios.
Por ejemplo: 
```
public class Filter<X> {
  private X equal;
  ...
}

public class IntervalFilter<X> extends Filter<X> {
    private X greater;
    private X less;
    private X greaterOrEqual;
    private X lessOrEqual;
    ...
}

public class AdCriteria implement Criteria {
    private IntervalFilter<Integer> score;
}

public interface AdService {
  List<Ad> find(final Criteria criteria);
}

```

Como se puede observar, con la solución propuesta se pueden aplicar difirentes criterios de búsqueda sin necesidad de modificar la lógica del servicio AdServiceImpl.
Es decir, esta solución respeta el principio abierto/cerrado.

## Gestión de errores y Logging
- No se aprecia ningún mecanismos para la gestón de errores, así como no se incluye mecanismos para trazar que ocurre en el sistema.

## Arquitectura

La solución se encuentra implementada usando una aproximación de arquitectura hexagonal. Sin embargo no se encuentra correctamente implementada, ya que se observa algunos errores
donde la lógica de negocio no se encuentra aislada correctamente. Por ejemplo el uso de las clases PublicAd y QualityAd en el servicio AdsService/AdsServiceImpl.

Se propone la siguiente reestructuración:
- com.idealista.ads
  - domain
    - repository
      - AdRepository.java
    - service
      - AdService.java
      - impl
        - AdServiceImpl.java
    - model
      - Ad.java
      - Picture.java
      - Quality.java
      - Typology.java
    - Exception
      - AdExceptionApp.java
    - AdConstant.java
  - config
    - AdApplication.java
    - AddRepositoryConfig.java
    - AddServiceConfig.java
  - adapter
    - in
      - api
        - dto
          - AdDTO.java
          - PictureDTO.java
        - mapper
          - AdDTOMapper.java
          - PictureDTOMapper.java
        - AdRestController.java
    - out
      - persistence
        - vo
          - AdVO
          - PictureVO
        - mapper
          - AdVOMapper.java
          - PictureVOMapper.java
        - InMemoryPersistence

- Domain: Continene la lógica de negocio del sistema. Debe mantenerse aislado de frameworks y de adaptadores. Es decir,
  no anotaciones Spring y ningún uso de clases que se encuentren definidas en los adaptadores.
- Adapter: Contiene la implementacion de los distintos puertos de nuestro sistema. Estos serán clasificados como entradas o salidas:
  - api: Implementación del API Rest.
    - mapper: paquete que contiene las reglas de transformación de una clase del modelo a su correspondiente DTO.
  - persistence: Implementación de lógica para la persistencia de datos.
    - mapper: paquete que contiene las reglas de transformación de una clase del modelo a su correspondiente VO. 
- config:
  - Código responsable de la preparación de los Beans gestionados por Spring.

## Posibles mejoras

Se recomienda el uso de las siguientes utilidades ampliamente reconocidas en proyectos basados en Java:
- [Lombok](https://projectlombok.org/):  Librería Java cuyo objetivo es facilitarnos el desarrollo de nuestro código evitándonos tener que escribir ciertos métodos, repetitivos y que no aportan lógica al negocio: getter, setter, equals, hashcode, constructores...
- [Apache Commons](https://commons.apache.org/): Conjunto de librerías que proporcionan bastantes utilidades para el desarrollo de aplicaciones basadas en Java 
- [Mapstruct](https://mapstruct.org/): Librería java que permite generar las clases encargadas de mapeos entre objetos en java a partir de las interfaces.






