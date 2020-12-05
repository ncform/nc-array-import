# nc-array-import

nc-array-import widget for [ncform](https://github.com/ncform/ncform)

## Install and basic usage

```
npm i -s @ncform/nc-array-import
```

**Add the widget**

```
import ncArrayImport from '@ncform/nc-array-import';

Vue.use(vueNcform, { extComponents: { ncArrayImport } });

// or vm.$ncformAddWidget({name: 'ncArrayImport', widget: ncArrayImport});

```

**Use the widget**

```
{
  "type": "object",
  "properties": {
    "qq": {
      "type": "array",
      "items": {
        "type": "number",
        "ui": {},
      },
      "ui": {
        "showLegend": false,
        "label": "QQ号",
        "widget": "nc-array-import",
        "widgetConfig": {
          "disableCollapse": true,
        }
      }
    },
  }
}
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

You only need to care about `src/components/index.vue`

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Run your end-to-end tests
```
npm run test:e2e
```

### Run your unit tests
```
npm run test:unit
```
