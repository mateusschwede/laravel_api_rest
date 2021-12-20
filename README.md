# API REST com Laravel
<img src="https://ubsocial.github.io/anexos/icons/logoBranco.png" alt="UB Social" width="20"/> <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Laravel.svg/1200px-Laravel.svg.png" alt="Laravel" width="33"/>

[Acesse o post oficial no UB Social](https://ubsocial.github.io/posts/post12.html)

## Conceitos teóricos
- **API**: Application Programming Interface tem função de intermediar comunicação universal entre diferentes sistemas, assim como do banco de dados com o próprio sistema (Interno ou externo), utilizando protocolo de comunicação HTTP (XML/JSON), requisitados via URL específica, independente da tecnologia do sistema que irá consumí-la (Acessá-la). Pode-se solicitar dados via API, através de métodos como POST, assim como a API poderá fornecer-lhe dados, via métodos como GET. Em ambos casos, utilizando a comunicação universal citada.
*Funcionamento*: Front-end requisita, via URL específica, rotinas ao Back-end. Conforme URL, serão realizadas rotinas de código com acesso/manipulação ao Banco de Dados. Após isso, Back-end envia feedback, em formato XML/JSON, para Front-end. Todo o processo descrito realizado no Back-end é o conceito de API.

- **REST** (Representational State Transfer): Técnica que consiste em princípios que, quando seguidos, permitem a criação de projetos (Web services) com interfaces/arquitetura bem definidas, utilizando novos métodos REST HTTP, como PUT e DELETE. Os Web services que estão em conformidade com o estilo arquitetural REST são denominados Web services RESTful. Em síntese, API que intermedia a comunicação do Front-end com o Back-end do sistema, fazendo-os se conversarem por meio de linguagem XML/JSON utilizando métodos REST (Front-end solicita coisas via URL ao Back-end que, por meio de padrões de código e métodos REST (Como DELETE, UPDATE...), cria JSON com resposta e o reenvia ao Front-end, que organizará visualmente o conteúdo ao usuário, técnica chamada 'Consumir API').

- **RESTful**: Nome de categoria dado ao sistema que utiliza API REST em seu funcionamento. (Ex: "O sistema XYZ é RESTful". Portanto, o sistema XYZ funciona por meio de API REST). Para que uma API seja RESTful, precisa seguir os seguintes critérios:
1. Ter arquitetura cliente/servidor formada por clientes, servidores e recursos, com solicitações gerenciadas via HTTP;
2. Realizar comunicação cliente/servidor stateless. Isso significa que cada solicitação é separada e não conectada com outras, e que nenhuma informação do cliente é armazenada entre elas;
3. Armazenar dados em cache para otimizar as interações entre cliente e servidor;
4. Ter interface uniforme entre os componentes para que as informações sejam transferidas em um formato padronizado. Para tanto, é necessário que:
    4.1. Recursos solicitados sejam identificáveis e estejam separados das representações enviadas ao cliente;
    4.2. Recursos possam ser manipulados pelo cliente por meio da representação recebida com informações suficientes para tais ações;
    4.3. Mensagens autodescritivas retornadas ao cliente contenham informações suficientes para descrever como processá-las;
    4.4. Hipermídia seja incorporada. Isso significa que após acessar um recurso, o cliente precisa ser capaz de usar hiperlinks para encontrar todas as demais ações disponíveis para ele no momento.
5. Ter um sistema em camadas que organiza os tipos de servidores (responsáveis pela segurança, pelo carregamento de carga e assim por diante) envolvidos na recuperação das informações solicitadas em hierarquias que o cliente não pode ver;
6. Possibilitar código sob demanda (opcional): a capacidade de enviar do servidor para o cliente um código executável quando solicitado, a fim de ampliar as funções do cliente.

## Funcionamento na prática
Aplicação Front-end envia requisição HTTP com métodos REST para Back-end, que, de acordo com a URL, realizará determinadas rotinas e gerará um JSON com os resultados da requisição. Essa requisição, geralmente, será reenviada ao Front-end, que o 'consumirá' em um layout interativo ao usuário.
*Exemplo consultar dados de cliente*: Usuário clica em Consultar Cliente, que enviará URL requisitando dados do cliente ao Back-end. Conforme parâmetros na URL requisitada (ID do usuário, URL via método GET), com isso, o Back-end, por meio da API REST (Métodos e padrões de código) com Laravel, consultará BD e criará JSON com resultado dos dados do cliente solicitado e o enviará de volta ao Front-end, que lerá o JSON (Consumir API REST) com React, mostrando os dados do cliente em interface gráfica agradável para o usuário.


## Pré-instalações
1. Instalação e configuração do Laravel (Ver resumo Laravel aqui no UB Social)
2. Instalar Postman:
    2.1. Baixar, no site oficial, Postman Client (.tar.xz) e extraí-lo
    2.2. No diretório do Postman, realizar os comandos: sudo mv Postman /opt && sudo ln -s /opt/Postman/Postman /usr/local/bin/postman
    2.3. Criar lançador desktop para fácil acesso: sudo nano /usr/share/applications/postman.desktop
(Conteúdo dentro de poastman.desktop)
[Desktop Entry]
Type=Application
Name=Postman
Icon=/opt/Postman/app/resources/app/assets/icon.png
Exec="/opt/Postman/Postman"
Comment=Postman GUI
Categories=Development;Code;
    2.4. O Postman também pode ser acessado informando 'postman' no shell
3. Criar BD: Iniciar Apache e MySQL XAMPP e, no Phpmyadmin, criar BD: 'CREATE DATABASE artigos;'
4. Criar projeto Laravel: 'composer create-project --prefer-dist laravel/laravel laravelApiRest'
5. Configurar BD em .env: 'DB_DATABASE=artigos'


## Passo a passo
1. Criar Model, migration e Controller:
php artisan make:model Artigo -m
php artisan make:controller ArtigoController --resource

2. Configurar Migration create_artigos_table.php (Após código, realizar comando 'php artisan migrate')
~~~php
<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateArtigosTable extends Migration {

    public function up() {
        Schema::create('artigos', function (Blueprint $table) {
            $table->id();
            $table->string('titulo');
            $table->text('conteudo');
            $table->timestamps();
        });
    }

    public function down() {
        Schema::dropIfExists('artigos');
    }
}
~~~

3. Definir Controller ArtigoController
~~~php
<?php
namespace App\Http\Controllers;
use App\Models\Artigo as Artigo;
use App\Http\Resources\Artigo as ArtigoResource;
use Illuminate\Http\Request;

class ArtigoController extends Controller {

    public function index() {
        $artigos = Artigo::paginate(15);
        return ArtigoResource::collection($artigos);
    }

    public function show($id) {
        $artigo = Artigo::findOrFail( $id );
        return new ArtigoResource($artigo);
    }

    public function store(Request $request) {
        $artigo = new Artigo;
        $artigo->titulo = $request->input('titulo');
        $artigo->conteudo = $request->input('conteudo');

        if($artigo->save()) {
            return new ArtigoResource($artigo);
        }
    }

    public function update(Request $request) {
        $artigo = Artigo::findOrFail($request->id);
        $artigo->titulo = $request->input('titulo');
        $artigo->conteudo = $request->input('conteudo');

        if( artigo->save()) {
            return new ArtigoResource($artigo);
        }
    } 

    public function destroy($id) {
        $artigo = Artigo::findOrFail($id);
        if($artigo->delete()) {
            return new ArtigoResource($artigo);
        }
    }
}
~~~

4. Criar Routes em routes/api.php
~~~php
<?php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ArtigoController;

Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});

