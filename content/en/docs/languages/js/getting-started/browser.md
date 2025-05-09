---
title: Browser
aliases: [/docs/js/getting_started/browser]
description: Learn how to add OpenTelemetry to your browser app
weight: 20
---

{{% include browser-instrumentation-warning.md %}}

While this guide uses the example application presented below, the steps to
instrument your own application should be similar.

## Prerequisites

Ensure that you have the following installed locally:

- [Node.js](https://nodejs.org/en/download/)
- [TypeScript](https://www.typescriptlang.org/download), if you will be using
  TypeScript.

## Example Application

This is a very simple guide, if you'd like to see more complex examples go to
[examples/opentelemetry-web](https://github.com/open-telemetry/opentelemetry-js/tree/main/examples/opentelemetry-web).

Copy the following file into an empty directory and call it `index.html`.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Document Load Instrumentation Example</title>
    <base href="/" />
    <!--
      https://www.w3.org/TR/trace-context/
      Set the `traceparent` in the server's HTML template code. It should be
      dynamically generated server side to have the server's request trace ID,
      a parent span ID that was set on the server's request span, and the trace
      flags to indicate the server's sampling decision
      (01 = sampled, 00 = not sampled).
      '{version}-{traceId}-{spanId}-{sampleDecision}'
    -->
    <meta
      name="traceparent"
      content="00-ab42124a3c573678d4d8b21ba52df3bf-d21f7bc17caa5aba-01"
    />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    Example of using Web Tracer with document load instrumentation with console
    exporter and collector exporter
  </body>
</html>
```

### Installation

To create traces in the browser, you will need `@opentelemetry/sdk-trace-web`,
and the instrumentation `@opentelemetry/instrumentation-document-load`:

```shell
npm init -y
npm install @opentelemetry/api \
  @opentelemetry/sdk-trace-web \
  @opentelemetry/instrumentation-document-load \
  @opentelemetry/context-zone
```

### Initialization and Configuration

If you are coding in TypeScript, then run the following command:

```shell
tsc --init
```

Then acquire [parcel](https://parceljs.org/), which will (among other things)
let you work in TypeScript.

```shell
npm install --save-dev parcel
```

Create an empty code file named `document-load` with a `.ts` or `.js` extension,
as appropriate, based on the language you've chosen to write your app in. Add
the following code to your HTML right before the `</body>` closing tag:

{{< tabpane text=true >}} {{% tab TypeScript %}}

```html
<script type="module" src="document-load.ts"></script>
```

{{% /tab %}} {{% tab JavaScript %}}

```html
<script type="module" src="document-load.js"></script>
```

{{% /tab %}} {{< /tabpane >}}

We will add some code that will trace the document load timings and output those
as OpenTelemetry Spans.

### Creating a Tracer Provider

Add the following code to the `document-load.ts|js` to create a tracer provider,
which brings the instrumentation to trace document load:

```js
/* document-load.ts|js file - the code snippet is the same for both the languages */
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { DocumentLoadInstrumentation } from '@opentelemetry/instrumentation-document-load';
import { ZoneContextManager } from '@opentelemetry/context-zone';
import { registerInstrumentations } from '@opentelemetry/instrumentation';

const provider = new WebTracerProvider();

provider.register({
  // Changing default contextManager to use ZoneContextManager - supports asynchronous operations - optional
  contextManager: new ZoneContextManager(),
});

// Registering instrumentations
registerInstrumentations({
  instrumentations: [new DocumentLoadInstrumentation()],
});
```

Now build the app with parcel:

```shell
npx parcel index.html
```

and open the development web server (e.g. at `http://localhost:1234`) to see if
your code works.

There will be no output of traces yet, for this we need to add an exporter.

### Creating an Exporter

In the following example, we will use the `ConsoleSpanExporter` which prints all
spans to the console.

In order to visualize and analyze your traces, you will need to export them to a
tracing backend. Follow [these instructions](../../exporters) for setting up a
backend and exporter.

You may also want to use the `BatchSpanProcessor` to export spans in batches in
order to more efficiently use resources.

To export traces to the console, modify `document-load.ts|js` so that it matches
the following code snippet:

```js
/* document-load.ts|js file - the code is the same for both the languages */
import {
  ConsoleSpanExporter,
  SimpleSpanProcessor,
} from '@opentelemetry/sdk-trace-base';
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { DocumentLoadInstrumentation } from '@opentelemetry/instrumentation-document-load';
import { ZoneContextManager } from '@opentelemetry/context-zone';
import { registerInstrumentations } from '@opentelemetry/instrumentation';

const provider = new WebTracerProvider({
  spanProcessors: [new SimpleSpanProcessor(new ConsoleSpanExporter())],
});

provider.register({
  // Changing default contextManager to use ZoneContextManager - supports asynchronous operations - optional
  contextManager: new ZoneContextManager(),
});

// Registering instrumentations
registerInstrumentations({
  instrumentations: [new DocumentLoadInstrumentation()],
});
```

Now, rebuild your application and open the browser again. In the console of the
developer toolbar you should see some traces being exported:

```json
{
  "traceId": "ab42124a3c573678d4d8b21ba52df3bf",
  "parentId": "cfb565047957cb0d",
  "name": "documentFetch",
  "id": "5123fc802ffb5255",
  "kind": 0,
  "timestamp": 1606814247811266,
  "duration": 9390,
  "attributes": {
    "component": "document-load",
    "http.response_content_length": 905
  },
  "status": {
    "code": 0
  },
  "events": [
    {
      "name": "fetchStart",
      "time": [1606814247, 811266158]
    },
    {
      "name": "domainLookupStart",
      "time": [1606814247, 811266158]
    },
    {
      "name": "domainLookupEnd",
      "time": [1606814247, 811266158]
    },
    {
      "name": "connectStart",
      "time": [1606814247, 811266158]
    },
    {
      "name": "connectEnd",
      "time": [1606814247, 811266158]
    },
    {
      "name": "requestStart",
      "time": [1606814247, 819101158]
    },
    {
      "name": "responseStart",
      "time": [1606814247, 819791158]
    },
    {
      "name": "responseEnd",
      "time": [1606814247, 820656158]
    }
  ]
}
```

### Add Instrumentations

If you want to instrument Ajax requests, User Interactions and others, you can
register additional instrumentations for those:

```javascript
registerInstrumentations({
  instrumentations: [
    new UserInteractionInstrumentation(),
    new XMLHttpRequestInstrumentation(),
  ],
});
```

## Meta Packages for Web

To leverage the most common instrumentations all in one you can simply use the
[OpenTelemetry Meta Packages for Web](https://www.npmjs.com/package/@opentelemetry/auto-instrumentations-web)
