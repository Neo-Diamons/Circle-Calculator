<script lang="ts">
	import { m } from '$lib/paraglide/messages.js';

	let selectedCalculation = $state('diameter');

	let diameter = $state(0);
	let angle = $state(0);
	let height = $state(0);
	let chordLength = $state(0);

	let asComputed = $state(false);

	let radius = $state(0);
	let arcLength = $state(0);

	function onsubmit(event: Event) {
		event.preventDefault();

		const form = event.target as HTMLFormElement;
		const formData = new FormData(form);

		try {
			switch (formData.get('calcul')) {
				case 'diameter': {
					diameter = Number(formData.get('diameter'));
					angle = Number(formData.get('angle'));

					if (diameter >= 0 && angle >= 0) {
						radius = diameter / 2;
						const angleInRadians = (angle * Math.PI) / 180;
						height = radius * (1 - Math.cos(angleInRadians / 2));
						arcLength = radius * angleInRadians;
						chordLength = 2 * radius * Math.sin(angleInRadians / 2);
					} else {
						throw new Error('Please provide both diameter and angle.');
					}
					break;
				}
				case 'height': {
					height = Number(formData.get('height'));
					chordLength = Number(formData.get('chord'));

					if (height >= 0 && chordLength >= 0) {
						radius = (chordLength * chordLength) / (8 * height) + height / 2;
						diameter = radius * 2;
						const angleInRadians = 2 * Math.asin(chordLength / (2 * radius));
						angle = (angleInRadians * 180) / Math.PI;
						arcLength = radius * angleInRadians;
					} else {
						throw new Error('Please provide both height and chord.');
					}
					break;
				}
				default:
					throw new Error('Unknown calculation type');
			}

			asComputed = true;
		} catch (error) {
			console.error((error as Error).message);
		}
	}
</script>

<main>
	<form {onsubmit}>
		<div class="field">
			<label for="calcul">{m.select_calculation()}</label>
			<select id="calcul" name="calcul" bind:value={selectedCalculation}>
				<option value="diameter">{m['calculation_options.diameter']()}</option>
				<option value="height">{m['calculation_options.height']()}</option>
			</select>
		</div>

		{#if selectedCalculation === 'diameter'}
			<div class="field">
				<label for="diameter">{m.diameter()} (mm)</label>
				<input type="number" id="diameter" name="diameter" min={0} placeholder="300" required />
			</div>
			<div class="field">
				<label for="angle">{m.angle()} (°)</label>
				<input
					type="number"
					id="angle"
					name="angle"
					min={0}
					max={360}
					placeholder="90"
					required
				/>
			</div>
		{:else if selectedCalculation === 'height'}
			<div class="field">
				<label for="height">{m.height()} (mm)</label>
				<input type="number" id="height" name="height" min={0} placeholder="50" required />
			</div>
			<div class="field">
				<label for="chord">{m.chord()} (mm)</label>
				<input type="number" id="chord" name="chord" min={0} placeholder="200" required />
			</div>
		{/if}

		<button type="submit">{m.calculate()}</button>
	</form>

	{#if asComputed}
		<div>
			<h2>{m.results()}</h2>
			<table>
				<thead>
					<tr>
						<th>{m.measure()}</th>
						<th>{m.value()}</th>
					</tr>
				</thead>
				<tbody>
					{#each [[m.radius(), radius, 'mm'], [m.diameter(), diameter, 'mm'], [m.angle(), angle, '°'], [m.arc_length(), arcLength, 'mm'], [m.chord_length(), chordLength, 'mm']] as [label, value, unit] (label)}
						<tr>
							<td class="text-left">{label}</td>
							<td class="text-right">{(value as number).toFixed(2)} {unit}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>

		<div>
			<h2>{m.visual_representation()}</h2>
			<div>
				<svg
					id="cercle-svg"
					width="100%"
					height="300"
					viewBox="0 0 300 300"
					xmlns="http://www.w3.org/2000/svg"
					style="max-width: 100%; display: block;"
				>
					{#if radius > 0}
						{@const svgSize = 300}
						{@const scale = svgSize / 2 / (radius * 1.2)}
						{@const centerX = svgSize / 2}
						{@const centerY = svgSize / 2}
						{@const angleRad = (angle * Math.PI) / 180}
						{@const endX = centerX + radius * scale * Math.cos(angleRad - Math.PI / 2)}
						{@const endY = centerY + radius * scale * Math.sin(angleRad - Math.PI / 2)}
						<!-- Reference circle -->
						<circle
							cx={centerX}
							cy={centerY}
							r={radius * scale}
							stroke="#ccc"
							stroke-width="1"
							fill="none"
						/>
						<!-- First radius -->
						<line
							x1={centerX}
							y1={centerY}
							x2={centerX}
							y2={centerY - radius * scale}
							stroke="black"
							stroke-width="1"
						/>
						<!-- Second radius -->
						<line x1={centerX} y1={centerY} x2={endX} y2={endY} stroke="black" stroke-width="1" />
						<!-- Angle label -->
						<text x={centerX + 10} y={centerY - 10} font-size="14" fill="black">
							{angle.toFixed(2)}°
						</text>
						<!-- Chord -->
						<line
							x1={centerX}
							y1={centerY - radius * scale}
							x2={endX}
							y2={endY}
							stroke="red"
							stroke-width="2"
						/>
						<!-- Chord length label -->
						<text
							x={centerX + 5}
							y={(centerY - radius * scale + endY) / 2}
							font-size="14"
							fill="red"
						>
							{chordLength.toFixed(2)} mm
						</text>
						<!-- Arc -->
						{@const arcStartX = centerX}
						{@const arcStartY = centerY - radius * scale}
						<path
							d={`M ${arcStartX} ${arcStartY} A ${radius * scale} ${radius * scale} 0 ${angle > 180 ? 1 : 0} 1 ${endX} ${endY}`}
							stroke="blue"
							stroke-width="2"
							fill="none"
						/>
						<!-- Arc length label -->
						<text
							x={(centerX + endX) / 2}
							y={(centerY - radius * scale + endY) / 2 + 20}
							font-size="14"
							fill="blue"
						>
							{arcLength.toFixed(2)} mm
						</text>
					{/if}
				</svg>
			</div>
		</div>
	{/if}
</main>

<style lang="postcss">
	@reference "tailwindcss";

	main {
		@apply mx-auto flex w-full max-w-2xl flex-col gap-8 p-8;

		form {
			@apply flex flex-col items-center border border-gray-300 p-2;

			.field {
				@apply mb-4 flex w-full flex-col;

				label {
					@apply mb-2 font-semibold;
				}
				label:before {
					content: '*';
					@apply mr-1 text-red-500;
				}
				label:after {
					content: ':';
				}
			}

			select,
			input {
				@apply rounded border border-gray-300 p-2;
			}

			button {
				@apply w-full rounded bg-blue-500 p-2 text-white hover:bg-blue-600;
			}
		}

		div {
			h2 {
				@apply mb-4 text-xl font-bold;
			}

			table {
				@apply w-full border-collapse;

				th,
				td {
					@apply border border-gray-300 p-2;
				}

				th {
					@apply text-center;
				}

				th {
					@apply bg-gray-200;
				}
			}

			#cercle-svg {
				@apply border border-gray-300;
			}
		}
	}
</style>