Route::get('artigos',[ArtigoController::class,'index']); //Listar artigos
Route::get('artigo/{id}',[ArtigoController::class,'show']); //Ver artigo
Route::post('artigo',[ArtigoController::class,'store']); //Criar artigo
Route::put('artigo/{id}',[ArtigoController::class,'update']); //Editar artigo
Route::delete('artigo/{id}',[ArtigoController::class,'destroy']); //Excluir artigo
~~~

5. Criar Resource em app/Http/Resources/Artigo.php: 'php artisan make:resource Artigo'
~~~php
<?php
namespace App\Http\Resources;
use Illuminate\Http\Resources\Json\JsonResource;

class Artigo extends JsonResource {

    public function toArray($request){
        //return parent::toArray($request);
        return [
            'id' => $this->id,
            'titulo' => $this->titulo,
            'conteudo' => $this->conteudo
        ];
    }

    /* public function with( $request ){
        return [
            'version' => '1.0.0',
            'author_url' => url('https://ubsocial.github.io')
        ];
    } */
}
~~~

6. Executar projeto Laravel: 'php artisan serve' (http://127.0.0.1:8000)
7. No Postman, testar API para inserir artigo: (Deverá retornar JSON e inserir no BD)
POST: http://127.0.0.1:8000/api/artigo
(Headers)
Key: Content-Type
Value: application/json
(Body raw) (Enviar 1 por vez)
~~~json
{
    "titulo": "Primeiro artigo pelo Postman",
    "conteudo": "Descrição desse artigo de número um."
}

{
    "titulo": "Segundo artigo pelo Postman",
    "conteudo": "Descrição desse artigo de número dois."
}

{
    "titulo": "Terceiro artigo pelo Postman",
    "conteudo": "Descrição desse artigo de número três."
}
~~~

8. No Postman, testar API para listar artigos: (Mostrará em data no JSON)
GET: http://127.0.0.1:8000/api/artigos

9. No Postman, testar API para ver somente o 1º artigo:
GET: http://127.0.0.1:8000/api/artigo/1

10. No Postman, testar API para editar o artigo 1:
PUT: http://127.0.0.1:8000/api/artigo/1
(Headers)
Key: Content-Type
Value: application/json
(Body raw)
~~~json
{
    "titulo": "ATUALIZANDO Primeiro artigo pelo Postman",
    "conteudo": "Descrição desse artigo de número um sendo ATUALIZADO."
}
~~~

11. No Postman, testar API para remover o artigo 1:
DELETE: http://127.0.0.1:8000/api/artigo/1
