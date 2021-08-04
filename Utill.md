
1. Зарузка картинок
*  **Picasso**
  [Ссылка](https://square.github.io/picasso/)
  [Ссылка](https://code.tutsplus.com/ru/tutorials/code-an-image-gallery-android-app-with-picasso--cms-30966)
   ```kotlin
   Picasso.get()
    .load(url)
    .placeholder(R.drawable.user_placeholder)
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
   ```
  * **get()** - возвращает глобальный экземпляр Picasso
  * **load()** - запускает запрос изображения с использованием указанного пути
  * **error(int errorResId)** - можно использовать, если запрошенное изображение не может быть загружено
  * **into(imageView)** - изображение целевого изображения, в которое будет помещено изображение.

