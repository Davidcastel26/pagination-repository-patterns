# pagination-repository-patterns
This a quick setup in order to create the pagination

## Dir structure 

- 1 🏃🏽‍♂️ into domain we follow this structured 
```
└── 📁domain
    └── 📁datasources
      └── 📁product-datasource
        └── produc.datasource.ts
    └── 📁dto
      └── 📁products-dto
          └── 📁product
            └── find-product.dto.ts
      └── 📁shared-dto
          └── pagination.dto.ts
    └── 📁entities
        └── 📁products-entities
            └── product.entities.ts
    └── 📁repositories
        └── 📁products-repository
            └── index.ts
            └── product-repository.ts
    └── 📁use-cases
        └── 📁products-cases
            └── index.ts
            └── 📁product-cases
                └── get-all-products.ts
```
- 2 😵‍💫 this is infrastructure
```
└── 📁infrastructure
    └── 📁datasource
        └── 📁product-datasource-inf
            └── index.ts
            └── product-datasource.impl.ts
    └── 📁repositories
        └── 📁products-repositories-inf
            └── index.ts
            └── product.repository.impl.ts   
```
- 3 🤓 and this is in to Presentation
```
└── 📁presentation
    └── 📁orders
        └── 📁products
            └── product.controller.ts
            └── product.routes.ts
    └── 📁routes
        └── index.ts
        └── microsoft.ts
    └── routes.ts
    └── server.ts
```

- 😵‍💫 Feeling lost? dont worry now take a look the code how to start with this:
    next 14 already available but this set only works with next 13, remember this app should be with this next structure /src/pages and does not run with next router
    ```
    <!-- this is #1 -->
     npx create-next-app@13.5.5 shell-App --typescript --tailwind --eslint
    <!-- this is #2 -->
     npx create-next-app@13.5.5 mfe-orders-App --typescript --tailwind --eslint
    ```
- ⚡ Fun fact: This app is runing with [shadcn/ui](https://ui.shadcn.com/docs/installation) for more info
    ```
     npx shadcn-ui@latest init
    ```
- 🔥 Install module federation with this version 7.0.8 into both projects
    ```
    npm i @module-federation/nextjs-mf@7.0.8
    ```
- 🏃🏽‍♂️ Last: this is a must to install into both projects
    ```
    npm install webpack@5.89.0 webpack-cli webpack-dev-server --save-dev
    ```

## Setting up
- On the Remote app add NextFederationPlugin to next.config.js and define modules ( components or pages) that need to be shared.

```js 
const NextFederationPlugin = require('@module-federation/nextjs-mf');

const nextConfig = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'my-shell',
        filename: 'static/shellRemoteEntry.js',
        remotes: {
          cardMfe: 'cardMfe@http://localhost:3001/_next/static/cardMfeRemoteEntry.js',          
        }
      })
      
    );
    return config;
  },
}

module.exports = nextConfig
```
- On the Host app add NextFederationPlugin to next.config.js and specify remotes that should be consumed.


```js
const NextFederationPlugin = require('@module-federation/nextjs-mf');

const nextConfig = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'cardMfe',
        filename: 'static/cardMfeRemoteEntry.js',        
        exposes: {
          './card': './src/components/Card',                  
          // './pages-map': './pages-map.js',
        },        
      })
    );
    return config;
  },
}

module.exports = nextConfig
```

- Now remote modules can be consumed by the Host app using next/dynamic

remember this is into /component/CardMfe.tsx
```tsx
  import dynamic from 'next/dynamic';

 export const CardMfe = dynamic(
  // @ts-ignore
    () => import('cardMfe/card'),
    { 
      ssr: false,
      suspense: true 
    }
  );
```

then we can use the component where we want
```tsx
import { CardMfe } from '@/components/CardMfe'

export default function Home() {
  return (
    <>
      <h1>hola</h1>
      <CardMfe />
    </>
  )
}

```


Teka a look into the [repository](http://10.111.102.10:3000/desa03/lacarreta_backend/src/branch/development)

---
⭐️ From [@Davidcastel26](https://github.com/Davidcastel26) 
