# Nível 02

## Backend com Node.js e TypeScript

Na primeira parte deste módulo, criamos um projeto Node e configuramos o TypeScript, o EditorConfig, o ESLint, Prettier e o debug do VSCode.

### Typescript
Para configurar o TypeScript, instalamos e iniciamos o tsc. Depois disso, alteramos apenas 2 configurações no [tsconfig.json](/tsconfig.json):
- `"outDir": "./dist",` para que o código transpilado seja todo salvo na pasta `dist` na raíz do projeto;
- `"rootDir": "./src",` para que a o compilador entenda que código desenvolvido da aplicação começa a partir da pasta `src`.

### EditorConfig
Essa extensão do VSCode permite a criação de um arquivo `.editorconfig` que rege algumas configurações de escopo do projeto, sobrescrevendo os padrões definidos pelo usuário. Assim, evita conflitos de IDEs ou editores de textos de diferentes desenvolvedores trabalhando no mesmo projeto.

### ESLint
O ESLint verifica possíveis erros e indica melhores práticas no desenvolvimento. Neste projeto, ele foi configurado para, também, forçar um estilo de código, o que mantém um padrão de desenvolvimento único no projeto, independente de como cada desenvolvedor individualmente prefira escrever o seu código. Se algum dev não quiser colocar `;` no final de uma sentença, o ESLint se encarrega de colocar automaticamente ao salvar o arquivo.

Para isso, nas configurações do VSCode foi adicionada a opção:
```
"editor.codeActionsOnSave": {
  "source.fixAll.eslint": true
},
```
Essa opção fica disponível com a instalação da extensão ESLint.

### Prettier
Enquanto o ESLint marca erros no código, inclusive referentes a formatação do código, e sugere as devidas correções, o Prettier vai fazer a refatoração do código inteiro para garantir que as regras de formatação estejam sendo seguidas, de forma bem opinada e sem deixar margem para o programador escolher opções de formatação. Dessa forma, garante-se que o código siga o padrão de estilo de forma coerente entre todos desenvolvedores do projeto.

Para resolver os conflitos entre as regras do ESLint e as regras do Prettier, foram definidas as seguintes regras no arquivo [prettier.config.js](/prettier.config.js):
- `singleQuote` para utilizar aspas simples;
- `trailingComma` para sempre adicionar vírgula ao final de um objeto que tenha sido quebrado em várias linhas;
- `arrowParens` para que não seja adicionado parênteses quando uma Arrow Function tiver apenas um parâmetro.

### Debug do VSCode
Ao criar uma configuração de debug, foram modificadas as seguintes configurações:

- `"request": "attach"` para que o debug se conecte à execução atual do node ao invés de iniciar uma nova execução;
- `"protocol": "inspector"` para possibilitar o debug a inspecionar o código node (e por isso é passada a flag `--inspect` no comando `tsc-node-dev`);
- `"restart": true` para que o debugger tente refazer o attach automaticamente após o término da sessão Node.js, ou seja, reattach automático na reinicialização do servidor após alguma edição percebida pelo ts-node-dev.
Mais informações sobre as configurações de debug do VS Code: [https://code.visualstudio.com/docs/nodejs/nodejs-debugging](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)

### Model
O model é uma classe responsável por definir uma entidade (objeto) com o qual o código trabalha, normalmente descrevendo a estrutura dessa numa tabela de banco de dados. No caso desse projeto, temos uma entidade Appointment, que é um agendamento de horário com um profissional (provider), portanto, essa entidade tem seu próprio model que descreve as propriedades de um objeto do tipo appointment. Com isso, também deixa de ser necessária a criação de uma interface Appointments para trabalhar com esse objeto.

### Repository
Um repository é uma camada intermediária entre o local de persistência dos dados (BD) e as rotas da aplicação que lidam com esses dados. Por padrão, para cada model, tem-se um repository, pois para listar, criar, atualizar ou deletar um objeto (descrito pelo model) no banco de dados, passa-se pelo repository da respectiva entidade.

### Separation of Concerns (SoC)
Cada parte da aplicação deve ser preocupar apenas com uma responsabilidade ou um tipo de atividade. Por exemplo, as rotas devem servir apenas como forma de roteamento da aplicação, enquanto os repositories, como camada de acesso aos dados, devem ser responsáveis pelo acesso às suas respectivas entidades. SoC é equivalente ao princípio de responsabilidade única (Single Responsability) do SOLID.

### Data Transfer Object (DTO)
Quando passamos dados entre arquivos do projeto, estamos fazendo um procedimento chamado de DTO. Uma boa prática nesse sentido é que os dados passados estejam em formato de objeto para podermos trabalhar com desestruturação do objeto recebido por parâmetro. Assim, ao se modificar o objeto (por um novo campo na entidade, por exemplo), não é necessário alterar a assinatura da função que lida com esse objeto. Caso cada propriedade do objeto fosse passada por parâmetro dessa função, ao se adicionar uma nova propriedade nessa entidade, a função precisaria ter sua assinatura modificada para comportar a alteração e, assim, poderíamos quebrar o código em outra parte do projeto.

### Services
Seguindo o princípio das SoC, os services têm responsabilidade de conter toda a lógica que envolve algum procedimento, normalmente entre a rota e o repository. Por exemplo, para criar um Appointment, temos algumas regras de negócio (não permitir mais de 1 agendamento para o mesmo horário por exemplo) e a rota da aplicação não deve estar preocupada com isso. As rotas devem receber uma requisição, chamar um arquivo que computa o retorno e repassar essa resposta para a camada requisitante. Por isso, nesse caso, a lógica de controlar a possibilidade ou o erro de um novo agendamento é abstraída para um Service. Um Service, também deve ter uma única responsabilidade, então para cada funcionalidade, tem-se um service específico. No caso da criação de agendamentos, essa funcionalidade teve suas regras de negócio abstraída para o [CreateAppointmentService.ts](/src/services/CreateAppointmentService.ts). Nesse sentido, via de regra, um service consiste de uma classe composta por apenas 1 método, normalmente chamado de `execute()` ou `run()`, responsável por executar a funcionalidade objetivo do service.
Um service não tem acesso aos dados de requisição de uma rota, nem do retorno da mesma. Ele apenas recebe por parâmetro os dados que precisa e retorna uma resposta que a rota da aplicação se encarrega de tratar e encaminhar adiante. Com isso, entra um novo conceito também, que é a inversão de dependências.

### Dependency Inversion
A inversão de dependências também é um princípio SOLID (o **D**, no caso) e ele serve para abstrair as dependências de uma classe para evitar conflitos gerados pelos diferentes escopos/namespaces do projeto. No nosso exemplo, ao invés de instanciar um novo AppointmentsRepository dentro do service de CreateAppointments, a instância dessa classe é passada por parâmetro do construtor do service, assim, dentro do service temos a referência para a mesma instância eventualmente utilizada em outras partes da aplicação.
