---
layout: post
title: "Usando o 'assets pipeline' + compressão de Javascript com AngularJS"
date: 2012-09-20 12:55
comments: true
categories: [assets, rails, angularjs, asset pipeline]
---
Se você, como eu, está recebendo o erro 
```
Unknown provider: eProvider <- e
```
ao fazer deploy de uma aplicação Rails usando Asset Pipeline + AngularJS, então muito provavelmente seus arquivo de configurações referente ao ambiente onde o deploy está sendo realizado contém a linha:
``` ruby
config.assets.js_compressor = :uglifier
```

Este erro acontece porquê a obfuscação que ocorre no processo de obfuscação usando o uglifier impede o AngularJS de realizar a identificação interna de DI (dependency injection).

Para corrigir o problema, é simples. Basta trocar a linha mencionada acima por:

``` ruby
config.assets.js_compressor = Sprockets::LazyCompressor.new { Uglifier.new(:mangle => false) }
```
Isso vai impedir o obfuscador do unglifier de atuar sob nomes de variáveis e irá corrigir o problema.