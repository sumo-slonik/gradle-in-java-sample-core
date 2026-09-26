# Gradle w Javie – szybki tutorial

## Wprowadzenie

Gradle to system automatyzacji budowania projektów (*build tool*). Pozwala kompilować, testować, uruchamiać
i pakować aplikację jednym poleceniem, niezależnie od IDE. Jest szczególnie popularny w ekosystemie Javy i Androida.

Na laboratoriach z Programowania Obiektowego projekt `oolab` jest projektem Gradle - IntelliJ korzysta z Gradle
do budowania i uruchamiania testów, a GitHub Actions (CI) buduje projekt tym samym Gradle'em.

### Dlaczego Gradle?

- Automatyzuje budowanie projektu - to samo polecenie działa u Ciebie, u prowadzącego i na serwerze CI.
- Zarządza zależnościami - wystarczy wpisać nazwę biblioteki, a Gradle sam ją pobierze.
- Jest szybki dzięki **incremental build** (buduje tylko to, co się zmieniło) i cache'owi.
- Integruje się z IDE, np. IntelliJ IDEA czy Eclipse.

---

## Jak działa Gradle?

Gradle używa **skryptów konfiguracyjnych** pisanych w jednym z dwóch języków (tzw. DSL):

- **Groovy DSL** - pliki `build.gradle`, `settings.gradle` (tego używamy na laboratoriach z PO),
- **Kotlin DSL** - pliki `build.gradle.kts`, `settings.gradle.kts` (domyślny wybór w `gradle init`;
  w nim jest przykładowy projekt w tym repozytorium).

Oba robią dokładnie to samo, różni się tylko składnia.

Podstawowe elementy:

- **Plugins** - rozszerzają Gradle, np. `java` (kompilacja Javy) albo `application` (uruchamianie programu przez `run`).
- **Repositories** - skąd pobierać biblioteki (najczęściej Maven Central).
- **Dependencies** - biblioteki, z których korzysta projekt.
- **Tasks** - zadania do wykonania, np. `build`, `test`, `run`, `clean`.

Przykładowy plik `build.gradle` (Groovy DSL) - taki, jaki powstaje na laboratoriach z PO:

```groovy
// Sekcja 'plugins' dodaje wtyczki, które rozszerzają możliwości Gradle.
// 'java' - kompilacja kodu w Javie, 'application' - uruchamianie programu taskiem 'run'.
plugins {
    id 'application'
    id 'java'
}

// 'group' i 'version' identyfikują projekt (przydatne np. przy publikowaniu bibliotek).
group = 'org.example'
version = '1.0-SNAPSHOT'

// Klasa z metodą main, uruchamiana przez './gradlew run'.
application {
    getMainClass().set('agh.ics.oop.World')
}

// Wersja Javy, którą Gradle kompiluje projekt (toolchain).
java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(25))
    }
}

// Skąd Gradle ma pobierać zależności - Maven Central to główne repozytorium bibliotek Javy.
repositories {
    mavenCentral()
}

dependencies {
    // BOM ("bill of materials") ustala spójne wersje wszystkich modułów JUnit,
    // dlatego przy kolejnych zależnościach JUnit nie podajemy już wersji.
    testImplementation platform('org.junit:junit-bom:5.10.0')
    // JUnit 5 (Jupiter) - biblioteka do testów jednostkowych
    testImplementation 'org.junit.jupiter:junit-jupiter'
    // Launcher uruchamiający testy - w Gradle 9 trzeba go dodać jawnie
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

// Konfiguracja taska 'test': testy uruchamiamy przez JUnit Platform (JUnit 5).
test {
    useJUnitPlatform()
}
```

> **Uwaga:** w Gradle 9 linia `testRuntimeOnly 'org.junit.platform:junit-platform-launcher'` jest obowiązkowa -
> bez niej testy nie uruchomią się (błąd `Failed to load JUnit Platform`).

### Rodzaje zależności w Gradle

