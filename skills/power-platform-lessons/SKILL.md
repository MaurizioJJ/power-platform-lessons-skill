---
name: power-platform-lessons
description: "Field-tested lessons for Power Apps (canvas and modern controls), Power Fx, Power Automate and Dataverse. Use before writing any Power Fx formula, .pa.yaml, flow JSON, Dataverse query or Studio paste instructions, and when debugging delegation warnings, PA2108/PA2109 paste errors, modern-control property errors, flow schema errors or Dataverse/TDS query failures."
---

# Power Platform Lessons

Hard-won, field-tested lessons for building on Power Apps (canvas, modern controls), Power Fx, Power Automate and Dataverse. Microsoft Learn is often behind or silent on these; check here first.

## How to use this skill

1. **Before writing any formula, YAML, flow JSON or query:** check the matching section below and apply every relevant lesson.
2. **Trust levels:** a lesson tagged **Verified** (confirmed on Microsoft Learn or tested live) beats general training knowledge. A lesson tagged **Inference** (seen in practice, cause not proven) must be stated as an inference if you rely on it.
3. **Versions matter:** modern controls change between versions. The lessons name the version they were seen on; check the version the app actually uses (its own `.pa.yaml` or `References/Templates.json`) before writing a property.
4. **Project lessons:** if the project keeps its own lessons file (for example `docs/power-platform-lessons.md`), read it at task start. At session end, add new lessons there in the same format: one line, the lesson, then **Verified** or **Inference**. Search first and extend an existing line instead of duplicating; when a new lesson contradicts an older one, keep both and say which wins and why.

---

## Power Apps canvas and modern controls

Modern controls differ by version from what Learn shows: check the app's own uses before writing a property.

