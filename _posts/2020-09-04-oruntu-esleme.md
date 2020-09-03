---
layout: post
title: "Örüntü Eşlemeyi Anlamak"
date: 2020-09-03
excerpt: "Rust'tın örüntü eşleme modeli üzerine bir makale"
tags: [örüntü eşleme, pattern matching, rust, match]
comments: true
---

> Bu makalenin aslı [Bruno Oliveira](https://dev.to/brunooliveira) tarafından yazılmış olup [dev.to](https://dev.to/> brunooliveira/learning-rust-understanding-pattern-matching-10b3) sitesinde yayınlanmıştır.


Rust, varsayılan olarak değişmezliği destekler, bu durum diğer dillere aşina olanlar için beklenmedik davranışlara yol açabilir. Bu değişmezliğin örüntü eşleme ile nasıl birleştirilebileceğini ve belirli girdilere göre belirli eylemleri işletirken ne denli güçlü bir araç olduğunu anlamaya çalışacağız.   

## Anahtar kelime `let`

Rust’ta değişkenleri bildirmek için `let` anahtar kelimesi kullanılır. Bu anahtar kelimenin ardından mut anahtar kelimesinin kullanılması, bu değişken değerinin çalışma zamanında yeniden atanarak değiştirilebileceğini gösterir. Söz dizimi aşağıdaki gibidir.

```rust
fn main() {
    let x = 4;
    let mut y = 1;
}
```

Yani, yukarıdaki örnekte, `x` ve `y` adlarında tamsayı değerlerini tutan iki değişkeni bildirir ve başlatırız. Bu iki değişken arasındaki temel fark; `y` değişkeninin değeri çalışma zamanında değiştirilebilir iken, `x` değişkeninin hiçbir koşulda değiştirilemiyor olmamasıdır. Haliyle `y` değişkeni değerini bir arttıran aşağıdaki kod gayet güzel çalışacaktır. 

```rust
println!("Değeri arttırılmadan önceki y: {}", y);
    
y = y + 1;
println!("Arttırıldıktan sonraki y: {}", y);
```

Bu kodu çalıştırdığımızda aşağıdaki gibi bir çıktıyla karşılaşmış oluruz.

```console
Değeri arttırılmadan önceki y: 1
Arttırıldıktan sonraki y: 2
```

Fakat aynı işlemi `x` değişmezi için yapmaya kalkıştığımızda alacağımız şey:

```console
error[E0384]: cannot assign twice to immutable variable `x`
```

Hatanın açıklamasında `x` değişmezine yeniden bir değer atanamayacağı bildirilir. Bu pratik üzerinden; değişebilir ve değişmez ilklendirmelerinde `let` anahtar sözcüğünün kullanımın aynı olduğunu, fakat `mut` ön eki kullanılarak değişirliğin eklenmesiyle, çalışma zamanındaki süreçlerin etkileneceğini gözlemleyebiliyoruz.

## Örüntü Eşleme

Oysa `let` anahtar sözcüğü ile basit değer atamalarından fazlası yapılabilir. Karmaşık bir ifadeyle uğraşırken belirli bir eylemi yürütmek için ifadeyle eşleşecek bir örüntü kullanabiliriz. 
Örüntüler, basit ve karmaşık veri yapılarını eşleştirmek için kullanılan Rust’a özel söz dizimleridir. Örüntüleri, eşleme ifadeleri ve diğer yapılarla birlikte kullanmak programın akışında daha fazla kontrol olanağı sağlar. Bir desen aşağıdakilerin bazı kombinasyonlarından oluşur:

* Değişmezler *-Literals*
* Yapılandırılmamış diziler, sıralamalar *(enums)*, yapılar *(struct)* veya çokuzlular *(tuples)*
* Değişkenler *-Variables*
* Joker karakterler *-Wildcards*
* Yer tutucular *-Placeholders*

Bu bileşenler, üzerinde çalıştığımız verilerin şeklini açıklar. Programımızın belirli bir kod parçasını çalıştırırken doğru verilere sahip olup olmadığını bu değerlerleri eşleştirmek suretiyle denetleyerek gerçekleştiririz.
Bir örüntünün kullanılabilmesi için öncelikle bir değerle karşılaştırılması gereklidir. Örüntü karşılaştırılan değerle eşleştiğinde kodun değer parçaları devreye girer.

Bir örüntü, üzerinde çalışılan verileri tanımlayan bileşenlerin bir kombinasyonudur. Bu kombinasyon bir değerle karşılaştırıldığında bir eşleşme sağlanıyorsa belirli bir eylem gerçekleştirilir. Örüntüyle eşleşmeyen değerlerle ilgili işlem gerçekleştirilmez.

Aşağıdaki basit örnekte görüleceği gibi, belirli bir zamanda, on numaralı tren yolu hattına bir trenin gelip gelmeyeceğini kontrol edebilir, değerlendirme sonucuna göre yolcular için uygun bir bilgi mesajı yazdırabiliriz. 

```rust
fn main() {
   let hat_no = 10;
   
   match hat_no {
       10 => println!("Tramvay 10. hatta gelecek"),
       _ => println!("Kara tren gecikir belki hiç geeelmeez"),
   }
}
```
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=3d733583a8170cb373881b7fed698b1b)

