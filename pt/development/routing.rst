Roteamento
##########

.. php:namespace:: Cake\Routing

.. php:class:: Router

O Roteamento fornece ferramentas que mapeiam URLs para ações de controlador. Ao definir rotas, você pode separar como seu
aplicativo é implementado e de como seus URLs são estruturados.

O roteamento em CakePHP também engloba a idéia de roteamento reverso, onde uma matriz de parâmetros pode ser transformada em
uma string de URL. Usando roteamento reverso, você pode rafatorar a estrutura do URL da sua aplicação sem ter que atualizar
todo o seu código.

.. index:: routes.php

Visita rápida
=============

Esta seção irá ensinar-lhe por exemplo os usos mais comuns do roteamento no CakePHP. Normalmente, você deseja exibir algo como
uma página de destino, para que você adicione isso ao seu arquivo **routes.php**::

    use Cake\Routing\Router;

    // Usando o construtor de rota com escopo.
    Router::scope('/', function ($routes) {
        $routes->connect('/', ['controller' => 'Articles', 'action' => 'index']);
    });

    // Usando o método estático.
    Router::connect('/', ['controller' => 'Articles', 'action' => 'index']);

O ``Router`` fornece duas interfaces para conectar rotas. O método estático é uma interface compatível com versões anteriores,
enquanto os construtores com escopo oferecem sintaxe mais concisa ao construir várias rotas e melhor desempenho.

Isso executará o método de índice no ``ArticlesController`` quando a página inicial do seu site for visitada. Às vezes você
precisa de rotas dinâmicas que aceitarão vários parâmetros, este seria o caso, por exemplo de uma rota para ver o conteúdo de
um artigo::

    Router::connect('/articles/*', ['controller' => 'Articles', 'action' => 'view']);

A rota acima aceitará qualquer URL que pareça ``/articles/15`` e invoque o método ``view(15)`` no ``ArticlesController``.
Isso não impedirá, porém, que as pessoas tentem acessar URLs que parecem ``/articles/foobar``. Se desejar, você pode
restringir alguns parâmetros para se adequar a uma expressão regular::

    Router::connect(
        '/articles/:id',
        ['controller' => 'Articles', 'action' => 'view'],
        ['id' => '\d+', 'pass' => ['id']]
    );

O exemplo anterior alterou o marcador de estrela por um novo marcador de posição ``:id``. O uso de espaços reservados nos
permite validar partes do URL, neste caso usamos a expressão regular ``\d+`` para que apenas os dígitos sejam correspondidos.
Finalmente, dissemos ao roteador que tratasse o marcador de posição ``id`` como um argumento de função para a função
``view()``, especificando a opção ``pass``. Mais sobre como usar essa opção mais tarde.

O roteador CakePHP também pode inverter rotas de correspondência. Isso significa que a partir de uma matriz contendo
parâmetros correspondentes, é capaz de gerar uma string URL::

    use Cake\Routing\Router;

    echo Router::url(['controller' => 'Articles', 'action' => 'view', 'id' => 15]);
    // Saída
    /articles/15

As rotas também podem ser rotuladas com um nome exclusivo, o que permite que você as faça referência rapidamente ao criar
links em vez de especificar cada um dos parâmetros de roteamento::

    use Cake\Routing\Router;

    Router::connect(
        '/login',
        ['controller' => 'Users', 'action' => 'login'],
        ['_name' => 'login']
    );

    echo Router::url(['_name' => 'login']);
    // Saída
    /login
Para ajudar a manter seu código de roteamento DRY, o roteador tem o conceito de 'escopos'. Um escopo define um segmento de
caminho comum e, opcionalmente, padrões de rota. Quaisquer rotas conectadas dentro de um escopo herdarão o path/defaults de
seus escopos de envolvimento::

    Router::scope('/blog', ['plugin' => 'Blog'], function ($routes) {
        $routes->connect('/', ['controller' => 'Articles']);
    });

A rota acima corresponderia ``/blog/`` e enviá-la-á para ``Blog\Controller\ArtigosController::index()``.

O esqueleto da aplicação vem com algumas rotas para você começar. Depois de adicionar as suas próprias rotas, pode remover as
rotas predefinidas caso não as necessite.

.. index:: :controller, :action, :plugin
.. index:: greedy star, trailing star
.. _connecting-routes:
.. _routes-configuration:

Conexão de rotas
================