- **ModernText@1.0.0** has no `Tooltip` (PA2108 on paste). For a tooltip on text, overlay a transparent ModernButton. — Verified
- **ModernButton@1.0.0** has no `IconPosition` (PA2108). `Tooltip` is supported. Removing `ButtonLayout.TextOnly` lets the `Icon` show. — Verified
- **ModernRadio@1.0.2** has no `Tooltip`; put an info ModernIcon beside it. — Verified
- **ModernDropdown:** input `Default` takes a record (`{Value: x}`, never `Blank()`), output `Selected`; no `Tooltip`; `ItemDisplayText` accepts only a column (no UDF, no `If`), so add a label column to Items. It can show its Default while `Selected` is stale: filter on a variable. — Verified
- **Modern inputs (TextInput, Dropdown, Toggle, Checkbox):** `Default` / `Items` cannot be read from another control ("isn't recognized"); read only outputs (`Text`, `Selected`, `Checked`, `SelectedDate`). — Verified
- **ModernToggle / ModernCheckbox:** `Default` is the input, `Checked` is the output (breaking change from the PCF Toggle). — Verified
- **ModernTextInput@1.1.1:** output `Text`, `Placeholder` (not HintText), no `OnSelect`, no Number type. `Reset()` is unreliable while the input holds focus at `OnSelect`; `TriggerOutput` has only `Delayed` and `KeyPress`. — Verified
- **ModernDatePicker@1.0.1:** `DefaultDate` in, `SelectedDate` out, no clear button; clear with a flag plus `Reset`. No `AccessibleLabel` or `Tooltip`. — Verified
- **ModernIcon@1.1.1:** `Icon` is a string, color is `IconColor`; the `IconStyle` enum is `IconStyle.Outline` (default) / `IconStyle.Filled`. There is no "Regular". — Verified (Learn)
- **Icon names:** only names from the Fluent icon list the platform ships are valid (for example no `PauseCircle`, no `SubtractCircle`; `LockClosed`, not `Lock`). — Verified
- **ModernTabList:** Items must keep exactly the record shape its Default reads; an extra column breaks Default and `Selected.Key`. Tab lists keep their tab between screen visits: `Reset` them in OnVisible. — Verified
- **ModernCombobox** has no `Tooltip` (PA2108), only `AccessibleLabel`. It keeps its selection when `Items` change: `Reset` it from every control its Items filter on. `Reset` restores `DefaultSelectedItems`, so a derived default repopulates instead of clearing. `ItemDisplayText` rejects `Coalesce()`: plain column only. — Verified
- **Read-only ModernCombobox** (`DisplayMode.View`) fed only by `DefaultSelectedItems` from a Dataverse multi-choice column showed empty, and `Concat(SelectedItems, Value, ",")` reached the flow as null. `JSON()` writes a blank as null, not `""`. Look at what the box shows before trusting `SelectedItems`. — Verified (payload) / Inference (cause)
- **Galleries keep their selected row** between visits and after Items change. "Sorts then snaps back" or "opens at the bottom" means a retained selection: `Reset(gallery)`. — Verified
- **Gallery@2.15.0 variants:** `Vertical`, `Horizontal` or a full `BrowseLayout_…_ver5.0` name; `BrowseLayout_Vertical` alone raises PA2109. — Verified
- **Modern renames vs classic:** `FontSize`→`Size`, `FontColor`→`Color`, `FontItalic`→`Italic`, `BorderRadius`→four `Radius*` properties. ModernButton@1.0.0 has no `BorderStyle`, `VerticalAlign` or `AutoDisableOnSelect`. — Verified
- **Legacy enums survive in modern buttons:** `'ButtonCanvas.*'` values left from older builds must become `ButtonAppearance` / `IconStyle` / `ButtonLayout`. — Verified
- **Label@2.5.1** has no `Radius*` and no `AccessibleLabel`. **HtmlViewer@2.1.0** has no `OnMessage`. **GroupContainer@1.4.0** in ManualLayout has no `BorderRadius`. — Verified
- **Components:** a control inside a component cannot reference a sibling in the same component; `ThisItem` fails on a gallery template's root container inside a component; instance inputs cannot read screen context variables (use a global); `Set` inside an instance event does not type the variable, so define it on the screen too. — Verified
- **Components, state and navigation:** no `UpdateContext` inside a component; keep instance state in a one-row collection. With `AccessAppScope: true` a component can read app collections. A nav menu that only calls `Navigate(ThisItem.Screen)` silently does nothing for a link entry (`Navigate(Blank())` is silent): branch to `Launch()` when a link field is set. — Verified
- **Message-box component height:** a blank fixed height came out too tight and scrolled. Size each instance from its data (rows × line height, capped at a share of `App.Height`). Wrapped ModernText with AutoHeight seemed to under-report its height. — Inference (cause)
- **Attachments control** inside a Form with a blank Item exposes no files; use a standalone Attachments control in a container. — Verified
- **Switching the record that feeds several Defaults** (`Set(varRecord, ThisItem)`): `Reset` every input whose Default reads it. An input also resets when its Default changes. — Verified (Learn)
- **Replacing a group with a component** deleted the controls inside that group. A copied header also brought another screen's title and a `Back()` that skipped an unsaved-changes check. Compare the screen's control list before and after. — Verified

## Power Fx, delegation and named formulas

Aim for zero delegation warnings; bind every value before the Dataverse query.