Yukarıdaki kodu çalıştırdığımızda ekrana "Tramvay 10. hatta gelecek" mesajı yazdırılır. Şimdi bu örnek üzerinden sözdizimini kısaca inceleyelim. 

Yukarıda bir `match` anahtar kelimesinin bildirildiğini görüyoruz. Eşleme ifadeleri, `match` anahtar sözcüğü, eşleşecek bir değer ve bir örüntüden oluşan bir veya daha fazla eşleşme kolu ve değerin o kolun örüntüsüyle eşleşmesi halinde çalıştırılacak bir ifade olarak tanımlanırlar.

```rust
match DEGER {
    ORUNTU => IFADE,
    ORUNTU => IFADE,
    ORUNTU => IFADE,
}
```

Eşleştirme ifadelerinde kullanılan değerlerin karşılaştırılmasında tüm karşılaştırma olasılıklarının değerlendirilme zorunluluğu, bu ifadelerin oldukça kapsamlı hale getirir. Her olasılığı değerlendirmenin bir yolu, son kol için bir tüm olasılıklar örüntüsüne sahip olmaktır. Örneğin, herhangi bir değerle eşleşen değişken adı asla başarısız olamayacağından geriye kalan her olasılığı veya durumu doğallıkla kapsamış olacaktır.

Özel bir hata baskılama işleci olan alt çizgi **`_`** herhangi bir örüntü kalıbıyla eşleşmesine rağmen hiçbir zaman bir değişkene bağlanmaz. Bu nedenle son eşleme kolunda kullanılır. Bu **`_`** işleç genellikle belirtilmeyen bir değerin yok sayılması gerektiğinde oldukça yararlıdır.

Eşleştirme ifadelerinde bir dizi değerin de karşılaştırılması yapılabilir. Yukarıdaki tramvay örneğini *tramvayın geliş süresini on dakika* olacak şekilde güncelleyelim. Eğer eşleştirilen değer bu sürenin dışında ise tramvayın gecikeceğini bildiren bir mesaj yazdıralım.

Kapsayıcı bir değer aralığıyla nasıl eşleştirme yapılacağı aşağıdaki örnek üzerinde açıklanmıştır:

```rust
fn main() {
   let varis = 10;
   
   match varis {
       1..=10 => println!("Trenimiz 10 dakika içinde gelecek!"),
       _ => println!("Gecikme var"),
   }
}
```
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=3d733583a8170cb373881b7fed698b1b)


Özel bir sözdizimi olan **`..=deger`**, bir örüntüyü bir dizi değerle eşleştirmek için kullanılabilir.
Örüntüyü bir değer ya da başka bir değer ile eşleştirmek için **`|`** boru işlecini de kullanabiliriz. 

```rust
fn main() {
   let bos_koltuk = 2;
   
   match bos_koltuk {
       1 | 2 => println!("Trende 1 ya da 2 koltuk boş"),
       3 => println!("Trende 3 koltuk boş"),
       _ => println!("boş koltuk adedi bilinmiyor"),
   }
}
```
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=62828b727ed6239cec9ecc96e2e37f32)

Bu makalede örüntü eşlemeyi kullanarak Rust programlarının yürütme akışını kontrol etmeye hızlıca bir giriş yaptık.  Örüntü eşlemenin kümeler ve değer aralıkları gibi karmaşık durumları ela almak için nasıl kullanılabileceğini basit olan `if-else` yapılarından daha becerikli ve çok yönlü olduğunu kavramaya çalıştık.

Herhangi bir eşleşmemiş örüntünün kapsanabilmesi için zorunlu olarak kullanılan bir örüntü kodu daha sağlam ve güvenli hale getirecektir.