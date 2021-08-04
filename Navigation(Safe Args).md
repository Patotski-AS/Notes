# Safe Args
 **Safe Args** - это подключаемый модуль Gradle, который служит дл пеедачи параметров из оного фрамента в друой. Он генерирует код и классы, которые помогают обнаруживать ошибки во время компиляции, которые в противном случае не могли бы появиться до тех пор, пока приложение не запустится.

1. Добавляем плагин
   * Добавьте navigation-safe-args-gradle-plugin зависимость в **build.gradle(project)**
   ```groovy
        dependencies {
        // Adding the safe-args dependency to the project Gradle file
        classpath "androidx navigation:navigation-safe-args-gradle-plugin:$navigationVersion"
        }
   ```
   * В **build.gradle(app)** добавбте плагин *androidx.navigation.safeargs*
   ```groovy
    // Adding the apply plugin statement for safeargs
        apply plugin: 'androidx.navigation.safeargs'
   ```