- **Bind first:** inside a Dataverse `Filter` / `LookUp`, compare columns only with plain values bound earlier in `With()`: no `Text()`, `Value()`, `Lower()`, `GUID()` or `var.field` inside the condition. — Verified
- **No column-less comparison in a Dataverse filter** (`Cur > 1`, or an `If()` choosing between two Filters inside `SortByColumns`): the server refuses it. Turn a switch into a bound value compared with a column (`ShowAll || IsActive = ActiveFlag`). — Verified
- **Several audiences in one list:** separate `Filter()` calls chosen by `If()` work; OR-ing audiences through bound flags returned no rows. Each privileged branch must still include the user's own records. — Verified
- **SortByColumns delegates** only with a literal column name and a literal `SortOrder`; a direction held in a variable warns, so write one call per direction. — Verified
- **Dataverse delegation (Learn):** `in` (substring) and `StartsWith` both delegate on Text columns; `IsBlank` does not delegate on Choice columns; `Text(column)` never does. — Verified (Learn)
- **Named formulas** over Dataverse re-evaluate after an in-app `Patch()` without `Refresh()`. A filter shared by several controls belongs in one named formula. — Verified
- **`Table(t1, t2)` concatenates** tables; `Ungroup(Table({R:t1},{R:t2}), R)` showed an empty part as a blank gallery row. — Inference (cause)
- **Display-name mode:** Studio rewrites logical names for direct data-source references, not for `var.Field` or record literals; write those with display names. Shadow columns (attributeof set) are invisible to Power Fx. — Verified
- **Typed blanks:** `If(false, 0)` / `If(false, First(Table))` give a variable its type without a value. — Verified
- **Rules by level, not by name:** role checks use numeric levels (`>=`), status checks use values; never a bare role or status name in code. — Verified
- **"Nothing assigned yet" ≠ "not started":** gate on the record status, not on the absence of child rows. — Verified
- **Colors in HTML:** `Mid(JSON(Theme.Color), 2, 7)` turns a color value into `#rrggbb` for CSS. — Verified (Learn, JSON)
- **Comments:** `/* */` only, never `//`. — Verified
- **`IfError` will not unify branches of different types** (a Patch returning a record vs a notification). Use `If(IsError(…), …)`. `FirstError` exists only inside `IfError`; with `IsError`, read `Errors(DataSource)` before the next data call clears it. — Verified
- **`ClearCollect`, `Clear`, `Set` and `UpdateContext` are illegal inside `ForAll`.** — Verified
- **`Split()` and `ForAll(table, scalar)`** return a column named `Value`. — Verified
- **Option sets:** compare by value (`'Category (Orders)'.Retail`), never `Text(x) = "Retail"`. `=` on text is case-sensitive in Power Fx while the Dataverse SQL endpoint is case-insensitive, so a SQL match proves nothing about Power Fx. — Verified
- **`ShowColumns(Table, "logical_name", …)`** with strings gives columns named by logical name, so display-name formulas find nothing. Use identifiers (`ShowColumns(Orders, OrderNumber, Title)`). — Verified
- **`CountRows`** rejects a UDF's table return type even when it is declared as a table. — Verified
- **A control reset earlier in the same formula** reads its reset value later in that formula: derive the condition from data captured before the reset (for example from the record key, not the toggle). — Verified
- **`Patch(DS, Defaults(DS), {...})` returns the created record:** `Set` it and bind child rows to it, instead of a `LookUp` on an unindexed text key. — Verified
- **`JSON()` text sent through a flow text input can arrive double-encoded** (a string that holds JSON): the receiver must parse again while the result is still a string. — Inference

## Studio paste, YAML and tooling

Check every paste in the Tree view and the property pane: Studio can place it, or change it, unexpectedly.

