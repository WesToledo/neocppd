# Diagrama UML de casos de uso

Este diagrama representa os principais atores e casos de uso identificados no sistema NeoCPPD.

```mermaid
flowchart LR
    Visitante((Visitante))
    Servidor((Servidor/docente))
    RH((RH))
    Gestor((Gestor))
    Avaliador((Avaliador))
    Administrador((Administrador))
    Google((Google OAuth))

    subgraph Sistema[NeoCPPD]
        UC1[Consultar informações públicas]
        UC2[Consultar documentos e normativas]
        UC3[Autenticar usuário]
        UC4[Autenticar com Google]
        UC5[Consultar perfil e menus]
        UC6[Solicitar progressão]
        UC7[Acompanhar requerimento]
        UC8[Receber requerimento]
        UC9[Abrir processo]
        UC10[Registrar avaliação]
        UC11[Gerenciar critérios e pontos]
        UC12[Consultar processo]
        UC13[Gerenciar servidores e carreira]
        UC14[Gerenciar campi, cursos e setores]
        UC15[Gerenciar usuários e papéis]
        UC16[Gerenciar privilégios e menus]
    end

    Visitante --> UC1
    Visitante --> UC2

    Servidor --> UC3
    Servidor --> UC5
    Servidor --> UC6
    Servidor --> UC7
    Servidor --> UC12

    Google --> UC4
    UC4 -.->|especializa| UC3

    RH --> UC8
    RH --> UC9
    RH --> UC12
    RH --> UC13

    Gestor --> UC12
    Gestor --> UC13
    Gestor --> UC10

    Avaliador --> UC10
    Avaliador --> UC11

    Administrador --> UC14
    Administrador --> UC15
    Administrador --> UC16

    UC6 -.->|inclui| UC3
    UC7 -.->|inclui| UC3
    UC8 -.->|inclui| UC3
    UC9 -.->|inclui| UC3
    UC10 -.->|inclui| UC3
    UC13 -.->|inclui| UC3
    UC14 -.->|inclui| UC3
    UC15 -.->|inclui| UC3
    UC16 -.->|inclui| UC3
```

## Atores

| Ator | Papel no produto |
|---|---|
| Visitante | Consulta o conteúdo público da CPPD. |
| Servidor/docente | Acessa seu perfil, solicita progressão e acompanha processos. |
| RH | Recebe requerimentos, abre processos e mantém dados funcionais. |
| Gestor | Consulta processos e participa das avaliações e decisões. |
| Avaliador | Registra avaliações, critérios e pontuações. |
| Administrador | Mantém estrutura, usuários, papéis, privilégios e menus. |
| Google OAuth | Fornece a autenticação externa para o login Google. |

## Relações principais

- `Autenticar com Google` é uma variação de `Autenticar usuário`.
- Os casos de uso da área restrita dependem de autenticação.
- `Solicitar progressão` inicia a jornada que pode resultar em um processo.
- `Receber requerimento` pode levar à abertura de um processo.
- `Registrar avaliação` utiliza critérios e pontos.
- Os menus apresentados dependem dos papéis e privilégios do usuário.

## Observação

O diagrama descreve os casos de uso observados ou inferidos a partir do código. Ele não substitui uma especificação formal de requisitos e deve ser revisado com os responsáveis pelo processo da CPPD.
