# OkHttp

[Документация](https://square.github.io/okhttp/)      [Стек возможных багов](https://stackoverflow.com/questions/tagged/okhttp?sort=active)

```groovy
 dependencies {
       // определить спецификацию и ее версию
       implementation(platform("com.squareup.okhttp3:okhttp-bom:4.9.0"))

       // определить любые необходимые артефакты OkHttp без версии
       implementation("com.squareup.okhttp3:okhttp")
       implementation("com.squareup.okhttp3:logging-interceptor")
    }
```