- **Paste target:** with a container or gallery selected, the paste goes inside it (a gallery repeats it on every row). To paste a sibling, select a non-container sibling first. — Verified
- **Tree view: top = front.** The first child in the Tree is drawn on top (it is last in the YAML). — Verified
- **Studio can change values on paste** into a gallery or a container mid-resize (a button arrived with Height 1 and no X/Y; a cell lost its Width). After every paste, check Width / Height / X / Y. — Verified
- **Control names are app-wide unique:** pasting a control that still exists gives `name_1`. Delete first, then paste. — Verified
- **Wrong-property paste:** "Expected Enum value" / "Expected Record value" usually means the formula landed in the neighboring property (`VerticalAlign` next to `Visible`). — Verified
- **YAML files:** one control per file, starting at column 0; no `# ***` banner on screen YAML; `Gallery`, `GroupContainer` and `TypedDataCard` need `Variant`; `Form@2.4.4` needs `Variant` plus `Layout: Vertical`; `CanvasComponent` needs `ComponentName` (PA1011). — Verified
- **YAML `key: value` inside a block scalar** gives `YamlInvalidSyntax`; use `|-` or `{key:"value"}`. — Verified
- **Property files** carry the raw expression: no leading `=`, no YAML wrapper. Values extracted from `.pa.yaml` start with `=`, so strip it and check the first character of every delivered file. Deliver Power Fx as `.txt`; `.fx` files did not preview. — Verified
- **Validating `.pa.yaml` in Python:** the bare `=` is a YAML tag, so subclass `SafeLoader` with a constructor for `tag:yaml.org,2002:value`; never only regex-substitute it (that masks PA1001). Formulas can also contain TAB characters, which PyYAML rejects: use `yaml.BaseLoader` or line-based extraction. — Verified
- **Some properties are JSON-quoted single-line scalars** (every `"` written `\"`): decode the scalar before concluding a grep found or missed something. — Verified
- **A grep hit can sit inside a `/* */` comment:** read the context before calling a reference live. — Verified
- **Solution exports carry only published components.** Use Studio "Download a copy" for the latest saved state, and check an export contains the latest delivered changes before building from it. — Verified
- **Publish can land in the Test slot** while Live stays on an old version, and an unmanaged export carries only Live. Before exporting, check the Live badge (Apps → Details → Versions; promote with "Publish this version"). After exporting, compare `AppVersion` in `customizations.xml` with the last publish. — Verified (symptoms; Learn: exports carry published only) / Inference (Test-slot mechanism)
- **Whole-screen YAML silently reverts** every fix made after the export it was built from: merge those fixes first, or deliver per property. — Verified
- **A copied or staged solution folder can be incomplete:** list the source folder itself before saying a file is missing. — Verified
- **App-wide search/replace** also hits data writes, labels and mail text: use exact multi-token strings with expected hit counts. — Verified
- **Diagnose by evaluating:** ask for the formula-bar result in Studio instead of guessing. — Verified
- **`ErrOpeningDocument_UnknownError`** when opening an app in Studio: switching back to the previous edit tool opened it. — Verified workaround, cause unknown
- **`<flow>.Run failed: InternalServerError` with no run in the flow history,** after many flow add/remove cycles, is a stale Studio session: save, publish, close Studio and reopen. — Verified
- **A diff showing only a control version bump** (for example ModernDataGrid 1.5.2→1.7.0) with no property change is Studio's own upgrade on open; no action needed. — Inference

## Layout, theming and UI consistency

Size everything from parents and theme tokens; nothing literal.

- **No literal colors or pixels on controls:** bind theme tokens (for example an `AppTheme` named formula); RGBA only inside App.Formulas. Add a missing token to the theme first. — Verified
- **Theme tokens can alias each other** (`Secondary` = `Error`, `Success` = `Primary`): check the value before using a token to carry meaning. — Verified
- **Auto-layout children:** fixed child = `FillPortions 0` plus an explicit size; flexible child = `FillPortions 1`; always set `LayoutMinHeight 0` and `LayoutMinWidth 0` on pasted controls (the default min height of 100 pushed content out of a 40 px header). `FlexibleHeight` does not exist. — Verified
- **Classic Gallery in an auto-layout row:** `Height = Parent.Height` made the row grow to about 400 px; give it a fixed height. — Inference (cause)
- **Column alignment:** header, filter row and gallery row need the same FillPortions, gap and padding; a fixed cell (such as Actions) must exist in all three or be replaced by equal padding. — Verified
- **Inside a component,** a child chain that feeds the root size must use `App.Width` / `App.Height`, not `Parent` (circular reference). — Verified
- **`Parent.LayoutGap` fails** when the pasted control lands outside its container; prefer spacing tokens in widths. — Verified
- **Dynamic font sizes must be clamped** (`Max(min, Min(computed, max, Parent.Height * 0.6))`); an unclamped ratio overflows on wide or narrow screens. Route all text sizing through one UDF. — Verified

Recommended UI conventions (team style, adopt or skip):

- One look per button role (confirm, save, cancel, destructive), with the same radius, height and font, and an icon on every button. Cancel/close uses the Secondary appearance, is slightly shorter and narrower than the primary, and the same action always gets the same icon (Reset/Clear = Eraser, Refresh = ArrowClockwise).
- Zoom/info buttons with nothing to show are disabled, not hidden.
- One message-box component and one toast per screen instead of `Notify()` and ad-hoc popups.
- One status legend for the whole app (icon plus color per status), reused in every list.

