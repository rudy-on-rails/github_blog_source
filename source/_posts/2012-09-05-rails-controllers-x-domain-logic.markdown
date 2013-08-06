---
layout: post
title: "Rails - Controllers x Lógica de domíno"
date: 2012-09-25 20:10
comments: true
categories: [design, domain model, ddd, rails, controller, domain services]
---

Muitas vezes, em um projeto em Rails, nos vemos encorajados a escrever parte da lógica da aplicação em nossos controllers. Talvez isso acontença, principalmente, pelo fato da grande maioria dos exemplos existentes na internet e literatura (de Rails) não mostrar preocupação com a arquitetura da aplicação (até pelo foco nestes casos ser a tecnologia, em si).

Acho que essa falta de precupação com design mais rico se justifica na maioria dos casos, onde a aplicação é pequena, o escopo do projeto é bem limitado ou existe uma necessidade imediata de lançar um produto ao público (como acontece em muitas startups - e nada impede um 'refactoring' posterior, mas isto é outra estória...). Existem, porém, outros casos... Aplicações grandes, complexas ou onde a criticidade das informações é elevada, requerem cuidados maiores com o design da solução (ou não... dependendo de quão suicida o time for!).

Após trabalhar em algumas aplicações grandes escritas em Rails (apps com mais de 15.000 linhas de código), muitas coisas que passariam despercebidas em alguns casos, tornam-se evidentes, principalmente quando modificações são necessárias:  implementação de nova funcionalidade, refactoring ou correção de bugs...

Abaixo segue um exemplo de extração de objetos e lógica de domínio do controller:

```ruby
  class PedidosController < ApplicationController

    def cancelar_pedido
      @pedido  = Pedido.find(params[:id])
      @estoque = Estoque.find_by_loja(params[:loja_id])    
      @pedido.data_cancelamento = Date.today
      @pedido.save
      GatewayDePagamento.estornar_pagamento(@pedido.pagamento) if @pedido.pagamento      
      @estoque.estornar_estoque_por_pedido(@pedido)    
      NotificacaoDeUsuarios.notificar_cancelamento_pedido(@pedido) #classe que herda de ActionMailer::Base
    end
    
    .
    .
    .
  end
```
O principal problema existente neste controller é que ele, enquanto camada de parte da camada de visualização, têm conhecimento de como aplicar uma regra de negócio (além de outros problemas de design, como testabilidade, violação SOLID, enfim, este não é o foco no post).

A 'regra de negócio', enquanto parte de maior importância no sistema, devia estar isolada, independente de como é implementada sua parte "visual".

Imagine, neste cenário, que será necessário fornecer esta mesma funcionalidade via WebService SOAP. É fácil copiar e colar este código no "handle" do request via SOAP, não?!? 
É fácil até que exista um novo requerimento ou que seja necessário uma adição/modificação do que existe.  Por exemplo, surge a necessidade de que pedidos cancelados devem notificar o departamento de vendas... 
2 lugares para modificar, maiores as chances de que algo seja esquecido no caminho, fora os outros problemas anteriormente descritos...

Então, para evitar estes problemas, podemos seguir a seguinte abordagem:

1) Extraímos a lógica de cancelamento e notificação para um objeto ("Domain Service", se quisermos definir assim, segundo concepções do DDD):
```ruby
class CancelamentoDePedidos
  def initialize(pedido, estoque, gateway_pagamento=GatewayDePagamento)
    @pedido = pedido
    @estoque = estoque    
    @gateway_pagamento = gateway_pagamento
  end

  def proceder_com_cancelamento
    #os dois métodos abaixo em Pedido serão descritos logo mais
    @pedido.cancelar 
    @pedido.estornar_pagamento_usando(@gateway_pagamento)
    @estoque.estornar_estoque_por_pedido(@pedido)
    # A decisão de manter este código no serviço vai depender de 
    # quão importante é para o caso de uso de "Cancelar Pedido"
    # esta notificação. Neste caso, trata-se como parte fundamental 
    # dele
    NotificacaoDeUsuarios.notificar_cancelamento_pedido(@pedido) 
  end
end
``` 
2) Adicionamos a manipulação dos atributos internos relativos ao cancelamento de pedidos ao próprio pedido, que deveria ter este conhecimento.:
```ruby
class Pedido    
  .
  .
  .
  def cancelar
    self.data_cancelamento = Date.today
    self.save  
  end

  def estornar_pagamento_usando(gateway_pagamento)
    gateway_pagamento.estornar_pagamento(self.pagamento) if esta_pago?
  end

  def esta_pago?
    !pagamento.nil?
  end
end
``` 

3) Por último, modificamos nosso controller:
```ruby
class PedidosController < ApplicationController

  def cancelar_pedido
    @pedido  = Pedido.find(params[:id])
    @estoque = Estoque.find_by_loja(params[:loja_id])    
    CancelamentoDePedidos.new(@pedido, @estoque).proceder_com_cancelamento
  end  
  
  .
  .
  .
end
```

Acho que a idéia é mais ou menos essa... Com esse pequeno (e fácil!) refactoring do controller, explicitamos o caso de uso mencioando e nosso controller passa a ser somente um consumidor de nossa API para cancelamento de pedidos!!! Parece razoável (apesar de ainda ser possível melhorar isso)...

O livro do Eric Evans - "Domain Driven Design - tackling complexity in the heart of software" é, na minha opinião, um livro um pouco difícil de ser lido e "digerido", porém consegue fornecer um insumo suficiente para desenvolver software com um "mindset" que prima a representação em código do negócio de nossos clientes.