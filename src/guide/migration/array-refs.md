---
title: v-for Refs Dizileri
badges:
- kırıcı
---

# {{ $frontmatter.title }} <MigrationBadges :badges="$frontmatter.badges" />

Vue 2'de `ref` niteliğini `v-for` içerisinde kullanmak ilişkili `$refs` değerini bir refs dizisi ile dolduruyordu. Bu işleyiş birbiri içerisine yerleştirilmiş `v-for`lar için anlaşılması zor ve verimsiz bir sonuç ortaya koyuyordu.

Vue 3'de bu yöntem `$refs` içerisinde otomatik olarak bir dizi oluşturmayacak. Tek bir bağ aracılığıyla birçok refs elde etmek için `ref`i bir fonksiyona bağlayın. Bu teknik daha fazla esneklik temin edecektir (bu teknik Vue 3'ün yeniliklerinden biri):

```html
<div v-for="item in list" :ref="setItemRef"></div>
```

Seçenekler API'si kullanıldığında:

```js
export default {
  data() {
    return {
      itemRefs: []
    }
  },
  methods: {
    setItemRef(el) {
      if (el) {
        this.itemRefs.push(el)
      }
    }
  },
  beforeUpdate() {
    this.itemRefs = []
  },
  updated() {
    console.log(this.itemRefs)
  }
}
```

Birleşim API'si kullanıldığında:

```js
import { onBeforeUpdate, onUpdated } from 'vue'

export default {
  setup() {
    let itemRefs = []
    const setItemRef = el => {
      if (el) {
        itemRefs.push(el)
      }
    }
    onBeforeUpdate(() => {
      itemRefs = []
    })
    onUpdated(() => {
      console.log(itemRefs)
    })
    return {
      setItemRef
    }
  }
}
```

Not:

- `itemRefs` bir dizi olmak zorunda değil: Bir nesne de olabilir ve refs'ler iterasyon anahtarları ile tayin edilir.

- Bu sayede gerekli olması halinde `itemRefs` reaktifleştirilebilir ve watch yöntemiyle izlenebilir.
