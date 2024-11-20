**SSG** (Static Site Generation) é um dos métodos de renderização que o Next.js utiliza para gerar páginas estáticas. Quando você usa SSG, o conteúdo da página é **pré-renderizado** no momento da **construção** do projeto, ou seja, antes de o usuário acessar a página. Isso gera arquivos HTML estáticos que podem ser servidos rapidamente pelo servidor, sem necessidade de execução de JavaScript para renderização inicial.

### Como Funciona o SSG (Static Site Generation)?

O SSG no Next.js gera **páginas estáticas** durante o **processo de build** (quando você executa `next build`), o que significa que o conteúdo é gerado no momento da construção do projeto e **não depende de requisições do servidor** no momento de renderizar a página. As páginas são servidas como arquivos HTML já prontos, o que melhora significativamente o tempo de carregamento e a performance.

### Quando Usar o SSG?

Você deve usar o SSG quando:
- O conteúdo da página não muda com frequência ou é o mesmo para todos os usuários.
- Precisa de alto desempenho e pagespeed, já que as páginas estão pré-geradas e podem ser armazenadas em cache.
- Você deseja otimizar SEO, pois o conteúdo das páginas é renderizado no momento da build, o que facilita a indexação pelos motores de busca.

### Exemplo de Uso do SSG no Next.js

No Next.js, você pode usar a função `getStaticProps()` para gerar conteúdo estático para uma página. Essa função é executada no momento da build e os dados são carregados antes de a página ser gerada. O conteúdo estático é então servido a partir do arquivo HTML gerado.

#### 1. **Criando uma Página Estática com SSG**

Vamos criar um exemplo básico de uma página de blog com conteúdo estático gerado usando `getStaticProps`.

```tsx
// pages/blog.tsx

import { GetStaticProps } from 'next';

type BlogPost = {
  id: number;
  title: string;
  content: string;
};

type Props = {
  posts: BlogPost[];
};

const Blog = ({ posts }: Props) => {
  return (
    <div>
      <h1>Blog</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.content}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};

// getStaticProps é executada no momento da build para gerar o conteúdo estático
export const getStaticProps: GetStaticProps = async () => {
  // Aqui você poderia buscar dados de uma API ou banco de dados
  const posts = [
    { id: 1, title: 'Primeiro Post', content: 'Conteúdo do primeiro post.' },
    { id: 2, title: 'Segundo Post', content: 'Conteúdo do segundo post.' },
  ];

  // Retorna as props para a página
  return {
    props: {
      posts,
    },
  };
};

export default Blog;
```

### Explicando o Código:

1. **`getStaticProps`**:
   - A função `getStaticProps` é executada **no momento da build** do projeto. Ela pode buscar dados de uma API, banco de dados ou simplesmente retornar um conjunto de dados estáticos como neste exemplo.
   - O Next.js usa os dados retornados pela função `getStaticProps` para gerar uma versão estática da página.
   - **Importante**: Qualquer dado obtido dentro de `getStaticProps` é usado para gerar a versão estática da página, que será servida diretamente ao usuário.

2. **Componente `Blog`**:
   - O componente `Blog` recebe as `posts` como **props** e exibe-os na página. Esses dados são obtidos de forma estática durante a build.

3. **Resultado**:
   - Quando você roda `next build`, o Next.js gera o HTML da página `blog.tsx` com os dados que foram retornados pela função `getStaticProps`. Esse HTML será servido como uma página estática, o que garante alta performance e SEO otimizado.

### Vantagens do SSG (Static Site Generation)

1. **Performance**: Como o conteúdo é gerado previamente e armazenado como arquivos HTML, as páginas carregam muito rapidamente. O conteúdo estático pode ser servido diretamente por um CDN, o que melhora ainda mais o tempo de resposta.

2. **SEO**: O conteúdo da página é gerado no servidor e é indexado pelos motores de busca logo no momento da construção, o que melhora a visibilidade da página nos resultados de pesquisa.

3. **Custo de Servidor**: Como as páginas são estáticas, você pode hospedá-las em servidores muito simples ou até mesmo em soluções de **CDN** (Content Delivery Network), como Vercel, Netlify, ou GitHub Pages.

4. **Escalabilidade**: Como o conteúdo é estático, a escalabilidade não é um problema. Não há necessidade de consultas a bancos de dados ou chamadas de API para cada requisição.

### Quando o SSG Não é Ideal?

Embora o SSG seja muito poderoso para páginas com conteúdo estático, ele pode não ser a melhor opção em cenários onde:
- O conteúdo muda frequentemente e precisa ser atualizado em tempo real (nesse caso, o **SSR** (Server-Side Rendering) ou **ISR** (Incremental Static Regeneration) são melhores).
- Páginas personalizadas para usuários, como dashboards ou áreas de administração, onde o conteúdo depende de informações dinâmicas específicas de cada usuário.

### Como Funciona o ISR (Incremental Static Regeneration)?

Uma funcionalidade complementar ao SSG no Next.js é o **Incremental Static Regeneration (ISR)**, que permite que você gere páginas estáticas e **as atualize de forma incremental** após a construção inicial. Com o ISR, você pode gerar páginas estáticas, mas também configurar a atualização dessas páginas com base em intervalos de tempo ou quando algo muda.

#### Exemplo de uso do ISR:

```tsx
// pages/posts/[id].tsx

import { GetStaticProps, GetStaticPaths } from 'next';

type PostProps = {
  post: { id: number; title: string; content: string };
};

export const getStaticPaths: GetStaticPaths = async () => {
  // Suponha que você tenha uma lista de IDs de posts que já existem
  const paths = [{ params: { id: '1' } }, { params: { id: '2' } }];
  
  return {
    paths,
    fallback: 'blocking', // ou 'true'/'false' dependendo de sua estratégia
  };
};

export const getStaticProps: GetStaticProps<PostProps> = async ({ params }) => {
  const post = {
    id: parseInt(params!.id!),
    title: 'Título do Post',
    content: 'Conteúdo do Post',
  };

  return {
    props: {
      post,
    },
    revalidate: 60, // Regenera a página a cada 60 segundos
  };
};

const Post = ({ post }: PostProps) => {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
};

export default Post;
```

No exemplo acima, a página de post será gerada estaticamente, mas se um usuário acessar essa página, o Next.js irá regenerá-la no background a cada 60 segundos (no caso do `revalidate: 60`), mantendo o conteúdo atualizado sem precisar de SSR para todas as requisições.

### Conclusão

O **SSG** (Static Site Generation) no Next.js é uma técnica poderosa para criar páginas altamente otimizadas e rápidas. Ele gera conteúdo estático durante o processo de build, o que melhora o desempenho e o SEO das páginas. No entanto, quando você precisa de dados mais dinâmicos ou atualizações frequentes, você pode complementar o SSG com funcionalidades como **SSR** ou **ISR** (Incremental Static Regeneration), dependendo dos requisitos do seu projeto.
