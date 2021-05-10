# Temeller

Web erişilebilirliği (a11y olarak da bilinir), herkes tarafından kullanılabilecek web siteleri oluşturma yöntemini ifade eder ve engelli bir kişi, yavaş bağlantı, eski, bozuk bir cihaz veya sadece elverişsiz bir ortamdaki kullanıcıları kapsar. Örneğin, bir videoya altyazı eklemek, hem işitme engelli hem de işitme güçlüğü çeken kullanıcılarınıza ve gürültülü bir ortamda bulunan ve telefonlarını duyamayan kullanıcılarınıza yardımcı olur. Benzer şekilde, metninizin çok düşük kontrastlı olmadığından emin olmak, hem az gören kullanıcılarınıza hem de telefonlarını parlak güneş ışığında kullanmaya çalışan kullanıcılarınıza yardımcı olacaktır.

Başlamaya hazırsınız ama nereden başlayacağınızı bilmiyor musunuz?

[World Wide Web Consortium (W3C)](https://www.w3.org/) tarafından hazırlanmış olan [web erişebilirlik planlama ve yönetme kılavuzuna](https://www.w3.org/WAI/planning-and-managing/) bakabilirsiniz.

## Bağlantıyı atla

Her sayfanın üstüne, doğrudan ana içerik alanına giden bir bağlantı eklemelisiniz, böylece kullanıcılar, Web sayfasında birden çok tekrarlanan içeriği atlayabilir.

Tipik olarak bu, tüm sayfalarınızda ilk odaklanılabilir öğe olacağı için `App.vue` dosyasının üstünde yapılır:

```html
<ul class="linki-gec">
  <li>
    <a href="#main" ref="linkiGec">Ana içeriği geç</a>
  </li>
</ul>
```

Odaklanmadıkça bağlantıyı gizlemek için aşağıdaki CSS stilini ekleyebilirsiniz:

```css
.linkiGec {

  white-space: nowrap;

  margin: 1em auto;

  top: 0;

  position: fixed;

  left: 50%;

  margin-left: -72px;

  opacity: 0;
}

.linkiGec:focus {
  opacity: 1;

  background-color: white;

  padding: 0.5em;

  border: 1px solid black;
}
```

Bir kullanıcı rotayı değiştirdiğinde, odağı atlama bağlantısına geri getirin. Bu, aşağıda verilen "ref" e focus fonksiyonu çağrılarak elde edilebilir:

```vue
<script>
export default {
  watch: {
    $route() {
      this.$refs.linkiGec.focus()
    },
  },
}
</script>
```

<common-codepen-snippet title="Skip to Main" slug="GRrvQJa" :height="350" tab="js,result" theme="light" :preview="false" :editable="false" />

[Ana içeriğe giden bağlantıyı atlamayla ilgili dökümanı okuyun](https://www.w3.org/WAI/WCAG21/Techniques/general/G1.html)

## İçeriğinizi Yapılandırın

Erişilebilirliğin en önemli parçalarından biri, tasarımın erişilebilir uygulamayı destekleyebildiğinden emin olmaktır. Tasarım sadece renk kontrastını, yazı tipi seçimini, metin boyutunu ve dili değil, aynı zamanda içeriğin uygulamada nasıl yapılandırıldığını da dikkate almalıdır.

### Başlık

Kullanıcılar başlıklar aracılığıyla bir uygulamada gezinebilirler. Uygulamanızın her bölümü için açıklayıcı başlıklara sahip olmak, kullanıcıların her bölümün içeriğini tahmin etmesini kolaylaştırır. Başlıklar söz konusu olduğunda, önerilen birkaç erişilebilirlik kuralı vardır:

- Başlıkları sıralamaya göre yerleştiriniz: `<h1>` - `<h6>`

* Bir section içindeki başlıkları atlamayın

- Başlıkların görsel görünümünü vermek için metnin stilini ayarlamak yerine gerçek başlık etiketlerini kullanın

[Daha fazla başlık ile ilgili](https://www.w3.org/TR/UNDERSTANDING-WCAG20/navigation-mechanisms-descriptive.html)

```html
<main role="ana" aria-labelledby="ana-baslik">
  <h1 id="ana-baslik">Ana Başlık</h1>

  <section aria-labelledby="section-baslik">
    <h2 id="section-baslik">Section Başlığı</h2>

    <h3>Section Alt Başlığı</h3>

    <!-- İçerik -->
  </section>

  <section aria-labelledby="section-baslik">
    <h2 id="section-baslik">Section Başlığı</h2>

    <h3>Section Alt Başlığı</h3>

    <!-- İçerik -->

    <h3>Section Alt Başlığı</h3>

    <!-- İçerik -->
  </section>
</main>
```

### Yer işaretleri (Landmark)

Yer işaretleri, bir uygulama içindeki bölümlere programlı erişim sağlar. Yardımcı teknolojiye güvenen kullanıcılar, uygulamanın her bölümüne gidebilir ve içeriği atlayabilir. [ARIA rollerini](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles) arşivlemek için kullanabilirsiniz.

| HTML            | ARIA Rolü           | Yer İşaret Amacı                                                                                      |
| --------------- | ------------------- | ----------------------------------------------------------------------------------------------------- |
| header          | rol="banner"        | İlk başlık: sayfanın başlığı                                                                          |
| nav             | rol="navigation"    | Sayfada veya ilgili sayfalarda gezinirken kullanıma uygun bağlantıların toplanması                    |
| main            | rol="main"          | Sayfanın ana veya merkezi içeriği.                                                                    |
| footer          | rol="contentinfo"   | Ana sayfa hakkında bilgiler: dipnotlar / telif hakları / gizlilik bildirimine bağlantılar             |
| aside           | rol="complementary" | Ana içeriği destekler, ancak kendi içeriğine göre ayrılmış ve anlamlıdır                              |
| _Not available_ | rol="search"        | Bu bölüm, uygulama için arama işlevini içerir.                                                        |
| form            | rol="form"          | Formla ilişkili öğelerin toplanması                                                                   |
| section         | rol="region"        | Alakalı ve kullanıcıların büyük olasılıkla gitmek isteyeceği içerik. Bu öğe için etiket sağlanmalıdır |

:::tip İpucu:
[HTML5 semantik öğelerini desteklemeyen eski tarayıcılarla uyumluluğunu](https://caniuse.com/#feat=html5semantic) en üst düzeye çıkarmak için etkisiz yer işareti rol özniteliklerine sahip HTML öğelerinin kullanılması önerilir.

:::

[Yer işaretleri ile ilgili daha fazla bilgi](https://www.w3.org/TR/wai-aria-1.2/#landmark_roles)