## Power Automate, connections and mail

A green run is not proof: check the catch path.

- **Try/catch hides failures:** a failed action inside a scope with a catch still shows the run as "Succeeded". Read the catch output or error mails. — Verified
- **`result()` works only on Scope actions;** for a single Dataverse action read `actions('X')['status']` and `actions('X')['error']['message']`. Dataverse actions expose no `statusCode`. — Verified
- **Shared-mailbox send 404 `ErrorItemNotFound`** means the connection account lacks Full Access plus Send As on that mailbox, or the address is wrong. — Inference, consistent with Learn
- **Outlook connection for shared-mailbox sends:** use the flow's own embedded connection reference, never "Provided by run-only user"; the invoking user usually has no Send As. Use the invoker's connection only when mail should go out as that user. — Verified
- **OAuth connections** cannot be shared with a person, only with a service principal; the connection owner turns the flow on once. — Verified (Learn)
- **Error and admin mails** come from a shared mailbox, not a personal one. They name the record, the failed step, the message and a run link; never dump `result()`. — Verified
- **Power Automate Tools (XrmToolBox) JSON:** the three-key document (`$schema`, flat `connectionReferences` with `connectionName` plus `connectionReferenceLogicalName`, and `definition` with `description`), never the solution-export shape. A missing `definition.description` gives an opaque ARM BadRequest. Build from the flow's own Tools export. — Verified
- **Converting a solution export to Tools JSON:** keep `properties.definition` and replace `properties.connectionReferences` with the live Tools block. Tools references carry no `authentication`, while export actions carry `@parameters('$authentication')`; never mix the two shapes. — Verified
- **Tools `connectionReferences.connectionName`** is an environment-specific connection id: take it from a flow in the same environment. Another environment's value fails with "Cannot find connection with name". — Verified
- **Descriptions:** action `description` ≤ 256 characters (`ActionDescriptionTooLong`), but the trigger fails at about 90 (`TriggerDescriptionTooLong`), so keep in-definition descriptions under 80. Flow description < 1024 characters. — Verified
- **PowerApps-triggered flows:** trigger `manual`, kind `PowerAppV2`, input keys `text` / `text_1` / `number` / `file` (friendly name in `title`), otherwise `FlowWadlConversionNotSupported`. — Verified
- **Optional PowerAppV2 inputs are omitted when blank:** read them as `triggerBody()?['text_1']` with `coalesce`. Required inputs are positional in `.Run()`. — Verified
- **Two `Respond to a PowerApp` actions** in one flow need byte-identical schemas, otherwise `ActionSchemaInvalid`. — Verified
- **Power Apps strips spaces from flow names:** "Orders - Export" is called as `'Orders-Export'.Run()`. A recreated flow is a new object: remove and re-add it in the app. — Verified (names) / Inference (re-add)
- **Variables:** one per Initialize action; `Initialize variable` is illegal inside Scope, Apply to each or parallel branches; `Set variable` rejects an empty value. — Verified
- **Nested Apply to each:** `item()` is the inner loop, so outer values need `items('OuterLoop')` or they return null silently. `Select` cannot expand one item into many. Avoid Apply to each where an array expression does the job, including single-record lookups. — Verified
- **Fallback values:** an expression that reads an action skipped inside a Condition fails, so run the cheap fallback lookup always and pick with `if(empty(…))`. — Inference
- **Dataverse file columns:** use `GetEntityFileImageFieldContent` / `UpdateEntityFileImageFieldContent` (entityName, recordId, fileImageFieldName). — Verified
- **Read what the app sent** from the flow's trigger (run → trigger card → Show raw outputs → `body.text_N`). Never open or paste an action whose inputs carry a secret (for example a custom-connector call with a `code` query parameter). — Verified
- **Never delete a flow or connection reference the Live app version depends on.** The delete fails ("Unable to delete flow with declared dependencies"), and publishing fails with `0x80095005` when a flow it activates points to a deleted connection reference. Repoint the flow, save, then publish. — Verified
- **Custom connector in a canvas app:** every user must create their own connection to it, and the app does not pick up connection references on custom connectors, so after import remove and re-add the data source under the same name. Keep secrets (function keys) out of the app: the app calls a flow, the flow calls the connector. — Verified (Learn for the connection; tested for the re-add)
- **Custom connector host from an environment variable** (`@environmentVariables("…")`): the variable needs a value before "Update connector", and the value is read at save, so re-save the connector after changing it. Ship the connector in its own solution, imported first. — Verified
- **A UI promise ("you will be notified") needs a real sender:** check that a flow actually triggers on that status before trusting it. — Verified

