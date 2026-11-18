# ServiceBlueprint — a ProM plugin

ServiceBlueprint turns an XES event log into an editable service blueprint inside the ProM
process mining framework. Each distinct activity is assigned to a service-design layer, either
by a large language model or by keyword rules, and the resulting diagram can be corrected by hand.

Version 1.0.1 · Java 8 · ProM 6.13 or later · GPL-3.0

## What it does

Process discovery produces directly-follows graphs, Petri nets and BPMN diagrams, which are
written for process analysts. Service designers reason about a process in a different notation:
the service blueprint, a layered diagram separating what the customer does from what the provider
does in front of the customer, behind the scenes, and in support of both.

This plugin derives that diagram directly from execution data, placing every distinct activity in
one of five horizontal layers:

- **Customer** — actions performed by the customer
- **Frontstage** — provider actions the customer can see
- **Backstage** — provider actions the customer cannot see
- **Support** — internal activities that support the service without taking part in its main flow
- **Unclassified** — activities that could not be assigned reliably

Blocks are ordered left to right following the dominant ordering observed in the log and annotated
with their occurrence frequency. Solid arrows connect activities within a layer, dashed arrows mark
transitions across layers.

## Requirements

- ProM 6.13 or later
- Java 8
- Maven 3.6 or later, to build from source
- An API key for a large language model, only for the AI classification mode

## Build from source

```
git clone https://github.com/jlfernandezp/service-blueprint-prom.git
cd service-blueprint-prom
mvn package
```

The build produces `target/service-blueprint-1.0.1.jar`.

The project compiles against three JARs that belong to ProM and are kept in `lib/`:
`OpenXES.jar`, `ProM-Framework.jar` and `ProM-Contexts.jar`. They are declared in `pom.xml` as
`system`-scoped dependencies, so `mvn package` fails with a missing-dependency error if any of
them is absent. If your ProM version differs, replace them with the ones from your own
installation:

```
cp /path/to/ProM613/lib/OpenXES.jar        lib/
cp /path/to/ProM613/lib/ProM-Framework.jar lib/
cp /path/to/ProM613/lib/ProM-Contexts.jar  lib/
```

## Install into ProM

1. Copy `service-blueprint-1.0.1.jar` into both the `dist/` and the `lib/` folder of your ProM
   installation.
2. Add this entry to `packages/packages.xml`, inside `<installed-packages>`:

```xml
<package name="ServiceBlueprint" version="1.0.1" os="all" url=""
         desc="Service Blueprint plugin for ProM"
         hasPlugins="true" license="L-GPL">
  <dependency name="ProM-Framework" />
  <dependency name="ProM-Contexts" />
  <dependency name="XESLite" />
</package>
```

3. Start ProM and look for **Discover Service Blueprint** in the Actions panel.

## API key

Only the AI mode needs one. Create a `.env` file in the folder from which ProM is launched, using
`.env.example` as a model:

```
LLM_API_KEY=your-key-here
LLM_MODEL=your-model-here
```

The key can also be supplied as an environment variable, or as a `-Dllm.api.key=...` system
property at startup. Never commit it: `.gitignore` excludes `.env`.

## How to use it

1. Import an XES log into ProM.
2. Run **Discover Service Blueprint**.
3. Choose a classification mode:

| Mode, as named in the dialog | What it does |
|---|---|
| IA (automatic classification) | Sends the distinct activity names to a large language model through the Claude API, in batches of up to 50, and reads back a JSON assignment. Only activity names are sent, so classification quality follows the quality of the names. Needs an API key. |
| Field Auto-Selection (JSON) | Inspects the log and decides which XES fields to match keywords against and with what weight. Takes a JSON file with the selection parameters and the keyword lists. Assignments carry a confidence score. |
| Manual configuration (JSON) | You state the fields, their weights and the keywords explicitly in a JSON file. Nothing is inferred from the log. |
| Default configuration | Keyword rules over `concept:name` (weight 1.0), `org:role` and `org:resource` (0.6), with no configuration file and no API key. |

4. The blueprint opens in the visualiser. In the keyword-based modes, assignments below a 20 %
   confidence threshold are drawn with an orange border: those are the ones worth checking first.

From the toolbar you can filter the diagram by any log attribute, set a minimum frequency for a
connection to be drawn, hide self-loops or frequency labels, highlight the most frequent path,
open a table of per-layer metrics and cross-layer handoffs, export the keyword matches as CSV, and
save the diagram as a PNG image. Any block can be reassigned to a different layer at any time.

## Project structure

```
src/main/java/org/processmining/serviceblueprint/
  classifier/     LLMClassifier, KeywordClassifier, AutoFieldSelector, ConfigParser
  metrics/        BlueprintMetrics
  model/          ServiceBlueprint, BlueprintActivity, BlueprintLevel, configuration types
  plugin/         ServiceBlueprintDiscoveryPlugin, ServiceBlueprintVisualizerPlugin
  visualization/  BlueprintPanel
```

The prompt sent to the language model is defined in `LLMClassifier.PROMPT_TEMPLATE`.

## Event logs

The plugin accepts any XES-compliant log. The logs used to evaluate it are publicly available at
[4TU.ResearchData](https://data.4tu.nl):

| Log | DOI |
|---|---|
| BPI Challenge 2012 | 10.4121/uuid:3926db30-f712-4394-aebc-75976070e91f |
| BPI Challenge 2014: Activity log for incidents | 10.4121/uuid:86977bac-f874-49cf-8337-80f26bf5d2ef |
| BPI Challenge 2017 | 10.4121/uuid:5f3067df-f10b-45da-b98b-86ae4c7a310b |
| BPI Challenge 2019 | 10.4121/uuid:d06aff4b-79f0-45e6-8ec8-e19730c248f1 |
| BPI Challenge 2020: Domestic Declarations | 10.4121/uuid:3f422315-ed9d-4882-891f-e180b5b4feb5 |
| BPI Challenge 2020: International Declarations | 10.4121/uuid:2bbf8f6a-fc50-48eb-aa9e-c4ea5ef7e8c5 |
| Road Traffic Fine Management Process | 10.4121/uuid:270fd440-1057-4fb9-89a9-b699b47990f5 |
| Sepsis Cases | 10.4121/uuid:915d2bfb-7e84-49ad-a286-dc35f063a460 |

## Authors

Jose Luis Fernández-Pascual, Francisco J. Pérez-Blanco and Juan Manuel Vara.
Universidad Rey Juan Carlos, Madrid, Spain.

Contact: jl.fernandezp@alumnos.urjc.es

## Related work

J. L. Fernández-Pascual, F. J. Pérez-Blanco, J. M. Vara, C. Gómez-Macías and D. Granada,
*Beyond Directly-Follows Graphs: Graphical Modelling in Process Mining*, ENASE 2026, SciTePress,
pp. 1141–1148. [doi:10.5220/0014830900004015](https://doi.org/10.5220/0014830900004015)

## License

GNU General Public License v3.0. See [`LICENSE.txt`](LICENSE.txt).

ProM is itself distributed under the GPLv3. The ProM and OpenXES JARs kept in `lib/` are the
property of their respective authors and are included only so that the project can be compiled.
