# abap api tools<!-- omit in toc -->

![NPM](https://img.shields.io/npm/l/abap-api-tools)

Command line tool for pattern based applications with ABAP/HANA systems.

- BAPI/RFM call templates ([What is BAPI/RFM?](https://sap.github.io/cloud-sdk/docs/java/features/bapi-and-rfc/bapi-and-rfc-overview/))
- ui components'with ABAP data annotations:
  - [Aurelia](http://aurelia.io/)
  - Angular, React and Vue by [SAP Fundamenal Library](https://sap.github.io/fundamental/)
  - Angular by [Microsot FAST](https://www.fast.design/docs/introduction/)
  - UI5 web components for [React](https://sap.github.io/ui5-webcomponents-react/?path=/story/getting-started--page)

## Content<!-- omit in toc -->

- [Installation](#installation)
- [Usage](#usage)
- [Examples](#examples)
  - [ABAP Function Module call template](#abap-function-module-call-template)
  - [ABAP API annotations for ui elements](#abap-api-annotations-for-ui-elements)
  - [ui elements](#ui-elements)
- [Custom ui configurations](#custom-ui-configurations)
- [Known Issues](#known-issues)
- [Getting Support](#getting-support)
- [Contributing](#contributing)
- [License](#license)

## Installation

Check [prerequisites](https://github.com/SAP/node-rfc#prerequisites) and install globally or locally:

```shell
npm install -g abap-api-tools
```

## Usage

Create project folder and maintain ABAP system(s) destinations in `sapnwrfc.ini` file:

`sapnwrfc.ini`

```ini
# TRACE=3

DEST=MME
USER=demo
PASSWD=welcome
ASHOST=coevi51
SYSNR=00
CLIENT=620
LANG=EN
```

ABAP API for Value Input Help annotations, if implemented in backend system (see [ABAP helpers](https://github.com/SAP/fundamental-tools/tree/master/abap-helpers)), shall be maindefined in `config/system.yaml` file. Use the same destination name like in `sapnwrfc.ini`:

```yaml
MME:
  search_help_api:
    determine: YWS_SHLP_DETERMINE
    dom_values: YWS_SHLP_DOMVALUES_GET
```

Run `abap` command to show help:

```shell
abap

Commands:
  abap call <dest> <rfm...> ABAP function module call template
  abap get <dest> [rfm...]  ABAP API annotations for ui elements
  abap make <ui> [rfm...]   Create ui elements for ABAP API
  abap cp <ui>              Copy ui configuration to local config folder
  abap rm <ui>              Remove local ui configuration

Options:
  --help     Show help
  --version  Show version number
```

or subcommand help:

```shell
abap call
```

## Examples

### ABAP Function Module call template

NodeJS call template of a single ABAP function module.

Echoed to console or saved to local `js` file if the `-s` option is used:

```shell
abap call MME stfc_structure -s
```

The call template provides a source code for ABAP function module invocation, with all parameters and their data structures:

- Optional parameters are commented and initialized with ABAP default values
- Required parameters are initialized with empty string, buffer or zero number
- Conversion Exit ("ALPHA Exit"), if attached to data element, is mentioned in data element comment

More than one ABAP function module

```shell
abap call MME stfc_connection stfc_performance
```

### ABAP API annotations for ui elements

```shell
abap get MME stfc_connection stfc_performancr bapi_user_get_detail
```

Call templates are now saved in `api` folder and annnotations for ui elements in `api/yaml`:

```shell
api
├── bapi_user_get_detail.js
├── stfc_connection.js
├── stfc_performance.js
└── yaml
    ├── alpha.yaml
    ├── fields.yaml
    ├── helps.yaml
    ├── parameters.yaml
    ├── stat.yaml
    └── usage.yaml
```

Use `-o` option for output folder other than the default `api`.

Using `-c` option a path to yaml file with ABAP function modules' names can be provided:

`my-api.yaml`

```yaml
transferOrder:
  - BAPI_WHSE_TO_CREATE_STOCK
  - BAPI_WHSE_TO_GET_DETAIL
  - BAPI_WHSE_TO_GET_LIST
FI:
  - BAPI_ACC_DOCUMENT_POST
  ```

```shell
abap get MME -c my-api # .yaml extension optional
```

Call templates and annotations are saved in respective sub-folders:

```shell
api
├── FI
│   ├── bapi_acc_document_post.js
│   └── yaml
│       ├── alpha.yaml
│       ├── fields.yaml
│       ├── helps.yaml
│       ├── parameters.yaml
│       ├── stat.yaml
│       └── usage.yaml
└── transferOrder
    ├── bapi_whse_to_create_stock.js
    ├── bapi_whse_to_get_detail.js
    ├── bapi_whse_to_get_list.js
    └── yaml
        ├── alpha.yaml
        ├── fields.yaml
        ├── helps.yaml
        ├── parameters.yaml
        ├── stat.yaml
        └── usage.yaml
```

### ui elements

After annotations are saved, ui elements can be created:

```shell
abap make fudamental-ngx -c my-api # .yaml extension optional
```

Now we have one `js` file with a call template one `html` file with ui components, for each ABAP function module:

```shell
bapi_whse_to_get_detail.js
bapi_whse_to_get_detail.html
```

The `html` file contains ui components' templates, for each BAPI/RFM parameter and structure/table data field.

Annoted with:

- Default binding
- Data type, length
- Texts (label, caption)
- Currency or quantity reference fields (unit of measure, currency)
- Value Input Help: field domain values, check table, elementary or complex search help
- SU3 parameters (User SET/GET parameters)

Ui components look like this (Aurelia example):

```html
<ui-input bind="DATA_GENERAL.DISTR_CHAN" shlp.bind="{type: 'SH', id: 'CSH_TVTW'}"
    data-abap.bind="{type:'CHAR', mid:'VTW', length:'2'}"
    label="Distribution Channel">
</ui-input>

<ui-checkbox value.bind="DATA_SPECIFIC.READ_CUREF" label="Referenced Configuration"></ui-checkbox>

<ui-date date.bind="DATA_FLEET.EXPIRY_DATE" label="Validity end date"></ui-date>

<ui-combo bind="DATA_GENERAL.COSTCENTER" shlp.bind="{type: 'CT', id: 'CSKS'}"
    data-abap.bind="{type:'CHAR', alpha:'ALPHA', mid:'KOS', length:'10'}"
    label="Cost Center">
</ui-combo>

<ui-combo bind="INSPROVIDER_X.IV_DISPATCH" shlp.bind="{type: 'FV', id: 'KUEVERS'}"
    data-abap.bind="{type:'CHAR', length:'1'}"
    label="IS-H: Send IV Request Only if Diagnosis Is Maintained">
</ui-combo>
```

## Custom ui configurations

Using two configuration files, ABAP data types can be mapped to custom ui components, of practically any ui framework.

First copy the standard `ui5-react` configuration for example, to local config folder:

```shell
abap cp ui5-react

tree config
config
├── ui5-react-abap.yaml
└── ui5-react.yaml
```

The file with `-abap` suffix defines mapping of ABAP data types to ui components:

`ui5-react-abap.yaml`

```yaml
# Date field (YYYYMMDD) stored as char(8)
DATS:
  type: string
  format: date
  tag: datepicker
  initial: >-
    ""
  comment: YYYYMMDD
```

ABAP `DATS` datatype shall be here represented by `datepicker` ui component.

The ui component layout is defined in the ui config file without `abap` suffix:

`ui5-react.yaml`

```yaml
datepicker: >-
  <FormItem label="~label">
      <DatePicker value={this.~bind}/>
  </FormItem>
```

Elements with tilde prefix `~` are placeholders for texts, data binding and value input helps, documented in `yaml` files.

Custom configuration in local config folder, if present, is used instead of the standard configuration. To go back to standard, remove it from `config` folder or run:

```shell
abap rm ui5-react
```

## Known Issues

Click [here](https://github.com/SAP/fundamental-toolset/issues) to view the current issues.

## Getting Support

If you encounter an issue, you can [create a ticket](https://github.com/SAP/fundamental-toolset/issues/new).

## Contributing

If you want to contribute, please check the [CONTRIBUTING.md](https://github.com/SAP/fundamental-tools#contributing) documentation for contribution guidelines.

## License

Copyright (c) 2018 SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the [LICENSE file](https://github.com/SAP/fundamental-tools#license).