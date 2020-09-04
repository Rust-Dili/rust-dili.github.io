---
layout: post
title: "Rust'ta Result, Option, unwrap ve ? İşleci ile Hata Yönetimi"
date: 2020-09-04
excerpt: "Rust öğrenirken, Result, Option, unwrap ve ? işlecinden oluşan bu dört kavramın genellikle birbirleriyle birlikte tartışıldığını fark ettim. Bu yazıda, Rust’ı anlamamı ve daha deyimsel olmamı sağlayan bu kavramlar üzerindeki düşünce sürecimi belgelemek ve sizlerle paylaşmak istiyorum."
tags: [option, unwrap, result, hata işleme, null, None, Err, Some, Ok,json, serde_json, Serialize, Deserialize, panic!]
comments: false
---

> Bu yazının orjinal hali [Kevin Le](Kevin Le) tarafından [dev.to](https://dev.to/codeprototype/taking-the-unhappy-path-with-result-option-unwrap-and-operator-in-rust-482b)
> sitesinde tarafından yayınlanmıştır.

Rust öğrenirken, Result, Option, unwrap ve ? işlecinden oluşan bu dört kavramın genellikle birbirleriyle birlikte tartışıldığını fark ettim. Bu yazıda, Rust’ı anlamamı ve daha deyimsel olmamı sağlayan bu kavramlar üzerindeki düşünce sürecimi belgelemek ve sizlerle paylaşmak istiyorum.

## `Option<T>`

Rust’ ta `null` kavramı var olmasına rağmen ortak bir model olarak yaygın şekilde kullanılmaz. Örneğin; bir mobil işletim sistemi adı verildiğinde o işletim sistemine ait çevrimiçi mağaza adının döndürüleceği işlev yazmamız gerektiğini varsayalım. Bu işleve iletilen dizge değişmezi iOS olduğunda "App Store", android olduğunda ise "Play Store" döndürülecek ve diğer girdilerin geçersiz kabul edileceğini düşünelim. Böyle bir durumda diğer pek çok dilde işlevden geri döndürülecek değeri null veya geçersiz bir dizge değişmezi yahut ona benzer bir şey olarak seçebilirken, Rust'ta bu tür bir değer döndüremezsiniz.

Rust'ta bu türden işlevleri gerçekleştirmenin deyimsel yollarından biri, o işlevden `Option` türü döndürmekten geçer. `Option` veya tam olarak söylemek gerekirse `Option<T>` bir genellemedir ve; ya `Some<T>` ya da `None` varyantlarından birini döndürebilir. *(Bu noktadan itibaren sözlerimin daha anlaşılabilir ve sade olabilmesi için yazımı genelleme tür parametresi T olmadan aktarmaya gayret edeceğim.)*

Rust diğer dillerde eşdeğerleri bulunmayan ve bir enum türü olan `Option`'u `Some<T>` ve `None` değerlerinden oluşan varyantlar olarak kullanıma sunar. Bizim örneğimizde işlevimiz, başarılı dönüş değerlerinden `Some` varyantına sarılı "App Store" ya da "Play Store" dizgesini, başarısız dönüş değeri olarakta `None` varyantını döndürecektir.

```rust
fn magaza_bul(m_is: &str) -> Option<&str> {
    match m_is {
        "iOS" => Some("App Store"),
        "android" => Some("Play Store"),
        _ => None
    }
}
````

`magaza_bul()` işlevini kullanacak main işleviniyse basitçe aşağıdaki gibi oluşturabiliriz: 

```rust
fn main() {
    println!("{}", match magaza_bul("windows") {
        Some(s) => s,
        None => "Bu geçerli bir Mobil İşletim Sistemi değildir!"
    });
}

// Bu geçerli bir Mobil İşletim Sistemi değildir!
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=c92d6e3ba3f7493d3b9594706705e387)

## `Result<T, E>`

Yine bir `enum` türünden oluşturulan `Result<T, E>` ise, olası değer yokluğunu `None` varyantına sarıp döndüren `Option`'a benzemekle beraber, olası hata değerini tanımlayan daha kesin ve zengin bir sürümdür.

Result<T, E> İki varyant durumundan birine sahip olabilir:

* **Ok(T):** T türünde sonuç değerini tutan bir varyant ile,
* **Err(E):** E türünde hata değerini tutan başka bir varyant içermektedir. 

Daha önce gördüğümüz gibi `Option<T>` sadece `Some<T>` ya da `None` içine sarmalanmış değer bilgisini döndürebiliyorken, `Result<T, E>` ise Option türünde bulunmayan hata hakkında bilgi de içerebilmektedir.

Aşağıdaki `serde_json` sandığından kopyaladığım ve JSON belgesi ayrıştırarak Result türü döndüren işlevi inceleyelim. Bu işlevin imzası aşağıdaki gibidir:

```rust
pub fn from_str<'a, T>(s: &'a str) -> Result<T, Error> 
where
   T: Deserialize<'a>,
````

Ve aşağıdaki gibi String türünde bir veri setini ayrıştırmak istediğimizi varsayalım:

```rust
let json_veri = r#"
        {
            "isim": "John Doe",
            "yas": 43,
            "tel": [
                "+90 1234567",
                "+90 2345678"
            ]
        }"#;
