# [svelte-reparent](https://tristan-f-r.github.io/svelte-reparent)

[![npm](https://img.shields.io/npm/v/svelte-reparent)](https://npmjs.com/svelte-reparent)

Reparent elements with ease. Svelte non-internal using alternative to [react-reparenting](https://github.com/paol-imi/react-reparenting).

## Example

(See it on [Svelte REPL](https://svelte.dev/repl/0ebf76d9cbb347fd8c2639f3c3825eba?version=4.2.1)!)

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import { Portal, Limbo, teleport } from 'svelte-reparent';

	let component: HTMLElement;

	function send(label: string) {
		return () => teleport(component, label);
	}

	onMount(send('a'));
</script>

<main>
	<Limbo bind:component>
		<input placeholder="Enter unkept state" />
	</Limbo>
	<div class="container">
		<h1>Container A</h1>
		<Portal key="a" {component} />
		<button on:click={send('a')}>Move Component Here</button>
	</div>
	<div class="container">
		<h1>Container B</h1>
		<Portal key="b" {component} />
		<button on:click={send('b')}>Move Component Here</button>
	</div>
</main>
```

## Features

- No need to worry about keeping state in sync between components.
- Ownership model prevents bugs where components are destroyed while still in use.
- No dependencies on internal svelte APIs, unlike React and Vue alternatives.
- Simple API with only three exported functions.

Since this library is relatively new, there may be bugs. (Please report them! Every bug report helps!)

## Alternatives

If you're trying to teleport a node to a different location in the DOM, you can use
[svelte-portal](https://npmjs.com/package/svelte-portal) instead. This library
is better suited for teleporting _inside_ the svelte app, rather than outside.

## How it works

This library is split into three main concepts:

- `Limbo`, which serves as the "owner" of a component to be teleported.
- `Portal`, which serves as the "receiver" of a component to be teleported, and displays it.
- Teleportation, which transfers borrowship of a component from one `Portal` to another.

The `Limbo` component is the "root" component of the `Portal` component. There is a global
registry, which maps component instances to what portal ID they belong in. When a `Portal`
is destroyed, it is moved back to `Limbo` and removed from the registry.

In order to move the DOM around, this library extensively uses `<div style="display: contents">`.
The usage of this allows for `svelte-reparent` to _ensure_ that svelte components
have a single root element, which is moved around (in the case of `Limbo`), or
appended to (in the case of `Portal`).
