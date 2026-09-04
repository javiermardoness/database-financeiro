# database-financeiro

Repositório independente para publicar datasets financeiros estáticos consumidos por diferentes aplicações. O projeto não contém uma aplicação web: a Vercel publica os CSVs diretamente e aplica somente os rewrites e cabeçalhos definidos em `vercel.json`.

## Estrutura

```text
database-financeiro/
├── dados/
│   ├── ativos/
│   │   └── ativos_atualizados.csv
│   └── corretoras/
│       └── corretoras_atualizadas.csv
├── .gitignore
├── README.md
└── vercel.json
```

As categorias representam o assunto dos dados, nunca a aplicação que os consome. As categorias iniciais são `corretoras` e `ativos`.

## URLs públicas

O formato canônico é:

```text
https://www.javiermardones.com/dados/{categoria}/{hash}/{arquivo}
```

URLs atuais:

- Corretoras: `https://www.javiermardones.com/dados/corretoras/7f3a9c2e8b41d6054a91ef72c8360bd5/lista_corretoras.csv`
- Ativos: `https://www.javiermardones.com/dados/ativos/db3967b2ff9941708c727df48d2871e6/ativos_atualizacao.csv`

O hash é apenas um identificador opaco, destinado a tornar a URL difícil de adivinhar. Ele não autentica, não autoriza e não protege o conteúdo, que é público.

Por compatibilidade, a URL legada abaixo continua sendo atendida por rewrite, sem redirecionar o consumidor:

```text
https://www.javiermardones.com/dados/lista_corretoras_atualizadas.csv
```

## Como adicionar um dataset

1. Escolha uma categoria por assunto em `dados/`; não use nomes de produtos ou aplicações consumidoras.
2. Adicione o CSV à pasta da categoria, preservando UTF-8 e um cabeçalho consistente.
3. Gere um identificador aleatório e opaco. Não use `v1`, `v2` ou outro número de versão.
4. Adicione em `vercel.json` um rewrite da URL canônica para o arquivo físico.
5. Documente a nova URL neste README.
6. Abra uma branch, publique um deployment de preview e valide status HTTP, MIME type e conteúdo antes do merge.

Quando um dataset for atualizado sem mudança incompatível de schema, substitua seu arquivo físico e mantenha a URL canônica. Se houver quebra de contrato, adicione outro arquivo e outro hash; não reutilize silenciosamente uma URL com formato incompatível.

## Publicação no GitHub e na Vercel

O GitHub é a fonte versionada. Depois que o GitHub App da Vercel receber acesso ao repositório, a integração Git cria previews para branches e publica a branch `main` em produção depois do merge. Enquanto esse acesso não estiver autorizado, os deployments são publicados manualmente com o Vercel CLI a partir de uma árvore de trabalho revisada.

Não há etapa de build nem dependências: o preset é `Other` e a raiz do repositório é servida como conteúdo estático.

Fluxo recomendado:

```text
branch -> commit -> push -> preview Vercel -> testes -> pull request -> merge em main
```

Nunca inclua `.vercel/`, tokens ou outras credenciais no repositório.

## Encaminhamento pelo domínio principal

`www.javiermardones.com` permanece vinculado ao projeto `javiermardones-site`. Esse projeto deve encaminhar somente `/dados/:path*` para o domínio de produção do projeto `database-financeiro`:

```text
www.javiermardones.com/          -> javiermardones-site
www.javiermardones.com/dados/... -> database-financeiro
```

O encaminhamento é um rewrite (proxy), portanto a URL vista pelo consumidor não muda. O domínio principal não deve ser adicionado diretamente ao projeto de dados, pois isso substituiria o site existente. A regra do site principal e a compatibilidade da URL legada devem ser validadas em preview antes do merge.
