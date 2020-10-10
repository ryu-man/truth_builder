<script>
	import { fade } from "svelte/transition";
	export let name;
	let input = "";
	/**@type {Set<string>}*/
	let variables = new Set();
	let rows = [];
	let body = "";
	let data = [];
	let res = [];
	let exp = []; // list of operations
	// operation is an objet which have one or two args and retun a boolean
	let errors = [];
	$: if (variables) {
		data = prepare();
	}

	function prepare() {
		let temp = [];
		for (let i = 0; i < 2 ** variables.size; i++) {
			const row = [];
			for (let j = 0; j < variables.size; j++) {
				row.push(Math.floor(i / 2 ** (variables.size - j - 1)) % 2);
			}
			temp = [...temp, row];
		}
		return temp;
	}

	/**
	 * @param {string} value -
	 * */
	function parser(value) {
		res = [];
		data.forEach((e) => parse(value, e));
	}
	/**
	 * @param {string} value -
	 */

	function validate(key) {
		if (/[()+*a-z]/s.test(key)) {
			return true;
		}

		return false;
	}
	/**
	 * @param {string} value -
	 */
	function isSyntaxValid(value) {
		const openedBrackets = value.match(/\(/g)?.length || 0;
		const closedBrackets = value.match(/\)/g)?.length || 0;
	
		return openedBrackets == closedBrackets;
	}
	/**
	 * @param {string} exp -
	 * @param {Array<number>} values -
	 */
	function parse(exp, values) {
		const regex = /\s?(\d)\s?([*+])\s?(\d)\s?|()([!])(\d)/g;
		let newExp = exp;
		let array = Array.from(variables);
		for (let i = 0; i < array.length; i++) {
			newExp = newExp.replace(array[i], values[i]);
		}
		while (newExp.search(/[+*!]/g) > 0) {
			newExp = simplify(newExp);
			newExp = newExp.replace(regex, function (
				match,
				left,
				operator,
				right,
				offset,
				string
			) {
				if (operator == "+") {
					return +left || +right;
				} else if (operator == "*") {
					return +left && +right;
				} else if (operator == "!") {
					return +right == 0 ? 1 : 0;
				}
				return 0;
			});
		}
		res = [...res, +newExp];
		/**
		 * @param {string} exp -
		 */
		function simplify(exp) {
			return exp.replace(/\(\s?(\d)\s?\s?\)|\(\s?(\d)\s?\s?\)/g, "$1");
		}
	}
</script>

<style>
	@import "../node_modules/bootstrap/dist/css/bootstrap.css";
	table th,
	table td {
		transition: all 0.4s;
	}
	/* table {
		border: 1px solid black;
		margin: auto;
	} */
	.root.column{
		padding: 1em;
		box-shadow: 0 0 2px 0 rgba(0, 0, 0, 0.4);
	}
	.input{
		font-size: 2em;
    	padding: 0.3em;
    	margin: .1em;
    	box-shadow: 0 1px 2px 0px #00000052;
	}
</style>

<div class="root column" style="height: auto">
	<span class="input row center align">
		<span>{'( '}</span>
		<div

			contenteditable
			textContent={input}
			type="text"
			style="display: inline-block;min-width: 2vw;outline: none;"
			on:keydown={function (e) {
				if (e.keyCode != 8) {
					if (validate(e.key) === false) {
						e.preventDefault();
					} else {
						if (/\w/g.test(e.key)) {
							variables.add(e.key);
						}
						variables = variables;
					}
				}
			}}
			on:keyup={function (e) {
				let regex = /\w[+*]$|^[*+]\w$|\)\w?\(|\(\)|[+*][+*]|[+*]$|^[+*]|[!]$/g;
				if (!regex.test(e.target.textContent)) {
					if (isSyntaxValid(e.target.textContent)) {
						parser(e.target.textContent);
					}
				}
				input = e.target.textContent;
			}} />
		<span>{' )'}</span>
	</span>
	<p>{errors.join(' ')}</p>

	<div class="row">
		{#if input.length > 0}
			<table class="table" style="width: -webkit-fill-available;">
				<thead>
					<tr>
						{#each Array.from(variables) as variable}
							<th transition:fade|local={{ duration: 300 }}>{variable}</th>
						{/each}

					</tr>
				</thead>
				<tbody>
					{#each data as items, i}
						<tr>
							{#each items as value, ii}
								<td transition:fade|local={{ duration: 300 }}>{value}</td>
							{/each}

						</tr>
					{/each}
				</tbody>
			</table>
			<table class="table" style="width:auto;min-width: fit-content;">
				<thead>
					<tr>
						<th>{input}</th>
					</tr>
				</thead>
				<tbody>
					{#each res as item}
						<tr>
							<td transition:fade|local={{ duration: 300 }}>{item | ''}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		{/if}
	</div>
</div>
