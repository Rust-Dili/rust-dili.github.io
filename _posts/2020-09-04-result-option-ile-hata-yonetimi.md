---
layout: post
title: "Rust'ta Result, Option, unwrap ve ? İşleci ile Hata Yönetimi"
date: 2020-09-04
excerpt: "Rust öğrenirken, Result, Option, unwrap ve ? işlecinden oluşan bu dört kavramın genellikle birbirleriyle birlikte tartışıldığını fark ettim. Bu yazıda, Rust’ı anlamamı ve daha deyimsel olmamı sağlayan bu kavramlar üzerindeki düşünce sürecimi belgelemek ve sizlerle paylaşmak istiyorum."
tags: [option, unwrap, result, hata işleme]
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

