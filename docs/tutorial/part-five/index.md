---
title: Source Plugins
typora-copy-images-to: ./
disableTableOfContents: true
---

> This tutorial is part of a series about Gatsby’s data layer. Make sure you’ve gone through [part 4](/tutorial/part-four/) before continuing here.

## Czego nauczysz się w tym poradniku?

W tym poradniku dowiesz się, jak pobierać dane do strony Gatsby przy użyciu GraphQL i wtyczek źródłowych. Zanim jednak dowiesz się o tych wtyczkach, najpierw musisz wiedzieć, jak korzystać z czegoś co nazywa się GraphiQL - narzędzia, które pomaga ułorzyć strukturę Twoich zapytań.

## Przedstawiamy GraphiQL

GraphiQL jest zintegrowanym środowiskiem deweloperskim (IDE) w GraphQL. Jest to potężne (i pod wieloma względami niesamowite) narzędzie, którego będziesz często używać podczas tworzenia stron internetowych w Gatsby.

Masz do niego dostęp, gdy Serwer Deweloperski Twojej strony działa zwyczajnie pod adresem
<http://localhost:8000/___graphql>.

<video controls="controls" autoplay="true" loop="true">
  <source type="video/mp4" src="/graphiql-explore.mp4"></source>
  <p>Twoja przeglądarka nie obsługuje tego elementu wideo.</p>
</video>

Rozejrzyj się po "typie" `Site` i sprawdź jakie pola są dostępne -- sprawdź też obiekt `siteMetadata` który którkego zapytanie wykonałeś wcześniej. Spróbuj otworzyć GraphiQL i pobawić się trochę ze swoimi danymi! Wciślnij <kbd>Ctrl + Spacja</kbd> (lub <kbd>Shift + Spacja</kbd> jako alternatywny skrót), żeby wyświetlić okno autouzupełniania i <kbd>Ctrl + Enter</kbd>, aby wykonać zapytanie GraphQL. Będziesz używał GraphQL znacznie więcej w dalszej części poradnika.

## Używanie Eksploratora GraphiQL

Eksplorator GraphiQL pozwoli Tobie interaktywnie budować kompletne zapytania poprzez kliknięcia w dostępne pola oraz wejścia/inputy, oszczędzając Ci żmudnego procesu wpisywania tych zapytań ręcznie na klawiaturze.

<EggheadEmbed
  lessonLink="https://egghead.io/lessons/gatsby-build-a-graphql-query-using-gatsby-s-graphiql-explorer"
  lessonTitle="Build a GraphQL Query using Gatsby’s GraphiQL Explorer"
/>

## Wtyczki Sources

Dane w stronach Gatsby mogą pochodzić zewsząd: API, baz danych, CMS, plików lokalnych, itp.

Wtyczki Sources pobierają dane ze swoejgo źródła. Np. wtyczka `filesystem source` wie jak pobierać dane z systemu plików. Wtyczka WordPress wie jak pobierać dane z WordPress API.

Dodaj wtyczkę [`gatsby-source-filesystem`](/packages/gatsby-source-filesystem/) i sprawdź jak ona działa.

Po pierwsze, zainstaluj wtyczkę do katalogu głównego projektu:

```shell
npm install --save gatsby-source-filesystem
```

Then add it to your `gatsby-config.js`:

```javascript:title=gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Pandas Eating Lots`,
  },
  plugins: [
    // highlight-start
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        name: `src`,
        path: `${__dirname}/src/`,
      },
    },
    // highlight-end
    `gatsby-plugin-emotion`,
    {
      resolve: `gatsby-plugin-typography`,
      options: {
        pathToConfigModule: `src/utils/typography`,
      },
    },
  ],
}
```

Save that and restart the gatsby development server. Then open up GraphiQL again.

In the explorer pane, you'll see `allFile` and `file` available as selections:

![graphiql-filesystem](graphiql-filesystem.png)

Click the `allFile` dropdown. Position your cursor after `allFile` in the query area, and then type <kbd>Ctrl + Enter</kbd>. This will pre-fill a query for the `id` of each file. Press "Play" to run the query:

![filesystem-query](filesystem-query.png)

In the Explorer pane, the `id` field has automatically been selected. Make selections for more fields by checking the field's corresponding checkbox. Press "Play" to run the query again, with the new fields:

![filesystem-explorer-options](filesystem-explorer-options.png)

Alternatively, you can add fields by using the autocomplete shortcut (<kbd>Ctrl + Space</kbd>). This will show queryable fields on the `File` nodes.

![filesystem-autocomplete](filesystem-autocomplete.png)

Try adding a number of fields to your query, pressing <kbd>Ctrl + Enter</kbd>
each time to re-run the query. You'll see the updated query results:

![allfile-query](allfile-query.png)

The result is an array of `File` "nodes" (node is a fancy name for an object in a
"graph"). Each `File` node object has the fields you queried for.

## Build a page with a GraphQL query

Building new pages with Gatsby often starts in GraphiQL. You first sketch out
the data query by playing in GraphiQL then copy this to a React page component
to start building the UI.

Let's try this.

Create a new file at `src/pages/my-files.js` with the `allFile` GraphQL query you just
created:

```jsx:title=src/pages/my-files.js
import React from "react"
import { graphql } from "gatsby"
import Layout from "../components/layout"

export default ({ data }) => {
  console.log(data) // highlight-line
  return (
    <Layout>
      <div>Hello world</div>
    </Layout>
  )
}

export const query = graphql`
  query {
    allFile {
      edges {
        node {
          relativePath
          prettySize
          extension
          birthTime(fromNow: true)
        }
      }
    }
  }
`
```

The `console.log(data)` line is highlighted above. It's often helpful when
creating a new component to console out the data you're getting from the GraphQL query
so you can explore the data in your browser console while building the UI.

If you visit the new page at `/my-files/` and open up your browser console
you will see something like:

![data-in-console](data-in-console.png)

The shape of the data matches the shape of the GraphQL query.

Add some code to your component to print out the File data.

```jsx:title=src/pages/my-files.js
import React from "react"
import { graphql } from "gatsby"
import Layout from "../components/layout"

export default ({ data }) => {
  console.log(data)
  return (
    <Layout>
      {/* highlight-start */}
      <div>
        <h1>My Site's Files</h1>
        <table>
          <thead>
            <tr>
              <th>relativePath</th>
              <th>prettySize</th>
              <th>extension</th>
              <th>birthTime</th>
            </tr>
          </thead>
          <tbody>
            {data.allFile.edges.map(({ node }, index) => (
              <tr key={index}>
                <td>{node.relativePath}</td>
                <td>{node.prettySize}</td>
                <td>{node.extension}</td>
                <td>{node.birthTime}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
      {/* highlight-end */}
    </Layout>
  )
}

export const query = graphql`
  query {
    allFile {
      edges {
        node {
          relativePath
          prettySize
          extension
          birthTime(fromNow: true)
        }
      }
    }
  }
`
```

And now visit [http://localhost:8000/my-files](http://localhost:8000/my-files)… 😲

![my-files-page](my-files-page.png)

## What's coming next?

Now you've learned how source plugins bring data _into_ Gatsby’s data system. In the next tutorial, you'll learn how transformer plugins _transform_ the raw content brought by source plugins. The combination of source plugins and transformer plugins can handle all data sourcing and data transformation you might need when building a Gatsby site. Learn about transformer plugins in [part six of the tutorial](/tutorial/part-six/).