| Typ zależności       | Zastosowanie | Przykład |
|---------------------|-------------|----------|
| `implementation`    | Standardowa zależność kodu produkcyjnego, niewidoczna dla modułów korzystających z naszego | `implementation 'com.google.guava:guava:33.4.6-jre'` |
| `api`               | Zależność kodu produkcyjnego widoczna także dla modułów korzystających z naszego (wymaga pluginu `java-library`) | `api 'org.apache.commons:commons-lang3:3.17.0'` |
| `compileOnly`       | Potrzebna tylko w czasie kompilacji, nie trafia do zbudowanej aplikacji | `compileOnly 'jakarta.servlet:jakarta.servlet-api:6.1.0'` |
| `runtimeOnly`       | Potrzebna tylko w czasie działania aplikacji | `runtimeOnly 'com.mysql:mysql-connector-j:9.3.0'` |
| `testImplementation`| Zależność tylko do kompilowania i uruchamiania testów | `testImplementation 'org.junit.jupiter:junit-jupiter'` |
| `testRuntimeOnly`   | Zależność potrzebna tylko w czasie uruchamiania testów | `testRuntimeOnly 'org.junit.platform:junit-platform-launcher'` |
| `annotationProcessor`| Narzędzia generujące kod w czasie kompilacji (np. Lombok) | `annotationProcessor 'org.projectlombok:lombok:1.18.38'` |
| `compileOnly` + `annotationProcessor` | Lombok/MapStruct - generowanie kodu bez dołączania biblioteki do aplikacji | `compileOnly 'org.projectlombok:lombok:1.18.38'`<br>`annotationProcessor 'org.projectlombok:lombok:1.18.38'` |

