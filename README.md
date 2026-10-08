# Sistema de Gestão de Alunos

## 1. Descrição do projeto

Aplicação web para gerenciamento de alunos de uma instituição de ensino.
Um usuário autenticado pode **cadastrar, consultar, alterar e excluir** alunos.
O acesso é protegido por uma tela de login e as rotas da API de alunos são protegidas com Spring Security.

Fluxo da aplicação:

```
Vue.js -> API REST -> Spring Boot -> Spring Security -> Service -> JPA -> MySQL
```

## 2. Tecnologias utilizadas

- Java
- Spring Boot
- Spring Data JPA
- Spring Security (BCrypt para a senha)
- Bean Validation (validações dos dados)
- Vue.js 3 (Vite + Vue Router)
- MySQL
- API REST

## 3. Requisitos para execução

- JDK 22 (versão definida no `pom.xml`)
- Node.js 20.19 ou superior (com npm)
- MySQL em execução em `localhost:3306`
- Não é necessário instalar o Maven (o projeto inclui o Maven Wrapper `mvnw`)

## 4. Configuração do MySQL

1. Execute o script `database/script.sql` no MySQL. Ele cria o banco `sistema_alunos`, as tabelas `usuarios` e `alunos` e o usuário de teste:

   ```
   mysql -u root -p < database/script.sql
   ```

   (ou abra o arquivo no MySQL Workbench e execute)

2. Confira o usuário e a senha do MySQL em `backend/src/main/resources/application.properties`:

   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/sistema_alunos
   spring.datasource.username=root
   spring.datasource.password=
   ```

   Ajuste `username` e `password` se o seu MySQL for diferente.

## 5. Como executar o backend

```
cd backend
./mvnw spring-boot:run
```

No Windows: `mvnw.cmd spring-boot:run`

O backend fica disponível em `http://localhost:8081`.

## 6. Como executar o frontend

```
cd frontend
npm install
npm run dev
```

Acesse `http://localhost:5173`.

> O backend precisa estar rodando. O frontend se comunica com `http://localhost:8081/api`
> (configurado em `frontend/src/services/api.js`) e o backend libera o CORS para `http://localhost:5173`.

## 7. Usuário para teste

| Usuário | Senha  |
|---------|--------|
| admin   | 123456 |

A senha é armazenada no banco somente como hash BCrypt e nunca é enviada para o frontend.

## 8. Endpoints da API

Login e logout:

| Método | Rota          | Descrição                              | Respostas       |
|--------|---------------|----------------------------------------|-----------------|
| POST   | `/api/login`  | Autentica o usuário (cria a sessão)    | 200, 400, 401   |
| POST   | `/api/logout` | Encerra a sessão                       | 204, 401        |

Corpo do login: `{ "username": "admin", "password": "123456" }`

Alunos (**exigem autenticação**; sem login a API responde `401`):

| Método | Rota                | Descrição        | Respostas             |
|--------|---------------------|------------------|-----------------------|
| POST   | `/api/alunos`       | Criar aluno      | 201, 400, 401, 409    |
| GET    | `/api/alunos`       | Listar alunos    | 200, 401              |
| GET    | `/api/alunos/{id}`  | Buscar por ID    | 200, 401, 404         |
| PUT    | `/api/alunos/{id}`  | Atualizar aluno  | 200, 400, 401, 404, 409 |
| DELETE | `/api/alunos/{id}`  | Excluir aluno    | 204, 401, 404         |

Exemplo de corpo (POST / PUT):

```json
{
  "nome": "Maria Silva",
  "email": "maria@email.com",
  "cpf": "123.456.789-00",
  "curso": "Desenvolvimento de Sistemas",
  "idade": 20,
  "ativo": true
}
```

Validações:

- `nome`: obrigatório
- `email`: obrigatório, formato válido e único
- `cpf`: obrigatório e único
- `curso`: obrigatório
- `idade`: entre 16 e 100
- `ativo`: obrigatório
- `dataCadastro`: preenchida automaticamente

Formato das respostas de erro:

```json
{
  "mensagem": "Dados inválidos.",
  "erros": { "idade": "A idade deve estar entre 16 e 100 anos." }
}
```

Códigos usados: `200 OK`, `201 Created`, `204 No Content`, `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `409 Conflict`.

## Estrutura do projeto

```
sistema-alunos
├── backend
│   └── src/main/java/com/example/demo
│       ├── controller   (AlunoController, AuthController)
│       ├── service      (AlunoService)
│       ├── repository   (AlunoRepository, UsuarioRepository)
│       ├── model        (Aluno, Usuario)
│       ├── dto          (AlunoRequestDTO, AlunoResponseDTO, ...)
│       ├── security     (SecurityConfig, UsuarioDetailsService, ...)
│       └── exception    (exceções e GlobalExceptionHandler)
├── frontend
│   └── src
│       ├── views        (Login, Alunos, Cadastro, Edição)
│       ├── components   (AlunoForm)
│       ├── router
│       └── services     (api.js)
├── database
│   └── script.sql
└── README.md
```
