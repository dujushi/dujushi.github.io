---
title: 'i18n With React Intl and SimpleLocalize'
---
[React Intl](https://formatjs.io/docs/react-intl/){:target="_blank"} is a popular internationalization library. This article demonstrates how to set it up in a new React project and how to manage translations with [SimpleLocalize](https://simplelocalize.io/){:target="_blank"}.

### Create React App

```
yarn create react-app react-intl-tutorial --template typescript
```

Create a `React` app with `Typescript` support.

### Install react-intl

```
yarn add react-intl
```

### Automatic Message ID Generation
We need an ID to identify the message to be translated. While explicit ID would lead to potential collision, we can use `babel-plugin-formatjs` to generate the ID automatically. 

```
yarn add -D babel-plugin-formatjs
```

We then need to add the plugin to Babel configuration. But we cannot do this with the default `create-react-app` setup. Run `react eject` to expose the Babel configuration. Open `webpack.config.js` and add the following configure to `babel-loader`:

```
[
    "formatjs",
    {
        "idInterpolationPattern": "[sha512:contenthash:base64:6]",
        "ast": true
    }
]
```

Refer to [this file](https://github.com/dujushi/react-intl-tutorial/blob/main/config/webpack.config.js#L423-L429){:target="_blank"}.

### Add IntlProvider
`react-intl` provides locale info and translations via `IntlProvider` component. 

```
ReactDOM.render(
  <React.StrictMode>
    <IntlProvider locale={'en'} defaultLocale="en" messages={{}}>
      <App />
    </IntlProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

`locale` and `messages` are hardcoded currently. We will fix this when we have the translations ready.

### useIntl hook
We can now use `useIntl` hook to get translations.

```
function App() {
  const { formatMessage } = useIntl();
  return (
    <div className="App">
      <p>
        {formatMessage({
          defaultMessage: "Hello everyone",
        })}
      </p>
    </div>
  );
}
```

### Message Extraction
The app has defined a simple message. `@formatjs/cli` provides an `extract` command to scan through the application and gather all messages to be translated into a file. The format can be customized depending on the Translation Management System you use. For more details, refer to [this document](https://formatjs.io/docs/getting-started/message-extraction){:target="_blank"}. We will use `SimpleLocalize` which `formatjs` provides a `simple` format for it.

Install `@formatjs/cli`:

```
yarn add -D @formatjs/cli
```

Add `"extract": "formatjs extract"` to `scripts` in `package.json`.

Then run the command to extract the messages:

```
yarn extract src/**/*.tsx --out-file SimpleLocalize/en.json --format simple
```

### Upload default messages to SimpleLocalize
`SimpleLocalize` allows us to upload default messages via Web UI, [Command Line](https://simplelocalize.io/docs/cli/upload-translations/){:target="_blank"}, or API. 

```
curl -s https://get.simplelocalize.io/install | bash
simplelocalize upload \
    --apiKey PROJECT_API_KEY  \
    --uploadPath SimpleLocalize/en.json \
    --uploadFormat single-language-json \
    --languageKey en
```

PS: The Command Line Tool doesn't support Windows. 

### Download translations
When translation finishes, `SimpleLocalize` allows us to download translations via the Command Line Tool as well.

```
simplelocalize download \
    --apiKey PROJECT_API_KEY \
    --downloadPath SimpleLocalize/{lang}.json \
    --downloadFormat single-language-json
```

### Message Compilation
When translations are downloaded, use `@formatjs/cli` to compile the messages to the format `react-intl` requires.

Add `"compile": "formatjs compile"` to `scripts` in `package.json`. Then run the command to compile the messages:

```
for translationFile in SimpleLocalize/*.json
do
    yarn compile $translationFile --ast --out-file src/lang/$(basename -- $translationFile) --format simple
done
```

### Consume translations
We can now load translations into the application.

```
const locale = 'fr';
import(`./lang/${locale}.json`).then(messages => ReactDOM.render(
  <React.StrictMode>
    <IntlProvider locale={locale} defaultLocale="en" messages={messages}>
      <App />
    </IntlProvider>
  </React.StrictMode>,
  document.getElementById("root")
));
```

### Github Actions
We can use Github Actions to automate the message management flow. 

```
name: Upload Translation Keys to SimpleLocalize

on:
  workflow_dispatch:

jobs:
  Upload-Translation-Keys:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "14.x"

      - name: Extract Translation Keys
        run: |
          yarn install
          yarn extract src/**/*.tsx --out-file SimpleLocalize/en.json --format simple

      - name: Upload Translation Keys
        run: |
          curl -s https://get.simplelocalize.io/install | bash
          simplelocalize upload \
            --apiKey ${{secrets.SIMPLE_LOCALIZE_API_KEY}} \
            --uploadPath SimpleLocalize/en.json \
            --uploadFormat single-language-json \
            --languageKey en
```

And 

```
name: Download Translations From SimpleLocalize

on:
  workflow_dispatch:

jobs:
  Download-Translations:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "14.x"

      - name: Download Translations
        run: |
          curl -s https://get.simplelocalize.io/install | bash
          simplelocalize download \
            --apiKey ${{secrets.SIMPLE_LOCALIZE_API_KEY}} \
            --downloadPath SimpleLocalize/{lang}.json \
            --downloadFormat single-language-json

      - name: Compile Translations
        run: |
          yarn install

          for translationFile in SimpleLocalize/*.json
          do
            yarn compile $translationFile --ast --out-file src/lang/$(basename -- $translationFile) --format simple
          done

      - name: Commit translation updates
        run: |
          git config user.email "${{secrets.GIT_MESSAGE_EMAIL}}"
          git config user.name "${{secrets.GIT_MESSAGE_NAME}}"
          git add src
          git commit -m "Update translations" || echo "No translation updates."
          git push
```

### ICU Message Syntax
`FormatJS` supports robust ICU syntax. Refer to [this document](https://formatjs.io/docs/core-concepts/icu-syntax){:target="_blank"} for more details. It also provides a [Linter](https://formatjs.io/docs/guides/develop){:target="_blank"} to help you write the syntax correctly.