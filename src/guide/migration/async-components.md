---
badges:
  - yeni
---

# Asenkron Bileşenler <MigrationBadges :badges="$frontmatter.badges" />

## Genel Bakış

Değişikliklere dair özet bilgileri aşağıda bulabilirsiniz:

- Yeni `defineAsyncComponent` yardımcı metodu asenkron bileşenleri açıkça tanımlamayı sağlıyor
- `component` seçeneğinin adı `loader`olarak değiştirildi
- Loader fonksiyonu kendiliğinden `resolve` ve `reject` argümanlarına ihtiyaç duymuyor ve mutlaka bir Promise göndermesi gerekiyor

Aşağıda bu yeniliklere dair daha detaylı açıklamaları bulabilirsiniz!

## Giriş

Şimdiye kadar asenkron bileşenler bir promise gönderen bir fonksiyonun tanımlanmasıyla oluşturuluyordu. Örneğin:

```js
const asyncModal = () => import('./Modal.vue')
```

Veya daha ileri düzey, seçenekler içeren bir bileşen sentaksi ile örnek verecek olursak:

```js
const asyncModal = {
  component: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```

## 3.x Sentaksı

Şu andan itibaren, Vue 3'de fonksiyonel bileşenler saf fonksiyonlar olarak tanımlandığı için asenkron bileşenlerin yeni `defineAsyncComponent` yardımcı methodu kullanılarak açık bir şekilde tanımlanması gerekiyor:

```js
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// Async component without options
const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))

// Async component with options
const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

2.x'e kıyasla farklılık gösteren bir diğer yenilik `component` seçeneğinin adının `loader` olarak değiştirilmesi. Bu değişikliğin sebebi bir bileşen tanımının doğrudan kullanılamayacağını doğru bir şekilde ifade edebilmek.

```js{4}
import { defineAsyncComponent } from 'vue'

const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

Ayrıca 2.x'nin tersine loader fonksiyonu artık `resolve` ve `reject` parametrelerini almıyor ve her daim bir Promise göndermesi gerekiyor.

```js
// 2.x versiyonu
const oldAsyncComponent = (resolve, reject) => {
  /* ... */
}

// 3.x versiyonu
const asyncComponent = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      /* ... */
    })
)
```

Asenkron bileşenlerin kullanımı hakkında daha fazla bilgi için şu başlığa bakabilirsiniz:

- [Kılavuz: Dinamik ve Asenkron Bileşenler](/guide/component-dynamic-async.html#dynamic-components-with-keep-alive)
