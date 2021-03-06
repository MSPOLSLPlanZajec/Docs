# Plan zajęć

## Wymagania systemu

### Wymagania sprzętowe

* Procesor: 2 GHz 2 rdzenie lub więcej
* Pamięć RAM: 2 GB lub więcej
* Windows Server 2008 lub wyżej
* Połączenie z siecią internet

### Wymagania funkcjonalne

#### Użytkownicy

* Uczniowie/Nauczyciele
* Administrator

#### Cel projektu

Umożliwienie uczniom i nauczycielom (dalej UP) przeglądania planów dla odpowiednich grup, sal i nauczycieli. Mogą oni wyszukać plany zajęć a także prowadzących przy pomocy odpowiednich pól do wpisywania bądź z listy rozwijalnej.
Administrator ma dostęp do panelu administracyjnego po zalogowaniu się.
Administrator ma możliwość tworzenia, edycji i usuwania przedmiotów, planów zajęć, sal, grup oraz prowadzących. 
Administrator ma możliwość ustawiania planu poprzez przeciąganie odpowiednich przedmiotów na plan metodą Drag-and-Drop.
Podczas edycji planu administrator widzi wyróżnione luki w planie - gdzie dla danego przedmiotu dostępne są: sala, grupa oraz prowadzący - podczas przeciągania przedmiotu lub po jego kliknięciu.
Dostęp do każdej z funkcjonalności ma być dostępny przez przeglądarkę internetową.

#### Funkcjonalności programu (P - podstawowa, D - dodatkowa):

Każdy (UP i Administrator):

* (P) Przeglądanie planu dla danej grupy
* (P) Podział grup na podgrupy
* (P) Przeglądanie planu prowadzącego
* (P) Przeglądanie planu dla sali
* (D) Przeglądanie planu danego studenta
* (D) Możliwość zalogowania się studenta i przejrzenie jego planu

Administrator:

* (P) Układanie planu dla danej grupy
* (P) Defniowanie przedmiotu - czas trwania zajęć, prowadzący (także wielu)
* (P) Przydział zajęć do grupy
* (P) Definiowanie sal - pojemność, wydział
* (D) Układanie planu dla prowadzącego
* (D) Opcja wybrania wydziałów do poszukiwania sal przy przeszukiwaniu ich dostępności

## Opis systemu

### Front - end

Część aplikacji działająca po stronie klienta jest wykonana w technologii **SPA** (Single Page Application). Aplikacja pobiera niezbędne dane z odpowiednich endpoint'ów **API** typu **REST**.

Część kliencka jest wykonana w oparciu o bibliotekę **AngularJS 1**, z routingiem obsługiwanym przez **ui-router**, a komunikacja z API poprzez **ng-resource**. Całość jest napisana w standardzie **ECMAScript 7** języka **JavaScript** i budowana przy użyciu **Webpack**.

Zdecydowaliśmy się skorzystać z **AngularJS 1**, gdyż jest to sprawdzony framework do tworzenia aplikacji front-end'owych, pozwalający na łatwą kontrolę nad przepływem danych. W porównaniu ze starszymi metodami tworzenia aplikacji internetowych (np. **jQuery**), **AngularJS 1** pozwala na łatwiejsze debug'owanie kodu i wprowadza wygodną ideę **two-way data binding**, wg. której zmiana w modelu, automatycznie aktualizuje widok i vice versa.

Możliwy był wybór między **AngularJS 1**, **AngularJS 2** i **ReactJS**, jednak w chwili obecnej w pełni działający pakiet **Angular Material** (zbiór komponentów UI spełniający wytyczne Material Design) jest dostępny tylko dla **AngularJS 1**.

Główny plik aplikacji, w którym, korzystając z funkcjonalności **webpack'a**, budowane są komponenty i usługi oraz konfiguracja aplikacji:

```javascript
import app from 'appModule';
import shared from 'shared';
import components from 'components';
import buildComponents from 'utils/buildComponents';
import buildShared from 'utils/buildShared';

import mockPromise from 'utils/mockPromise';
import mocks from 'mocks';

import routes from 'appRoutes';
import defaults from 'appDefaults';
import run from 'appRun';
import config from 'appConfig';

import styles from '../assets/styles/style.less'

window.App = app;

buildComponents(components);
buildShared(shared);

app.defaults = defaults; // global settings
app.config(config);

app.config(routes);

app.mockPromise = mockPromise;
app.mocks = mocks;

app.run(run);
```

Aplikacja składa się z komponentów, które są wyświetlane w ramach routingu na stronie. Za konfigurację routingu odpowiedzialny jest plik **appRoutes.js**:

```javascript
import components from 'components';

module.exports = function ($stateProvider, $locationProvider, $urlRouterProvider) {
    $locationProvider.html5Mode(true);

    $urlRouterProvider.otherwise('/');

    $stateProvider
        .state(angular.extend({ name: 'home', url: '/' }, components.home))
        .state(angular.extend({ name: 'schedule', url: '/schedule/:id/:type' }, components.schedule))
        .state(angular.extend({ name: 'dataEditor', url: '/data-editor' }, components.dataEditor));
}
```
## Opis części back-endowej