````

Bu kişi nesnesini oluşturabilmek için kullanacağımız yapı ise aşağıdakine benzeyecektir:

```rust
#[derive(Serialize, Deserialize)]
struct Kisi {
    isim: String,
    yas: u8,
    tel: Vec<String>,
}
````

Önceki örnekte yer alan `json_veri` öğesini `Kisi` nesnesine ayrıştıran kod ise aşağıdaki gibidir:

```rust
let k: Kisi = match serde_json::from_str(json_veri) {
        Ok(k) => k,
        Err(e) => // Şimdi burada neler olacağını tartışacağız
    };
````

Bu programın başarıyla çalışlacağı açıktır. Fakat `json_veri` girişinde bir yazım hatası olması durumunda, `match` eşlemesi program akışını `Err(e)` koluna yönlendireceğinden böyle bir durumda yapabileceğimiz yalnızca iki şey vardır:

  1. **panic!** üretmek,
  2. **Err()** içine sarılı bir hata bilgisi döndürmek.

1- Örneğimize geri dönelim. **panic!** üretmeyi tercih ettiğimizde kodumuz aşağıdakine benzeyecektir:

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Kisi {
    isim: String,
    yas: u8,
    tel: Vec<String>,
}

fn bir_ornek() -> Result<()> {
    // yas2 bilerek hatalı veriliyor
    let json_veri = r#"
        {
            "isim": "John Doe",
            "yas2": 43,
            "tel": [
                "+90 1234567",
                "+90 2345678"
            ]
        }"#;

    let k: Kisi = match serde_json::from_str(json_veri) {
        Ok(k) => k,
        Err(e) => panic!("JSON ayrıştırması başarısız {:?}", e),
    };

    println!("Lütfen bu numaradan {}, {} adlı çalışanımızı arayın. ", k.tel[0], k.isim);

    Ok(())
}

fn main() {
    match bir_ornek() {
        Ok(_) => println!("Program başarıyla çalışıyor"),
        Err(_) => println!("Program hatalı çalışıyor"),
    }
}
// thread 'main' panicked at 'JSON ayrıştırması başarısız 
// Error("missing field `yas`", line: 9, column: 9)', src/main.rs:25:19
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6bf6a5dfed453656c86b170171fc3dbe)

2- Tercihimiz hata bildirmek olacaksa ayrıştırma ve eşleme işleminden dönen hata değerini işlememiz gerekiyor:

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Kisi {
    isim: String,
    yas: u8,
    tel: Vec<String>,
}

fn bir_ornek() -> Result<()> {
    // yas2 bilerek hatalı veriliyor
    let json_veri = r#"
        {
            "isim": "John Doe",
            "yas2": 43,
            "tel": [
                "+90 1234567",
                "+90 2345678"
            ]
        }"#;

    let k: Kisi = match serde_json::from_str(json_veri) {
        Ok(k) => k,
        Err(e) => return Err(e.into()), // Değişen kısım sadece burası
    };

    println!("Lütfen bu numaradan {}, {} adlı çalışanımızı arayın. ", k.tel[0], k.isim);

    Ok(())
}

fn main() {
    match bir_ornek() {
        Ok(_) => println!("Program başarıyla çalışıyor"),
        Err(_) => println!("Program hatalı çalışıyor"),
    }
}
// Program hatalı çalışıyor
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6903f9313e9b6b67aaf76b7adf38f2d9)

## `unwrap()`

Önceki, panik üretmeyi tercih ettiğimiz senaryoyu şu şekilde yeniden yazabiliriz:

```rust
let k: Kisi = match serde_json::from_str(json_veri) {
        Ok(k) => k,
        Err(e) => panic!("JSON ayrıştırması başarısız {:?}", e), // Panik üretir
};
````

Ancak bu kod oluşacak başarılı durumları kesinlikle beklendiği gibi değerlendirecektir. Fakat bir hata oluşması durumunda `Err` varyantından *panic!* döndürüleceğinden programın çalıştırılmasına kesintiye uğreyacaktır. Şimdi bu kodu daha basiti olan aşağıdaki kod ile değiştirelim:

```rust
let k: Kisi = serde_json::from_str(json_veri).unwrap();
````

Hata olasılığı karşısında `Err` kolunda üretilecek panic! sayesinde programın işletilmesi durdurulacağından, `json_veri` öğesinin `Kisi` nesnesine her zaman ve sorunsuz olarak dönüştürüleceğinden emin olduğumuz hallerde, `unwrap()` metodundan yararlanabiliriz. Bu metod geliştirme sürecimizde programın genel akışı hakkında bize hızlıca bilgi sağlayacağından oldukça kullanışlı bir prototiptir. 

