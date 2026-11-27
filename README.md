# base-docs-avro

The base family's type system in AVRO: core types and their variants, named by URI, their schemas, the registries.

Every value the family keeps has an AVRO schema. A type is named by a
URI; its AVRO full name is derived from it behind the scenes, and its
identity is its fingerprint. A core type is declared once in the
family; its variants carry more or less detail, by where they are used,
and AVRO's own resolution reads one as another. The modules:
`mycelium-avro`, the schemas; `mycelium-value`, a value's bytes; how a
URI and a data name are written: `base-docs-uri`.

## Links

- [test](test/)
- [test-reports](test-reports/)
- [modules](modules/)

## Why AVRO

Brought forward, to develop with from the ground up (the human,
2026-11-21: "I think the size, plus the data contracts in place plus the
rpc server - I think there is a case for bringing avro forward and
develop with it from the ground up"); it is the data contracts, not
containers only (the human, 2026-11-25: "the task for avro is far
greater than just containers, it has to do all data schema
contracts").

- **Smaller**: measured on records shaped like ours, AVRO's binary is
  about 2 to 2.7 times smaller than JSON per record, the single-object
  header included (279 B to 133 B; 225 B to 84 B with small fields
  only); 1.2 times once deflated together, random GUIDs not compressing.
- **Namespaced, without a registry's limits** (the human: "We need
  namespacing, but I don't want the limiting rules that are on the avro
  schema registry as often encountered"): "The registry".
- **One schema, every form**: a document in git stays JSON, a record in a
  topic is AVRO's binary, both typed by the same schema.

## The family

The scope of consistency (the human, 2026-11-21: "We develop within a
'family' scope (base at the moment), and that is where we provide
consistency across the whole. A new family would mean import/export
across a boundary.").

- **The core types are a family's**: within it, versions and variants
  read as AVRO reads them, is-like where asked.

## Data shapes

The shapes JSON data keeps, so that every value has an AVRO schema (the
team, 2026-11-21: "That will give us a natural flow into AVRO"). Rules
for what is written from now on; checked once the schemas are there.

- **An object is a record or a map, never both**: a record has known
  fields; a map has data keys only, its values one type. Fixed fields
  with open keys beside them become a field holding a map
  (`{"size":3,"nodes":{...}}`).
- **An array holds one type**: no tuples, no mixed items; a record with
  named fields instead.
- **A field has one shape**: one of a value or a record is fine; never
  sometimes one array or map and sometimes another.
- **Absent and `null` mean the same**: a missing field takes its
  default; nothing hangs on the difference.
- **Integers up to 2^53**; ids and counters beyond, strings.
- **No free-form JSON, no "any"**: every field of a concrete type. A
  value whose type is not in the structure names it: a stream's record
  by where it is, an attachment by its `type`. (The first build's
  declared `json` type, `mycelium-value-avro/json`, is not used by what
  is brought to the request.)
- **A time is ISO 8601 text in JSON**, milliseconds in AVRO
  (`timestamp-millis`): a record's `timestamp`, a log entry's `time`.
- **Bytes are base64 text in JSON and in memory**, bytes in AVRO, a
  logical type of the engine's (`base64`): an attachment's `content`.
  Strict: text that does not decode and encode back to itself is
  refused.
- **Values that are names stay strings**: kinds, positions, URI parts
  (they have hyphens; an AVRO enum's symbols cannot).
- **Documents stay plain JSON**, validated as values against their
  schema; not AVRO's JSON encoding, which wraps unions.

## Structure or language

What is structured and what is said, a rule (the human, 2026-11-26:
"Structured data is necessary but don't push for it, use it where
operationally necessary, and where natural language is appropriate";
"Natural language doesn't mean it isn't easily parsed"). Its judge is
the AI, as the main user of the data (the human: "you are the judge of
it").

- **Structured, where something acts on it exactly**, and it can be
  named what does:
  - passed on as it is to another command: an identifier (a URI, an
    offset, a fingerprint, a key);
  - branched on by code: an outcome, a state;
  - an address: where it is, where it goes;
  - aggregated or compared: a number, a time;
  - answered for: who asked, and for whom.
- **Said, in natural language, what is read**: what was asked, what
  happened, why it failed, what to do next, what a thing is.
- **Where in doubt, said**: structure is added when what acts on it is
  there, not before.
- **A fact once**: never as a field and in the text beside it.
- **Written so it parses**: the outcome first, a fact a line, a name
  exact and in its canonical form, a fixed place for each kind of
  statement.
- **Data that is the product**, a document read, a set to go through:
  attached, typed by its own type, not spread into the answer, and not
  also said; large, pointed to.

## Core types

A core type and its variants: a design principle (the human,
2026-11-21): "we mean the same thing with more or less detail and the
detail depends on the context it is being used in"; "it will allow us
to deal with all sorts of extensions while at the same type have a base
type that can travel broadly. This constraint creates a dual namespace
from a single string: core type in the name, variant in the namespace
context."

- **One string, two parts**: a type's full name is
  `<namespace>.<name>`; the name is the core type, the namespace the
  context of its variant. `haiccing.basePackageRns.logEntry` and
  `splectral.baseComponentLogViewer.logEntry` are one core type,
  `logEntry`, in two variants.
- **One concept: a data structure of a functional context** (the human,
  2026-11-21: "I think there is only one core type: a data structure
  associated with a functional context"): the Kafka record is one, the
  request another, an action's answer another.
- **No one-to-one with protocols**: a package declares the core types of
  the contexts it holds, several or none (the human: "we don't have to
  enforce a one to one. We just add a type registry folder to a package
  and declare.").
- **Types are a package's, never a module's** (the human, 2026-11-26:
  "types are created once at package level, not module. schema
  contracts operate at package level and dock onto protocol actions and
  data structure but not the underlying code."): a contract is on what a
  package offers, its actions and its data; a module is code under it
  and declares none. The schemas the first build put in modules
  (`record`, `json`, `request`, `data`) are that build's, to be declared
  by packages.
- **A core type is unique in the family** (the human, 2026-11-25: "We
  will enforce that within a package family (base for the current)"):
  its name without a namespace is declared once, by one package; the
  package's namespace is its base variant's. A rule for what is
  written; its check is not built.
- **A variant keeps the core type's name**, the last segment of its full
  name, whether its schema is the base's, derived from it, or its own:
  the namespace alone says whose it is.
- **A base type travels broadly**: a reader holding the base reads every
  variant, their extra detail skipped; a variant reads the base, its own
  detail from its defaults. AVRO resolves a record by its name without
  the namespace, the core type, so this holds as AVRO is.
- **More or less detail, by context**: a variant extends its core type
  within AVRO's rules (fields added with defaults, fields left out,
  numbers widened, unions extended); where it is read decides how much
  detail is read.

## Names

A type's name is its URI, where it comes from; AVRO's name is derived
from it, behind the scenes (the human, 2026-11-25: "we can use URIs
within our system and carry a compatible AVRO name 'under the hood'").

- **Named by its URI**: `<org>/<package>/<name>`, a module's
  `<org>/<module>/<name>`: `haiccing/base-package-help/annotate-answer`.
  Each part a data name's URI form (`base-docs-uri`, "Data names").
- **Its AVRO full name by the data-name mapping**, exact both ways, each
  part its data name: `/` becomes `.`, each part its data name,
  `haiccing.basePackageHelp.annotateAnswer`; the name the core type, the
  namespace its variant. A schema whose name does not map is still a
  schema, known by its fingerprint only.
- **AVRO's own types by a URI too**, `avro/<name>` (`avro/string`,
  `avro/long`, `avro/bytes`, the eight primitives): no schema file, held
  by every mrepo without asking; so a type is always named by a URI.
  `avro` is reserved as a URI's first part (the human, 2026-11-27: "yes"):
  no organisation by that name; an `avro/` URI that is none of the
  eight is refused.
- **Named once, where it is declared**: the record and the request are
  used by every package and declared by one, the runtime package
  (`haiccing/base-package-runtime/record`). A type is a package's,
  never a module's ("Core types"); the first build's module names
  (`pear-full-square/mycelium-value-avro/record`) are not used.

## Fingerprints

- **A schema's identity its fingerprint**: the AVRO specification's
  CRC-64-AVRO (Rabin), 16 hex digits, of the schema's contract form:
  the schema whole, self-contained, without its descriptions. Not of
  AVRO's Parsing Canonical Form (the human, 2026-11-27: "can't you
  amend how the fingerprint is created?"): that form also leaves out
  defaults, aliases and logical types, which change how data is read;
  here they are part of the identity, only the descriptions are not. A
  schema never changes; a new version is a new schema, a new
  fingerprint, kept beside the old; the registry refuses another
  fingerprint under a URI it holds. The form includes the full name: a
  variant has its own fingerprint even where its fields are the base's.
- **What a fingerprint is for**: a schema's evolution over time, and
  what crosses between installs. Within an install a type's URI names
  one schema (the registry refuses another under it), so bytes there
  are read by the type their URI names and carry no fingerprint: an
  attachment's content.
- **A record names its own**, where bytes are kept over time or cross:
  a topic is not bound to one schema; each value carries its writer's
  fingerprint, AVRO's single-object encoding (two marker bytes, `C3
  01`, the fingerprint's 8 bytes, the value). The first build's; the
  plain form, the value's bytes alone, is `mycelium-value`'s.

## The cascade

Schemas cascade (the human, 2026-11-21: "there is the kafka record, the
request (plus response), and then within potentially many other schemas
optional depending on the context"): each level a schema of its own,
self-contained, knowing nothing of the one around it. As built, the
runtime package's types (`haiccing/base-package-runtime`, `types/`):

1. **The record**, the transport's envelope, Kafka's: `topic`, `key`,
   `offset`, `timestamp`, `headers`. Its base holds no value: a
   stream's record with its value is a variant of it, its value the
   stream's type. Read by what carries records.
2. **The request**, self-contained: its `command`, the line as typed;
   who asked and where its answer goes (`from`, `to`, `reply`,
   `origin`); its `form`, the command as understood, its flags a record,
   a field a flag; its `processing` (log, metrics, what it changed, what
   it waits for, the request it became where it was sent on, a link);
   what is `returned`: `done` or `failed`, a message in natural
   language, its attachments.
3. **An attachment**, data that is the product, a wrapper, one fixed
   type: its `name`, its `type` (a type's URI, always), its `content`
   (AVRO's binary of the value by that type; base64 text in JSON), or
   where its value is when it is not carried (`at`). The one place a
   value names its type. Written and read under the hood: an action
   hands over a value and is handed a value; an mrepo that passes an
   answer on carries the bytes and needs nothing of their type.

Knowledge stays with its concern; each level versions on its own. A
request sent on to another mrepo is a request of its own there: linked,
not copied.

## In a package

A package carries its types; where it is deployed they are known.

- **`types/`, its core types**: `types/<name>.avsc`, the base schema of
  each core type the package declares, a `doc` on it and on every field;
  a `README.md` beside them, a line each
  (`haiccing/base-package-rns/types/lookup-entry.avsc`,
  `haiccing.basePackageRns.lookupEntry`). Carried; known where the
  package is placed through the schema registry (`base-docs-vision`,
  "Type system" and "Solutions": what is agreed, what is built).
- **`schemas/`, the first build's form**: `schemas/<name>.json`, one
  schema a file; a module's in its own `schemas/`. No longer registered
  at deploy (removed with the request's new structure, 2026-11-26);
  still carried by the packages not yet brought to `types/`. The
  pattern's how-to: `base-docs-package`, `schemas`.
- **Read with `info`**: `info/get schemas`, every schema an mrepo's
  packages carry, by URI and fingerprint; a binary record is decoded by
  `crud/read`, every level.

## The registry

The first build's (`mycelium-value-avro`), an install's index by
fingerprint in its store: its code is there, nothing writes it since
the registration at deploy was removed (2026-11-26). The registry now
is the component, `base-component-avro-registry`: `base-docs-vision`,
"Type system". What the first build settled about a registry, kept:

- **What a fingerprint is, what a name has been**: schemas by their URI
  and by their fingerprint, in the install's store (`<store>/schemas`),
  a folder of plain JSON documents, one a schema, named by its
  fingerprint, the schema whole (every named type it uses defined in
  it). The first build's store; the component keeps its types in its
  `main`, a file a type at its URI, and no process reads a folder for a
  type.
- **No subjects bound to topics, no global ids, no compatibility
  modes** (the first build never refused a valid schema; the component
  refuses one thing: a changed schema under a URI it holds). Whether data reads as a schema is
  the reader's question.
- **Three questions**: the core types there are; a core type's variants
  (by namespace); a variant's versions (by fingerprint, with the commit
  that brought it, in the order registered). No version numbers.
- **Only what is written with**: a reader's schema is not registered; a
  reader's type is made against the names the registry knows.

## Is and is-like

Compatibility is the reader's question, at read time (the human,
2026-11-21: "does this schema allow me to read it in 'this schema
way'? The is-like question, rather than is."): AVRO's resolution, the
writer's schema from its fingerprint against the reader's; fields by
name, the writer's extra skipped, the reader's missing from their
defaults, numbers widened, unions resolved.

- **Is**: one core type, its versions and variants, read as AVRO reads
  it, a record matched by its name without the namespace, as the
  specification says; or a declared alias, a type renamed or moved
  keeping its old name.
- **Is-like**: a different core type read as another, only when the
  reader asks: aliases generated for that read, at the same positions
  (a record's fields by name, an array's items, a map's values, a
  union's branch by structure, the first that fits), then the same
  resolution. The aliases applied are part of the answer.
- **Aliases only between core types**: never needed within one.

## Typed topics

The human, 2026-11-21: "Better to work in typed topics to keep
structure. The structure of every log is the same at the kafka level,
but the type is the value type - that is the data payload".

- **A Kafka record's structure**: every topic's records are the record
  ("The cascade"); what a topic holds is its value's core type.
- **Named by its type**: `request-summary`, never a generic word
  ("log"); whose it is, is where it is (a node mrepo's
  `request-summary` topic).

## What stays JSON

- **Documents in git**: plain JSON, validated as values against their
  schema.
- **The request layers, the inbox and the outbox**: JSON documents, a
  record around a request, checked at the inbox by their types, the
  mrepo's own, in its `avro` layer, strict. The same structure in JS,
  JSON and AVRO. An attachment's content inside them is AVRO's binary,
  as base64 text.

## What is next

Built since: the registry as a component, its type data in its `main`;
the `avro` layer in every mrepo, the runtime's three essential types in
it; an attachment carrying its content, converted under the hood. Agreed and not built: a runtime asking the
registry for a type it does not hold, through the link in its `avro` layer; variants; the
`avro` package; `mycelium-value-avro` retired into `mycelium-value` as
the parked packages are brought over. The vision, `base-docs-vision`,
"Type system" and "Solutions".
