Paginação
##########

.. php:namespace:: Cake\Controller\Component

.. php:class:: PaginatorComponent

Um dos principais obstáculos encontrados no desenvolvimento de aplicações web 
está em projetar uma interface intuitiva para o usuário. Muitas aplicações tendem a
crescer em tamanho e complexidade rapidamente, e designers e programadores acabam 
se perdendo. Eles são incapazes de lidar com a exibição de centenas ou milhares de registros.
Refatoração leva tempo e o desempenho e a satisfação do usuário podem sofrer com isso.

A exibição de um número razoável de registros por página sempre foi uma parte crítica
de toda aplicação e causa muita dor de cabeça para os desenvolvedores. O CakePHP alivia
a carga sobre o desenvolvedor, fornecendo uma maneira rápida e fáil de paginar os dados.

A paginação no CakePHP é oferecida por um componente no controller, responsável pela 
construção das consultas paginadas. Na View, a helper :php:class:`~Cake\\View\\Helper\\PaginatorHelper`
é usada para gerar os links e botões da paginação.


Usando Controller::paginate()
============================

No controlador, nós começaremos definindo as condiçes padrões que a paginação utilizará
na variável ``$paginate``. Estas condições servem como base para as suas consultas de
paginação. Estes são sobrescritos pelos parâmetros passados pela url ``sort``, ``direction``
``limit`` e ``page``. É importante notar que o parâmetro ``order`` deve ser definido em
um array, como abaixo::

    class ArticlesController extends AppController
    {

        public $paginate = [
            'limit' => 25,
            'order' => [
                'Articles.title' => 'asc'
            ]
        ];

        public function initialize()
        {
            parent::initialize();
            $this->loadComponent('Paginator');
        }
    }

Você pode também incluir algumas das opções suportafas pelo 
:php:meth:`~Cake\\ORM\\Table::find()`, tais como ``fields``::

    class ArticlesController extends AppController
    {

        public $paginate = [
            'fields' => ['Articles.id', 'Articles.created'],
            'limit' => 25,
            'order' => [
                'Articles.title' => 'asc'
            ]
        ];

        public function initialize()
        {
            parent::initialize();
            $this->loadComponent('Paginator');
        }
    }


Embora você pode passar a maioria das opções de consulta da propriedade paginate, 
é muitas vezes mais simples agrupar suas opções de paginação em um: 
:ref:`custom-find-methods`. Você pode definir a paginação do finder usando a opção ``finder`` ::

    class ArticlesController extends AppController
    {

        public $paginate = [
            'finder' => 'published',
        ];
    }

Como os métodos de finders personalizados também podem receber opções, 
é assim que você passa as opções em um método de finder personalizado dentro
da propriedade paginate ::

    class ArticlesController extends AppController
    {

        // find articles by tag
        public function tags()
        {
            $tags = $this->request->getParam('pass');

            $customFinderOptions = [
                'tags' => $tags
            ];
            // O método customizado finder  chamado findTagged dentro de ArticlesTable.php
            // Ele deve se parecer com isso
            // public function findTagged(Query $query, array $options) {
            $this->paginate = [
                'finder' => [
                    'tagged' => $customFinderOptions
                ]
            ];
            $articles = $this->paginate($this->Articles);
            $this->set(compact('articles', 'tags'));
        }
    }

Além de definir valores gerais de paginação, você pode definir mais de um 
conjunto de padrões de paginação no controller, basta nomear as chaves do 
array depois do modelo que você deseja configurar ::

    class ArticlesController extends AppController
    {

        public $paginate = [
            'Articles' => [],
            'Authors' => [],
        ];
    }

Uma vez que a propriedade ``$paginate``  for definida, nós podemos
usar o método :php:meth:`~Cake\\Controller\\Controller::paginate()` 
para criar a paginação dos dados e o ``PaginatorHelper`` se ainda não tiver sido 
adicionada. O método paginate do controller irá retornar o resultado da consulta
paginada e definir os metadados da paginação. Você pode acessar os metadados da 
paginação em ``$this->request->getParam('paging')``. Um exemplo mais completo
usando ``paginate()`` seria::

    class ArticlesController extends AppController
    {

        public function index()
        {
            $this->set('articles', $this->paginate());
        }
    }

Por padrão o método ``paginate()`` irá usar o model padrão para um controller.
Você então pode passar o resultado da consulta através de um método find::

     public function index()
     {
        $query = $this->Articles->find('popular')->where(['author_id' => 1]);
        $this->set('articles', $this->paginate($query));
    }

Se quiser paginar um model diferente, você pode fornecer uma consulta para ele,
o próprio objeto table ou seu nome::

    // Using a query
    $comments = $this->paginate($commentsTable->find());

    // Using the model name.
    $comments = $this->paginate('Comments');

    // Using a table object.
    $comments = $this->paginate($commentTable);