Başka bir ifadeyle aslında `unwrap()` metodu örtük şekilde gerçekleştirilen *panic!* işleminden başka bir şey değildir. Ve daha ayrıntılı olarak tasarladığımız önceki kodla arasında bir fark olmamasına rağmen, bazı durumlarda  `unwrap()` sürümündeki *panic!* tehlikesi örtük biçimde gerçekleşeceğinden, program akışını kavramanız açısından açık versiyonu kullanmanız ayrıntılı sonuçlar üretecektir. 

Eğer programımızı *panic!* üretecek şekilde güncellemek istersek kodumuz aşağıdaki gibi görünecektir:  

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Kisi {
    isim: String,
    yas: u8,
    tel: Vec<String>,
}

fn bir_ornek() -> Result<()> {
    // yas2 bilerek hatalı veriliyor
    let json_veri = r#"
        {
            "isim": "Mert Ataol",
            "yas2": 43,
            "tel": [
                "+90 1234567",
                "+90 2345678"
            ]
        }"#;

    let k: Kisi = serde_json::from_str(json_veri).unwrap();
    
    println!("Lütfen bu numaradan {}, {} adlı çalışanımızı arayın. ", k.tel[0], k.isim);

    Ok(())
}

fn main() {
    match bir_ornek() {
        Ok(_) => println!("Program başarıyla çalışıyor"),
        Err(_) => println!("Program hatalı çalışıyor"),
    }
}
// thread 'main' panicked at 'called `Result::unwrap()` on an `Err`
// value: Error("missing field `yas`", line: 9, column: 9)', src/main.rs:23:19
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=2ffb28aae53eb89f3829358bc3625847)

## `?` İşleci

Her hata kurtarılamaz durumda olamayacağından bazı hatalar ile karşılaşıldığında *panic!* üretmek yerine bir hata döndürmek isteyebilirsiniz. Bu konuyu açıklamak için, hata ile sonuçlandırdığımız kodu yeniden değerlendirelim:

```rust
let k: Kisi = match serde_json::from_str(json_veri) {
        Ok(k) => k,
        Err(e) => return Err(e.into()),
};
````

Yukarıdaki `Err` ifadesinin yer aldığı satırı, tıpkı programımızın *panic!* sürümünde `unwrap()` metodu için yaptığımız gibi, `Err` için de `?` işlecini kullanarak  kısaltabiliriz:

```rust
let k: Kisi = serde_json::from_str(json_veri)?;
````

Programın çalışan sürümü aşağıda yer almaktadır:

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Kisi {
    isim: String,
    yas: u8,
    tel: Vec<String>,
}

fn bir_ornek() -> Result<()> {
    // yas2 bilerek hatalı veriliyor
    let json_veri = r#"
        {
            "isim": "Mert Ataol",
            "yas2": 43,
            "tel": [
                "+90 1234567",
                "+90 2345678"
            ]
        }"#;

    let k: Kisi = serde_json::from_str(json_veri)?;
    
    println!("Lütfen bu numaradan {}, {} adlı çalışanımızı arayın. ", k.tel[0], k.isim);

    Ok(())
}

fn main() {
    match bir_ornek() {
        Ok(_) => println!("Program başarıyla çalışıyor"),
        Err(_) => println!("Program hatalı çalışıyor"),
    }
}
// Program hatalı çalışıyor
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=a2537e6041e5bb1b3e8f47e32200c434)

>Çevirenin notu:
>Eğer bir option veya result türünden panic! döndürüldüğünden eminsek, o türün varyantlarında olan bilgiyi görebilmek için unwrap metodunu, 
>err ile bir hata açıklaması döndürüldüğünden eminsek ? işlecini kullanmamız kısa kod yazmamızı sağlıyor.

## Bir `Option<T>` türünü `unwrap()` ve `?` ile Açmak

`unwrap()` metodu ve `?` işlecini `Result<T, E>` ile kullanabileceğimiz gibi, bir `Option<T>` türü ile de kullanabiliriz. Eğer gerçek değeri `None` olan bir `Option<T>` türünü `unwrap()` kullanarak açmaya kalkışırsanız programınız panik üretecektir:

```rust
fn sonraki_yas_gunu(simdiki_yas: Option<u8>) -> Option<String> {
	// Eğer `current_age`, `None` değerindeyse, `None` döndürülür.
	// Eğer `current_age`  `Some` değerindeyse, 
	// `u8` türü değerine 1 eklenerek `yeni_yas` olusturulur
    let yeni_yas: u8 = simdiki_yas?;
    Some(format!("Gelecek yıl {} yaşımda olacağım:-)", yeni_yas + 1))
}

fn main() {
  let s = sonraki_yas_gunu(None); // Some(35) denendiğinde 36
  match s {
      Some(a) => println!("{:#?}", a),
      None => println!("Sonraki yaş günü bilinmiyor!")
  }
}
// Sonraki yaş günü bilinmiyor!
````
[Rust oyun alanında dene](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=1e4aab08a096a07de2d169815b57c068)

Aynı şekilde, bir `Option<T>` türünü `?` işleci ile de açabiliriz. Eğer döndürülen değer `None` ise, program yürütülmekte olan işlevi sonlandırarak bu değeri döndürecektir.