## Dataverse, SQL 4 CDS, XrmToolBox

Check the column type and name in the metadata before every query.

- **Use logical names** in queries and flows. A text key compared with a number key on another table needs `Value()` on the constant side. — Verified
- **Entity set names** can be irregular plurals (`…_detailses`); look them up, don't guess. — Verified
- **Lookups in raw Web API calls and flows:** `field@odata.bind` = `/entityset(guid)`, with the navigation property's exact case (`pub_Order`, not `pub_order`). — Verified ($metadata)
- **Alternate keys** are never copied to a new row. They enforce uniqueness; they are not general-purpose indexes. Indexes help only server-side: a Filter array after List rows wastes them. — Verified
- **After adding columns,** refresh the data source in the app before referencing them. — Verified
- **TDS / SQL endpoint:** file columns cannot be selected or filtered (use Admin Center capacity). A stale "Invalid object name" means refresh metadata. No CTEs, `CROSS APPLY`, `INFORMATION_SCHEMA` or `sys.columns`; collation is case-insensitive. — Verified
- **TDS endpoint writes:** documented as read-only, yet INSERT and DELETE ran in at least one environment. Do not rely on either behavior. — Verified (tested live)
- **SQL 4 CDS:** qualify `dbo.<table>`; use `COUNT(column)`, not `COUNT(*)`, in grouped queries; no `TRY_CONVERT(date, x, style)`; a join on `CAST()` runs one retrieval per row, so read each side once and `HASH JOIN`; paste into a cleared window. — Verified
- **Bulk DELETE** may queue as async bulk-delete jobs: check Data management before assuming it failed. For bulk actions, check for an XrmToolBox tool first. — Verified
- **Delete children before parents** when clearing data. — Verified
- **`callbackregistration`** is environment-scoped: filter by entity name. — Verified
- **Reading the `Solutions` table** needs `prvReadSolution`; ordinary users get a connector error banner that `IfError` cannot suppress, so do not read it at app start. — Verified

## Working method

One change at a time, from the source, with exact paste steps.

- **Read the source first** (the latest export, highest version number), then Learn, then ask. Never invent identifiers or behavior; label anything unproven as Inference. — Verified
- **One change per response,** naming the target by its full Tree path. Give each requested change an id and keep a register of them. — Verified
- **Multi-file deliveries** get numbered paste steps naming the exact control and property; state where a new control goes (parent, siblings) and who sees it. — Verified
- **Moving controls:** deliver the full container YAML with a delete-then-paste process, not manual cut and paste. — Verified
- **A full-property replacement** is built from the newest version (the last delivered file, else the export); otherwise earlier fixes are lost. — Verified
- **Before working from a new export,** diff it against the previous one control by control (added, removed and changed properties per tree path) and list every change you cannot explain. A per-file diff hides moves and renames. Keep the previous export beside the new one. — Verified
- **Copied screens keep copied bugs:** a tab bar or dialog copied between screens or apps carries its tooltips, accessibility labels and redirects. When fixing one copy, search every screen for the same controls. — Verified
- **Ambiguous tester notes:** ask one question that separates the readings before coding. — Verified
- **Two failed theories = stop and instrument** (a probe label showing the evaluated value) before a third paste. — Verified
- **Check a theory about where a bug came from against the baseline** before writing it down. — Verified
- **Never claim a delivered file matches the live one** without diffing the user's own copy; comparing two of your own copies proves nothing. — Verified
- **Session end:** write a short handover (change register, open items, export checklist) and record new lessons in the project's lessons file. — Verified
