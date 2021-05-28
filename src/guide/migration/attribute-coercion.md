---
badges:
  - kırıcı
---

# Nitelik Zorlama Davranışı <MigrationBadges :badges="$frontmatter.badges" />

::: info Bilgi
Bu API içerisinde gerçekleştirilen düşük seviyeli bir değişikliktir ve programcıların büyük bir çoğunluğunu etkilemez.
:::

## Genel Bakış

Değişikliklerin kısa bir özetini aşağıda bulabilirsiniz:

- Zorlanan niteliklere dayalı dahili konsept geride bırakılıyor ve bu nitelikler boolean olmayan nitelikler ile aynı şekilde dikkate alınıyor
- **KIRICI**: Değerin boolean `false` olduğu durumlarda artık nitelik ortadan kaldırılmıyor. Onun yerine attr="false" olarak tayin ediliyor. Niteliği ortadan kaldırmak için `null` veya `undefined` değerlerini kullanın.

Daha detaylı bilgiler için okumaya devam edin!

## 2.x Sentaksı

2.x versiyonlarında `v-bind` değerlerini zorlamak için aşağıdaki stratejileri uyguluyorduk:

- Bazı nitelik/element çiftleri için Vue her daim karşılık gelen IDL niteliğini (özelliği) kullanıyor: [örneğin `<input>`, `<select>`, `<progress>`, vb elemenlerin `value` niteliği](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L11-L18).

- "[Boolean nitelikler](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L33-L40)" ve [xlinkler](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L44-L46) için, Vue "yanlış" ([`undefined`, `null` or `false`](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L52-L54)) olarak değerlendirilen nitelikleri ortadan kaldırırken diğer türden verileri olduğu gibi ekliyor ([buraya](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L66-L77) ve [buraya](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L81-L85) bakın).

