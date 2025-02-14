import { Callout } from 'nextra-theme-docs'

# Uso com Next.js

## Fetching de Dados Client-Side

Se sua página contém dados que atualizam frequentemente, e você não precisa pré-renderizar os dadoos, SWR é uma excelente opção e nenhuma configuração especial é necessária: apenas importe `useSWR` e use o hook dentro de qualquer componente que use os dados.

É assim que funciona:

- Primeiro, imediatamente mostre a página sem dados. Você pode mostrar estados de carregamento para dados ausentes.
- Depois, busque os dados no cliente e exiba quando estiver pronto.

Esse método funciona bem para páginas de painel de usuário, por exemplo. Porque um painel de usuário é um painel privado, especifico do usuário, onde o SEO não é relevante e a página não precisa ser pré-renderizada. Os dados são frequentemente atualizados, que requerem o carregamento de dados no cliente.

## Pré-renderizando com Dados Padrão

Se a página precisa ser pré-renderizada, Next.js suporta [2 formas de pré-renderizar](https://nextjs.org/docs/basic-features/data-fetching):
**Static Generation (SSG)** e **Server-side Rendering (SSR)**.

Junto com SWR, você pode pré-renderizar a página para SEO, e também ter recursos como cache, revalidation, tracking de foco, refetching em intervalo no cliente.

Você pode usar a opção `fallback` do [`SWRConfig`](/docs/global-configuration) para passar os dados pré-carregados como o valor inicial de todos os hooks SWR.
For exemplo o `getStaticProps`:

```jsx
 export async function getStaticProps () {
  // `getStaticProps` é executado no lado do servidor
  const article = await getArticleFromAPI()
  return {
    props: {
      fallback: {
        '/api/article': article
      }
    }
  }
}

function Article() {
  // `data` sempre estará disponível como `fallback`
  const { data } = useSWR('/api/article', fetcher)
  return <h1>{data.title}</h1>
}

export default function Page({ fallback }) {
  // hooks SWR dentro do limite do `SWRConfig` usarão esses valores.
  return (
    <SWRConfig value={{ fallback }}>
      <Article />
    </SWRConfig>
  )
}
```

A página continua sendo pré-renderizada, possui SEO amigável, rápida para resposta, mas também é totalmente fornecida pelo SWR no cliente. Os dados podem ser dinâmicos e atualizados ao longo do tempo.

<Callout emoji="💡">
O componente `Article` renderizará os dados pré-gerados primeiro, depois a página será carregada, e ele irá buscar os dados mais recentes denovo para manter a atualização.
</Callout>

### Chaves Complexas

`useSWR` pode ser usado com chaves que são do tipo `array` e `function`. Usando dados pré-carregados com esses tipos de chaves requer serializar as chaves `fallback` com `unstable_serialize`.

```jsx
import useSWR, { unstable_serialize } from 'swr'

export async function getStaticProps () {
  const article = await getArticleFromAPI(1)
  return {
    props: {
      fallback: {
        // unstable_serialize() array style key
        [unstable_serialize(['api', 'article', 1])]: article,
      }
    }
  }
}

function Article() {
  // usando uma chave no estilo array.
  const { data } = useSWR(['api', 'article', 1], fetcher)
  return <h1>{data.title}</h1>
}

export default function Page({ fallback }) {
  return (
    <SWRConfig value={{ fallback }}>
      <Article />
    </SWRConfig>
  )
}
```
