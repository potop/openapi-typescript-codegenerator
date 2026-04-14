# Important announcement

> [!IMPORTANT] 
> Please migrate your projects to use [@hey-api/openapi-ts](https://github.com/hey-api/openapi-ts)

Due to time limitations on my end, this project has been unmaintained for a while now. The `@hey-api/openapi-ts`
project started as a fork with the goal to resolve the most pressing issues. going forward they are planning to
maintain the OpenAPI generator and give it the love it deserves. Please support them with their work and make
sure to migrate your projects: https://heyapi.dev/openapi-ts/migrating.html#openapi-typescript-codegen

- All open PR's and issues will be archived on the 1st of May 2024
- All versions of this package will be deprecated in NPM

👋 Thanks for all the support, downloads and love! Cheers Ferdi.

---

# OpenAPI Typescript Codegen

[![NPM][npm-image]][npm-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]
[![Build][build-image]][build-url]

> Node.js library that generates Typescript clients based on the OpenAPI specification.

## Why?
- Frontend ❤️ OpenAPI, but we do not want to use JAVA codegen in our builds
- Quick, lightweight, robust and framework-agnostic 🚀
- Supports generation of TypeScript clients
- Supports generations of Fetch, Node-Fetch, Axios, Angular and XHR http clients
- Supports OpenAPI specification v2.0 and v3.0
- Supports JSON and YAML files for input
- Supports generation through CLI, Node.js and NPX
- Supports tsc and @babel/plugin-transform-typescript
- Supports aborting of requests (cancelable promise pattern)
- Supports external references using [json-schema-ref-parser](https://github.com/APIDevTools/json-schema-ref-parser/)

## Fork divergence

This is a maintained fork of [`openapi-typescript-codegen`](https://github.com/ferdikoomen/openapi-typescript-codegen). All core behavior is preserved; the differences below are additive and only affect the **`fetch`** client. Other clients (`axios`, `xhr`, `node`, `angular`) are unchanged from upstream.

The additions cover three cross-cutting concerns that consumers otherwise need to bolt on via post-generation hand-edits: response caching, opt-in file downloads, and session-expiry detection.

### Extended `OpenAPIConfig` fields

All new fields are optional and default to `undefined`/`false`, so a consumer that sets nothing gets vanilla upstream behavior.

| Field | Type | Purpose |
| --- | --- | --- |
| `CACHE_OPTIONS` | `{ enabled: boolean; ttl?: number; key?: string; featureToggled?: boolean }` | Per-call caching policy. Passed via `configOverrides` on a service method: `Service.get(id, { CACHE_OPTIONS: { enabled: true, key: 'foo' } })`. If `enabled` is true **and** `CACHE_PROVIDER` is configured, the request is served from cache on hit and stored on miss. |
| `CACHE_PROVIDER` | `CacheProvider` (structural: `getItem`, `setItem`, optional `removeItem` / `destroy`) | Pluggable cache backend. The generator ships **no** implementation — wire in your own (IndexedDB, `localStorage`, in-memory Map, etc.) at init time: `OpenAPI.CACHE_PROVIDER = myCache;`. Cache keys default to `JSON.stringify(options)` unless `CACHE_OPTIONS.key` is set. |
| `SMART_DOWNLOAD` | `boolean` | Two effects when `true`: (1) the response is parsed as text even under a JSON content-type (useful for endpoints that return raw YAML/XML with the wrong `Content-Type`), and (2) if the response carries `Content-Disposition: attachment`, a browser "Save file" dialog is triggered automatically. Non-attachment responses flow through unchanged. |
| `REDIRECT` | `RequestRedirect` (`'follow'` \| `'error'` \| `'manual'`) | Forwarded to `fetch`'s `RequestInit.redirect`. Default is the browser's native default (`'follow'`). Set to `'manual'` if you want to observe session-expiry redirects via `ON_OPAQUE_REDIRECT`. |
| `ON_OPAQUE_REDIRECT` | `(response: Response) => void` | Invoked when `fetch` returns a response with `type === 'opaqueredirect'` (only possible when `REDIRECT: 'manual'` is set). Typical use: clear the cache, show a "session expired" modal, redirect to login. |

### Built-in file-download behavior

When `SMART_DOWNLOAD: true`, the generated `request.ts` includes three internal helpers that implement the download pipeline — **no consumer code or hooks required**:

- `handleFileDownload` — bails out unless `Content-Disposition: attachment` is present; silent no-op otherwise.
- `getContentDispositionFilename` — parses the header. Handles RFC 5987 `filename*=UTF-8''...`, quoted `filename="..."`, and bare `filename=...`. Falls back to `'download'`. For exotic cases (non-UTF-8 extended params, multi-line continuations) swap for the `content-disposition` npm package.
- `downloadBlob` — standard synthetic-anchor + `URL.createObjectURL` + click + revoke routine. Browser-only.

### Example consumer wiring

```ts
import { OpenAPI } from './generated/open-api';
import { MyCache } from './my-cache';

OpenAPI.BASE = '/api';
OpenAPI.CACHE_PROVIDER = new MyCache();
OpenAPI.REDIRECT = 'manual';
OpenAPI.ON_OPAQUE_REDIRECT = () => {
    OpenAPI.CACHE_PROVIDER?.destroy?.({ clearData: true });
    showSessionExpiredModal();
};

// Per-call usage:
await MyService.getThing(id, { CACHE_OPTIONS: { enabled: true, key: `thing:${id}` } });
await MyService.exportCsv({ SMART_DOWNLOAD: true });
```

## Install

```
npm install openapi-typescript-codegen --save-dev
```

## Usage

```
$ openapi --help

  Usage: openapi [options]

  Options:
    -V, --version             output the version number
    -i, --input <value>       OpenAPI specification, can be a path, url or string content (required)
    -o, --output <value>      Output directory (required)
    -c, --client <value>      HTTP client to generate [fetch, xhr, node, axios, angular] (default: "fetch")
    --name <value>            Custom client class name
    --useOptions              Use options instead of arguments
    --useUnionTypes           Use union types instead of enums
    --exportCore <value>      Write core files to disk (default: true)
    --exportServices <value>  Write services to disk (default: true)
    --exportModels <value>    Write models to disk (default: true)
    --exportSchemas <value>   Write schemas to disk (default: false)
    --indent <value>          Indentation options [4, 2, tab] (default: "4")
    --postfixServices         Service name postfix (default: "Service")
    --postfixModels           Model name postfix
    --request <value>         Path to custom request file
    -h, --help                display help for command

  Examples
    $ openapi --input ./spec.json --output ./generated
    $ openapi --input ./spec.json --output ./generated --client xhr
```

Documentation
===

The main documentation can be found in the [openapi-typescript-codegen/wiki](https://github.com/ferdikoomen/openapi-typescript-codegen/wiki)

Sponsors
===

If you or your company use the OpenAPI Typescript Codegen, please consider supporting me. By sponsoring I can free up time to give this project some love! Details can be found here: https://github.com/sponsors/ferdikoomen

If you're from an enterprise looking for a fully managed SDK generation, please consider our sponsor:

<a href="https://speakeasyapi.dev/?utm_source=ferdi+repo&utm_medium=github+sponsorship">
    <img alt="speakeasy" src="https://storage.googleapis.com/speakeasy-design-assets/ferdi-sponsorship.png" width="640"/>
</a>

[npm-url]: https://npmjs.org/package/openapi-typescript-codegen
[npm-image]: https://img.shields.io/npm/v/openapi-typescript-codegen.svg
[license-url]: LICENSE
[license-image]: http://img.shields.io/npm/l/openapi-typescript-codegen.svg
[coverage-url]: https://codecov.io/gh/ferdikoomen/openapi-typescript-codegen
[coverage-image]: https://img.shields.io/codecov/c/github/ferdikoomen/openapi-typescript-codegen.svg
[downloads-url]: http://npm-stat.com/charts.html?package=openapi-typescript-codegen
[downloads-image]: http://img.shields.io/npm/dm/openapi-typescript-codegen.svg
[build-url]: https://circleci.com/gh/ferdikoomen/openapi-typescript-codegen/tree/main
[build-image]: https://circleci.com/gh/ferdikoomen/openapi-typescript-codegen/tree/main.svg?style=svg
