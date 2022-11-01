---
description: >-
  En çok kullanılan Jetpack ve 3. parti kütüphanelerini içerir. Proje de gerekli
  bağımlılıkları tekrar tekrar aramamak için burada toplamak istiyorum.
---

# Android Dependencies

### Room - Database

```groovy
dependencies {
    def room_version = "2.4.3"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"

    // To use Kotlin annotation processing tool (kapt)
    kapt "androidx.room:room-compiler:$room_version"
    // To use Kotlin Symbol Processing (KSP)
    ksp "androidx.room:room-compiler:$room_version"

    // optional - RxJava2 support for Room
    implementation "androidx.room:room-rxjava2:$room_version"

    // optional - RxJava3 support for Room
    implementation "androidx.room:room-rxjava3:$room_version"

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation "androidx.room:room-guava:$room_version"

    // optional - Test helpers
    testImplementation "androidx.room:room-testing:$room_version"

    // optional - Paging 3 Integration
    implementation "androidx.room:room-paging:2.5.0-beta01"
}
```

### Lifecycle (ViewModel - LiveData )

```groovy
    dependencies {
        def lifecycle_version = "2.6.0-alpha02"
        def arch_version = "2.1.0"

        // ViewModel
        implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
        // ViewModel utilities for Compose
        implementation "androidx.lifecycle:lifecycle-viewmodel-compose:$lifecycle_version"
        // LiveData
        implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
        // Lifecycles only (without ViewModel or LiveData)
        implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"
        // Annotation processor
        kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
        
    }
```

### Navigation

```groovy
dependencies {
  def nav_version = "2.5.2"

  // Kotlin
  implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
  implementation "androidx.navigation:navigation-ui-ktx:$nav_version"

  // Jetpack Compose Integration
  implementation "androidx.navigation:navigation-compose:$nav_version"
}
```

### Retrofit

```groovy
implementation 'com.squareup.retrofit2:retrofit:2.6.0'
implementation "com.squareup.okhttp3:logging-interceptor:4.5.0"

//Adapters -  RxJava3
implementation 'com.squareup.retrofit2:adapter-rxjava3:latest.version'

//Converters - Moshi
implementation 'com.squareup.retrofit2:converter-moshi:latest.version'
implementation 'com.squareup.retrofit2:converter-gson:2.6.0'
```

### Picasso

```groovy
implementation 'com.squareup.picasso:picasso:2.8'
```

### Glide

```groovy
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.14.2'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.14.2'
}
```

### Hilt - DI

{% code title="Groovy" %}
```groovy
plugins {
  ...
  id 'com.google.dagger.hilt.android' version '2.44' apply false
}

```
{% endcode %}

{% code title="Groovy" %}
```groovy
...
plugins {
  id 'kotlin-kapt'
  id 'com.google.dagger.hilt.android'
}

android {
  ...
}

dependencies {
  implementation "com.google.dagger:hilt-android:2.44"
  kapt "com.google.dagger:hilt-compiler:2.44"
}

// Allow references to generated code
kapt {
  correctErrorTypes true
}

```
{% endcode %}

### Paging

```
dependencies {
  def paging_version = "3.1.1"

  implementation "androidx.paging:paging-runtime:$paging_version"

  // alternatively - without Android dependencies for tests
  testImplementation "androidx.paging:paging-common:$paging_version"

  // optional - RxJava2 support
  implementation "androidx.paging:paging-rxjava2:$paging_version"

  // optional - RxJava3 support
  implementation "androidx.paging:paging-rxjava3:$paging_version"

  // optional - Guava ListenableFuture support
  implementation "androidx.paging:paging-guava:$paging_version"

  // optional - Jetpack Compose integration
  implementation "androidx.paging:paging-compose:1.0.0-alpha17"
}

```
