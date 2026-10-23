# 📚 Modelagem do Sistema de Empréstimo de Livros

Modelagem orientada a objetos, em **UML**, de um sistema de biblioteca que cobre **reserva, empréstimo, renovação e devolução de livros**, além de multa por atraso com cobrança via boleto bancário.

Este repositório reúne os modelos do sistema sob cinco visões complementares: **estática** (domínio e classes), **funcional** (casos de uso), **dinâmica** (sequência, estados e atividades) e **arquitetural** (componentes e implantação). Não há código-fonte da aplicação; o foco é o modelo e a coerência entre suas partes.

---

## Sumário

- [Visão geral do sistema](#visão-geral-do-sistema)
- [Atores e funcionalidades](#atores-e-funcionalidades)
- [Regras de negócio](#regras-de-negócio)
- [Diagramas](#diagramas)
  - [1. Casos de uso](#1-casos-de-uso)
  - [2. Domínio e classes de projeto](#2-domínio-e-classes-de-projeto)
  - [3. Sequência](#3-sequência)
  - [4. Estados do Livro](#4-estados-do-livro)
  - [5. Atividades](#5-atividades)
  - [6. Componentes](#6-componentes)
  - [7. Implantação](#7-implantação)
- [Decisões de projeto](#decisões-de-projeto)
- [Consistência entre os modelos](#consistência-entre-os-modelos)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como visualizar e editar](#como-visualizar-e-editar)
- [Autores](#autores)

---

## Visão geral do sistema

Uma biblioteca precisa informatizar o processo de empréstimo de livros. O sistema permite que estudantes reservem, peguem emprestado, renovem e devolvam livros, que atendentes registrem devoluções e emitam multas, e que administradores mantenham o acervo e os cadastros de pessoas, estudantes e atendentes.

**Reserva, empréstimo e devolução** são especializações de uma **transação**. Toda transação ocorre em uma data e envolve um livro e dois papéis: um estudante e um atendente. Cada papel é assumido por uma pessoa.

Ao longo de seu ciclo de vida, um livro passa pelos estados:

```
LIVRE → RESERVADO → EMPRESTADO → RENOVADO → DEVOLVIDO → LIVRE | REPARANDO → LIVRE | INCINERADO
```

## Atores e funcionalidades

| Ator | Interage por meio de | Funcionalidades |
|---|---|---|
| **Estudante** | `TratadorDeEmprestimo` | Solicitar reserva (f1), consultar reserva (f2), efetivar empréstimo (f3), renovar empréstimo (f4) |
| **Atendente** | `TratadorDeDevolucao` | Registrar devolução (f5), emitir boleto de multa (f6), inspecionar livro, enviar/retornar de reparo |
| **Administrador** | `TratadorDeAcervo`, `TratadorDePessoa`, `TratadorDeEstudante`, `TratadorDeAtendente` | Manter (inserir, consultar, atualizar, excluir) Livro, Pessoa, Estudante e Atendente |
| **Banco** *(sistema externo)* | `iCobranca` / `iQuitacao` | Recebe a cobrança do boleto e informa a quitação ao Financeiro |

## Regras de negócio

- **Reserva:** só é aceita se o livro estiver cadastrado, o estudante estiver cadastrado e o livro estiver `LIVRE`.
- **Empréstimo:** só é efetivado se houver reserva do livro para o mesmo estudante. A reserva é removida da `Movimentacao` e o empréstimo é inserido.
- **Renovação:** no máximo **2 renovações**, cada uma estendendo o vencimento em **30 dias**. A primeira muda o estado para `RENOVADO`; a segunda mantém `RENOVADO`.
- **Devolução:** registra a devolução, remove o empréstimo da `Movimentacao` e leva o livro a `DEVOLVIDO`. Em seguida há **inspeção**: em bom estado o livro volta a `LIVRE`; danificado vai para `REPARANDO`.
- **Reparo:** ao retornar, o livro volta a `LIVRE`; se o reparo for inviável, o livro é `INCINERADO` (estado final).
- **Multa:** **R$ 1,00 por dia de atraso**, emitida como boleto bancário e enviada ao estudante por e-mail. O boleto passa de `EMITIDO` para `PAGO` quando o banco informa a quitação.
- **Cadastros:** um estudante ou atendente só pode ser cadastrado se a **pessoa** associada já estiver cadastrada.
- **Exclusões:** a integridade referencial não é tratada, por simplificação.

---

## Diagramas

Todos os diagramas foram feitos no **Astah** e exportados como imagem em [`docs/diagramas/`](docs/diagramas/). Nomes de classes, interfaces, operações e estados são os mesmos em todos eles.

### 1. Casos de uso

![Diagrama de casos de uso](docs/diagramas/UseCase_Diagram0.png)

- **Estudante:** renovar empréstimo, efetivar empréstimo, registrar reserva, consultar reserva de livros e consultar empréstimo.
- **Atendente:** registrar devolução, emitir multa (`«extend»` de *Registrar Devolução*) e consultar reserva de livros.
- **Administrador:** manter livro (inserir no acervo, atualizar, excluir, consultar acervo), manter pessoa, estudante e atendente.
- Relacionamentos `«include»` reaproveitam as consultas (acervo, pessoas cadastradas, empréstimo, reserva).

### 2. Domínio e classes de projeto

**Modelo de domínio:**

![Modelo de domínio](docs/diagramas/Domain_Diagram0.png)

**Classes de projeto, com atributos e operações:**

![Diagrama de classes](docs/diagramas/Class_Diagram0.png)

| Grupo | Elementos |
|---|---|
| **Pessoas e papéis** | `Pessoa`, `Papel` *(abstrata)*, `Estudante`, `Atendente`, `Administrador` |
| **Acervo** | `Acervo`, `Livro`, enum `EstadoLivro` |
| **Transações** | `Transacao` *(abstrata)*, `Reserva`, `Emprestimo`, `Devolucao`, `Movimentacao` |
| **Multa** | `Multa`, enum `EstadoMulta` (`EMITIDA`, `QUITADA`) |
| **Cadastros** | `Relacionamento` (ponto único de acesso a pessoas, estudantes e atendentes) |
| **Tratadores (controladores)** | `TratadorDeEmprestimo`, `TratadorDeDevolucao`, `TratadorDeAcervo`, `TratadorDePessoa`, `TratadorDeEstudante`, `TratadorDeAtendente` |
| **Interfaces** | `iLivro`, `iPessoa`, `iEstudante`, `iAtendente`, `iEmprestimo`, `iDevolucao` |

### 3. Sequência

Um diagrama de sequência por grupo de casos de uso.

**Consultar, emprestar, renovar e reservar livro.** O `TratadorDeEmprestimo` coordena `Movimentacao`, `Acervo`, `Livro` e `Relacionamento`.

![Sequência: consultar, emprestar, renovar e reservar livro](docs/diagramas/Consultar__Emprestar__Renovar_e_Reservar_livro.png)

**Registrar devolução.** Cria a `Devolucao`, verifica o atraso, inspeciona o livro e remove o `Emprestimo`.

![Sequência: registrar devolução](docs/diagramas/Registrar_devolucao.png)

**Emitir boleto de multa.** Consulta o empréstimo, obtém os dias de atraso e cria a `Multa`.

![Sequência: emitir boleto de multa](docs/diagramas/Emitir_boleto_de_multa.png)

**Manter Livro, Pessoa, Estudante e Atendente.** Inserir, atualizar, consultar e excluir, conduzidos pelo Administrador.

![Sequência: manter livro, pessoa, estudante e atendente](docs/diagramas/Manter_Livro__Pessoa__Estudante_e_Atendente_.png)

### 4. Estados do Livro

![Diagrama de estados do Livro](docs/diagramas/Statemachine_Diagram0.png)

| Origem | Evento | Guarda | Destino |
|---|---|---|---|
| *(inicial)* | `inserirLivro()` | | `Livre` |
| `Livre` | `solicitarReserva()` | estudante cadastrado | `Reservado` |
| `Reservado` | `realizarEmprestimo()` | reserva registrada para o mesmo estudante | `Emprestado` |
| `Emprestado` | `renovarEmprestimo()` | `numRenovacoes = 0` | `Renovado` |
| `Renovado` | `renovarEmprestimo()` | `numRenovacoes < 2` | `Renovado` *(auto-transição)* |
| `Emprestado` / `Renovado` | `registrarDevolucao()` | | `Devolvido` (com `emitirBoletoDeMulta` se houver atraso) |
| `Devolvido` | `inspecionarLivro()` | `emBomEstado = true` | `Livre` |
| `Devolvido` | `inspecionarLivro()` | `emBomEstado = false` | `Reparando` |
| `Livre` | `enviarParaReparo()` | dano identificado no acervo | `Reparando` |
| `Reparando` | `retornarDoReparo()` | | `Livre` |
| `Reparando` | | reparo inviável | `Incinerado` *(final)* |
| `Livre` | `excluirLivro()` | | *(final)* |

O estado `Renovado` tem atividade interna `entry / numRenovacoes++`.

### 5. Atividades

Dois diagramas com partições (swimlanes) para atores e departamentos: **Acadêmico, Estudante, RH, Atendente, Biblioteca, Financeiro e Banco** (e **Administrador** no cadastro).

**Cadastros:** o administrador registra Pessoa, Estudante, Atendente e Livro. Estudante e atendente dependem da pessoa já existir no RH.

![Atividades: cadastros](docs/diagramas/Cadastros.png)

**Ciclo completo na biblioteca:** reserva → empréstimo → renovação → devolução e inspeção → reparo ou incineração → multa → pagamento no banco → quitação.

![Atividades na biblioteca](docs/diagramas/AtividadesNaBiblioteca.png)

Os fluxos de objeto mostram os estados de `Livro` (`RESERVADO`, `EMPRESTADO`, `RENOVADO`, `DEVOLVIDO`, `REPARANDO`, `INCINERADO`, `LIVRE`) e de `Boleto` (`EMITIDO`, `PAGO`).

### 6. Componentes

![Diagrama de componentes](docs/diagramas/Component_Diagram0.png)

Os componentes se conectam **exclusivamente por interfaces** fornecidas (pirulito) e requeridas (soquete).

| Componente | Interfaces fornecidas | Interfaces requeridas | Classes realizadoras |
|---|---|---|---|
| **Interface Web** | — | `iLivro`, `iEmprestimo`, `iDevolucao`, `iEstudante`, `iAtendente`, `iPessoa` | — |
| **Biblioteca** | `iLivro`, `iEmprestimo`, `iDevolucao` | `iEstudante`, `iAtendente`, `iMulta` | `Livro`, `Acervo`, `Movimentacao`, `TratadorDeAcervo`, `TratadorDeEmprestimo`, `TratadorDeDevolucao` |
| **Acadêmico** | `iEstudante` | `iPessoa` | `TratadorDeEstudante`, `Relacionamento` |
| **RH** | `iPessoa`, `iAtendente` | — | `TratadorDePessoa`, `TratadorDeAtendente`, `Relacionamento` |
| **Financeiro** | `iMulta`, `iQuitacao` | `iCobranca` | `Multa`, `TratadorDeDevolucao` |
| **Banco** *(externo)* | `iCobranca` | `iQuitacao` | — |

**Principais operações das interfaces:**

- `iLivro`: `consultarLivro`, `inserirLivro`, `excluirLivro`, `atualizarLivro`
- `iEmprestimo`: `solicitarReserva`, `consultarReservaLivro`, `realizarEmprestimo`, `renovarEmprestimo`
- `iDevolucao`: `registrarDevolucao`, `inspecionarLivro`, `emitirBoletoDeMulta`, `enviarParaReparo`, `retornarDoReparo`

### 7. Implantação

![Diagrama de implantação](docs/diagramas/Deployment_Diagram0.png)

| Nó | Tipo | Ambiente de execução | Artefato |
|---|---|---|---|
| `pc1 : Computador do Usuário` | `«device»` | WebBrowser | `index.html` |
| `srvApp : Servidor de Aplicação` | `«device»` | Ubuntu ⊃ JVM 1090 ⊃ Tomcat 10.0.11 | `source.jar` |
| `srvMySQL : Dell Edge 3600` | `«device»` `«external»` | Windows 10 ⊃ MySQL | Banco de dados principal |
| `srvOracle : Dell Edge 3600` | `«device»` `«external»` | Ubuntu ⊃ Oracle | Banco de dados espelho |

**Caminhos de comunicação:** navegador → servidor de aplicação via **HTTPS** (`0..*` → `1`); aplicação → MySQL via **JDBC**; MySQL → Oracle por **replicação** (`1` → `1..*`).

---

## Decisões de projeto

- **Delegação:** os tratadores não guardam dados; delegam para `Relacionamento`, `Acervo` e `Movimentacao`.
- **Ponto único de acesso interno:** `Movimentacao` centraliza reservas, empréstimos e devoluções (associações distintas, acesso único). `Relacionamento` faz o mesmo para pessoas, estudantes e atendentes.
- **Tratadores como controladores:** cada ator conversa com uma classe de entrada, que implementa uma interface (`iLivro`, `iEmprestimo`, `iDevolucao` etc.).
- **Papéis separados de pessoas:** `Pessoa` *tem* `Papel`s (`Estudante`, `Atendente`, `Administrador`), o que permite que a mesma pessoa assuma papéis diferentes.
- **Estado do livro como enum:** `EstadoLivro` é a fonte única dos estados, usada no diagrama de classes, no de estados e nos de sequência e atividades.
- **Multa calculada na própria classe:** `Multa.calcularValor()` aplica R$ 1,00 por dia de atraso, sendo `Emprestimo.diasDeAtraso()` e `estaAtrasado()` os responsáveis por informar o atraso.

## Consistência entre os modelos

Os diagramas foram revisados para manter as seguintes correspondências:

- Os estados do `Livro` são os mesmos no atributo `estado`, no diagrama de estados e nos fluxos de objeto do diagrama de atividades.
- Os eventos do diagrama de estados correspondem a operações das classes de projeto (`Livro`, `TratadorDeEmprestimo`, `TratadorDeDevolucao`, `TratadorDeAcervo`).
- As interfaces dos componentes correspondem às interfaces e aos tratadores do diagrama de classes.
- Cada componente tem um artefato implantado em algum nó do diagrama de implantação.

## Estrutura do repositório

```
.
├── README.md
├── modelo/
│   └── sistema-biblioteca.asta        # arquivo-fonte do Astah
└── docs/
    └── diagramas/
        ├── UseCase_Diagram0.png
        ├── Domain_Diagram0.png
        ├── Class_Diagram0.png
        ├── Consultar__Emprestar__Renovar_e_Reservar_livro.png
        ├── Registrar_devolucao.png
        ├── Emitir_boleto_de_multa.png
        ├── Manter_Livro__Pessoa__Estudante_e_Atendente_.png
        ├── Statemachine_Diagram0.png
        ├── Cadastros.png
        ├── AtividadesNaBiblioteca.png
        ├── Component_Diagram0.png
        └── Deployment_Diagram0.png
```

## Como visualizar e editar

- **Visualizar:** abra as imagens em [`docs/diagramas/`](docs/diagramas/) ou use os links deste README.
- **Editar:** abra o arquivo `.asta` em [`modelo/`](modelo/) com o [Astah](https://astah.net/) (UML ou Professional).
- **Exportar:** no Astah, use *File → Save Diagram as Image* para atualizar as imagens após qualquer alteração.