Usando o Paginador Diretamente
============================

Se você precisa paginar dados de outro componente você pode querer usar o 
PaginatorComponent diretamente. Ele possui uma API semelhante para o método
do controller::

    $articles = $this->Paginator->paginate($articleTable->find(), $config);

    // Or
    $articles = $this->Paginator->paginate($articleTable, $config);

O primeiro parâmetro deverá ser uma objeto de consulta de um objeto do método find
em um objeto table que você deseja paginar os resultados. Opcionalmente, você pode
passar o objeto table e deixar a query ser construida para você. O segundo parâmtro
deverá ser um array de configurações para a paginação. Este array tem a mesma estrutura
que a propriedade ``$paginate`` no controller. Quando paginamos um objeto ``Query``,
a opção ``finder`` será ignorada. 

Paginando Multiplas Consultas
===========================

Você pode paginar vários models em uma única action do controller usando a opção 
``scope`` em ambas as propriedades ``$paginate`` e na chamada do método ``paginate()``::

    // Paginate property
    public $paginate = [
        'Articles' => ['scope' => 'article'],
        'Tags' => ['scope' => 'tag']
    ];
    
    // In a controller action
    $articles = $this->paginate($this->Articles, ['scope' => 'article']);
    $tags = $this->paginate($this->Tags, ['scope' => 'tag']);
    $this->set(compact('articles', 'tags'));

A opção ``scope`` resultará na busca do escopo nos parâmtros da URL pelo 
``PaginatorComponent``. Por exemplo, a seguinte URL pode ser usada para paginar
as Tags e Articles ao mesmo tempo::

    /dashboard?article[page]=1&tag[page]=3

Veja a seção :ref:`paginator-helper-multiple` para saber como gerar elementos HTML com escopo
e URLs para paginação.

.. versionadded:: 3.3.0
    Paginação Múltipla foi adicionada na 3.3.0

Controle quais campos serão usados para ordenação
======================================

Por padrão, a ordenação pode ser feita em qualque coluna não virtual que a tabela tiver.
Porém, algumas vezes não é o que desejamos pois permite que usuários ordenem colunas
não indexadas, o que pode custar caro para a aplicação. Você pode definir uma lista branca
com os campos que você deseja ordenar, utilizando a opção ``sortWhitelist``. Esta opção é 
requerida quando você quer ordenar qualquer dado associado ou campos computados que podem fazer 
parte da sua consulta de paginação::

    public $paginate = [
        'sortWhitelist' => [
            'id', 'title', 'Users.username', 'created'
        ]
    ];


Quaisquer solicitações que tentam ordenar campos não incluídos na lista, serão
ignoradas.

Limitar o número máximo de linhas por página
=========================================

O número de resultados que são trazidos por página é exposto para o usuário com
o parâmetro ``limit``. Geralmente, não é desejável que o usuário possa trazer todos
os registros em uma única página. A opção ``maxLimit`` declara que ninguém possa definir
um limite superior ao mesmo. Por padrão, o CakePHP limita o número máximo de registros 
em 100 por página. Se o padrão não for apropriado para sua aplicação, você pode ajustar 
nas opções de paginação, por exemplo, reduzindo para ``10``::

    public $paginate = [
        // Other keys here.
        'maxLimit' => 10
    ];

Se o parâmetro limit for superior a este valor, ele será reduzido ao valor de ``maxLimit``.


Associaçes adicionais
===============================

Associações adicionais podem ser carregadas para uma tabela paginada utilizando 
o parâmetro ``contain``::

    public function index()
    {
        $this->paginate = [
            'contain' => ['Authors', 'Comments']
        ];

        $this->set('articles', $this->paginate($this->Articles));
    }

Pedidos de página fora do intervalo
==========================

O PaginatorComponent irá lançar um ``NotFoundException`` quando tentar
acessar uma página não existente, ex: o número da página requisitada é
maior que o número total de páginas.

Assim, você pode deixar a página de erro normal ser processada ou usar um bloco try catch
e tomar a ação apropriada quando um `` NotFoundException`` é chamado ::

    use Cake\Network\Exception\NotFoundException;

    public function index()
    {
        try {
            $this->paginate();
        } catch (NotFoundException $e) {
            //  Faz algo aqui, como redirecionar para a primeira ou última página
            // $this->request->getParam('paging') will give you required info.
        }
    }

Paginação na View
======================

Verifique a documentação de :php:class:`~Cake\\View\\Helper\\PaginatorHelper` para criar 
links de navegação paga a paginação
how to create links for pagination navigation.


.. meta::
    :title lang=pt: Paginação
    :keywords lang=pt: ordenação, complexidade, parâmetros, paginação, cakephp, desenvolvedores, condições, web, aplicações, vetores