- "[Zorlanan nitelikler](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L20)" için (şimdilik `contenteditable`, `draggable` ve `spellcheck`), Vue bu değerleri dizgiye [dönüştürür](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L24-L31) (`contenteditable` şimdilik istisna oluşturmaktadır. Düzeltme takibi için buraya bakabilirsiniz [vuejs/vue#9397](https://github.com/vuejs/vue/issues/9397)).

- Diğer nitelikler için "yanlış" olarak değerlendirilen değerleri (`undefined`, `null`, veya `false`) ortadan kaldırır ve diğer değerleri olduğu gibi korur ([buraya](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L92-L113)) bakın.

Aşağıdaki tablo Vue'nin "zorlanan nitelikleri" istisnai olarak normal boolean olmayan değerlere nasıl dönüştürdüğünü açıklar:

| Bağlanan ifade  | `foo` <sup>normal</sup> | `draggable` <sup>zorlanmış</sup> |
| ------------------- | ----------------------- | --------------------------------- |
| `:attr="null"`      | /                       | `draggable="false"`               |
| `:attr="undefined"` | /                       | /                                 |
| `:attr="true"`      | `foo="true"`            | `draggable="true"`                |
| `:attr="false"`     | /                       | `draggable="false"`               |
| `:attr="0"`         | `foo="0"`               | `draggable="true"`                |
| `attr=""`           | `foo=""`                | `draggable="true"`                |
| `attr="foo"`        | `foo="foo"`             | `draggable="true"`                |
| `attr`              | `foo=""`                | `draggable="true"`                |

Yukarıdaki tablodan görebileceğimiz gibi, bu sistem `true`yu `'true'`ya zorlar fakat değer `false` ise niteliği kaldırır. Bu aynı zamanda tutarsızlığa yol açıyor ve kullanıcıların `aria-selected`, `aria-hidden` vb. `aria-*` nitelikleri gibi çok yaygın kullanım durumlarında boolean değerlerini manuel olarak dizeye zorlamalarını gerektiriyor.

## 3.x Sentaksı

Bu dahili "zorlanan nitelik" konseptini bırakmayı ve bunları normal, boolean olmayan HTML nitelikleri olarak ele almayı amaçlıyoruz.

- Bu, normal boolean olmayan nitelikler ve “zorlanan nitelikler” arasındaki tutarsızlığı çözüyor
- Bu aynı zamanda `'true'` ve `'false'` dışındaki değerleri ve hatta gelecekte kullanılmaya başlanabilecek anahtar kelimeleri `contenteditable` gibi nitelikler için kullanmayı mümkün kılıyor

Boolean olmayan nitelikler için Vue, `false` değerlerini kaldırmayı durduracak ve bunun yerine onları `'false'` olmaya zorlayacak.

- Bu, `true` ve `false` arasındaki tutarsızlığı çözüyor ve `aria-*` niteliklerinin daha kolay kullanılmasını sağlıyor

Aşağıdaki tablo bu yeni davranışı tarif eder:

| Bağlanan ifade  | `foo` <sup>normal</sup>    | `draggable` <sup>zorlanmış</sup> |
| ------------------- | -------------------------- | --------------------------------- |
| `:attr="null"`      | /                          | / <sup>*</sup>                    |
| `:attr="undefined"` | /                          | /                                 |
| `:attr="true"`      | `foo="true"`               | `draggable="true"`                |
| `:attr="false"`     | `foo="false"` <sup>*</sup> | `draggable="false"`               |
| `:attr="0"`         | `foo="0"`                  | `draggable="0"` <sup>*</sup>      |
| `attr=""`           | `foo=""`                   | `draggable=""` <sup>*</sup>       |
| `attr="foo"`        | `foo="foo"`                | `draggable="foo"` <sup>*</sup>    |
| `attr`              | `foo=""`                   | `draggable=""` <sup>*</sup>       |

<small>*: değişti</small>

Boolean niteliklerin zorlanması olduğu gibi bırakıldı.

## Yeni Versiyona Geçiş Stratejisi

### Zorlanan nitelikler

Zorlanan niteliklerin ve `attr="false"`nin kaldırılması aşağıda tarif edilen farklı (ve gerçek durumu yansıtan) IDL nitelik değerlerine yol açabilir:

| Kaldırılan zorlanmış nitelik | IDL nitelik ve değer                     |
| ---------------------- | ------------------------------------ |
| `contenteditable`      | `contentEditable` &rarr; `'inherit'` |
| `draggable`            | `draggable` &rarr; `false`           |
| `spellcheck`           | `spellcheck` &rarr; `true`           |

Eski davranışı korumak için ve `false`nin `'false'`ye zorlanmaya başlanması nedeniyle, Vue'nin 3.x versiyonlarında programcıların `contenteditable` ve `spellcheck` için `v-bind` ifadesini `false` or `'false'` olacak şekilde tayin etmeleri gerekiyor.

2.x versiyonlarında geçersiz değerler zorlanan nitelikler için `'true'`ye dönüştürülüyordu. Bu genellikle kasıtsızdı ve büyük ölçekte güvenilmesi pek olası değildi. 3.x versiyonlarında `true` veya `'true'`nin açıkça belirtilmesi gerekiyor.

### Niteliğin kaldırılması yerine `false` veya `'false'`ye dönüştürülmesi

3.x versiyonlarında niteliği ortadan kaldırmak için `null` veya `undefined` değerleri açık bir şekilde kullanılmalıdır.

### 2.x ve 3.x versiyonları arasındaki fark

<table>
  <thead>
    <tr>
      <th>Nitelik</th>
      <th><code>v-bind</code> değeri <sup>2.x</sup></th>
      <th><code>v-bind</code> değeri <sup>3.x</sup></th>
      <th>HTML sonucu</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">2.x “Zorlanan nitelikler”<br><small>örn. <code>contenteditable</code>, <code>draggable</code> ve <code>spellcheck</code>.</small></td>
      <td><code>undefined</code>, <code>false</code></td>
      <td><code>undefined</code>, <code>null</code></td>
      <td><i>çıkarılıyor</i></td>
    </tr>
    <tr>
      <td>
        <code>true</code>, <code>'true'</code>, <code>''</code>, <code>1</code>,
        <code>'foo'</code>
      </td>
      <td><code>true</code>, <code>'true'</code></td>
      <td><code>"true"</code></td>
    </tr>
    <tr>
      <td><code>null</code>, <code>'false'</code></td>
      <td><code>false</code>, <code>'false'</code></td>
      <td><code>"false"</code></td>
    </tr>
    <tr>
      <td rowspan="2">Boolean olmayan diğer nitelikler<br><small>örn. <code>aria-checked</code>, <code>tabindex</code>, <code>alt</code>, vs.</small></td>
      <td><code>undefined</code>, <code>null</code>, <code>false</code></td>
      <td><code>undefined</code>, <code>null</code></td>
      <td><i>çıkarılıyor</i></td>
    </tr>
    <tr>
      <td><code>'false'</code></td>
      <td><code>false</code>, <code>'false'</code></td>
      <td><code>"false"</code></td>
    </tr>
  </tbody>
</table>
