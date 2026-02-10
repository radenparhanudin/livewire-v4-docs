# 🚀 Livewire Documentation Roadmap

## 📌 Prologue & Getting Started

* [Quickstart](docs/quickstart.md)
* [Installation](docs/installation.md)
* [Upgrade Guide](docs/upgrade-guide.md)
* [Troubleshooting](docs/troubleshooting.md)
* [Contribution Guide](docs/contribution-guide.md)

---

## 🏗️ UI Components

### Essentials

* [Components](docs/components.md) | [Pages](docs/pages.md) | [Properties](docs/properties.md)
* [Actions](docs/actions.md) | [Forms](docs/forms.md) | [Events](docs/events.md)
* [Lifecycle Hooks](docs/lifecycle-hooks.md) | [Nesting Components](docs/nesting.md) | [Testing](docs/testing.md)

### Features & Optimization

* [Alpine Integration](docs/alpine.md) | [Styles](docs/styles.md) | [Navigate](docs/navigate.md)
* [Islands](docs/islands.md) | [Lazy Loading](docs/lazy-loading.md) | [Loading States](docs/loading-states.md)
* [Validation](docs/validation.md) | [File Uploads](docs/file-uploads.md) | [Pagination](docs/pagination.md)
* [Query Parameters](docs/url-params.md) | [Computed Properties](docs/computed.md)
* [Redirecting](docs/redirecting.md) | [File Downloads](docs/downloads.md) | [Teleport](docs/teleport.md)

---

## 🛠️ HTML Directives (`wire:`)

* **Data & Actions:** [`wire:model`](docs/wire-model.md) | [`wire:click`](docs/wire-click.md) | [`wire:submit`](docs/wire-submit.md) | [`wire:bind`](docs/wire-bind.md)
* **UI States:** [`wire:loading`](docs/wire-loading.md) | [`wire:dirty`](docs/wire-dirty.md) | [`wire:transition`](docs/wire-transition.md) | [`wire:cloak`](docs/wire-cloak.md)
* **Navigation:** [`wire:navigate`](docs/wire-navigate.md) | [`wire:current`](docs/wire-current.md)
* **Lifecycle:** [`wire:init`](docs/wire-init.md) | [`wire:intersect`](docs/wire-intersect.md) | [`wire:poll`](docs/wire-poll.md) | [`wire:offline`](docs/wire-offline.md)
* **DOM Control:** [`wire:ignore`](docs/wire-ignore.md) | [`wire:ref`](docs/wire-ref.md) | [`wire:replace`](docs/wire-replace.md) | [`wire:show`](docs/wire-show.md) | [`wire:stream`](docs/wire-stream.md)
* **Others:** [`wire:confirm`](docs/wire-confirm.md) | [`wire:sort`](docs/wire-sort.md) | [`wire:text`](docs/wire-text.md)

---

## 🏷️ PHP Attributes

* **State Management:** [`#[Url]`](docs/attr-url.md) | [`#[Session]`](docs/attr-session.md) | [`#[Locked]`](docs/attr-locked.md) | [`#[Modelable]`](docs/attr-modelable.md)
* **Reactivity:** [`#[Reactive]`](docs/attr-reactive.md) | [`#[Computed]`](docs/attr-computed.md) | [`#[On]`](docs/attr-on.md) | [`#[Validate]`](docs/attr-validate.md)
* **Performance:** [`#[Lazy]`](docs/attr-lazy.md) | [`#[Isolate]`](docs/attr-isolate.md) | [`#[Defer]`](docs/attr-defer.md) | [`#[Async]`](docs/attr-async.md)
* **UI/Render:** [`#[Layout]`](docs/attr-layout.md) | [`#[Title]`](docs/attr-title.md) | [`#[Renderless]`](docs/attr-renderless.md) | [`#[Transition]`](docs/attr-transition.md)
* **JS Bridge:** [`#[Js]`](docs/attr-js.md) | [`#[Json]`](docs/attr-json.md)

---

## 🧩 Blade Directives

* [`@island`](docs/blade-island.md)
* [`@placeholder`](docs/blade-placeholder.md)
* [`@persist`](docs/blade-persist.md)
* [`@teleport`](docs/blade-teleport.md)

---

## 🛡️ Advanced & Security

* **Core Concepts:** [Morphing](docs/morphing.md) | [Hydration](docs/hydration.md) | [Synthesizers](docs/synthesizers.md)
* **Security:** [Security Overview](docs/security.md) | [Content Security Policy (CSP)](docs/csp.md)
* **Integration:** [JavaScript API](docs/javascript-api.md) | [Package Development](docs/package-dev.md)
