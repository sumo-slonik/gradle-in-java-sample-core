# Gradle w Javie – Szybki Tutorial

## Wprowadzenie
Gradle to nowoczesny system automatyzacji budowania projektów (build tool), który umożliwia kompilowanie, testowanie i pakowanie aplikacji w prosty i elastyczny sposób.  
Jest szczególnie popularny w ekosystemie Javy i Androida.  

### Dlaczego Gradle?
- Automatyzuje proces budowania projektu.
- Obsługuje zależności (biblioteki zewnętrzne) w prosty sposób.
- Jest szybki i elastyczny dzięki **incremental build**.
- Integruje się z IDE, jak IntelliJ IDEA czy Eclipse.

---

## Jak działa Gradle?
Gradle używa **skryptów konfiguracyjnych** (`build.gradle`) napisanych w Groovy lub Kotlin DSL.  

Podstawowe elementy:
- **Plugins** – np. `java` dla projektów w Javie.
- **Dependencies** – deklaracja bibliotek, z których korzysta projekt.
- **Tasks** – np. `build`, `test`, `run`.

Przykładowy plik `build.gradle`:

```groovy

// Sekcja 'plugins' pozwala na dodanie wtyczek, które rozszerzają funkcjonalność Gradle.
// Tutaj dodajemy wtyczkę 'java', aby Gradle wiedział, że projekt jest w Javie.
plugins {
    id 'java'
}

// 'group' i 'version' służą do identyfikacji Twojego projektu,
// przydatne np. przy publikowaniu artefaktów do repozytoriów.
group 'com.example'
version '1.0-SNAPSHOT'

// 'repositories' określa, skąd Gradle ma pobierać zależności (biblioteki zewnętrzne).
// Tutaj używamy Maven Central, popularnego repozytorium dla Javy.
repositories {
    mavenCentral()
}

// 'dependencies' definiuje wszystkie zewnętrzne biblioteki, których projekt potrzebuje.
// testImplementation oznacza, że zależność jest potrzebna tylko do testów.
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0' // JUnit 5 do testów jednostkowych
}

// Konfiguracja zadania testowego
// Tutaj mówimy Gradle, aby używał JUnit Platform (JUnit 5) do uruchamiania testów.
test {
    useJUnitPlatform()
}


```

### Rodzaje zależności w Gradle

| Typ zależności       | Zastosowanie | Przykład |
|---------------------|-------------|----------|
| `implementation`    | Standardowa zależność do kodu produkcyjnego, nie eksponowana na zewnątrz modułu | `implementation 'com.google.guava:guava:32.1.2-jre'` |
| `api`               | Zależność do kodu produkcyjnego, eksponowana na zewnątrz modułu | `api 'org.apache.commons:commons-lang3:3.13.0'` |
| `compileOnly`       | Potrzebna tylko w czasie kompilacji, nie dołączana do JAR | `compileOnly 'javax.servlet:javax.servlet-api:4.0.1'` |
| `runtimeOnly`       | Potrzebna tylko w czasie wykonywania aplikacji | `runtimeOnly 'mysql:mysql-connector-java:8.1.0'` |
| `testImplementation`| Zależność tylko do testów jednostkowych/integracyjnych | `testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'` |
| `testRuntimeOnly`   | Zależność tylko w czasie uruchamiania testów | `testRuntimeOnly 'org.junit.platform:junit-platform-launcher:1.10.0'` |
| `annotationProcessor`| Dla narzędzi generujących kod w czasie kompilacji (np. Lombok) | `annotationProcessor 'org.projectlombok:lombok:1.18.30'` |
| `compileOnly + annotationProcessor` | Lombok/MapStruct – kompilacja bez dodawania do JAR, generowanie kodu | `compileOnly 'org.projectlombok:lombok:1.18.30'`<br>`annotationProcessor 'org.projectlombok:lombok:1.18.30'` |


## Instalacja Gradle
# Instrukcja instalacji Java i Gradle

### 1. Sprawdź, czy masz Java zainstalowaną

Sprawdź wersję Javy:

```bash
java -version
```

Jeśli Java nie jest zainstalowana, zainstaluj ją:

- **Windows / macOS / Linux** – pobierz z [Adoptium](https://adoptium.net/) lub [Oracle JDK](https://www.oracle.com/java/technologies/javase-downloads.html)
- **Linux (Debian/Ubuntu)**:

```bash
sudo apt update
sudo apt install openjdk-17-jdk
```

---

### 2. Zainstaluj Gradle

Oficjalna instrukcja: [https://gradle.org/install/](https://gradle.org/install/)

Sprawdź instalację:

```bash
gradle -v
```

## Tworzenie projektu Java z Gradle

### Krok 1: Inicjalizacja projektu
1. Otwórz terminal i utwórz nowy katalog projektu:
```
mkdir MyGradleApp
cd MyGradleApp
```
2. Zainicjalizuj projekt Gradle typu `application`:
```
gradle init
```
- Wybierz typ projektu: **application**  
- Wybierz język: **Java**
- Application structure: **Single application project**
- Pozostałe opcje możesz zostawić domyślnie.

### Krok 2: Struktura projektu
Po inicjalizacji Gradle utworzy strukturę projektu:

```
MyGradleApp/
 ├─ build.gradle
 ├─ settings.gradle
 ├─ app/src/
 │   ├─ main/java/App.java
 │   └─ test/java/AppTest.java
 └─ gradle/
     └─ wrapper/
```

### Krok 3: Budowanie projektu
Aby skompilować projekt i uruchomić testy:
```
gradle build
```
- Gradle pobiera zależności i tworzy folder `build/` z wynikami kompilacji.  
- Testy jednostkowe zostaną uruchomione automatycznie.

### Krok 4: Uruchamianie aplikacji
Jeśli projekt jest typu `application`, możesz uruchomić aplikację:
```
gradle run
```
- Zobaczysz w konsoli wynik działania metody `main()` w `App.java`.

### Krok 5: Uruchamianie testów
Aby uruchomić tylko testy jednostkowe:
```
gradle test
```
- Wyniki testów znajdziesz w folderze `build/reports/tests/test/index.html`.

### Gratulacje!
Właśnie stworzyłeś prosty projekt Java z Gradle, zbudowałeś go, uruchomiłeś i przetestowałeś.  
Możesz teraz eksperymentować, dodając własne klasy i testy.

Jeśli chciałbyś, aby GitHub sam sprawdzał, czy Twój projekt buduje się poprawnie, a testy przechodzą, możesz do tego wykorzystać plik, który znajduje się w tym repozytorium w folderze `.github/workflows/check_run.yml` wystarczy że umieścisz go w głównym katalogu swojego proejktu ( z zachowaniem hierarchi folderów .github/workflows).  

Jeśli zainteresował Cię temat GitHub Actions, zapraszam do tego [repozytorium](https://github.com/sumo-slonik/git-hub-actiosns-sample-core)


