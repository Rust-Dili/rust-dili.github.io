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