### Wybrana technologia

Baza danych zarządzana jest systemem **MySQL** firmy **Oracle**.

Wybraną przez nas technologią dla tej części aplikacji jest **.NET** z użyciem frameworka sieciowego **Web API 2** oraz konektora **MySQL** dla tejże technologii.
Wybraliśmy tą technologię ponieważ umożliwia ona proste tworzenie sieciowego api opartego o komunikację **REST**-ową.

### Informacje użytkowe o wybranej technologii

Wybrany przez nas framework umożliwia nam szybkie tworzenie **REST**-owych endpointów do komunikacji z częścią front-endową. 
Proces tworzenia składa się z dwóch etapów
* Konfiguracji routingu
* Tworzenia controllerów

#### Konfiguracja routingu

Konfiguracja ta opiera się na ustawieniu odpowiednich scieżek dostępowych do naszej konfiguracji end-pointów.
Proces ten przeprowadza się w pliku [WebApiConfig.cs](https://github.com/MSPOLSLPlanZajec/Back-end/blob/master/TimetableServer/App_Start/WebApiConfig.cs)

```csharp
config.Routes.MapHttpRoute(
    name: "Schedules",
    routeTemplate: "{controller}/{id}/{typeOfSchedule}",
    defaults: new {id = "", typeOfSchedule = ""});
config.Routes.MapHttpRoute(
    name: "TimeSuggestion",
    routeTemplate: "{controller}/{id}",
    defaults: new { });
config.Routes.MapHttpRoute(
    name: "GroupsList",
    routeTemplate: "{controller}",
    defaults: new {});
```

Tworzymy trzy rodzaje ścieżek dostępowych:
* {controller}/{id}/{typeOfSchedule}
* {controller}/{id}
* {controller}

{controller} jest pewnego rodzaju "zastępstwem" dzięki któremu mimo posiadania wielu contollerów pod tym samym routingiem aplikacja sama dopasuje odpowiedni end-point dzięki nazwie podanego controllera. Pola {id} i {typeOfSchedule} są już nazwami parametrów controllerów. Muszą one być identyczne jak nazwy parametrów. Dlatego kontrolerowi z metodą **GET** w formie:
```csharp
public BaseSchedule GetSchedule(string id, string typeOfSchedule){...}
```
będzie odpowiadać routing {controller}/{id}/{typeOfSchedule}.

Dodatkowo ustaliliśmy sposób komunikacji przy użyciu **JSON**'a, dlatego należało ustawić 
```csharp
config.Formatters.JsonFormatter.SupportedMediaTypes.Add(new MediaTypeHeaderValue("text/html"));
```
w konfiguracji **WebApi**.

#### Tworzenia controllerów

Controllery są obiektami odpowiadającymi za obsługę żądań wysyłanych na end-pointy. Dziedziczą one po klasie ApiController. Umożliwia im to odpowiadanie na zapytania **GET**, **POST**, **UPDATE** i **DELETE**. Dzięki **WebAPI 2** każdy end-point automatycznie konwertuje odpowiedź na formę **JSON**-ową. Tak samo dzieje się z bardziej złożonymi klasami jako argumenty. Jedyny problem napotkaliśmy przy end-point'cie /command. Aby obsługiwać wiele rodzajów zapytań na nim postanowiliśmy tworzyć odpowiednią klasę na podstawie parametru type a treść zapytania przekazywać jako zwykłego stringa. Jednakże dzięki wbudowanym parserom **JSON** bez większych problemów byliśmy w stanie przekształcać dane **JSON**'owe w obiekty i vice versa. 

#### Komunikacja między aplikacjami

Aby umożliwić komunikację między obiema częściami systemu użyliśmy wbudowany w ASP.Net moduł zarządzający mechanizmem CORS. Umożliwia on aplikacjom korzystającym z zasobów umieszczonych na różnych serwerach internetowych. Bez tego mechanizmu, korzystanie z danych zasobów między różnymi aplikacjami byłoby niemożliwe.

### Baza danych

#### Diagram związków encji

![Diagram związków encji](ERD.png)

#### Model relacyjny

![Diagram związków encji](diagram.png)

#### Opis
Zaprojektowana baza danych składa się z 12 tabel:
* tabela **subjects** - przechowuje informacje o przedmiotach
* tabela **teaching** - przechowuje informacje o tym jakich przedmiotów może uczyć dany nauczyciel 
* tabela **titles** - przechowuje informacje o tytułach i stopniach naukowych
* tabela **teachers** - przechowuje informacje o prowadzących zajęcia
* tabela **days** - przechowuje informacje o dniach tygodnia
* tabela **lessons** - przechowuje informacje o zajęciach
* tabela **classrooms** - przechowuje informacje o salach, w których są prowadzone zajęcia
* tabela **classroom type** - przechowuje informacje o typach sal, w których są prowadzone zajęcia
* tabela **department** - przechowuje informacje o tym, na jakim wydziale znajduje się dana sala
* tabela **faculty** - przechowuje informacje o kierunku na jakim jest dana grupa
* tabela **semesters** - przechowuje informacje o semestrach
* tabela **groups** - przechowuje informacje o grupach studenckich

## Opis odpytywanych danych

### 1. Prezentacja danych

- lista grup

    GET params: ()

    endpoint path: /group
    
    response:
    ```javascript
    [{
        name: 'Nazwa grupy',
        id: 'UUID',
        groups: [
            {
                name: 'Nazwa grupy',
                id: 'UUID',
                groups: [
                    
                ]
            }
        ]
    }]
    ```

- plan:

    GET params: (id: 'string', type: [ 'teacher' || 'group'])

    endpoint path = /schedule/:id/:type

    response: 
    ```javascript
    {
        name: 'Nazwa grupy || nauczyciela'
        schedule: [
            [{
                name: 'Nazwa przedmiotu',
                teacher: {
                    name,
                    id
                }
                classroom: {
                    name,
                    id
                },
                group: {
                    name,
                    id
                }
                type: [ 'lecture' || 'laboratories' || 'excercise' ],
                duration,
                startsAt
            }], // poniedziałek
            [] // dzień bez zajęć
        ],
        notScheduled: [
            {
                name,
                teacher: {
                    name,
                    id
                },
                type: [ 'lecture' || 'laboratories' || 'excercise' ],
                duration
            }
        ]
    }
    ```


### 2. Edytor
- komendy

    enpoint path: /command

    POST params: 

    ```javascript
    {
        type: [ 'add_study_plan' || 'add_classroom' || 'add_teacher' || 'select_start' ]
        data: {

        }
    }
    ```

    - type: 'add_study_plan':

      1. dodajemy kierunek 
      2. dodajemy semestr
      3. dodajemy grupę
      4. dodajemy przedmioty (czas trwania i typ) - wybór prowadzącego

      POST
      ```javascript
      data: {
          major: 'Nazwa kierunku',
          semesters: [
              subjects: [
                  {
                      name,
                      teacher_id,
                      duration : int (ilość bloków - blok trwa 15 min)
                      type: [ 'lecture' || 'laboratories' || 'excercise' ]
                  }
              ]
              subgroups: [
              {
		          name: '',
            	  subjects: [
                	  {
                          name,
                  	      teacher_id,
                	      duration : int (ilość bloków - blok trwa 15 min)
                          type: [ 'lecture' || 'laboratories' || 'excercise' ]  
                	  }]
		           subgroups: [{}] //And so on...
               }
          ]
      }
      ```
      response:
      ```javascript
      {
          id: "",
          major: 'Nazwa kierunku',
          semesters: [
              id: "",
              subjects: [
                  {
                      name,
                      teacher_id,
                      duration : int (ilość bloków - blok trwa 15 min)
                      type: [ 'lecture' || 'laboratories' || 'excercise' ]
                  }
              ]
              subgroups: [
              {
                  id: "",
		          name: '',
            	  subjects: [
                	  {
                          name,
                  	      teacher_id,
                	      duration : int (ilość bloków - blok trwa 15 min)
                          type: [ 'lecture' || 'laboratories' || 'excercise' ]  
                	  }]
		           subgroups: [{}] //And so on...
               }
          ]
      }
      ```
          
    - type: 'add_classroom'
    
      POST
      ```javascript
      data: {
          number
      }
      ```
      response:
      ```javascript
      {
          name: '',
          id: '',
      }          
      ```
      
    - type: 'add_teacher'

      POST
      ```javascript
      data: {
          name,
      surname,
          title // Wybrany z listy (endpoint /degree)
      }
      ```
      response:
      ```javascript
      {
          id: '',
          name: '',
          surname: '',
          title: ''
      }          
      ```

    - type: 'select_start'

      POST
      ```javascript
      data: {
          id, // id zajęć
          day : int (0 - 6)
          startsAt,
          room
      }
      ```
      response:
      ```javascript
      {
          id, // id zajęć
          day : int (0 - 6)
          startsAt
      }
      ```


- nauczyciele
    endpoint path: /teacher 
    
    GET

    response: 
    ```javascript
    [
        {
            id: '',
            name: '',
            surname: '',
            title: ''                
        }
    ]
    ```

- stopnie naukowe
    endpoint path: /degree 

    GET

    response:
    ```javascript
    [
        'dr',
        'dr hab.'
    ]
    ```

3. Układanie planu
    
    zapytanie do endpoint'u plan
    /time-suggestion/:classes_id

    GET

    response: 
    ```javascript
    [
        [ // poniedziałek
            {
                startsAt,
                duration,
                rooms: []
            }
        ],
        [ // wtorek

        ]
    ]
    ```

## Podział pracy
### Front-end
* Tomasz Pierzchała - Komponent przydzielania zajęć i wyświetlanie planów
* Wilhelm Olejnik - Edytor nauczycieli, klas i planów
### Back-end
* Kamil Rutkowski - Kontorlery: TimeSugestions, Groups, LessonsTypes, Degrees;
* Michał Wolszleger - Kontrolery: Command, Classroom, Teacher, Schedule; Obsługa logowania przy pomocy tokenu, obsługa CORS
### Baza danych
* Jakub Rup - Zarządznie bazą danych oraz połączeniem z nią przy pomocy Entity Framework.