Aktualne wersje bibliotek znajdziesz na [Maven Central](https://central.sonatype.com/).

---

## Gradle Wrapper (`gradlew`)

**Gradle Wrapper** to mały skrypt dołączony do projektu, który sam pobiera i uruchamia wersję Gradle wymaganą
przez projekt. Dzięki temu:

- **nie musisz instalować Gradle** - wystarczy Java,
- każdy (Ty, prowadzący, GitHub Actions) buduje projekt **dokładnie tą samą wersją Gradle**.

Wrapper to te pliki - wszystkie muszą być w repozytorium (commitujemy je):

```
gradlew                                 # skrypt dla Linuksa/macOS
gradlew.bat                             # skrypt dla Windowsa
gradle/wrapper/gradle-wrapper.jar
gradle/wrapper/gradle-wrapper.properties  # tu jest zapisana wersja Gradle (distributionUrl)
```

Natomiast katalogi `.gradle/` i `build/` są generowane przy każdym buildzie i powinny być w `.gitignore`.

Wszystkie polecenia w dalszej części uruchamiamy przez wrapper:

- Linux / macOS: `./gradlew <task>`
- Windows: `gradlew.bat <task>` (w PowerShellu: `.\gradlew.bat <task>`)

> Jeśli na Linuksie/macOS lub w GitHub Actions dostajesz `Permission denied` przy `./gradlew`, plik nie ma prawa
> do wykonywania. Napraw to raz w repozytorium: `git update-index --chmod=+x gradlew` i zrób commit.

Wersję Gradle w projekcie zmienisz poleceniem `./gradlew wrapper --gradle-version <wersja>`
(np. `latest`) - zaktualizuje ono pliki wrappera, które potem commitujesz.

---

## Instalacja Javy

Gradle potrzebuje zainstalowanego JDK. Na laboratoriach z PO używamy **Javy 25** (wersja LTS).

Sprawdź, czy masz JDK i w jakiej wersji:

```bash
java -version
```

Jeśli nie masz JDK 25:

- **najprościej** - w IntelliJ: *File → Project Structure → SDKs → + → Download JDK*, wersja 25,
  dystrybucja np. *Eclipse Temurin*,
- **Windows / macOS / Linux** - pobierz instalator Temurin 25 ze strony [Adoptium](https://adoptium.net/),
- **macOS / Linux** - przez [SDKMAN!](https://sdkman.io/): `sdk install java 25.0.x-tem`, gdzie `x` to najnowsza
  poprawka (listę wersji pokaże `sdk list java`).

Gradle uruchamiany z konsoli korzysta z JDK wskazanego przez zmienną `JAVA_HOME` (albo z `java` w `PATH`).

Java 25 wymaga **Gradle 9.1 lub nowszego** - sprawdź `distributionUrl` w `gradle/wrapper/gradle-wrapper.properties`.

---

## Tworzenie projektu Java z Gradle

Projekt można utworzyć na dwa sposoby:

- **w IntelliJ** (tak robimy na laboratoriach z PO): *New Project* → `Language: Java`, `Build system: Gradle`,
  `Gradle DSL: Groovy`, `JDK: 25`. IntelliJ sam wygeneruje pliki Gradle razem z wrapperem;
- **z konsoli** poleceniem `gradle init` - to jedyny moment, w którym potrzebny jest zainstalowany Gradle
  ([instrukcja instalacji](https://gradle.org/install/)). Poniżej opisujemy ten wariant, bo dobrze pokazuje,
  co Gradle generuje.

### Krok 1: Inicjalizacja projektu

1. Utwórz nowy katalog projektu:
   ```bash
   mkdir MyGradleApp
   cd MyGradleApp
   ```
2. Zainicjalizuj projekt Gradle:
   ```bash
   gradle init
   ```
   - Type of build: **Application**
   - Implementation language: **Java**
   - Java version: **25**
   - Application structure: **Single application project**
   - Build script DSL: **Kotlin** lub **Groovy** (przykład w tym repozytorium używa Kotlin)
   - Test framework: **JUnit Jupiter**
   - Pozostałe opcje możesz zostawić domyślne.

### Krok 2: Struktura projektu

Po inicjalizacji Gradle utworzy taką strukturę (dokładnie taką, jak w tym repozytorium):

```
MyGradleApp/
 ├─ settings.gradle.kts          # nazwa projektu i lista modułów (tu: app)
 ├─ gradle.properties            # ustawienia Gradle
 ├─ gradle/
 │   ├─ libs.versions.toml       # katalog wersji bibliotek (version catalog)
 │   └─ wrapper/                 # Gradle Wrapper
 ├─ gradlew
 ├─ gradlew.bat
 └─ app/
     ├─ build.gradle.kts         # konfiguracja modułu: pluginy, zależności, toolchain
     └─ src/
         ├─ main/java/org/example/App.java
         └─ test/java/org/example/AppTest.java
```

Projekt z laboratoriów (utworzony w IntelliJ) jest prostszy: nie ma modułu `app`, więc `build.gradle`
i `src/` leżą bezpośrednio w katalogu projektu.

### Krok 3: Budowanie projektu

Aby skompilować projekt i uruchomić testy:

```bash
./gradlew build
```

- Gradle pobiera zależności i tworzy katalog `build/` z wynikami kompilacji.
- Testy jednostkowe uruchamiają się automatycznie w ramach `build` - jeśli któryś nie przejdzie, build się nie powiedzie.

### Krok 4: Uruchamianie aplikacji

Jeśli projekt używa pluginu `application`, możesz uruchomić aplikację:

```bash
./gradlew run
```

Argumenty dla metody `main()` przekazujemy przez `--args`:

```bash
./gradlew run --args="f b r l"
```

### Krok 5: Uruchamianie testów

Aby uruchomić tylko testy jednostkowe:

```bash
./gradlew test
```

- Raport z testów znajdziesz w pliku `app/build/reports/tests/test/index.html`
  (w projekcie z laboratoriów: `build/reports/tests/test/index.html`).

### Inne przydatne polecenia

| Polecenie | Co robi |
|---|---|
| `./gradlew tasks` | lista dostępnych tasków |
| `./gradlew clean` | usuwa katalog `build/` |
| `./gradlew clean build` | buduje projekt od zera |
| `./gradlew test --tests "org.example.AppTest"` | uruchamia wybraną klasę testową |
| `./gradlew dependencies` | drzewo zależności projektu |
| `./gradlew --version` | wersja Gradle i Javy, z której korzysta Gradle |

### Gradle w IntelliJ

IntelliJ importuje projekt na podstawie plików Gradle. Po zmianie `build.gradle` kliknij ikonę
**Load Gradle Changes** (ikona słonia ze strzałką) albo użyj okna **Gradle** (prawy pasek) → *Reload All Gradle Projects*.
W tym samym oknie znajdziesz wszystkie taski (`Tasks → build`, `Tasks → verification → test`), które można
uruchomić dwuklikiem.

---

## Automatyczne budowanie i testy na GitHubie (CI)

Jeśli chcesz, aby GitHub sam sprawdzał po każdym pushu, czy projekt się buduje, a testy przechodzą, skopiuj
plik [`.github/workflows/ci.yml`](.github/workflows/ci.yml) z tego repozytorium do swojego projektu,
zachowując ścieżkę `.github/workflows/` w **głównym katalogu repozytorium**. Jeśli projekt Gradle nie leży
w głównym katalogu repozytorium, ustaw jego katalog w `working-directory`.

Co to jest CI, jak działają GitHub Actions i jak skonfigurować je dla projektu z laboratoriów z PO -
opisujemy w repozytorium [git-hub-actiosns-sample-core](https://github.com/sumo-slonik/git-hub-actiosns-sample-core).
