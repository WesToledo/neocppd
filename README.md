# NeoCPPD — Wiki simplificada

Sistema de gerenciamento para a Comissão Permanente de Pessoal Docente (CPPD) do IFSULDEMINAS. Reúne um portal público e código para cadastros administrativos, gestão de servidores e acompanhamento de avaliações e progressões.

O repositório combina uma base CodeIgniter 4 com controladores legados de CodeIgniter 3. A presença de um módulo no código não garante que ele esteja funcionando na versão atual.

## Navegação

- [Conhecer os módulos](#conhecer-os-módulos)
- [Encontrar os arquivos](#encontrar-os-arquivos)
- [Preparar o ambiente local](#preparar-o-ambiente-local)
- [Entender o acesso](#entender-o-acesso)
- [Verificar e resolver problemas](#verificar-e-resolver-problemas)

## Conhecer os módulos

| Área | Conteúdo encontrado | Código principal |
| --- | --- | --- |
| Portal público | Informações da CPPD, documentos, tabelas, progressão, afastamento, normativa e RSC | `app/Controllers/Home.php`, `app/Views/site/` |
| Autenticação | Login com e-mail e senha ou conta Google; encerramento da sessão | `app/Controllers/Login.php` |
| Administração | Painel, campi, cursos, configurações e vínculo entre cursos e campi | `app/Controllers/Restrito/administracao/` |
| Recursos humanos | Servidores, setores, carreira, classes, níveis e titulações | `app/Controllers/Restrito/rh/` |
| Avaliação | Requerimentos, processos, progressões, critérios, pontos e avaliações docente, discente e da chefia | `app/Controllers/Restrito/avaliacao/` |
| Controle de acesso | Usuários, papéis, privilégios, menus e cadastro de controladores | `app/Controllers/Restrito/uac/` |

Campi e Setor são exemplos de controladores com estrutura CodeIgniter 4. Diversos outros arquivos, especialmente os de avaliação, ainda usam `CI_Controller`, `$this->load` e APIs do CodeIgniter 3.

## Encontrar os arquivos

```text
app/
├── Config/        # Rotas, filtros, banco, sessão e configurações
├── Controllers/   # Portal, login e módulos restritos
├── Database/      # Migrations e seeds da aplicação ainda sem implementação
├── Filters/       # Verificação de sessão da área restrita
├── Language/      # Traduções, incluindo pt-br
├── Libraries/     # Grocery CRUD e montagem dos menus
├── Models/        # Consultas de usuários, menus e suporte ao CRUD
└── Views/         # Páginas, layout administrativo e telas de erro
public/            # Entrada web e arquivos estáticos
tests/             # Testes de exemplo do starter
writable/          # Arquivos gerados em execução
composer.json      # Dependências PHP e comando de testes
composer.lock      # Versões fixadas das dependências
spark              # Interface de linha de comando do CodeIgniter
```

Os controllers recebem as requisições, os models consultam o banco e as views montam as páginas. O Grocery CRUD gera telas de cadastro; `app/Views/crud/index.php` exibe seu resultado dentro de `app/Views/template/admin/layout.php`.

Há templates e recursos com nomes `old`, além de cópias e exemplos. Confira as referências antes de usar esses arquivos como base para mudanças.

## Preparar o ambiente local

### Requisitos e dependências

- PHP compatível com `^8.1`, conforme `composer.json`, com extensões `intl`, `mbstring` e suporte a MySQLi/mysqlnd.
- Composer para instalar as dependências PHP.
- MySQL e acesso ao esquema e aos dados iniciais da aplicação.
- Credenciais OAuth do Google para utilizar esse tipo de login.

O `composer.lock` fixa CodeIgniter 4.5.5. O projeto também declara traduções do CodeIgniter, Google API Client e PHPUnit 10. O Grocery CRUD está incluído no próprio código, e o painel utiliza recursos AdminLTE.

O `package.json` encontrado declara a ferramenta Codex; não define comandos de compilação do frontend.

### 1. Instalar as dependências PHP

Na raiz do repositório:

```bash
composer install
```

### 2. Configurar o ambiente

Crie ou ajuste o arquivo `.env` na raiz, preservando configurações locais existentes. Exemplo para desenvolvimento:

```dotenv
CI_ENVIRONMENT = development
app.baseURL = 'http://localhost:8080/'

database.default.hostname = localhost
database.default.database = neocppd
database.default.username = usuario_local
database.default.password = 'substitua-pela-senha-local'
database.default.DBDriver = MySQLi
database.default.port = 3306

ClientId = 'seu-client-id-google'
ClientSecret = 'seu-client-secret-google'
```

Não publique o `.env` com credenciais reais. Os nomes `ClientId` e `ClientSecret` são os lidos pelo controller `Login`.

### 3. Obter o banco da aplicação

Não foi encontrado dump SQL nem migrations/seeds que criem as tabelas da aplicação. Solicite o esquema e os dados iniciais ao responsável pelo projeto; executar migrations sozinho não prepara esse banco.

As consultas dependem, entre outras, das tabelas `usuario`, `servidor`, `papel`, `papel_usuario`, `menu`, `controller` e `privilegio`. O cadastro automático de usuário também consulta `laboratorio` e atribui papéis com IDs fixos 2 e 3. Esses registros precisam corresponder às regras esperadas pelo código.

### 4. Iniciar o servidor

```bash
php spark serve
```

Acesse `http://localhost:8080/`. Para outro servidor web, configure a raiz pública em `public/`. Garanta permissão de escrita em `writable/`.

Esses passos descrevem a preparação necessária; o funcionamento completo depende do banco e da adaptação dos módulos legados.

## Entender o acesso

As configurações de `app/Config/Routes.php` definem `Home::site` como entrada padrão e habilitam roteamento automático. Exemplos de caminhos derivados dos controllers:

| Caminho | Finalidade |
| --- | --- |
| `/` | Portal público |
| `/home/documentos` | Documentos |
| `/home/tabelas` | Tabelas |
| `/login/site` | Tela de login |
| `/login/google` | Retorno do login Google |
| `/login/verificar` | Processamento do login por e-mail e senha |
| `/login/logoff` | Encerramento da sessão |
| `/restrito/administracao/dashboard/site` | Método de exibição do painel |
| `/restrito/administracao/campi` | Cadastro de campi |
| `/restrito/rh/setor` | Cadastro de setores |

As rotas explícitas `/pessoal`, `/crud/(:any)` e `/outra` apontam para controllers de exemplos.

### Login e sessão

O login por senha procura o usuário pelo e-mail do servidor e valida o hash com `password_verify`. No login Google, o e-mail precisa corresponder a um servidor já cadastrado; o sistema pode então criar seu usuário e associar papéis.

Para o Google, configure no cliente OAuth a URI de retorno correspondente à URL base, por exemplo `http://localhost:8080/login/google`.

Após autenticar, `Login` grava os dados em `session('user')`. O filtro `webauth`, aplicado a `restrito/*`, exige a existência dessa chave. Os menus são consultados conforme os papéis e privilégios no banco; essa montagem do menu, por si só, não comprova autorização de cada operação.

## Verificar e resolver problemas

Após instalar as dependências, o comando declarado para testes é:

```bash
composer test
```

Os testes encontrados são exemplos do starter para saúde da aplicação, banco e sessão; não demonstram cobertura dos fluxos de negócio da CPPD. A configuração fica em `phpunit.xml.dist`, com instruções adicionais em `tests/README.md`.

| Sintoma | O que conferir |
| --- | --- |
| Dependências ou framework não encontrados | Execute `composer install` e confira a instalação de PHP e suas extensões. |
| Erro de conexão ou tabela inexistente | Revise o `.env` e obtenha o banco da aplicação. |
| Login Google retorna ao formulário sem liberar acesso | Confira as credenciais OAuth, a URI de retorno e o cadastro do servidor pelo e-mail. |
| Falha após login ao abrir o painel | O login redireciona para `/restrito/administracao/dashboard`, mas `Dashboard` define `site()` e não `index()`; esse alinhamento precisa ser revisado. |
| Erro envolvendo `CI_Controller`, `$this->load` ou `Grocery_CRUD` | O módulo utiliza APIs legadas e precisa de adaptação para CodeIgniter 4. |
| Foto do usuário não é salva | Confira o caminho relativo `assets/uploads/user/` usado por `UsuarioModel` e suas permissões no ambiente de execução. |

O arquivo `AuthController.php` contém outro fluxo de autenticação, mas referencia `UserModel` e uma view `login` que não foram encontrados. O fluxo que utiliza o model existente `UsuarioModel` está em `Login.php`.

Documentação baseada na leitura estática do repositório. A execução da aplicação e dos testes ainda precisa ser validada em um ambiente com PHP, dependências e banco configurados.
