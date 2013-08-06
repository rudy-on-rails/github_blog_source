---
layout: post
title: "Mock e Stubs - Afinal, o que é isso e quando devo usar?"
date: 2013-08-06 15:23
comments: true
published: false
categories: [tdd, mocks, stubs]
---

Diversas vezes, no dia-a-dia de trabalho, vejo alguns colegas com dificuldades em testar objetos que têm dependências externas e/ou de outros objetos.

Imaginando o cenário abaixo, onde queremos ler dados do facebook e associá-los ao nosso usuário:

```ruby
  class FacebookApi
  	def	intitialize
  		#algumas coisas da conexao ao facebook aqui
  	end

  	def	read_profile_info(facebook_user_id)
  		# simplesmente retorna um hash (criado à partir do json) contendo as propriedades do perfil
  		# ou lança uma FacebookConnectionError, caso algo dê errado
  	end
  end

  class FacebookProfileInformationReader
    def initialize(user)
    	@user = user
    end

    def read_profile_information
      retrieved_profile_info = FacebookApi.new.read_profile_info(user.facebook_id)
      user.photo_url = retrieved_profile_info[:profile_image]
      user.email = retrieved_profile_info[:email]
      user.name = retrieved_profile_info[:user_name]
      user.save!
    end
  end
```
