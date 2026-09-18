# Revisa

Assistente de conversa que transforma uma lista de tópicos soltos em um plano de revisão distribuído nos dias disponíveis até a prova.

Protótipo desenvolvido para a **Atividade Prática — Tópico 13 (Colocando a IA em ação: da ideia à solução)**, na disciplina de Experiência do Usuário.
Tema escolhido: **apoio aos estudos e aprendizagem**.

**[Abrir o protótipo](https://SEU-USUARIO.github.io/revisa/)** · **[Ver a apresentação](https://SEU-USUARIO.github.io/revisa/slides.html)**

---

## O problema

O aluno costuma saber *o que* precisa estudar, mas não *quando* estudar cada coisa. Na semana de provas, quatro matérias chegam empilhadas e sem ordem, e a véspera acaba absorvendo tudo: leitura corrida, sem exercício e sem retorno ao conteúdo.

Aplicativos de tarefa existentes registram bem o que precisa ser feito, mas devolvem a decisão mais difícil para o aluno — a distribuição no tempo. É exatamente essa decisão que o Revisa assume.

## A solução

O aluno descreve a situação em linguagem normal. O assistente pergunta o que falta saber (prazo e tempo disponível por dia) e devolve um plano dia a dia, ajustável dentro da própria conversa.

| Recurso | O que faz |
|---|---|
| Distribuição por dia | Divide os tópicos nos dias até a prova, sem sobrecarregar um único dia |
| Dia de fechamento | Reserva o último dia para revisão geral, nunca para conteúdo novo |
| Repetição espaçada | Reagenda o que o aluno errou para voltar alguns dias depois |
| Explicação em contexto | Detalha um conceito e propõe exercício sem sair do fluxo da conversa |
| Ajuste conversacional | O plano é renegociado por mensagem, não por formulário |

---

## Sobre esta demonstração

> Este é um protótipo de interface. As respostas são **roteirizadas**: nenhum modelo de linguagem é chamado.

A escolha foi deliberada. O objetivo da atividade é avaliar o **fluxo de interação**, e um roteiro fixo torna a apresentação previsível — sem depender de rede, de chave de API ou da variação de resposta de um modelo ao vivo.

| Aspecto | Neste protótipo | Em uma versão real |
|---|---|---|
| Geração das respostas | Roteiro fixo em JavaScript | Chamada a uma API de modelo de linguagem |
| Entrada do usuário | Botões com mensagens prontas | Campo de texto livre |
| Persistência do plano | Nenhuma, recomeça a cada carregamento | Banco de dados com conta de usuário |
| Lembretes diários | Apenas mencionados na conversa | Notificação push ou e-mail |

### Por que botões em vez de campo de texto

Com respostas prontas, quem assiste à demonstração percorre o caminho certo sem digitar e sem sair do roteiro. O custo é perder a sensação de conversa aberta — aceitável para uma apresentação de 5 a 10 minutos, onde o risco de uma resposta inesperada estragar a demo é maior do que o ganho de realismo.

---

## Como rodar

Não há build nem dependências. Basta abrir o arquivo:

```bash
git clone https://github.com/SEU-USUARIO/revisa.git
cd revisa
# abra index.html no navegador, ou sirva localmente:
python3 -m http.server 8000
```

### Publicar no GitHub Pages

1. Envie os arquivos para a branch `main`
2. Em **Settings → Pages**, selecione a branch `main` e a pasta `/ (root)`
3. O site fica disponível em `https://SEU-USUARIO.github.io/revisa/` em alguns minutos

---

## Estrutura

```
revisa/
├── index.html      # protótipo do chat (HTML + CSS + JS em arquivo único)
├── slides.html     # apresentação navegável por teclado
└── README.md
```

Tudo em arquivo único e sem framework: o protótipo precisa abrir direto no GitHub Pages, sem etapa de build.

## Como o roteiro funciona

A conversa é um grafo de nós no objeto `ROTEIRO`. Cada nó guarda as falas do assistente e as opções de resposta do aluno; cada opção aponta para o próximo nó pelo campo `vai`.

```js
prova: {
  bot: ["Cinco dias dá tempo. Quais conteúdos caem na prova?"],
  opcoes: [
    // "vai" é o id do próximo nó — é isso que dá o efeito de conversa
    { texto: "Distribuição de frequências, probabilidade e amostragem", vai: "tempo" },
    { texto: "Tenho o PDF do professor, mas não li ainda",              vai: "pdf"   }
  ]
}
```

Nós que entregam um plano de estudo incluem também a chave `plano`, renderizada como tabela dentro do balão de resposta:

```js
plano: {
  titulo: "Plano de revisão — Estatística, 5 dias",
  linhas: [
    ["Dia 1", "Distribuição de frequências: tabelas, classes e regra de Sturges"],
    ["Dia 2", "Exercícios de frequência + construção de histograma"]
  ]
}
```

Para adicionar um caminho novo, basta criar um nó com o mesmo formato e apontar alguma opção existente para ele. Nenhuma outra alteração é necessária.

### Caminhos disponíveis

```
início ─┬─ prova ──────────┐
        ├─ perdido ────────┤
        └─ revisão ────────┴─ tempo ─┬─ plano (2h/dia) ─┬─ sturges
                                     │                  ├─ exercício ─ correção
                                     │                  └─ ajuste ─ fim
                                     ├─ plano enxuto (1h/dia) ─ cortes
                                     └─ plano de fim de semana
```

---

## Ferramentas de IA usadas no processo

| Ferramenta | Uso |
|---|---|
| ChatGPT | Levantamento de técnicas de estudo com evidência: repetição espaçada, prática de recuperação, blocos com pausa |
| Google Gemini | Comparação de aplicativos existentes — o que resolvem e onde param |
| Geração de imagens | Referências visuais para a identidade do protótipo |

## Decisões técnicas

| Decisão | Motivo |
|---|---|
| Sem framework | O protótipo precisa abrir direto no GitHub Pages, sem etapa de build |
| Arquivo único por página | Facilita revisão em grupo e evita quebra de caminhos relativos no Pages |
| Atraso proporcional ao tamanho da fala | Um atraso fixo denuncia a simulação; variável parece processamento real |
| `prefers-reduced-motion` respeitado | Quem tem a preferência ativada recebe as respostas sem animação nem espera |
| Tema claro e escuro | Acompanha a preferência do sistema, já que a demo pode rodar em qualquer máquina |

## Acessibilidade

- Navegação por teclado com foco visível em todos os botões
- Região do chat marcada com `aria-live="polite"`, para leitores de tela anunciarem mensagens novas
- Contraste de texto acima de 4.5:1 nos dois temas
- Layout responsivo até 360px de largura

## Limitações conhecidas

- O plano não é salvo: recarregar a página recomeça a conversa
- Não há entrada de texto livre — só as opções previstas no roteiro
- Os planos gerados são fixos, não calculados a partir dos tópicos informados

## Licença

Trabalho acadêmico, uso livre para fins educacionais.
