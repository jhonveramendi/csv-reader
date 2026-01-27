<template>
	<div>
		<section id="container-tools">
			<input type="file" @change="getCsv" />
			<button @click="downloadCsv(dataCsv)">Descargar</button>
			<input type="checkbox" v-model="isTitle" />fila titulo
			<button @click="addColumn">Agregar Columna</button>
			<button @click="addRow">Agregar Fila</button>
      {{ maxColumn }}
		</section>
		<div style="width: 100%; height: 800px; overflow: scroll">
			<table>
				<tr>
					<td></td>
					<td
						:style="
							index === active.col
								? ' background-color: gray;  '
								: '  background-color: #64FF00; '
						"
						style="position: sticky; top: 0; color: #000"
						v-for="(col, index) in maxColumn"
						:key="index"
					>
						<div :style="'width:' + width + 'px'">
							{{ numToChars(index) }}
						</div>
					</td>
				</tr>
				<tr v-for="(dataRow, i) in dataCsv" :key="i">
					<td
						:style="
							i === active.row ? ' background-color: gray;' : '  background-color: #ccc; '
						"
						style="position: sticky; left: 0; color: #000; border: 1px solid #fff"
					>
						{{ i + 1 }}
					</td>
					<td v-for="(dataCol, j) in dataRow" :key="j">
						<input v-model="dataCsv[i][j]" @click="cellActive(i, j)" />
					</td>
				</tr>
			</table>
		</div>
	</div>
</template>

<script>
	import {computed, ref} from 'vue';
	export default {
		setup(props, {emit}) {
			const width = ref(175);
			const heigth = ref(50);
			const active = ref({});

			const data = ref('');
			const getCsv = function (e) {
				
				const file = e.target.files[0];
				const reader = new FileReader();
				reader.onload = function (e) {
					data.value = e.target.result;
					emit('update:modelValue', data.value);
          saveDataToLocalStorage(data.value);
				};
        
				reader.readAsText(file);
			};
			const csvToArray = function (str, delimiter = ',') {
				let rows = str.split('\n');
				for (let i = 0; i < rows.length; i++) {
					const element = rows[i].split(delimiter);
					rows[i] = element;
				}
				return rows;
			};

			const numToChars = function (num) {
				numToChars.letters =
					numToChars.letters || 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('');

				if (num > numToChars.letters.length - 1) {
					let first = num / 26;
					num = num % 26;
					return numToChars([Math.floor(first) - 1]) + numToChars.letters[num];
				} else {
					return numToChars.letters[num];
				}
			};
			const getMaxColumn = function (arr) {
				let max = 0;
				if (arr) {
					arr.forEach((elArr) => {
						if (elArr.length > max) {
							max = elArr.length;
						}
					});
				}
				return max;
			};

			const cellActive = function (indexRow, indexCol) {
				console.log('index', indexRow, indexCol);
				active.value = {
					row: indexRow,
					col: indexCol,
				};
			};

			const dataCsv = computed({
				get() {
					return csvToArray(data.value);
				},
				set(value) {
					emit('update:data', value);
				},
			});

			const maxColumn = computed(() => {
				return getMaxColumn(csvToArray(data.value));
			});

			const downloadCsv = (array) => {
				console.log('array', array);
				// (B) ARRAY TO CSV STRING
				let csv = '';
				array.forEach((row) => {
					row.forEach((col, index) => {
						csv += col;
						if (index !== row.length - 1) {
							csv += ',';
						}
					});
					csv += '\n';
				});
				var myBlob = new Blob([csv], {type: 'text/csv'});
				var url = window.URL.createObjectURL(myBlob);
				var anchor = document.createElement('a');
				anchor.href = url;
				anchor.download = 'demo.csv';
				anchor.click();
				window.URL.revokeObjectURL(url);
				anchor.remove();
			};

			const addColumn = () => {
				dataCsv.value.forEach((row) => {
					row.push('');
				});
				dataCsv.value[0][maxColumn.value] = 'Nueva columna';
			};

			const addRow = () => {
				const newRow = [];
				for (let i = 0; i < maxColumn.value; i++) {
					newRow.push('');
				}
				dataCsv.value.push(newRow);
			};

			const saveDataToLocalStorage = (data) => {
				localStorage.setItem('lastCsvData', JSON.stringify(data));
			};

			const loadLastCsvData = () => {
				const lastCsvData = localStorage.getItem('lastCsvData');
				if (lastCsvData) {
					this.data = JSON.parse(lastCsvData);
				}
			};
			return {
				dataCsv,
				numToChars,
				maxColumn,
				width,
				heigth,
				active,
				cellActive,
				downloadCsv,
				getCsv,

				addColumn,
				addRow,

        loadLastCsvData
			};
		},
	};
</script>

<style>
	#container-tools {
		margin: 20px;
	}
	table {
		border-collapse: collapse;
		width: 100%;
	}
	tr {
		border: 1px solid #ccc;
	}
	td {
		border: 1px solid #ccc;
	}
	td input {
		width: 100%;
		height: 100%;
		margin: 0;
		padding: 0;
		border: 0;
	}
</style>
