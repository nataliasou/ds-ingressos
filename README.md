## Desafio de Design de Sistemas

Imagine que você é o(a) arquiteto(a) responsável pelo desenho de um sistema de venda de ingressos para um site extremamente concorrido de ingressos de um mega show de Rock em Rio (got it?). Dado que o número de ingressos é limitado e muito inferior a quantidade de acessos no dia da venda, você precisa garantir que o sistema proposto vai conseguir lidar com a grande quantidade de acessos simultâneos o site só irá finalizar a venda para pessoas que realmente vão receber o ingresso, ou seja, você não pode deixar uma pessoa comprar um ingresso sem que haja mais disponíveis. Além disso, um cliente com internet mais lenta não ficaria feliz de não conseguir comprar seu ingresso pois uma pessoa com internet mais veloz passou sua frente.

Desenhe um diagrama de arquitetura de software que sustente esse serviço indepentende da demanda, pode ser um desenho simples, da maneira que você preferir, desde que o mesmo transmita a ideia da arquitetura que você teve para quem o lê.

## Requisitos extraídos

-> Número de ingressos limitados

-> Não deixar o usuário comprar o ingresso sem ter mais disponível

-> Lidar com a alta quantidade de acessos simultâneos

-> Não deve ser priorizado quem tem um internet mais veloz (ou seja, quem consegue finalizar a compra mais rápido)

## Fluxo desenhado

O fluxo começa com a inclusão de um **CDN**, para distribuir conteúdos estáticos e reduzir a latência. Em seguida, as requisições passam pelo **load balancer**, responsável por distribuir o tráfego entre as instâncias disponíveis do **serviço de fila virtual**. Esse serviço gerencia a sala de espera, atribuindo um token único a cada usuário para controlar sua posição e permitir o acesso à etapa de compra de forma ordenada. Enquanto aguarda, o usuário recebe atualizações sobre o estado da fila e sobre a disponibilidade dos ingressos. Quando chega sua vez, o token é validado e o usuário é direcionado para o **fluxo de venda**.

No fluxo de venda, o microsserviço responsável solicita ao serviço de disponibilidade a reserva temporária de um ingresso. Se houver estoque disponível, o ingresso é bloqueado por até 10 minutos, permitindo que o usuário finalize a compra durante esse período. Caso o pagamento seja confirmado dentro do prazo, a reserva é efetivada no **banco de dados relacional PostgreSQL**, e o estoque e o estado da reserva são atualizados no **Redis**, que mantém informações temporárias para agilizar as consultas.

Se o usuário não concluir o pagamento dentro dos 10 minutos, a reserva temporária expira e o ingresso volta a ficar disponível para outros compradores. Por fim, quando o estoque de ingressos disponíveis chega a zero, o serviço de disponibilidade identifica o esgotamento e comunica essa informação ao serviço de fila virtual. A fila deixa de admitir usuários para a compra, e os usuários que ainda estão aguardando recebem uma notificação em tempo real informando que os ingressos se esgotaram.

## Diagrama criado

<img width="1197" height="428" alt="ingressos (1)" src="https://github.com/user-attachments/assets/2fef9150-531c-40e3-b03e-75b525bae061" />

